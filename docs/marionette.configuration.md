# Marionette Configuration

Marionette has a few globally configurable settings that will
change how the system works. While many of these subjects are covered
in other docs, this configuration doc should provide a list of the
most common items to change.

У Marionette есть несколько глобальных настроек в конфигурации, которые определяют работу системы.


## Documentation Index

* [Marionette.$](#marionette_)

## Marionette.$

Marionette makes use of jQuery, by default, to manipulate DOM
elements. To get a reference to jQuery, though, it assigns the
`Marionette.$` attribute to `Backbone.$`. This provides consistency
with Backbone in which exact version of jQuery or other DOM manipulation
library is used.

По умолчанию, Marionette использует всю мощь jQuery для манипуляций с элементами DOM.
Но поскольку Marionette является надстройкой над Backbone, для соответствия Backbone, в Marionette используется та же самая версия jQuery.

If you wish to change to a specific version of a DOM manipulation
library, you can directly assign these settings:

Если необходимо изменить версию специфическую версию библиотеки для взаимодействия с DOM, нужно напрямую установить следующие настройки:

```js
Backbone.$ = myDOMLib;
Mariоnеtte.$ = myDOMLib;
```

То-есть одновременно настроить и Backbone и Marionette для работы с конкретной DOM-ориентированной библиотекой.
