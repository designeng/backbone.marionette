# Marionette.RegionManager

Region managers provide a consistent way to manage 
a number of Marionette.Region objects within an
application. The RegionManager is intended to be
used by other objects, to facilitate the addition,
storage, retrieval, and removal of regions from
that object. For examples of how it can be used,
see the Marionette.Application and Marionette.Layout
objects.

RegionManager предоставляет внятный способ управления объектами Marionette.Region в приложении.
RegionManager предназначается для использования другими объектами приложения, для облегчения
хранения, поиска и удаления регионов из них. Примеры использования находятся в Marionette.Application и Marionette.Layout.

## Documentation Index

* [Basic Use](#Basic Use)
* [RegionManager.addRegion](#regionmanager-addregion)
* [RegionManager.addRegions](#regionmanager-addregions)
  * [addRegions default options](#addregoins-default-options)
* [RegionManager.get](#regionmanager-get)
* [RegionManager.removeRegion](#regionmanager-removeregion)
* [RegionManager.removeRegions](#regionmanager-removeregions)
* [RegionManager.closeRegions](#regionmanager-closeregions)
* [RegionManager.close](#regionmanager-close)
* [RegionManager Events](#regionmanager-events)
  * [region:add event](#region-add-event)
  * [region:remove event](#region-remove-event)
* [RegionManager Iterators](#regionmanager-iterators)

## Basic Use

## Основы использования

RegionManagers can be instantiated directly, and can
have regions added and removed via several methods:

RegionManagers инстанциируется явным образом, посредством ключевого слова new, и к нему могут
быть добавлены регионы - с помощью нескольких очевидных методов:

```js
var rm = new Marionette.RegionManager();

var region = rm.addRegion("foo", "#bar");

var regions = rm.addRegions({
  baz: "#baz",
  quux: "ul.quux"
});

regions.baz.show(myView);

rm.removeRegion("foo");
```

## RegionManager.addRegion

Regions can be added individually using the `addRegion`
method. This method takes two parameters: the region name
and the region definition.

Данный метод принимает два параметра - имя региона
и его определение:

```js
var rm = new Marionette.RegionManager();

var region = rm.addRegion("foo", "#bar");
```

In this example, a region named "foo" will be added
to the RegionManager instance. It is defined as a
jQuery selector that will search for the `#bar`
element in the DOM. 

В этом примере к экземпляру RegionManager-а будет добавлен
регион с именем "foo", определенный через jQuery-селектор.
В DOM ему будет соответствовать элемент `#bar`.

There are a lot of other ways to define a region,
including object literals with various options, and
instances of Region objects. For more information 
on this, see the Region documentation.

Существует много других способов добавления региона,
включая объектные литералы с различными параметрами,
и экземпляры Region-а (см. документацию по Marionette.Region) 

## RegionManager.addRegions

Regions can also be added en-masse through the use
of the `addRegions` method. To use this method,
pass an object literal that contains the names of
the regions as keys, and the region definitions as
values.

Регионы также могут быть массово добавлены методом `addRegions`,
который принимает в качестве параметра объектный литерал, содержащий 
ключи - имена регионов, и значения - их дифиниции.

```js
var rm = new Marionette.RegionManager();

var regions = rm.addRegions({
  foo: "#bar",
  baz: {
    selector: "#quux",
    type: MyRegionType
  }
});

regions.foo; //=> экземпляр "foo" 
regions.bar; //=> экземпляр "bar" 
```

This method returns an object literal that contains
all of the named region instances. 

Этот метод возвращает [объектный литерал, содержащий имена всех регионов](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.regionManager.js#L32).

### addRegions default options

### addRegions параметры по умолчанию

When adding multiple regions it may be useful to
provide a set of defaults that get applied to all 
of the regions being added. This can be done through
the use of a `defaults` parameter. Specify this
parameter as an object literal with `key: value`
pairs that will be applied to every region added.

При добавлении множества регионов в RegionManager может оказаться полезным
предоставлять ряд значений по умолчанию, которые могут быть применены ко всем добавляемым регионам.
Это может быть сделано посредством параметра `defaults`, который представляет собой объектный литерал
с парами ключ-значение - все они, как было сказано, имеют отношение к любому из добавляемых регионов.


```js
var rm = new Marionette.RegionManager();

var defaults = {
  type: MyRegionType
};

var regions = {
  foo: "#bar",
  baz: "#quux"
};

rm.addRegions(regions, defaults);
```

In this example, all regions will be added as
instances of `MyRegionType`.

В этом примере все регионы будут добавлены как экземпляры типа `MyRegionType`.

## RegionManager.get

A region instance can be retrieved from the 
RegionManager instance using the `get` method and
passing in the name of the region.

Получить экземпляр региона можно с помощью метода `get`, которому задано в качестве параметра имя данного региона.


```js
var rm = new Marionette.RegionManager();
rm.addRegion("foo", "#bar");

var region = rm.get("foo");
```

## RegionManager.removeRegion

A region can be removed by calling the `removeRegion`
method and passing in the name of the region.

Так же просто регион может быть удален: 

```js
var rm = new Marionette.RegionManager();
rm.addRegion("foo", "#bar");

rm.removeRegion("foo");
```

A region will have it's `close` method called before
it is removed from the RegionManager instance. 

Прежде чем регион будет уделен из экземпляра RegionManager-а,
сработает его метод `close`.

## RegionManager.removeRegions

You can quickly remove all regions from the
RegionManager instance by calling the `removeRegions`
method.

Можно быстро удалить все имеющиеся в экземпляре RegionManager-а регионы с помощью команды `removeRegions`:

```js
var rm = new Marionette.RegionManager();
rm.addRegions({
  foo: "#foo",
  bar: "#bar",
  baz: "#baz"
});

rm.removeRegions();
```

This will close all regions, and remove them.

Которая, соответственно, сначала закроет регионы, а потом удалит их.

## RegionManager.closeRegions

You can quickly close all regions from the RegionManager
instance by calling the `closeRegions` method.

Можно также быстро закрыть все имеющиеся в экземпляре RegionManager-а регионы с помощью команды `closeRegions`:

```js
var rm = new Marionette.RegionManager();
rm.addRegions({
  foo: "#foo",
  bar: "#bar",
  baz: "#baz"
});

rm.closeRegions();
```

This will close the regions without removing them
from the RegionManager instance.

Эта команда закроет все регионы без удаления их из экземпляра RegionManager-а.

## RegionManager.close

A RegionManager instance can be closed entirely by
calling the `close` method. This will both close
and remove all regions from the RegionManager instance.

Экземпляр RegionManager-а может напрямую быть закрыт посредством вызова его метода `close`.
Таким образом будут закрыты и удалены все его регионы.

```js
var rm = new Marionette.RegionManager();
rm.addRegions({
  foo: "#foo",
  bar: "#bar",
  baz: "#baz"
});

rm.close();
```

## RegionManager Events

A RegionManager will trigger various events as it
is being used.

По ходу работы с экземпляром RegionManager-а он будет порождать ряд различных событий. 

### Событие region:add 

The RegionManager will trigger a "region:add"
event when a region is added to the manager. This
allows you to use the region instance immediately,
or attach the region to an object that needs a
reference to it:

Событие "region:add" будет порождено, когда регион добавлен. 
Это позволит немедленно пускать в дело экземпляр региона, или, например,
прикреплять регион к объекту, для сохранения имени данного региона в качестве ссылки на него:

```js
var rm = new Marionette.RegionManager();

rm.on("region:add", function(name, region){

  // add the region instance to an object
  myObject[name] = region;

});

rm.addRegion("foo", "#bar");
```

### Событие region:remove

The RegionManager will trigger a "region:remove"
event when a region is removed from the manager. 
This allows you to use the region instance one last
time, or remove the region from an object that has a
reference to it:

Соответственно, событие "region:remove" произойдет при удалении региона:

```js
var rm = new Marionette.RegionManager();

rm.on("region:remove", function(name, region){
  delete myObject[name];
});

rm.addRegion("foo", "#bar");

rm.removeRegion("foo");
```

## RegionManager Iterators

## RegionManager итераторы

The RegionManager has several methods for iteration
attached to it, from underscore.js. This works in the
same way as the Backbone.Collection methods that have
been imported. For example, you can easily iterate over
the entire collection of region instances by calling
the `each` method:

RegionManager наследует от underscore.js ряд методов для перебора объектов.
Они работают в том же ключе, что и методы Backbone.Collection, также импортированные из underscore.
Например, можно перебрать коллекцию экземпляров регионов с помощью метода `each`:

```js
var rm = new Marionette.RegionManager();

rm.each(function(region){
  // do stuff w/ the region instance here
});
```

The list of underscore methods includes:

Список методов underscore:

* forEach
* each
* map
* find
* detect
* filter
* select
* reject
* every
* all
* some
* any
* include
* contains
* invoke
* toArray
* first
* initial
* rest
* last
* without
* isEmpty
* pluck
