# 3. Storage and Retrieval
[english](03-storage-and-retrieval.md) | russian

**TL;DR:**
Индексы и физическая организация хранения определяют реальные
характеристики БД: latency, throughput, amplification-эффекты, поведение
под нагрузкой и масштабируемость. Разные структуры (B-tree, LSM, column
store, inverted index) оптимизированы под разные типы workload.

------------------------------------------------------------------------

## Data Structures That Power Your Database

Индекс --- альтернативное физическое представление данных,
оптимизированное под конкретный тип доступа:

-   Point lookup
-   Range scan
-   Sorting
-   Aggregation
-   Text search

Любой индекс создаёт:

1.  Write cost
2.  Storage cost
3.  Complexity cost

Главный принцип:
**Индекс --- это компромисс между locality записи и locality чтения.**

------------------------------------------------------------------------

## Hash Indexes

### Физическая модель

    hash(key) → bucket → pointer → value

Используется:

-   In-memory hash table
-   Extendible hashing
-   Cuckoo hashing

### Ограничения

-   Нет range scan
-   Нет упорядоченности
-   Дорогой rehash

Хорош для KV workload.

------------------------------------------------------------------------

## SSTables and LSM-Trees

LSM минимизирует random write и переносит сложность в чтение.

### Memtable

Обычно:

-   Skip list
-   Red-Black tree

Skip list удобен для lock-free реализации.

------------------------------------------------------------------------

## Constructing and Maintaining SSTables

SSTable содержит:

-   Data blocks
-   Index blocks
-   Bloom filters
-   Footer

Индекс внутри SSTable обычно:

-   Sparse (каждый N-й ключ)
-   Не хранит все ключи полностью

Это уменьшает memory footprint.
SSTable использует **sequential I/O**, что идеально для SSD и HDD.

------------------------------------------------------------------------

## Making an LSM-tree out of SSTables

Чтение требует проверки:

-   Memtable
-   L0
-   L1
-   L2...

### Compaction trade-off

Leveling:

-   Меньше read amplification
-   Больше write amplification

Tiering:

-   Меньше write cost
-   Больше read cost

------------------------------------------------------------------------

## Performance Optimizations (LSM)

-   Bloom filters
-   Block cache
-   Write batching
-   Compression per level
-   Rate-limited compaction

LSM настраивается политикой compaction.

------------------------------------------------------------------------

## Amplification Effects

### Write Amplification (WA)

WA = фактические записи / логические записи

Высокая WA:

-   Изнашивает SSD
-   Увеличивает latency

### Read Amplification (RA)

RA = фактические чтения / логические чтения

LSM увеличивает RA при:

-   Глубокой иерархии
-   Большом overlap


------------------------------------------------------------------------

## B-Trees

Страницы 4--16KB.

Глубина дерева мала (3--4).

Page split вызывает random write.

------------------------------------------------------------------------

## Making B-Trees Reliable

### WAL

1.  Запись в журнал
2.  Обновление страницы

После сбоя --- WAL replay.

------------------------------------------------------------------------

## B-Tree Optimizations

-   Prefix compression
-   Fill factor
-   Pointer swizzling
-   Buffer pool tuning

Производительность зависит от размера страницы и cache policy.

------------------------------------------------------------------------

## Comparing B-Trees and LSM-Trees

  Характеристика        B-tree          LSM-tree
  --------------------- --------------- ---------------------
  Write throughput      Средний         Очень высокий
  Read latency          Предсказуемая   Может варьироваться
  Write amplification   Ниже            Выше
  Read amplification    Ниже            Выше

------------------------------------------------------------------------

## Advantages of LSM-Trees

-   Высокий ingest
-   Sequential writes
-   Хорошая компрессия
-   Хорошо подходят для SSD

------------------------------------------------------------------------

## Downsides of LSM-Trees

-   Compaction spikes
-   Write stall
-   Сложность настройки
-   Merge cost при чтении

------------------------------------------------------------------------

## Other Indexing Structures

### Storing Values Within the Index

-   Heap table + secondary index
-   Clustered index (данные внутри B-tree)

Clustered быстрее читает, дороже обновляет.

------------------------------------------------------------------------

### Multi-column Indexes

Left-prefix rule:

(A, B, C) поддерживает A и A+B, но не только B.

------------------------------------------------------------------------

### Full-text search and fuzzy indexes

Inverted index:

    term → posting list

Fuzzy поиск:

-   n-grams
-   Trigram index
-   Levenshtein
-   BK-tree

------------------------------------------------------------------------

### Keeping Everything in Memory

In-memory index:

-   Минимальная latency
-   Ограничен RAM

Durability достигается через:

-   Append-only log
-   Snapshotting
-   Replication

------------------------------------------------------------------------

## Transaction Processing or Analytics?

OLTP:

-   Point lookup
-   Secondary indexes
-   Row storage

OLAP:

-   Full scans
-   Aggregates
-   Column storage

------------------------------------------------------------------------

## Data Warehousing

-   Отдельное хранилище
-   ETL / ELT
-   Column storage
-   Batch ingestion

------------------------------------------------------------------------

## The divergence between OLTP databases and data warehouses

OLTP:

-   ACID
-   Random access

OLAP:

-   Scan speed
-   Compression
-   Vectorized execution

------------------------------------------------------------------------

## Stars and Snowflakes: Schemas for Analytics

Star schema:

-   Факт + измерения
-   Быстрые join

Snowflake:

-   Нормализация измерений
-   Больше join cost

------------------------------------------------------------------------

## Column-Oriented Storage

-   Хранение по колонкам
-   Читаются только нужные столбцы
-   Отличная компрессия

------------------------------------------------------------------------

## Column Compression

-   Dictionary encoding
-   Run-length encoding
-   Bit-packing
-   Delta encoding

------------------------------------------------------------------------

## Memory Bandwidth and Vectorized Processing

Vectorized execution:

-   Batch обработка
-   SIMD
-   Меньше branch misprediction

В аналитике bottleneck --- память.

------------------------------------------------------------------------

## Sort Order in Column Storage

Сортировка:

-   Улучшает компрессию
-   Ускоряет фильтрацию
-   Делает zone maps эффективнее

------------------------------------------------------------------------

## Several Different Sort Orders

Некоторые системы хранят multiple projections.

Цена:

-   Больше storage
-   Сложнее обновление

------------------------------------------------------------------------

## Writing to Column-Oriented Storage

-   Delta store (row-oriented)
-   Merge в column segments

Похоже на LSM-подход.

------------------------------------------------------------------------

## Aggregation: Data Cubes and Materialized Views

Data cube:

-   Предагрегированные измерения

Materialized view:

-   Предрасчитанный запрос

Используются для ускорения OLAP.
