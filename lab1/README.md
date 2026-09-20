Lab 1
Цель
Познакомиться с основами Docker: образами, контейнерами, портами, логами и volumes.
Выполненные задания

1. Установка и проверка Docker
Установлен Docker Desktop for Mac
Проверена версия Docker:
docker -version
Запущен тестовый контейнер:
docker run hello-world
Проверены команды docker images, docker ps и docker ps -a

2. Работа с Ubuntu
Загружен образ ubuntu:latest
Запущен интерактивный контейнер Ubuntu
Установлен curl
Проверена работа curl -version
3. Работа с Nginx
Запущен контейнер nginx:alpine
Настроено перенаправление порта 8080:80
Проверена работа Nginx через http://localhost:8080
Просмотрены логи контейнера с помощью docker logs
Выполнен вход в контейнер через docker exec

4. Управление контейнерами
Были использованы команды:
docker ps
docker ps -a
docker stop
docker start
docker rm
docker rmi

5. Работа с Docker Volume
Создан volume my-volume
Создан контейнер с подключённым volume
В /data создан файл test.txt.
После удаления первого контейнера был создан новый контейнер с тем же volume.
Файл test.txt сохранился и был успешно прочитан из нового контейнера.
Результат проверки:
Hello from volume

Результат
В ходе работы я изучил основные операции Docker: создание и запуск контейнеров, работа с образами, проброс портов, просмотр логов, выполнение команд внутри контейнера и сохранение данных с помощью volumes

## Star task

### Flask-приложение

Создан файл `app.py` с простым Flask-приложением.

### Зависимости

Создан файл `requirements.txt` с необходимыми Python-зависимостями.

### Dockerfile

Создан Dockerfile для сборки и запуска Flask-приложения.

### Сборка и запуск

Docker-образ собран командой:

`docker build -t my-flask-app .`

Контейнер запущен командой:

`docker run -d -p 5001:5000 --name flask-container my-flask-app`

### Проверка

Работоспособность приложения проверена командой:

`curl http://localhost:5001`

Результат:

`Hello from Docker!`

### Результат

Flask-приложение успешно собрано в Docker-образ и запущено в контейнере.