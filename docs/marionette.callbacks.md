# Marionette.Callbacks

The `Callbacks` object assists in managing a collection of callback
methods, and executing them, in an async-safe manner.

Объект `Callbacks` помогает в управлении коллекцией колбэков и заведует их выполнением в асинхронно-безопасной манере.

There are only two methods: 

Имеет всего 2 метода:

* `add`
* `run`

The `add` method adds a new callback to be executed later. 

Метод [`add`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.callbacks.js#L14-L24) добавляет новый коллбэк, который выполнится позднее.

The `run` method executes all current callbacks in, using the
specified context for each of the callbacks, and supplying the
provided options to the callbacks.

Метод [`run`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.callbacks.js#L26-L31)запускает на выполнение все зарегистрированные коллбэки, используя для них переданный в качестве второго аргумента контекст и переданные первым аргументом настройки. 

## Documentation Index

* [Basic Usage](#basic-usage)
* [Specify Context Per-Callback](#specify-context-per-callback)
* [Advanced / Async Use](#advanced--async-use)

## Basic Usage

## Базовое использование

```js
var callbacks = new Backbone.Marionette.Callbacks();

callbacks.add(function(options){
  alert("I'm a callback with " + options.value + "!");
});

callbacks.run({value: "options"}, someContext);
```

This example will display an alert box that says "I'm a callback
with options!". The executing context for each of the callback
methods has been set to the `someContext` object, which is an optional
parameter that can be any valid JavaScript object.

Пример выведет предупреждение с текстом "I'm a callback
with options!". Контекст выполнения для обоих коллбэков установлен как `someContext` объект,
это опциональный параметр, являющийся валидным JavaScript-объектом.

## Specify Context Per-Callback

## Определение контекста выполнения для каждого коллбэка

You can optionally specify the context that you want each callback to be
executed with, when adding a callback:

Можно также задать контекст выполнения для каждого коллбэка, передавая его вторым параметром:

```js
var callbacks = new Backbone.Marionette.Callbacks();

callbacks.add(function(options){
  alert("I'm a callback with " + options.value + "!");
}, myContext);
 
callbacks.run({value: "options"}, someContext);
```

Поскольку контекст выполнения (`this`) в данном примере изначально установлен как myContext, попытка задать `someContext` в этом качестве будет [проигнорирована](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.callbacks.js#L21).

## Advanced / Async Use

## Продвинутое / асинхронное использование

The `Callbacks` executes each callback in an async-friendly 
manner, and can be used to facilitate async callbacks. 
The `Marionette.Application` object uses `Callbacks`
to manage initializers (see above). 

`Callbacks` запускает на выполнение каждый коллбэк в асинхронно-дружественной манере.
Объект `Marionette.Application` [использует](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.application.js#L9) `Callbacks`, чтобы обеспечить реализацию [инициализаторов](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.application.js#L33-L38).

It can also be used to guarantee callback execution in an event
driven scenario, much like the application initializers.

В сценариях, базирующихся на событийной архитектуре, механизм коллбэков, равно как и механизм инициализаторов, может быть задействован для гарантированного выполнения зарегистрированных функций.

