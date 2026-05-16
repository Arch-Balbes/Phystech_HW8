# ДЗ 8: мониторинг (Prometheus + Grafana)

## SLO сервиса инференса (пример)

| Показатель | Цель | Метрика Prometheus |
|------------|------|---------------------|
| Задержка | p95 &lt; 1 с | `histogram_quantile(0.95, sum(rate(request_latency_seconds_bucket[5m])) by (le))` |
| Ошибки | error rate &lt; 1% | (при появлении счётчика ошибок) `rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])` |
| Доступность | &gt; 99% | uptime/blackbox или успешные scrape |

## Запуск локально

1. Установите зависимости сервиса: `pip install fastapi uvicorn prometheus-client`
2. Запустите ML-сервис: `uvicorn ml_service:app --host 0.0.0.0 --port 8000`
3. Запуск стека мониторинга: `docker compose up -d`
4. Prometheus: http://localhost:9090/targets — цель `ml_service` должна быть UP.
5. Grafana: http://localhost:3000 (admin / admin) → Connections → Prometheus → URL `http://prometheus:9090`.
6. Импорт дашборда: Dashboards → Import → файл `grafana_dashboard_ml_latency.json` (при необходимости выберите источник Prometheus).

## Алерт

- В Grafana: правило на панели с условием p95 &gt; 1 и интервалом оценки 1m (как в методичке).
- В Prometheus: правила в `prometheus_rules.yml` (группа `latency_alerts`).

Проверка: отправьте несколько запросов с задержкой:

```bash
curl -X POST "http://localhost:8000/predict?slow=true"
```

## Дрифт данных

В ноутбуке — расчёт отчёта Evidently (`drift_report.html`): эталон vs текущая выборка со смещением масштаба.

## Архитектура VPP (раздел 5)

В ноутбуке — диаграмма на библиотеке Diagrams: выбран **Kappa-поток** (непрерывный лог событий кадров/регионов для низкой задержки и масштабирования по партициям), с офлайн-путём для переобучения.


