### Hey 👋

I am a software engineer from Russia. Now I am working at Yandex as a technical lead.

### Connect with me

[<img align="left" alt="linked-in" src="https://img.shields.io/badge/linkedin-%230077B5.svg?&style=for-the-badge&logo=linkedin&logoColor=white" />](https://www.linkedin.com/in/mohammad-faisal-2665b5134)


<br>

### Last blog posts (on russian)

<!-- BLOG-POST-LIST:START -->- 14/06/2021: **«[Typescript challenge: flatArray](https://amorgunov.com/posts/2021-06-14-typescript-challenge-flat-array/)»** - <p>Я уже давно пишу на тайпскрипте, но продолжаю открывать для себя новые возможности, подходы к написанию кода и проблемы компилятора. Чтобы аккумулировать новые знания, решил сделать серию постов по типизации различных функций и модулей в TypeScript.</p>
<p>В сегодняшнем (дебютном) посте попробуем написать типы для функции <code>flatArray</code> (упрощенный аналог <code>Array.prototype.flat</code> из ES2019), которая &quot;разворачивает&quot; подмассивы, т.е. все элементы из вложенных подмассивов поднимает на верхний уровень:</p>
<pre><code class="language-ts">const arr = ['foo', ['bar', 'baz', ['xyz']]];

flatArray(arr); // ['foo', 'bar', 'baz', 'xyz'];
</code></pre>
<p>Реализовать функцию можно через рекурсию, стек или воспользоваться встроенным методом flat. Но реализация нам сейчас не интересна, важно затипизировать функцию.</p>
<blockquote>
<p>Все примеры из поста удобно запускать в онлайн песочнице <a href="https://www.typescriptlang.org/play">https://www.typescriptlang.org/play</a></p>
</blockquote>
<h2 id="%D0%BF%D0%B5%D1%80%D0%B2%D1%8B%D0%B9-%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4">Первый подход <a class="direct-link" href="#%D0%BF%D0%B5%D1%80%D0%B2%D1%8B%D0%B9-%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4" aria-hidden="true">#</a></h2>
<p>Начнем с простого: описать исходных массив, состоящий из строк и массивов, содержащих строки и массивы, содержащие... Стоп, давайте выйдем из рекурсии и все же опишем такой тип:</p>
<pre><code class="language-ts">// Элемент массива (строка или массив строк)
type NestedArrayElement = string | string[];

// Исходный массив
type NestedArray = NestedArrayElement[];

const arr: NestedArray = ['foo', ['bar', 'baz', ['xyz']]];
                                                ^^^^^^^
</code></pre>
<p>Но код выполнится с ошибкой. Если внимательно посмотреть на типы, можно понять, почему TypeScript выкинет ее. Все потому, что в качестве элемента кроме массива строк могут быть, например, массивы массивов строк. К счастью, нам повезло, что мы живем в то время, когда TypeScript позволяет описать такую рекурсивную структуру. Достаточно заменить <code>string[]</code> на <code>NestedArrayElement[]</code>, то есть элемент может быть как строкой, так и вложенным массивом:</p>
<pre><code class="language-ts">// Элемент массива (строка или массив строк)
type NestedArrayElement = string | NestedArrayElement[];

// Исходный массив
type NestedArray = NestedArrayElement[];

const arr: NestedArray = ['foo', ['bar', 'baz', ['xyz']]];
</code></pre>
<p>Сделаем тип универсальным (не только для строк). Для этого воспользуемся дженериком (<a href="https://www.typescriptlang.org/docs/handbook/2/generics.html">документация</a>), заменив <code>string</code> на тип из дженерика <code>T</code>:</p>
<pre><code class="language-ts">type NestedArrayElement&lt;T&gt; = T | NestedArrayElement&lt;T&gt;[];

type NestedArray&lt;T = unknown&gt; = NestedArrayElement&lt;T&gt;[];
</code></pre>
<p>Укажем для <code>T</code> по умолчанию тип <code>unknown</code>, чтобы его не пришлось задавать явно. Тип готов, опишем функцию:</p>
<blockquote>
<p>С помощью <code>declare</code> можно декларировать фукнцию или переменную, не реализовывая ее.</p>
</blockquote>
<pre><code class="language-ts">declare function flatArray&lt;T&gt;(arr: NestedArray&lt;T&gt;): Array&lt;T&gt;;
</code></pre>
<p>В TypeScript есть прекрасная возможность выводить типы - <a href="https://www.typescriptlang.org/docs/handbook/type-inference.html"><em>infer type</em></a> - когда тип не описывается явно, а формируется на основе данных. Это и будет происходить с типом T внутри функции выше:</p>
<pre><code class="language-ts">// Тип stringArr будет string[]
const stringArr = flatArray([&quot;foo&quot;, [&quot;bar&quot;, [&quot;baz&quot;]]]);
// Тип mixedArr будет (string | number)[]
const mixedArr = flatArray([&quot;foo&quot;, 55, [&quot;bar&quot;, [&quot;baz&quot;]]]);
</code></pre>
<p>Неплохо, правда! В альтернативной вселенной на этом можно было заканчивать статью, так как функцию мы типизировали. Но, к сожалению, TypeScript не всегда может верно автоматически вывести тип в рекурсивных структурах, например у массива <code>[1, [&quot;foo&quot;]]</code> будет выведен тип <code>string[]</code> вместо <code>(string | number)[]</code>.</p>
<pre><code class="language-ts">const arr1 = flatArray(['foo', 5]); // Array&lt;string | number&gt;
const arr2 = flatArray(['foo', [5, 'bar']]); // Array&lt;string | number&gt;
const arr3 = flatArray(['foo', 5, ['bar']]); // Array&lt;string | number&gt;

const arr4 = flatArray(['foo', [5]]); // Array&lt;number&gt;
                        ^^^^^
//                      Type 'string' is not assignable to type
//                      'NestedArrayElement&lt;number&gt;'.
</code></pre>
<p>Почему так происходит? Я не знаю. Пол года назад я задавал этот вопрос на stackoverflow, создавал issue, но так и не получил ответа. Возможно когда то в будущем я все же найду силы залезть в недра компилятора и разобраться, почему вывод работает именно так, но не сегодня.</p>
<p>Чтобы избегать ошибку, можно явно передавать тип структуры при вызове метода (<code>flatArray&lt;string | number&gt;</code>), а можно перейти к следующей главе.</p>
<h2 id="%D0%BA%D0%BE%D0%BF%D0%B0%D0%B5%D0%BC-%D0%B3%D0%BB%D1%83%D0%B1%D0%B6%D0%B5">Копаем глубже <a class="direct-link" href="#%D0%BA%D0%BE%D0%BF%D0%B0%D0%B5%D0%BC-%D0%B3%D0%BB%D1%83%D0%B1%D0%B6%D0%B5" aria-hidden="true">#</a></h2>
<p>Выше была попытка описать тип входного массива. Можно пойти другим путем и сразу описать тип выходного массива (а тип исходного массива оставить без изменений). Функция будет выглядеть следующим образом:</p>
<pre><code class="language-ts">declare function flatArray&lt;T&gt;(arr: T): FlatArray&lt;T&gt;[];
</code></pre>
<p>С помощью <a href="https://www.typescriptlang.org/docs/handbook/type-inference.html"><code>infer</code> и <em>conditional types</em></a> можно вывести тип вложенных структур, в том числе и элементов массива в виде <em>union type</em> (объединения типов):</p>
<pre><code class="language-ts">type FlatArray&lt;T&gt; = T extends ReadonlyArray&lt;infer R&gt; ? R : T;

type ResulArr = FlatArray&lt;[1, 'bar', ['foo']]&gt;; // 1 | 'bar' | ['foo']
type ResultStr = FlatArray&lt;'bar'&gt;; // 'bar'
</code></pre>
<p>Что происходит в <code>FlatArray</code>? В типе проверяем, является ли T из дженерика подмножеством массива (<code>T extends ReadonlyArray&lt;infer R&gt;</code>). Если да, то возвращаем объединение типов элементов массива, иначе возвращаем исходный элемент.</p>
<p>Получилось развернуть верхний уровень массива. Чтобы развернуть все вложенные массивы, достаточно рекурсивно вызвать <code>FlatArray</code> на <code>R</code>:</p>
<pre><code class="language-ts">type FlatArray&lt;T&gt; = T extends ReadonlyArray&lt;infer R&gt; ? FlatArray&lt;R&gt; : T;

type ResulArr = FlatArray&lt;[1, 'bar', ['foo']]&gt;; // 1 | 'bar' | 'foo'
</code></pre>
<p>TypeScript при получении в качестве проверяемого значения объединение типов, выполнит проверку (в нашем случае на массив) каждый элемент по отдельности.</p>
<p>Проверим на проблемном кейсе из первой части:</p>
<pre><code class="language-ts">const arr4 = flatArray(['foo', [5]]); // Array&lt;string | number&gt;
</code></pre>
<p>Все работает, мы описали тип, поздравляю:</p>
<pre><code class="language-ts">type FlatArray&lt;T&gt; = T extends ReadonlyArray&lt;infer R&gt; ? FlatArray&lt;R&gt; : T;

declare function flatArray&lt;T&gt;(arr: T): FlatArray&lt;T&gt;[];
</code></pre>
<h2 id="%D1%80%D0%B5%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F-array.prototype.flat">Реализация Array.prototype.flat <a class="direct-link" href="#%D1%80%D0%B5%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F-array.prototype.flat" aria-hidden="true">#</a></h2>
<p>Метод <code>Array.prototype.flat</code> (<a href="https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Array/flat">документация на MDN</a>) интересен тем, что в него можно передать аргумент <em>depth</em> (глубина), на сколько уровней вложенности уменьшается мерность исходного массива (по умолчанию 1).</p>
<p>До мая 2020 года типы были описаны максимально просто (посмотреть можно <a href="https://github.com/microsoft/TypeScript/commit/35c1ba67baac2fd5152908184f8b2ec565815942#diff-d1641fc29156fd1998b9b563300edf5febc5a055428f976ef32337d74612f198L45-L222">здесь</a>), которые типизировали только массивы с элементами одного типа и ограниченной вложенности, до четвертого уровня, где все случаи были описаны вручную, например вот так:</p>
<pre><code class="language-ts">flat&lt;U&gt;(this:
    ReadonlyArray&lt;U[][][][]&gt; |

    ReadonlyArray&lt;ReadonlyArray&lt;U[][][]&gt;&gt; |
    ReadonlyArray&lt;ReadonlyArray&lt;U[][]&gt;[]&gt; |
    ReadonlyArray&lt;ReadonlyArray&lt;U[]&gt;[][]&gt; |
    ReadonlyArray&lt;ReadonlyArray&lt;U&gt;[][][]&gt; |

    ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;U[][]&gt;&gt;&gt; |
    ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;U&gt;[][]&gt;&gt; |
    ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;U&gt;&gt;[][]&gt; |
    ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;U&gt;[]&gt;[]&gt; |
    ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;U[]&gt;&gt;[]&gt; |
    ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;U[]&gt;[]&gt;&gt; |

    ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;U[]&gt;&gt;&gt;&gt; |
    ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;U&gt;[]&gt;&gt;&gt; |
    ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;U&gt;&gt;[]&gt;&gt; |
    ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;U&gt;&gt;&gt;[]&gt; |

    ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;ReadonlyArray&lt;U&gt;&gt;&gt;&gt;&gt;,
    depth: 4): U[];
</code></pre>
<p>Но некий Нейтон Сандерс написал вот такой тип, который используется до сих пор:</p>
<pre><code class="language-ts">type FlatArray&lt;Arr, Depth extends number&gt; = {
    &quot;done&quot;: Arr,
    &quot;recur&quot;: Arr extends ReadonlyArray&lt;infer InnerArr&gt;
        ? FlatArray&lt;InnerArr, [-1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20][Depth]&gt;
        : Arr
}[Depth extends -1 ? &quot;done&quot; : &quot;recur&quot;];
</code></pre>
<p>Этот тип так же разворачивает массив, но на каждом шаге уменьшает значение переменной <em>Depth</em>, тем самым ограничивая рекурсию до указанной глубины. Интересно, что максимальная глубина вложенности равна 21. Видимо, было предположение, что задача автоматически выводить тип у более глубоких структур не встречается в жизни или TS просто не справляется с таким.</p>
<pre><code class="language-ts">const arr = ['foo', ['bar', ['baz']]];

arr.flat(0); // Array&lt;string | (string | string[])[]&gt; - исходный тип
arr.flat(1); // Array&lt;string | string[]&gt; - развернули только первый уровень вложенности
arr.flat(2); // Array&lt;string&gt;
arr.flat(3); // Array&lt;string&gt;
</code></pre>
<hr>
<p>На основе этих трех примеров можно определить ваши навыки работы с TypeScript:</p>
<ul>
<li>Вы уверенный <strong>Junior TypeScript разработчик</strong>, если смогли бы реализовать первый вариант. Затипизировать 90% кода в проекте для Вас не проблема;</li>
<li>Вы <strong>Middle TypeScript разработчик</strong>, если умеете использовать infer и опциональные типы;</li>
<li>Вы <strong>Senior TypeScript разработчик</strong>, если смогли бы реализовать тип <code>FlatArray</code> из ES2019.</li>
</ul>
<p>Конечно же это просто шуточное деление на уровни и не стоит воспринимать его дословно. Но по мере того, как я изучал TypeScript, мне бы пригодилась такая разбивка, для понимания, на каком уровне я знаю TS.</p>
<p>На сегодня все, до следующего выпуска!</p>

- 08/12/2020: **«[Server side rendering в React](https://amorgunov.com/posts/2020-12-08-server-side-rendering-in-react/)»** - <h2 id="tl%3Bdr">Tl;dr <a class="direct-link" href="#tl%3Bdr" aria-hidden="true">#</a></h2>
<p>В существующее приложение на React поэтапно будем внедрять SSR, разбирая все до мелочей. Материл большой (пройдемся подробно по всем основным шагам) и для знакомства может быть избыточным, поэтому если вы раньше не слышали про SSR, то рекомендую начать со следующих туториалов:</p>
<ul>
<li><a href="https://www.freecodecamp.org/news/server-side-rendering-your-react-app-in-three-simple-steps-7a82b95db82e/">Server side rendering your react app in three simple steps</a></li>
<li><a href="https://www.digitalocean.com/community/tutorials/react-server-side-rendering">How to Enable Server-Side Rendering for a React App</a></li>
</ul>
<h2 id="%D0%B2%D1%81%D1%82%D1%83%D0%BF%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5">Вступление <a class="direct-link" href="#%D0%B2%D1%81%D1%82%D1%83%D0%BF%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5" aria-hidden="true">#</a></h2>
<p>Доброго времени суток, уважаемый читатель. Сегодня мы с нуля добавим в React приложение поддержку <em>Server side rendering</em> (SSR), сделав приложение изоморфным (работающим на стороне сервера и клиента).</p>
<p>SSR - это популярная техника для отрисовки приложений (в нашем случае это <em>client-side</em> одностраничные приложения) на стороне сервера и последующей отправкой полностью отрендереной страницы клиенту. Подход полезен для SEO (поисковые боты до сих пор не умеет правильно обрабатывать JavaScript), UX (пользователи сразу получают отрендеренные страницы) и лучших показателей метрик производительности.</p>
<p>Уже существует немало решений с поддержкой SSR из коробки (тот же <a href="https://nextjs.org/">Next.js</a>), но понимание, как все работает изнутри и тонкая настройка порой просто необходимы при разработке.</p>
<p>Это второй выпуск по SSR в React-е:</p>
<div class="post-series">
    <h4>Серия статей:</h4>
    <ol>
        <li><a href="https://amorgunov.com/posts/2019-05-04-how-do-work-with-cookies-in-universe-react-apps/">Работа с cookies в универсальных приложениях на React</a></li>
        <li>Server side rendering в React <em>(Этот пост)</em></li>
    </ol>
</div>
<p>Работать будем над самым типичным приложением для React-экосистемы. Но искусственные проекты без темы - скучные, поэтому сегодня будем готовить каталог кроссовок с главной страницей, каталогом и карточкой товара.</p>
<p><img style="width: 100px" class="lazyload" alt=" Иконка сайта" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/0.min.png" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/0.png"></p>
<p>Для достижения цели потребуется поработать над многими частями проекта: начнем с вебпак конфига для сервера, запустим сервер на NodeJS, подготовим роутинг, настроим подгрузку данных для Redux и сделаем многое другое. Изначальный и финальный варианты приложения выложены на Github (<a href="https://github.com/noveogroup-amorgunov/react-ssr-tutorial/tree/client-side-version">без SSR</a>, <a href="https://github.com/noveogroup-amorgunov/react-ssr-tutorial">с SSR</a>), а само приложение развернуто на Heroku и доступно по ссылке <a href="https://react-ssr-tutorial.herokuapp.com/">https://react-ssr-tutorial.herokuapp.com/</a>.</p>
<blockquote>
<p>Приложение написано на TypeScript (но вы можете повторить все шаги и на обычном JS, переименовав файлы с расширением .tsx на jsx и удалив TS типы):</p>
</blockquote>
<p>Быстрые переходы по частям:</p>
<ul>
<li><a href="#%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%82%D1%83%D1%80%D0%B0-react-%D0%BF%D1%80%D0%B8%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D1%8F">Структура React приложения</a></li>
<li><a href="#%D1%88%D0%B0%D0%B3-1.-%D0%BF%D0%B5%D1%80%D0%B2%D1%8B%D0%B5-%D1%88%D0%B0%D0%B3%D0%B8-ssr">Шаг 1. Первые шаги SSR</a></li>
<li><a href="#%D1%88%D0%B0%D0%B3-2.-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%B0%D0%B8%D0%B2%D0%B0%D0%B5%D0%BC-router-%D0%B8-redux">Шаг 2. Router и Redux</a></li>
<li><a href="#%D1%88%D0%B0%D0%B3-3.-meta-%D1%82%D0%B5%D0%B3%D0%B8">Шаг 3. Meta теги</a></li>
<li><a href="#%D1%88%D0%B0%D0%B3-4.-saga%2Fthunk-%D0%B8-%D0%B0%D1%81%D0%B8%D0%BD%D1%85%D1%80%D0%BE%D0%BD%D0%BD%D0%B0%D1%8F-%D0%BF%D0%BE%D0%B4%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85">Шаг 4. Saga/Thunk и асинхронная подгрузка данных</a></li>
<li><a href="#%D1%88%D0%B0%D0%B3-5.-code-splitting-%D0%B8-prefetch">Шаг 5. Code spitting + Prefetch</a></li>
<li><a href="#%D0%B7%D0%B0%D0%BA%D0%BB%D1%8E%D1%87%D0%B5%D0%BD%D0%B8%D0%B5">Заключение</a></li>
</ul>
<h2 id="%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%82%D1%83%D1%80%D0%B0-react-%D0%BF%D1%80%D0%B8%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D1%8F">Структура React приложения <a class="direct-link" href="#%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%82%D1%83%D1%80%D0%B0-react-%D0%BF%D1%80%D0%B8%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D1%8F" aria-hidden="true">#</a></h2>
<p>Еще раз скину <a href="https://github.com/noveogroup-amorgunov/react-ssr-tutorial/tree/client-side-version">ссылку на ветку в github</a>, в которой мы внедрим SSR. Вы можете склонировать проект и пошагово внедрять изменения. Проект использует базовый стек технологий: Webpack/React/Redux/Redux-saga:</p>
<pre><code>├─ src
  ├─ components
  ├─ pages
  ├─ store
  ├─ styles
  ├─ types
  ├─ client.tsx
  └─ index.html
├─ static/images
└─ webpack
</code></pre>
<p>Компонент <code>components/App</code> используется как layout и подключает все роуты, которые в свою очередь рендерят страницы из папки <code>pages</code>. Страницы - это обычные верхнеуровневые компоненты, которые забирают данные из редакса и рендерят контент.</p>
<p>В директории <code>store/ducks</code> лежат модули для работы с redux стором (подробнее про даки можно почитать <a href="https://medium.freecodecamp.org/scaling-your-redux-app-with-ducks-6115955638be">на медиуме</a>). Если работали с vue, то по организации очень похоже на модули во vuex:</p>
<pre><code>├─ components
├─ store
  └─ ducks
      └─ homepage
          ├─ actions.ts
          ├─ reducer.ts
          ├─ saga.ts
          ├─ selectors.ts
          ├─ service.ts
          └─ types.ts
├─ pages
├─ ...
</code></pre>
<p>Вы могли заметить, что нет директории <code>containers</code>. Приложение небольшое, логика для связи редакса и реактовских компонентов помещена в сами компоненты. Например, вот так выглядит компонент главной страницы (<code>Home.tsx</code>) - берем экшен, данные из селекторов и пробрасываем в компонент:</p>
<pre><code class="language-js">// src/pages/Home/Home.tsx

// Реализация компонента ...

const mapStateToProps = (state: State) =&gt; ({
    data: getHomepage(state),
    isLoading: isLoading(state),
});
const mapDispatchToProps = { fetchHomepage };

export default connect(
    mapStateToProps,
    mapDispatchToProps
)(Home);
</code></pre>
<blockquote>
<p>Недавно я писал статью об использовании хуков с редаксом, в которой остановился на варианте с использованием connect: <a href="https://amorgunov.com/posts/2020-04-12-use-redux-with-react-hooks/">https://amorgunov.com/posts/2020-04-12-use-redux-with-react-hooks/</a></p>
</blockquote>
<p>На самом деле в моих последних рабочих проектах подход разделения на контейнеры не использовался и никаких проблем не возникало, поэтому жизнь без контейнеров возможна и в больших приложениях.</p>
<p>В проекте не используются реальные API и все данные заранее подготовлены. Но при их получении используется задержка в 500мс для эмуляции реальной загрузки с внешнего ресурса:</p>
<pre><code class="language-js">import mockData from './mock.json';

export const fetchCatalog = () =&gt;
    timeout(500)
        .then(() =&gt; mockData)
        .then(data =&gt; data.items.map(shoesSerializer));
</code></pre>
<p>Конфиг вебпака вынесен в отдельную директорию и разбит на файлы (все лоадеры разнесены по отдельным файлам) для более удобного конфигурирования:</p>
<pre><code>├─ src
├─ static/images
└─ webpack
  ├─ loaders
    ├─ css.ts
    ├─ file.ts
    └─ js.ts
  ├─ env.ts
  └─ client.config.ts
</code></pre>
<h2 id="%D1%88%D0%B0%D0%B3-1.-%D0%BF%D0%B5%D1%80%D0%B2%D1%8B%D0%B5-%D1%88%D0%B0%D0%B3%D0%B8-ssr">Шаг 1. Первые шаги SSR <a class="direct-link" href="#%D1%88%D0%B0%D0%B3-1.-%D0%BF%D0%B5%D1%80%D0%B2%D1%8B%D0%B5-%D1%88%D0%B0%D0%B3%D0%B8-ssr" aria-hidden="true">#</a></h2>
<p>Ветка в <em>Github</em> по результатам этой части: <a href="https://github.com/noveogroup-amorgunov/react-ssr-tutorial/tree/01-prepare-webpack-and-express-server">https://github.com/noveogroup-amorgunov/react-ssr-tutorial/tree/01-prepare-webpack-and-express-server</a></p>
<h3 id="%D1%81%D1%85%D0%B5%D0%BC%D0%B0-%D0%B7%D0%B0%D0%BF%D1%83%D1%81%D0%BA%D0%B0">Схема запуска <a class="direct-link" href="#%D1%81%D1%85%D0%B5%D0%BC%D0%B0-%D0%B7%D0%B0%D0%BF%D1%83%D1%81%D0%BA%D0%B0" aria-hidden="true">#</a></h3>
<p>Сначала нужно определиться, как будет запускаться приложение. Текущая схема довольна проста: запускается <em>webpack-dev-server</em> (который внутри себя поднимает сервер).</p>
<p><img class="lazyload" alt="Текущая схема запуска проекта" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/1.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/1.jpg"></p>
<p>С SSR запуск немного усложняется. Хоть приложение и универсальное, теперь понадобятся две точки входа в приложение - серверная и клиентская. Серверный бандл будет отрабатывать на сервере и нужен для формирования html-страницы, клиентский - обычный js/css файлы, которые скачиваются и запускаются в браузере. Серверный бандл не нужно разбивать на чанки, не нужно минимизировать и вообще можно обойти большинство обработок (не собирать css, не собирать модули из node_modules), которые нужны для клиентского бандла. А это значит, что нужно запускать <em>Webpack</em> для сборки под каждую среду (конфиг рассмотрим далее).</p>
<p>Запускать сервер нужно самим, как и перезапускать при изменении бандла для режима разработки. Посмотрим на схему запуска проекта с SSR:</p>
<p><img class="lazyload" alt="Схема запуска проекта с SSR" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/2.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/2.jpg"></p>
<p>Перезапускать сервер будет Nodemon, а собирать бандлы - Webpack.</p>
<p>Установим необходимые зависимости:</p>
<pre><code class="language-bash">npm i --save express compression
npm i --save-dev nodemon null-loader webpack-node-externals npm-run-all @types/express @types/webpack-node-externals
</code></pre>
<ul>
<li><code>express</code> - Веб-сервер</li>
<li><code>compression</code> - Для сжатия статики</li>
<li><code>nodemon</code> - CLI для перезапуска веб-сервера</li>
<li><code>null-loader</code> - Loader для серверного конфига вебпака</li>
<li><code>webpack-node-externals</code> - Плагин для серверной сборки</li>
<li><code>@types/*</code> - TS-тайпинги</li>
</ul>
<p>И обновим секцию со скриптами в <em>package.json</em>:</p>
<div class="code-path">package.json</div>
<pre><code class="language-diff">- &quot;start&quot;: &quot;NODE_ENV=development webpack serve --hot --mode development --config webpack/client.config.ts&quot;,
+ &quot;start:webpack&quot;: &quot;webpack --mode=development --watch&quot;,
+ &quot;start:server&quot;: &quot;nodemon index.js --watch dist/server.js&quot;,
+ &quot;start&quot;: &quot;NODE_ENV=development npm-run-all --print-label --parallel start:*&quot;
</code></pre>
<blockquote>
<p>Webpack будет собирать код параллельно для двух сред, еще и Nodemon будет перезапускать сервер, поэтому для понимания, что сейчас собирается, очень помогает опция <code>--print-label</code> для <code>npm-run-all</code>, которая будет выводить лейбл выполняющегося процесса перед каждой строкой лога в терминале:</p>
</blockquote>
<p><img class="lazyload" alt="Вывод лейбла в термине" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/3.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/3.jpg"></p>
<h3 id="%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80-%D0%BD%D0%B0-express">Сервер на express <a class="direct-link" href="#%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80-%D0%BD%D0%B0-express" aria-hidden="true">#</a></h3>
<p>На клиенте метод <code>ReactDOM.render</code> заменим на <code>ReactDOM.hydrate</code>. Он строит приложение не с нуля, а на основе html-разметки, которая сгенерирована на сервере, что работает в разы быстрее, так как не требуется заново генерировать DOM:</p>
<div class="code-path">src/client.tsx</div>
<pre><code class="language-diff">- ReactDOM.render(
+ ReactDOM.hydrate(
    &lt;ReduxProvider store={store}&gt;
        &lt;BrowserRouter&gt;
            &lt;App /&gt;
        &lt;/BrowserRouter&gt;
    &lt;/ReduxProvider&gt;,
    document.getElementById('mount'),
);
</code></pre>
<p>Далее нам понадобится веб-сервер, в котором на все запросы с помощью метода <code>renderToString()</code> сгенерируем из приложения html-строку (пока без HOC-ов для <em>Redux</em> и <em>ReactRouter</em>). Сразу вынесем рендеринг приложения в отдельный файл (миддлевару), а запуск сервера в <code>index.js</code> файл.</p>
<blockquote>
<p>В реальных приложениях в файле <code>app.js|ts</code> принято экспортировать сервер, а запускать его отдельно. Это удобно как для интеграционных тестов (чтобы не запускать реальный сервер), так и для эксплуатации (например, возможность запускаться через pm2).</p>
</blockquote>
<br>
<p>В отдельном файле запускаем сервер на порту 9001 из собранного файла <code>./dist/server.js</code>. Заметьте, это JavaScript файл, который можно запускать из NodeJS без трансформаций и компиляций.</p>
<div class="code-path">index.js</div>
<pre><code class="language-javascript">const { app } = require('./dist/server.js');

const port = process.env.PORT || 9001;

app.listen(port, () =&gt; {
    console.log('Application is started on localhost:', port);
});
</code></pre>
<p>В файле <code>server.ts</code> создаем express-приложение с одной миддлеварой на все принимаемые запросы:</p>
<div class="code-path">src/server.ts</div>
<pre><code class="language-typescript">import path from 'path';
import express from 'express';
import compression from 'compression';
import 'babel-polyfill';
import serverRenderMiddleware from './server-render-middleware';

const app = express();

// Рекомендую использовать только для разработки
// А в production раздавать статику через Nginx или CDN
app.use(compression())
    .use(express.static(path.resolve(__dirname, '../dist')))
    .use(express.static(path.resolve(__dirname, '../static')));

app.get('/*', serverRenderMiddleware);

export { app };
</code></pre>
<p>А в файле <code>server-render-middleware.tsx</code> отрендерим <em>JSX</em> в html:</p>
<div class="code-path">src/server-render-middleware.tsx</div>
<pre><code class="language-typescript">import React from 'react';
import { renderToString } from 'react-dom/server';
import { Request, Response } from 'express';
import { App } from './components/App/App';

export default (req: Request, res: Response) =&gt; {
    const jsx = (&lt;App /&gt;);
    const reactHtml = renderToString(jsx);

    res.send(getHtml(reactHtml));
};

// function getHtml(reactHtml: string) {}
</code></pre>
<p>Далее полученную строку вставляем в заготовленную html-обертку, которую и отдаем клиенту:</p>
<div class="code-path">src/server-render-middleware.tsx</div>
<pre><code class="language-ts">function getHtml(reactHtml: string) {
    return `
    &lt;!DOCTYPE html&gt;
    &lt;html lang=&quot;en&quot;&gt;
    &lt;head&gt;
        &lt;meta charset=&quot;UTF-8&quot;&gt;
        &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
        &lt;meta http-equiv=&quot;X-UA-Compatible&quot; content=&quot;ie=edge&quot;&gt;
        &lt;link rel=&quot;shortcut icon&quot; type=&quot;image/png&quot; href=&quot;/images/favicon.jpg&quot;&gt;
        &lt;title&gt;Sneakers shop&lt;/title&gt;
        &lt;link href=&quot;/main.css&quot; rel=&quot;stylesheet&quot;&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;div id=&quot;mount&quot;&gt;${reactHtml}&lt;/div&gt;
        &lt;script src=&quot;/main.js&quot;&gt;&lt;/script&gt;
    &lt;/body&gt;
    &lt;/html&gt;
    `;
}
</code></pre>
<p>Так как веб-сервер раздает директорию <code>dist</code>, то стили и JS можно подключить напрямую. Шаблон html-страницы <code>src/index.html</code> больше не понадобится и его можно удалить.</p>
<h3 id="%D0%BE%D0%B1%D0%BD%D0%BE%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-webpack-%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3%D0%B0">Обновление webpack конфига <a class="direct-link" href="#%D0%BE%D0%B1%D0%BD%D0%BE%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-webpack-%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3%D0%B0" aria-hidden="true">#</a></h3>
<p>Чтобы <em>NodeJS</em> могла работать с <em>JSX</em> (да и <em>TypeScript</em>-ом), нужно собирать серверный код через <em>Webpack</em>, но не так, как для клиента. Для этого нужно создать отдельный конфиг для сборки серверного бандла. В этом конфиге нужно сделать две вещи:</p>
<ul>
<li>Не собирать в бандл код из стандартных библиотек типа <code>path</code>, <code>fs</code> и из <em>node_modules</em> библиотек с помощью плагина <code>webpack-node-externals</code>:</li>
</ul>
<div class="code-path">webpack/server.config.ts</div>
<pre><code class="language-typescript">{
    target: 'node',
    externals: [
        nodeExternals({ allowlist: [/\.(?!(?:tsx?|json)$).{1,5}$/i] })
    ],
}
</code></pre>
<p>Помимо этого нужно поправить поле <em>output</em>, чтобы собрать все в один файл. Полный конфиг можете <a href="https://github.com/noveogroup-amorgunov/react-ssr-tutorial/blob/01-prepare-webpack-and-express-server/webpack/server.config.ts">взять в репозитории</a>.</p>
<ul>
<li>Не нужно собирать <em>CSS</em> и картинки на сервере. Это можно сделать c помощью <code>NullLoader</code>.</li>
</ul>
<blockquote>
<p>Замечу, что это может не сработать для CSS-Modules или Styled-Components, только для обычного CSS.</p>
</blockquote>
<br>
<div class="code-path">webpack/loaders/css.ts</div>
<pre><code class="language-diff">export default {
    client: {
        test: /\.css$/,
        use: [...loaders]
    },
+   server: {
+       test: /\.css$/,
+       loader: 'null-loader',
+   },
};
</code></pre>
<p>Так же нужно добавить лоадеры для JS и файлов (посмотреть можно <a href="https://github.com/noveogroup-amorgunov/react-ssr-tutorial/tree/01-prepare-webpack-and-express-server/webpack/loaders">тут</a>). На вход <em>Webpack</em> можно подать массив из двух конфигов, и он соберет бандлы для каждого:</p>
<div class="code-path">./webpack.config.ts</div>
<pre><code class="language-diff">import clientConfig from './webpack/client.config';
+ import serverConfig from './webpack/server.config';

module.exports = [
    clientConfig,
+   serverConfig
];
</code></pre>
<p>На этом с подготовкой все, но при запуске и переходе на <em>localhost:9001</em> получим ошибку <em>Error: Invariant failed: You should not use <NavLink> outside a <Router></Router></NavLink></em>. Но она ожидаема, так как мы не оборачивали наше приложение в RouterProvider, что сделаем следующим шагом.</p>
<h2 id="%D1%88%D0%B0%D0%B3-2.-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%B0%D0%B8%D0%B2%D0%B0%D0%B5%D0%BC-router-%D0%B8-redux">Шаг 2. Настраиваем Router и Redux <a class="direct-link" href="#%D1%88%D0%B0%D0%B3-2.-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%B0%D0%B8%D0%B2%D0%B0%D0%B5%D0%BC-router-%D0%B8-redux" aria-hidden="true">#</a></h2>
<p>Ветка в <em>Github</em> по результатам этой части: <a href="https://github.com/noveogroup-amorgunov/react-ssr-tutorial/tree/02-add-redux-and-react-router">https://github.com/noveogroup-amorgunov/react-ssr-tutorial/tree/02-add-redux-and-react-router</a></p>
<h3 id="%D1%80%D0%BE%D1%83%D1%82%D0%B5%D1%80">Роутер <a class="direct-link" href="#%D1%80%D0%BE%D1%83%D1%82%D0%B5%D1%80" aria-hidden="true">#</a></h3>
<p>На сервере нет доступа к <em>location</em> и <em>history</em> объектам, из-за чего нет возможности использовать компонент <em>Router</em>. Но можно использовать <em>StaticRouter</em>, в который мы можем явно передать текущий url из запроса.</p>
<div class="code-path">./src/server-render-middleware.tsx</div>
<pre><code class="language-diff">+ import { StaticRouter } from 'react-router-dom';
+ import { StaticRouterContext } from 'react-router';

export default (req: Request, res: Response) =&gt; {
+   const location = req.url;
+   const context: StaticRouterContext = {};

    const jsx = (
+        &lt;StaticRouter context={context} location={location}&gt;
            &lt;App /&gt;
+        &lt;/StaticRouter&gt;
    );
    const reactHtml = renderToString(jsx);

    // ...
};
</code></pre>
<p>Вы можете заметить еще одно свойство - <code>context</code>. В него роутер записывает информацию об изменении локейшена при рендеринге приложения. Например, если внутри реактовского приложения произошел редирект (<code>&lt;Redirect to={to} /&gt;</code>), то нужно выполнить этот редирект сразу на сервере. Для этого достаточно проверить, что в <em>context</em> есть свойство <code>url</code>.</p>
<pre><code class="language-typescript">export default (req: Request, res: Response) =&gt; {
    // ...

    if (context.url) {
        res.redirect(context.url);
        return;
    }

    // ...
};
</code></pre>
<p>И наконец в <em>context</em> можно записать <em>statusCode</em> прямо внутри приложения и вернуть страницу браузеру с нужным статусом.</p>
<pre><code class="language-typescript">export default (req: Request, res: Response) =&gt; {
    // ...

    res
        .status(context.statusCode || 200)
        .send(getHtml(reactHtml));
};
</code></pre>
<p>Как установить статус, можете посмотреть в компоненте Status в <a href="https://github.com/noveogroup-amorgunov/react-ssr-tutorial/blob/master/src/pages/404/404.tsx#L12">src/pages/404/404.tsx</a>.</p>
<h2 id="redux">Redux <a class="direct-link" href="#redux" aria-hidden="true">#</a></h2>
<p>C Redux все проще. Нужно формировать стейт на сервере и передавать его в сериализованном виде на клиент. А на клиенте использовать этот стейт в качестве начального состояния. Также при необходимости можно задиспатчить инициализирующиеся экшены.</p>
<p>Немного освежим файл <code>src/store/rootStore.ts</code>, в котором инициализируется <em>Redux store</em>. Создадим хелпер <code>isServer</code>:</p>
<div class="code-path">./src/store/rootStore.ts</div>
<pre><code class="language-typescript">export const isServer = !(
    typeof window !== 'undefined' &amp;&amp;
    window.document &amp;&amp;
    window.document.createElement
);
</code></pre>
<p>Этот хелпер можно вынести в общие утилиты и использовать везде, где необходима различная логика работы на сервере и клиенте. Изначально я его считал костылем, который лучше не использовать, но на всех проектах с SSR встречал его в каком-либо виде.</p>
<p>Сразу используем его в 3 местах: не будем на сервере подключать <em>DevTools</em> плагин для <em>Redux</em>, не будем запускать <em>Saga</em> и для <em>connected-react-router</em> (используется для хранения состояния роутера в <em>Redux</em>) объект history будем брать из <em>createMemoryHistory</em> (который используется как раз для серверного рендеринга):</p>
<div class="code-path">./src/store/rootStore.ts</div>
<pre><code class="language-diff">function getComposeEnhancers() {
-    if (process.env.NODE_ENV !== 'production') {
+    if (process.env.NODE_ENV !== 'production' &amp;&amp; !isServer) {
        return window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;
    }

    return compose;
}

// ...

export function configureStore(initialState: State, url = '/') {
+    const history = isServer
+        ? createMemoryHistory({ initialEntries: [url] })
+        : createBrowserHistory();

    // ...

+    if (!isServer) {
            sagaMiddleware.run(rootSaga);
+    }
</code></pre>
<p>Далее обернем приложение на сервере в провайдер:</p>
<div class="code-path">./src/server-render-middleware.tsx</div>
<pre><code class="language-diff">+ import { Provider as ReduxProvider } from 'react-redux';
+ import { configureStore } from './store/rootStore';
+ import { getInitialState } from './store/getInitialState';

export default (req: Request, res: Response) =&gt; {
    // ...
+   const { store } = configureStore(getInitialState(), location);

    const jsx = (
+        &lt;ReduxProvider store={store}&gt;
            &lt;StaticRouter context={context} location={location}&gt;
                &lt;App /&gt;
            &lt;/StaticRouter&gt;
+        &lt;/ReduxProvider&gt;
    );
    const reactHtml = renderToString(jsx);

    // ...
};
</code></pre>
<p>При необходимости можно сразу задиспатчить желаемый экшен, но у нас в проекте таких нет:</p>
<pre><code class="language-typescript">const { store } = configureStore(getInitialState(), location);
store.dispatch(initializeSession());
</code></pre>
<p>Получаем сформированный стейт и добавим его в нашу html-обертку...</p>
<div class="code-path">./src/server-render-middleware.tsx</div>
<pre><code class="language-diff">export default (req: Request, res: Response) =&gt; {
    const reactHtml = renderToString(jsx);
+   const reduxState = store.getState();

+   res.send(getHtml(reactHtml, reduxState));
};
</code></pre>
<p>В <em>getHtml</em> добавим переменную <code>window.__INITIAL_STATE__</code>, в которую положим <code>reduxState</code>, чтобы он был доступен на клиенте:</p>
<div class="code-path">./src/server-render-middleware.tsx</div>
<pre><code class="language-html">&lt;div id=&quot;mount&quot;&gt;${reactDom}&lt;/div&gt;
&lt;script&gt;
  window.__INITIAL_STATE__ = ${JSON.stringify(reduxState)}
&lt;/script&gt;
&lt;script src=&quot;/main.js&quot;&gt;&lt;/script&gt;
</code></pre>
<blockquote>
<p>Если данные в стейте могут задавать пользователи (<em>UGC</em>) или они формируется из внешних API, то возможна XSS уязвимость. В таких случаях данные нужно проверять и как минимум экранизировать.</p>
</blockquote>
<p>В браузере в исходном коде страницы мы можем увидеть эту переменную со сформированным на сервере стейтом:</p>
<p><img class="lazyload" alt="Redux-стейт сформированный на сервере" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/4.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/4.jpg"></p>
<p>На клиенте ее считываем и прокидываем при создании стора в качестве <em>preloadedState</em>.</p>
<div class="code-path">./src/client.tsx</div>
<pre><code class="language-typescript">const initialState = window.__INITIAL_STATE__;
const { store, history } = configureStore(initialState);
</code></pre>
<p>На этом моменте проснется <em>TypeScript</em>, и скажет, что в <em>window</em> нет такой переменной. Для этого можно доопределить глобальный интерфейс:</p>
<div class="code-path">./src/client.tsx</div>
<pre><code class="language-typescript">import { State } from 'types';

declare global {
    interface Window {
        __INITIAL_STATE__: State;
    }
}
</code></pre>
<p>Кстати да, запустив приложение, оно будет работать, на клиент будет приезжать html, но пока без подгруженных данных:</p>
<p><img class="lazyload" alt="Работа приложения" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/5.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/5.gif"></p>
<h2 id="%D1%88%D0%B0%D0%B3-3.-meta-%D1%82%D0%B5%D0%B3%D0%B8">Шаг 3. Meta-теги <a class="direct-link" href="#%D1%88%D0%B0%D0%B3-3.-meta-%D1%82%D0%B5%D0%B3%D0%B8" aria-hidden="true">#</a></h2>
<p>Ветка в <em>Github</em> по результатам этой части: <a href="https://github.com/noveogroup-amorgunov/react-ssr-tutorial/tree/03-add-react-helmet">https://github.com/noveogroup-amorgunov/react-ssr-tutorial/tree/03-add-react-helmet</a></p>
<p><em>Helmet</em> - это библиотека, с помощью которой можно внутри React задавать тайтл страницы и мета-теги прямо из компонент. Обычно helmet используется в компонентах-страницах:</p>
<pre><code>&lt;Helmet&gt;
    &lt;title&gt;Home page&lt;/title&gt;
    &lt;meta
        name=&quot;description&quot;
        content=&quot;Buy awesome snickers&quot; /&gt;
&lt;/Helmet&gt;
</code></pre>
<p>В проекте для удобства создан компонент враппер <a href="https://github.com/noveogroup-amorgunov/react-ssr-tutorial/blob/master/src/components/PageMeta/PageMeta.tsx">PageMeta</a>, в котором создаются как стандартные теги, так и для социальных сетей.</p>
<p>Процесс подключения на стороне сервера <a href="https://github.com/nfl/react-helmet#server-usage">описан в документации</a> и он довольно простой. После добавления мета-тегов на компоненты-страницы, на сервере нужно воспользоваться методом <code>Helmet.renderStatic()</code>, который вернет все метатеги и вставить их в html-заготовку:</p>
<div class="code-path">./src/server-render-middleware.tsx</div>
<pre><code class="language-diff">+ import Helmet, { HelmetData } from 'react-helmet';
// ...
const reactHtml = renderToString(jsx);
const reduxState = store.getState();
+ const helmetData = Helmet.renderStatic();

- res.send(getHtml(reactHtml, reduxState));
+ res.send(getHtml(reactHtml, reduxState, helmetData));

// ...

- function getHtml(reactHtml: string, reduxState = {}, helmetData: HelmetData) {
+ function getHtml(reactHtml: string, reduxState = {}, helmetData: HelmetData) {
    // ...
-   &lt;title&gt;Sneakers shop&lt;/title&gt;
    &lt;link href=&quot;/main.css&quot; rel=&quot;stylesheet&quot;&gt;
+   ${helmetData.title.toString()}
+   ${helmetData.meta.toString()}
</code></pre>
<p>Теперь страницы будут понятны поисковикам и ботам для шаринга в социальных сетях:</p>
<p><img class="lazyload" alt="Мета теги в разметке" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/6.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/6.jpg"></p>
<p>Далее мы настроим подгрузку данных на сервере и вот так страница будет отображаться в превью телеграма:</p>
<p><img class="lazyload" alt="Превью проекта в телеграме" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/7.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/7.jpg"></p>
<h2 id="%D1%88%D0%B0%D0%B3-4.-saga%2Fthunk-%D0%B8-%D0%B0%D1%81%D0%B8%D0%BD%D1%85%D1%80%D0%BE%D0%BD%D0%BD%D0%B0%D1%8F-%D0%BF%D0%BE%D0%B4%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85">Шаг 4. Saga/Thunk и асинхронная подгрузка данных <a class="direct-link" href="#%D1%88%D0%B0%D0%B3-4.-saga%2Fthunk-%D0%B8-%D0%B0%D1%81%D0%B8%D0%BD%D1%85%D1%80%D0%BE%D0%BD%D0%BD%D0%B0%D1%8F-%D0%BF%D0%BE%D0%B4%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85" aria-hidden="true">#</a></h2>
<p>Ветка в <em>Github</em> по результатам этой части: <a href="https://github.com/noveogroup-amorgunov/react-ssr-tutorial/tree/04-add-redux-saga-and-async-data">https://github.com/noveogroup-amorgunov/react-ssr-tutorial/tree/04-add-redux-saga-and-async-data</a></p>
<h3 id="redux-saga">Redux-saga <a class="direct-link" href="#redux-saga" aria-hidden="true">#</a></h3>
<p>Вы можете пропустить этот раздел, если используете что-то другое. Саги, построенные на генераторах, нужны для работы с асинхронными вещами, с которыми обычный редакс не работает. Одни из самых частых асинхронных операций в приложении - это отправка запросов в API за данными - как раз наш случай.</p>
<p>На клиенте сага запускается в фоновом режиме и &quot;бесконечно крутится&quot;, слушая события из редакса (живет своей жизнью, отдельной от реакт приложения). На сервере же у нас нет такой возможности, нам нужно загрузить данные и поскорее отправить ответ клиенту.</p>
<p>Для остановки саги есть специальный экшен <code>END</code>. Самое классное, что сага сперва обработает все запущенные операции, и только после закончит выполнение. Процесс работы выглядит следующим образом: запускаем сагу на сервере, скармливаем ей экшены, после диспатчим <code>END</code> и ждем, пока сага завершит выполнение. Посмотрим, как будет выглядеть в коде.</p>
<p>Добавим тип <code>AppStore</code>, в котором опишем два метода (для запуска и остановки саги):</p>
<div class="code-path">./src/types/redux.ts</div>
<pre><code class="language-ts">import { Store } from 'redux';
import { SagaMiddleware } from '@redux-saga/core';

export type AppStore = Store &amp; {
    runSaga: SagaMiddleware['run'];
    close: () =&gt; void;
};
</code></pre>
<p>И добавим методы в объект стора:</p>
<div class="code-path">./src/store/rootStore.ts</div>
<pre><code class="language-diff">+ import createSagaMiddleware, { END, SagaMiddleware } from 'redux-saga';

const store = createStore(
    createRootReducer(history),
    initialState,
    composeEnhancers(applyMiddleware(...middlewares))
- );
+ ) as AppStore;

+ // Методы для использования на сервере
+ store.runSaga = sagaMiddleware.run;
+ store.close = () =&gt; store.dispatch(END);
</code></pre>
<p>Запустим сагу на сервере. Все, что было в миддлеваре для рендеринга перенесем в метод <code>renderApp</code>, который будем вызывать после того, как отработает сага.</p>
<div class="code-path">./src/server-render-middleware.tsx</div>
<pre><code class="language-diff">+ import rootSaga from './store/rootSaga';

export default (req: Request, res: Response) =&gt; {
    const location = req.url;
    const context: StaticRouterContext = {};
    const { store } = configureStore(getInitialState(location), location);

+    function renderApp() {
+       // (5)
        const jsx = (
            &lt;ReduxProvider store={store}&gt;
                &lt;StaticRouter context={context} location={location}&gt;
                    &lt;App /&gt;
                &lt;/StaticRouter&gt;
            &lt;/ReduxProvider&gt;
        );
        const reactHtml = renderToString(jsx);
        const reduxState = store.getState();
        const helmetData = Helmet.renderStatic();
    
        if (context.url) {
            res.redirect(context.url);
            return;
        }
    
        res.status(context.statusCode || 200).send(
            getHtml(reactHtml, reduxState, helmetData)
        );
+    }

+    // (1)
+    store 
+        .runSaga(rootSaga)
+        .toPromise()
+        // (4)
+        .then(() =&gt; renderApp())
+        .catch(err =&gt; { throw err; });

+    // (2)
+    // TODO: Добавить все асинхронные вещи в массив dataRequirements
+    const dataRequirements: (Promise&lt;void&gt; | void)[] = [];

+    // Когда все асинхронные экшены будут закончены
+    // вызываем экшен для закрытия саги

+    // (3)
+    return Promise.all(dataRequirements)
+        .then(() =&gt; store.close())
+        .catch(err =&gt; { throw err; });
};
</code></pre>
<p>Разберем все по шагам:</p>
<ol>
<li>
<p>Запускаем сагу с помощью <code>store.runSaga(rootSaga).toPromise()</code>. Промис зарезолвится тогда, когда сага получит экшен END и обработает все текущие запущенные генераторы.</p>
</li>
<li>
<p>Собираем в массив функции, которые выполняют какую-то асинхронную работу (далее мы в этом месте будем подгружать данные с API для конкретной страницы).</p>
</li>
<li>
<p>Ждем пока зарезолвятся все промисы из массива выше. Это действие не нужно для саги (он нужен для <em>redux-thunk</em>), так как для нее достаточно просто задиспатчить синхронный экнеш. После диспачим экшен <code>END</code>.</p>
</li>
<li>
<p>Вызываем функцию <code>renderApp</code>. На данном этапе стор уже заполнен данными, осталось отрендерить приложение.</p>
</li>
</ol>
<h3 id="%D0%B0%D1%81%D0%B8%D0%BD%D1%85%D1%80%D0%BE%D0%BD%D0%BD%D0%B0%D1%8F-%D0%BF%D0%BE%D0%B4%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85">Асинхронная подгрузка данных <a class="direct-link" href="#%D0%B0%D1%81%D0%B8%D0%BD%D1%85%D1%80%D0%BE%D0%BD%D0%BD%D0%B0%D1%8F-%D0%BF%D0%BE%D0%B4%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85" aria-hidden="true">#</a></h3>
<p>Сага настроена, теперь самый интересный шаг. Для каждой (почти) страницы необходимо подгружать данные. Другими словами в зависимости от текущего роута делать запрос в API энпоинты, чтобы заполнить стор. Соберем все требования в пункты:</p>
<ul>
<li>Нужно перед рендером заполнить стор данными;</li>
<li>Данные зависят от текущей страницы;</li>
<li>Данные загружаются с помощью redux-saga/thunk;</li>
<li>На клиенте не загружать данные повторно.</li>
</ul>
<p>Вынесем роуты в отдельный файл. Сравнивая path с текущим адресом страницы мы сможем определить нужный компонент, который необходимо отрендерить:</p>
<div class="code-path">./src/routes.ts</div>
<pre><code class="language-ts">import { fetchCatalog } from 'store/ducks/catalog/actions';
import { fetchHomepage } from 'store/ducks/homepage/actions';
import { fetchShoes } from 'store/ducks/shoes/actions';
import { AppRouterProps } from 'types';

import CatalogPage from 'pages/Catalog/Catalog';
import UpcomingPage from 'pages/Upcoming/Upcoming';
import SneakersPage from 'pages/Sneakers/Sneakers';
import HomePage from 'pages/Home/Home';
import NotFoundPage from 'pages/404/404';

export default [
    {
        path: '/',
        component: HomePage,
        exact: true,
    },
    {
        path: '/catalog',
        component: CatalogPage,
        exact: true,
    },
    {
        path: '/sneakers/:slug',
        component: SneakersPage,
        exact: true,
    },
    {
        path: '/upcoming',
        component: UpcomingPage,
        exact: true,
    },
    {
        path: '*',
        component: NotFoundPage,
        exact: true,
    },
];
</code></pre>
<p>Добавим в каждый роут, которому нужны данные с сервера, метод <code>fetchData</code> (название можно выбрать любое), в котором, в случае саги достаточно просто диспатчить необходимые экшены, в случае с <em>redux-thunk</em> - возвращать промисы:</p>
<div class="code-path">./src/routes.ts</div>
<pre><code class="language-diff">export default [
    {
        path: '/',
        component: HomePage,
        exact: true,
+       fetchData({ dispatch }: RouterFetchDataArgs) {
+         dispatch(fetchHomepage());
+       },
    },
    {
        path: '/catalog',
        component: CatalogPage,
        exact: true,
+       fetchData({ dispatch }: RouterFetchDataArgs) {
+           dispatch(fetchCatalog());
+       },
    },
    {
        path: '/sneakers/:slug',
        component: SneakersPage,
        exact: true,
+       fetchData({ dispatch, match }: RouterFetchDataArgs) {
+           dispatch(fetchShoes(match.params.slug));
+           dispatch(fetchHomepage());
+       },
    },
    {
        path: '/upcoming',
        component: UpcomingPage,
        exact: true,
    },
    {
        path: '*',
        component: NotFoundPage,
        exact: true,
    },
];
</code></pre>
<p>Помимо <code>dispatch</code>, метод <code>fetchData</code> так же принимает объект роута <code>match</code>, так как информация из него тоже необходима, чтобы понимать, что загружать (например, <code>match.params.slug</code> используется для получения slug-a из url). Опишем тип <code>RouterFetchDataArgs</code> - аргументы этого метода:</p>
<div class="code-path">./src/types/index.ts</div>
<pre><code class="language-ts">export type RouterFetchDataArgs = {
    dispatch: Dispatch&lt;ReduxAction&gt;;
    match: match&lt;{ slug: string }&gt;;
};
</code></pre>
<p>И исправим рендеринг роутов в <code>App.tsx</code>:</p>
<div class="code-path">./src/routes.ts</div>
<pre><code class="language-diff">function App() {
    return (
        &lt;div className=&quot;app&quot;&gt;
            &lt;Header /&gt;
            &lt;Switch&gt;
-                &lt;Route path=&quot;/&quot; component={HomePage} exact /&gt;
-                &lt;Route path=&quot;/catalog&quot; component={CatalogPage} exact /&gt;
-                &lt;Route path=&quot;/sneakers/:slug&quot; component={SneakersPage} exact /&gt;
-                &lt;Route path=&quot;/upcoming&quot; component={UpcomingPage} exact /&gt;
-                &lt;Route path=&quot;*&quot; component={NotFoundPage} exact /&gt;
+                {routes.map(({ fetchData, ...routeProps }) =&gt; (
+                    &lt;Route key={routeProps.path} {...routeProps} /&gt;
+                ))}
            &lt;/Switch&gt;
            &lt;Footer /&gt;
        &lt;/div&gt;
    );
}
</code></pre>
<p>Дело осталось за малым - вызывать этот метод соответствующего роута перед рендером приложения. На самом деле это тот момент, который открыл мне глаза, как вообще работает Server-side-rendering и если бы объем статьи нужно было уменьшить до минимума, этот фрагмент кода там точно был бы:</p>
<div class="code-path">./src/server-render-middleware.tsx</div>
<pre><code class="language-ts">const dataRequirements: (Promise&lt;void&gt; | void)[] = [];

routes.some(route =&gt; {
    const { fetchData: fetchMethod } = route;
    const match = matchPath&lt;{ slug: string }&gt;(
        url.parse(location).pathname,
        route
    );

    if (match &amp;&amp; fetchMethod) {
        dataRequirements.push(
            fetchMethod({
                dispatch: store.dispatch,
                match,
            })
        );
    }

    return Boolean(match);
});
</code></pre>
<p>Что происходит? Обходим все роуты из массива и с помощью <code>matchPath</code> из <code>react-router</code> определяем, соответствует ли они текущему адресу страницы. Если роут соответствует текущему пути и метод <code>fetchData</code> присутствует, складываем в массив <code>dataRequirements</code> вызов метода, а в качестве параметров передаем <code>dispatch</code> и <code>match</code>.</p>
<p>И если нужный роут найден, то выходим из цикла (имитируем работу <em>react-router-a</em>). Далее уже все написано - ждем, пока вся асинхронщина из <code>dataRequirements</code> выполнится и запускаем рендеринг приложения.</p>
<p>На сервере не отрабатывает многие методы жизненного цикла компонента, потому что реального монтирование в dom дерево не происходит.
На клиенте мы можем этим воспользоваться и проверять, если у нас список с данными не пуст, значит он уже загружен и нет смысла фетчить еще раз:</p>
<pre><code class="language-ts">// На сервере не вызывается
componentDidMount() {
    const { data, fetchHomepage } = this.props;

    if (!data.popular.length) {
        fetchHomepage();
    }
}
</code></pre>
<p>или в случае с функциональными компонентами:</p>
<pre><code class="language-ts">// На сервере не вызывается
React.useEffect(() =&gt; {
    if (!data.popular.length) {
        fetchHomepage();
    }
}, []);
</code></pre>
<p>Посмотрим, как это работает:</p>
<p><img class="lazyload" alt="Работа приложения в режиме SSR" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/12.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/12.gif"></p>
<p>Если вам кажется, что ничего не происходит, то присмотритесь - я обновляю страницу в левом верхнем углу. После обновления страницы на клиенте уже отрендеренная версия страницы (поэтому кажется, что ничего не меняется), после перехода на другую страницу она начинает загружаться на клиенте. Если посмотреть на процесс загрузки, то после того, как сервер отдал html, то пользователь сразу увидит весь контент:</p>
<p><img class="lazyload" alt="Процесс загрузки приложения в режиме SSR" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/10.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/10.jpg"></p>
<p>Без SSR следующая картина: пользователь получает быстрее html (но пустую), потом инициализируется реакт и пользователь видит заглушки (какой-нибудь лоадер в общем случае), и только после получения данных получает страницу с контентом:</p>
<p><img class="lazyload" alt="Процесс загрузки приложения без SSR" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/8.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/8.jpg"></p>
<p>И давайте еще посмотрим на метрику <em>Performance</em> в Lighthouse:</p>
<h4 id="%D0%B4%D0%BE-ssr">До SSR <a class="direct-link" href="#%D0%B4%D0%BE-ssr" aria-hidden="true">#</a></h4>
<p><img class="lazyload" alt="Метрика Performance без SSR" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/13.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/13.jpg"></p>
<h4 id="c-ssr">C SSR <a class="direct-link" href="#c-ssr" aria-hidden="true">#</a></h4>
<p><img class="lazyload" alt="Метрика Performance с SSR" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/14.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/14.jpg"></p>
<p>Метрика поднялась с 87 баллов до 96 баллов, а это очень хороший результат для приложения (хоть и такого маленького) на реакте.</p>
<p>Мы проделали большую работу, но это еще не все. Есть еще один интересный момент, который нельзя обойти стороной - <em>Code splitting</em>.</p>
<h2 id="%D1%88%D0%B0%D0%B3-5.-code-splitting-%D0%B8-prefetch">Шаг 5. Code splitting и Prefetch <a class="direct-link" href="#%D1%88%D0%B0%D0%B3-5.-code-splitting-%D0%B8-prefetch" aria-hidden="true">#</a></h2>
<p>Ветка в <em>Github</em> по результатам этой части: <a href="https://github.com/noveogroup-amorgunov/react-ssr-tutorial/tree/05-add-code-splitting">https://github.com/noveogroup-amorgunov/react-ssr-tutorial/tree/05-add-code-splitting</a></p>
<p>Даже у такого небольшого приложения продакшен бандл будет иметь внушительный объем - 1.32 МБ не в сжатом виде (155 КБ в Gzipped):</p>
<p><img class="lazyload" alt="Из чего состоит бандл" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/15.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/15.jpg"></p>
<p>Для ее решения нужно разбить бандл на несколько небольших. <em>Code splitting</em> (дословно, <em>разделение кода</em>) как раз про это. Когда пользователь запрашивает страницу, браузер будет загружать только нужные части, а остальные подгружать по необходимости, тем самым можно очень сильно уменьшить первоначальный размер загружаемых ресурсов.</p>
<p>В <em>React 16</em> появился <a href="https://ru.reactjs.org/docs/code-splitting.html#reactlazy">механизм Lazy и Suspense</a>, который позволяет делать ленивые компоненты, но только на клиенте. На сервере не все так просто: необходимо отрендерить все компоненты, даже которые будут ленивыми, и составить список ресурсов, которые нужно предзагрузить клиенту; и это нужно сделать до гидрации приложения (так как если нужные части не будут подгружены, то нечего будет показывать).</p>
<p>Механизм доступен в вебпак из коробки (отдельные бандлы называются чанками), а с помощью плагина <a href="https://babeljs.io/docs/en/babel-plugin-syntax-dynamic-import">plugin-syntax-dynamic-import</a> можно делать асинхронные чанки, которые подгружаются по мере необходимости. Например, для json-моков в текущем проекте были добавлены динамические импорты, и их код автоматически будет вынесен в отдельные бандлы:</p>
<pre><code class="language-diff">- import mock from './mock.json';

// Emulate api request
export const fetchCatalog = () =&gt;
    timeout(500)
-        .then(() =&gt; mock)
+        .then(() =&gt; import('./mock.json'));
</code></pre>
<h3 id="dll-plugin">DLL Plugin <a class="direct-link" href="#dll-plugin" aria-hidden="true">#</a></h3>
<p>На одном из рабочих проектов мы решили не внедрять <em>Code splitting</em>, но использовали <a href="https://webpack.js.org/plugins/dll-plugin/">DLL плагин</a>, который позволяет вынести внешние зависимости (типа, <em>React</em>, список нужно указать самому) в отдельный бандл. Почти бесплатно получаем профит:</p>
<ul>
<li><strong>Ускорение сборки</strong>, которое достигается за счет того, что внешние зависимости можно собрать один раз перед сборкой;</li>
<li><strong>Ускорение загрузки страниц</strong>, так как в процессе разработки собирается только ваш код, а бандл с внешними зависимостями лежит в кеше браузера пользователя (обновляется только при обновлении самих зависимостей).</li>
</ul>
<p>На рабочем проекте получилось два бандла (vendor бандл и с кодом проекта), которые выглядят следующим образом:</p>
<p><img class="lazyload" alt="Бандлы с DLL плагином" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/16.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/16.jpg"></p>
<p>Но в наше приложение с кроссовками я не мог не попробовать внедрить честный <em>Code splitting</em>, что мы и сделаем далее.</p>
<h3 id="loadable-components">Loadable components <a class="direct-link" href="#loadable-components" aria-hidden="true">#</a></h3>
<p>Есть <em>React</em> есть два популярных решения, <em>react-loadable</em> и <a href="https://loadable-components.com/"><em>loadable-components</em></a>, но первое из них - уже пару лет не поддерживается, поэтому его использовать я не советую. Да и в официальной документации реакта советуют использовать именно вторую библиотеку (ее мы и интегрируем). Шагов будет много, но все они довольно просты.</p>
<p>Установим зависимости:</p>
<pre><code class="language-bash">npm i --save @loadable/component @loadable/server
npm i --save-dev @loadable/babel-plugin @loadable/webpack-plugin @types/loadable__component @types/loadable__server @types/loadable__webpack-plugin
</code></pre>
<p>Подключим библиотеку в <code>.babelrc</code> и <code>webpack/client.config.ts</code></p>
<div class="code-path">./babelrc</div>
<pre><code class="language-diff">{
    &quot;presets&quot;: [],
    &quot;plugins&quot;: [
        &quot;react-hot-loader/babel&quot;,
        &quot;@babel/plugin-proposal-class-properties&quot;,
        &quot;@babel/plugin-syntax-dynamic-import&quot;,
+        &quot;@loadable/babel-plugin&quot;
    ]
}
</code></pre>
<div class="code-path">./webpack/client.config.ts</div>
<pre><code class="language-diff">+ import LoadablePlugin from '@loadable/webpack-plugin';

plugins: [
    new MiniCssExtractPlugin({ filename: '[name].css' }),
    !IS_DEV &amp;&amp; new CompressionPlugin(),
+   new LoadablePlugin(),
].filter(Boolean) as Plugin[],
</code></pre>
<p>Обернем все импорты компонент-страниц в <code>loadable</code> (на самом деле можно оборачивать импорты любых компонентов):</p>
<div class="code-path">./src/routes.ts</div>
<pre><code class="language-diff">+ import loadable from '@loadable/component';

- import CatalogPage from 'pages/Catalog/Catalog';
- import UpcomingPage from 'pages/Upcoming/Upcoming';
- import SneakersPage from 'pages/Sneakers/Sneakers';
- import HomePage from 'pages/Home/Home';
- import NotFoundPage from 'pages/404/404';
+ const CatalogPage = loadable(() =&gt; import('./pages/Catalog/Catalog'));
+ const UpcomingPage = loadable(() =&gt; import('./pages/Upcoming/Upcoming'));
+ const SneakersPage = loadable(() =&gt; import('./pages/Sneakers/Sneakers'));
+ const HomePage = loadable(() =&gt; import('./pages/Home/Home'));
+ const NotFoundPage = loadable(() =&gt; import('./pages/404/404'));
</code></pre>
<p>На клиенте перед рендером необходимо подождать, пока загрузятся все чанки и это можно сделать с помощью функции <code>loadableReady</code>:</p>
<div class="code-path">./src/client.tsx</div>
<pre><code class="language-diff">+ import { loadableReady } from '@loadable/component';

+ loadableReady(() =&gt; {
    hydrate(
        &lt;ReduxProvider store={store}&gt;
            &lt;ConnectedRouter history={history}&gt;
                &lt;App /&gt;
            &lt;/ConnectedRouter&gt;
        &lt;/ReduxProvider&gt;,
        document.getElementById('mount')
    );
+ });
</code></pre>
<p>Как же функция понимает, какие именно бандлы необходимо подгружать? На сервере рендерится полное приложение без лайзи модулей, и <code>loadable-components</code> собирает информацию, какие компоненты были отрендерены. Далее с помощью файла <code>loadable-stats.json</code> (который генерируется при сборке), хранящем в себе дерево зависимостей компонентов и чанков, определяется, какие бандлы будут добавлены в html-страницу, отдаваемую клиенту. Опишем это в коде:</p>
<div class="code-path">./src/server-render-middleware.tsx</div>
<pre><code class="language-diff">+ import path from 'path';
+ import { ChunkExtractor } from '@loadable/server';

// ...

function renderApp() {
+    const statsFile = path.resolve('./dist/loadable-stats.json');
+    const chunkExtractor = new ChunkExtractor({ statsFile });

-    const jsx = (
+    const jsx = chunkExtractor.collectChunks(
        &lt;ReduxProvider store={store}&gt;
            &lt;StaticRouter context={context} location={location}&gt;
                &lt;App /&gt;

// ...

res.status(context.statusCode || 200).send(
-    getHtml(reactHtml, reduxState, helmetData)
+    getHtml(reactHtml, reduxState, helmetData, chunkExtractor)
</code></pre>
<p><code>chunkExtractor</code> будет содержать всю информацию о js и ccs-файлах, которые необходимо подключить. Поэтому вместо явного указания файлов, используем информацию из этой переменной:</p>
<div class="code-path">./src/server-render-middleware.tsx</div>
<pre><code class="language-diff">function getHtml(
    reactHtml: string,
    reduxState = {},
    helmetData: HelmetData,
+    chunkExtractor: ChunkExtractor
) {
+    const scriptTags = chunkExtractor.getScriptTags();
+    const linkTags = chunkExtractor.getLinkTags();
+    const styleTags = chunkExtractor.getStyleTags();

// ...

-   &lt;link href=&quot;/main.css&quot; rel=&quot;stylesheet&quot;&gt;
    ${helmetData.title.toString()}
    ${helmetData.meta.toString()}
+   ${linkTags}
+   ${styleTags}

// ...

-    &lt;script src=&quot;/main.js&quot;&gt;&lt;/script&gt;
+    ${scriptTags}
</code></pre>
<p>Посмотрим, что выдаст <code>webpack-bundle-analyzer</code>:</p>
<p><img class="lazyload" alt="Из чего состоит разделенный бандл" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/17.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/17.jpg"></p>
<p>Сейчас у нас приложение разбито на чанки, главный чанк, два чанка с замоканными данными и небольшие бандлы с компоненты для каждой из страниц. Мы получили то, что и хотели изначально. Стоит заметить, что если один код используется на разных страницах, то он будет вынесен в общий дополнительный бандл.</p>
<p>Библиотека предоставляет возможность предзагружать бандлы с помощью метода <code>preload</code>. Например, при наведении мышкой на ссылку загружать нужный компонент. Давайте сделаем это:</p>
<pre><code class="language-diff">+ import loadable from '@loadable/component';

+ const menu = [
+    { to: '/', exact: true, page: PageName.Home },
+    { to: '/catalog', exact: true, page: PageName.Catalog },
+    { to: '/upcoming', exact: true, page: PageName.Upcoming },
+ ];

+ const preloadPage = (pageName: string) =&gt;
+     loadable(() =&gt; import(`../../pages/${pageName}/${pageName}`));

export function Header() {
    // ...
    &lt;NavLink
        activeClassName=&quot;header__nav-item_active&quot;
        to={data.to}
        className={b('nav-item')}
+        onMouseMove={() =&gt; preloadPage(data.page).preload()}
    &gt;
        {data.page}
    &lt;/NavLink&gt;
</code></pre>
<p>Результат:</p>
<p><img class="lazyload" alt="Подгрузка бандлов" src="https://amorgunov.com/assets/images/2020-12-08-server-side-rendering-in-react/18.min.jpg" data-src="/assets/images/2020-12-08-server-side-rendering-in-react/18.gif"></p>
<p>Как вы можете видеть, при наведении на ссылки в шапке сайта автоматически подгружаются бандлы для этих страниц. Можно пойти дальше и интегрировать библиотеку <a href="https://guess-js.github.io/">https://guess-js.github.io/</a>, которая на основе машинного обучения и собранной аналитики google определяет, куда сейчас будет переходить пользователь и сама подгружает нужные бандлы.</p>
<p>Финальный результат можно посмотреть здесь: <a href="https://react-ssr-tutorial.herokuapp.com/">https://react-ssr-tutorial.herokuapp.com/</a>.</p>
<h2 id="%D0%B7%D0%B0%D0%BA%D0%BB%D1%8E%D1%87%D0%B5%D0%BD%D0%B8%D0%B5">Заключение <a class="direct-link" href="#%D0%B7%D0%B0%D0%BA%D0%BB%D1%8E%D1%87%D0%B5%D0%BD%D0%B8%D0%B5" aria-hidden="true">#</a></h2>
<p>Вот и подошла к концу сегодняшняя история, мы собрали полностью работающие приложение на React с интегрированным Server side рендерингом. Мне в свое время потребовалось перечитать огромное количество источников, чтобы разобраться в теме. Поэтому надеюсь, что после прочтения материала у вас не только появилось общее представление, но и понимания конкретных шагов по интеграции.</p>
<p>У приложений c SSR есть как очевидные преимущества (SEO, sharing, лучшие метрики по перфомансу и UX), так и недостатки (обслуживание сервера, усложнение логики приложения и сборки, проблемы на сервере при повышенной нагрузке, не изоморфный код и т.д.), но попробовать его определенно стоит.</p>
<p>На самом деле есть еще много всего интересного, что мы не успели рассмотреть: настройка hot-reload, оптимизация производительности, проблемы при больших нагрузках и потоковый стриминг, css-modules и многое другое осталось за скопом этого поста. Но если вы хотите продолжения, то дайте знать (написав под постом в телеграмме, или просто оставив реакцию чуть ниже).</p>

- 03/08/2020: **«[Как написать свой Virtual DOM](https://amorgunov.com/posts/2020-08-03-create-own-virtual-dom/)»** - <p>Всем привет! Сегодня нас ждет удивительное приключение реализации виртуального DOM-a с нуля.</p>
<p>Материал получился большим, чтение у вас может занять до получаса, имейте это ввиду. А если у вас нет столько времени, то вы можете сразу посмотреть результат в песочнице <a href="https://codesandbox.io/s/vdom130-12fsf">codesandbox</a> или <a href="https://github.com/noveogroup-amorgunov/vdom130">на гитхабе</a>.</p>
<p>Быстрые переходы по частям:</p>
<ul>
<li><a href="#%D1%87%D1%82%D0%BE-%D1%82%D0%B0%D0%BA%D0%BE%D0%B5-virtual-dom%3F">Что такое Virtual DOM?</a></li>
<li><a href="#createvnode(tagname%2C-props%2C-children)">Реализуем метод createVNode</a></li>
<li><a href="#createnode(vnode)">Создаем реальный DOM-узлы</a></li>
<li><a href="#mount(node%2C-target)">Монтируем DOM-узел в DOM-дерево</a></li>
<li><a href="#patchnode(node%2C-vnode%2C-nextvnode)">Сравнение виртуальных нод</a></li>
<li><a href="#patchprops(node%2C-props%2C-nextprops)">Сравнение атрибутов</a></li>
<li><a href="#patchchildren(...)">Сравнение дочерних нод</a></li>
<li><a href="#%D1%81%D0%BE%D0%B1%D0%B8%D1%80%D0%B0%D0%B5%D0%BC-%D0%B2%D1%81%D0%B5-%D0%B2%D0%BC%D0%B5%D1%81%D1%82%D0%B5">Собираем все вместе</a></li>
<li><a href="#%D0%BE%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D1%87%D0%B8%D0%BA%D0%B8-%D1%81%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D0%B9">Обработчики событий</a></li>
<li><a href="#%D0%B8%D0%BD%D1%82%D0%B5%D0%B3%D1%80%D0%B8%D1%80%D1%83%D0%B5%D0%BC-jsx">Бонус: интегрируем JSX</a></li>
</ul>
<h2 id="%D1%87%D1%82%D0%BE-%D1%82%D0%B0%D0%BA%D0%BE%D0%B5-virtual-dom%3F">Что такое Virtual DOM? <a class="direct-link" href="#%D1%87%D1%82%D0%BE-%D1%82%D0%B0%D0%BA%D0%BE%D0%B5-virtual-dom%3F" aria-hidden="true">#</a></h2>
<p>Если речь заходит про виртуальный DOM, то почти всегда дело касается React-а. Но на самом деле это общая концепция, используемая и за пределами мира React-а.</p>
<p><strong>Виртуальный DOM</strong> (VDOM) - это представление пользовательского интерфейса в памяти (например, в JS переменной), по которому можно сформировать &quot;настоящий&quot; DOM. Виртуальный DOM отвечает не только представление в памяти, но и за синхронизацию с реальным DOM.</p>
<p>Например, у нас есть следующий код:</p>
<pre><code class="language-html">&lt;div id=&quot;app&quot;&gt;
 Hello world
&lt;/div
</code></pre>
<p>Виртуальное представление может выглядеть следующим образом:</p>
<pre><code class="language-js">const vNode = {
  tagName: &quot;div&quot;,
  props: {
    id: &quot;app&quot;
  },
  children: [
    &quot;Hello world&quot;
  ],
}
</code></pre>
<p>Такой паттерн позволяет описывать интерфейсы в декларативном подходе, абстрагирует нас от прямой работы с DOM, обработчиками событий и атрибутов, и самое важное: оптимизирует работу обновления только нужных частей DOM-дерева (отвечает за синхронизацию с реальным DOM).</p>
<p>На медиуме есть <a href="https://medium.com/@nickbulljs/how-virtual-dom-work-567128ed77e9">классный пост</a>, как работает VDOM в React-е, а мы реализуем его сами (конечно очень упрощенную, но рабочую версию). Приступим!</p>
<h2 id="createvnode(tagname%2C-props%2C-children)">createVNode(tagName, props, children) <a class="direct-link" href="#createvnode(tagname%2C-props%2C-children)" aria-hidden="true">#</a></h2>
<p>В первом разделе реализуем метод, который будет создавать виртуальный элемент (узел или ноду, все это синонимы,  далее по тексту они будут использоваться в одном контексте). В большинстве реализаций виртуального DOM-а функцию создания виртуальных элементов называют либо <code>createElement</code>, либо сокращенно <code>h</code>. Но мы назовем ее с ключевым словом <strong>vNode</strong>, чтобы явно указать на работу с виртуальным деревом.</p>
<p>Реализация этого метода довольна простая, поэтому приступим:</p>
<div class="code-path">src/vdom.js</div>
<pre><code class="language-js">export const createVNode = (tagName, props = {}, children = []) =&gt; {
  return {
    tagName,
    props,
    children,
  };
};
</code></pre>
<p>Метод принимает три параметра:</p>
<ul>
<li><code>tagName</code> - имя тега (например, <code>div</code> или <code>span</code>);</li>
<li><code>props</code> - свойства узла (например, атрибуты - <code>class</code> или <code>id</code>);</li>
<li><code>children</code> - дочерние элементы.</li>
</ul>
<p>Формат этой функции и названия полей мы задаем сами, как захотим. Свойство <code>props</code> может называться <code>attrs</code>, а <code>tagName</code> - <code>type</code>; Смотрите на виртуальный DOM как на концепцию без четкого интерфейса реализации.</p>
<p>Для свойств и детей используем значения по умолчанию, чтобы при вызове <code>createVNode('h1')</code> быть уверенным, что у созданной виртуальной ноды будут проставлены как свойства, так и дети, и избегать этих проверок в дальнейшем.</p>
<p>Напишем наш первый виртуальный элемент:</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-js">import { createVNode } from './vdom';

const vNode = createVNode('div', {
  class: 'container'
});

console.log(vNode);
</code></pre>
<p>В результате мы получим следующий объект:</p>
<pre><code>Object
    tagName: &quot;div&quot;
    props: Object
        class: &quot;container&quot;
    children: Array[0]
</code></pre>
<p>Придумаем что-нибудь посложнее:</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-js">import { createVNode } from './vdom';

const vNode = createVNode('div', { class: 'container' }, [
  createVNode('h1', {}, ['Hello, Virtual DOM']),
  createVNode('img', { src: 'https://i.ibb.co/M6LdN5m/2.png', width: 200 }),
]);

console.log(vNode);
</code></pre>
<p>И посмотрим на вывод в терминале:</p>
<pre><code>Object
    tagName: &quot;div&quot;
    props: Object
        class: &quot;container&quot;
    children: Array[2]
        0: Object
            tagName: &quot;h1&quot;
            props: Object
            children: Array[1]
                0: &quot;Hello, Virtual DOM&quot;
        1: Object
            tagName: &quot;img&quot;
            props: Object
                src: &quot;https://i.ibb.co/M6LdN5m/2.png&quot;
                width: 200
            children: Array[0]
</code></pre>
<p>Мы только что научились строить виртуальное дерево или виртуальный DOM, поздравляю! Но это только самое начало. Далее мы будет из этого дерева получать реальные DOM элементы.</p>
<h2 id="createnode(vnode)">createNode(vNode) <a class="direct-link" href="#createnode(vnode)" aria-hidden="true">#</a></h2>
<p>Теперь из виртуального дерева необходимо сформировать DOM узел. Рассмотрим реализацию ниже:</p>
<div class="code-path">src/vdom.js</div>
<pre><code class="language-js">export const createDOMNode = vNode =&gt; {
  const { tagName, props, children } = vNode;

  // создаем DOM-узел
  const node = document.createElement(tagName);

  // Добавляем атрибуты к DOM-узлу
  Object.entries(props).forEach(([key, value]) =&gt; {
    node.setAttribute(key, value);
  });

  // Рекурсивно обрабатываем дочерные узлы
  children.forEach(child =&gt; {
    node.appendChild(createDOMNode(child));
  });

  return node;
};
</code></pre>
<blockquote>
<p>Вы можете заметить, что рекурсия не лучшее решение, особенно для обхода DOM-узлов, вложенность которых может быть очень глубокой, и лучше использовать стек (чтобы не словить ошибку при достижении лимитов рекурсии). И вы будете правы, но для простоты реализации оставим именно этот вариант.</p>
</blockquote>
<p>В DOM существуют целых <a href="https://developer.mozilla.org/ru/docs/Web/API/Node/nodeType">семь типов элементов</a> (выше мы обработали самый используемый тип - <em>ElementNode</em>). Обработаем еще один тип - <em>TextNode</em> (простой текст). В нашем виртуальном дереве это будет выглядеть следующим образом:</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-js">const vNode = createVNode('div', { class: 'container' }, [
  createVNode('h1', {}, ['Hello, Virtual DOM']),
  'Text node without tags', // &lt;- TextNode
  createVNode('img', { src: 'https://i.ibb.co/M6LdN5m/2.png', width: 200 }),
]);
</code></pre>
<p>У текстовых нод нет ни аттрибутов, ни детей, поэтому в виртуальном DOM-е их можно хранить в виде обычных строк. Добавим в метод <em>createDOMNode</em> условие на обработку текстовых узлов - они будут создаваться с помощью метода <em>document.createTextNode</em>, если виртуальная нода представлена строкой.</p>
<div class="code-path">src/dom.js</div>
<pre><code class="language-js">export const createDOMNode = vNode =&gt; {
  if (typeof vNode === &quot;string&quot;) {
    return document.createTextNode(vNode);
  }

  const { tagName, props, children } = vNode;
  // ...
}
</code></pre>
<p>Попробуем метод в действии:</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-js">import { createVNode, createDOMNode } from './vdom';

const vNode = createVNode('div', { class: 'container' }, [
  createVNode('h1', {}, ['Hello, Virtual DOM']),
  'Text node without tags',
  createVNode('img', { src: 'https://i.ibb.co/M6LdN5m/2.png', width: 200 }),
]);

const node = createDOMNode(vNode);

console.log(node);
</code></pre>
<p>После выполнения кода в терминале будет отображено полученное DOM-дерево:</p>
<pre><code class="language-html">&lt;div class=&quot;container&quot;&gt;
  &lt;h1&gt;Hello, Virtual DOM&lt;/h1&gt;
  Text node without tags
  &lt;img src=&quot;https://i.ibb.co/M6LdN5m/2.png&quot; width=&quot;200&quot;&gt;&lt;/img&gt;
&lt;/div&gt;
</code></pre>
<p>Очередной шаг завершен. Мы научились строить виртуальное DOM-дерево и формировать из него реальное. Теперь нужно отобразить результат в DOM страницы.</p>
<h2 id="mount(node%2C-target)">mount(node, target) <a class="direct-link" href="#mount(node%2C-target)" aria-hidden="true">#</a></h2>
<p>Нам понадобится index.html страница, на которую и будем монтировать полученное DOM-дерево:</p>
<div class="code-path">src/index.html</div>
<pre><code class="language-html">&lt;!doctype html&gt;
&lt;html lang=&quot;en&quot;&gt;
&lt;head&gt;
  &lt;meta charset=&quot;UTF-8&quot;&gt;
  &lt;title&gt;Virtual DOM&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
  &lt;div id=&quot;app&quot;&gt;&lt;/div&gt;
  &lt;script src=&quot;./app.js&quot;&gt;&lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;
</code></pre>
<p>В элемент с <code>id=app</code> необходимо вставить полученное DOM-дерево. Способов сделать это не мало, например с помощью <code>innerHTML</code> отчистить содержимое родительской ноды и добавить в нее нужный элемент, используя <code>appendChild</code>. Мы же сделаем еще проще и заменим текущую DOM-ноду на переданную:</p>
<div class="code-path">src/dom.js</div>
<pre><code class="language-js">export const mount = (node, target) =&gt; {
  target.replaceWith(node);
  return node;
};
</code></pre>
<blockquote>
<p>У выбранного способа есть проблема - если с сервера к нам приходит уже готовый HTML (сервер сайд рендеринг), то лишний раз полностью перерисовывается DOM. Ближе к концу материала мы исправим это.</p>
</blockquote>
<p>Используем метод <code>mount</code> в нашем приложении и посмотрим на результат в браузере:</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-js">import { mount, createVNode, createDOMNode } from &quot;./vdom&quot;;

const vNode = createVNode(&quot;div&quot;, { class: &quot;container&quot; }, [
  createVNode(&quot;h1&quot;, {}, [&quot;Hello, Virtual DOM&quot;]),
  &quot;Text node without tags&quot;,
  createVNode(&quot;img&quot;, { src: &quot;https://i.ibb.co/M6LdN5m/2.png&quot;, width: 200 })
]);

const app = document.getElementById(&quot;app&quot;);

mount(createDOMNode(vNode), app);
</code></pre>
<p><img class="lazyload" alt="Результат монтирования DOM-дерева на страницу" src="https://amorgunov.com/assets/images/2020-08-03-create-own-virtual-dom/2.min.jpg" data-src="/assets/images/2020-08-03-create-own-virtual-dom/2.jpg"></p>
<h3 id="%D1%81%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D0%B5">Состояние <a class="direct-link" href="#%D1%81%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D0%B5" aria-hidden="true">#</a></h3>
<p>Усложним пример и добавим на страницу состояние. Конечно же, что может быть лучше старого доброго счетчика, который мы и добавим. Будем обновлять каждую секунду счетчик (с помощью <code>setInterval</code>) и перерисовывать DOM-дерево.</p>
<p>Виртуальное дерево перенесем в функцию <code>createVApp</code>, которая будет принимать наше состояние (обычный объект <code>state = { count }</code>) и использовать его в двух местах (просто выводить на странице и записываться в data-атрибут корневого элемента):</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-js">import { mount, createVNode, createDOMNode } from &quot;./vdom&quot;;

const createVApp = state =&gt; {
  const { count } = state;
  return createVNode(&quot;div&quot;, { class: &quot;container&quot;, &quot;data-count&quot;: count }, [ // &lt;- тут
    createVNode(&quot;h1&quot;, {}, [&quot;Hello, Virtual DOM&quot;]),
    createVNode(&quot;div&quot;, {}, [`Count: ${count}`]), // &lt;- и тут
    &quot;Text node without tags&quot;,
    createVNode(&quot;img&quot;, { src: &quot;https://i.ibb.co/M6LdN5m/2.png&quot;, width: 200 })
  ]);
};

// ...
</code></pre>
<p>И добавим код с интервалом и перерендером:</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-js">// ...

const state = { count: 0 };
const app = document.getElementById(&quot;app&quot;);

mount(createDOMNode(createVApp(state)), app);

setInterval(() =&gt; {
  state.count++;
  mount(createDOMNode(createVApp(state)), app);
}, 1000);
</code></pre>
<p>Вернувшись в браузер, мы увидим, что счетчик обновляется, как мы и планировали! На данном этапе мы можем писать приложения в декларативном стиле, что интуитивно понятно и просто.</p>
<p>Но есть и несколько серьезных проблем:</p>
<ul>
<li>Сбрасываются состояния (например, текст или фокус в инпуте) и обработчики событий после каждого ререндера (которых у нас сейчас нет, но это так);</li>
<li>На каждое обновление счетчика постоянно перерисовывается DOM-дерево. А его перерисовка - достаточно тяжелая операция (одна из основных причин, зачем нужен VDOM);</li>
</ul>
<p>Чтобы увидеть реальные перерисовки браузера, можно включить в devtools браузера вкладку Rendering и поставить галку у пункта &quot;Paint flashing&quot; (области, которые браузер перерисовывает, будут подсвечиваться зеленым цветом):</p>
<p><img class="lazyload" alt="Как включить параметр Paint flashing" src="https://amorgunov.com/assets/images/2020-08-03-create-own-virtual-dom/4.min.jpg" data-src="/assets/images/2020-08-03-create-own-virtual-dom/4.gif"></p>
<p>И посмотрим на результат:</p>
<p><img class="lazyload" alt="Paint flashing в действии" src="https://amorgunov.com/assets/images/2020-08-03-create-own-virtual-dom/5.min.jpg" data-src="/assets/images/2020-08-03-create-own-virtual-dom/5.gif"></p>
<p>Браузер перерисовывает все элементы, несмотря на то, что часть из них не меняется.</p>
<p>Для решения проблемы нам нужно не полностью заменять ноду, а обновлять (патчить) ее атрибуты и детей. Сделать это можно как раз с помощью виртуальных элементов - сравнив два элемента, можно найти отличия и точечно сделать изменения в реальном DOM.</p>
<h2 id="patchnode(node%2C-vnode%2C-nextvnode)">patchNode(node, vNode, nextVNode) <a class="direct-link" href="#patchnode(node%2C-vnode%2C-nextvnode)" aria-hidden="true">#</a></h2>
<p>Мы подошли к самому интересному - сравнение виртуальных нод и применение изменений к реальной DOM-ноде. Идея в следующем:</p>
<ul>
<li>Строится виртуальное дерево, по которому в свою очередь строится реальное и монтируется в DOM;</li>
<li>Как только меняется состояние (например, счетчик увеличивается), то строится новое виртуальное дерево и сравнивается с предыдущим;</li>
<li>Все найденные отличия патчатся в реальный DOM-узел;</li>
<li>Текущий виртуальный DOM заменяем новым.</li>
</ul>
<blockquote>
<p>Далее в коде переменные, содержащие новое виртуальное дерево, его дочерние ноды или атрибуты, будут начинаться со слова <code>next</code>, помогая нам понять, что это именно следующее значение, которое нужно сравнивать с текущим.</p>
</blockquote>
<p>Выглядеть это будет следующим образом:</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-js">// ...

const state = { count: 0 };
let vApp = createVApp(state);
let rootNode = mount(createDOMNode(vApp), document.getElementById(&quot;app&quot;));

setInterval(() =&gt; {
  state.count++;

  // Формируем новое виртуальное дерево
  const nextVApp = createVApp(state);

  // Применяем изменения к DOM-ноде
  rootNode = patchNode(rootNode, vApp, nextVApp);

  // Текущее виртуальное дерево заменяем новым
  vApp = nextVApp;
}, 1000);
</code></pre>
<p>Приступим к реализации метода <code>patchNode</code>. Начнем с описания кейсов, которые нужно обработать при сравнении элементов. Первое, что приходит в голову, это изменение атрибутов и дочерних узлов. Все верно, но эти пункты мы рассмотрим далее, а пока опишем возможные случаи с самим элементом:</p>
<ul>
<li>Если <code>nextVNode</code> равен <code>undefined</code>, то реальную ноду нужно удалить;</li>
<li>Если хотя бы одно из значений <code>vNode</code> и <code>nextVNode</code> равно строке (не виртуальный элемент) и они не равны друг другу (например, <code>vNode</code> это строка, а <code>nextVNode</code> - нет, наоборот или два значения - это строки, которые не равны), то можно просто создать новую DOM-ноду с помощью <code>createNode</code> и заменить ей текущую с помощью <code>node.replaceWith</code>;</li>
<li>Если <code>vNode.tagName</code>, не равен <code>nextVNode.tagName</code>, то предположим, что это новый элемент и просто создадим новую DOM-ноду, заменив текущую;</li>
<li>Если у элементов одинаковый тег, то нужно сравнить дочерние узлы и атрибуты.</li>
</ul>
<div class="code-path">src/vdom.js</div>
<pre><code class="language-js">export const patchNode = (node, vNode, nextVNode) =&gt; {
  // Удаляем ноду, если значение nextVNode не задано
  if (nextVNode === undefined) {
    node.remove();
    return;
  }

  if (typeof vNode === &quot;string&quot; || typeof nextVNode === &quot;string&quot;) {
    // Заменяем ноду на новую, если как минимум одно из значений равно строке
    // и эти значения не равны друг другу
    if (vNode !== nextVNode) {
      const nextNode = createDOMNode(nextVNode);
      node.replaceWith(nextNode);
      return nextNode;
    }

    // Если два значения - это строки и они равны,
    // просто возвращаем текущую ноду
    return node;
  }

  // Заменяем ноду на новую, если теги не равны
  if (vNode.tagName !== nextVNode.tagName) {
    const nextNode = createDOMNode(nextVNode);
    node.replaceWith(nextNode);
    return nextNode;
  }

  // Патчим свойства (реализация будет далее)
  patchProps(node, vNode.props, nextVNode.props);

  // Патчим детей (реализация будет далее)
  patchChildren(node, vNode.children, nextVNode.children);

  // Возвращаем обновленный DOM-элемент
  return node;
};
</code></pre>
<h2 id="patchprops(node%2C-props%2C-nextprops)">patchProps(node, props, nextProps) <a class="direct-link" href="#patchprops(node%2C-props%2C-nextprops)" aria-hidden="true">#</a></h2>
<p>Начнем с метода <code>patchProps</code>, который довольно простой в реализации:</p>
<ul>
<li>С помощью деструктизации объектов мержим <code>props</code> и <code>nextProps</code>, получая единый объект с атрибутами элемента;</li>
<li>Далее перебираем ключи это объекта:</li>
<li>Если атрибут имеет одинаковые значения, ничего не делаем;</li>
<li>Если <code>nextProp</code> равен <code>undefined</code>, <code>null</code> или <code>false</code>, то удаляем атрибут у ноды (с помощью <code>removeAttribute</code>);</li>
<li>В остальных случаях вызываем <code>setAttribute</code>, который перезапишет старое значение новым.</li>
</ul>
<blockquote>
<p>Вполне логичным замечанием будет то, почему мы мержим старые и новые свойства и перебираем их, когда старые достаточно было бы удалить. В разделе с обработчиками событий нам это пригодится.</p>
</blockquote>
<p>Обновление самого свойства вынесем в дополнительный метод <code>patchProp</code>:</p>
<div class="code-path">src/vdom.js</div>
<pre><code class="language-js">const patchProp = (node, key, value, nextValue) =&gt; {
  // Если новое значение не задано, то удаляем атрибут
  if (nextValue == null || nextValue === false) {
    node.removeAttribute(key);
    return;
  }

  // Устанавливаем новое значение атрибута
  node.setAttribute(key, nextValue);
};

const patchProps = (node, props, nextProps) =&gt; {
  // Объект с общими свойствами
  const mergedProps = { ...props, ...nextProps };

  Object.keys(mergedProps).forEach(key =&gt; {
    // Если значение не изменилось, то ничего не обновляем
    if (props[key] !== nextProps[key]) {
      patchProp(node, key, props[key], nextProps[key]);
    }
  });
};
</code></pre>
<p>В методе <code>createDOMNode</code> теперь тоже можно использовать метод <code>patchProps</code>, просто передавая в качестве текущих свойств пустой объект:</p>
<pre><code class="language-diff">-  Object.entries(props).forEach(([key, value]) =&gt; {
-     node.setAttribute(key, value);
-   });
+  patchProps(node, {}, props);
</code></pre>
<p>Стоит отметить, что <code>setAttribute</code> преобразует значение в строчку, поэтому если нужно хранить объекты, массивы или методы, то их необходимо сеттить в <code>node</code>, например:</p>
<pre><code class="language-js">const key = 'customArray';
const value = [1, 5];

node[key] = value;
</code></pre>
<p>В нашей реализации мы этого делать не будем.</p>
<h2 id="patchchildren(...)">patchChildren(...) <a class="direct-link" href="#patchchildren(...)" aria-hidden="true">#</a></h2>
<p><code>patchChildren(parentNode, vChildren, nextVChildren)</code></p>
<p>Обновление дочерних нод - процесс без какой либо магии и всего в несколько строк кода. В метод нужно передать родительскую ноду, так как именно ее детей необходимо править (удалять, добавлять и обновлять). Рассмотрим возможные кейсы:</p>
<ul>
<li>Если количество детей каждого виртуального элемента одинаково, то просто сравниваем их в методе <code>patchNode</code>;</li>
<li>Если у текущего виртуального дерева детей больше, то необходимо удалить эти ноды. но нам не придется ничего писать дополнительно, так как метод <code>patchNode</code> умеет удалять DOM-ноды;</li>
<li>Если у следующего виртуального элемента детей больше, то их просто нужно добавить в родительский DOM-элемент с помощью <code>appendChild</code>.</li>
</ul>
<p>Метод выглядит следующим образом:</p>
<div class="code-path">src/vdom.js</div>
<pre><code class="language-js">const patchChildren = (parent, vChildren, nextVChildren) =&gt; {
  parent.childNodes.forEach((childNode, i) =&gt; {
    patchNode(childNode, vChildren[i], nextVChildren[i]);
  });

  nextVChildren.slice(vChildren.length).forEach(vChild =&gt; {
    parent.appendChild(createDOMNode(vChild));
  });
};
</code></pre>
<p>С помощью <code>slice</code> мы получаем массив из дочерних нод, которые необходимо просто вставить в родительскую DOM-ноду.</p>
<blockquote>
<p>Так же можно для виртуальных нод добавить поле key, чтобы использовать его в списках, как это сделано в React, и при добавлении элемента в начало списка не обновлять все следующие ноды. Но этот пункт мы так же пропустим.</p>
</blockquote>
<h2 id="%D1%81%D0%BE%D0%B1%D0%B8%D1%80%D0%B0%D0%B5%D0%BC-%D0%B2%D1%81%D0%B5-%D0%B2%D0%BC%D0%B5%D1%81%D1%82%D0%B5">Собираем все вместе <a class="direct-link" href="#%D1%81%D0%BE%D0%B1%D0%B8%D1%80%D0%B0%D0%B5%D0%BC-%D0%B2%D1%81%D0%B5-%D0%B2%D0%BC%D0%B5%D1%81%D1%82%D0%B5" aria-hidden="true">#</a></h2>
<p>Файл <code>vdom.js</code> содержит 120 строк кода и реализует простейший вариант Virtual DOM-a. Попробуем его в действии: будем на каждое обновление стейта формировать новый виртуальный DOM и патчить ноду.</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-js">import { patchNode, mount, createVNode, createDOMNode } from &quot;./vdom&quot;;

const createVApp = state =&gt; {
  const { count } = state;
  return createVNode(&quot;div&quot;, { class: &quot;container&quot;, &quot;data-count&quot;: count }, [
    createVNode(&quot;h1&quot;, {}, [&quot;Hello, Virtual DOM&quot;]),
    createVNode(&quot;div&quot;, {}, [`Count: ${count}`]),
    &quot;Text node without tags&quot;,
    createVNode(&quot;img&quot;, { src: &quot;https://i.ibb.co/M6LdN5m/2.png&quot;, width: 200 })
  ]);
};

let vApp = createVApp(state);
let app = mount(createDOMNode(vApp), document.getElementById(&quot;app&quot;));

setInterval(() =&gt; {
  state.count++;

  const nextVApp = createVApp(state);

  app = patchNode(app, vApp, nextVApp);
  vApp = nextVApp;
}, 1000);
</code></pre>
<p>Посмотрим на результат в браузере:</p>
<p><img class="lazyload" alt="Результат обновления измененных элементов" src="https://amorgunov.com/assets/images/2020-08-03-create-own-virtual-dom/6.min.jpg" data-src="/assets/images/2020-08-03-create-own-virtual-dom/6.gif"></p>
<p>А так же в DevTools на изменения в DOM-дереве:</p>
<p><img class="lazyload" alt="Результат обновления измененных элементов в DOM" src="https://amorgunov.com/assets/images/2020-08-03-create-own-virtual-dom/7.min.jpg" data-src="/assets/images/2020-08-03-create-own-virtual-dom/7.gif"></p>
<p>Что же, мы решили проблемы и сейчас обновляются только измененные элементы! Но если мы посмотрим на код в <code>app.js</code>, то он не сильно удобный для использования (постоянно хранить ссылку как на DOM-элемент, так и на виртуальный DOM, который еще нужно и заменять). Инкапсулируем всю эту логику в методе <code>patch</code>.</p>
<h2 id="patch(vnode%2C-node)">patch(vNode, node) <a class="direct-link" href="#patch(vnode%2C-node)" aria-hidden="true">#</a></h2>
<p>В метод <code>patch</code> будем передавать виртуальный элемент и реальный, содержимое которого нужно обновить. Текущее виртуальное дерево будем хранить в самой DOM-ноде в свойстве <code>v</code>.</p>
<div class="code-path">src/vdom.js</div>
<pre><code class="language-js">export const patch = (nextVNode, node) =&gt; {
  // Получаем текущее виртуальное дерево из DOM-ноды
  const vNode = node.v;

  // Патчим DOM-ноду
  node = patchNode(node, vNode, nextVNode);

  // Сохраняем виртуальное дерево в DOM-ноду
  node.v = nextVNode;

  return node;
};
</code></pre>
<p>Это будет работать и теперь появилась возможность использовать <code>patch</code> вместо метода <code>mount</code>. Но у нас еще осталась проблема с гидрацией, когда с сервера уже приходит HTML (SSR). Так как изначально <code>node.v</code> равно <code>undefined</code>, то текущий элемент будет просто заменен новым (в методе <code>patchNode</code>).</p>
<p>Однако мы можем решить это, восстановив виртуальное дерево из реального DOM-узла. Полностью восстанавливать мы его не будем (например, атрибуты), но восстановим текстовые элементы (понять, что элемент текстовый, можно по свойству <code>nodeType</code>, которое будет равно 3) и структуру элементов с правильными тегами:</p>
<div class="code-path">src/vdom.js</div>
<pre><code class="language-js">const TEXT_NODE_TYPE = 3;

const recycleNode = node =&gt; {
  // Если текстовая нода - то возвращаем текст
  if (node.nodeType === TEXT_NODE_TYPE) {
    return node.nodeValue;
  }

  //  Получаем имя тега
  const tagName = node.nodeName.toLowerCase();

  // Рекурсивно обрабатываем дочерние ноды
  const children = [].map.call(node.childNodes, recycleNode);

  // Создаем виртуальную ноду
  return createVNode(tagName, {}, children);
};
</code></pre>
<p>Используем этот метод в <code>patch</code>:</p>
<div class="code-path">src/vdom.js</div>
<pre><code class="language-diff">export const patch = (nextVNode, node) =&gt; {
+  const vNode = node.v || recycleNode(node);
  // ...
};
</code></pre>
<p>И перепишем <code>app.js</code> (теперь можно не использовать метод <code>mount</code> и обойтись <code>patch</code>):</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-diff">// ...
let vApp = createVApp(state);
- let app = mount(createDOMNode(vApp), document.getElementById(&quot;app&quot;));
+ let app = patch(vApp, document.getElementById(&quot;app&quot;));

setInterval(() =&gt; {
  state.count++;

-  const nextVApp = createVApp(state);
-
-  app = patchNode(app, vApp, nextVApp);
-  vApp = nextVApp;
+  app = patch(createVApp(state), app);
}, 1000);
</code></pre>
<p>Конечно же вручную запускать <code>patch</code> на каждое обновление стейта не хочется, поэтому усложним несколько наше состояние, добавив метод <code>setState</code> и слушателя <code>onStateChanged</code>:</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-js">const store = {
  state: { count: 0 },
  onStateChanged: () =&gt; {},
  setState(nextState) {
    this.state = nextState;
    this.onStateChanged();
  }
};

let app = patch(createVApp(store), document.getElementById(&quot;app&quot;));

// На каждое изменение состояния патчим DOM-элемент
store.onStateChanged = () =&gt; {
  app = patch(createVApp(store), app);
};
</code></pre>
<p>Создавали VDOM, а по ходу реализовали очень легкую версию flux-стора. Теперь достаточно только обновлять <code>state</code> и DOM автоматически будет обновлен:</p>
<pre><code class="language-js">setInterval(() =&gt; {
    store.setState({ count: store.state.count + 1 });
}, 1000);
</code></pre>
<p>Финальный вариант файла <code>app.js</code> будет выглядеть следующим образом:</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-js">import { patch, createVNode } from &quot;./vdom&quot;;

const createVApp = store =&gt; {
  const { count } = store.state;
  return createVNode(&quot;div&quot;, { class: &quot;container&quot;, &quot;data-count&quot;: count }, [
    createVNode(&quot;h1&quot;, {}, [&quot;Hello, Virtual DOM&quot;]),
    createVNode(&quot;div&quot;, {}, [`Count: ${count}`]),
    &quot;Text node without tags&quot;,
    createVNode(&quot;img&quot;, { src: &quot;https://i.ibb.co/M6LdN5m/2.png&quot;, width: 200 })
  ]);
};

const store = {
  state: { count: 0 },
  onStateChanged: () =&gt; {},
  setState(nextState) {
    this.state = nextState;
    this.onStateChanged();
  }
};

let app = patch(createVApp(store), document.getElementById(&quot;app&quot;));

store.onStateChanged = () =&gt; {
  app = patch(createVApp(store), app);
};

setInterval(() =&gt; {
    store.setState({ count: store.state.count + 1 });
}, 1000);
</code></pre>
<h2 id="%D0%BE%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D1%87%D0%B8%D0%BA%D0%B8-%D1%81%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D0%B9">Обработчики событий <a class="direct-link" href="#%D0%BE%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D1%87%D0%B8%D0%BA%D0%B8-%D1%81%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D0%B9" aria-hidden="true">#</a></h2>
<p>Даже для простой версии виртуального DOM-а обработчики событий обязательны. Сначала добавим кнопки &quot;+1&quot; и &quot;-1&quot; для изменения значения счетчика и удалим <code>setInterval</code>. Для кнопок напишем свою фабрику или компонент (если говорить на языке React-а) с двумя свойствами <code>onclick</code> и <code>text</code>:</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-js">const createVButton = props =&gt; {
  const { text, onclick } = props;

  return createVNode(&quot;button&quot;, { onclick }, [text]);
};
</code></pre>
<p>И добавим две кнопки в <code>createVApp</code>:</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-diff">const createVApp = store =&gt; {
  const { count } = store.state;
  return createVNode(&quot;div&quot;, { class: &quot;container&quot;, &quot;data-count&quot;: count }, [
    createVNode(&quot;h1&quot;, {}, [&quot;Hello, Virtual DOM&quot;]),
    createVNode(&quot;div&quot;, {}, [`Count: ${count}`]),
    &quot;Text node without tags&quot;,
    createVNode(&quot;img&quot;, { src: &quot;https://i.ibb.co/M6LdN5m/2.png&quot;, width: 200 }),
+    createVNode(&quot;div&quot;, {}, [
+      createVButton({
+        text: &quot;-1&quot;,
+        onclick: () =&gt; store.setState({ count: store.state.count - 1 })
+      }),
+      &quot; &quot;,
+      createVButton({
+        text: &quot;+1&quot;,
+        onclick: () =&gt; store.setState({ count: store.state.count + 1 })
+      })
+    ])
  ]);
};
</code></pre>
<p>Запустив этот код, кнопки появятся, но при нажатии на них ничего не будет происходить. Почему так? Если посмотреть в исходный код, то увидим, что код обработчиков (функции) находятся в атрибуте <code>onclick</code> в виде строки. И даже если бы он вызывался, то произошла бы ошибка, так как переменная <code>store</code> не определена.</p>
<p><img class="lazyload" alt="Отображение кнопок в DOM" src="https://amorgunov.com/assets/images/2020-08-03-create-own-virtual-dom/8.min.jpg" data-src="/assets/images/2020-08-03-create-own-virtual-dom/8.jpg"></p>
<p>Чтобы сохранить контекст вызова, нужно сохранить обработчик прямо в DOM-ноду. Сперва создадим функцию <code>listerner</code>:</p>
<div class="code-path">src/vdom.js</div>
<pre><code class="language-js">function listener(event) {
  return this[event.type](event);
}
</code></pre>
<p>Зачем она нам нужна и что делает? Эта функция будет вызываться при вызове события (например <code>click</code>), <code>this</code> указывает на DOM-элемент, <code>this[event.type]</code> на метод, который мы указываем в виртуальном элементе.</p>
<p>Теперь добавим обработку событий в метод <code>patchNode</code>. Чтобы отличать события от не событий можно называть их с префикса <em>on</em> (например, <em>onclick</em>, <em>onkeydown</em>), тем самым отделяя их от остальных атрибутов:</p>
<div class="code-path">src/vdom.js</div>
<pre><code class="language-js">const patchProp = (...) =&gt; {
  if (key.startsWith(&quot;on&quot;)) {
    // ...
  }
</code></pre>
<p>Когда мы поняли, что нужно обработать событие, нужно вытащить его имя (удалив <code>on</code>), добавить функцию в DOM-ноду, и навесить обработчик с помощью <code>addEventListener</code>, если значение в текущем виртуальном узле не задано (или удалить, если значение не задано в следующем виртуальном узле):</p>
<div class="code-path">src/vdom.js</div>
<pre><code class="language-js">const patchProp = (...) =&gt; {
  if (key.startsWith(&quot;on&quot;)) {
    const eventName = key.slice(2);

    node[eventName] = nextValue;

    if (!nextValue) {
      node.removeEventListener(eventName, listener);
    } else if (!value) {
      node.addEventListener(eventName, listener);
    }
    return;
  }
</code></pre>
<p>Проверяем работу событий в действии:</p>
<p><img class="lazyload" alt="Финальный результат" src="https://amorgunov.com/assets/images/2020-08-03-create-own-virtual-dom/9.min.jpg" data-src="/assets/images/2020-08-03-create-own-virtual-dom/9.gif"></p>
<h2 id="%D0%B8%D0%BD%D1%82%D0%B5%D0%B3%D1%80%D0%B8%D1%80%D1%83%D0%B5%D0%BC-jsx">Интегрируем JSX <a class="direct-link" href="#%D0%B8%D0%BD%D1%82%D0%B5%D0%B3%D1%80%D0%B8%D1%80%D1%83%D0%B5%D0%BC-jsx" aria-hidden="true">#</a></h2>
<p>Для компиляции JSX кода в JS можно воспользоваться babel-плагинами <a href="https://babeljs.io/docs/en/next/babel-plugin-transform-react-jsx.html"><code>@babel/plugin-transform-react-jsx</code></a> и <code>@babel/plugin-syntax-jsx</code>. Для этого их нужно установить и создать <code>.babelrc</code> в директории проекта:</p>
<pre><code class="language-json">{
  &quot;plugins&quot;: [
    &quot;@babel/plugin-syntax-jsx&quot;,
    [&quot;@babel/plugin-transform-react-jsx&quot;, { &quot;pragma&quot;: &quot;createVNode&quot; }]
  ]
}
</code></pre>
<p>Также нужно передать опцию <code>pragma</code>, в которой указать название метода (в который JSX-код будет преобразован). Например, следующиий JSX:</p>
<pre><code class="language-jsx">&lt;div&gt;Count&lt;/div&gt;
</code></pre>
<p>будет скомпилирован в следующий JS-код:</p>
<pre><code class="language-js">createVNode(&quot;div&quot;, {}, &quot;Count&quot;)
</code></pre>
<p>Для интеграции JSX нужно немного изменить метод <code>createVNode</code>, а именно добавить возможность передачи функции и обработки дочерних элементов:</p>
<div class="code-path">src/vdom.js</div>
<pre><code class="language-js">export const createVNode = (tagName, props = {}, ...children) =&gt; {
  if (typeof tagName === &quot;function&quot;) {
    return tagName(props, children);
  }

  return {
    tagName,
    props,
    children: children.flat(),
  };
};
</code></pre>
<blockquote>
<p>Нужно для приведения возвращаемого значения к формату <a href="https://github.com/hyperhype/hyperscript">hyperscript</a>.</p>
</blockquote>
<p>Теперь изменим наш пример:</p>
<div class="code-path">src/app.js</div>
<pre><code class="language-js">const createVApp = store =&gt; {
  const { count } = store.state;
  const decrement = () =&gt; store.setState({ count: store.state.count - 1 });
  const increment = () =&gt; store.setState({ count: store.state.count + 1 });

  return (
    &lt;div {...{ class: &quot;container&quot;, &quot;data-count&quot;: String(count) }}&gt;
      &lt;h1&gt;Hello, Virtual DOM&lt;/h1&gt;
      &lt;div&gt;Count: {String(count)}&lt;/div&gt;
      Text node without tags
      &lt;img src=&quot;https://i.ibb.co/M6LdN5m/2.png&quot; width=&quot;200&quot; /&gt;
      &lt;button onclick={decrement}&gt;-1&lt;/button&gt;
      &lt;button onclick={increment}&gt;+1&lt;/button&gt;
    &lt;/div&gt;
  );
};
</code></pre>
<p>Выглядит компактнее, а для любителей реакта и привычнее. Как вы можете заметить, мы используем ключ <code>class</code>, а не <code>className</code> (так как обработка <code>className</code> в атрибут <code>class</code> происходит на стороне Virtual DOM-а, не JSX-а, и у нас такой обработки нет, в отличие от React-а).</p>
<h2 id="%D0%B8%D1%82%D0%BE%D0%B3%D0%B8">Итоги <a class="direct-link" href="#%D0%B8%D1%82%D0%BE%D0%B3%D0%B8" aria-hidden="true">#</a></h2>
<p>Сегодня мы проделали большую работу, реализовав свою версию Virtual DOM с блекджеком, обработчиками событий, методом патчинга реальной ноды и даже интегрировали JSX.</p>
<p>Итоговый вариант залит на <a href="https://github.com/noveogroup-amorgunov/vdom130">гитхаб</a> и в <a href="https://codesandbox.io/s/vdom130-12fsf">codesandbox</a> (версия без jsx).</p>
<p>В процессе написания поста я наткнулся на библиотеку <a href="https://github.com/jorgebucaran/superfine">superfine</a>, которая предоставляет свою реализацию Virtual DOM в 300 строк кода. Если хотите погрузиться в тему поглубже, то рекомендую начать с изучения именно данного решения (в ней есть работа с key в списках, обработка svg, выполнение на стороне сервера и многое другое).</p>
<p>Чтобы не пропускать посты на блоге, подписывайтесь на телеграм канал <a href="https://t.me/amorgunov">https://t.me/amorgunov</a>. До связи!</p>

- 07/05/2020: **«[Использование AWS Lambda с TypeScript](https://amorgunov.com/posts/2020-05-07-using-aws-lambda-with-typescript/)»** - <p>Сегодня мы создадим облачную функцию на TypeScript, которая будет возвращать текущую погоду для переданного города («weather app» на лямбдах), рассмотрим основные моменты работы, покроем код тестами и задеплоим функцию в AWS Lambda.</p>
<p>Данный выпуск - третий по serverless технологиям в блоге, и сегодня мы поговорим о работе с TypeScript. С другими постами по теме вы можете ознакомиться по ссылкам ниже:</p>
<div class="post-series">
    <h4>Серия статей:</h4>
    <ol>
        <li><a href="https://amorgunov.com/posts/2019-03-25-get-started-with-serverless-aws-lambda/">Что такое serverless технологии</a></li>
        <li><a href="https://amorgunov.com/posts/2019-03-26-create-telegram-echo-bot-with-serverless/">Создаем телеграм бота с помощью serverless на nodejs</a></li>
        <li>Использование AWS Lambda с TypeScript <em>(Этот пост)</em></li>
    </ol>
</div>
<p>Есть несколько способов для работы с TS в лямбдах: собирать TypeScript-исходники с помощью ts-node, собирать с помощью webpack или использовать плагин <code>serverless-plugin-typescript</code> при использовании фреймворка <a href="https://serverless.com/">serverless</a>.</p>
<p>Собирать вебпаком код в один бандл стоит, если размер функции со всеми хелперами и вспомогательными библиотеками весит больше 50 МБ (например, из-за больше веса библиотек в node_modules). Но есть нюансы: (1) нужно самому описать webpack конфиг и подготовить код для лямбды (это можно сделать с помощью плагина: <a href="https://github.com/serverless-heaven/serverless-webpack">serverless-webpack</a>) и (2) зависимости с bin-исходниками вебпак может не собрать и после деплоя функции в облаке код может не запускаться.</p>
<p>В данном материале рассмотрим 3-ий вариант - использование плагина, который избавит от ручной сборки и подготовит все за нас.</p>
<p>Если вы хотите посмотреть код, полученный в результате, можете сразу открывать github: <a href="https://github.com/noveogroup-amorgunov/aws-lambda-typescript-weather-app">aws-lambda-typescript-weather-app</a>.</p>
<h2 id="%D0%BF%D0%BE%D0%B4%D0%B3%D0%BE%D1%82%D0%BE%D0%B2%D0%BA%D0%B0-%D0%BF%D1%80%D0%BE%D0%B5%D0%BA%D1%82%D0%B0">Подготовка проекта <a class="direct-link" href="#%D0%BF%D0%BE%D0%B4%D0%B3%D0%BE%D1%82%D0%BE%D0%B2%D0%BA%D0%B0-%D0%BF%D1%80%D0%BE%D0%B5%D0%BA%D1%82%D0%B0" aria-hidden="true">#</a></h2>
<h3 id="%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-%D0%B0%D0%BA%D0%BA%D0%B0%D1%83%D0%BD%D1%82%D0%B0-aws">Настройка аккаунта AWS <a class="direct-link" href="#%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-%D0%B0%D0%BA%D0%BA%D0%B0%D1%83%D0%BD%D1%82%D0%B0-aws" aria-hidden="true">#</a></h3>
<p><strong>Важно</strong>: Для деплоя функции у вас должен быть аккаунт в AWS, добавлены переменные окружения <code>AWS_ACCESS_KEY_ID</code> и <code>AWS_SECRET_ACCESS_KEY</code>:</p>
<ul>
<li>Создайте AWS аккаунт <a href="https://portal.aws.amazon.com/billing/signup#/start">здесь</a></li>
<li>Получите AWS Credentials (<a href="https://serverless.com/framework/docs/providers/aws/guide/credentials/">почитать подробнее</a>, где их получить)</li>
<li>Добавьте credentials в <code>~/.aws/credentials</code>:</li>
</ul>
<pre><code>[default]
aws_access_key_id = &lt;ACCESS_KEY_ID&gt;
aws_secret_access_key = &lt;SECRET_ACCESS_KEY&gt;
</code></pre>
<h3 id="%D0%B7%D0%B0%D0%B2%D0%B8%D1%81%D0%B8%D0%BC%D0%BE%D1%81%D1%82%D0%B8-%D0%B8-%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3%D0%B8">Зависимости и конфиги <a class="direct-link" href="#%D0%B7%D0%B0%D0%B2%D0%B8%D1%81%D0%B8%D0%BC%D0%BE%D1%81%D1%82%D0%B8-%D0%B8-%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3%D0%B8" aria-hidden="true">#</a></h3>
<p>Первым делом установим зависимости и настроим два конфигурационных файла (для TS, и для и Serverless). Создадим директорию для проекта, создадим <code>package.json</code> (npm init -f) и установим зависимости:</p>
<pre><code class="language-bash">mkdir weather-app
cd weather-app
npm init -f
npm i --save-dev @types/node @types/aws-lambda @types/axios @types/jest typescript serverless serverless-offline serverless-plugin-typescript serverless-dotenv-plugin jest ts-jest
npm i --save axios
</code></pre>
<ul>
<li><code>@types/*</code> - TypeScript типы, необходимые для работы кода</li>
<li><code>typescript</code> - TypeScript компилятор</li>
<li><code>serverless</code> - фреймворк для работы с лямбдами и плагины для него (плагин <code>serverless-offline</code> позволит запускать лямбду локально, <code>serverless-plugin-typescript</code> - использовать TypeScript, <code>serverless-dotenv-plugin</code> - читать переменные окружения из <code>.env</code> файла);</li>
<li><code>jest</code>, <code>ts-jest</code> - для тестов</li>
<li><code>axios</code> - библиотека для http-запросов</li>
</ul>
<p><strong>Чтобы сохранить размер загружаемых данных в AWS небольшим, важно добавлять зависимости в <code>devDependencies</code>.</strong></p>
<p>Добавим в секцию <code>scripts</code> скрипты для запуска лямбды локально, для деплоя и для прогона тестов:</p>
<pre><code class="language-json">{
    &quot;local&quot;: &quot;sls offline start&quot;,
    &quot;deploy&quot;: &quot;sls deploy&quot;,
    &quot;test&quot;: &quot;jest&quot;
}
</code></pre>
<p>Данные о погоде будем брать из <a href="https://weatherstack.com/">API Weatherstack</a>, для работы которого нужен <code>API_KEY</code> (для его получения достаточно зарегистрироваться). Ключ нужно положить в <code>.env</code> файл. Содержимое этого файла выглядит следующим образом:</p>
<pre><code>WEATHERSTACK_API_KEY=&lt;API_KEY&gt;
</code></pre>
<p>Так же нужно добавить в <code>package.json</code> опцию jest, в которой указать, что все тесты с расширением <code>.ts</code> или <code>.tsx</code> прогонять через <code>ts-jest</code> (по умолчанию jest не умеет работать с TypeScript):</p>
<pre><code class="language-json">{
  &quot;jest&quot;: {
    &quot;transform&quot;: {
      &quot;.+\\.tsx?$descriptionquot;: &quot;ts-jest&quot;
    }
  }
}
</code></pre>
<p>Далее настроим конфиги.</p>
<h3 id="%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3-serverless">Конфиг Serverless <a class="direct-link" href="#%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3-serverless" aria-hidden="true">#</a></h3>
<p>Все доступные параметры конфигурации описаны <a href="https://www.serverless.com/framework/docs/providers/aws/guide/serverless.yml/">в официальной документации</a>.</p>
<p>Файл <code>serverless.yml</code>:</p>
<pre><code>service: aws-lambda-typescript-weather-app

plugins:
  - serverless-plugin-typescript
  - serverless-dotenv-plugin
  - serverless-offline

provider:
  name: aws
  runtime: nodejs12.x
  stage: dev
  region: us-east-2
  environment:
    WEATHERSTACK_API_KEY: ${env:WEATHERSTACK_API_KEY}

functions:
  getWeather:
    handler: src/getWeather.hander
    events:
      - http:
          path: /weather/{city}/current
          method: get
</code></pre>
<p>Из интересного:</p>
<ul>
<li>Конфиг описывает одну функцию <code>getWeather</code>, которая будет триггериться с помощью HTTP-события (в рамках AWS - это API Gateway);</li>
<li>На момент написания статьи AWS Lambda поддерживает Nodejs с версии 12 (локально можно разрабатываться на 8 и 10, но в облаке код будет выполняться на 12.x версии);</li>
<li>Важно подключать <code>serverless-plugin-typescript</code> перед <code>serverless-offline</code>, чтобы сначала код скомпилировался, а потом уже запускался локально.</li>
</ul>
<h3 id="%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3-typescript">Конфиг TypeScript <a class="direct-link" href="#%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3-typescript" aria-hidden="true">#</a></h3>
<p>Файл <code>tsconfig.json</code>:</p>
<pre><code class="language-json">{
  &quot;compilerOptions&quot;: {
    &quot;strictNullChecks&quot;: true,
    &quot;noImplicitAny&quot;: true,
    &quot;outDir&quot;: &quot;.build&quot;,
    &quot;rootDir&quot;: &quot;./&quot;,
    &quot;module&quot;: &quot;commonjs&quot;,
    &quot;lib&quot;: [&quot;es2019&quot;, &quot;es2020.bigint&quot;, &quot;es2020.string&quot;, &quot;es2020.symbol.wellknown&quot;],
    &quot;target&quot;: &quot;es2019&quot;
  }
}
</code></pre>
<p>По традиции рассмотрим интересное:</p>
<ul>
<li><code>strictNullChecks</code>, <code>noImplicitAny</code> - не обязательные правила, но которые включают более строгие проверки TypeScript и позволяют писать более затипизированный код;</li>
<li>TypeScript позволяет собирать код под разные ECMASCRIPT стандарты (например, можно собрать код под es5 c var-ами). Nodejs v12 полностью поддерживает <code>es2019</code> (подробнее можно почитать здесь: <a href="https://stackoverflow.com/questions/59787574/typescript-tsconfig-settings-for-node-js-12">https://stackoverflow.com/questions/59787574/typescript-tsconfig-settings-for-node-js-12</a>). <code>target</code> сообщает компилятору, какую версию библиотеки включать при компиляции, поэтому в конфиге указан с соответствующим значением.</li>
<li>Опция lib говорит TypeScript, что в платформе реализованы указанные фичи и не нужно их дополнительно преобразовывать. Реальный кейс - когда мы собираем код под ES5 и подключаем полифилл для Promise, можно указать <code>&quot;lib&quot;: [&quot;es5&quot;, &quot;es2015.promise&quot;]</code> и TS не будет дополнительно транспилировать промисы.</li>
</ul>
<blockquote>
<p>Стоит иметь ввиду, что TypeScript не подключает полифилы и о их подключнии нужно позаботиться самому.</p>
</blockquote>
<h2 id="%D0%BF%D0%BE%D0%B4%D0%B3%D0%BE%D1%82%D0%BE%D0%B2%D0%BA%D0%B0-%D1%82%D0%B8%D0%BF%D0%BE%D0%B2">Подготовка типов <a class="direct-link" href="#%D0%BF%D0%BE%D0%B4%D0%B3%D0%BE%D1%82%D0%BE%D0%B2%D0%BA%D0%B0-%D1%82%D0%B8%D0%BF%D0%BE%D0%B2" aria-hidden="true">#</a></h2>
<p>Создадим файл <code>scr/types.ts</code> в котором опишем необходимые типы для работы приложения. В пакете <code>@types/aws-lambda</code> уже есть необходимые типы для работы с лямбда-функциями, поэтому с нуля писать их не нужно. Например, вот так выглядит тип <code>ApiGatewayProxyEventBase</code>:</p>
<p><img class="lazyload" alt="Типы aws-lambda" src="https://amorgunov.com/assets/images/2020-05-07-using-aws-lambda-with-typescript/5.min.png" data-src="/assets/images/2020-05-07-using-aws-lambda-with-typescript/5.jpg"></p>
<p>Для начала опишем типы для входного события лямбда-функции и возвращаемый результат:</p>
<pre><code class="language-ts">import { APIGatewayProxyEvent, APIGatewayProxyResult } from 'aws-lambda';

export type HttpEventRequest&lt;T = null&gt; = Omit&lt;APIGatewayProxyEvent, 'pathParameters'&gt; &amp; {
    pathParameters: T
}

export type HttpResponse = Promise&lt;APIGatewayProxyResult&gt;;
</code></pre>
<p>Если с типом <code>HttpEventResponse</code> все должно быть понятно - так как будем использовать async-функцию, то ожидаем в качестве ответа Promise, который вернет уже готовый тип <code>APIGatewayProxyResult</code>.</p>
<p>А что касается <code>HttpEventRequest</code>, могут возникнуть вопросы. Сейчас рассмотрим проблему и приведенный выше способ решения. В базовом типе <code>APIGatewayProxyEvent</code> свойство <code>pathParameters</code> описано следующим образом:</p>
<pre><code>pathParameters: { [name: string]: string } | null;
</code></pre>
<p>И если в коде попытаться получить из pathParameters параметр пути в url (в нашем случае это <code>weather/{city}/current</code> и параметр <code>city</code>), то TypeScript будет выдавать ошибку:</p>
<p><img class="lazyload" alt="Ошибка типа pathParameters" src="https://amorgunov.com/assets/images/2020-05-07-using-aws-lambda-with-typescript/6.min.png" data-src="/assets/images/2020-05-07-using-aws-lambda-with-typescript/6.jpg"></p>
<p>Это связано с тем, что тип <code>pathParameters</code> может быть null, который нельзя деструктизовать. Для решения проблемы есть два варианта:</p>
<ol>
<li>Использовать &quot;!&quot;, что указать TypeScript, что pathParameters не равен null:</li>
</ol>
<pre><code class="language-ts">const { city } = event.pathParameters!;
</code></pre>
<ol start="2">
<li>С помощью встроенного хелпера <code>Omit</code>, который удаляет из типа переданный ключ, удалить из типа <code>APIGatewayProxyEvent</code> свойство <code>pathParameters</code> и добавить его отдельным типом с использованием дженерика. Такой тип можно использовать вот так:</li>
</ol>
<pre><code class="language-ts">const event: HttpEventRequest = {...};
</code></pre>
<p>Если не ожидается использование <code>pathParameters</code> (параметр будет равен null) или вот так:</p>
<pre><code class="language-ts">const event: HttpEventRequest&lt;{ city: string }&gt; = {...};
</code></pre>
<p>В данном случае ожидается обязательный параметр <code>city</code> (который мы явно передали) в <code>pathParameters</code>. Если попробовать взять другое свойство, то TypeScript ожидаемо подсветит эту строчку:</p>
<p><img class="lazyload" alt="Ошибка использования неожидаемого path параметра" src="https://amorgunov.com/assets/images/2020-05-07-using-aws-lambda-with-typescript/3.min.png" data-src="/assets/images/2020-05-07-using-aws-lambda-with-typescript/3.jpg"></p>
<p>Я выбрал второй способ, который запретит брать из параметров неожидаемые значения.</p>
<p>Опишем еще несколько типов:</p>
<pre><code class="language-ts">// Тип body, возвращаемый пользователю из лямбды
export type HttpResponseBody = {
    city: string;
    temperature: number;
    textWeather: string[];
}

// Тип успешного ответа API Weatherstack
export type WeatherstackSuccessResponse = {
    request: {
        type: string;
        query: string;
        language: string;
        unit: string;
    };
    location: {
        name: string;
        country: string;
        region: string;
        lat: string;
        lon: string;
    };
    current: {
        temperature: number;
        weather_descriptions: string[];
        wind_speed: number;
        pressure: number;
    };
};

// Тип ответа с ошибкой API Weatherstack
export type WeatherstackErrorResponse = {
    success: false;
    error: object;
}

// Тип ответа API Weatherstack
export type WeatherstackResponse = WeatherstackSuccessResponse | WeatherstackErrorResponse;
</code></pre>
<h2 id="%D1%82%D0%B5%D1%81%D1%82%D1%8B">Тесты <a class="direct-link" href="#%D1%82%D0%B5%D1%81%D1%82%D1%8B" aria-hidden="true">#</a></h2>
<blockquote>
<p>Рекомендую почитать материал &quot;<a href="https://amorgunov.com/posts/2020-01-28-how-write-tests-in-nodejs/">Как писать тесты в Nodejs</a>&quot; о правильных практиках и подходах по написанию тестов.</p>
</blockquote>
<p>Давайте пойдем по методологии TDD и опишем два тест-кейса для лямбды, после приступим к реализации. В данном примере достаточно проверить два кейса: когда API Weatherstack возвращает информацию о погоде и когда возвращает ошибку.</p>
<p>Для начала нам нужны стабы (заглушки) ответа от API Weatherstack (успешный и неудачный), а так же объект event, который принимает лямбда-функция (можете взять <a href="https://github.com/noveogroup-amorgunov/aws-lambda-typescript-weather-app/tree/master/src/__test__/mocks">из репозитория</a>).</p>
<p>Перед описанием тест-кейсов создадим перемененную с дефолтным event и хук <code>beforeEach</code>, в котором перед каждым тестом будем отчищать моки, установленные jest-ом.</p>
<pre><code class="language-ts">const defaultEvent = {
    // стаб объекта event, сформированный api gatetway
    ...httpEventMock,
    pathParameters: { city: 'london' },
} as any;

beforeEach(() =&gt; {
    jest.clearAllMocks();
});

describe('getWeather handler', () =&gt; {
    // ...
});
</code></pre>
<p>И опишем два тест-кейса:</p>
<pre><code class="language-ts">it('should respond current weather by city', async () =&gt; {
    const requestSpy = jest
        .spyOn(axios, 'get')
        .mockImplementation(async () =&gt; ({ data: weatherstackSuccessResponse }));

    const actual = await handler(defaultEvent);
    const expected = respondJson({
        city: 'Lakefront Airport',
        temperature: 22,
        textWeather: ['Clear']
    }, 200);

    expect(actual).toEqual(expected);
    expect(requestSpy).toHaveBeenCalled();
})
</code></pre>
<pre><code class="language-ts">it('should respond error if weatherstack API respond error', async () =&gt; {
    const requestSpy = jest
        .spyOn(axios, 'get')
        .mockImplementation(async () =&gt; ({ data: weatherstackErrorResponse }));

    const actual = await handler(defaultEvent);
    const expected = respondJson({ error: true }, 200);

    expect(actual).toEqual(expected);
    expect(requestSpy).toHaveBeenCalled();
});
</code></pre>
<p>С помощью <code>jest.spyOn</code> замокаем http-запрос до API. Далее вызываем функцию, передавая <code>defaultEvent</code> в качестве первого аргумента. А с помощью хелпера <code>respondJson</code> формируем ответ лямбды. Также стоит проверить, что spy-агент был вызван.</p>
<p>Теперь запустим тесты <code>npm test</code>:</p>
<p><img class="lazyload" alt="Тесты упали, можно приступать к реализации" src="https://amorgunov.com/assets/images/2020-05-07-using-aws-lambda-with-typescript/4.min.png" data-src="/assets/images/2020-05-07-using-aws-lambda-with-typescript/4.jpg"></p>
<p>Они ожидаемо упали, можно приступать к реализации.</p>
<h2 id="%D0%BF%D0%B8%D1%88%D0%B5%D0%BC-%D0%BB%D1%8F%D0%BC%D0%B1%D0%B4%D1%83">Пишем лямбду <a class="direct-link" href="#%D0%BF%D0%B8%D1%88%D0%B5%D0%BC-%D0%BB%D1%8F%D0%BC%D0%B1%D0%B4%D1%83" aria-hidden="true">#</a></h2>
<p>Напишем код хелпера для формирования ответа и лямбду:</p>
<pre><code class="language-ts">export function respondJson(body: object, statusCode: number) {
    return {
        statusCode,
        body: JSON.stringify(body)
    };
}

export async function handler(event: HttpEventRequest&lt;{ city: string }&gt;): HttpResponse {
    const { city } = event.pathParameters;

    return respondJson({ city }, 200);
}
</code></pre>
<p>Эту функцию можно запустить локально. После запуска будет создана директория <code>.build</code>, в которой можно посмотреть скомпилированный в JavaScript код:</p>
<p><img class="lazyload" alt="Скомпилированный TypeScript код в JS" src="https://amorgunov.com/assets/images/2020-05-07-using-aws-lambda-with-typescript/2.min.png" data-src="/assets/images/2020-05-07-using-aws-lambda-with-typescript/2.jpg"></p>
<p>Допишем отправку запроса в API, обработку ответа от API и формирования ответа лямбды.</p>
<pre><code class="language-ts">const API_KEY = process.env.WEATHERSTACK_API_KEY;

export async function handler(event: HttpEventRequest&lt;{ city: string }&gt;): HttpResponse {
    const { city } = event.pathParameters;

    // Делаем запрос в API Weatherstack
    const endpoint = 'http://api.weatherstack.com/current';
    const { data } = await axios.get&lt;WeatherstackResponse&gt;(endpoint, {
        params: { access_key: API_KEY, query: city }
    });

    // Если есть ошибка, возвращаем это пользователю
    // Оператор in помогает TypeScript работать с union-типами
    if ('error' in data) {
        return respondJson({ error: true }, 200);
    }

    // Формируем ответ
    const response: HttpResponseBody = {
        city: data.location.name,
        temperature: data.current.temperature,
        textWeather: data.current.weather_descriptions,
    }

    return respondJson(response, 200);
}
</code></pre>
<p>Теперь можно еще раз запустить функцию локально и сделать запросы в браузерной строке для возврата погоды:</p>
<pre><code>http://localhost:3000/dev/weather/{city}/current
</code></pre>
<p>И запустить тесты, чтобы убедиться в их успешном прохождении:</p>
<p><img class="lazyload" alt="Успешно пройденные тесты" src="https://amorgunov.com/assets/images/2020-05-07-using-aws-lambda-with-typescript/7.min.png" data-src="/assets/images/2020-05-07-using-aws-lambda-with-typescript/7.jpg"></p>
<h2 id="%D0%B4%D0%B5%D0%BF%D0%BB%D0%BE%D0%B9-%D0%B2-aws">Деплой в AWS <a class="direct-link" href="#%D0%B4%D0%B5%D0%BF%D0%BB%D0%BE%D0%B9-%D0%B2-aws" aria-hidden="true">#</a></h2>
<p>Командой <code>npm run deploy</code> можно задеплоить функцию в AWS. В терминале вы будете видеть весь процесс деплоя лямбды (все файлы складываются в zip архив и заливаются в S3). В итоге вы получите постоянный эндпоинт, что-то типа: <a href="https://xxx.execute-api.us-east-2.amazonaws.com/dev/">https://xxx.execute-api.us-east-2.amazonaws.com/dev/</a>.</p>
<p>Делая запрос на <code>GET https://xxx.execute-api.us-east-2.amazonaws.com/dev/weather/{city}/current</code> вы получите информацию о погоде:</p>
<p><img class="lazyload" alt="Ответ лямбды с информацией о погоде" src="https://amorgunov.com/assets/images/2020-05-07-using-aws-lambda-with-typescript/8.min.png" data-src="/assets/images/2020-05-07-using-aws-lambda-with-typescript/8.jpg"></p>
<h2 id="%D0%B8%D1%82%D0%BE%D0%B3%D0%BE">Итого <a class="direct-link" href="#%D0%B8%D1%82%D0%BE%D0%B3%D0%BE" aria-hidden="true">#</a></h2>
<p>Писать лямбды на TypeScript довольно просто, достаточно добавить конфигурационный файл <code>tsconfig.json</code> и использовать плагин <code>serverless-plugin-typescript</code>.</p>
<p>Написанная нами лямбда не готова для продакшена: нужно предусмотреть валидацию входных данных, обработку ошибок и другие вещи, присущие всем API Endpoint-ам, но старт работы с TypeScript положен, проект подготовлен. Дальше - только ваши бизнес требования и фантазия.</p>

- 12/04/2020: **«[Стоит ли использовать Redux с React Hooks](https://amorgunov.com/posts/2020-04-12-use-redux-with-react-hooks/)»** - <blockquote>
<p>Уже много материалов написано по этой теме, во всяком случае в англоязычном сообществе, но мало где рассматривается честное сравнение между использованием connect и хуками, чем сегодня мы и займемся.</p>
</blockquote>
<p>Хайп уже почти закончился: одни прониклись философией хуков и используют их везде, другие еще не дошли до их использования, а есть и те, кто попробовал и решил, что эта концепция не для него. Но глупо спорить, React Hooks все больше и больше внедряются в экосистему реакта. Если у вас есть библиотека на реакте и в ней нет хуков, то что-то здесь не так. Документация многих пакетов переписывается на примеры с использованием хуков как основной способ использования (<em>formik</em>, <em>react-dnd</em>). На мой взгляд, это дело времени, пока все не начнут использовать хуки, пусть даже неявным образом.</p>
<p><a href="https://react-redux.js.org/">React-redux</a>, начиная с версии 7.1 добавили долгожданную поддержку хуков. На самом деле это произошло давно, но в своем проекте я решил на днях посмотреть, насколько будет удобно их использовать. Внедрение хуков означало, что теперь можно избавиться от <code>connect</code> (компонента высшего порядка) и использовать Redux внутри функциональных компонентов.</p>
<p>В посте рассмотрим, как начать использовать React Hooks с редаксом, какие могут возникнуть проблемы и постараюсь ответить на главный вопрос: &quot;Стоит ли в своих проектах избавляться от <code>connect</code> в пользу хуков&quot;?</p>
<h2 id="%D1%87%D1%82%D0%BE-%D1%82%D0%B0%D0%BA%D0%BE%D0%B5-react-hooks">Что такое React Hooks <a class="direct-link" href="#%D1%87%D1%82%D0%BE-%D1%82%D0%B0%D0%BA%D0%BE%D0%B5-react-hooks" aria-hidden="true">#</a></h2>
<p>В реакте 16.8 появились хуки. Они позволили использовать такие вещи, как состояние, возможности методов жизненного цикла в функциональных компонентах, которые ранее были доступны только в компонентах на классах.</p>
<p>Например, у нас есть компонент со состоянием, написанный на классе:</p>
<pre><code class="language-js">class AwesomeComponent extends React.Component {
    state = {
        counter: 0,
    };

    onClick = () =&gt; {
        this.setState({ count: this.state.count + 1 });
    };

    render() {
        return (
            &lt;div&gt;
                &lt;p&gt;Count: {this.state.count}&lt;/p&gt;
                &lt;button onClick={this.onClick}&gt;Add +1&lt;/button&gt;
            &lt;/div&gt;
        );
    }
}
</code></pre>
<p>Сейчас этот компонент может быть переписан на хуки, например, так:</p>
<pre><code class="language-js">import { useState } from 'react';

function AwesomeComponent() {
    const [count, setCount] = useState(0);

    return (
        &lt;div&gt;
            &lt;p&gt;Count: {count}&lt;/p&gt;
            &lt;button onClick={() =&gt; setCount(count + 1)}&gt;Add +1&lt;/button&gt;
        &lt;/div&gt;
    );
}
</code></pre>
<p>Думаю вы согласитесь, что код с хуками выглядит более лаконично. И он позволяет в функциональные компоненты добавлять фичи, ранее недоступные без переписывания компонента на класс. По хукам есть <a href="https://ru.reactjs.org/docs/hooks-intro.html">отличная документация</a> на официальном сайте (на английском и русском языках), поэтому если хотите разобраться в них, рекомендую к прочтению.</p>
<h2 id="%D0%BA%D0%B0%D0%BA-%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D1%8C-redux-%D1%81-%D1%85%D1%83%D0%BA%D0%B0%D0%BC%D0%B8">Как использовать Redux с хуками <a class="direct-link" href="#%D0%BA%D0%B0%D0%BA-%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D1%8C-redux-%D1%81-%D1%85%D1%83%D0%BA%D0%B0%D0%BC%D0%B8" aria-hidden="true">#</a></h2>
<p>На самом деле очень просто! В библиотеки react-redux есть уже готовые <code>useSelector</code> и <code>useDispatch</code>, которые можно использовать вместо коннект.</p>
<p><a href="https://react-redux.js.org/api/hooks#useselector"><code>useSelector</code></a> - это аналог <code>mapStateToProps</code>. Хук принимает на вход селектор - метод, который принимает <em>redux state</em> и возвращает из него необходимые данные.</p>
<p><a href="https://react-redux.js.org/api/hooks#usedispatch"><code>useDispatch</code></a> - замена для <code>mapDispatchToProps</code>, только в довольно упрощенном виде. Хук возвращает <em>dispatch</em> метод из редакса, с помощью которого можно диспатчить экшены. С одной стороны это избавляет нас от <em>action creators</em>, с другой - ломает уже принятую парадигму не использовать <em>dispatch</em> напрямую.</p>
<p>У меня сразу возник вопрос, зачем мне нужен <em>dispatch</em>, если у меня есть заготовленные <em>action creators</em>. В документации я увидел следующее:</p>
<p><img class="lazyload" alt="Хук useActions" src="https://amorgunov.com/assets/images/2020-04-12-use-redux-with-react-hooks/1.min.jpg" data-src="/assets/images/2020-04-12-use-redux-with-react-hooks/1.jpg"></p>
<p>Как оказалось, изначально хук <code>useActions</code> был добавлен в альфу, но потом его выпилили из-за комментария Дена Абрамова (<a href="https://github.com/reduxjs/react-redux/issues/1252#issuecomment-488160930">раз</a>, <a href="https://github.com/facebook/create-react-app/issues/6880#issuecomment-488158024">два</a>). Ден сказал о том, что паттерн <em>action creators as a props</em> добавляет лишние абстракции и сложность в мире хуков и привел хороший пример:</p>
<blockquote>
<p>You don't <code>useFunction(sum, 2, 2)</code> to obtain a <code>boundSum</code> and then call <code>boundSum</code>. You just call <code>sum(2, 2)</code>. This is the same.</p>
</blockquote>
<p>Что ж, раз автор редакса сказал, не использовать <em>bindActionCreator</em>, не будем. Но в ознакомительных целях хук <em>useActions</em> может выглядеть следующим образом:</p>
<pre><code class="language-js">import { useMemo } from 'react';
import { useDispatch } from 'react-redux';
import { bindActionCreators } from 'redux';

export function useActions(actions, dependencies = []) {
    const dispatch = useDispatch();

    return useMemo(
        () =&gt; actions.map(a =&gt; bindActionCreators(a, dispatch)),
        [dispatch, ...dependencies]
    );
}
</code></pre>
<p>В любом случае давайте перепишем компонент с <code>connect</code> на хуки. Первоначально компонент может выглядеть так:</p>
<pre><code class="language-js">import React from 'react';
import { connect } from 'react-redux';
import { incrementCount } from './store/counter/actions';

export function AwesomeReduxComponent(props) {
    const { count, incrementCount } = props;

    return (
        &lt;div&gt;
            &lt;p&gt;Count: {count}&lt;/p&gt;
            &lt;button onClick={incrementCount}&gt;Add +1&lt;/button&gt;
        &lt;/div&gt;
    );
}

const mapStateToProps = state =&gt; ({ count: state.counter.count });
const mapDispatchToProps = { incrementCount };

export default connect(mapStateToProps, mapDispatchToProps)(AwesomeReduxComponent);
</code></pre>
<p>Теперь, с хуками это может выглядеть вот так:</p>
<pre><code class="language-js">import React from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { incrementCount } from './store/counter/actions';

export const AwesomeReduxComponent = () =&gt; {
    const count = useSelector(state =&gt; state.counter.count);
    const dispatch = useDispatch();

    return (
        &lt;div&gt;
            &lt;p&gt;Count: {count}&lt;/p&gt;
            &lt;button onClick={() =&gt; dispatch(incrementCount())}&gt;Add +1&lt;/button&gt;
        &lt;/div&gt;
    );
};
</code></pre>
<p>Выглядит более просто, чем с использованием функции <code>connect</code>, <em>props</em> компонента не смешиваются со свойствами из редакса. Также большое преимущество, что теперь не нужно оборачивать свои компоненты в HOC-и, тем самым избавляясь от <em>connect hell</em>:</p>
<p><img class="lazyload" alt="connect hell" src="https://amorgunov.com/assets/images/2020-04-12-use-redux-with-react-hooks/4.min.jpg" data-src="/assets/images/2020-04-12-use-redux-with-react-hooks/4.jpg"></p>
<p>Каждый компонент, который использует <em>redux</em>, оказывается обернут в <code>Connect(ComponentName)</code>, тем самым увеличивая глубину дерева с компонентами.</p>
<p>В <a href="https://react-redux.js.org/api/hooks">документации</a> хорошо описаны доступные хуки, рекомендую почитать.</p>
<h2 id="redux-hooks-%D0%BF%D1%80%D0%BE%D1%82%D0%B8%D0%B2-connect">Redux hooks против connect <a class="direct-link" href="#redux-hooks-%D0%BF%D1%80%D0%BE%D1%82%D0%B8%D0%B2-connect" aria-hidden="true">#</a></h2>
<p>Преимущества хуков в рамках редакса мы уже рассмотрели, теперь поговорим о недостатках. На самом деле некоторые <em>gotchas</em> описаны в документации в разделе <a href="https://react-redux.js.org/api/hooks#usage-warnings">usage warnings</a>:</p>
<ul>
<li><code>useSelector</code> использует по умолчанию строгое равенство для сравнения объектов, которые возвращает селектор (из-за этого в случае возврата нового объекта компонент постоянно будет перерисовываться) и нужно использовать свой метод для сравнения. Или можно написать свой хук:</li>
</ul>
<pre><code class="language-js">import { useSelector, shallowEqual } from 'react-redux';

export function useShallowEqualSelector(selector) {
   return useSelector(selector, shallowEqual);
}
</code></pre>
<p>И использовать его:</p>
<pre><code class="language-js">export const AwesomeReduxComponent = () =&gt; {
    // Хук необходим, если селектор возвращает новый объект
    const { count } = useShallowEqualSelector(state =&gt; {
        count: state.counter.count;
    });
    const dispatch = useDispatch();

    return &lt;div /&gt;;
};
</code></pre>
<ul>
<li>К тому же, в отличие от <code>connect</code>, хук <code>useSelector</code> не предотвращает повторный ререндер компонента, когда перерисовывается родитель, даже если пропы не изменились. Поэтому для оптимизации стоит использовать <em>React.memo()</em>:</li>
</ul>
<pre><code class="language-js">export const AwesomeReduxComponent = React.memo(() =&gt; {
    // Хук необходим, если селектор возвращает новый объект
    const { count } = useShallowEqualSelector(state =&gt; {
        count: state.counter.count;
    });
    const dispatch = useDispatch();

    return &lt;div /&gt;;
});
</code></pre>
<ul>
<li>При передаче <em>callback-a</em> с <em>dispatch</em> дочерним компонентам следует оборачивать метод в <em>useCallback</em>, что бы дочерние компоненты не рендерились без необходимости:</li>
</ul>
<pre><code class="language-js">export const AwesomeReduxComponent = React.memo(() =&gt; {
    // Хук необходим, если селектор возвращает новый объект
    const { count } = useShallowEqualSelector(state =&gt; {
        count: state.counter.count;
    });
    const dispatch = useDispatch();
    const onClick = useCallback(
        () =&gt; dispatch(incrementCount()),
        [dispatch]
    );

    return &lt;div /&gt;;
});
</code></pre>
<p>Уже не выглядит так лаконично, не правда ли? На тестовом проекте у меня получилось следующее:</p>
<pre><code class="language-js">function SneakersPage() {
    const { popular } = useSelector(getHomepage);
    const isLoading = useSelector(isLoadingSelector);
    const data = useSelector(getShoes);
    const dispatch = useDispatch();
    const fetchShoes = React.useCallback(
        slug =&gt; dispatch(fetchShoesActionCreator(slug)),
        [dispatch]
    );

    // ...
}
</code></pre>
<p>К компоненту добавилось 8 дополнительных строчек кода. Как это будет выглядеть в больших проектах, 20-30 дополнительных строчек кода? Думаю в этот момент захочется переписать все обратно. Но есть решение: в таком случае можно использовать композицию хуков - все хуки выносить в отдельный хук для компонента. Выглядеть это будет как-то так:</p>
<pre><code class="language-js">function SneakersPage() {
    const {
        popular,
        isLoading,
        data,
        dispatch,
        fetchShoes
    } = useSneakersPage();

    // ...
}
</code></pre>
<p>К этим кейсам можно привыкнуть, найти ответы на *stackoverflow и использовать хуки для редакса. Но все они убивают простоту и понятность кода.</p>
<p>Есть еще несколько моментов:</p>
<ul>
<li>
<p><strong>Усложнение тестирования</strong>. Для тестирования компонента придется всегда создавать стор и оборачивать компонент в <em>ReduxProvider</em>, т.е. придется писать интеграционные тесты. В случае с <em>connect</em>, мы можем экспортировать компонент и тестировать его независимо.</p>
</li>
<li>
<p><strong>Нарушение принципа единой ответственности</strong>. Компонент становится ответственным за слишком многое, тем самым становится более сложным. Дядюшка Боб будет недоволен.</p>
</li>
<li>
<p><strong>Дебаг</strong>. В своем тестовом приложении я могу изменять значения пропсов компонента в <em>dev tools</em> (которые приходят из <em>connect-a</em>) и смотреть, как компонент будет выглядеть в таком случае. Например, ниже, я меняю проп <code>isLoading</code> и элементы с кроссовками меняются на заглушки:</p>
</li>
</ul>
<p><img class="lazyload" alt="Изменение свойств у компонента" src="https://amorgunov.com/assets/images/2020-04-12-use-redux-with-react-hooks/2.min.jpg" data-src="/assets/images/2020-04-12-use-redux-with-react-hooks/2.gif"></p>
<p>Как дела с хуками? С хуками у нас следующая картина:</p>
<p><img class="lazyload" alt="Неинформативная информация о хуках" src="https://amorgunov.com/assets/images/2020-04-12-use-redux-with-react-hooks/3.min.jpg" data-src="/assets/images/2020-04-12-use-redux-with-react-hooks/3.jpg"></p>
<p>К сожалению, нет возможности просматривать текущие значения и изменять их как пропсы. Есть только понимание, что используются три <em>useSelector</em> и <em>useDispatch</em>. Как воркэраунд, можно вынести хуки в компонент высшего порядка, но в таком случае смысл использования хуков пропадает.</p>
<h2 id="%D0%B8%D1%82%D0%BE%D0%B3%D0%BE">Итого <a class="direct-link" href="#%D0%B8%D1%82%D0%BE%D0%B3%D0%BE" aria-hidden="true">#</a></h2>
<p><em>React Hooks</em> - это крутая фича, которая добавила в реакт возможности по использованию функциональных компонентов, которые ранее были невозможными, делая код проще и лаконичнее. Сообщество движется к использованию функциональных компонентов и хуков, когда это возможно.</p>
<p>Но что касается Redux, то я придерживаюсь мнения, что с хуками код выглядит сложнее. Нарушается принцип единой ответственности, сложнее тестировать и дебажить компоненты. Если вынести хуки в отдельный компонент, то получится тот же самый connect, но без дополнительных полезных обработчиков для оптимизации перерендеров.</p>
<p>Это лишь мое мнение, и попробовать хуки в этом кейсе определенно стоит. Возможно, через время я переосмыслю это и напишу новый пост, что я был не прав, но не сейчас.</p>
<h3 id="%D1%81%D1%81%D1%8B%D0%BB%D0%BA%D0%B8-%D0%BF%D0%BE-%D1%82%D0%B5%D0%BC%D0%B5">Ссылки по теме <a class="direct-link" href="#%D1%81%D1%81%D1%8B%D0%BB%D0%BA%D0%B8-%D0%BF%D0%BE-%D1%82%D0%B5%D0%BC%D0%B5" aria-hidden="true">#</a></h3>
<ul>
<li><a href="https://ru.reactjs.org/docs/hooks-intro.html">Официальная документация по хукам</a></li>
<li><a href="https://blog.logrocket.com/use-hooks-and-context-not-react-and-redux/">Как context и хуки могут заменить redux</a></li>
<li><a href="https://medium.com/better-programming/unit-testing-react-redux-hooks-ce7d69e1e834">Как тестировать хуки с редаксом</a></li>
</ul>

<!-- BLOG-POST-LIST:END -->
