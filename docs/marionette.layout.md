# Marionette.Layout

A `Layout` is a specialized hybrid between an `ItemView` and
a collection of `Region` objects, used for rendering an application
layout with multiple sub-regions to be managed by specified region managers.

`Layout` - это специальный гибрид, представляющий собой нечто промежуточное между `ItemView` и коллекцией из `Region` объектов,
который используется для формирования макета приложения со множеством суб-регионов, которые будут управляться с помощью определенных Marionette.RegionManager.

A layout manager can also be used as a composite-view to aggregate multiple
views and sub-application areas of the screen where multiple region managers need
to be attached to dynamically rendered HTML.

Layout может быть включен в приложение в качестве управляющей структуры, и использован вместо `composite-view` для 
работы с областями экрана, относящимися к подчиненным приложениям (sub-application), в которых реализуется
отображение множества представлений внутри регионов, посредством динамически отображаемого HTML.


For a more in-depth discussion on Layouts, see the blog post
[Manage Layouts And Nested Views With Backbone.Marionette](http://lostechies.com/derickbailey/2012/03/22/managing-layouts-and-nested-views-with-backbone-marionette/)

## Documentation Index

* [Basic Usage](#basic-usage)
* [Region Availability](#region-availability)
* [Re-Rendering A Layout](#re-rendering-a-layout)
  * [Avoid Re-Rendering The Entire Layout](#avoid-re-rendering-the-entire-layout)
* [Nested Layouts And Views](#nested-layouts-and-views)
* [Closing A Layout](#closing-a-layout)
* [Custom Region Type](#custom-region-type)

## Basic Usage

## Базовое применение

The `Layout` extends directly from `ItemView` and adds the ability
to specify `regions` which become `Region` instances that are attached
to the layout.

`Layout` происходит (extends) непосредственно от `ItemView` и привносит возможность определять регионы (`regions`) - экземпляры `Region`,
прикрепленные к layout-у.

```html
<script id="layout-template" type="text/template">
  <section>
    <navigation id="menu">...</navigation>
    <article id="content">...</article>
  </section>
</script>
```

```js
AppLayout = Backbone.Marionette.Layout.extend({
  template: "#layout-template",

  regions: {
    menu: "#menu",
    content: "#content"
  }
});

var layout = new AppLayout();
layout.render();
```

Once you've rendered the layout, you now have direct access
to all of the specified regions as region managers.

Поскольку layout определен, возможен прямой доступ ко всем зарегистированным в нем регионам, играющим роль region managers.

```js
layout.menu.show(new MenuView());

layout.content.show(new MainContentView());
```

### Specifying Regions As A Function

### Задание Regions посредством функции

Regions can be specified on a Layout using a function that returns
an object with the region definitions. The returned object follow the
same rules for defining a region, as outlined above.

Регионы могут быть заданы внутри Layout-а посредством функции, возвращающей объект
с дифиницией региона. Данный объект определяет регион по тем же правилам, что и описанный выше объектный литерал. 

```js
Marionette.Layout.extend({
  // ...

  regions: function(options){
    return {
      fooRegion: "#foo-element"
    };
  },

  // ...
});
```

Note that the function recieves the view's `options` arguments that 
were passed in to the view's constructor. `this.options` is not yet
available when the regions are first initialized, so the options
must be accessed through this parameter.

Обратите внимание, что функция принимает `options` в качестве аргумента, 
которые были переданы в конструктор view. Когда регион впервые инициализован,
значение `this.options` еще не доступно, так что options должны быть получены из этого параметра.

## Region Availability

## Доступ к региону

Any defined regions within a layout will be available to the
layout or any calling code immediately after instantiating the
layout. This allows a layout to be attached to an existing 
DOM element in an HTML page, without the need to call a render
method or anything else, to create the regions.

Любой определенный внутри layout-а регион будет доступен этому layout-у
или любому другому коду сразу после того, как layout был инициализован.
Благодаря этому любой layout может быть вставлен в любой DOM-элемент на HTML странице,
со всеми определенными в нем регионами, готовыми к работе, без необходимости вызова метода render() или каких-либо других 
дополнительных действий.


However, a region will only be able to populate itself if the
layout has access to the elements specified within the region
definitions. That is, if your view has not yet rendered, your
regions may not be able to find the element that you've
specified for them to manage. In that scenario, using the
region will result in no changes to the DOM.

Однако, следует заметить, что регион может быть наполнен содержанием только в том случае,
если layout имеет доступ к элементу, определенному при дифиниции региона.
То-есть это значит, что если представление еще не отрендерилось, регионы могут не найти соответствующие
им DOM-элементы, и связанный с ними сценарий не внесет никаких изменений в DOM.

## Re-Rendering A Layout

## Повторный рендеринг layout-a

A layout can be rendered as many times as needed, but renders
after the first one behave differently than the initial render.

Layout может быть отрендерен столько раз, сколько это необходимо,
но инициализирующий render и все последующие имеют в своем поведении некоторые отличия.

The first time a layout is rendered, nothing special happens. It just
delegates to the `ItemView` prototype to do the render. After the
first render has happened, though, the render function is modified to
account for re-rendering with regions in the layout.

Первый раз, когда layout отрендерен, ничего особенного не происходит. Единственно -  
[вызывается метод render() прототипа `ItemView` на данном layout-е](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.layout.js#L45).

В следующий раз, когда нужно будет отрендерить layout, 
[метод render(), кроме вышеназванного, запустит код первичной или повторной инициализации регионов](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.layout.js#L34-42).

After the first render, all subsequent renders will force every
region to close by calling the `close` method on them. This will
force every view in the region, and sub-views if any, to be closed
as well. Once the regions have been closed, the regions will be
reset so that they are no longer referencing the element of the previous
layout render. 

После первого рендеринга каждый последующий будет принудительно вызывать метод close() для каждого региона, закрывая его.
Это также повлечет закрытие каждого присутствующего (показанного, отрисованного) в регионе представления (view),
и каждого связанного с этим представлением sub-view. Как только регионы окажутся закрыты,  
[все регионы будут сброшены при помощи свойственного им метода reset()](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.layout.js#L106-108), [чтобы они больше не ссылались на элементы предыдущего рендеринга](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L180-183).

Then after the Layout is finished re-rendering itself,
showing a view in the layout's regions will cause the regions to attach
themselves to the new elements in the layout.

И уже когда layout закончит процесс собственного ре-рендеринга, вызов метода show для каждого региона и связанный с ним показ представления
приведет к тому, что регион [свяжет себя с новым элементом в layout-е](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.region.js#L149-150).

### Avoid Re-Rendering The Entire Layout

### Как избежать ререндеринга целого layout-а

There are times when re-rendering the entire layout is necessary. However,
due to the behavior described above, this can cause a large amount of
work to be needed in order to fully restore the layout and all of the
views that the layout is displaying.

Случается, что необходим повторный рендеринг всего layout-а.
Однако, как следует из вышеописанного, это может привести к чрезмерно большому количеству необходимой
для полной перезагрузки layout-а и всех отображенных в нем представлений работы.

Therefore, it is suggested that you avoid re-rendering the entire
layout unless absolutely necessary. Instead, if you are binding the
layout's template to a model and need to update portions of the layout,
you should listen to the model's "change" events and only update the
neccesary DOM elements.

Это значит, что вы должны избегать ре-рендеринга целого layout-а без крайней необходимости.
Вместо этого, вам следует привязать шаблон layout-а к модели и, прослушивая события изменения модели, 
обновлять layout небольшими порциями, работая только с необходимыми DOM-элементами.

## Nested Layouts And Views

## Встроенные Layouts и Views

Since the `Layout` extends directly from `ItemView`, it
has all of the core functionality of an item view. This includes
the methods necessary to be shown within an existing region manager.

Поскольку `Layout` происходит непосредственно от `ItemView`, он наследует всю базовую функциональность `ItemView`.
Что включает в себя и все методы, необходимые для показа внутри существующего region manager-а. (???)

```js
MyApp = new Backbone.Marionette.Application();
MyApp.addRegions({
  mainRegion: "#main"
});

var layout = new AppLayout();
MyApp.mainRegion.show(layout);

layout.show(new MenuView());
```

You can nest layouts into region managers as deeply as you want.
This provides for a well organized, nested view structure.

Ограничений на глубину встраивания layout-в в region managers нет, это дает возможность создавать структуры представлений любой степени сложности. 

## Closing A Layout

## Закрытие экземпляра layout-а

When you are finished with a layout, you can call the
`close` method on it. This will ensure that all of the region managers
within the layout are closed correctly, which in turn
ensures all of the views shown within the regions are closed correctly.

Когда вы заканчиваете работу с layout-ом, вы можете вызвать его метод `close`. Это будет гарантировать,
что все region managers внутри layout-а будут корректно выключены,
что в свою очередь гарантирует, что все views, показанные внутри регионов (см. метод show() региона)
будут выключены (закрыты) корректно.

If you are showing a layout within a parent region manager, replacing 
the layout with another view or another layout will close the current 
one, the same it will close a view.

Если вы показываете layout внутри родительского region manager-а, замена
layout-а другим layout-ом или другим view закроет текущий layout точно так же, как закрыл бы view.

All of this ensures that layouts and the views that they
contain are cleaned up correctly. 

Вышеописанное поведение гарантирует корректное закрытие layout-в и содержащихся в них представлений.

## Custom Region Type

## Пользовательские типы регионов

If you have the need to replace the `Region` with a region class of
your own implementation, you can specify an alternate class to use
with the `regionType` propery of the `Layout`.

Если вам необходимо заменить `Region` с помощью реализованного вами класса,
вы можете указать этот класс посредством свойства `regionType` `Layout`-а:

```js
MyLayout = Backbone.Marionette.Layout.extend({
  regionType: SomeCustomRegion 
});
```

You can also specify custom `Region` classes for each `region`:

Вы можете также задавать пользовательские `Region`-классы для любого региона, определенного в Layout-е:

```js
AppLayout = Backbone.Marionette.Layout.extend({
  template: "#layout-template",

  regionType: SomeDefaultCustomRegion,

  regions: {
    menu: {
      selector: "#menu",
      regionType: CustomRegionTypeReference
    },
    content: {
      selector: "#content",
      regionType: CustomRegionType2Reference
    }
  }
});
```

## Adding And Removing Regions

## Добавление и удаление регионов

Regions can be added and removed as needed, in a
Layout instance. Use the following methods:

Регионы могут добавляться и удаляться по мере необходимости в экземпляре Layout-а с помощью следующих методов:

* `addRegion`
* `addRegions`
* `removeRegion`

addRegion:

```js
var layout = new MyLayout();

// ...

layout.addRegion("foo", "#foo");
layout.foo.show(new someView());
```

addRegions: 

```js
var layout = new MyLayout();

// ...

layout.addRegions({
  foo: "#foo",
  bar: "#bar"
});
```

removeRegions:

```js
var layout = new MyLayout();

// ...

layout.removeRegion("foo");
```

Any region can be removed, whether it was defined
in the `regions` attribute of the region definition,
or added later.

Любой регион может быть удален, независимо от того, был ли он определен в аттрибутах региона или в его дефиниции, или добавлен позже.

For more information on using these methods, see
the `regionManager` documentation.

За дальнейшей информацией по использованию этих методов, следует обращаться к документации по `Marionette.regionManager`.
