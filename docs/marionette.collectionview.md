# Marionette.CollectionView

The `CollectionView` will loop through all of the models in the
specified collection, render each of them using a specified `itemView`,
then append the results of the item view's `el` to the collection view's
`el`.

`CollectionView` проходит циклом по всем моделям, представленным в данной коллекции,
отображает их посредством определенного `itemView`, и все `el` из `itemView`, с отрендеренным в них html, добавляет в `el` `сollectionView`.

## Documentation Index

* [CollectionView's `itemView`](#collectionviews-itemview)
* [CollectionView's `itemViewOptions`](#collectionviews-itemviewoptions)
* [CollectionView's `emptyView`](#collectionviews-emptyview)
* [CollectionView's `buildItemView`](#collectionviews-builditemview)
* [Callback Methods](#callback-methods)
  * [onBeforeRender callback](#onbeforerender-callback)
  * [onRender callback](#onrender-callback)
  * [onItemAdded callback](#onitemadded-callback)
  * [onBeforeClose callback](#beforeclose-callback)
  * [onClose callback](#onclose-callback)
* [CollectionView Events](#collectionview-events)
  * ["before:render" / onBeforeRender event](#beforerender--onbeforerender-event)
  * ["render" / onRender event](#render--onrender-event)
  * ["before:close" / onBeforeClose event](#beforeclose--onbeforeclose-event)
  * ["closed" / "collection:closed" event](#closed--collectionclosed-event)
  * ["before:item:added" / "after:item:added"](#beforeitemadded--afteritemadded)
  * ["item:removed" / onItemRemoved](#itemremoved--onitemremoved)
  * ["itemview:\*" event bubbling from child views](#itemview-event-bubbling-from-child-views)
* [CollectionView's `itemViewEventPrefix`](#collectionviews-itemvieweventprefix)
* [CollectionView render](#collectionview-render)
* [CollectionView: Automatic Rendering](#collectionview-automatic-rendering)
* [CollectionView: Re-render Collection](#collectionview-re-render-collection)
* [CollectionView's appendHtml](#collectionviews-appendhtml)
* [CollectionView's children](#collectionviews-children)
* [CollectionView close](#collectionview-close)

## CollectionView's `itemView`

## CollectionView: `itemView`

Specify an `itemView` in your collection view definition. This must be
a Backbone view object definition, not an instance. It can be any 
`Backbone.View` or be derived from `Marionette.ItemView`.

Необходимо задать `itemView` для `сollectionView`, - `itemView` является не конкретным экземпляром, а объектом, выводимым (extended) из `Backbone.View` или `Marionette.ItemView`.

```js
MyItemView = Backbone.Marionette.ItemView.extend({});

Backbone.Marionette.CollectionView.extend({
  itemView: MyItemView
});
```

Alternatively, you can specify an `itemView` in the options for
the constructor:

В качестве альтернативы, можно определить `itemView` в конструкторе:

```js
MyCollectionView = Backbone.Marionette.CollectionView.extend({...});

new MyCollectionView({
  itemView: MyItemView
});
```

If you do not specify an `itemView`, an exception will be thrown
stating that you must specify an `itemView`.

Если `itemView` не определен, [будет брошено исключение с соответствующим предупреждением](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L125-L127).

If you need a view specific to your model, you can override 
`getItemView`:

Если выбор itemView зависит от данных модели, `getItemView` может быть переопределен: 

```js
Backbone.Marionette.CollectionView.extend({
  getItemView: function(item) {
    // логика для вычисления соответствующего представления
    return someItemSpecificView;
  }
})
```

## CollectionView's `itemViewOptions`

## CollectionView: `itemViewOptions`

There may be scenarios where you need to pass data from your parent
collection view in to each of the itemView instances. To do this, provide
a `itemViewOptions` definition on your collection view as an object
literal. This will be passed to the constructor of your itemView as part
of the `options`.

Встречаются случаи, когда необходимо передать данные из родительского CollectionView
в каждый экземпляр itemView. Для этого в CollectionView нужно задать `itemViewOptions` в качестве объектного литерала, 
тогда эти данные добавятся к `options` конструктора при создании экземпляра itemView.



```js
ItemView = Backbone.Marionette.ItemView({
  initialize: function(options){
    console.log(options.foo); // => "bar"
  }
});

CollectionView = Backbone.Marionette.CollectionView({
  itemView: ItemView,

  itemViewOptions: {
    foo: "bar"
  } 
});
```

You can also specify the `itemViewOptions` as a function, if you need to
calculate the values to return at runtime. The model will be passed into
the function should you need access to it when calculating
`itemViewOptions`. The function must return an object, and the attributes 
of the object will be copied to the `itemView` instance's options.

Вы можете также задать в качестве `itemViewOptions` функцию, тогда объектный литерал будет вычислен и возвращен на этапе исполнения кода. Одним из параметров для этой функции является модель, поскольку логика вычисления `itemViewOptions` основывается на данных модели. Атрибуты возвращенного объекта добавятся к `options` конструктора при создании экземпляра itemView.

```js
CollectionView = Backbone.Marionette.CollectionView({
  itemViewOptions: function(model, index) {
    // производим вычисления на основании данных модели
    return {
      foo: "bar",
      itemIndex: index
    }   
  }  
});
```

## CollectionView's `emptyView`

## Атрибут `emptyView` CollectionView

When a collection has no items, and you need to render a view other than
the list of itemViews, you can specify an `emptyView` attribute on your
collection view.

Если коллекция еще (или уже) пуста, и вам необходимо вывести представление, отличающееся от списка отдельных itemView, то вы можете определить атрибут [`emptyView`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L100) в CollectionView.

```js
NoItemsView = Backbone.Marionette.ItemView.extend({
  template: "#show-no-items-message-template"
});

Backbone.Marionette.CollectionView.extend({
  // ...

  emptyView: NoItemsView
});
```

This will render the `emptyView` and display the message that needs to
be displayed when there are no items.

В этом примере `emptyView` отобразится на месте списка, если тот пуст.

## CollectionView's `buildItemView`

## Атрибут `buildItemView` CollectionView

When a custom view instance needs to be created for the `itemView` that
represents an item, override the `buildItemView` method. This method
takes three parameters and returns a view instance to be used as the
item view.

Если вам нужно создать экземпляр пользовательского представления для `itemView`, переопределите метод [`buildItemView`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L190-L193). Этот метод принимает три параметра: item, ItemViewType, itemViewOptions.

```js
buildItemView: function(item, ItemViewType, itemViewOptions){
  // формируем конечный список параметров для item view, который будет выстраиваться по определенному типу
  var options = _.extend({model: item}, itemViewOptions);
  // создаем новый экземпляр
  var view = new ItemViewType(options);
  // возвращаем его
  return view;
},
```

## Callback Methods

## Методы обратного вызова

There are several callback methods that can be provided on a
`CollectionView`. If they are found, they will be called by the
view's base methods. These callback methods are intended to be
handled within the view definition directly.

`CollectionView` может предоставлять ряд коллбэков. Если они обнаружатся в представлении, то будут вызваны с помощью базовых методов представления.
Предполагается, что эти коллбэки обрабатываются прямо внутри определения представления. (???)

### onBeforeRender callback

### onBeforeRender

A `onBeforeRender` callback will be called just prior to rendering
the collection view.

`onBeforeRender` вызовется непосредственно перед рендерингом представления.

```js
Backbone.Marionette.CollectionView.extend({
  onBeforeRender: function(){
    // делаем что-то
  }
});
```

### onRender callback

### onRender

After the view has been rendered, a `onRender` method will be called.
You can implement this in your view to provide custom code for dealing
with the view's `el` after it has been rendered:

Сразу после того, как представление отреднерено, будет вызван метод `onRender`.
Вы можете реализовать его в представлении, например, чтобы взаимодействовать c элементом представления `el`, который также уже будет отреднерен. 

```js
Backbone.Marionette.CollectionView.extend({
  onRender: function(){
    // делаем что-то
  }
});
```

### onItemAdded callback

### onItemAdded коллбэк

This callback function allows you to know when an item / item view
instance has been added to the collection view. It provides access to
the view instance for the item that was added.

Данный коллбэк уведомляет, что к collection view был добавлен элемент, либо экземпляр item view. Внутри него мы получаем доступ к соответствующему элементу коллекции экземпляру itemView.

```js
Backbone.Marionette.CollectionView.extend({
  onItemAdded: function(itemView){
    // делаем что-то с экземпляром itemView 
  }
});
```

### onBeforeClose callback

### onBeforeClose коллбэк

This method is called just before closing the view.

Данный метод вызывается прямо перед закрытием представления.

```js
Backbone.Marionette.CollectionView.extend({
  onBeforeClose: function(){
    // делаем что-то
  }
});
```

### onClose callback

### onClose коллбэк

This method is called just after closing the view.

Данный метод вызывается сразу после закрытия представления.

```js
Backbone.Marionette.CollectionView.extend({
  onClose: function(){
    // делаем что-то
  }
});
```

## CollectionView Events

## События CollectionView 

There are several events that will be triggered during the life
of a collection view. Each of these events is called with the
[Marionette.triggerMethod](./marionette.functions.md) function,
which calls a corresponding "on{EventName}" method on the
view instance.

В течение жизненного цикла collection view порождается ряд событий.
Каждое из этих событий генерится с помощью функции [Marionette.triggerMethod](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.triggermethod.js#L8-L36), которая одновременно вызывает соответствующий событию метод "on{EventName}".

### "before:render" / onBeforeRender event

### Событие "before:render" или метод onBeforeRender


Triggers just prior to the view being rendered. Also triggered as 
"collection:before:render" / `onCollectionBeforeRender`.

Порождается непосредственно перед рендерингом представления. Синонимом данного события является событие ["collection:before:render"](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L51), которому соответствует метод `onCollectionBeforeRender`. 

```js
MyView = Backbone.Marionette.CollectionView.extend({...});

var myView = new MyView();

myView.on("before:render", function(){
  alert("the collection view is about to be rendered");
});

myView.render();
```

### "render" / onRender event

### Событие "render" или метод onRender

A "collection:rendered" / `onCollectionRendered` event will also be fired. This allows you to
add more than one callback to execute after the view is rendered,
and allows parent views and other parts of the application to
know that the view was rendered.

Порождается сразу после рендеринга представления. Синонимом данного события является событие ["collection:rendered"](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L58), которому соответствует метод `onCollectionRendered`.

```js
MyView = Backbone.Marionette.CollectionView.extend({...});

var myView = new MyView();

myView.on("render", function(){
  alert("the collection view was rendered!");
});

myView.on("collection:rendered", function(){
  alert("the collection view was rendered!");
});

myView.render();
```

### "before:close" / onBeforeClose event

### Событие "before:close" или метод onBeforeClose

Triggered just before closing the view. A "collection:before:close" /
`onCollectionBeforeClose` event will also be fired

Порождается непосредственно перед закрытием представления. Одновременно с ним породится событие ["collection:before:close"](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L247), которому соответствует метод `onCollectionBeforeClose`.

```js
MyView = Backbone.Marionette.CollectionView.extend({...});

var myView = new MyView();

myView.on("collection:before:close", function(){
  alert("the collection view is about to be closed");
});

myView.close();
```

### "closed" / "collection:closed" event

### Событие "closed" / ["collection:closed"](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L249)

Triggered just after closing the view, both with corresponding
method calls.

Порождается сразу после закрытия представления, с вызовом соответствующих методов.

```js
MyView = Backbone.Marionette.CollectionView.extend({...});

var myView = new MyView();

myView.on("collection:closed", function(){
  alert("the collection view is now closed");
});

myView.close();
```

(Событие "closed" - разве есть?)

### "before:item:added" / "after:item:added"

### События "before:item:added" / "after:item:added"

The "before:item:added" event and corresponding `onBeforeItemAdded`
method are triggered just after creating a new itemView instance for 
an item that was added to the collection, but before the
view is rendered and added to the DOM.

Событие ["before:item:added"](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L148) (которому соответствует метод `onBeforeItemAdded`) порождается сразу после создания нового экземпляра itemView для элемента, добавленного в коллекцию,
но перед тем как представление [будет отрендерено и добавлено в DOM](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L160).

The "after:item:added" event and corresponding `onAfterItemAdded`
method are triggered after rendering the view and adding it to the
view's DOM element.

Событие ["after:item:added"](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L164)
(которому соответствует метод `onAfterItemAdded`) порождается [сразу после рендеринга представления](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L160)

```js
var MyCV = Marionette.CollectionView.extend({
  // ...

  onBeforeItemAdded: function(){
    // ...
  },

  onAfterItemAdded: function(){
    // ...
  }
});

var cv = new MyCV({...});

cv.on("before:item:added", function(viewInstance){
  // ...
});

cv.on("after:item:added", function(viewInstance){
  // ...
});
```

### "item:removed" / onItemRemoved

### Событие "item:removed" или метод onItemRemoved

Triggered after an itemView instance has been closed and
removed, when it's item was deleted or removed from the
collection.

Событие ["item:removed"](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L217)
порождается сразу после того, как экземпляр itemView будет закрыт или удален ([ищутся методы 'close' или 'remove'](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L210-L212)), что произойдет в свою очередь после удаления соответствующего ему элемента из коллекции.

```js
cv.on("item:removed", function(viewInstance){
  // ...
});
```

### "itemview:\*" event bubbling from child views

### Всплытие события "itemview:\*" из встроенных представлений

When an item view within a collection view triggers an
event, that event will bubble up through the parent 
collection view with "itemview:" prepended to the event
name. 

Когда item view внутри collection view [порождает какое-то событие](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L145), оно всплывает сквозь родительский collection view, и название события [предваряется конструкцией "itemview:"](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L172-L177) заданной [по умолчанию](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L9).

That is, if a child view triggers "do:something", the 
parent collection view will then trigger "itemview:do:something".

То-есть, если встроенное представление сгенерит "do:something", родительское представление сгенерит соответственно "itemview:do:something".

```js
// задаем базовую коллекцию
var myModel = new MyModel();
var myCollection = new MyCollection();
myCollection.add(myModel);

//  создаем и рендерим collection view
colView = new CollectionView({
  collection: myCollection
});
colView.render();

// привязываем всплывающие события
colView.on("itemview:do:something", function(childView, msg){
  alert("I said, '" + msg + "'");
});

// это хак - просто чтобы сгенерить событие для нашего примера
var childView = colView.children[myModel.cid];
childView.trigger("do:something", "do something!");
```

В результате будет выведено предупреждение с текстом "I said, 'do something!'". 

Also note that you would not normally grab a reference to
the child view the way this is showing. I'm merely using
that hack as a way to demonstrate the event bubbling. 
Normally, you would have your item view listening to DOM
events or model change events, and then triggering an event
of it's own based on that.

Добавим, что мы использовали хак с получением прямой ссылки на встроенное представление просто для генерации события, обычно все обстоит иначе - генерация событий в item view обусловлена его подпиской на прослушивание DOM-событий или событий, связанных с изменением модели.

## CollectionView's `itemViewEventPrefix`

## CollectionView: `itemViewEventPrefix`

You can customize the event prefix for events that are forwarded
through the collection view. To do this, set the `itemViewEventPrefix`
on the collection view.

Префикс события itemView, всплывающего сквозь collection view, может быть кастомизирован. Для этого нужно задать атрибут [`itemViewEventPrefix`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L9) в экземпляре collection view.


```js
var CV = Marionette.CollectionView.extend({
  itemViewEventPrefix: "some:prefix"
});

var c = new CV({
  collection: myCol
});

c.on("some:prefix:render", function(){
  // item view был отрендерен, поэтому что-то делаем
});

c.render();
```

The `itemViewEventPrefix` can be provided in the view definition or
in the constructor function call, to get a view instance.

`itemViewEventPrefix` может быть определен в дефиниции представления или в вызове конструктора.

## CollectionView render

## Рендеринг CollectionView

The `render` method of the collection view is responsible for
rendering the entire collection. It loops through each of the
items in the collection and renders them individually as an
`itemView`.

Метод `render` collection view отвечает за отображение всей коллекции. Он проходит циклом по всем элементам коллекции и отображает их индивидуально в `itemView`.

```js
MyCollectionView = Backbone.Marionette.CollectionView.extend({...});

new MyCollectionView().render().done(function(){
  // все встроенные представления к этому моменту уже отобразились, делаем что-то
});
```

## CollectionView: Automatic Rendering

## CollectionView: автоматический рендеринг

The collection view binds to the "add", "remove" and "reset" events of the
collection that is specified. 

Сollection view привязывается к событиям "add", "remove" и "reset" коллекции.

When the collection for the view is "reset", the view will call `render` on
itself and re-render the entire collection.

Как только коллекция будет переустановлена (произойдет событие "reset"), [представление вызовет собственный метод `render`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L27) и заново отобразит обновленную коллекцию.


When a model is added to the collection, the collection view will render that
one model in to the collection of item views.

Когда в коллекцию будет добавлена модель, collection view [отобразит](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L25) данную модель в наборе item views.

When a model is removed from a collection (or destroyed / deleted), the collection
view will close and remove that model's item view.

Когда модель будет [удалена из коллекции](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L26), collection view [закроет или удалит](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L211-L212) соответствующий item view.

## CollectionView: Re-render Collection

## CollectionView: повторное отображение коллекции

If you need to re-render the entire collection, you can call the
`view.render` method. This method takes care of closing all of
the child views that may have previously been opened.

Если необходимо перерендерить целиком всю коллекцию, можно вызвать метод `view.render`.
Этот метод позаботится о закрытии всех встроенных представлений, которые были открыты ранее.

## CollectionView's appendHtml

## CollectionView: appendHtml

By default the collection view will call jQuery's `.append` to
move the HTML contents from the item view instance in to the collection
view's `el`.

По умолчанию collection view [вызывает метод jQuery `.append`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L233), чтобы добавить HTML-содержимое экземпляра item view к `el` коллекции.

You can override this by specifying an `appendHtml` method in your 
view definition. This method takes three parameters and has no return
value.

Это поведение может быть переопределено в методе [`appendHtml`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L229-L234) в конкретном представлении.
Данный метод принимает три параметра и не возвращает значений.

```js
Backbone.Marionette.CollectionView.extend({

	// The default implementation:
  appendHtml: function(collectionView, itemView, index){
    collectionView.$el.append(itemView.el);
  }

});
```

The first parameter is the instance of the collection view that 
will receive the HTML from the second parameter, the current item
view instance. 

Первый параметр - экземпляр CollectionView, второй - текущий экземпляр item view, предоставляющий готовый для рендеринга HTML в своем itemView.el.

The third parameter, `index`, is the index of the
model that this itemView instance represents, in the collection 
that the model came from. This is useful for sorting a collection
and displaying the sorted list in the correct order on the screen.

Третий параметр - `index`. Это индекс модели, которую представляет экземпляр itemView.
Он может оказаться полезным при сортировке коллекции и отображении отсортированного списка в должном порядке.

## CollectionView's children

## CollectionView: встроенные представления

The CollectionView uses [Backbone.BabySitter](https://github.com/marionettejs/backbone.babysitter)
to store and manage it's child views. This allows you to easily access
the views within the collection view, iterate them, find them by
a given indexer such as the view's model or collection, and more.

CollectionView использует [Backbone.BabySitter](https://github.com/marionettejs/backbone.babysitter) для хранения и организации встроенных представлений,
что предполагает легкий доступ к представлениям внутри базового представления, перебор их и нахождение по данному индексу точно так же, как если бы мы работали с моделями внутри коллекции, и ряд других действий.

```js
var cv = new Marionette.CollectionView({
  collection: someCollection
});

cv.render();


// нахождение представления по модели
var v = cv.children.findByModel(someModel);

// перебор представлений и их обработка
cv.children.each(function(view){

  // делаем что-то с представлением

});
```

For more information on the available features and functionality of
the `.children`, see the [Backbone.BabySitter documentation](https://github.com/marionettejs/backbone.babysitter).


Большая информация доступна в [Backbone.BabySitter документации](https://github.com/marionettejs/backbone.babysitter).

## CollectionView close

## Закрытие CollectionView

CollectionView implements a `close` method, which is called by the 
region managers automatically. As part of the implementation, the 
following are performed:

CollectionView реализует метод `close`, вызываемый автоматически из region managers.
В частности, происходит следующее: 

* unbind all `listenTo` events
* unbind all custom view events
* unbind all DOM events
* unbind all item views that were rendered
* remove `this.el` from the DOM
* call an `onClose` event on the view, if one is provided

* удаляется подписка на все события, задействованные в `listenTo`
* удаляется подписка на все события, определенные пользователем
* удаляется подписка на все DOM-события
* удаляются все отрендеренные item views
* `this.el` удаляется из DOM
* вызывается метод `onClose` представления, если он определен 

By providing an `onClose` event in your view definition, you can
run custom code for your view that is fired after your view has been
closed and cleaned up. This lets you handle any additional clean up
code without having to override the `close` method.

Путем добавления обработчика события 'close' - метода `onClose` в представление,
можно запустить пользовательский код с какими-то дополнительными действиями (например, по очистке ресурсов),
который выполнится, когда представление будет закрыто.

```js
Backbone.Marionette.CollectionView.extend({
  onClose: function(){
    // здесь код, который должен быть выполнен после закрытия представления 
  }
});
```

