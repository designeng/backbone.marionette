# Marionette.Application.module

Marionette allows you to define a module within your application,
including sub-modules hanging from that module. This is useful for creating 
modular, encapsulated applications that are split apart in to multiple 
files.

Marionette разрешает вам регистрировать собственные модули внутри приложения,
включая подмодули, зависящие от определенного модуля. Это полезно для создания
сложных приложений, где каждый модуль инкапсулирует свои собственные свойства и методы,
а функциональность, благодаря модульной структуре приложения, вынесена в отдельные файлы. 

Marionette's modules allow you to have unlimited sub-modules hanging off of
your application, and serve as an event aggregator in themselves.

Модули в Marionette могут иметь неограниченное количество подмодулей, зависящих от 
вашего приложения, и являющихся самими по себе агрегаторами событий.

## Documentation Index

* [Основы использования](#basic-usage)
* [Запуск и остановка модулей](#starting-and-stopping-modules)
  * [Запуск модулей](#starting-modules)
  * [Рассылка событий](#start-events)
  * [Предохранение от самозапуска модулей](#preventing-auto-start-of-modules)
  * [Запуск подмодулей их владельцами](#starting-sub-modules-with-parent)
  * [Остановка модулей](#stopping-modules)
  * [Остановка рассылки событий](#stop-events)
* [Определение подмодулей при помощи .(dot) нотации](#defining-sub-modules-with--notation)
* [Module Definitions](#module-definitions)
  * [Module Initializers](#module-initializers)
  * [Module Finalizers](#module-finalizers)
* [The Module's `this` Argument](#the-modules-this-argument)
* [Аргументы, определенные пользователем](#custom-arguments)
* [Разбиение описания модуля на отдельные части](#splitting-a-module-definition-apart)

## Основы использования

A module is defined directly from an Application object as the specified
name:
Модуль определяется непосредственно из объекта Приложения - для этого модулю назначается  имя:

```js
var MyApp = new Backbone.Marionette.Application();

var myModule = MyApp.module("MyModule");

MyApp.MyModule; // => a new Marionette.Application object

myModule === MyApp.MyModule; // => true
```

If you specify the same module name more than once, the
first instance of the module will be retained and a new
instance will not be created.

Если вы определите модуль с одним и тем же именем более одного раза,
первый экземпляр сохранится в памяти, а второй просто не будет создан.

## Запуск и остановка модулей

Modules can be started and stopped independently of the application and
of each other. This allows them to be loaded asynchronously, and also allows
them to be shut down when they are no longer needed. This also facilitates
easier unit testing of modules in isolation as you can start only the
module that you need in your tests.

Модули могут быть запущены/остановлены независимым образом: инициатором запуска/остановки 
может являться как само приложение, так и другой модуль. Это позволяет им независимо и асинхронно 
загружаться в приложение, и также дает возможность выгрузить их в один прекрасный момент, когда
они больше не используются. Такой подход значительно облегчает жизнь тестировщику: каждый модуль 
может быть запущен и протестирован изолированно.

### Запуск модулей

Запуск модуля может быть произведен одним из двух способов:

1. Автоматически из внешнего модуля (или из Приложения), когда будет вызван метод `.start()` внешнего модуля (Приложения)
2. Вручную, вызывая метод `.start()` самого модуля.

В этом примере модуль будет запущен автоматически, когда на объекте родительского приложения
выполнится его метод `start`:

```js
MyApp = new Backbone.Marionette.Application();

MyApp.module("Foo", function(){

  // здесь идет код модуля

});

MyApp.start();
```

Note that modules loaded and defined after the `app.start()` call will still
be started automatically.
Следует заметить, что модули, загруженные и определенные даже после вызова `app.start()`
все равно будут запускаться автоматически.

### Start Events

When starting a module, a "before:start" event will be triggered prior
to any of the initializers being run. A "start" event will then be 
triggered after they have been run.
Во время запуска модуля произойдут два события: "before:start" и "start".
До всяких действий по инициализации будет вброшено событие "before:start".
Само событие "start" будет запущено только после инициализация модуля.

```js
var mod = MyApp.module("MyMod");

mod.on("before:start", function(){
  // делаем что-то до старта модуля
});

mod.on("start", function(){
  // и что-то после того, как модуль запущен
});
```

#### Passing Data to Start Events

`.start` takes a single `options` parameter that will be passed to start events and their equivalent methods (`onStart` and `onBeforeStart`.) 
Метод `.start` принимает один-единственный параметр -  `options`, который 
будет передан start-событиям и их функциональным эквивалентам (`onStart` and `onBeforeStart`.) 

```js
var mod = MyApp.module("MyMod");

mod.on("before:start", function(options){
  // делаем что-то до старта модуля
});

mod.on("start", function(options){
  // и что-то после того, как модуль запущен
});

var options = {
 // какие-то данные
};
mod.start(options);
```

### Preventing Auto-Start Of Modules
### Предотвращение автозапуска модулей

If you wish to manually start a module instead of having the application
start it, you can tell the module definition not to start with the parent:

Если вы хотите запускать модуль вручную, вместо того, чтобы предоставить этот процесс приложению,
вы можете явным образом указать в параметрах определения модуля:

```js
var fooModule = MyApp.module("Foo", function(){

  // не запускать родителем
  this.startWithParent = false;

  // ... код модуля идет здесь
});

// запуск приложения
MyApp.start();

// и позже, запуск модуля
fooModule.start();
```

Note the use of an object literal instead of just a function to define
the module, and the presence of the `startWithParent` attribute, to tell it
not to start with the application. Then to start the module, the module's 
`start` method is manually called.

Обратите внимание на использование объектного литерала вместо просто функции для определения модуля,
и присутствие аттрибута `startWithParent`, говорящего модулю, что он не должен запускаться вместе с запуском приложения. Теперь, чтобы запустить модуль, метод модуля `start` долен быть вызван вручную.

You can also grab a reference to the module at a later point in time, to
start it:

Вы также можете получить ссылку на модуль в какой-то более поздний момент времени,
чтобы запустить его:

```js
MyApp.module("Foo", function(){
  this.startWithParent = false;
});

// получаем ссылку - и запускаем модуль 
MyApp.module("Foo").start();
```

#### Specifying `startWithParent: false` setting as an object literal
#### Задание `startWithParent: false` в объектном литерале 

There is a second way of specifying `startWithParent` in a `.module`
call, using an object literal:

Второй способ задания директивы `startWithParent` в методе `.module` - 
это использование объектного литерала:

```js
var fooModule = MyApp.module("Foo", { startWithParent: false });
```

This is most useful when defining a module across multiple files and
using a single definition to specify the `startWithParent` option.

Это наиболее удобный способ, когда у нас есть громоздкий модуль, для описания которого требуется
не один файл, а несколько, и мы используем отдельное описане для задания директивы `startWithParent`.

If you wish to combine the `startWithparent` object literal
with a module definition, you can include a `define` attribute on
the object literal, set to the module function:

Если вы хотите объединить объектный литерал, описывающий `startWithParent`, с дифиницией модуля,
вы можете включить аттрибут `define` в объектный литерал, переданный в функцию создания модуля:

```js
var fooModule = MyApp.module("Foo", {
  startWithParent: false,

  define: function(){
    // здесь идет код модуля
  }
});
```

### Starting Sub-Modules With Parent
### Запуск подмодулей родителем

Starting of sub-modules is done in a depth-first hierarchy traversal. 
That is, a hierarchy of `Foo.Bar.Baz` will start `Baz` first, then `Bar`,
and finally `Foo.

Запуск подмодулей совершается в порядке, обратном порядку иерархии - то-есть из глубины. Например, в иерархии `Foo.Bar.Baz` сначала будет запущен `Baz`-модуль, потом `Bar`, и, наконец, `Foo`.



Submodules default to starting with their parent module.
По умолчанию, запуск подмодуля производится с запуском его родителя.

```js
MyApp.module("Foo", function(){...});
MyApp.module("Foo.Bar", function(){...});

MyApp.start();
```

In this example, the "Foo.Bar" module will be started with the call to
`MyApp.start()` because the parent module, "Foo" is set to start
with the app.

В этом примере модуль "Foo.Bar" будет запущен, когда метод `MyApp.start()`
будет вызван, поскольку его модуль-родитель "Foo" обязан запуститься одновременно со стартом приложения.

A sub-module can override this behavior by setting it's `startWithParent`
to false. This prevents it from being started by the parent's `start` call.

Но, как уже должно быть ясно, это поведение в подмодуле может быть переопределено, если значение директивы `startWithParent` задано как false. Это предохранит его от запуска в момент вызова метода `start` его родителя.

```js
MyApp.module("Foo", function(){...});

MyApp.module("Foo.Bar", function(){
  this.startWithParent = false;
})

MyApp.start(); 
```

Now the module "Foo" will be started, but the sub-module "Foo.Bar" will
not be started.

Теперь модуль "Foo" будет запущен, а подмодуль "Foo.Bar" - нет.

A sub-module can still be started manually, with this configuration:
Подмодуль, тем не менее, может быть запущен вручную:

```js
MyApp.module("Foo.Bar").start();
```

### Stopping Modules
### Выключение модулей

A module can be stopped, or shut down, to clear memory and resources when
the module is no longer needed. Like starting of modules, stopping is done
in a depth-first hierarchy traversal. That is, a hierarchy of modules like
`Foo.Bar.Baz` will stop `Baz` first, then `Bar`, and finally `Foo`.

Модуль может быть остановлен, или "выключен", чтобы очистить занимаемые ресурсы,
когда модуль больше не нужен. Подобно запуску, остановка происходит в порядке обратном
порядку иерархии, "из глубины". Так, в иерархии модулей `Foo.Bar.Baz` 
сначала будет остановлен `Baz`-модуль, потом `Bar`, и, наконец, `Foo`.

To stop a module and it's children, call the `stop()` method of a module.

Чтобы выключить модуль и все его подмодули, достаточно вызвать метод модуля `stop()`.

```js
MyApp.module("Foo").stop();
```

Modules are not automatically stopped by the application. If you wish to 
stop one, you must call the `stop` method on it. The exception to this is
that stopping a parent module will stop all of it's sub-modules.

Модули не выключаются автоматически Приложением. Если вы хотите остановить какой-либо модуль,
вам нужно явно вызвать его метод `stop`. Исключение составляет то обстоятельство, что с выключением
внешнего модуля будут выключены и все его подмодули.

```js
MyApp.module("Foo.Bar.Baz");

MyApp.module("Foo").stop();
```

This call to `stop` causes the `Bar` and `Baz` modules to both be stopped
as they are sub-modules of `Foo`. For more information on defining
sub-modules, see the section "Defining Sub-Modules With . Notation".

Вызов `stop` в данном случае повлечет выключение `Bar` и `Baz`, поскольку они
подмодули `Foo`. (Для детальной информации по определению подмодулей см. раздел
"Определение подмодулей при помощи .(dot) нотации"). 

### События при остановке модуля

When stopping a module, a "before:stop" event will be triggered prior
to any of the finalizers being run. A "stop" event will then be triggered
after they have been run.

Когда модуль останавливается, прежде чем какие-либо заключительные действия (finalizers) будут произведены,
разошлется специальное событие "before:stop". И, по аналогии со "start"-событием, событие "stop" будет разослано, после того, как сработают заключительные действия (finalizers).

```js
var mod = MyApp.module("MyMod");

mod.on("before:stop", function(){
  // do stuff before the module is stopped
});

mod.on("stop", function(){
  // do stuff after the module has been stopped
});
```

## Defining Sub-Modules With . Notation

Sub-modules or child modules can be defined as a hierarchy of modules and 
sub-modules all at once:

```js
MyApp.module("Parent.Child.GrandChild");

MyApp.Parent; // => a valid module object
MyApp.Parent.Child; // => a valid module object
MyApp.Parent.Child.GrandChild; // => a valid module object
```

When defining sub-modules using the dot-notation, the 
parent modules do not need to exist. They will be created
for you if they don't exist. If they do exist, though, the
existing module will be used instead of creating a new one.

## Module Definitions

You can specify a callback function to provide a definition
for the module. Module definitions are invoked immediately
on calling `module` method. 

The module definition callback will receive 6 parameters:

* The module itself
* The Parent module, or Application object that `.module` was called from
* Backbone
* Backbone.Marionette
* jQuery
* Underscore
* Any custom arguments

You can add functions and data directly to your module to make
them publicly accessible. You can also add private functions
and data by using locally scoped variables.

```js
MyApp.module("MyModule", function(MyModule, MyApp, Backbone, Marionette, $, _){

  // Приватные свойства и методы
  // --------------------------

  var myData = "this is private data";
 
  var myFunction = function(){
    console.log(myData);
  }


  // Публичные свойства и методы
  // -------------------------

  MyModule.someData = "public data";

  MyModule.someFunction = function(){
    console.log(MyModule.someData);
  }
});

console.log(MyApp.MyModule.someData); //=> публичное свойство
MyApp.MyModule.someFunction(); //=> публичный метод
```

### Module Initializers

Modules have initializers, similarly to `Application` objects. A module's
initializers are run when the module is started.

```js
MyApp.module("Foo", function(Foo){

  Foo.addInitializer(function(){
    // initialize and start the module's running code, here.
  });

});
```

Any way of starting this module will cause it's initializers to run. You
can have as many initializers for a module as you wish.

### Module Finalizers
### "Завершители" модуля (module finalizers)

Modules also have finalizers that are run when a module is stopped.

Модуль также включает в свой состав "завершители" - т.е. функции, которые будут вызваны,
когда модуль останавливается.

```js
MyApp.module("Foo", function(Foo){

  Foo.addFinalizer(function(){
    // tear down, shut down and clean up the module, here
  });

});
```

Calling the `stop` method on the module will run all that module's 
finalizers. A module can have as many finalizers as you wish.

Вызов метода `stop` на конкретном модуле приведет к вызову всех "завершителей" 
данного модуля. Модуль может иметь сколько угодно "завершителей".

## The Module's `this` Argument

The module's `this` argument is set to the module itself.

Аргумент модуля `this` указывает на сам этот модуль.

```js
MyApp.module("Foo", function(Foo){
  this === Foo; //=> true
});
```

## Custom Arguments
## Аргументы, определенные пользователем

You can provide any number of custom arguments to your module, after the
module definition function. This will allow you to import 3rd party
libraries, and other resources that you want to have locally scoped to
your module.

Вы можете передать модулю любое количество пользовательских аргументов,
сделать это при необходимости можно после определяющей модуль функции, как указано в примере:


```js
MyApp.module("MyModule", function(MyModule, MyApp, Backbone, Marionette, $, _, Lib1, Lib2, LibEtc){

  // Lib1 === LibraryNumber1;
  // Lib2 === LibraryNumber2;
  // LibEtc === LibraryNumberEtc;

}, LibraryNumber1, LibraryNumber2, LibraryNumberEtc);
```

## Splitting A Module Definition Apart
## Разбиение описания модуля на отдельные части

Sometimes a module gets to be too long for a single file. In
this case, you can split a module definition across multiple
files:

Иногда модуль становится слишком большим, и для его описания становится удобнее разбить
модуль на отдельные фрагменты, которые представлены в ряде файлов: 

```js
MyApp.module("MyModule", function(MyModule){
  MyModule.definition1 = true;
});

MyApp.module("MyModule", function(MyModule){
  MyModule.definition2 = true;
});

MyApp.MyModule.definition1; //=> true
MyApp.MyModule.definition2; //=> true
```
