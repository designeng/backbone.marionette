# Marionette.AppRouter

Reduce the boilerplate code of handling route events and then calling a single method on another object.
Have your routers configured to call the method on your object, directly.



## Documentation Index

* [Конфигурация роутеров](#configure-routes)
* [Назначение контроллера](#specify-a-controller)

## Конфигурация роутеров

Configure an AppRouter with `appRoutes`. The route definition is passed on to Backbone's standard routing handlers. This means that you define routes like you normally would.  However, instead of providing a callback method that exists on the router, you provide a callback method that exists on the controller, which you specify for the router instance (see below.)

AppRouter приложения должен быть сконфигурирован с помощью `appRoutes`. Описание роутеров
Это значит, что вы определяете роутеры как будто бы вы имеете дело с обычным Backbone-маршрутизатором.
Однако, вместо того, чтобы помещать функцию, обрабатывающую экземпляр пути, внутри роутера,
вам следует добавить ее в соответствующий конструктор (см. ниже)

```js
MyRouter = Backbone.Marionette.AppRouter.extend({
  // "someMethod" must exist at controller.someMethod
  appRoutes: {
    "some/route": "someMethod"
  },
  
  /* standard routes can be mixed with appRoutes/Controllers above */
  /* стандартные Backbone-роуты могут дополнять appRoutes/Controllers, расположенные выше */

  routes : {
	"some/otherRoute" : "someOtherMethod"
  },
  someOtherMethod : function(){
	// делаем что-то
  }
  
});
```

You can also add standard routes to an AppRouter with methods on the router.

Вы также можете добавить стандартные роуты к AppRouter с помощью методов самих роутеров. (?)

## Назначение контроллера

App routers can only use one `controller` object. You can either specify this
directly in the router definition:

Каждый роутер приложения может использовать только один контроллер.
Вы можете также назначить контроллер прямо в определении роутера:

```js
someController = {
  someMethod: function(){ /*...*/ }
};

Backbone.Marionette.AppRouter.extend({
  controller: someController
});
```

... или через параметр, передаваемый в контруктор:

```js
myObj = {
  someMethod: function(){ /*...*/ }
};

new MyRouter({
  controller: myObj
});
```

The object that is used as the `controller` has no requirements, other than it will 
contain the methods that you specified in the `appRoutes`.

К объекту, используемому в качестве контроллера, не предъявляется никаких особенных требований,
кроме одного: он должен содержать все те методы, каторые вы определили в объекте `appRoutes`.

It is recommended that you divide your controller objects into smaller pieces of related functionality
and have multiple routers / controllers, instead of just one giant router and controller.

Рекомендуется объединять роутеры и соответствующие контроллеры по функциональности. Нормально, если приложение
будет иметь несколько сравнительно небольших роутеров / контроллеров, вместо одного гигантского роутера и контроллера.