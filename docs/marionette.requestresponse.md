# Marionette.RequestResponse

An application level request/response system. This allows components in
an application to request some information or work be done by another
part of the app, but without having to be explicitly coupled to the 
component that is performing the work.

Представляет собой слой запрос-ответ (request/response) в приложении.
Позволяет компонентам приложения запрашивать какую-либо информацию у других компонентов, 
либо направлять запрос на совершение какой-либо работы другим частям приложения, без необходимости тесно связывать между собой части приложения.

A return response is expected when making a request.

Отправка запроса всегда предполагает собой получение ответа.



Facilitated by [Backbone.Wreqr](https://github.com/marionettejs/backbone.wreqr)'s 
RequestResponse object.

Произошел от объекта RequestResponse из [Backbone.Wreqr](https://github.com/marionettejs/backbone.wreqr)

## Documentation Index

* [Register A Request Handler](#register-a-request-handler)
* [Request A Response](#request-a-response)
* [Remove / Replace A Request Handler](#remove--replace-a-request-handler)

## Register A Request Handler

## Регистрация обработчика запроса

To register a command, call `App.reqres.setHandler` and provide a name for
the command to handle, and a callback method.

Чтобы зарегистрировать команду, вызовите `App.reqres.setHandler` и предоставьте в качестве аргументов название обрабатываемой команды и собственно сам обработчик.

```js
var App = new Marionette.Application();

App.reqres.setHandler("foo", function(bar){
  return bar + "-quux";
});
```

## Request A Response

## Отправка запроса

To execute a command, either call `App.reqres.request` or the more direct
route of `App.request`, providing the name of the command to execute and
any parameters the command needs:

Чтобы выполнить команду, нужно вызвать `App.reqres.request` или более прямой способ - `App.request`,
со следующими аргументами: названием команды и какими-либо необходимыми для запроса параметрами:

```js
App.request("foo", "baz"); // => вернет "baz-quux"
```

## Remove / Replace A Request Handler

## Удаление / замена обработчика

To remove a request handler, call `App.reqres.removeHandler` and provide the
name of the request handler to remove. 

Чтобы удалить обработчик запроса, вызовите `App.reqres.removeHandler` с названием команды, для которой следует удалить обработчик, в качестве аргумента.

To remove all request handlers, call `App.reqres.removeAllHandlers()`.

Чтобы удалить все существующие обработчики, нужно вызвать `App.reqres.removeAllHandlers()`.

To replace a request handler, simply register a new handler for an existing
request handler name. There can be only one request handler 
for a given request name.

Чтобы заменить обработчик запроса, просто зарегистрируйте новый обработчик для существующей команды. Каждой команде может соответствовать только один-единственный обработчик.
