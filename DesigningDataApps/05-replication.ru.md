# 5. Replication
[english](05-replication.md) | russian

**TL;DR:** Зачем нужны реплики, как работают leader/follower, multi‑leader и leaderless схемы, какие проблемы создаёт replication lag и как распознавать/разрешать конфликты.

---

## Leaders and Followers

- Лидер принимает записи, реплики применяют лог; чтения с реплик ускоряют, но могут быть устаревшими.

## Synchronous Versus Asynchronous Replication

* **Synchronous replication**: запись подтверждается только после фиксации у лидера и на фолловере (или кворуме). Минимальный риск потери данных, но выше latency из‑за ожидания подтверждений.
* **Asynchronous replication**: лидер подтверждает запись сразу после локальной фиксации. Ниже задержки и выше throughput, но при падении лидера возможна потеря последних записей.
* Возможны промежуточные варианты (semi-sync, кворум), чтобы балансировать скорость и надёжность.

## Setting Up New Followers

* Новый фолловер получает **snapshot** — консистентную копию данных на определённый момент.
* Затем выполняется **догон лога** — применение изменений после snapshot.
* Ключевы точки: **LSN / offset / term-index**, чтобы точно определить место продолжения.
* Snapshot и логи должны быть согласованы, иначе потребуется пересинхронизация.

## Handling Node Outages

* Поведение зависит от роли узла (follower или leader).
* Важны RPO/RTO и механизмы автоматического переключения.

### Follower failure: Catch-up recovery

* После восстановления фолловер догоняет изменения по логу с последнего LSN.
* Если нужных логов нет (retention), выполняется новый snapshot.
* Большой replication lag снижает актуальность узла.

### Leader failure: Failover

* Выбирается новый лидер (обычно через алгоритм консенсуса).
* Клиенты переключаются на новый endpoint.
* Риски: **split brain** и потеря последних записей при async-репликации.
* Защита: кворум, term/epoch-контроль, fencing-механизмы.


## Implementation of Replication Logs

- Statement‑based (хрупко), WAL shipping (низкий уровень), logical log (устойчивее), triggers (гибко/рискованно).

### Statement-based replication

- Недетерминизм (NOW/RAND), побочные эффекты, разные планы выполнения.

### Write-ahead log (WAL) shipping

- Быстро, но привязано к внутреннему формату/версии движка.

### Logical (row-based) log replication

- Передаёт изменения строк/событий; удобно для интеграций и версий.

### Trigger-based replication

- Логика в триггерах; сложно сопровождать и тестировать.

## Problems with Replication Lag

* **Replication lag** — отставание фолловеров от лидера по применению записей.
* Приводит к нарушению пользовательских ожиданий: пользователь может не увидеть только что записанные данные.
* Может ломать причинный порядок (causality), если чтения идут с разных, по‑разному отстающих реплик.
* Требует явных стратегий маршрутизации и контроля консистентности при чтении.

### Reading Your Own Writes

* Пользователь ожидает видеть собственные изменения сразу после записи.
* Решения: читать у лидера, использовать **sticky sessions** (закрепление клиента за узлом) или передавать версию/LSN и требовать чтение не ниже неё.

### Monotonic Reads

* В рамках одной сессии данные не должны «откатываться назад».
* Достигается закреплением за одной репликой или хранением минимальной наблюдённой версии и чтением только с узлов, где она достигнута.

### Consistent Prefix Reads

* Наблюдения должны сохранять причинный порядок: нельзя увидеть результат события, не увидев его причину.
* Требует чтения с реплик, которые применили операции в корректном порядке без пропусков.

### Solutions for Replication Lag

* Маршрутизация чтений по уровню свежести (read from leader / nearest up-to-date replica).
* Ожидание до достижения нужной версии (read-after-write через LSN/offset).
* Мониторинг **replication lag / staleness** и исключение слишком отстающих узлов из чтения.

## Multi-Leader Replication

- Несколько лидеров принимают записи (multi‑DC, офлайн, совместное редактирование), но конфликты неизбежны. Основная сложность обеспечение сходимости (convergence), все реплики должны в итоге прийти к одному согласованному состоянию.

### Use Cases for Multi-Leader Replication

- multi‑datacenter, offline clients, collaborative editing.

#### Multi-datacenter operation

- Локальные записи и переживание падения DC ценой сложной консистентности. Записи из разных DC могут конфликтовать. Репликация асинхронная - eventual consistency.

#### Clients with offline operation

- Локальные изменения + синхронизация позже → нужна стратегия merge.

#### Collaborative editing

- Параллельные правки требуют CRDT(Conflict‑free Replicated Data Types)/OT(Operational Transformation)‑класса идей или логики merge.

### Handling Write Conflicts

- Избегать (routing/ownership) или разрешать (merge/LWW/custom rules); цель — convergence.

#### Synchronous versus asynchronous conflict detection

- Sync: координация заранее (распределённые блокировки или кворумные протоколы); Async: обнаружение и разрешение позже.

#### Conflict avoidance

- Направлять записи для ключа в один лидер (routing by key) или назначать владельца (ownership of data), сегментировать ключи между DC.

#### Converging toward a consistent state

Реплики должны в итоге сходиться даже при разных порядках доставки. 
От системы требуется:
- идемпотентность операций,
- детерминированные правила merge,
- устойчивость к повторной доставке сообщений,
- отсутствие бесконечных циклов обновлений.

#### Custom conflict resolution logic

Доменно‑специфичные правила merge, часто сложные. Варианты:
-Last Write Wins;
-merge на уровне полей;
-доменно‑специфичные правила (например, суммирование счётчиков);
-ручное разрешение конфликтов пользователем.

#### What is a conflict?

- Конфликт = нарушение инварианта, а не просто параллельная запись.

### Multi-Leader Replication Topologies

- Ring/star/mesh влияют на задержки распространения и циклы обновлений.

## Leaderless Replication

Запись на несколько узлов без лидера; reconciliation на чтении/в фоне. Чтение тоже идёт с нескольких узлов. Cassandra, DynamoDB, Riak.

Как работает:
- Для каждого ключа есть набор реплик (обычно через consistent hashing).
- При записи координатор (любой узел) рассылает запись на N реплик.
- При чтении координатор запрашивает несколько реплик и сравнивает версии.

Проблемы:
- Возможны расхождения между репликами.
- Нужны механизмы обнаружения и разрешения конфликтов.

### Writing to the Database When a Node Is Down

- Запись на доступные узлы; потом догон/перенос при восстановлении.

### Read repair and anti-entropy

- Read repair лечит при чтении; anti‑entropy синхронизирует в фоне.

### Quorums for reading and writing

- W+R>N повышает шанс свежего чтения, но не гарантирует линейризуемость.

### Limitations of Quorum Consistency

- Сеть/таймауты/конкурентные записи/clock skew (lww) ломают ожидания; возможна устаревшая видимость.

### Monitoring staleness

Важно измерять:
- replication lag
- Версии (vector clock size, offset)
- convergence lag

Инструменты: replication delays, conflicts, hinted handoff queue size.


### Sloppy Quorums and Hinted Handoff

- Пишем на “временные” узлы при сбоях и потом отдаём владельцам; повышает availability, усложняет консистентность.

#### Multi-datacenter operation

- Меж‑DC задержки увеличивают окно конфликтов и staleness. Local quorum/Global quorum.

### Detecting Concurrent Writes

- Нужно отличать причинный порядок от конкуренции (happens‑before).

#### Last write wins (discarding concurrent writes)

- LWW прост, но может терять обновления (затирание).

#### The happens-before relationship and concurrency

- Причинность ≠ timestamp; concurrent = нет happens‑before.

#### Capturing the happens-before relationship

- Версии/метаданные помогают выявить конкуренцию.

#### Merging concurrently written values

- Merge‑стратегии (например, CRDT‑подходы) уменьшают ручные конфликты.

#### Version vectors

- Вектор версий по репликам для причинности и выявления конфликтов.

## Summary

- Репликация улучшает availability/latency, но требует стратегии консистентности и конфликтов.
