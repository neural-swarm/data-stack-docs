# 1. Reliable, Scalable, and Maintainable Applications
[english](01-reliable-scalable-and-maintainable-applications.md) | russian

**TL;DR:** Три цели data‑intensive систем: надёжность, масштабируемость, поддерживаемость — и как их обсуждать измеримо (метрики, SLO, компромиссы).

---

## Thinking About Data Systems

- Data‑intensive системы = комбинация БД, кешей, очередей, стримов; свойства важны на уровне всей цепочки.
- Дизайн — это выбор компромиссов: latency ↔ cost, consistency ↔ availability, simplicity ↔ feature richness.

## Reliability

- Цель: при сбоях (faults) не допустить внешней ошибки (failure) или быстро восстановиться (recovery).
- Частичные отказы — норма: планируй деградацию и восстановление заранее.

### Hardware Faults

- Диски/сеть/питание ломаются; в распределении вероятность отказа выше.
- Типовые меры: репликация, избыточность, health checks, автоматический failover.

### Software Errors

- Баги, утечки, дедлоки, неверные предположения вызывают каскадные отказы.
- Помогают: лимиты, изоляция (bulkheads), тесты, observability, упрощение.

### Human Errors

- Ошибки конфигов/деплоев — частая причина инцидентов.
- Помогают: автоматизация, staged rollout, feature flags, least privilege, бэкапы и “undo”.

### How Important Is Reliability?

- Решение — экономическое: стоимость мер vs цена инцидента (данные, доверие, деньги).

## Scalability

- Способность расти по нагрузке с предсказуемой стоимостью/сложностью.
- Важно понимать параметры нагрузки и “хвосты” задержек.

### Describing Load

- Опиши нагрузку: RPS, размер данных, read/write mix, fan‑out, hot keys, burstiness.

### Describing Performance

- Смотри на latency и throughput; latency измеряй перцентилями (p95/p99), а не средним.
- Очереди/конкуренция усиливают tail latency.

### Approaches for Coping with Load

- Scale up проще, но ограничен; scale out требует распределения данных/состояния.
- Практики: кеши, асинхронность, репликация, партицирование, backpressure, precompute.

## Maintainability

- Поддерживаемость = operability + simplicity + evolvability; TCO часто доминирует.

### Operability: Making Life Easy for Operations

- Наблюдаемость, алерты, runbooks, быстрые deploy/rollback, автоматизация рутинных операций.

### Simplicity: Managing Complexity

- Снижай “случайную” сложность: единые интерфейсы, явные зависимости, меньше магии.

### Evolvability: Making Change Easy

- Проектируй изменения: совместимость, миграции, постепенные релизы, расширяемые контракты.

## Summary

- Сначала свойства системы (reliability/scalability/maintainability), затем технологии.
