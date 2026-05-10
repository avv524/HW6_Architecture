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

## Запуск

```powershell
docker compose up
```

Если нужно пересобрать образ:

```powershell
docker compose up --build
```

Для первой демонстрации Cassandra 3-node лучше стартовать с чистыми volumes:

```powershell
docker compose down -v
docker compose up --build
```

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

## Быстрая проверка

```powershell
docker exec cassandra-1 nodetool status

'{"event_id":"demo-1","event_type":"PRODUCT_RECEIVED","event_timestamp":"2026-04-01T12:00:00Z","product_id":"SKU-001","zone_id":"ZONE-A","quantity":100}' | curl.exe -s -X POST http://localhost:8080/events -H "Content-Type: application/json" --data-binary "@-"

docker exec cassandra-1 cqlsh -e "SELECT * FROM warehouse.inventory_by_product_zone WHERE product_id='SKU-001' AND zone_id='ZONE-A';"
```

В `nodetool status` ожидаются 3 строки `UN`.

Подробный разбор архитектуры, файлов и сценариев защиты находится в `readme2.md`.
