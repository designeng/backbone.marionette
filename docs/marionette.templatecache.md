# Marionette.TemplateCache

The `TemplateCache` provides a cache for retrieving templates
from script blocks in your HTML. This will improve
the speed of subsequent calls to get a template.

Объект `TemplateCache` предоставляет кэш для загрузки шаблонов из блоков типа <script type = "text/template"></script>,
находящихся в HTML. Это увеличивает скорость обработки и внедрения шаблона при последующих обращениях к нему.


## Documentation Index

* [Basic Usage](#basic-usage)
* [Clear Items From cache](#clear-items-from-cache)
* [Customizing Template Access](#customizing-template-access)
* [Override Template Retrieval](#override-template-retrieval)
* [Override Template Compilation](#override-template-compilation)

## Basic Usage

## Базовое использование

To use the `TemplateCache`, call the `get` method on TemplateCache directly.
Internally, instances of the TemplateCache type will be created and stored
but you do not have to manually create these instances yourself. `get` will
return a compiled template function.

Чтобы начать использовать `TemplateCache`, просто прямо вызовите его [`get`-метод](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.templatecache.js#L19-L28).
Внутри механизма `TemplateCache`, экземпляры `TemplateCache`-а будут созданы и сохранены,
и вручную создавать их не придется. Метод `get` возвратит откомпилированный шаблон - то-есть превращенный в js-функцию.

```js
var template = Backbone.Marionette.TemplateCache.get("#my-template");
// используем шаблон
template({param1:'value1', paramN:'valueN'});
```

Making multiple calls to get the same template will retrieve the
template from the cache on subsequence calls.

Последующие обращения к одному и тому же шаблону будут возвращать уже эту откомпилированную функцию.

### Clear Items From cache

### Удаление элементов из кеша

You can clear one or more, or all items from the cache using the
`clear` method. Clearing a template from the cache will force it
to re-load from the DOM (via the `loadTemplate`
function which can be overridden, see below) the next time it is retrieved.

Можно по желанию удалять один или более элементов из кеша, с помощью метода `clear`.
Удаление шаблона из кеша повлечет его повторную загрузку из DOM при следующем к нему обращении (и очевидно, что функция `loadTemplate` может быть переопределена, как будет показано ниже).

If you do not specify any parameters, all items will be cleared
from the cache:

Функция clear() без параметров удалит из кеша все имеющиеся шаблоны:

```js
Backbone.Marionette.TemplateCache.get("#my-template");
Backbone.Marionette.TemplateCache.get("#this-template");
Backbone.Marionette.TemplateCache.get("#that-template");

// удаляем все
Backbone.Marionette.TemplateCache.clear()
```

If you specify one or more parameters, these parameters are assumed
to be the `templateId` used for loading / caching:

Задавая один параметр или более, можно очищать кеш от произвольного количества элементов:

```js
Backbone.Marionette.TemplateCache.get("#my-template");
Backbone.Marionette.TemplateCache.get("#this-template");
Backbone.Marionette.TemplateCache.get("#that-template");

// удаляем 2 экземпляра
Backbone.Marionette.TemplateCache.clear("#my-template", "#this-template")
```

## Customizing Template Access

## Кастомизация доступа к шаблонам

If you want to use an alternate template engine while
still taking advantage of the template caching functionality, or want to customize
how templates are stored and retreived, you will need to customize the
`TemplateCache object`. The default operation of `TemplateCache`, is to
retrive templates from the DOM based on the containing element's id
attribute, and compile the html in that element with the underscore.js
`template` function.

Если вы хотите использовать альтернативный движок шаблонов, и при этом иметь все те преимущества, которые предоставляет
механизм их кеширования, или вам нужно кастомизировать способ, которым шаблоны будут найдены и сохранены,
вы должны переопределить объект `TemplateCache`. По умолчанию, для извлечения шаблона из DOM, `TemplateCache` обращается к id атрибуту, 
а в качестве компилирующей html функции используется функция `template` из underscore.js.



### Override Template Retrieval

### Переопределение способа отыскания шаблона

The default template retrieval is to select the template contents
from the DOM using jQuery. If you wish to change the way this
works, you can override the `loadTemplate` method on the
`TemplateCache` object.

По умолчанию, `TemplateCache` находит шаблон в DOM [используя jQuery](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.templatecache.js#L77), но ничто не мешает действовать иначе: 

```js
Backbone.Marionette.TemplateCache.prototype.loadTemplate = function(templateId){

  // например, написать свою функцию создания шаблона по templateId и использовать ее здесь 
  var myTemplate = myTemplateFunc(templateId);

  // и возвращаем результат
  return myTemplate;
}
```

### Override Template Compilation

### Переопределение способа компиляции шаблона

The default template compilation passes the results from
`loadTemplate` to the `compileTemplate` function, which returns
an underscore.js compiled template function. When overriding `compileTemplate`
remember that it must return a function which takes an object of parameters and values
and returns a formatted HTML string.

По умолчанию, в процессе загрузки шаблона последовательно отрабатывают 
[`loadTemplate` и `compileTemplate`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.templatecache.js#L65-L66).
Это значит, что переопределяя метод [`compileTemplate`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.templatecache.js#L86-L92),
нужно позаботиться о том, чтобы он возвращал функцию, которая принимает некоторый объект с параметрами и значениями и возвращает строку, содержащую отформатированный HTML. 

```js
Backbone.Marionette.TemplateCache.prototype.compileTemplate = function(rawTemplate) {
  // применяем Handlebars.js для компиляции
  return Handlebars.compile(rawTemplate);
}
```


