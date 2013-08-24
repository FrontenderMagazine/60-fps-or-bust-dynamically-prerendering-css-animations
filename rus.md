# 60 кадров в секунду или динамический пререндеринг CSS-анимаций

Создание всё более красивых анимаций для сайтов — это то, чем я занимался
последние пару лет. И только что я обнаружил новый способ предоставляющий больше гибкости и точности, чем все предыдущие. Я назвал этот способ «CSS-пререндеринг» и он проще, чем вы могли подумать.


## Анимации c помощью CSS и JavaScript

Вначале было слово. И этим словом был JavaScript. Многие годы JavaScript был
единственной возможностью добавить анимацию на страницу. По мере становления
веб-стандартов, CSS пополнялся новыми возможностями и в определенный момент в
нем появились свойства `animation` и `transition`. Средствами CSS появилась 
возможность создавать намного более плавную анимацию, чем с помощью браузера 
за счет того, что она оптимизировалась на уровне браузера. Однако CSS анимации 
не обеспечивают необходимой гибкости, и разработчикам приходится выбирать между
плавной анимацией с частотой 60 кадров в секунду и возможностью адаптировать
анимацию под свои нужды «налету». 

Анимации с использованием `@keyframes` сложно назвать гибкими так как они 
должны быть заранее описаны в CSS. Использовать `transitions` несколько проще, 
так как они реагируют на любые изменения в CSS которые выполняются с помощью
JavaScript. Однако наши возможности при использовании `transitions` достаточно
ограничены — затруднительно получить анимацию состоящую из нескольких шагов.
Именно эту проблему должны были решить анимации с помощью `@keyframe`, но они
не обеспечивают достаточной свободы, когда речь идет об изменениях «налету».


Кроме того, CSS-анимации ограничены набором доступных кривых Безье — их всего 
шесть. Этого недостаточно. Джо Ламберт (Joe Lambert) создал [Morf][1] — 
инструмент для генерации `@-webkit-keyframe` анимаций с большим количеством 
крохотных шагов. Это позволяет реализовать любые функции описывающие процесс
изменения значений. На основе этого инструмента я создал функцию [`toCSS`][3] 
для библиотеки [Rekapi][2], которая для генерации кроссбраузерного CSS 
использует обычный [keyframe API][4]. Затем я использовал эту возможность для
создания библиотеки [Stylie][5], которая обеспечивает пользовательский интерфейс для создания анимаций, и экспорта их в CSS.

Я чувствовал, что надёжный интерфейс для создания и редактирования 
CSS-анимаций важен, но сам процесс, включающий создание анимации и копирования 
сотен строк в статический файл с CSS некрасив и негибок. Я верил, что существует лучший путь. Так я пришел CSS-пререндерингу.

## CSS-пререндеринг

Главная идея CSS-пререндеринга проста и прозрачна — вы налету создаёте строчку с
`@keyframe`, вставляете в только что созданный `style`-элемент и вставляете этот
элемент в DOM-дерево. По окончании анимации вы просто удаляете потерявший
необходимость `style`-элемент. Такой способ работает удивительно хорошо, в
частности в `WebKit`-браузерах (другие движки ещё предстоит потестировать).

Я не смог найти ни одного инструмента, который позволил бы реализовать 
подобное, поэтому я решил решить восполнить пробел с помощью 
[модуля `CSSRenderer`][8] для библиотеки Rekapi. Цель Rekapi в том, чтобы 
обеспечить вас гибким и удобным API для создание анимаций на основе ключевых 
кадров и исторически сложилось, что она позволяет создавать JavaScript-
анимации. Используя `CSSRenderer`, вы можете создавать те же анимации, только 
уже на базе CSS, гораздо более плавных и не блокирующих выполнение JavaScript-
потока. Убедитесь в этом сами, [разницу можно заметить невооруженным глазом][7]. 
Разница в качестве сильнее заметна на мобильных устройствах, особенно на iOS.

## Создание CSS-анимаций с помощью Rekapi

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

  // Определяем поддержку @keyframe
   if (kapi.css.canAnimateWithCSS()) {
     kapi.css.play();
   } else {
     kapi.play();
   }

Rekapi избавляет вам от головной боли с префиксами и позволяя вам единожды
написать саму анимацию. Если браузер не поддерживает CSS-анимации, Rekapi
реализует ее с помощью старого, доброго JavaScript, который доступен на любой
платформе. Что бы увидеть с анимациями какой сложности может работать Rekapi я 
рекомендую поиграть со [Stylie][5], который представляет из себя графическую 
надстройку над ним.

## Более качественные анимации

СSS позволяет разработчикам создавать плавные, эффективные с точки зрения 
производительности и анимации, которые не блокируют потоки JavaScript. Однако, 
нам не хватает инструментов для их создания. Я действительно хочу решить эту 
проблему и думаю, что [Rekapi][2] с [`CSSRenderer`][6] сделают нахождение 
компромисса между производительностью и гибкостью анимации проще. Надеюсь CSS-
пререндеринг найдет свое применение, так как он позволяет обеспечить лучший
пользовательский опыт. [Я бы хотел знать][8], как вы используете CSS-
пререндеринг и, если вы обнаружите ошибки — [сообщите о них][9]. Весёлой 
анимации!

[1]: http://www.joelambert.co.uk/morf/
[2]: http://rekapi.com/
[3]: http://rekapi.com/dist/doc/ext/to-css/rekapi.to-css.js.html
[4]: http://rekapi.com/dist/doc/src/rekapi.actor.js.html#keyframe
[5]: http://jeremyckahn.github.io/stylie/
[6]: http://rekapi.com/dist/doc/ext/css-animate/rekapi.css-animate.context.js.html
[7]: http://rekapi.com/ext/css-animate/sample/play-many-actors.html
[8]: https://twitter.com/jeremyckahn
[9]: https://github.com/jeremyckahn/rekapi/issues?page=1&state=open