# Marionette.Renderer

The `Renderer` object was extracted from the `ItemView` rendering
process, in order to create a consistent and re-usable method of
rendering a template with or without data.

Объект `Renderer` был извлечен из процесса рендеринга `ItemView`,
чтобы обеспечить удобный и отчетливый метод рендеринга шаблона с данными или без них.



## Documentation Index

* [Basic Usage](#basic-usage)
* [Pre-compiled Templates](#pre-compiled-templates)
* [Custom Template Selection And Rendering](#custom-template-selection-and-rendering)
* [Using Pre-compiled Templates](#using-pre-compiled-templates)

## Basic Usage

## Базовое использование

The basic usage of the `Renderer` is to call the `render` method.
This method returns a string containing the result of applying the 
template using the `data` object as the context.

Базовое использование `Renderer`-а сводится к вызову метода `render`.
Этот метод возвращает [строку, содержащую результат обработки шаблона в контексте объекта `data`](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.renderer.js#L14).

```js
var template = "#some-template";
var data = {foo: "bar"};
var html = Backbone.Marionette.Renderer.render(template, data);

// делаем что-то с HTML
```

## Pre-compiled Templates

## Прекомпиляция шаблонов

If the `template` parameter of the `render` function is itself a function,
the renderer treats this as a pre-compiled template and does not try to
compile it again. This allows any view that supports a `template` parameter
to specify a pre-compiled template function as the `template` setting.

Если параметр `data` функции `render` сам является функцией, 
[renderer считает, что имеет дело с заранее компилированным шаблоном](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.renderer.js#L13), и не пытается откомпилировать его еще раз. Благодаря этому любое представление, 
в котором задан параметр `template`, будет брать в качестве его значения функцию, представляющую собой откомпилированный шаблон.

```js
var myTemplate = _.template("<div>foo</div>");
Backbone.Marionette.ItemView.extend({
  template: myTemplate
});
```

The template function does not have to be any specific template engine. It
only needs to be a function that returns valid HTML as a string from the
`data` parameter passed to the function.

Что касается функции, возвращающей откомпилированный шаблон, то нужно заметить, что
ее выбор не накладывает никаких ограничений - она может быть взята из api любого движка шаблонов.
Важно только, чтобы она действительно возвращала валидный HTML в качестве строки из переданных в нее данных.

## Custom Template Selection And Rendering

## Пользовательский механизм загрузки и рендеринга шаблонов

By default, the renderer will take a jQuery selector object as
the first parameter, and a JSON data object as the optional
second parameter. It then uses the `TemplateCache` to load the
template by the specified selector, and renders the template with
the data provided (if any) using Underscore.js templates.

По умолчанию, renderer [принимает](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.renderer.js#L12) jQuery-селектор в качестве первого параметра, и JSON-объект в качестве опционального второго параметра.
Почле чего он [загружает](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.templatecache.js#L65) шаблон из `Marionette.TemplateCache`, [пользуясь в качестве ключа указанным селектором](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.templatecache.js#L19-L28),
 и [рендерит](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.templatecache.js#L66) в шаблон предоставленные в качестве параметра данные, если таковые имеются, с помощью Underscore.js [микро-шаблонизатора](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.templatecache.js#L91).

If you wish to override the way the template is loaded, see
the `TemplateCache` object. 

Если есть необходимость [переопределить способ, которым загружается шаблон](https://github.com/marionettejs/backbone.marionette/blob/master/src/marionette.templatecache.js#L71-L84), вам следует обратить внимание на объект `TemplateCache`.

If you wish to override the template engine used, change the 
`render` method to work however you want:

Если же вы хотите назначить другой движок шаблонов, вместо используемого по умолчанию, 
переопределите метод `render`:


```js
Backbone.Marionette.Renderer.render = function(template, data){
  return $(template).tmpl(data);
});
```

This implementation will replace the default Underscore.js 
rendering with jQuery templates rendering.

В этом примере движок шаблонов Underscore.js заменен на аналогичный из jQuery.

If you override the `render` method and wish to use the 
`TemplateCache` mechanism, remember to include the code necessary to 
fetch the template from the cache in your `render` method:

Если вы переопредяете метод `render`, но, тем не менее, хотите продолжать использовать механизм `TemplateCache`,
не забудьте вставить строчку кода, отвечающую за загрузку шаблона из кеша:

```js
Backbone.Marionette.Renderer.render = function(template, data){
  var template = Marionette.TemplateCache.get(template);
  // делаем что-то с шаблоном...
};
```

## Using Pre-compiled Templates

## Использование прекомпилированных шаблонов

You can easily replace the standard template rendering functionality
with a pre-compiled template, such as those provided by the JST or TPL
plugins for AMD/RequireJS. 

Вы легко можете заменить стандартный механизм рендеринга шаблонов
на предварительно компилированные шаблоны, как их видят плагины JST или TPL для AMD/RequireJS.

(см. также дополнение к документации - ["Using marionette with requirejs"](https://github.com/marionettejs/backbone.marionette/wiki/Using-marionette-with-requirejs))

To do this, just override the `render` method to return your executed 
template with the data.

Просто переопределите метод `render`, чтобы он возвращал необходимую функцию, с переданными в нее данными:

```js
Backbone.Marionette.Renderer.render = function(template, data){
  return template(data);
});
```

Then you can specify the pre-compiled template function as your view's
`template` attribute:

После этого вы можете использовать прекомпилированную функцию для работы с шаблоном в качестве значения атрибута `template`:

```js
var myPrecompiledTemplate = _.template("<div>some template</div>");

Backbone.Marionette.ItemView.extend({
  template: myPrecompiledTemplate
});
```


