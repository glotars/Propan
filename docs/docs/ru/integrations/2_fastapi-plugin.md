# **FastAPI** Plugin

### Прием сообщений

**Propan** может использоваться как полноценная часть **FastAPI**.

Для этого просто импортируйте нужный вам **PropanRouter** и объявите обработчик сообщений
с помощью декоратора `@event`. Этот декоратор аналогичен декоратору `@handle` у соответсвующих брокеров.

!!! tip
    При использовании таким образом **Propan** не использует собственную систему зависимостей, а интегрируется в **FastAPI**.
    Т.е. вы можете использовать `Depends`, `BackgroundTasks` и прочие инструменты **FastAPI** так, если бы это был обычный HTTP-endpoint.

    Обратите внимание, что в коде ниже используется `fastapi.Depends`, а не `propan.Depends`.

{! includes/integrations/fastapi/fastapi_plugin.md !}

При обработке сообщения из брокера все тело сообщения помещается одновременно и в `body`, и в `path` параметры запроса: вы можете достать получить к ним доступ любым удобным для вас способом. Заголовок сообщения помещается в `headers`.

Также этот роутер может полноценно использоваться как `HttpRouter` (наследником которого он и является). Поэтому вы можете
объявлять с его помощью любые `get`, `post`, `put` и прочие HTTP методы. Как например, это сделано в строке **19**.

### Отправка сообщений

Внутри каждого роутера есть соответсвующий брокер. Вы можете легко получить к нему доступ, если вам необходимо отправить сообщение в MQ.

{! includes/integrations/fastapi/fastapi_plugin_send.md !}

Вы можете оформить доступ к брокеру в виде `Depends`, если хотите использовать его в разных частях вашей программы.

{! includes/integrations/fastapi/fastapi_plugin_depends.md !}