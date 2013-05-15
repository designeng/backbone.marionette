# Marionette.CompositeView

A `CompositeView` extends from `CollectionView` to be used as a 
composite view for scenarios where it should represent both a 
branch and leaf in a tree structure, or for scenarios where a
collection needs to be rendered within a wrapper template.

`CompositeView` [происходит от `CollectionView`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.compositeview.js#L7), его стоит использовать в деревовидных сценариях, [где нужно отобразить одновременно лист и ветвь](http://lostechies.com/derickbailey/2012/04/05/composite-views-tree-structures-tables-and-more/), или там, где коллекция должна быть отрендерена с применением оборачивающего шаблона.




For example, if you're rendering a treeview control, you may 
want to render a collection view with a model and template so 
that it will show a parent item with children in the tree.

К примеру, если рендерится представление для компонента "tree", может понадобиться рендеринг collection view, где модель и шаблон
устроены таким образом, чтобы компонент имел возможность отбражать все внутренние ветки (деревья).

You can specify a `modelView` to use for the model. If you don't
specify one, it will default to the `Marionette.ItemView`.

Тут можно [предложить реализовать `modelView`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.compositeview.js#L6) в качестве модели,
но если в вашем проекте нет аналогичной структуры, [по умолчанию в этом качестве выступит `Marionette.ItemView`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.compositeview.js#L11).

```js
CompositeView = Backbone.Marionette.CompositeView.extend({
  template: "#leaf-branch-template"
});

new CompositeView({
  model: someModel,
  collection: someCollection
});
```

For more examples, see my blog post on 
[using the composite view.](http://lostechies.com/derickbailey/2012/04/05/composite-views-tree-structures-tables-and-more/)

## Documentation Index

* [Composite Model `template`](#composite-model-template)
* [CompositeView's `itemViewContainer`](#compositeviews-itemviewcontainer)
* [CompositeView's `appendHtml`](#compositeviews-appendhtml)
* [Recursive By Default](#recursive-by-default)
* [Model And Collection Rendering](#model-and-collection-rendering)
* [Events And Callbacks](#events-and-callbacks)
* [Organizing UI elements](#organizing-ui-elements)
* [modelEvents and collectionEvents](#modelevents-and-collectionevents)

## Composite Model `template`

## Шаблон композиционной модели

When a `CompositeView` is rendered, the `model` will be rendered
with the `template` that the view is configured with. You can
override the template by passing it in as a constructor option:

В процессе отображения `CompositeView`  на странице, модель будет отрендерена с применением [шаблона](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.compositeview.js#L91) (`template`), описанного в конфигурации.

```js
new MyComp({
  template: "#some-template"
});
```

## CompositeView's `itemViewContainer`

## CompositeView: `itemViewContainer`

By default the composite view uses the same `appendHtml` method that the
collection view provides. This means the view will call jQuery's `.append` 
to move the HTML contents from the item view instance in to the collection
view's `el`. 

По умолчанию, composite view использует тот же самый метод [`appendHtml`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.compositeview.js#L95-L102), [который представлен в collection view](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.collectionview.js#L229-L234).

This is typically not very useful as a composite view will usually render
a container DOM element in which the item views should be placed.

Впрочем, пользы от этого немного, поскольку composite view обыкновенно использует в качестве контейнера для отображения DOM-элемент,
в котором развернутся его встроенные item views. 

For example, if you are building a table view, and want to append each
item from the collection in to the `<tbody>` of the table, you might
do this with a template:

К примеру, если вы проектируете представление табличного вида, и хотите добавлять элементы коллекции внутрь `<tbody>`, вы можете сделать в шаблоне следующее:

```html
<script id="row-template" type="text/html">
  <td><%= someData %></td>
  <td><%= moreData %></td>
  <td><%= stuff %></td>
</script>

<script id="table-template" type="text/html">
  <table>
    <thead>
      <tr>
        <th>Some Column</th>
        <th>Another Column</th>
        <th>Still More</th>
      </tr>
    </thead>

    <!-- сюда мы хотим помещать элементы коллекции -->
    <tbody></tbody>

    <tfoot>
      <tr>
        <td colspan="3">some footer information</td>
      </tr>
    </tfoot>
  </table>
</script>
```

To get your itemView instances to render within the `<tbody>` of this
table structure, specify an `itemViewContainer` in your composite view,
like this:

Чтобы заставить экземпляры itemView отображаться внутри `<tbody>` имеющейся табличной структуры,
определите атрибут `itemViewContainer` в composite view, примерно так:

```js
RowView = Backbone.Marionette.ItemView.extend({
  tagName: "tr",
  template: "#row-template"
});

TableView = Backbone.Marionette.CompositeView.extend({
  itemView: RowView,

  // определяем jQuery-селектор для размещения экземпляров itemView в соответствующем элементе:
  itemViewContainer: "tbody",

  template: "#table-template"
});
```

This will put all of the `itemView` instances in to the `<tbody>` tag of
the composite view's rendered template, correctly producing the table
structure.

Alternatively, you can specify a function as the `itemViewContainer`. This
function needs to return a jQuery selector string, or a jQuery selector
object.

В качестве альтернативы, вы можете определить `itemViewContainer` как функцию.
Она должна возвращать [либо jQuery-селектор в строковом представлении, либо jQuery-объект](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.compositeview.js#L114).

```js
TableView = Backbone.Marionette.CompositeView.extend({
  // ...

  itemViewContainer: function(){
    return "#tbody"
  }
});
```

Using a function allows for logic to be used for the selector. However,
only one value can be returned. Upon returning the first value, it will
be cached and that value will be used for the remainder of that view
instance' lifecycle.

Если использовать функцию, можно поэкспериментировать с возвращаемым значением. Однако следует заметить, что [значение будет возвращено только единожды: оно кешируется](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.compositeview.js#L107-L109) и все дальнейшие обращения к `itemViewContainer` в течение жизненного цикла представления будут иметь дело уже с этим закешированным значением.


## CompositeView's `appendHtml`

## Метод `appendHtml` CompositeView


Sometimes the `itemViewContainer` configuration is insuficient for
specifying where the itemView instance should be placed. If this is the
case, you can override the `appendHtml` method with your own implementation.

Иногда конфигурации `itemViewContainer` недостаточно для определения, где должен быть размещен экземпляр itemView.
В этом случае стоит переопределить метод [`appendHtml`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.compositeview.js#L95-L102).

Например:

```js
TableView = Backbone.Marionette.CompositeView.extend({
  itemView: RowView,

  template: "#table-template",

  appendHtml: function(collectionView, itemView, index){
    collectionView.$("tbody").append(itemView.el);
  }
});
```

For more information on the parameters of this method, see the
[CollectionView's documentation](https://github.com/marionettejs/backbone.marionette/blob/master/docs/marionette.collectionview.md).

## Recursive By Default

## Рекурсия по умолчанию

The default rendering mode for a `CompositeView` assumes a
hierarchical, recursive structure. If you configure a composite
view without specifying an `itemView`, you'll get the same
composite view type rendered for each item in the collection. If
you need to override this, you can specify a `itemView` in the
composite view's definition:

В обычном режиме отображения, принятом по умолчанию, `CompositeView` предполагает иерархическую рекурсивную структуру.
Если сконфигурировать composite view без специфического `itemView`, мы получим однотипное отображение composite view для всех
элементов коллекции (чтобы понять, что имеется в виду, можно [поэкспериментировать с GridView](http://jsfiddle.net/derickbailey/me4NK/) - прим.пер.).
Если необходимо переопределить данное поведение, нужно задать `itemView` в определении composite view:


```js
var ItemView = Backbone.Marionette.ItemView.extend({});

var CompView = Backbone.Marionette.CompositeView.extend({
  itemView: ItemView
});
```

## Model And Collection Rendering

## Отображение модели и коллекции

The model and collection for the composite view will re-render
themselves under the following conditions:

Модель и коллекция composite view заключает в себе следующие предпосылки для повторного отображения: 

* When the collection's "reset" event is fired, it will only re-render the collection within the composite, and not the wrapper template
* When the collection has a model added to it (the "add" event is fired), it will render that one item to the rendered list
* When the collection has a model removed (the "remove" event is fired), it will remove that one item from the rendered list

* Когда происходит событие "reset" коллекции, коллекция будет отображена заново внутри композиции, но не внутри шаблона-обертки (???)
* Когда в коллекцию добавляется новая модель (и в коллекции порождается событие "add"), только один соответствующий модели элемент будет повторно отображен
* Когда модель удаляется из коллекции (порождается событие "remove"), только один соответствующий модели элемент будет удален из списка отображения

You can also manually re-render either or both of them:

Вы также можете запускать рендеринг вручную:


* If you want to re-render everything, call the `.render()` method
* If you want to re-render the model's view, you can call `.renderModel()`


* Если вы хотите заново отрендерить весь composite view, вызовите метод [`.render()`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.compositeview.js#L52-L74)
* Если вы хотите заново отрендерить представление, соответствующее обособленной модели, вызовите метод [`.renderModel()`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.compositeview.js#L83-L93)

## Events And Callbacks

## События и коллбэки

During the course of rendering a composite, several events will
be triggered. These events are triggered with the [Marionette.triggerMethod](./marionette.functions.md)
function, which calls a corresponding "on{EventName}" method on the view.

В процессе рендеринга композиции, могут происходить некоторые события. Все они [порождаются](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.compositeview.js#L67-L79) с помощью функции [Marionette.triggerMethod](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.triggermethod.js#L8-L33), которая пытается вызвать соответствующие методы "on{EventName}", если они определены в базовом представлении.

* "composite:model:rendered" / `onCompositeModelRendered` - after the `modelView` has been rendered
* "composite:collection:rendered" / `onCompositeCollectionRendered` - after the collection of models has been rendered
* "render" / `onRender` and "composite:rendered" / `onCompositeRendered` - after everything has been rendered

* событие "composite:model:rendered"  (метод `onCompositeModelRendered`) - произойдет после того, как `modelView` отобразится
* событие "composite:collection:rendered" (метод `onCompositeCollectionRendered`) - произойдет после того, как отобразится коллекция моделей 
* события "render" / (метод `onRender`) и "composite:rendered" (метод `onCompositeRendered`) - произойдет после того, как все будет отображено

Additionally, after the composite view has been rendered, an 
`onRender` method will be called. You can implement this in 
your view to provide custom code for dealing with the view's 
`el` after it has been rendered:

Дополнительно, после того как composite view будет отображен, вызовется метод `onRender`. Вы можете реализовать его в представлении, например, чтобы взаимодействовать с объектом `el` представления, который к этому времени уже будет отображен:

```js
Backbone.Marionette.CompositeView.extend({
  onRender: function(){
    // делаем что-то
  }
});
```

## Organizing UI elements

## Организация UI элементов

Similar to ItemView, you can organize the UI elements inside the
CompositeView by specifying them in the `UI` hash. It should be
noted that the elements that can be accessed via this hash are
the elements that are directly rendered by the composite view
template, not those belonging to the collection.

Подобно ItemView, можно организовать UI элементы внутри CompositeView, определив их в `UI` хеше. Уточним, что элементы, которые доступны через данный хеш, это те самые элементы, которые напрямую внедряются в страницу в шаблоне CompositeView, а не те, что принадлежат коллекции.

The UI elements will be accessible as soon as the composite view
template is rendered (and before the collection is rendered),
which means you can even access them in the `onBeforeRender` method.

UI элементы будут доступны сразу, как только шаблон composite view [отобразится на странице](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.compositeview.js#L63-L66) (и прежде чем отрендерится коллекция).

## modelEvents and collectionEvents

## modelEvents и collectionEvents

CompositeViews can bind directly to model events and collection events
in a declarative manner:

CompositeViews также поддерживает привязку методов к событиям модели и коллекции в декларативной манере:

```js
Marionette.CompositeView.extend({
  modelEvents: {
    "change": "modelChanged"
  },

  collectionEvents: {
    "add": "modelAdded"
  }
});
```

For more information, see the [Marionette.View](./marionette.view.md) documentation.
