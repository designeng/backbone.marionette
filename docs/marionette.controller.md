# Marionette.Controller

A multi-purpose object to use as a controller for
modules and routers, and as a mediator for workflow
and coordination of other objects, views, and more.

Многоцелевой объект для использования в качестве контроллера для модулей и роутеров, а также в качестве медиатора для рабочего процесса приложения
и координации остальных объектов, view и т.д.

## Documentation Index

* [Basic Use](#basic-use)
* [Closing A Controller](#closing-a-controller)
* [On The Name 'Controller'](#on-the-name-controller)

## Basic Use

## Базовое использование

A `Marionette.Controller` can be extended, like other
Backbone and Marionette objects. It supports the standard
`initialize` method, has a built-in `EventBinder`, and
can trigger events, itself.

Marionette.Controller может быть расширен, как и другие Backbone и Marionette объекты. Он поддерживает стандартный `initialize` метод, в нем присутствует
`EventBinder`, и он сам может генерить события.

```js
// define a controller
var MyController = Marionette.Controller.extend({

  initialize: function(options){
    this.stuff = options.stuff;
  },

  doStuff: function(){
    this.trigger("stuff:done", this.stuff);
  }

});

// create an instance
var c = new MyController({
  stuff: "some stuff"
});

// use the built in EventBinder
c.listenTo(c, "stuff:done", function(stuff){
  console.log(stuff);
});

// do some stuff
c.doStuff();
```

## Closing A Controller

## Закрытие контрроллера

Each Controller instance has a built in `close` method that handles
unbinding all of the events that are directly attached to the controller
instance, as well as those that are bound using the EventBinder from
the controller.

Любой экземпляр контроллера имеет встроенный `close`-метод, который отвязывает все события, которые были присоединены непосредственно к экземпляру контроллера,
а также те события, которые были привязаны посредством использования EventBinder-а контроллера.

The `close` method will trigger a "close" event and corresponding
`onClose` method call:

Метод `close` генерит событие "close" и вызывает соответствующий `onClose`-метод:

```js
// определяем контроллер с методом onClose:
var MyController = Marionette.Controller.extend({
  onClose: function(){
    // какой-то код, связанный с закрытием контроллера
  }
})
 
// создаем новый экземпляр контроллера
var contr = new MyController();
 
// и обработчики событий
contr.on("close", function(){ ... });
contr.listenTo(something, "bar", function(){...});
 
// закрываем контроллер - отвязываем все обработчики событий, вызываем событие "close" и метод onClose:
controller.close();
```

## On The Name 'Controller'

## По поводу названия 'Controller'

The name `Controller` is bound to cause a bit of confusion, which is
rather unfortunate. There was some discussion and debate about what to
call this object, the idea that people would confuse this with an 
MVC style controller came up a number of times. In the end, we decided
to call this a controller anyways, as the typical use case is to control
the workflow and process of an application and / or module. 

Так уж повелось, что именование `Controller` является источником определенной путаницы, и может показаться неудачным, особенно учитывая тот факт, что его обязательно будут путать с контроллером из MVC-архитектуры. Надо сказать, что среди создателей Marionette неоднократно происходили дебаты и дискуссии по поводу того, что должно соответствовать данному объекту. В конце концов, мы все равно решили закрепить за ним название контроллера, поскольку типичный случай его использования - контролировать рабочий процесс приложения и (или) модулей.

But the truth is, this is a very generic, multi-purpose object that can
serve many different roles in many different scenarios. We are always open
to suggestions, with good reason and discussion, on renaming objects to
be more descriptive, less confusing, etc. If you would like to suggest a
different name, please do so in either the mailing list or the github
issues list.

Но в действительности, это многоцеловой объект, который может исполнять совершенно разные роли в различных ситуациях.


