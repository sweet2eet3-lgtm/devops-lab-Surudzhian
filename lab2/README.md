# Лабораторная работа №2

## CI/CD для Docker-приложения

## Ход работы

### 1. Подготовка проекта

В первой лабораторной работе было создано Flask-приложение и Dockerfile. Для второй лабораторной работы необходимые файлы были перенесены в отдельную директорию `lab2` существующего GitHub-репозитория.

Структура проекта:

```text
devops-lab-Surudzhian/
├── .github/
│   └── workflows/
│       └── docker-build.yml
├── lab1/
│   ├── Dockerfile
│   ├── README.md
│   ├── app.py
│   └── requirements.txt
├── lab2/
│   ├── Dockerfile
│   ├── README.md
│   ├── app.py
│   └── requirements.txt
├── CONTRIBUTING.md
└── README.md
```

Для лабораторной работы №2 в директорию `lab2` были скопированы файлы `app.py`, `requirements.txt` и `Dockerfile`.

Файл `app.py`:

```python
from flask import Flask

app = Flask(__name__)


@app.route("/")
def hello():
    return "Hello from Docker!"


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

Приложение представляет собой простой Flask-сервис. При обращении к корневому маршруту `/` возвращается сообщение `Hello from Docker!`. Адрес `0.0.0.0` позволяет приложению принимать подключения через сетевой интерфейс контейнера.

Файл `requirements.txt`:

```text
Flask>=3.1,<4
```

Файл `Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

Docker-образ собирается на основе `python:3.12-slim`. В контейнер устанавливаются зависимости из `requirements.txt`, после чего копируется исходный код приложения. Команда `CMD` задаёт запуск Flask-приложения при старте контейнера. Инструкция `EXPOSE 5000` указывает используемый приложением порт.

Локальная сборка образа была выполнена командой:

```bash
docker build -t my-flask-app .
```

Сборка завершилась успешно.

В Docker Hub был создан репозиторий:

```text
haru1meow/my-flask-app
```

### 2. Настройка секретов

Для авторизации GitHub Actions в Docker Hub в репозитории GitHub в разделе **Settings → Secrets and variables → Actions** были добавлены следующие секреты:

| Имя               | Назначение                       |
| ----------------- | -------------------------------- |
| `DOCKER_USERNAME` | Логин Docker Hub: `haru1meow`    |
| `DOCKER_PASSWORD` | Personal Access Token Docker Hub |

В качестве значения `DOCKER_PASSWORD` используется не обычный пароль, а Personal Access Token Docker Hub с правами, необходимыми для публикации Docker-образов.

Сам токен не записывается в код workflow, README или отчёт. GitHub Actions получает его через контекст `secrets`.

### 3. Настройка GitHub Actions

В репозитории был создан файл:

```text
.github/workflows/docker-build.yml
```

Содержимое workflow:

```yaml
name: Docker Build and Deploy

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: ./lab2
          push: true
          tags: haru1meow/my-flask-app:latest

      - name: Deploy
        run: echo "Deploying to production server..."
```

Workflow запускается автоматически при каждом `push` в ветку `main`.

Выполнение производится на виртуальном runner с операционной системой Ubuntu.

Основные этапы pipeline:

| Шаг                           | Действие                                                                |
| ----------------------------- | ----------------------------------------------------------------------- |
| `Checkout code`               | Загружает содержимое репозитория на runner                              |
| `Set up Docker Buildx`        | Настраивает Docker Buildx для сборки образа                             |
| `Log in to Docker Hub`        | Выполняет авторизацию в Docker Hub через GitHub Secrets                 |
| `Build and push Docker image` | Собирает Docker-образ из директории `lab2` и публикует его в Docker Hub |
| `Deploy`                      | Выводит демонстрационное сообщение о деплое                             |

Параметр:

```yaml
context: ./lab2
```

указывает директорию `lab2` как контекст сборки Docker-образа.

Параметр:

```yaml
push: true
```

включает автоматическую публикацию собранного образа в Docker Hub.

После успешной сборки публикуется образ:

```text
haru1meow/my-flask-app:latest
```

Шаг `Deploy` является демонстрацией этапа деплоя. Он выводит сообщение в лог, но не выполняет подключение к реальному серверу.

### 4. Первый запуск и исправление ошибки авторизации

После добавления workflow и выполнения `push` в ветку `main` GitHub Actions автоматически запустил первый workflow.

Первый запуск завершился ошибкой на этапе авторизации в Docker Hub:

```text
unauthorized: incorrect username or password
```

Ошибка возникла из-за некорректных данных для авторизации в Docker Hub.

Для исправления проблемы в Docker Hub был создан Personal Access Token с правами **Read & Write**. После этого значение секрета `DOCKER_PASSWORD` в настройках GitHub Actions было заменено на созданный токен.

После обновления секрета был выполнен новый запуск workflow.

### 5. Проверка успешного запуска

После исправления авторизации следующий запуск GitHub Actions завершился успешно.

На странице **Actions** все этапы workflow были отмечены зелёными индикаторами.

В частности, успешно выполнились:

* получение исходного кода;
* настройка Docker Buildx;
* авторизация в Docker Hub;
* сборка Docker-образа;
* публикация образа в Docker Hub;
* демонстрационный шаг `Deploy`.

В логе последнего шага отображается сообщение:

```text
Deploying to production server...
```

После этого был выполнен дополнительный коммит с README:

```text
95cadad
```

с сообщением:

```text
Add Lab 2 README
```

Он также автоматически запустил workflow. Запуск **Docker Build and Deploy #3** завершился успешно.

Таким образом, CI/CD pipeline корректно реагирует на изменения в ветке `main` и автоматически выполняет сборку и публикацию Docker-образа.

### 6. Проверка Docker Hub и локального запуска

После успешного выполнения GitHub Actions в Docker Hub был опубликован тег:

```text
haru1meow/my-flask-app:latest
```

Это подтверждает, что GitHub Actions успешно собрал Docker-образ и отправил его в Docker Hub.

Для проверки получения опубликованного образа была выполнена команда:

```bash
docker pull --platform linux/amd64 haru1meow/my-flask-app:latest
```

Образ был успешно загружен.

Параметр `--platform linux/amd64` был использован, поскольку локальная машина работает на Apple Silicon, а workflow собирает образ для архитектуры `linux/amd64`.

После этого контейнер был запущен с публикацией порта:

```bash
docker run -p 5002:5000 --platform linux/amd64 haru1meow/my-flask-app:latest
```

После запуска в браузере по адресу `http://localhost:5002` было получено сообщение:

```text
Hello from Docker!
```

Это подтверждает, что опубликованный в Docker Hub образ можно скачать и запустить, а Flask-приложение внутри контейнера корректно обрабатывает HTTP-запросы.

## Результат работы

В ходе лабораторной работы был настроен CI/CD pipeline для Docker-приложения с использованием GitHub Actions и Docker Hub.

В результате:

* создан Docker-образ Flask-приложения;
* создан репозиторий `haru1meow/my-flask-app` в Docker Hub;
* настроена безопасная авторизация через GitHub Secrets;
* создан workflow GitHub Actions;
* настроена автоматическая сборка Docker-образа при `push` в `main`;
* настроена автоматическая публикация образа в Docker Hub;
* реал
