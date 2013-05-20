# Marionette.Region

Regions provide a consistent way to manage your views and
show / close them in your application. They use a jQuery selector
to show your views in the correct place. They also call extra
methods on your views to facilitate additional functionality.

Регионы обеспечивают последовательное управление представлениями в приложении и предоставляют для этого специальные методы "show" и "close".

## Documentation Index

* [Defining An Application Region](#defining-an-application-region)
* [Initialize A Region With An `el`](#initialize-a-region-with-an-el)
* [Basic Use](#basic-use)
* [`reset` A Region](#reset-a-region)
* [Set How View's `el` Is Attached](#set-how-views-el-is-attached)
* [Attach Existing View](#attach-existing-view)
  * [Set `currentView` On Initialization](#set-currentview-on-initialization)
  * [Call `attachView` On Region](#call-attachview-on-region)
* [Region Events And Callbacks](#region-events-and-callbacks)
  * [View Callbacks And Events For Regions](#view-callbacks-and-events-for-regions)
* [Custom Region Types](#custom-region-types)
  * [Attaching Custom Region Types](#attaching-custom-region-types)
  * [Instantiate Your Own Region](#instantiate-your-own-region)

## Defining An Application Region

## Задание региона в приложении

Regions can be added to the application by calling the `addRegions` method on
your application instance. This method expects a single hash parameter, with
named regions and either jQuery selectors or `Region` objects. You may
call this method as many times as you like, and it will continue adding regions
to the app. 

Регионы могут быть добавлены в приложение с помощью [метода `addRegions`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.application.js#L51-L57), вызванного на экземпляре приложения.
Данный метод принимает единственный параметр - хеш соответствий: именам регионов в нем соответствуют jQuery-селекторы.
Можно вызывать этот метод столько раз, сколько потребуется, в любой момент работы приложения.

```js
MyApp.addRegions({
  mainRegion: "#main-content",
  navigationRegion: "#navigation"
});
```

As soon as you call `addRegions`, your regions are available on your
app object. In the above, example `MyApp.mainRegion` and `MyApp.navigationRegion`
would be available for use immediately.

Как только мы вызовем `addRegions`, регионы станут доступны в объекте приложения для дальнейшей работы.
В примере выше определены два региона: `MyApp.mainRegion` и `MyApp.navigationRegion`.

If you specify the same region name twice, the last one in wins.

Для одного и того же региона, определенного дважды, будет справедливо последнее определение.

## Initialize A Region With An `el`

## Инициализация региона с помощью `el`

You can specify an `el` for the region to manage at the time
that the region is instantiated:

Вы можете задать в объекте, переданном в качестве параметра в конструктор региона, атрибут `el`:

```js
var mgr = new Backbone.Marionette.Region({
  el: "#someElement"
});
```

## Basic Use

## Базовое использование

Once a region has been defined, you can call the `show`
and `close` methods on it to render and display a view, and then
to close that view:

Как только регион был определен, можно вызывать его методы [`show`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L112-L133) и [`close`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L153-L166) для отображения в регионе представления:

```js
var myView = new MyView();

// внедряем и показываем представление
MyApp.mainRegion.show(myView);

// текущее представление для данного региона будет закрыто
MyApp.mainRegion.close();
```

If you replace the current view with a new view by calling `show`,
it will automatically close the previous view.

[Предыдущее представление, определенное в регионе, будет автоматически закрыто](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L121-L127) с показом нового представления посредством вызова метода `show`:

```js
// показываем первое представление
var myView = new MyView();
MyApp.mainRegion.show(myView);

// заменяем его на другое - метод `close` выполнится автоматически
var anotherView = new AnotherView();
MyApp.mainRegion.show(anotherView);
```

## `reset` A Region

## Сброс региона (метод [`reset`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L176-L184))

A region can be `reset` at any time. This will close any existing view
that is being displayed, and delete the cached `el`. The next time the
region is used to show a view, the region's `el` will be queried from
the DOM.

Регион может быть деинсталлирован в любой момент. Эта операция закроет всякое прежде отображенное в нем представление, и удалит закешированный `el`.
В следующий раз, когда регион будет использован для показа представления, `el` будет запрошен из DOM.


```js
myRegion.reset();
```

This is useful for scenarios where a region is re-used across view
instances, or in unit testing.

Это может оказаться полезным в сценариях с повторным использованием региона различными экземплярами приложения, или при юнит-тестировании.

## Set How View's `el` Is Attached

## Установка способа прикрепления `el` к DOM

If you need to change how the view is attached to the DOM when
showing a view via a region, override the `open` method of the
region. This method receives one parameter - the view to show.

Если необходимо изменить способ, которым представление встраивается в DOM при показе его в регионе, нужно переопределить метод `open` региона.
Метод принимает единственный переметр - экземпляр представления, который должен быть показан.

The default implementation of `open` is:

По умолчанию, реализация метода [`open`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L147-L151) следующая:

```js
Marionette.Region.prototype.open = function(view){
  this.$el.empty().append(view.el);
}
```

This will replace the contents of the region with the view's
`el` / content. However, `open`'s behaviour can be overriden 
to facilitate transition effects and more.

В результате содержимое региона (оно находится в $el) заменяется содержимым представления. Но, конечно же, такое поведение может быть переопределено,
например, для реализации различных переходных эффектов и т.д. и т.п. 

```js
Marionette.Region.prototype.open = function(view){
  this.$el.hide();
  this.$el.html(view.el);
  this.$el.slideDown("fast");
}
```

This example will cause a view to slide down from the top
of the region, instead of just appearing in place.

В данном примере представление будет "вдвигаться" в регион сверху.

## Attach Existing View

## Присоединение существующего представления

There are some scenarios where it's desirable to attach an existing
view to a region , without rendering or showing the view, and
without replacing the HTML content of the region. For example, SEO and
accessibiliy often need HTML to be generated by the server, and progressive
enhancement of the HTML.

Бывают сценарии, где необходимо добавить экземпляр представления к региону, без его отображения, и без замены HTML-контента региона.

There are two ways to accomplish this: 

Есть два способа достичь этого:

* set the `currentView` in the region's constructor
* call `attachView` on the region instance

* определить `currentView` в конструкторе региона
* вызвать `attachView` для экземпляра региона

### Set `currentView` On Initialization

### Установка `currentView` при инициализации региона

```js
var myView = new MyView({
  el: $("#existing-view-stuff")
});

var region = new Backbone.Marionette.Region({
  el: "#content",
  currentView: myView
});
```

### Call `attachView` On Region 

### Вызов `attachView` для экземпляра региона

```js
MyApp.addRegions({
  someRegion: "#content"
});

var myView = new MyView({
  el: $("#existing-view-stuff")
});

MyApp.someRegion.attachView(myView);
```

## Region Events And Callbacks

## События и коллбэки

A region will raise a few events when showing and
closing views:

Регион может триггерить следующие события при показе и закрытии представлений:

* "show" / `onShow` - Called on the view instance when the view has been rendered and displayed.
* "show" / `onShow` - Called on the region instance when the view has been rendered and displayed.
* "close" / `onClose` - Called when the view has been closed.

* "show" (и метод `onShow`) - [Вызывается на экземпляре представления](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L129), когда представление [отрендерено и показано](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L123-L126).
* "show" (и метод `onShow`) - [Вызывается на экземпляре региона](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L130), когда представление [отрендерено и показано](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L123-L126).
* "close" (и метод `onClose`) - [Вызывается](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L163), когда представление [закрыто](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L160-L161).

These events can be used to run code when your region 
opens and closes views.

Таким образом, у нас есть возможность связывать с данными событиями вызов соответствующего кода.



```js
MyApp.mainRegion.on("show", function(view){
  // что-то делаем с представлением или проводим дополнительные операции над регионом посредством указания ключевого слова `this`
});

MyApp.mainRegion.on("close", function(view){
  // . . . . .
});

MyRegion = Backbone.Marionette.Region.extend({
  // ...

  onShow: function(view){
    // представление `view` уже показано, можно что-то делать
  }
});

MyView = Marionette.ItemView.extend({
  onShow: function(){
    // вызывается, когда представление показано
  }
});
```

### View Callbacks And Events For Regions

### Функции обратного вызова и события представления для регионов

The region will call an `onShow` method on the view
that was displayed. It will also trigger a "show" event
from the view:

Когда представление отобразится в регионе, регион вызовет `onShow` метод на этом представлении.
Заодно в представлении породится событие "show":

```js
MyView = Backbone.View.extend({
  onShow: function(){
    // представление показано, делаем что-то
  }
});

view = new MyView();

view.on("show", function(){
  // представление показано, делаем что-то
});

MyApp.mainRegion.show(view);
```

## Custom Region Types

## Пользовательские типы регионов

You can define a custom region by extending from
`Region`. This allows you to create new functionality,
or provide a base set of functionality for your app.

Можно определить пользовательский регион-объект, произведя его от `Region`, для добавления новой функциональности.

### Attaching Custom Region Types

### Присоединение региона пользовательского типа

Once you define a region type, you can attach the
new region type by specifying the region type as the
value. In this case, `addRegions` expects the constructor itself, not an instance.

Раз уж вы определили новый регион-объект, вы можете использовать регион данного типа в приложении:

```js
var FooterRegion = Backbone.Marionette.Region.extend({
  el: "#footer"
});

MyApp.addRegions({
  footerRegion: FooterRegion
});
```

You can also specify a selector for the region by using
an object literal for the configuration.

Вы можете также указать селектор для региона, определив для конфигурации объектный литерал: 

```js
var FooterRegion = Backbone.Marionette.Region.extend({
  el: "#footer"
});

MyApp.addRegions({
  footerRegion: {
    selector: "#footer",
    regionType: FooterRegion
  }
});
```

Note that a region must have an element to attach itself to. If you
do not specify a selector when attaching the region instance to your
Application or Layout, the region must provide an `el` either in its
definition or constructor options.

Заметим, что регион, чтобы он появился на странице, [должен быть связан с DOM-элементом](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L119). Если вы не определяете селектор в экземпляре региона,
при добавлении его к экземпляру Application или Layout посредством "addRegions", то внутри самого экземпляра региона должно быть указано значение для атрибута `el` 
при создании его через вызов конструктора. 

### Instantiate Your Own Region 

### Создание экземпляра для объекта региона, определенного пользователем

There may be times when you want to add a region to your
application after your app is up and running. To do this, you'll
need to extend from `Region` as shown above and then use
that constructor function on your own:

Бывают случаи, когда нужно добавить регион в приложение после того, как оно сконфигурировано и запущено.
Тогда необходимо наследовать новый объект от объекта `Region`, и затем использовать получившийся в результате конструктор:

```js
var SomeRegion = Backbone.Marionette.Region.extend({
  el: "#some-div",

  initialize: function(options){
    // код инициации
  }
});

MyApp.someRegion = new SomeRegion();

MyApp.someRegion.show(someView);
```

You can optionally add an `initialize` function to your Region
definition as shown in this example. It receives the `options`
that were passed to the constructor of the Region, similar to
a Backbone.View.

Как показано в этом примере, можно добавить [опциональную функцию `initialize`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L18-L21) в Region-объект, которая получает в качестве аргумента параметр `options`, [который приходит в нее из конструктора](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L7), аналогично Backbone.View.


