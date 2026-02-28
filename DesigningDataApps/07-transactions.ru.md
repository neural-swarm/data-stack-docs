# 7. Transactions
[english](07-transactions.md) | russian

**TL;DR:** ACID, уровни изоляции и аномалии конкурентности; как добиться сериализуемости через serial execution, 2PL и SSI.

---

## The Slippery Concept of a Transaction

- Транзакция = набор гарантий корректности; в разных системах “транзакции” означают разное.

## The Meaning of ACID

- Atomicity (всё/ничего), Consistency (инварианты), Isolation (конкурентность), Durability (не теряем после commit).

### Atomicity

- Частичных эффектов не видно; abort откатывает изменения.

### Consistency

- Инварианты домена; БД помогает constraints, но логика часто в приложении.

### Isolation

- Параллельные транзакции не должны давать аномалии; на практике уровни изоляции компромиссные.

### Durability

- После commit данные переживают сбой (WAL/fsync/репликация/снапшоты).

## Single-Object and Multi-Object Operations

- Многие системы атомарны на уровне одного объекта; но инварианты часто меж‑объектные.

### Single-object writes

- Атомарные операции над ключом/документом упрощают дизайн, но не решают меж‑объектные инварианты.

### The need for multi-object transactions

- Переводы/счётчики/связи требуют транзакций на несколько записей.

### Handling errors and aborts

- Retries должны учитывать идемпотентность и внешние побочные эффекты.

## Weak Isolation Levels

- Слабые уровни допускают аномалии ради производительности; важно знать, какие.

### Read Committed

- Запрещает dirty reads/writes; всё ещё возможны non-repeatable reads и фантомы.

#### No dirty reads

- Не читать незакоммиченные данные другой транзакции.

#### No dirty writes

- Не перезаписывать незакоммиченные данные другой транзакции.

#### Implementing read committed

- Блокировки на запись + правила видимости версий.

### Snapshot Isolation and Repeatable Read

- Чтение из снимка + контроль конфликтов на запись; хорошо для читателей.

#### Implementing snapshot isolation

- MVCC хранит версии; каждая транзакция видит консистентный snapshot.

#### Visibility rules for observing a consistent snapshot

- Видим только версии, закоммиченные до “момента снимка”.

#### Indexes and snapshot isolation

- Индексы должны учитывать видимость, иначе возможны “призрачные” результаты.

#### Repeatable read and naming confusion

- Термины различаются между СУБД: иногда repeatable read = snapshot isolation.

### Preventing Lost Updates

- Lost update решается атомарными операциями, блокировками или оптимистичной проверкой версии.

#### Atomic write operations

- Инкремент/CAS на стороне БД вместо read‑modify‑write в приложении.

#### Explicit locking

- SELECT … FOR UPDATE сериализует доступ к строкам.

#### Automatically detecting lost updates

- Optimistic concurrency: обновляй, если версия/etag не изменилась.

#### Compare-and-set

- CAS — условная запись при совпадении ожидаемого состояния.

#### Conflict resolution and replication

- В репликации важно, где выявлять конфликт и как его merge‑ить.

### Write Skew and Phantoms

- Snapshot isolation не всегда защищает от write skew; фантомы могут нарушать инварианты.

#### Characterizing write skew

- Две транзакции читают условие и пишут разные строки, вместе нарушая правило.

#### More examples of write skew

- Бронирования, лимиты, расписания, где важен глобальный инвариант.

#### Phantoms causing write skew

- Новые строки, удовлетворяющие условию, появляются между чтением и записью.

#### Materializing conflicts

- Сделать “объект конфликта” явным (lock row) — чтобы конкуренты конфликтовали по одной записи.

### Serializability

- Самая сильная гарантия: как будто транзакции выполнились в некотором последовательном порядке.

### Actual Serial Execution

- Сериализация через выполнение по одной (или внутри шарда) проста, но ограничивает throughput.

#### Encapsulating transactions in stored procedures

- Меньше сетевых round‑trips; логика ближе к данным.

#### Pros and cons of stored procedures

- Плюсы: производительность/контроль; минусы: сложнее развивать и тестировать.

#### Partitioning

- Серийное выполнение легче внутри одного шарда; межшардовые транзакции сложнее.

#### Summary of serial execution

- Подходит при коротких транзакциях и контролируемой нагрузке.

### Two-Phase Locking (2PL)

- Пессимистичный подход: блокировки удерживаются до commit; убирает аномалии, но даёт contention и дедлоки.

#### Implementation of two-phase locking

- Shared/exclusive locks + строгие правила удержания.

#### Performance of two-phase locking

- Падает throughput, растёт latency; нужен дедлок‑детектор/таймауты.

#### Predicate locks

- Блокировки по условию, чтобы предотвратить фантомы.

#### Index-range locks

- Практическая форма predicate locks через диапазоны в индексе.

### Serializable Snapshot Isolation (SSI)

- Оптимистичная сериализуемость поверх snapshot isolation: отслеживание опасных зависимостей и abort.

#### Pessimistic versus optimistic concurrency control

- Пессимистично: блокируем заранее; оптимистично: выполняем и проверяем конфликт.

#### Decisions based on an outdated premise

- Опасность: решение принято на устаревшем snapshot.

#### Detecting stale MVCC reads

- Отслеживание чтений версий, которые позже конфликтуют с записями.

#### Detecting writes that affect prior reads

- Фиксация зависимостей read→write, которые делают расписание не‑сериализуемым.

#### Performance of serializable snapshot isolation

- Часто лучше 2PL при низкой конфликтности; при высокой — много abort/retry.

## Summary

- Выбирай уровень изоляции под инварианты; сериализуемость достигается блокировками или проверками+абортами.

---

## Термины

- **ACID:** Atomicity, Consistency, Isolation, Durability
- **Isolation level:** какие аномалии конкурентности допускаются
- **Dirty read/write:** чтение/запись незакоммиченных данных
- **Snapshot isolation:** чтение из snapshot + контроль конфликтов на запись
- **MVCC:** мультиверсии записей для конкурентности
- **Lost update:** потеря обновления из‑за гонки
- **Write skew:** нарушение инварианта при параллельных записях в разные строки
- **Phantom:** появление/исчезновение строк, удовлетворяющих условию
- **Serializability:** эквивалентность последовательному выполнению
- **2PL:** two‑phase locking: сериализация через блокировки
- **Predicate lock:** блокировка по условию для устранения фантомов
- **Index-range lock:** блокировка диапазона в индексе
- **SSI:** Serializable Snapshot Isolation: сериализуемость через обнаружение опасных зависимостей
- **Optimistic concurrency:** проверка конфликтов на commit (версии/CAS)
- **CAS:** compare‑and‑set: условное обновление
- **Idempotency:** повтор операции безопасен
