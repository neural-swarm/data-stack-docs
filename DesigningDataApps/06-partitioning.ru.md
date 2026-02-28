# 6. Partitioning
[english](06-partitioning.md) | russian

**TL;DR:** Как разбивать данные по узлам (range/hash), бороться с hot spots, строить вторичные индексы в шардинге, ребалансировать и маршрутизировать запросы.

---

## Partitioning and Replication

- Партицирование распределяет данные; репликация дублирует их для отказоустойчивости.

## Partitioning of Key-Value Data

- Цель — равномерная нагрузка и управляемый размер данных на узел.

## Partitioning by Key Range

- Хорошо для range queries; риск hot ranges (например, по времени).

## Partitioning by Hash of Key

- Ровнее распределяет, но диапазонные запросы требуют scatter/gather.

## Skewed Workloads and Relieving Hot Spots

- Hot keys ломают баланс; применяй кеши, salting, дробление популярных ключей, больше партиций.

## Partitioning and Secondary Indexes

- Вторичные индексы: локальные (by document) проще для записи, глобальные (by term) быстрее для чтения.

### Partitioning Secondary Indexes by Document

- Индекс локален шард‑данным; запрос по индексу часто идёт ко всем шардам. Cassandra, HBase.

### Partitioning Secondary Indexes by Term

- Глобальный индекс по значению; чтения быстрее, записи сложнее и требуют распределённых обновлений. MongoDB, DynamoDB (GSI), CockroachDB, Elasticsearch/OpenSearch.

## Rebalancing Partitions

- При изменении числа узлов переносим партиции без сильных просадок и без массовых перестроек.

### Strategies for Rebalancing

- Перемещай “куски” (партиции), а не отдельные ключи; ограничивай скорость миграции.

#### How not to do it: hash mod N

- Изменение N меняет почти все назначения → огромная миграция.

#### Fixed number of partitions

- Много партиций заранее → проще балансировать, но нужно предположить число. Kafka.

#### Dynamic partitioning

- Партиции делятся по мере роста; адаптивно, но сложнее управлять, split/merge - тяжёлые операции. Bigtable, HBase.

#### Partitioning proportionally to nodes

- Число партиций растёт с узлами; компромисс между гибкостью и управляемостью. Cassandra / Dynamo-style.

### Operations: Automatic or Manual Rebalancing

- Автоматизация удобна, но нужна защита от “миграции в плохой момент” (лимиты, окна).

## Request Routing

Клиент/координатор должен знать, где находится ключ: 
- routing layer (Elasticsearch coordinating node, MongoDB mongos)
- умный клиент (Cassandra driver, Kafka producer, Dynamo-style)
- каталог/метаданные (Bigtable)

## Parallel Query Execution

- Scatter/gather ускоряет большие запросы, но tail latency зависит от самого медленного шарда.

Как уменьшают влияние tail latency:
- Репликация + speculative execution. Если шард отвечает долго → отправляем запрос на реплику.
- Не делать scatter по всем шардам (делать key-based, range)
- Partial aggregation (на шарде)
- Timeout (вернуть частичный результат)

## Summary

- Выбирай range vs hash по типам запросов; заранее продумай routing и ребалансировку.

---

## Термины

- **Partition/Sharding:** разбиение данных на части по узлам
- **Range partitioning:** шардинг по диапазонам ключей
- **Hash partitioning:** шардинг по хэшу ключа
- **Hot spot:** ключ/диапазон с непропорционально высокой нагрузкой
- **Secondary index:** индекс по не‑primary полю
- **Scatter/Gather:** рассылка запроса по шардам и сбор результатов
- **Rebalancing:** перераспределение партиций при изменении кластера
- **Routing:** направление запроса на нужный шард
- **Salting:** добавка к ключу для распределения нагрузки
- **Coordinator:** узел/слой, выполняющий маршрутизацию/агрегацию
