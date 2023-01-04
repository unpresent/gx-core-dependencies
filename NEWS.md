## Common NEWS for all libs and services core-java

### v1.5.24
* Добавлена установка времени жизни (3 часа) для записей временных коллекций, которые используются для 
пакетной выгрузки в Redis. По истечение этого времени Redis будет удалять такие ключи. 

### v1.5.23

* Важное изменение в работе [AbstractWorker](/../gx-core-root-starter/src/main/java/ru/gx/core/worker/AbstractWorker.java).
Теперь основной поток запускается не с помощью `Thread`, а с использованием `ExecutorService`.
**Важно**! При перезапуске основного потока (инициирует `RestartingController`) теперь вместо мягкого `Thread.interrupt()`
используется связка `ExecutorService.awaitTermination(timeoutMs, TimeUnit.MILLISECONDS)`
и `ExecutorService.shutdownNow()`. Т.е. ждём самостоятельного останова основной потока Исполнителя в течение
задаваемого настройкой `wait-on-restart-ms` периода. Если не дожидаемся, то снимаем принудительно.

* В интерфейс [MessagesPrioritizedQueue](/../gx-core-root-starter/src/main/java/ru/gx/core/messaging/MessagesPrioritizedQueue.java)
добавлен метод `pushSystemMessage()`. Предполагается, что реализация данного метода будет класть сообщение в очередь 
с наивысшим приоритетом.

* В реализации [StandardMessagesPrioritizedQueue](/../gx-core-root-starter/src/main/java/ru/gx/core/messaging/StandardMessagesPrioritizedQueue.java)
метод `pushMessage()` теперь не может опубликовать сообщение в очередь с приоритетом 0. В эту очередь публикует
сообщения только метод `pushSystemMessage()`.

* Реализация [StandardMessagesExecutor](/../gx-core-root-starter/src/main/java/ru/gx/core/messaging/StandardMessagesExecutor.java)
теперь разбита на пару: [AbstractMessagesExecutor](/../gx-core-root-starter/src/main/java/ru/gx/core/messaging/AbstractMessagesExecutor.java)
и собственно [StandardMessagesExecutor](/../gx-core-root-starter/src/main/java/ru/gx/core/messaging/StandardMessagesExecutor.java).

### v1.5.22

* Исправление в [DataPublishProcessor](/../gx-core-publisher-starter/src/main/java/ru/gx/core/publisher_starter/service/DataPublishProcessor.java).
Исправлена установка флага `setLast()` для последней пачки в канале (была ошибка при обработке сразу нескольких каналов,
флаг `last` устанавливался для следующей после последней пачки, т.е. для первой пачки следующего канала).

### v1.5.21

* Добавлено данное описание [NEWS.md](/NEWS.md).

* Доработан [KafkaIncomeTopicsOffsetsController](/../gx-core-kafka-starter/src/main/java/ru/gx/core/kafka/load/KafkaIncomeTopicsOffsetsController.java).
Теперь при попытке установить offset на значение меньше самого раннего (например, очередь частично почистилась,
а наш сервис перезапускается и вычитал ранее сохраненное смещение много меньше минимального в очереди)
устанавливается смещение на начальное в очереди. 

* В [AbstractWorker](/../gx-core-root-starter/src/main/java/ru/gx/core/worker/AbstractWorker.java)
доработаны блокировки synchronized. Разделены 2 монитора: `executingMonitor` и `restartingMonitor`.
Попытка защиты от двойного запуска Runner-ов.

* Доработан [WebSecurityConfig](/../gx-core-rest-security-api-starter/src/main/java/ru/gx/core/securityapi/config/WebSecurityConfig.java)
открыт доступ к метрикам и к swagger описаниям.

### v1.5.20

* Добавлен новая библиотека [core-publisher-starter](/../gx-core-publisher-starter).
Предназначение - публикация в Redis коллекций частями. При публикации для экономии использования памяти сервиса
в Redis создаются временные коллекции и используются для отслеживания того, какие записи надо из Redis удалить
(которых более нет в источнике).

### v1.5.19

* Добавлена поддержка на уровне core обработки длительных задач, запускаемых по REST.
В библиотеке [core-root-starter](/../gx-core-root-starter) автоматически создается сервис
[LongtimeProcessService](/../gx-core-root-starter/src/main/java/ru/gx/core/longtime/LongtimeProcessService.java),
который нужен для управления описателями таких длительных задач.
Сценарий обработки таких задач:
1) В потоке REST-запроса создается (метод `createLongtimeProcess()` класса
[LongtimeProcessService](/../gx-core-root-starter/src/main/java/ru/gx/core/longtime/LongtimeProcessService.java))
описатель ([LongtimeProcess](/../gx-core-root-starter/src/main/java/ru/gx/core/longtime/LongtimeProcess.java))
такой длительной задачи.
2) Во внутреннюю очередь обработки сообщений
([MessagesPrioritizedQueue](/../gx-core-root-starter/src/main/java/ru/gx/core/messaging/MessagesPrioritizedQueue.java))
бросается событие
([LongtimeProcessEvent](/../gx-core-root-starter/src/main/java/ru/gx/core/longtime/LongtimeProcessEvent.java))
с указанием идентификатора данной задачи.
3) При обработке такого события необходимо выполнить полезную работу, т.е. обработать часть (действие) длительной
задачи, а затем (если обработка такой задачи не завершена) бросить во внутреннюю очередь сообщений ещё одно событие
([LongtimeProcessEvent](/../gx-core-root-starter/src/main/java/ru/gx/core/longtime/LongtimeProcessEvent.java))
с указанием идентификатора данной задачи. Обработка частей таких длительных задач осуществляется в потоке `MessageExecutor`.

### v1.5.18

* Исправлена загрузка offset-ов в
[KafkaTopicsOffsetsStorage](/../gx-core-std-starter/src/main/java/ru/gx/core/std/offsets/KafkaTopicsOffsetsStorage.java).
Если полученный offset из хранилища null, то такой не выдаем в результат.

### v1.5.17

* Исправлено создание бина
[ReloadRestController](/../gx-core-redis-starter/src/main/java/ru/gx/core/redis/controller/ReloadRestController.java).
* Небольшие описания для Swagger-а e REST-а `/dictionaries/reload-all`

### v1.5.16

* Исправлена ошибка в работе функции readyForSave() (определяем накоплен ли буфер достаточного размера, или
прошло ли достаточно времени накопления) в классе 
[DbSavingDescriptor](/../gx-core-data-commons/src/main/java/ru/gx/core/data/save/DbSavingDescriptor.java)
* Исправлена ошибка в работе функции checkBufferIsFull() в классе
[DbSavingDescriptor](/../gx-core-data-commons/src/main/java/ru/gx/core/data/save/DbSavingDescriptor.java)
* В утилиту [core-key-storage-util](/../gx-core-key-storage-util) добавлен REST DELETE /key-storage/{service}/{key}
* В [RedisOutcomeCollectionsUploader](/../gx-core-redis-starter/src/main/java/ru/gx/core/redis/upload/RedisOutcomeCollectionsUploader.java)
добавлена поддержка выгрузки пакетами данных в Redis. Методы:
1) `startBatchedUploadObjects()`
2) `finishBatchedUploadObjects()`
3) `batchedUploadDataObjects()`
* В [MessagesPrioritizedQueue](/../gx-core-root-starter/src/main/java/ru/gx/core/messaging/MessagesPrioritizedQueue.java)
добавлен метод returnErrorMessage(). Используется в
[StandardMessagesExecutor](/../gx-core-root-starter/src/main/java/ru/gx/core/messaging/StandardMessagesExecutor.java) - 
при ошибке сообщение возвращается в
[MessagesPrioritizedQueue](/../gx-core-root-starter/src/main/java/ru/gx/core/messaging/MessagesPrioritizedQueue.java).
