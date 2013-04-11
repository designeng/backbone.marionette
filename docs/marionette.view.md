# Marionette.View

Marionette has a base `Marionette.View` type that other views extend from.
This base view provides some common and core functionality for
other views to take advantage of.

В состав Marionette входит `Marionette.View` - базовое представление, которое берут за основу
другие представления, расширяющие его общую для всех представлений функциональность.

**Note:** The `Marionette.View` type is not intended to be 
used directly. It exists as a base view for other view types
to be extended from, and to provide a common location for
behaviors that are shared across all views.

**Замечание:** Базовое представление `Marionette.View` не предназначено
для использования напрямую. Оно существует всего лишь как основа для дальнейшего расширения,
в качестве исходного прототипа для будущих представлений вашего приложения.



## Documentation Index

* [Binding To View Events](#binding-to-view-events)
* [View close](#view-close)
* [View onBeforeClose](#view-onbeforeclose)
* [View "dom:refresh" / onDomRefresh event](#view-domrefresh--ondomrefresh-event)
* [View.triggers](#viewtriggers)
* [View.modelEvents and View.collectionEvents](#viewmodelevents-and-viewcollectionevents)
* [View.serializeData](#viewserializedata)
* [View.bindUIElements](#viewbinduielements)
* [View.templateHelpers](#viewtemplatehelpers)
  * [Basic Example](#basic-example)
  * [Accessing Data Within The Helpers](#accessing-data-within-the-helpers)
  * [Object Or Function As `templateHelpers`](#object-or-function-as-templatehelpers)
* [Change Which Template Is Rendered For A View](#change-which-template-is-rendered-for-a-view)

## Binding To View Events

Marionette.View extends `Marionette.BindTo`. It is recommended that you use
the `listenTo` method to bind model, collection, or other events from Backbone
and Marionette objects.

Marionette.View является расширением `Marionette.BindTo`. Рекомендуется
использовать метод `listenTo` для привязки к событиям моделей, коллекций, или для прослушивания
любых других событий, поступающих из объектов Backbone и Marionette.

```js
MyView = Backbone.Marionette.ItemView.extend({
  initialize: function(){
    this.listenTo(this.model, "change:foo", this.modelChanged);
    this.listenTo(this.collection, "add", this.modelAdded);
  },

  modelChanged: function(model, value){
  },

  modelAdded: function(model){
  }
});
```

The context (`this`) will automatically be set to the view. You can
optionally set the context by passing in the context object as the
4th parameter of `listenTo`.

Контекст (на него указывает ключевое слово `this`) будет автоматически
установлен. Вы можете опционально менять контекст, передавая контекстный объект 
в качестве 4-го параметра в функцию `listenTo`.

## View close

View implements a `close` method, which is called by the region
managers automatically. As part of the implementation, the following
are performed:

Представление реализует метод `close`, который автоматически вызывается RegionManagers.
Детали реализации предполагают выполнение следующих действий:

* unbind all `listenTo` events
* unbind all custom view events
* unbind all DOM events
* remove `this.el` from the DOM
* call an `onBeforeClose` event on the view, if one is provided
* call an `onClose` event on the view, if one is provided

* прекращается прослушивание всех событий, которые были переданы в `listenTo`
* прекращается прослушивание всех пользовательских событий представления
* прекращается прослушивание всех событий, поступающих из DOM
* элемент `this.el` удаляется из объектной модели документа
* вызывается метод `onBeforeClose`, если он существует в данном представлении
* вызывается метод `onClose`, если он существует в данном представлении

By providing an `onClose` event in your view definition, you can
run custom code for your view that is fired after your view has been
closed and cleaned up. This lets you handle any additional clean up
code without having to override the `close` method.

Таким образом, вы можете создавать код, который будет выполнен,
когда представление завершит работу и будут очищены связанные с ним ресурсы. Для этого вам не нужно
переопределять метод `close`, достаточно определить `onClose` в своем представлении.

```js
Backbone.Marionette.ItemView.extend({
  onClose: function(){
    // ваш собственный код, который сработает на выходе
  }
});
```

## View onBeforeClose

When closing a view, an `onBeforeClose` method will be called, if it
has been provided. If this method returns `false`, the view will not
be closed. Any other return value (including null or undefined) will
allow the view to be closed.

Точно так же и метод `onBeforeClose` будет вызван, если он задан в представлении.
Если этот метод вернет `false`, представление не будет выключено. Любые другие значения
(включая null и undefined) не помешают представлению выключиться. 

```js
MyView = Marionette.View.extend({

  onBeforeClose: function(){
    // если мы не хотим, чтобы представление было выключено
    return false;
  }

});

var v = new MyView();

v.close(); // представление останется
```

### View "dom:refresh" / onDomRefresh event

Triggered after the view has been rendered, has been shown in the DOM via a Marionette.Region, and has been
re-rendered.

This event / callback is useful for 
[DOM-dependent UI plugins](http://lostechies.com/derickbailey/2012/02/20/using-jquery-plugins-and-ui-controls-with-backbone/) such as 
[jQueryUI](http://jqueryui.com/) or [KendoUI](http://kendoui.com).

```js
Backbone.Marionette.ItemView.extend({
  onDomRefresh: function(){
    // здесь можно производить всевозможные действия над элементом представления `el`,
    // поскольку он уже внедрен в страницу, и заполнен всем необходимым данному представлению
    // HTML-содержимым, готовым к работе.
  }
});
```

For more information about integration Marionette w/ KendoUI (also applicable to jQueryUI and other UI
widget suites), see [this blog post on KendoUI + Backbone](http://www.kendoui.com/blogs/teamblog/posts/12-11-26/backbone_and_kendo_ui_a_beautiful_combination.aspx).

Для более детальной информации по поводу интеграции Marionette и KendoUI (что также применимо к jQueryUI и другим UI
библиотекам), смотрите [пост про интеграцию KendoUI и Backbone](http://www.kendoui.com/blogs/teamblog/posts/12-11-26/backbone_and_kendo_ui_a_beautiful_combination.aspx).

## View.triggers

Views can define a set of `triggers` as a hash, which will 
convert a DOM event into a `view.triggerMethod` call.

The left side of the hash is a standard Backbone.View DOM
event configuration, while the right side of the hash is the
view event that you want to trigger from the view.

```js
MyView = Backbone.Marionette.ItemView.extend({
  // ...

  triggers: {
    "click .do-something": "something:do:it"
  }
});

view = new MyView();
view.render();

view.on("something:do:it", function(args){
  alert("I DID IT!");
});

// "click" the 'do-something' DOM element to 
// demonstrate the DOM event conversion
view.$(".do-something").trigger("click");
```

The result of this is an alert box that says, "I DID IT!"

You can also specify the `triggers` as a function that 
returns a hash of trigger configurations

```js
Backbone.Marionette.CompositeView.extend({
  triggers: function(){
    return {
      "click .that-thing": "that:i:sent:you"
    };
  }
});
```

Triggers work with all View types that extend from the base
Marionette.View.

### Trigger Handler Arguments

A `trigger` event handler will receive a single argument that
includes the following:

* view
* model
* collection

These properties match the `view`, `model`, and `collection` properties of the view that triggered the event.

```js
MyView = Backbone.Marionette.ItemView.extend({
  // ...

  triggers: {
    "click .do-something": "some:event"
  }
});

view = new MyView();

view.on("some:event", function(args){
  args.view; // => экземпляр представления, который генерирует событие
  args.model; // => модель представления, если таковая существует
  args.collection; // => коллекция представления, если таковое существует
});
```

Having access to these allows more flexibility in handling events from
multiple views. For example, a tab control or expand/collapse widget such
as a panel bar could trigger the same event from many different views
and be handled with a single function.

То обстоятельство, что мы имеем доступ к данным полям представления, обеспечивает
большую гибкость в обработке событий. Например, компонент с возможностью переключения
между вкладками, или разворачивающийся виджет (аккордеон), могут порождать определенное событие
из различных вложенных в компонент представлений, которое будет обрабатываться одной-единственной функцией.


## View.modelEvents и View.collectionEvents

Similar to the `events` hash, views can specify a configuration
hash for collections and models. The left side is the event on
the model or collection, and the right side is the name of the
method on the view.

Подобно хешу событий (`events`), представления могут задавать конфигурационный хеш
для событий коллекции и модели, где в качестве ключа будет выступать наименование события,
совершающегося над моделью или коллекцией, а в качестве значения - название метода представления.

```js
Backbone.Marionette.CompositeView.extend({

  modelEvents: {
    "change:name": "nameChanged" // эквивалентно view.listenTo(view.model, "change:name", view.nameChanged, view)
  },

  collectionEvents: {
    "add": "itemAdded" // эквивалентно view.listenTo(view.collection, "add", collection.itemAdded, view)
  },

  // ... обработчики событий
  nameChanged: function(){ /* ... */ },
  itemAdded: function(){ /* ... */ },

})
```

These will use the memory safe `listenTo`, and will set the context
(the value of `this`) in the handler to be the view. Events are
bound at the time of instantiation instantiation, and an exception will be thrown
if the handlers on the view do not exist.

Таким образом, modelEvents и collectionEvents используют в определении зависимости метода от события
безопасный с точки зрения расхода памяти метод `listenTo`, и устанавливают в качестве контекста обработчика
(т.е. значения `this`) само данное представление. События привязываются к обработчикам 
в момент создания экземпляра представления, и если соответствующего обработчика не существует,
будет брошено исключение (exception).

The `modelEvents` and `collectionEvents` will be bound and
unbound with the Backbone.View `delegateEvents` and `undelegateEvents`
method calls. This allows the view to be re-used and have
the model and collection events re-bound.

И в modelEvents, и в collectionEvents события привязываются с помощью вызовов методов Backbone.View
`delegateEvents` и `undelegateEvents`. Это позволяет представлениям проходить через этапы
многократного использования и переназначения событий.


### Multiple Callbacks

Multiple callback functions can be specified by separating them with a
space. 

Множество функций, обрабатывающих одно событие, может быть определено,
если записать их названия в строку через пробел.

```js
Backbone.Marionette.CompositeView.extend({

  modelEvents: {
    "change:name": "nameChanged thatThing"
  },

  nameChanged: function(){ },

  thatThing: function(){ },
});
```

This works in both `modelEvents` and `collectionEvents`.

Это справедливо как для `modelEvents`, так и для `collectionEvents`.

### Callbacks As Function

A single function can be declared directly in-line instead of specifying a
callback via a string method name.

В качестве обработчика может быть также определена анонимная функция, тогда она будет выступать единственным обработчиком события: 

```js
Backbone.Marionette.CompositeView.extend({

  modelEvents: {
    "change:name": function(){
      // здесь обрабатываем событие "change:name"
    }
  }

});
```

This works for both `modelEvents` and `collectionEvents`.

Это справедливо как для `modelEvents`, так и для `collectionEvents`.

### Конфигурация событий в виде функции

A function can be used to declare the event configuration as long as
that function returns a hash that fits the above configuration options.

Стоит добавить, что modelEvents и collectionEvents могут быть представлены не в виде объектного литерала,
а в виде функции, которая возвращает хеш, соответствующий выше описанной конфигурации, т.е. ключ - это событие, а значение - функция, это событие обрабатывающая:

```js
Backbone.Marionette.CompositeView.extend({

  modelEvents: function(){
    return { "change:name": "someFunc" };
  }

});
```

## Метод View.serializeData

The `serializeData` method will serialize a view's model or
collection - with precedence given to collections. That is,
if you have both a collection and a model in a view, calling
the `serializeData` method will return the serialized
collection.

Метод `serializeData` сериализует данные модели/коллекции представления - отдавая
предпочтение коллекции. То-есть, если в вашем представлении одновременно присутствуют и коллекция, 
и модель, метод `serializeData` вернет сериализованную коллекцию.

## Метод View.bindUIElements

In several cases you need to access ui elements inside the view
to retrieve their data or manipulate them. For example you have a
certain div element you need to show/hide based on some state,
or other ui element that you wish to set a css class to it.
Instead of having jQuery selectors hanging around in the view's code
you can define a `ui` hash that contains a mapping between the
ui element's name and its jQuery selector. Afterwards you can simply
access it via `this.ui.elementName`.
See ItemView documentation for examples.

В некоторых случаях требуется иметь доступ к элементам пользовательского интерфейса внутри представления,
чтобы получать от них какие-то данные, или манипулировать самими элементами.
Например, если у вас имеется какой-то элемент, который в зависимости от его состояния требуется
показать или скрыть, или стоит задача добавить некоторый класс к определенному элементу.
Вместо того, чтобы жонглировать jQuery-селекторами по всему коду превставления,
вы можете просто задать `ui` хеш, который определит четкое соответствие между именами элементов
пользовательского интерфейса и их jQuery-селекторами. Впоследствии вы можете обращаться к ним
с помощью конструкции `this.ui.elementName`. Примеры находятся в разделе, посвященном Marionette.ItemView.

This functionality is provided via the `bindUIElements` method.
Since View doesn't implement the render method, then if you directly extend
from View you will need to invoke this method from your render method.
In ItemView and CompositeView this is already taken care of.

Важно заметить, что эта функциональность обеспечена методом `bindUIElements`. Поскольку само базовое представление (Marionette.View)
не реализует метод `render`, наследуя ваше представление напрямую от Marionette.View, вы столкнетесь с необходимостью
вызывать метод `bindUIElements` из метода `render` вашего представления.
В Marionette.ItemView and Marionette.CompositeView об этом уже позаботились.

## Аттрибут View.templateHelpers

There are times when a view's template needs to have some
logic in it and the view engine itself will not provide an
easy way to accomplish this. For example, Underscore templates
do not provide a helper method mechanism while Handlebars
templates do.

Бывают случаи, когда нужно добавить в шаблон представления (view's template)
определенную логику, а шаблонизатор не предоставляет такой возможности.
Например, мини-шаблонизатор Underscore не обеспечивает поддержку хелперов, в отличие от Handlebars.

A `templateHelpers` attribute can be applied to any View object that
renders a template. When this attribute is present it's contents 
will be mixed in to the data object that comes back from the 
`serializeData` method. This will allow you to create helper methods 
that can be called from within your templates.

Аттрибут `templateHelpers` может быть применен к любому объекту Marionette.View,
который внедряет шаблон для отбражения данных внутри элементов. `templateHelpers` добавляет свое содержимое на правах миксина к объекту, возвращаемому методом `serializeData`. Это дает вам возможность создавать свои собственные вспомогательные методы (helpers), которые впоследствии будут применены внутри шаблонов. 

### Пример

```html
<script id="my-template" type="text/html">
  I think that <%= showMessage() %>
</script>
```

```js
MyView = Backbone.Marionette.ItemView.extend({
  template: "#my-template",

  templateHelpers: {
    showMessage: function(){
      return this.name + " is the coolest!"
    }
  }

});

model = new Backbone.Model({name: "Backbone.Marionette"});
view = new MyView({
  model: model
});

view.render(); //=> "I think that Backbone.Marionette is the coolest!";
```

### Accessing Data Within The Helpers
### Доступ к данным внутри хелперов

In order to access data from within the helper methods, you
need to prefix the data you need with `this`. Doing that will
give you all of the methods and attributes of the serialized
data object, including the other helper methods.

Доступ к данным внутри хелпера производится с помощью добавления `this`.
в качестве префикса. Так вы можете получить любые поля и методы сериализованного объекта,
в том числе обратиться и к другим определенным для шаблона хелперам.  

```js
templateHelpers: {
  something: function(){
    return "Do stuff with " + this.name + " because it's awesome.";
  }
}
```

### Object Or Function As `templateHelpers`

You can specify an object literal (as shown above), a reference
to an object literal, or a function as the `templateHelpers`. 

В качестве `templateHelpers` вы можете определить объектный литерал (как показано выше),
ссылку на объектный литерал, или функцию.

If you specify a function, the function will be invoked 
with the current view instance as the context of the 
function. The function must return an object that can be
mixed in to the data for the view.

Если вы определяете в качестве `templateHelpers` функцию, она будет вызвана с текущим экземпляром представления в качестве контекста. Функция должна возвращать объект, который может быть добавлен к данным вашего представления на правах миксина.

```js
Backbone.Marionette.ItemView.extend({
  templateHelpers: function(){
    return {
      foo: function(){ /* ... */ }
    }
  }
});
```

## Change Which Template Is Rendered For A View
## Замена шаблона представления

There may be some cases where you need to change the template that is
used for a view, based on some simple logic such as the value of a
specific attribute in the view's model. To do this, you can provide
a `getTemplate` function on your views and use this to return the
template that you need.

Бывают случаи, когда нужно заменить шаблон, используемый в представлении, на какой-то другой, 
например, на основании специфического аттрибута модели. Это возможно, если вы определите функцию `getTemplate` в вашем представлении, которая реализует логику выбора необходимого шаблона, и вернет его идентификатор.

```js
MyView = Backbone.Marionette.ItemView.extend({
  getTemplate: function(){
    if (this.model.get("foo")){
      return "#some-template";
    } else {
      return "#a-different-template";
    }
  }
});
```

Это справедливо для всех типов представлений.