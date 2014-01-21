# Разрабатываем API на JavaScript лучше

Рано или поздно вы обнаружите, что пишете на JavaScript код, который сложнее
пары строчек плагина jQuery. Ваш код будет делать огромное количество вещей,
им будет (в идеале) пользоваться множество людей с разными подходами к вашему
коду. У них будут различные уровни потребностей, знаний и ожиданий.

![Время, потраченное на создание и на использование][1]

Эта статья описывает наиболее важные вещи, которые вам следует иметь в виду
до и во время написания собственных утилит и библиотек. Мы обратим внимание
на то, каким образом сделать код доступным для других разработчиков.
Для освещения нескольких тем мы затронем jQuery в целях демонстрации, однако
эта статья не о jQuery и не о том, как писать для него плагины.

Питер Друкер однажды выразился: «Компьютер — идиот». Не пишите код для
идиотов, пишите для людей! Давайте же окунёмся в **разработку таких API,
которые полюбятся разработчикам**.

### Текучий интерфейс

[Текучим интерфейсом][2] часто называют *цепные вызовы* (хотя это всего
половина правды). В глазах новичков он выглядит как *стиль jQuery*.
Хотя я и уверен, что стиль API стал ключевым фактором успеха jQuery, он не
был ими изобретён, лавры создателя, похоже, принадлежат Мартину Фаулеру,
который [придумал этот термин][3] ещё в 2005, примерно за год до релиза
jQuery. Но с другой стороны, Фаулер лишь дал этому понятию название, текучие
интерфейсы использовались и гораздо раньше.

Кроме значительных упрощений, jQuery позволил сгладить серьёзные различия
между браузерами. jQuery всегда был *текучим интерфейсом*, и это мне
нравилось больше всего в этой крайне успешной библиотеке. Я наслаждался именно
таким API настолько, что сразу стало ясно: именно такой стиль я хочу
использовать в [URI.js][4]. Во время тонкой настройки API URI.js, я постоянно
просматривал исходники jQuery в поисках маленьких трюков, которые позволили бы
сделать мою реализацию настолько простой, насколько это возможно. И оказалось,
что не я один прилагал усилия вэтом направлении. [Леа Веру][5] создала
[chainvas][6], утилиту для оборачивания обычных API с гетерами и сеттерами
в приятные текучие интефейсы. [`_.chain()`][7] в Underscore тоже делает что-то
подобное. В сущности, большинство библиотек нового поколения поддерживают 
цепные вызовы.

#### Цепные вызовы

Главная цель [цепных вызовов][8] в том, чтобы код как можно более легко
читался с тем, чтобы его можно понять быстрее. Используя *цепные вызовы*,
можно сделать код похожим на предложения, делая их проще для чтения и при этом
избавляясь от шума:

    // обычные вызовы API для смены парочки цветов и добавления обработчика события
    var elem = document.getElementById("foobar");
    elem.style.background = "red";
    elem.style.color = "green";
    elem.addEventListener('click', function(event) {
      alert("hello world!");
    }, true);
    
    // (воображаемый) API с цепным вызовом
    DOMHelper.getElementById('foobar')
      .setStyle("background", "red")
      .setStyle("color", "green")
      .addEvent("click", function(event) {
        alert("hello world");
      });

Обратите внимания, нам не пришлось хранить ссылку на элемент в переменной и
повторять её снова и снова.

#### Разделение команд и запросов

[Разделение команд и запросов][9] (CQS) — это концепция, пришедшая из
императивного программирования. Функции, изменяющие состояние (внутренние
значения) объекта, называются *командами*, функции, получающие значения,
называются *запросами*. Общий принцип: запросы возвращают данные, команды
меняют состояние, но никогда и то и другое вместе. Эта концепция лежит в
основе повседневных геттеров и сеттеров, которые мы можем увидеть в
большинстве современных библиотек. Коль скоро *текучие интерфейсы* возвращают
ссылку на себя для цепного вызова методов, мы уже нарушаем правило для
*команд*, потому как они по-хорошему не должны ничего возвращать. Вдовесок
к этой (легко игнорируемой) особенности мы (преднамеренно) отходим от этого
принципа с тем, чтобы придерживаться максимальной простоты в API.
Отличный пример такого подхода — метод [`css()`][10] jQuery:

    var $elem = jQuery("#foobar");
    
    // CQS - команда
    $elem.setCss("background", "green");
    // CQS - запрос
    $elem.getCss("color") === "red";
    
    // не-CQS - команда
    $elem.css("background", "green");
    // не-CQS - запрос
    $elem.css("color") === "red";

Как вы можете увидеть, и геттер, и сеттер объединены в один метод. Действие,
которое следует совершить (а именно, *запрос* или *команду*), определяется
по количеству аргументов, переданных в функцию, а не тем, какая функция
вызывается. Это позволяет экспортировать меньше методов и, в свою очередь,
меньше печатать, чтобы добиться той же цели.

Чтобы создать текучий интерфейс, сжимать геттеры и сеттеры в один метод вовсе
не обязательно, это скорее дело вкуса. Ваша документация должна явно
указывать, какой именно подход вы используете. Мы вернёмся к теме
документирования API позже, а сейчас я хотел бы заметить, что функции со
множественными сигнатурами, возможно, будет сложнее документировать.

#### Переходим на текучесть

Хотя текучесть уже по большей части обеспечивается цепными вызовами, мы ещё
не закончили. Следующий шаг к *текучести* поможет продемонстрировать такой
пример: представим, что мы пишем маленькую библиотечку для работы с
интервалами дат. Интервал начинается с даты и датой заканчивается. Дата
не обязательно связана с интервалом. Размышляя так мы приходим к простому
конструктору:

    // создаём новый временной интервал
    var interval = new DateInterval(startDate, endDate);
    // получаем посчитанное количество дней в интервале
    var days = interval.days();

Хотя на первый взгляд кажется, что всё верно, этот пример показывает, что
не так:

    var startDate = new Date(2012, 0, 1);
    var endDate = new Date(2012, 11, 31)
    var interval = new DateInterval(startDate, endDate);
    var days = interval.days(); // 365

Мы написали целую кучу переменных и хлама, который нам, возможно, не нужен.
Хорошим решением этой проблемы было бы добавление объекту Date функции,
которая возвращает интервал:

    // создаёт DateInterval для текучего вызова
    Date.prototype.until = function(end) {
    
      // если дата не была передана, создаём её
      if (!(end instanceof Date)) {
        // создаём дату, передавая полученные аргументы 
        // в конструктор без изменения, таким образом,
        // эта функция принимает точно такие
        // же параметры, как и конструктор Date.
        end = Date.apply(null, 
          Array.prototype.slice.call(arguments, 0)
        );
      }
    
      return new DateInterval(this, end);
    };

Так мы создали `DateInterval` в текучей, лёгкой для написания и прочтения
манере:

    var startDate = new Date(2012, 0, 1);
    var interval = startDate.until(2012, 11, 31);
    var days = interval.days(); // 365
    
    // сжатый вызов текучего интерфейса:
    var days = (new Date(2012, 0, 1))
      .until(2012, 11, 31) // возвращает экземпляр DateInterval
      .days(); // 365

Как вы видите, в последнем примере пришлось объявить меньше переменных и
написать меньше кода, а сама операция читается как предложение на английском
языке. Этот пример был призван помочь вам осознать, что цепные вызовы — это
лишь часть текучего интерфейса, и эти понятия не синонимы. Для обеспечения
текучести вы должны думать о потоках кода — откуда вы начинаете, и куда
направляетесь.

`//TODO: Перевести следующий абзац ближе к тексту`

Этот пример иллюстрировал текучесть через расширение нативного объекта
произвольной функцией. Это такой же предмет религиозных войн, как и вопрос,
надо ли использовать точки с запятой, или нет. В статье
[Расширение встроенных нативных объектов. Зло или нет?][11] [kangax][12]
объясняет плюсы и минусы такого подхода. Хотя у каждого есть своё мнение,
все сходятся в одном: в таких вещах нужно единообразие. Впрочем, даже
приверженцы позиции «нельзя загрязнять нативные объекты своими функциями»
допустили бы такой, по-своему текучий, приём:

    String.prototype.foo = function() {
      return new Foo(this);
    }
    
    "Я нативный объект".foo()
      .iAmACustomFunction();

С таким подходом ваши функции по-прежнему находятся внутри собственного
пространства имён, но при этом достуны через другой объект. Убедитесь, что
ваш эквивалент `.foo()` не является общим термином и очень навряд ли
пересечётся с другими API. Убедитесь, что вы должным образом предоставили
методы [`.valueOf()`][13] и [`.toString()`][14], чтобы объекты можно было
преобразовать обратно в изначальные примитивные типы.

### Единообразие

У [Джейка Арчибальд][15] был слайд с определением слова *единообразие*.
Оно гласило просто: [*не PHP*][16]. Не. Дай. Бог. Вы вздумаете назвать
функцию вроде *str_repeat()*, *strpos()*, *substr()*. Также никогда в жизни
не меняйте позиции аргументов. Если вы объявили в каком-то одном месте 
`find_in_array(haystack, needle)`, то добавление
`findInString(needle, haystack)` призовёт разъярённую толпу зомби подняться
из своих могил, выследить вас и заставить писать на delphi до скончания
жизни!

#### Именование вещей

> «В вычислительной науке всего две сложные задачи: инвалидация кэша
> и именование переменных.»
>
> — Фил Карлтон

`//TODO: двойное отрицание, проверить!`

Я посещал множество встреч и семинаров, где меня пытались научить тонкостям
именования вещей. Не было случая, чтобы ушёл оттуда не услышав упомянутую
выше цитату и не узнав, как же на самом деле следует именовать вещи.
Мой совет сводится к *называйте кратко но осмысленно, и доверьтесь интуиции*.
Но прежде всего, соблюдайте единообразие.

В примере с `DateInterval` выше был метод под названием `until()`. Мы могли
бы назвать эту функцию `interval()`. Последнее было бы ближе к возвращаемому
значению, хотя первое более *человеко-читаемое*. Подберите такие формулировки,
какие вам нравятся, и придерживайтесь их. Единообразие — это 90% того, что
имеет значение. Выберите один стиль и сохраняйте этот стиль, даже если в
будущем он вам разонравится.

### Обработка аргументов

![Благие намерения][17]

То, каким образом ваши методы принимают значения, важнее, чем стремление
сделать их цепными. Цепные вызовы — это достаточно обыденная вещь, которую
несложно воплотить в коде, а вот обработка аргументов — нет. Вам нужно
продумать то, как скорее всего будут использоваться предоставленные вами
методы. Будет ли код, использующий ваш API, повторять определённые
вызовы функций? Почему эти вызовы будут повторяться? Как может ваш API
помочь разработчику уменьшить шум от повторяющихся вызовов методов?

Метод [`css()`][10] в jQuery может устанавливать стили элементов DOM:

    jQuery("#some-selector")
      .css("background", "red")
      .css("color", "white")
      .css("font-weight", "bold")
      .css("padding", 10);

Тут же паттерн! Каждый вызов метода указывает имя стиля и определяет значение
для него. Возможность передать эти данные в виде объекта-словаря так и
просится:

    jQuery("#some-selector").css({
      "background" : "red",
      "color" : "white",
      "font-weight" : "bold",
      "padding" : 10
    });

Метод [`on()`][18] jQuery может регистрировать обработчики событий. Как и
`css()`, он может принимать словарь событий, но идёт ещё дальше, позволяя
зарегистрировать один обработчик на несколько событий:

    // binding events by passing a map
    jQuery("#some-selector").on({
      "click" : myClickHandler,
      "keyup" : myKeyupHandler,
      "change" : myChangeHandler
    });
    
    // binding a handler to multiple events:
    jQuery("#some-selector").on("click keyup change", myEventHandler);

You can offer the above function signatures by using the following *method
pattern*:

    DateInterval.prototype.values = function(name, value) {
      var map;
    
      if (jQuery.isPlainObject(name)) {
        // setting a map
        map = name;
      } else if (value !== undefined) {
        // setting a value (on possibly multiple names), convert to map
        keys = name.split(" ");
        map = {};
        for (var i = 0, length = keys.length; i < length; i++) {
          map[keys[i]] = value;
        }
      } else if (name === undefined) {
        // getting all values
        return this.values;
      } else {
        // getting specific value
        return this.values[name];
      }
    
      for (var key in map) {
        this.values[name] = map[key];
      }
    
      return this;
    };

If you are working with collections, think about what you can do to reduce the
number of loops an API user would probably have to make. Say we had a number of
`<input>` elements for which we want to set the default value:

    <input type="text" value="" data-default="foo">
    <input type="text" value="" data-default="bar">
    <input type="text" value="" data-default="baz">

We’d probably go about this with a loop:

    jQuery("input").each(function() {
      var $this = jQuery(this);
      $this.val($this.data("default"));
    });

What if we could bypass that method with a simple callback that gets applied to
each`<input>` in the collection? jQuery developers have thought of that
and allow us to write less
™:

    jQuery("input").val(function() {
      return jQuery(this).data("default");
    });

It’s the little things like accepting maps, callbacks or serialized attribute
names, that make using your API not only cleaner, but more comfortable and 
efficient to use. Obviously not all of your APIs’ methods will benefit from this
method pattern — it’s up to you to decide where all this makes sense and where 
it is just a waste of time. Try to be as consistent about this as humanly 
possible. *Reduce the need for boilerplate code with the tricks shown above and
people will invite you over for a drink.*

#### Handling Types {#handling-types}

Whenever you define a function that will accept arguments, you decide what data
that function accepts. A function to calculate the number of days between two 
dates could look like:

    DateInterval.prototype.days = function(start, end) {
      return Math.floor((end - start) / 86400000);
    };

As you can see, the function expects numeric input — a millisecond timestamp
, to be exact. While the function does what we intended it to do, it is not very
versatile. What if we’re working with`Date` objects or a string representation
of a date? Is the user of this function supposed to cast data all the time? No! 
Simply verifying the input and casting it to whatever we need it to be should be
done in a central place, not cluttered throughout the code using our API:

    DateInterval.prototype.days = function(start, end) {
      if (!(start instanceof Date)) {
        start = new Date(start);
      }
      if (!(end instanceof Date)) {
        end = new Date(end);
      }
    
      return Math.floor((end.getTime() - start.getTime()) / 86400000);
    };

By adding these six lines we’ve given the function the power to accept a Date
object, a numeric timestamp, or even a string representation like
`Sat Sep 08 2012 15:34:35 GMT+0200 (CEST)`. We do not know how and for what
people are going to use our code, but with a little foresight, we can make sure 
there is little pain with integrating our code.

The experienced developer can spot another problem in the example code. We’re
assuming`start` comes before `end`. If the API user accidentally swapped the
dates, he’d be given a negative value for the number of days between`start` and
`end`. Stop and think about these situations carefully. If you’ve come to the
conclusion that a negative value doesn’t make sense, fix it:

    DateInterval.prototype.days = function(start, end) {
      if (!(start instanceof Date)) {
        start = new Date(start);
      }
      if (!(end instanceof Date)) {
        end = new Date(end);
      }
    
      return Math.abs(Math.floor((end.getTime() - start.getTime()) / 86400000));
    };

JavaScript allows type casting a number of ways. If you’re dealing with
primitives (string, number, boolean) it can get as simple (as in “short”) as:

    function castaway(some_string, some_integer, some_boolean) {
      some_string += "";
      some_integer += 0; // parseInt(some_integer, 10) is the safer bet
      some_boolean = !!some_boolean;
    }

I’m not advocating you to do this everywhere and at all times. But these
innocent looking lines may save time and some suffering while integrating your 
code.

#### Treating `undefined` as an Expected Value {#treating-undefined-as-an-
expected-value
}

There will come a time when `undefined` is a value that your API actually
expects to be given for setting an attribute. This might happen to “unset” an 
attribute, or simply to gracefully handle bad input, making your API more robust.
To identify if the value`undefined` has actually been passed by your method,
you can check the[`arguments`][19] object:

    function testUndefined(expecting, someArgument) {
      if (someArgument === undefined) {
        console.log("someArgument was undefined");
      }
      if (arguments.length > 1) {
        console.log("but was actually passed in");
      }
    }
    
    testUndefined("foo");
    // prints: someArgument was undefined
    testUndefined("foo", undefined);
    // prints: someArgument was undefined, but was actually passed in

#### Named Arguments {#named-arguments}

    event.initMouseEvent(
      "click", true, true, window, 
      123, 101, 202, 101, 202, 
      true, false, false, false, 
      1, null);

The function signature of [Event.initMouseEvent][20] is a nightmare come true.
There is no chance any developer will remember what that`1` (second to last
parameter) means without looking it up in the documentation. No matter how good 
your documentation is, do what you can so people don’t have to look things up!

#### How Others Do It {#how-others-do-it}

Looking beyond our beloved language, we find Python knowing a concept called 
[named arguments][21]. It allows you to declare a function providing default
values for arguments, allowing your attributed names to be stated in the calling
context:

    function namesAreAwesome(foo=1, bar=2) {
      console.log(foo, bar);
    }
    
    namesAreAwesome();
    // prints: 1, 2
    
    namesAreAwesome(3, 4);
    // prints: 3, 4
    
    namesAreAwesome(foo=5, bar=6);
    // prints: 5, 6
    
    namesAreAwesome(bar=6);
    // prints: 1, 6

Given this scheme, initMouseEvent() could’ve looked like a self-explaining
function call:

    event.initMouseEvent(
      type="click", 
      canBubble=true, 
      cancelable=true, 
      view=window, 
      detail=123,
      screenX=101, 
      screenY=202, 
      clientX=101, 
      clientY=202, 
      ctrlKey=true, 
      altKey=false, 
      shiftKey=false, 
      metaKey=false, 
      button=1, 
      relatedTarget=null);

In JavaScript this is not possible today. While “the next version of
JavaScript” (frequently called ES.next, ES6, or Harmony) will have
[default parameter values][22] and [rest parameters][23], there is still no
sign of named parameters.

#### Argument Maps {#argument-maps}

JavaScript not being Python (and ES.next being light years away), we’re left
with fewer choices to overcome the obstacle of “argument forests”. jQuery (and 
pretty much every other decent API out there) chose to work with the concept of
“option objects”. The signature of[jQuery.ajax()][24] provides a pretty good
example. Instead of accepting numerous arguments, we only accept an object:

    function nightmare(accepts, async, beforeSend, cache, complete, /* and 28 more */) {
      if (accepts === "text") {
        // prepare for receiving plain text
      }
    }
    
    function dream(options) {
      options = options || {};
      if (options.accepts === "text") {
        // prepare for receiving plain text
      }
    }

Not only does this prevent insanely long function signatures, it also makes
calling the function more descriptive:

    nightmare("text", true, undefined, false, undefined, /* and 28 more */);
    
    dream({
      accepts: "text",
      async: true,
      cache: false
    });

Also, we do not have to touch the function signature (adding a new argument)
should we introduce a new feature in a later version.

#### Default Argument Values {#default-argument-values}

[jQuery.extend()][25], [_.extend()][26] and Protoype’s [Object.extend][27] are
functions that let you merge objects, allowing you to throw your own preset 
options object into the mix:

    var default_options = {
      accepts: "text",
      async: true,
      beforeSend: null,
      cache: false,
      complete: null,
      // …
    };
    
    function dream(options) {
      var o = jQuery.extend({}, default_options, options || {});
      console.log(o.accepts);
    }
    
    // make defaults public
    dream.default_options = default_options;
    
    dream({ async: false });
    // prints: "text"

You’re earning bonus points for making the default values publicly accessible
. With this, anyone can change`accepts` to “json” in a central place, and
thus avoid specifying that option over and over again. Note that the example 
will always append`|| {}` to the initial read of the option object. This allows
you to call the function without an argument given.

#### Good Intentions — a.k.a. “Pitfalls” {#good-intentions–aka-pitfalls}

Now that you know how to be truly flexible in accepting arguments, we need to
come back to an old saying:

> “With great power comes great responsibility!”
> 
> — Voltaire

As with most weakly-typed languages, JavaScript does automatic casting when it
needs to. A simple example is testing the truthfulness:

    var foo = 1;
    var bar = true;
    
    if (foo) {
      // yep, this will execute
    }
    
    if (bar) {
      // yep, this will execute
    }

We’re quite used to this automatic casting. We’re so used to it, that we
forget that although something is truthful, it may not be the boolean truth. 
Some APIs are so flexible they are *too smart* for their own good. Take a look
at the signatures of[jQuery.toggle()][28]:

    .toggle( /* int */ [duration] [, /* function */  callback] )
    .toggle( /* int */ [duration] [, /* string */  easing] [, /* function */ callback] )
    .toggle( /* bool */ showOrHide )

It will take us some time decrypting why these behave *entirely* different:

    var foo = 1;
    var bar = true;
    var $hello = jQuery(".hello");
    var $world = jQuery(".world");
    
    $hello.toggle(foo);
    $world.toggle(bar);

We were *expecting* to use the `showOrHide` signature in both cases. But what
really happened is`$hello` doing a toggle with a `duration` of one millisecond
. This is not a bug in jQuery, this is a simple case of *expectation not met*.
Even if you’re an experienced jQuery developer, you *will* trip over this from
time to time.

You are free to add as much convenience / sugar as you like — but do not
sacrifice a clean and (mostly) robust API along the way. If you find yourself 
providing something like this, think about providing a separate method like
`.toggleIf(bool)` instead. Whatever choice you make, keep your API consistent
!

### Extensibility {#extensibility}

![Developing Possibilities][29]

With option objects, we’ve covered the topic of extensible configuration. Let
’s talk about allowing the API user to extend the core and API itself. This is 
an important topic, as it allows your code to focus on the important things, 
while having API users implement edge-cases themselves. Good APIs are concise 
APIs. Having a hand full of configuration options is fine, but having a couple 
dozen of them makes your API feel bloated and opaque. Focus on the primary-use 
cases, only do the things most of your API users will need. Everything else 
should be left up to them. To allow API users to extend your code to suit their 
needs, you have a couple of options
…

#### Callbacks {#callbacks}

Callbacks can be used to achieve extensibility by configuration. You can use
callbacks to allow the API user to override certain parts of your code. When you
feel specific tasks may be handled differently than your default code, refactor 
that code into a configurable callback function to allow an API user to easily 
override that:

    var default_options = {
      // ...
      position: function($elem, $parent) {
        $elem.css($parent.position());
      }
    };
    
    function Widget(options) {
      this.options = jQuery.extend({}, default_options, options || {});
      this.create();
    };
    
    Widget.prototype.create = function() {
      this.$container = $("<div></div>").appendTo(document.body);
      this.$thingie = $("<div></div>").appendTo(this.$container);
      return this;
    };
    
    Widget.protoype.show = function() {
      this.options.position(this.$thingie, this.$container);
      this.$thingie.show();
      return this;
    };

    var widget = new Widget({
      position: function($elem, $parent) {
        var position = $parent.position();
        // position $elem at the lower right corner of $parent
        position.left += $parent.width();
        position.top += $parent.height();
        $elem.css(position);
      }
    });
    widget.show();

Callbacks are also a generic way to allow API users to customize elements your
code has created:

    // default create callback doesn't do anything
    default_options.create = function($thingie){};
    
    Widget.prototype.create = function() {
      this.$container = $("<div></div>").appendTo(document.body);
      this.$thingie = $("<div></div>").appendTo(this.$container);
      // execute create callback to allow decoration
      this.options.create(this.$thingie);
      return this;
    };

    var widget = new Widget({
      create: function($elem) {
        $elem.addClass('my-style-stuff');
      }
    });
    widget.show();

Whenever you accept callbacks, be sure to document their signature and provide
examples to help API users customize your code. Make sure you’re consistent 
about the context (where`this` points to) in which callbacks are executed in,
and the arguments they accept.

#### Events {#events}

Events come naturally when working with the DOM. In larger application we use
events in various forms (e.g. PubSub) to enable communication between modules. 
Events are particularly useful and feel most natural when dealing with UI 
widgets. Libraries like jQuery offer simple interfaces allowing you to easily 
conquer this domain.

Events interface best when there is something happening — hence the name.
Showing and hiding a widget could depend on circumstances outside of your scope.
Updating the widget when it’s shown is also a very common thing to do. Both can 
be achieved quite easily using jQuery’s event interface, which even allows for 
the use of delegated events:

    Widget.prototype.show = function() {
      var event = jQuery.Event("widget:show");
      this.$container.trigger(event);
      if (event.isDefaultPrevented()) {
        // event handler prevents us from showing
        return this;
      }
    
      this.options.position(this.$thingie, this.$container);
      this.$thingie.show();
      return this;
    };

    // listen for all widget:show events
    $(document.body).on('widget:show', function(event) {
      if (Math.random() > 0.5) {
        // prevent widget from showing
        event.preventDefault();
      }
    
      // update widget's data
      $(this).data("last-show", new Date());
    });
    
    var widget = new Widget();
    widget.show();

You can freely choose event names. Avoid using [native events][30] for
proprietary things and consider namespacing your events. jQuery UI’s event names
are comprised of the widget’s name and the event name`dialogshow`. I find that
hard to read and often default to`dialog:show`, mainly because it is
immediately clear that this is a custom event, rather than something some 
browser might have secretly implemented.

### Hooks {#hooks}

Traditional getter and setter methods can especially benefit from hooks. Hooks
usually differ from callbacks in their number and how they’re registered. Where 
callbacks are usually used on an instance level for a specific task, hooks are 
usually used on a global level to customize values or dispatch custom actions. 
To illustrate how hooks can be used, we’ll take a peek at
[jQuery’s cssHooks][31]:

    // define a custom css hook
    jQuery.cssHooks.custombox = {
      get: function(elem, computed, extra) {
        return $.css(elem, 'borderRadius') == "50%"
          ? "circle"
          : "box";
      },
      set: function(elem, value) {
        elem.style.borderRadius = value == "circle"
          ? "50%"
          : "0";
      }
    };
    
    // have .css() use that hook
    $("#some-selector").css("custombox", "circle");

By registering the hook `custombox` we’ve given jQuery’s `.css()` method the
ability to handle a CSS property it previously couldn’t. In my article
[jQuery hooks][32], I explain the other hooks that jQuery provides and how they
can be used in the field. You can provide hooks much like you would handle 
callbacks:

    DateInterval.nameHooks = {
      "yesterday" : function() {
        var d = new Date();
        d.setTime(d.getTime() - 86400000);
        d.setHours(0);
        d.setMinutes(0);
        d.setSeconds(0);
        return d;
      }
    };
    
    DateInterval.prototype.start = function(date) {
      if (date === undefined) {
        return new Date(this.startDate.getTime());
      }
    
      if (typeof date === "string" && DateInterval.nameHooks[date]) {
        date = DateInterval.nameHooks[date]();
      }
    
      if (!(date instanceof Date)) {
        date = new Date(date);
      }
    
      this.startDate.setTime(date.getTime());
      return this;
    };

    var di = new DateInterval();
    di.start("yesterday");

In a way, hooks are a collection of callbacks designed to handle custom values
within your own code. With hooks you can stay in control of almost everything, 
while still giving API users the option to customize.

### Generating Accessors {#generating-accessors}

![Duplication][33]

Any API is likely to have multiple accessor methods (getters, setters,
executors) doing similar work. Coming back to our`DateInterval` example, we’
re most likely providing`start()` and `end()` to allow manipulation of
intervals. A simple solution could look like:

    DateInterval.prototype.start = function(date) {
      if (date === undefined) {
        return new Date(this.startDate.getTime());
      }
    
      this.startDate.setTime(date.getTime());
      return this;
    };
    
    DateInterval.prototype.end = function(date) {
      if (date === undefined) {
        return new Date(this.endDate.getTime());
      }
    
      this.endDate.setTime(date.getTime());
      return this;
    };

As you can see we have a lot of repeating code. A DRY (Don’t Repeat Yourself
) solution might use this generator pattern:

    var accessors = ["start", "end"];
    for (var i = 0, length = accessors.length; i < length; i++) {
      var key = accessors[i];
      DateInterval.prototype[key] = generateAccessor(key);
    }
    
    function generateAccessor(key) {
      var value = key + "Date";
      return function(date) {
        if (date === undefined) {
          return new Date(this[value].getTime());
        }
    
        this[value].setTime(date.getTime());
        return this;
      };
    }

This approach allows you to generate multiple similar accessor methods, rather
than defining every method separately. If your accessor methods require more 
data to setup than just a simple string, consider something along the lines of:

    var accessors = {"start" : {color: "green"}, "end" : {color: "red"}};
    for (var key in accessors) {
      DateInterval.prototype[key] = generateAccessor(key, accessors[key]);
    }
    
    function generateAccessor(key, accessor) {
      var value = key + "Date";
      return function(date) {
        // setting something up 
        // using `key` and `accessor.color`
      };
    }

In the chapter *Handling Arguments* we talked about a method pattern to allow
your getters and setters to accept various useful types like maps and arrays. 
The method pattern itself is a pretty generic thing and could easily be turned 
into a generator:

    function wrapFlexibleAccessor(get, set) {
      return function(name, value) {
        var map;
    
        if (jQuery.isPlainObject(name)) {
          // setting a map
          map = name;
        } else if (value !== undefined) {
          // setting a value (on possibly multiple names), convert to map
          keys = name.split(" ");
          map = {};
          for (var i = 0, length = keys.length; i < length; i++) {
            map[keys[i]] = value;
          }
        } else {
          return get.call(this, name);
        }
    
        for (var key in map) {
          set.call(this, name, map[key]);
        }
    
        return this;
      };
    }
    
    DateInterval.prototype.values = wrapFlexibleAccessor(
      function(name) { 
        return name !== undefined 
          ? this.values[name]
          : this.values;
      },
      function(name, value) {
        this.values[name] = value;
      }
    );

Digging into the art of writing DRY code is well beyond this article. 
[Rebecca Murphey][34] wrote [Patterns for DRY-er JavaScript][35] and 
[Mathias Bynens’][36] slide deck on 
[how DRY impacts JavaScript performance][37] are a good start, if you’re new
to the topic.

### The Reference Horror {#the-reference-horror}

Unlike other languages, JavaScript doesn’t know the concepts of *pass by
reference* nor *pass by value*. Passing data by value is a safe thing. It makes
sure data passed to your API and data returned from your API may be modified 
outside of your API without altering the state within. Passing data by reference
is often used to keep memory overhead low, values passed by reference can be 
changed anywhere outside your API and affect state within.

In JavaScript there is no way to tell if arguments should be passed by
reference or value. Primitives (strings, numbers, booleans) are treated as *pass
by value*, while objects (any object, including Array, Date) are handled in a
way that’s comparable to *by reference*. If this is the first you’re hearing
about this topic, let the following example enlighten you:

    // by value
    function addOne(num) {
      num = num + 1; // yes, num++; does the same
      return num;
    }
    
    var x = 0;
    var y = addOne(x);
    // x === 0 <--
    // y === 1
    
    // by reference
    function addOne(obj) {
      obj.num = obj.num + 1;
      return obj;
    }
    
    var ox = {num : 0};
    var oy = addOne(ox);
    // ox.num === 1 <--
    // oy.num === 1

The *by reference* handling of objects can come back and bite you if you’re
not careful. Going back to the`DateInterval` example, check out this bugger:

    var startDate = new Date(2012, 0, 1);
    var endDate = new Date(2012, 11, 31)
    var interval = new DateInterval(startDate, endDate);
    endDate.setMonth(0); // set to january
    var days = interval.days(); // got 31 but expected 365 - ouch!

Unless the constructor of DateInterval *made a copy* (`clone` is the technical
term for a copy) of the values it received, any changes to the original objects 
will reflect on the internals of DateInterval. This is *usually* not what we
want or expect.

Note that the same is true for values returned from your API. If you simply
return an internal object, any changes made to it outside of your API will be 
reflected on your internal data. This is most certainly not what you want.
[jQuery.extend()][25], [_.extend()][26] and Protoype’s [Object.extend][27]
allow you to easily escape the reference horror.

If this summary did not suffice, read the excellent chapter 
[By Value Versus by Reference][38] from O’Reilly’s 
[JavaScript – The Definitive Guide][39].

### The Continuation Problem {#the-continuation-problem}

In a fluent interface, all methods of a chain are executed, regardless of the
state that the base object is in. Consider calling a few methods on a jQuery 
instance that contain no DOM elements:

    jQuery('.wont-find-anything')
      // executed although there is nothing to execute against
      .somePlugin().someOtherPlugin();

In non-fluent code we could have prevented those functions from being executed
:

    var $elem = jQuery('.wont-find-anything');
    if ($elem.length) {
      $elem.somePlugin().someOtherPlugin();
    }

Whenever we chain methods, we lose the ability to prevent certain things from
happening — we can’t escape from the chain. As long as the API developer knows 
that objects can have a state where methods don’t actually do anything but
`return this;`, everything is fine. Depending on what your methods do
internally, it may help to prepend a trivial`is-empty` detection:

    jQuery.fn.somePlugin = function() {
      if (!this.length) {
        // "abort" since we've got nothing to work with
        return this;
      }
    
      // do some computational heavy setup tasks
      for (var i = 10000; i > 0; i--) {
        // I'm just wasting your precious CPU!
        // If you call me often enough, I'll turn
        // your laptop into a rock-melting jet engine
      }
    
      return this.each(function() {
        // do the actual job
      });
    };

### Handling Errors {#handling-errors}

![Fail Faster][40]

I was lying when I said we couldn’t escape from the chain — there is an 
`Exception` to the rule (pardon the pun ☺).

We can always eject by throwing an Error (Exception). Throwing an Error is
considered a deliberate abortion of the current flow, most likely because you 
came into a state that you couldn’t recover from. But beware — not all Errors 
are helping the debugging developer:

    // jQuery accepts this
    $(document.body).on('click', {});
    
    // on click the console screams
    //   TypeError: ((p.event.special[l.origType] || {}).handle || l.handler).apply is not a function 
    //   in jQuery.min.js on Line 3

Errors like these are a major pain to debug. Don’t waste other people’s
time. Inform an API user if he did something stupid:

    if (Object.prototype.toString.call(callback) !== '[object Function]') { // see note
      throw new TypeError("callback is not a function!");
    }

Note: `typeof callback === "function"` should not be used, as older browsers
may report objects to be a`function`, which they are not. In Chrome (up to
version 12
) `RegExp` is such a case. For convenience, use [jQuery.isFunction()][41] or 
[_.isFunction()][42].

Most libraries that I have come across, regardless of language (within the weak
-typing domain) don’t care about rigorous input validation. To be honest, my own
code only validates where I anticipate developers stumbling. Nobody really does 
it, but all of us should. Programmers are a lazy bunch — we don’t write code 
just for the sake of writing code or for some cause we don’t truly believe in. 
The developers of Perl6 have recognized this being a problem and decided to 
incorporate something called *Parameter Constraints*. In JavaScript, their
approach might look something like this:

    function validateAllTheThings(a, b {where typeof b === "numeric" and b < 10}) {
      // Interpreter should throw an Error if b is
      // not a number or greater than 9
    }

While the syntax is as ugly as it gets, the idea is to make validation of input
a top-level citizen of the language. JavaScript is nowhere near being something 
like that. That’s fine — I couldn’t see myself cramming these constraints into 
the function signature anyways. It’s admitting the problem (of weakly-typed 
languages) that is the interesting part of this story.

JavaScript is neither weak nor inferior, we just have to work a bit harder to
make our stuff really robust. Making code robust does not mean accepting any 
data, waving your wand and getting some result. Being robust means not accepting
rubbish *and telling the developer about it*.

Think of input validation this way: A couple of lines of code behind your API
can make sure that no developer has to spend hours chasing down weird bugs 
because they accidentally gave your code a string instead of a number. This is 
the one time you can tell people *they’re wrong* and they’ll actually love you
for doing so.

### Going Asynchronous {#going-asynchronous}

So far we’ve only looked at synchronous APIs. Asynchronous methods usually
accept a callback function to inform the outside world, once a certain task is 
finished. This doesn’t fit too nicely into our fluent interface scheme, though:

    Api.protoype.async = function(callback) {
      console.log("async()");
      // do something asynchronous
      window.setTimeout(callback, 500);
      return this;
    };
    Api.protoype.method = function() {
      console.log("method()");
      return this;
    };

    // running things
    api.async(function() {
      console.log('callback()');
    }).method();
    
    // prints: async(), method(), callback()

This example illustrates how the asynchronous method `async()` begins its work
but immediately returns, leading to`method()` being invoked before the actual
task of`async()` completed. There are times when we want this to happen, but
generally we expect`method()` to execute *after* `async()` has completed its
job.

#### Deferreds (Promises) {#deferreds-promises}

To some extent we can counter the mess that is a mix of asynchronous and
synchronous API calls with[Promises][43]. jQuery knows them as [Deferreds][44]
`this`, which forces you to eject from method chaining. This may seem odd at
first, but it effectively prevents you from continuing synchronously after 
invoking an asynchronous method:

    Api.protoype.async = function() {
      var deferred = $.Deferred();
      console.log("async()");
    
      window.setTimeout(function() {
        // do something asynchronous
        deferred.resolve("some-data");
      }, 500);
    
      return deferred.promise();
    };

    api.async().done(function(data) {
      console.log("callback()");
      api.method();
    });
    
    // prints: async(), callback(), method()

The Deferred object let’s you register handlers using `.done()`, `.fail()`, 
`.always()` to be called when the asynchronous task has completed, failed, or
regardless of its state. See[Promise Pipelines In JavaScript][45] for a more
detailed introduction to Deferreds.

### Debugging Fluent Interfaces {#debugging-fluent-interfaces}

While *Fluent Interfaces* are much nicer to develop with, they do come with
certain limitations regarding de-buggability.

As with any code, *Test Driven Development* (TDD) is an easy way to reduce
debugging needs. Having written URI.js in TDD, I have not come across major 
pains regarding debugging my code. However, TDD only *reduces* the need for
debugging — it doesn’t replace it entirely.

Some voices on the internet suggest writing out each component of a chain in
their separate lines to get proper line-numbers for errors in a stack trace:

    foobar.bar()
      .baz()
      .bam()
      .someError();

This technique does have its benefits (though better debugging is not a solid
part of it). Code that is written like the above example is even simpler to read.
Line-based differentials (used in version control systems like SVN, GIT) might 
see a slight win as well. Debugging-wise, it is only Chrome (at the moment), 
that will show`someError()` to be on line four, while other browsers treat it
as line one.

Adding a simple method to logging your objects can already help a lot —
although that is considered “manual debugging” and may be frowned upon by people
used to “real” debuggers:

    DateInterval.prototype.explain = function() {
      // log the current state to the console
      console.dir(this);
    };
    
    var days = (new Date(2012, 0, 1))
      .until(2012, 11, 31) // returns DateInterval instance
      .explain() // write some infos to the console
      .days(); // 365

#### Function names {#function-names}

Throughout this article you’ve seen a lot of demo code in the style of 
`Foo.prototype.something = function(){}`. This style was chosen to keep
examples brief. When writing APIs you might want to consider either of the 
following approaches, to have your console properly identify function names:

    Foo.prototype.something = function something() {
      // yadda yadda
    };

    Foo.prototype.something = function() {
      // yadda yadda
    };
    Foo.prototype.something.displayName = "Foo.something";

The second option `displayName` was introduced by WebKit and later adopted by
Firebug / Firefox.`displayName` is a bit more code to write out, but allows
arbitrary names, including a namespace or associated object. Either of these 
approaches can help with anonymous functions quite a bit.

Read more on this topic in [Named function expressions demystified][46] by 
[kangax][12].

### Documenting APIs {#documenting-apis}

One of the hardest tasks of software development is documenting things.
Practically everyone hates doing it, yet everybody laments about bad or missing 
documentation of the tools they need to use. There is a wide range of tools that
supposedly help and automate documenting your code:

At one point or another all of these tool won’t fail to disappoint.
JavaScript is a very dynamic language and thus particularly diverse in 
expression. This makes a lot of things extremely difficult for these tools. The 
following list features a couple of reasons why I’ve decided to prepare 
documentation in vanilla HTML, markdown or[DocBoock][47] (if the project is
large enough). jQuery, for example, has the same issues and doesn’t document 
their APIs within their code at all.

1.  Function signatures aren’t the only documentation you need, but most
    tools focus only on them.
   
2.  Example code goes a long way in explaining how something works. Regular API
    docs usually fail to illustrate that with a fair trade-off.
   
3.  API docs usually fail horribly at explaining things *behind the scenes* (
    flow, events, etc
    ).
4.  Documenting methods with multiple signatures is usually a real pain.
5.  Documenting methods using option objects is often not a trivial task.
6.  Generated Methods aren’t easily documented, neither are default callbacks
    .

If you can’t (or don’t) want to adjust your code to fit one of the listed
documentation tools, projects like[Document-Bootstrap][48] might save you some
time setting up your home brew documentation.

Make sure your Documentation is more than just some generated API doc. Your
users will appreciate any examples you provide. Tell them how your software 
flows and which events are involved when doing something. Draw them a map, if it
helps their understanding of whatever it is your software is doing. And above 
all: keep your docs in sync with your code!

#### Self-Explanatory Code {#self-explanatory-code}

Providing good documentation will not keep developers from actually reading
your code — your code is a piece of documentation itself. Whenever the 
documentation doesn’t suffice (and every documentation has its limits), 
developers fall back to reading the actual source to get their questions 
answered. Actually, you are one of them as well. You are most likely reading 
your own code again and again, with weeks, months or even years in between.

You should be writing code that explains itself. Most of the time this is a non
-issue, as it only involves you thinking harder about naming things (functions, 
variables, etc) and sticking to a core concept. If you find yourself writing 
code comments to document how your code does something, you’re most likely 
wasting time — your time, and the reader’s as well. Comment on your code to 
explain *why* you solved the problem this particular way, rather than explaining
*how* you solved the problem. The *how* should become apparent through your
code, so don’t repeat yourself. Note that using comments to mark sections within
your code or to explain general concepts is totally acceptable.

### Conclusion {#conclusion}

*   An API is a contract between you (the provider) and the user (the consumer
    ). Don’t just change things between versions.
   
*   You should invest as much time into the question *How will people use my
    software?* as you have put into *How does my software work internally?* 
*   With a couple of simple tricks you can greatly reduce the developer’s
    efforts (in terms of the lines of code
    ).
*   Handle invalid input as early as possible — throw Errors.
*   Good APIs are flexible, better APIs don’t let you make mistakes.

Continue with [Reusable Code for good or for awesome][49] ([slides][50]), a
Talk by[Jake Archibald][15] on designing APIs. Back in 2007 Joshua Bloch gave
the presentation[How to Design A Good API and Why it Matters][51] at Google
Tech Talks. While his talk did not focus on JavaScript, the basic principles 
that he explained still apply.

Now that you’re up to speed on designing APIs, have a look at 
[Essential JS Design Patterns][52] by [Addy Osmani][53] to learn more about how
to structure your internal code.

*Thanks go out to [@bassistance][54], [@addyosmani][53] and [@hellokahlil][55]
for taking the time to proof this article.*

 [1]: img/Pie-chart.jpg "Time Spent On Creating Vs Time Spent On Using"
 [2]: http://en.wikipedia.org/wiki/Fluent_interface#JavaScript
 [3]: http://martinfowler.com/bliki/FluentInterface.html
 [4]: http://medialize.github.com/URI.js/
 [5]: https://twitter.com/leaverou
 [6]: http://lea.verou.me/chainvas/
 [7]: http://underscorejs.org/#chain
 [8]: http://en.wikipedia.org/wiki/Method_chaining
 [9]: http://en.wikipedia.org/wiki/Command-query_separation
 [10]: http://api.jquery.com/css/

 [11]: http://perfectionkills.com/extending-built-in-native-objects-evil-or-not/
 [12]: https://twitter.com/kangax

 [13]: https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Object/valueOf

 [14]: https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Object/toString
 [15]: https://twitter.com/jaffathecake
 [16]: http://www.slideshare.net/slideshow/embed_code/5426258?startSlide=59
 [17]: img/good-intention.jpg "Good Intentions"
 [18]: http://api.jquery.com/on/

 [19]: https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Functions_and_function_scope/arguments
 [20]: https://developer.mozilla.org/en-US/docs/DOM/event.initMouseEvent

 [21]: http://www.diveintopython.net/power_of_introspection/optional_arguments.html
 [22]: http://wiki.ecmascript.org/doku.php?id=harmony:parameter_default_values
 [23]: http://wiki.ecmascript.org/doku.php?id=harmony:rest_parameters
 [24]: http://api.jquery.com/jquery.ajax/
 [25]: http://api.jquery.com/jQuery.extend/
 [26]: http://underscorejs.org/#extend
 [27]: http://api.prototypejs.org/language/Object/extend/
 [28]: http://api.jquery.com/toggle/
 [29]: img/developing-possibilities.jpg "Developing Possibilities"
 [30]: https://developer.mozilla.org/en-US/docs/DOM/DOM_event_reference
 [31]: http://api.jquery.com/jQuery.cssHooks/
 [32]: http://blog.rodneyrehm.de/archives/11-jQuery-Hooks.html
 [33]: img/duplication.jpg "Duplication"
 [34]: https://twitter.com/rmurphey
 [35]: http://rmurphey.com/blog/2010/07/12/patterns-for-dry-er-javascript/
 [36]: https://twitter.com/mathias

 [37]: http://slideshare.net/mathiasbynens/how-dry-impacts-javascript-performance-faster-javascript-execution-for-the-lazy-developer
 [38]: http://docstore.mik.ua/orelly/webprog/jscript/ch11_02.htm
 [39]: http://docstore.mik.ua/orelly/webprog/jscript/index.htm
 [40]: img/fail-faster.jpg "Fail Fast"
 [41]: http://api.jquery.com/jQuery.isfunction/
 [42]: http://underscorejs.org/#isFunction
 [43]: http://wiki.commonjs.org/wiki/Promises/A
 [44]: http://api.jquery.com/category/deferred-object/
 [45]: http://sitr.us/2012/07/31/promise-pipelines-in-javascript.html
 [46]: http://kangax.github.com/nfe/
 [47]: http://en.wikipedia.org/wiki/DocBook
 [48]: http://gregfranko.com/Document-Bootstrap/
 [49]: http://vimeo.com/35689836

 [50]: http://www.slideshare.net/jaffathecake/reusable-code-for-good-or-for-awesome
 [51]: http://www.youtube.com/watch?v=heh4OeB9A-c
 [52]: http://addyosmani.com/resources/essentialjsdesignpatterns/book/
 [53]: https://twitter.com/addyosmani
 [54]: https://twitter.com/bassistance
 [55]: https://twitter.com/hellokahlil