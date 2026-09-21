# Лабораторная работа 2

## CI/CD для Docker-приложения

**Студент:** Arutyun Surudzhyan  
**Университет:** ИТМО

### Цель работы

Настроить CI/CD pipeline для Docker-приложения с использованием GitHub Actions и Docker Hub.

### Используемые технологии

- Python
- Flask
- Docker
- Docker Hub
- GitHub Actions

### Структура проекта

- `app.py` — Flask-приложение
- `requirements.txt` — зависимости Python
- `Dockerfile` — инструкция для сборки Docker-образа
- `.github/workflows/docker-build.yml` — CI/CD workflow

### CI/CD pipeline

При каждом push в ветку `main` GitHub Actions:

1. Получает исходный код репозитория.
2. Настраивает Docker Buildx.
3. Выполняет авторизацию в Docker Hub.
4. Собирает Docker-образ.
5. Публикует образ в Docker Hub с тегом `latest`.
6. Выполняет шаг deploy.

### Docker Hub

Docker image:

`haru1meow/my-flask-app:latest`

### Проверка

Docker-образ был успешно опубликован в Docker Hub и скачан командой:

`docker pull --platform linux/amd64 haru1meow/my-flask-app:latest`

После запуска контейнера приложение успешно отобразило:

`Hello from Docker!`

### Результат

CI/CD pipeline успешно выполняется в GitHub Actions, а Docker-образ автоматически публикуется в Docker Hub.