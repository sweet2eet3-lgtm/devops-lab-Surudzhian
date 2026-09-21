# Лабораторная работа №3

## Мониторинг с Prometheus и Grafana

### Цель работы

Настроить локальную систему мониторинга с использованием Prometheus и Grafana, подключить Node Exporter для сбора метрик и создать дашборд с показателями CPU, памяти и дискового пространства.

---

## 1. Создание конфигурации Prometheus

Была создана директория `prometheus` и файл `prometheus/prometheus.yml`.

Конфигурация:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

Конфигурация задаёт интервал сбора метрик 15 секунд и два источника данных: Prometheus и Node Exporter.

---

## 2. Запуск Node Exporter

Node Exporter был запущен в Docker-контейнере:

```bash
docker run -d 
  --name node-exporter 
  --restart=unless-stopped 
  -p 9100:9100 
  -v "/proc:/host/proc:ro" 
  -v "/sys:/host/sys:ro" 
  -v "/:/rootfs:ro" 
  prom/node-exporter 
  --path.procfs=/host/proc 
  --path.rootfs=/rootfs 
  --path.sysfs=/host/sys 
  --collector.filesystem.mount-points-exclude="^/(sys|proc|dev|host|etc)($$|/)"
```

Работоспособность Node Exporter была проверена командой:

```bash
curl http://localhost:9100/metrics
```

В ответ были получены метрики Node Exporter, например `go_gc_duration_seconds` и `go_goroutines`.
<img width="1429" height="951" alt="Снимок экрана 2026-09-21 в 07 51 04" src="https://github.com/user-attachments/assets/e54f4cf3-4379-49bb-967a-1885ee8096af" />




---

## 3. Настройка Prometheus

Были созданы Docker volume и сеть:

```bash
docker volume create prometheus-data
docker network create monitoring
```

Prometheus был запущен в Docker-контейнере и подключён к сети `monitoring`.

```bash
docker run -d 
  --name prometheus 
  --network monitoring 
  --restart=unless-stopped 
  -p 9090:9090 
  -v prometheus-data:/prometheus 
  -v "$(pwd)/prometheus:/etc/prometheus" 
  prom/prometheus 
  --config.file=/etc/prometheus/prometheus.yml 
  --storage.tsdb.path=/prometheus 
```

Node Exporter был подключён к той же Docker-сети:

    docker network connect monitoring node-exporter

После подключения Prometheus успешно получает метрики Node Exporter.

Проверка в Prometheus с помощью запроса `up` показала:

    up{instance="node-exporter:9100", job="node-exporter"} 1
    up{instance="localhost:9090", job="prometheus"} 1

Значение `1` означает, что оба источника метрик доступны.
<img width="1469" height="423" alt="Снимок экрана 2026-09-21 в 07 42 08" src="https://github.com/user-attachments/assets/ff1f3734-e652-4e8a-90f9-a35a9594d879" />

---

## 4. Запуск Grafana

Для хранения данных Grafana был создан Docker volume:

    docker volume create grafana-data

Grafana была запущена в Docker-контейнере и подключена к сети `monitoring`.

Контейнер Grafana использует порт `3000`.

После запуска веб-интерфейс Grafana был открыт по адресу `http://localhost:3000`.

---

## 5. Подключение Prometheus к Grafana

В Grafana был добавлен источник данных Prometheus.

Для подключения использовался адрес:

    http://prometheus:9090

Проверка подключения завершилась успешно сообщением:

    Successfully queried the Prometheus API.

<img width="1470" height="802" alt="Снимок экрана 2026-09-21 в 07 44 49" src="https://github.com/user-attachments/assets/37e487e4-29d6-4190-b872-2aebe7001b12" />

---

## 6. Создание Dashboard

В Grafana был создан Dashboard `System Monitoring`.

На Dashboard были добавлены три панели.

### CPU

Метрика:

    node_cpu_seconds_total

Название панели: `CPU metrics`.

### Оперативная память

Метрика:

    node_memory_MemAvailable_bytes

Название панели: `Memory metrics`.

### Дисковое пространство

Метрика:

    node_filesystem_avail_bytes

Название панели: `Disk metrics`.

Для всех панелей использовалась визуализация `Time series`.

<img width="1460" height="778" alt="Снимок экрана 2026-09-21 в 06 46 11" src="https://github.com/user-attachments/assets/c6beb5c1-f1be-4bb8-8d78-88fa35cd6078" />

---

## 7. Проверка работы

Состояние Docker-контейнеров было проверено командой:

    docker ps

Успешно работают:

- `node-exporter` — порт `9100`
- `prometheus` — порт `9090`
- `grafana` — порт `3000`

Все необходимые для лабораторной работы контейнеры имеют статус `Up`.
<img width="1156" height="107" alt="Снимок экрана 2026-09-21 в 07 47 54" src="https://github.com/user-attachments/assets/2f5eaff3-6215-414e-aaee-ed4e2b4dcb14" />


---

## Результат

В ходе лабораторной работы была настроена локальная система мониторинга на основе Prometheus и Grafana.

Node Exporter собирает системные метрики, Prometheus получает и хранит их, а Grafana используется для визуализации данных.

Был создан Dashboard `System Monitoring` с графиками для мониторинга CPU, доступной оперативной памяти и дискового пространства.
<img width="1122" height="244" alt="Снимок экрана 2026-09-21 в 07 48 47" src="https://github.com/user-attachments/assets/2682f973-2e8b-4c2e-93c4-67bba02c471e" />

