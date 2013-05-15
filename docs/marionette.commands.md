# Marionette.commands

An application level command execution system. This allows components in
an application to state that some work needs to be done, but without having
to be explicitly coupled to the component that is performing the work.

Слой, представляющий в приложении систему исполняемых команд.
Обеспечивает взаимодействие слабосвязанных компонентов. Команды описывают определенный объем работ, но без явного указания компонента-исполнителя.

No response is allowed from the execution of a command. It's a "fire-and-forget"
scenario.

Выполнение команды не предусматривает обязательный ответ, что описывается типом взаимодействия "заявил и забыл".

Facilitated by [Backbone.Wreqr](https://github.com/marionettejs/backbone.wreqr)'s 
Commands object.

Происходит от командного объекта [Backbone.Wreqr](https://github.com/marionettejs/backbone.wreqr).

## Documentation Index

* [Register A Command](#register-a-command)
* [Execute A Command](#execute-a-command)
* [Remove / Replace Commands](#remove--replace-commands)

## Register A Command

## Регистрация команды

To register a command, call `App.commands.setHandler` and provide a name for
the command to handle, and a callback method.

Чтобы зарегистрировать подписку на выполнение команды, вызовите `App.commands.setHandler` с указанием имени команды и обрабатывающего метода.

```js
var App = new Marionette.Application();

App.commands.setHandler("foo", function(bar){
  console.log(bar);
});
```

## Execute A Command

## Запуск команды на выполнение

To execute a command, either call `App.commands.execute` or the more direct
route of `App.execute`, providing the name of the command to execute and
any parameters the command needs:

Чтобы выполнить команду, вызовите `App.commands.execute` (есть еще более прямой способ - вызов `App.execute`), передавая в качестве аргуметов
название команды и дополнительные параметры, необходимые для выполнения команды:


```js
App.execute("foo", "baz");
//  "baz" выведется в консоли, сработает зарегистрированный для данной команды выше обработчик
```

## Remove / Replace Commands

## Удаление / замена команды

To remove a command, call `App.commands.removeHandler` and provide the
name of the command to remove. 

Чтобы удалить зарегистрированную команду, вызовите `App.commands.removeHandler` с указанием имени удаляемой команды.

To remove all commands, call `App.commands.removeAllHandlers()`.

Чтобы удалить все зарегистрированные команды, вызовите `App.commands.removeAllHandlers()`.

To replace a command, simply register a new handler for an existing
command name. There can be only one command handler for a given command name.

Чтобы заменить команду, просто зарегистрируйте новый обработчик для данного названия команды. Каждой команде может соответствовать только один обработчик.
