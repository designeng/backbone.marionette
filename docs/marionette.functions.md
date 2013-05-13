# Marionette functions

# Marionette: функции

Marionette provides a set of utility / helper functions that are used to
facilitate common behaviors throughout the framework. These functions may
be useful to those that are building on top of Marionette, as the provide
a way to get the same behaviors and conventions from your own code.

Marionette предоставляет ряд полезных функций, которые обеспечивают общее поведение объектов в рамках всего фреймворка.


## Documentation Index

* [Marionette.extend](#marionetteextend)
* [Marionette.getOption](#marionetteextend)
* [Marionette.triggerMethod](#marionettetriggermethod)
* [Marionette.bindEntityEvent](#marionettebindentityevents)

## Marionette.extend

Backbone's `extend` function is a useful utility to have, and is used in
various places in Marionette. To make the use of this method more consistent,
Backbone's `extend` has been aliased to `Marionette.extend`. This allows
you to get the extend functionality for your object without having to
decide if you want to use Backbone.View or Backbone.Model or another
Backbone object to grab the method from.

Очень полезна функция `extend`, присутствующая в Backbone. Она нашла свое применение и в различных местах кода Marionette.
Для совпадения функций и во избежание двусмысленности, Marionette.extend [является псевдонимом Backbone.extend](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.helpers.js#L20).

```js
var Foo = function(){};

// используем Marionette.extend для того, чтобы сделать объект Foo наследуемым, наподобие других Backbone и Marionette объектов 
Foo.extend = Marionette.extend;

// Now Foo can be extended to create a new type, with methods

// Теперь Foo может быть наследован (расширен), для создания экземпляров объекта нового типа, с рядом новых методов
var Bar = Foo.extend({

  someMethod: function(){ ... }

  // ...
});

// создаем экземпляр Bar
var b = new Bar();
```

## Marionette.getOption

Retrieve an object's attribute either directly from the object, or from
the object's `this.options`, with `this.options` taking precedence.

Дает возможность получить атрибуты объекта или непосредственно из самого объекта,
или из `this.options`, если при создании экземпляра объекта они были заданы.


```js
var M = Backbone.Model.extend({
  foo: "bar",

  initialize: function(){
    var f = Marionette.getOption(this, "foo");
    console.log(f);
  }
});

new M(); // => "bar"

new M({}, { foo: "quux" }); // => "quux"
```

This is useful when building an object that can have configuration set
in either the object definition or the object's constructor options.

Полезно, когда при построении объекта в конструктор передаются инициализирующие аргументы.

### Falsey values

### Свойства объекта с логическим значением false ([Falsey values](http://james.padolsey.com/javascript/truthy-falsey/))

The `getOption` function will return any falsey value from the `options`,
other than `undefined`. If an object's options has an undefined value, it will
attempt to read the value from the object directly.

Функция `getOption` будет [пытаться прочесть значения полей, приводимых к логическому false, непосредственно из объекта](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.helpers.js#L34).

Пример:

```js
var M = Backbone.Model.extend({
  foo: "bar",

  initialize: function(){
    var f = Marionette.getOption(this, "foo");
    console.log(f);
  }
});

new M(); // => "bar"

var f;
new M({}, { foo: f }); // => "bar"
```

In this example, "bar" is returned both times because the second 
example has an undefined value for `f`.

В обоих случаях возвращается "bar", поскольку во втором случае `f` имеет неопределенное значение.

## Marionette.triggerMethod

Trigger an event and a corresponding method on the target object.

Порождает событие и запускает соответствующий метод на целевом объекте.

When an event is triggered, the first letter of each section of the 
event name is capitalized, and the word "on" is tagged on to the front 
of it. Examples:

Когда событие порождено, происходит следующее: [к его названию, возведенному в верхний регистр по первой букве, добавляется приставка "on"](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.triggermethod.js#L22). Например:

* `triggerMethod("render")` fires the "onRender" function
* `triggerMethod("before:close")` fires the "onBeforeClose" function

* `triggerMethod("render")` [будет пытаться](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.triggermethod.js#L29) запустить функцию "onRender"
* `triggerMethod("before:close")` [будет пытаться](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.triggermethod.js#L29) запустить функцию "onBeforeClose"

All arguments that are passed to the triggerMethod call are passed along to both the event and the method, with the exception of the event name not being passed to the corresponding method.

Все аргументы, переданные в triggerMethod, будут транслированы одновременно как в событие, так и в соответствующую функцию - за тем исключением,
что [название события не передастся в функцию](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.triggermethod.js#L30-L31).



`triggerMethod("foo", bar)` вызовет `onFoo: function(bar){...})`

## Marionette.bindEntityEvents

This method is used to bind a backbone "entity" (collection/model) 
to methods on a target object. 

Этот метод привязывает backbone'овскую "сущность" (collection/model) к методу целевого объекта.

```js
Backbone.View.extend({

  modelEvents: {
    "change:foo": "doSomething"
  },

  initialize: function(){
    Marionette.bindEntityEvents(this, this.model, this.modelEvents);
  },

  doSomething: function(){
    // событие "change:foo" произошедшее в модели, запустит код, находящийся здесь
  }

});
```

The first paremter, `target`, must have a `listenTo` method from the
EventBinder object.

Первый параметр, `target`, должен иметь метод `listenTo` из объекта EventBinder.

The second parameter is the entity (Backbone.Model or Backbone.Collection)
to bind the events from.

Второй параметр - сущность (Backbone.Model или Backbone.Collection), к которой привязывается событие.

The third parameter is a hash of { "event:name": "eventHandler" }
configuration. Multiple handlers can be separated by a space. A
function can be supplied instead of a string handler name. 

Третий параметр - хеш из конфигурации { "event:name": "eventHandler" }. Если событию соответствует множество обработчиков,
они должны быть разграничены пробелом. Вместо строкового названия обработчика может стоять функция.

(Методу Marionette.bindEntityEvents сопоставлен метод [Marionette.unbindEntityEvents](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.bindEntityEvents.js#L86-L88), но в документации от этом не говорится - прим. пер.)