60 кадров в секунду или динамический пререндеринг CSS-анимаций
------------------------------------------------------------

Создание всё более красивых анимаций для сайтов — это то, чем я занимался
последние пару лет. И только что я открыл новый способ предоставляющий больше
гибкости и безукоризненой точности, чем все способы до этого. Я назвал этот
способ «CSS-пререндеринг» и он проще, чем вы могли подумать.


## CSS-анимации vs. JavaScript-анимации ##

Вначале было слово. И этим словом был JavaScript. Многие годы JavaScript был
единственной возможностью добавить анимацию на страницу. По мере взросления
интернета, CSS наполнялся возможностями и затем были добавлены анимации. Эти
встроенные в браузеры возможности отвественны за появление гораздо более плавных
и приятных глазу анимаций чем их JavaScript-реализации из-за того, что браузер
может оптимизировать нативную(!!!) анимацию. В то время, как CSS-анимации ужасно
не гибкие, что заставляет разработчиков выбирать между плавной, но не гибкой
CSS-анимацией и JS-анимацией, которую можно адаптировать к нуждам приложения на
лету.

CSS-анимации (`@keyframes`) не гибкие, потому что должны быть заранее определены
в коде. CSS-Transitions (!!!) немногим более легки в использовании, так как
могут легко реагировать на любые изменения CSS-свойств, инициированными через
JavaScript. Однако, сложность в том, что CSS-transitions не могут дать вам
свободы - анимации в несколько шагов очень трудно достичь. Эта именно та
проблема, которую и должны решать `@keyframe`, но они не могут обеспечить
сравнимый с CSS-transitions уровень динамической реакции на изменения.

CSS-анимации также ограничены набором доступных кривых Безье — их всего шесть. В
реальной жизни этого не хватает. Джо Лэмберт (Joe Lambert) спешит на помощь со
своим изобретением [Morf][1] — инструментом для генерации `@-webkit-keyframe`
анимаций с большим количеством коротких шагов. Это позволяет реализовать какую
угодно кривую Безье. Я подумал и решил развить эту идею с помощью [фичи
`toCSS`][3] библиотеки [Rekapi][2], c использование [keyframe API][4] для
создания кроссбраузерного CSS-кода. Затем я использовал эту возможность для
создания библиотеки [Stylie][5], которая обеспечивает пользовательский интерфейс
для создания анимации, а экспорта ее в CSS-код.

В том время, когда я чувствовал, что надёжный интерфейс для создания и
редактирования CSS-анимаций безусловно важен, но способ конфигурирования
анимаций и вставки сотен строк CSS-кода в файл некрасив и сильно негибок. Я
хотел надеяться, что существует лучший путь. Так я пришел CSS-пререндерингу.

## CSS-пререндеринг ##

Главная идея CSS-пререндеринга проста и прозрачна — вы налету создаёте строчку с
`@keyframe`, вставляете в только что созданый `style`-элемент и вставляете этот
элемент в DOM-дерево. По окончании анимации вы просто удаляете потерявший
необходимость `style`-элемент. Такой способ работает удивительно хорошо, в
частности в `WebKit`-браузерах (другие браузерные движки ещё нужно
потестировать).

До этого момента, я не мог найти ни одного инструмента способного на это,
поэтому я решил решить восполнить пробел с помощью [модуля `CSSRenderer`][8]
библиотеки Rekapi. Цель Rekapi в том, чтобы обеспечить вас гибким и удобным API
для создание keyframe-анимаций (!!!); исторически сложилось, что она позволяет
создавать JavaScript-анимации. Используя `CSSRenderer`, вы можете создавать те
же анимации, только уже на базе CSS, гораздо более плавных и не блокирующих
выполнение JavaScript-потока. Но не принимайте мои слова на веру, [разницу можно
заметить невооруженным глазом][7]. Разница в качестве сильнее заметна на
мобильных устройствах, особенно на iOS.


## Создание CSS-анимаций с помощью Rekapi ##

Для создания кроссбраузерной `@keyframe`-анимации вам необходим код похожий на
этот (сгенерировано c помощью Stylie)

  .stylie {
    -moz-animation-name: stylie-transform-keyframes;
    -moz-animation-duration: 2000ms;
    -moz-animation-delay: 0ms;
    -moz-animation-fill-mode: forwards;
    -moz-animation-iteration-count: infinite;
    -ms-animation-name: stylie-transform-keyframes;
    -ms-animation-duration: 2000ms;
    -ms-animation-delay: 0ms;
    -ms-animation-fill-mode: forwards;
    -ms-animation-iteration-count: infinite;
    -o-animation-name: stylie-transform-keyframes;
    -o-animation-duration: 2000ms;
    -o-animation-delay: 0ms;
    -o-animation-fill-mode: forwards;
    -o-animation-iteration-count: infinite;
    -webkit-animation-name: stylie-transform-keyframes;
    -webkit-animation-duration: 2000ms;
    -webkit-animation-delay: 0ms;
    -webkit-animation-fill-mode: forwards;
    -webkit-animation-iteration-count: infinite;
    animation-name: stylie-transform-keyframes;
    animation-duration: 2000ms;
    animation-delay: 0ms;
    animation-fill-mode: forwards;
    animation-iteration-count: infinite;
  }
  @-moz-keyframes stylie-transform-keyframes {
    0% {-moz-transform:translateX(0px) translateY(0px);}
    100% {-moz-transform:translateX(400px) translateY(0px);}
  }
  @-ms-keyframes stylie-transform-keyframes {
    0% {-ms-transform:translateX(0px) translateY(0px);}
    100% {-ms-transform:translateX(400px) translateY(0px);}
  }
  @-o-keyframes stylie-transform-keyframes {
    0% {-o-transform:translateX(0px) translateY(0px);}
    100% {-o-transform:translateX(400px) translateY(0px);}
  }
  @-webkit-keyframes stylie-transform-keyframes {
    0% {-webkit-transform:translateX(0px) translateY(0px);}
    100% {-webkit-transform:translateX(400px) translateY(0px);}
  }
  @keyframes stylie-transform-keyframes {
    0% {transform:translateX(0px) translateY(0px);}
    100% {transform:translateX(400px) translateY(0px);}
  }

…и затем вставить эти строчки в ваш CSS-код. Заметим, что это необходимо только
для одной самой простой анимации. Теперь посмотрим, как создать такую же
анимацию с помощью модуля `CSSRenderer` библиотеки Rekapi:

  var kapi = new Kapi();
  var actor = new Kapi.DOMActor(
      document.getElementsByClass('stylie')[0]);

  kapi.addActor(actor);
  actor
    .keyframe(0, {
        transform: 'translateX(0px) translateY(0px)'});
    .keyframe(2000, {
        transform: 'translateX(400px) translateY(0px)'});

  // Feature detect for @keyframe support
   if (kapi.css.canAnimateWithCSS()) {
     kapi.css.play();
   } else {
     kapi.play();
   }

Rekapi забирает себе всю головную боль о префиксах, позволяя вам единожды
написать саму анимацию. Если браузер не поддерживает CSS-анимации, Rekapi
сдеградирует их до старой, доброй реализации на JavaScript, доступном на любой
платформе. Чтобы осознать и потрогать на живой странице все те сложные анимации,
которые можно создать с помощью Rekapi, я призываю вас посмотреть на [Stylie][5]
— простейшая демостраничка Rekapi.


## Более качественные анимации ##

CSS позволяет разработчикам и дизайнерам создавать красивые, плавные и
неблокирующие, производительные анимации. В тоже время, нам недоставало
инструментов для простой работы с этими анимациями. Я действительно хочу решить
эту проблему и я думаю, что [Rekapi][2] и её [модуль `CSSRenderer`][6]
значительно сократят отставание в компромиссе между производительностью и
гибкостью. Я надеюсь что техника CSS-пререндеринга найдёт своё широкое
применение в интернете, так как улучшает пользовательский опыт. Мне [интересно
знать][8] как CSS-пререндеринг работает для вас и если вы найдёте баги, то,
пожалуйста, [сообщите о них][9].   Весёлой <del>новой</del> анимации!


[1]: http://www.joelambert.co.uk/morf/
[2]: http://rekapi.com/
[3]: http://rekapi.com/dist/doc/ext/to-css/rekapi.to-css.js.html
[4]: http://rekapi.com/dist/doc/src/rekapi.actor.js.html#keyframe
[5]: http://jeremyckahn.github.io/stylie/
[6]: http://rekapi.com/dist/doc/ext/css-animate/rekapi.css-animate.context.js.html
[7]: http://rekapi.com/ext/css-animate/sample/play-many-actors.html
[8]: https://twitter.com/jeremyckahn
[9]: https://github.com/jeremyckahn/rekapi/issues?page=1&state=open