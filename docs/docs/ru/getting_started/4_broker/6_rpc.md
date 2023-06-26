# RPC

## Зачем использовать RPC over MQ

Иногда вам может понадобиться не просто отправить сообщение, но и получить на него ответ.
Обычно для этого используют HTTP, однако у нас уже есть система доставки сообщений, почему бы нам не использовать ее?

**RPC** запросы поверх брокеров сообщений выполняются очень просто: мы отправляем сообщение в одну очередь, а ответ получаем в другую.
Выглядит несколь топорно, однако такой подход несет в себе некоторые преимущества.

1. **Время** между вопросом и ответом ничем не ограничено: мы можем отправить запрос, а ответ получить через сутки. HTTP запрос таких волностей нам не позволяет.
    Это может быть крайне полезно для сервисов, который выполняют продолжительную работу: обрабатывают файлы, запускают нейросети и т.д.
2. **Асинхронность**: мы сами можем решать, ждать нам ответ прямо сейчас или просто отправить запрос, а обработать, когда он будет готов.
3. **Один запрос - много ответов**: используя брокеры сообщений мы вполне можем получить множество ответов на один запрос. Например, запросом мы можем инициализировать канал связи, по которому обратно будут посылаться данные по мере готовности.

## Реализация

### Сервер

Со стороны сервера (принимающей стороны) вам не нужно никак изменять код: `return` вашей функции автоматически будет отправлен клиенту, если он ожидает ответ на ваше сообщение.

!!! note
    Результат вашей функции должен соответствовать допустимым типам параметра `message` функции `broker.publish`.

    Допустимые типы: `str`, `dict`, `Sequence`, `pydantic.BaseModel`, `bytes`, а также нативные сообщения используемой для брокера библиотеки.

{! includes/getting_started/broker/rpc/1_handler.md !}

### Клиент

#### Блокирующий запрос

Для того, чтобы дождаться результата выполнения запроса "прямо здесь" (как если бы это был HTTP запрос) необходимо просто указать
параметер `callback=True` при отправке сообщения.

{! includes/getting_started/broker/rpc/2_blocking_client.md !}

Для установки времени, которое клиент готов ожидать ответ от сервера, используйте параметр `callback_timeout` (по умолчанию - **30** секунд)

```python linenums="1" hl_lines="4"
{!> docs_src/quickstart/broker/rpc/3_blocking_client_timeout.py !}
```

1. Ожидает результат выполнения 3 секунды

Если вы готовы ждать ответ столько, сколько это понадобится вы можете выставить `callback_timeout=None`

```python linenums="1" hl_lines="4"
{!> docs_src/quickstart/broker/rpc/4_blocking_client_timeout_none.py !}
```

!!! warning
    Этот код будет ожидать ответ бесконечно, даже если сервер не сможет обработать сообщение или обработка занимает длительное время

По умолчанию, если **Propan** не дождался ответа сервера, функция вернет `None`. Если же вы хотите явным образом обработать `asyncio.TimeoutError`,
используйте параметр `raise_timeout`.

```python linenums="1" hl_lines="4"
{!> docs_src/quickstart/broker/rpc/5_blocking_client_timeout_error.py !}
```

#### Неблокирующий запрос

Для того, чтобы обработать ответ вне основного потока выполнения, вы можете просто инициализировать обработчик, а затем передать его адрес в качестве `reply_to` аргумента запроса.

{! includes/getting_started/broker/rpc/6_noblocking_client.md !}

!!! note
    Обратите внимание, что для работы неблокирующих сообщений, `broker` должен быть запущен. Это значит, что мы не можем
    работать с такими сообщениями, используя `broker` как контекстный менеджер.