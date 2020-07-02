# Тестовое задание

Требуется разработать сервис, принимающий websocket соединения. Клиент данного сервиса при подключении присылает
свой `client_id` как параметр запроса, где `client_id` - произвольный текстовый идентификатор, например UUID. После того,
как соединение с сервисом установлено, сервис присваивает клиенту текстовый идентификатор session_id, и отсылает
его клиенту как текст. Таким образом, каждый клиент, после установки соединения, имеет два идентификатора - `client_id` и
`session_id`.

Пример url для подключения к сервису:

`http://localhost:8880/?client_id=370f4521-e499-4e30-8778-5710f0497497`

В случае, если к сервису подключается клиент, используя `client_id`, который уже используется в другом соединении (то есть два и 
более клиента пытаются подключится к сервису используя один и тот же `client_id`), то необходимо предидущее соединение с данным
`client_id` прервать. В один момент времени с сервисом может быть только одно соединение с использованием конкретного
`client_id`.

Cервис должен быть масштабируемым - может быть запущено несколько процессов сервиса параллельно, которые вместе
образуют единый сервис. Подключение клиента к любому из процессов сервиса ведет к одинаковому результату.

Websocket соединение сервиса должно отвечать на `PING` сообщения и отсылать свои `PING` сообщения каждые 5 секунд если не происходит
активный обмен данными. сервис ждет ответ от клиента на `PING` сообщение 5 секунд, и в случае, если ответа нет, разрывает соединение и
удаляет соответствующую сессию.

Cервис должен возвращать список активных клиентов и присвоенных им идентификаторов сессий используя путь `/sessions`. Например,
в случае если один из процессов сервиса работает на `8880` порту на `localhost`, тогда запрос на получение сессий будет
в виде `http://localhost:8880/sessions`.

Список сессий представляет из себя обычный текст, в котором в каждой строке находится пара `client_id` и `session_id` разделенная пробелом,
например:

```
370f4521-e499-4e30-8778-5710f0497497 986de040-2760-45ad-9bb5-e63cc29e0444
ce6c6bb7-8614-4a62-8490-bc35e512e539 35a23b09-cb74-4d32-91d0-90597cb031be
2c34a3ee-d57b-4738-be9b-cd8612696c39 bba141c3-f329-4914-a370-985811f0731e
51aa8f44-acf6-43c5-8750-b18ec7ecb628 c8ec9a43-da39-4fdd-82c8-5e930b073478
c1efb4cc-3297-4a44-951a-95e0cb502e88 329c363c-c661-401a-ab49-f44e1cbd26e6
```

К тестовому заданию прилагается клиент (`client.py`), который устанавливает соединение. А так же `requirement.txt` файл, содержащий пакеты,
необходимые для работы клиента.

Пример использования клиента, в случае если один из процессов сервиса работает на `8880` порту на `localhost`:

`python3 client.py --endpoint=http://localhost:8880 --client-id=test1`

В случае, если имеются 3 процесса сервиса, работающие на `localhost`, и использующие порты `8880`, `8881`, `8882`,
то последовательный запуск трех экземпляров клиента с параметрами:

```
python3 client.py --endpoint=http://localhost:8880 --client-id=test1
python3 client.py --endpoint=http://localhost:8881 --client-id=test1
python3 client.py --endpoint=http://localhost:8882 --client-id=test1
```

должен привести к тому, что только клиент, подключившийся к порту 8882 будет иметь активное соединение. Остальные экземпляры клиента
должны завершить работу из-за отключения от сервиса, что удовлетворяет условию что только один клиент с идентификатором test1
может иметь единовременное подключение к сервису.

сервис должен быть написан на языке Python 3, с использованием любым фреймворков и дополнительных инструментов.

Дополнительным бонусом будет являтся возможность запустить данный сервис с использованием прокси с load balancer. Использование
docker приветствуется.
