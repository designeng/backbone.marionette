# Marionette.Application

The `Backbone.Marionette.Application` object is the hub of your composite 
application. It organizes, initializes and coordinates the various pieces of your
app. It also provides a starting point for you to call into, from your HTML 
script block or from your JavaScript files directly if you prefer to go that 
route.

Объект `Backbone.Marionette.Application` является композиционным центром вашего приложения.
Он организует, инициализирует и координирует различные части приложения. Он также воплощает в себе
отправную точку для прямых вызовов кода, разбросанного по javascript-файлам, если вы предпочитаете такой стиль программирования.

The `Application` is meant to be instantiated directly, although you can extend
it to add your own functionality.

Объект `Application` предполагает прямое создание экземпляра, хотя вы можете расширить его своими методами, чтобы создать собственный профиль функциональности.

```js
MyApp = new Backbone.Marionette.Application();
```

## Documentation Index

* [Adding Initializers](#adding-initializers)
* [Application Event](#application-event)
* [Starting An Application](#starting-an-application)
* [app.vent: Event Aggregator](#appvent-event-aggregator)
* [Regions And The Application Object](#regions-and-the-application-object)
  * [jQuery Selector](#jquery-selector)
  * [Custom Region Type](#custom-region-type)
  * [Custom Region Type And Selector](#custom-region-type-and-selector)
  * [Removing Regions](#removing-regions)

## Adding Initializers

## Добавление инициализаторов

Your application needs to do useful things, like displaying content in your
regions, starting up your routers, and more. To accomplish these tasks and
ensure that your `Application` is fully configured, you can add initializer
callbacks to the application.

Приложение предполагает выполнение ряда полезных действий,
таких как отображение контента в регионах, обеспечения маршрутизации и т.д.  Для достижения этой цели, а также для того, чтобы убедиться, что объект `Application` настроен как следует, вы можете добавить инициализирующие колбэки.


```js
MyApp.addInitializer(function(options){
  // do useful stuff here
  var myView = new MyView({
    model: options.someModel
  });
  MyApp.mainRegion.show(myView);
});

MyApp.addInitializer(function(options){
  new MyAppRouter();
  Backbone.history.start();
});
```

These callbacks will be executed when you start your application,
and are bound to the application object as the context for
the callback. In other words, `this` is the `MyApp` object, inside
of the initializer function.

Данные колбэки будут выполнены как только приложение будет запущено, и связаны с объектом приложения как контекстом для их выполнения.
Другими словами, ключевое слово `this` внутри инициализирующей функции является не чем иным, как объектом `MyApp`.

The `options` parameters is passed from the `start` method (see below).

Параметры `options` попадают в них из метода `start` (рассматривается ниже).

Initializer callbacks are guaranteed to run, no matter when you
add them to the app object. If you add them before the app is
started, they will run when the `start` method is called. If you
add them after the app is started, they will run immediately.

Инициализирующие колбэки гарантированно выполнятся, вне зависимости от того, в какой момент времени вы добавите их в объект приложения.
Если вы добавите их перед тем, как запустить приложение, они выполнятся, когда метод `start` будет вызван.
Если добавить их после запуска, они будут выполнены немедленно.



## Application Event

## События объекта приложения

The `Application` object raises a few events during its lifecycle, using the
[Marionette.triggerMethod](./marionette.functions.md) function. These events
can be used to do additional processing of your application. For example, you
may want to pre-process some data just before initialization happens. Or you may
want to wait until your entire application is initialized to start the
`Backbone.history`.

Объект `Application` порождает некоторые события в течение своего жизненного цикла, при помощи [Marionette.triggerMethod](./marionette.functions.md).
Эти события могут быть использованы для дополнительного процессинга вашего приложения. Например, вы можете предварительно обработать какие-то данные
перед тем, как произойдет инициализация объекта приложения. Или, например, вы хотите запустить `Backbone.history` только после того, как приложение будет до конца проинициализировано.

The events that are currently triggered, are:

Вот эти события:

* **"initialize:before" / `onInitializeBefore`**: fired just before the initializers kick off
* **"initialize:after" / `onInitializeAfter`**: fires just after the initializers have finished
* **"start" / `onStart`**: fires after all initializers and after the initializer events

* **"initialize:before" / `onInitializeBefore`**: порождается до того, как инициализаторы приступят к действию
* **"initialize:after" / `onInitializeAfter`**: порождается сразу после того, как инициализаторы завершат работу
* **"start" / `onStart`**: порождается сразу после того, как инициализаторы завершат работу и будут порождены все инициализирующие события

```js
MyApp.on("initialize:before", function(options){
  options.moreData = "Yo dawg, I heard you like options so I put some options in your options!"
});

MyApp.on("initialize:after", function(options){
  if (Backbone.history){
    Backbone.history.start();
  }
});
```

The `options` parameter is passed through the `start` method of the application
object (see below).

Параметр `options` попадет в колбэки из метода `start` объекта приложения (см. ниже).

## Starting An Application

## Запуск приложения

Once you have your application configured, you can kick everything off by 
calling: `MyApp.start(options)`.

Как только ваше приложение должным образом сконфигурировано, вы можете запустить его простым
вызовом `MyApp.start(options)`

This function takes a single optional parameter. This parameter will be passed
to each of your initializer functions, as well as the initialize events. This
allows you to provide extra configuration for various parts of your app, at
initialization/start of the app, instead of just at definition.

Данный метод принимает единственный опциональный параметр. Этот параметр будет передан во все инициализирующие колбэки,
а также во все инициализирующие события. Благодаря этому обеспечивается дополнительная конфигурация для различных частей приложения.

```js
var options = {
  something: "some value",
  another: "#some-selector"
};

MyApp.start(options);
```

## app.vent: Event Aggregator

## app.vent: агрегатор событий

Every application instance comes with an instance of `Backbone.Wreqr.EventAggregator` 
called `app.vent`.

Каждый экземпляр приложения приходит в комплекте с экземпляром `Backbone.Wreqr.EventAggregator`, так называемым `app.vent`.

```js
MyApp = new Backbone.Marionette.Application();

MyApp.vent.on("foo", function(){
  alert("bar");
});

MyApp.vent.trigger("foo"); // => alert box "bar"
```

See the [`Backbone.Wreqr`](https://github.com/marionettejs/backbone.wreqr) documentation for more details.

За дальнейшей информацией по `Backbone.Wreqr` обратитесь к [документации](https://github.com/marionettejs/backbone.wreqr).

## Regions And The Application Object

## Регионы и объект приложения

Marionette's `Region` objects can be directly added to an application by
calling the `addRegions` method.

Объекты типа `Region` могут быть непосредственно добавлены в приложение посредствои вызова метода `addRegions`.

There are three syntax forms for adding a region to an application object.

Предусмотрены три синтаксические формы для добавления региона в объект приложения.


### jQuery Selector

### jQuery-селектор

The first is to specify a jQuery selector as the value of the region
definition. This will create an instance of a Marionette.Region directly,
and assign it to the selector:

Первый предполагает задание jQuery-селектора в качестве значения дифиниции региона.
Так будет непосредственно создан экземпляр Marionette.Region, и назначен селектору:

```js
MyApp.addRegions({
  someRegion: "#some-div",
  anotherRegion: "#another-div"
});
```

### Custom Region Type

### Пользовательский тип региона

The second is to specify a custom region type, where the region type has
already specified a selector:

Согласно второму, определяется пользовательский тип региона, в котором селектор уже зарезервирован:

```js
MyCustomRegion = Marionette.Region.extend({
  el: "#foo"
});

MyApp.addRegions({
  someRegion: MyCustomRegion
});
```

### Custom Region Type And Selector

### Пользовательский тип региона и селектор

The third method is to specify a custom region type, and a jQuery selector
for this region instance, using an object literal:

Третий подразумевает одновременное задание пользовательского типа региона и jQuery селоктора для экземпляра региона - используется объектный литерал: 

```js
MyCustomRegion = Marionette.Region.extend({});

MyApp.addRegions({

  someRegion: {
    selector: "#foo",
    regionType: MyCustomRegion
  },

  anotherRegion: {
    selector: "#bar",
    regionType: MyCustomRegion
  }

});
```

### Removing Regions

### Удаление регионов

Regions can also be removed with the `removeRegion` method, passing in 
the name of the region to remove as a string value:

Регионы могут также быть удалены с помощью метода `removeRegion`, которому передается в качестве строкового значения имя региона.
 
```js
MyApp.removeRegion('someRegion');
```

Removing a region will properly close it before removing it from the
application object.

Удаление региона повлечет его корректное закрытие.

For more information on regions, see [the region documentation](./marionette.region.md)
