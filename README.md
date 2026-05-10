# HW6 — Smart Warehouse

Event-driven система управления складом на Kafka, Schema Registry и Cassandra.

## Что реализовано

- WMS producer на FastAPI публикует Avro-события в Kafka topic `warehouse-events`.
- Stateful consumer читает события в group `warehouse-state-consumer` и обновляет состояние склада в Cassandra.
- At-least-once обработка: offset commit только после успешной записи состояния.
- Идемпотентность через таблицу `processed_events`.
- Out-of-order защита через `entity_versions` и timestamp strategy.
- DLQ topic `warehouse-events-dlq` для невалидных событий.
- Cassandra cluster из 3 нод, RF=3, read/write consistency `QUORUM`.
- Avro schema evolution через Schema Registry: V1 и V2 (`supplier_id`).
- Prometheus metrics, `/health`, Grafana dashboard и alert на consumer lag.

## Порты

| Сервис | URL |
|--------|-----|
| WMS Producer | http://localhost:8080 |
| Consumer health/metrics | http://localhost:8000 |
| Lag exporter | http://localhost:8001 |
| Schema Registry | http://localhost:8081 |
| Prometheus | http://localhost:9090 |
| Grafana | http://localhost:3000 |
| Cassandra | localhost:9042 |
| Kafka | localhost:9092 |

Grafana login: `admin/admin`.

Swagger UI WMS producer: `http://localhost:8080/docs`. В `POST /events` и `POST /events/bulk` добавлены готовые examples для приемки, резервирования, out-of-order, DLQ, lag demo и schema evolution.
