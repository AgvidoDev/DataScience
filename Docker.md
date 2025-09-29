сохрани в формате md
markdown
# 100 Базовых Команд Docker с Примерами | Полное Руководство

В этой статье мы рассмотрим базовые команды Docker с примерами, команды для работы с образами, контейнерами, Docker Compose, томами, сетями, команды для просмотра логов и мониторинга, а также команды для работы с Docker Hub.

## Содержание

- [Команды Жизненного Цикла Docker Контейнера](#команды-жизненного-цикла-docker-контейнера)
- [Базовые Команды Docker](#базовые-команды-docker)
- [1. Команды для Работы с Docker Образами](#1-команды-для-работы-с-docker-образами)
- [2. Команды для Работы с Docker Контейнерами](#2-команды-для-работы-с-docker-контейнерами)
- [3. Команды Docker Compose](#3-команды-docker-compose)
- [4. Команды для Работы с Docker Томами](#4-команды-для-работы-с-docker-томами)
- [5. Команды для Работы с Docker Сетями](#5-команды-для-работы-с-docker-сетями)
- [6. Команды для Логирования и Мониторинга Docker](#6-команды-для-логирования-и-мониторинга-docker)
- [7. Команды Очистки Docker](#7-команды-очистки-docker)
- [8. Команды для Работы с Docker Hub](#8-команды-для-работы-с-docker-hub)

## Команды Жизненного Цикла Docker Контейнера

Ниже приведены основные команды жизненного цикла для работы с Docker контейнерами:

- `docker create` - Создать контейнер из образа, но не запускать его
- `docker run` - Запустить контейнер из образа
- `docker pause` - Приостановить все процессы в контейнере
- `docker unpause` - Возобновить приостановленные процессы в контейнере
- `docker stop` - Остановить работающий контейнер (корректное завершение)
- `docker start` - Запустить остановленный контейнер
- `docker restart` - Перезапустить контейнер
- `docker attach` - Подключиться к работающему контейнеру
- `docker wait` - Блокировать терминал до остановки контейнера, затем вывести код выхода
- `docker rm` - Удалить контейнер
- `docker kill` - Принудительно остановить контейнер (немедленное завершение)

## Базовые Команды Docker

Ниже приведены часто используемые базовые команды Docker.

### 1) `docker` — Показать все доступные команды Docker

bash
docker [опция] [команда] [аргументы]
### 2) docker version — Показать версию Docker
bash
docker version
### 3) docker info — Показать общесистемную информацию о Docker
bash
docker info
### 4) docker pull — Скачать образ из репозитория
bash
docker pull ubuntu
### 5) docker build — Собрать Docker образ из Dockerfile
bash
docker build <опции> <путь_к_директории>
Если вы хотите собрать образ, используя файлы из текущей директории:

bash
docker build .
Добавить тег для образа:

bash
docker build -t fosstechnix/nodejs:1.0 .
### 6) docker run — Запустить контейнер из образа
bash
docker run -i -t ubuntu /bin/bash
-i — Запустить интерактивную сессию

-t — Выделить псевдо-TTY (терминал)

ubuntu — Образ, используемый для создания контейнера

bash — Команда, запускаемая внутри контейнера

Примечание: Контейнер остановится, когда вы выйдете из него командой exit. Чтобы контейнер работал в фоновом режиме, добавьте опцию -d.

Запуск контейнера в фоновом режиме с именем:

bash
docker run -i -t --name=Ubuntu-Linux -d ubuntu /bin/bash
Проброс портов (например, для веб-сервера):

bash
docker run -p 8080:80 -d nginx
После этого веб-сервер будет доступен по адресу http://localhost:8080.

### 7) docker commit — Создать новый образ из изменений в контейнере
bash
docker commit [опции] <ID_контейнера> [РЕПОЗИТОРИЙ[:ТЕГ]]
Зафиксируем изменения в существующем контейнере и создадим новый образ:

bash
docker commit 023828e786e0 ubuntu-apache
### 8) docker ps — Показать список контейнеров
Показать работающие контейнеры:

bash
docker ps
Показать все контейнеры (включая остановленные):

bash
docker ps -a
### 9) docker start — Запустить остановленный контейнер
bash
docker start <ID_контейнера>
### 10) docker stop — Остановить работающий контейнер
bash
docker stop <ID_контейнера>
### 11) docker logs — Показать логи контейнера
bash
docker logs <ID_контейнера>
### 12) docker rename — Переименовать контейнер
bash
docker rename <Старое_Имя> <Новое_Имя>
### 13) docker rm — Удалить контейнер
bash
docker rm <ID_контейнера>
Удалить все остановленные контейнеры:

bash
sudo docker rm -f $(sudo docker ps -a -q)
## 1. Команды для Работы с Docker Образами
Docker Image — это шаблон приложения, включающий бинарные файлы и библиотеки, необходимые для запуска контейнера.

### 1) docker build — Собрать образ из Dockerfile
bash
docker build <опции> <путь_к_директории>
### 2) docker pull — Скачать образ из реестра
bash
docker pull ubuntu
Скачать образ из приватного реестра:

bash
docker login docker-fosstechnix.com --username=ИМЯ_ПОЛЬЗОВАТЕЛЯ
docker pull docker-fosstechnix.com/nodejs
Скачать образ с определенным тегом:

bash
docker pull "имя_репозитория"/"имя_образа"[:тег]
### 3) docker tag — Добавить тег к образу
bash
docker tag nodejsdocker fosstechnix/nodejsdocker:v1.0
### 4) docker images — Показать список образов
bash
docker images
Или:

bash
docker image ls
Показать непомеченные (dangling) образы:

bash
docker images -f "dangling=true"
### 5) docker push — Загрузить образ в репозиторий
Пример загрузки в приватный реестр:

bash
docker login docker-fosstechnix.com --username=USERNAME
docker tag ubuntu docker.fosstechnix.com/linux/ubuntu:latest
docker push docker.fosstechnix.com/linux/ubuntu:latest
### 6) docker history — Показать историю создания образа
bash
docker history <ID_образа> --no-trunc
### 7) docker inspect — Показать детальную информацию об образе
bash
docker inspect ID_ОБРАЗА
### 8) docker save — Сохранить образ в tar-архив
bash
docker save ubuntu_image:тег | gzip > ubuntu_image.tar.gz
### 9) docker import — Создать образ из tarball-архива
bash
docker import ./ubuntu_image.tar.gz ubuntu:latest
### 10) docker export — Экспортировать файловую систему контейнера
bash
docker export ID_контейнера | gzip > new_container.tar.gz
### 11) docker load — Загрузить образ из tar-архива
bash
docker load < ubuntu_image.tar.gz
### 12) docker rmi — Удалить образ
bash
docker rmi ID_ОБРАЗА
Удалить все образы:

bash
docker rmi $(docker images -q)
## 2. Команды для Работы с Docker Контейнерами
Docker Container — это виртуальная среда выполнения, созданная из образа.

### 1) docker start — Запустить контейнер
bash
docker start ID_контейнера
### 2) docker stop — Остановить контейнер
bash
docker stop ID_контейнера
Остановить все контейнеры:

bash
docker stop $(docker ps -q)
### 3) docker restart — Перезапустить контейнер
bash
docker restart ID_контейнера
### 4) docker pause — Приостановить процессы в контейнере
bash
docker pause ID_контейнера
Docker pause vs stop?
pause приостанавливает процессы, stop отправляет сигнал на корректное завершение.

### 5) docker unpause — Возобновить процессы в контейнере
bash
docker unpause ID_контейнера
### 6) docker run — Создать и запустить контейнер из образа
Запуск в интерактивном режиме:

bash
docker run -i -t --name=Ubuntu-Linux ubuntu /bin/bash
Запуск в фоновом режиме:

bash
docker run -d --name=My-Backend-Container nginx
### 7) docker ps — Показать список контейнеров
bash
docker ps -a
### 8) docker exec — Выполнить команду внутри контейнера
Вход в оболочку контейнера:

bash
docker exec -it ID_контейнера /bin/bash
Обновить пакеты внутри контейнера:

bash
docker exec ID_контейнера apt-get update
### 9) docker logs — Показать логи контейнера
bash
docker logs ID_контейнера
10) docker rename — Переименовать контейнер
bash
docker rename Старое_Имя Новое_Имя
### 11) docker rm — Удалить контейнер
bash
docker rm ID_контейнера
Удалить все остановленные контейнеры:

bash
docker rm $(docker ps -a -q)
### 12) docker inspect — Показать информацию о контейнере
bash
docker inspect ID_контейнера
Получить IP-адрес контейнера:

bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ИМЯ_КОНТЕЙНЕРА
### 13) docker attach — Подключиться к работающему контейнеру
bash
docker attach ID_контейнера
### 14) docker kill — Принудительно остановить контейнер
bash
docker kill ID_контейнера
### 15) docker cp — Копировать файлы между хостом и контейнером
С хоста в контейнер:

bash
docker cp ./file.txt ID_контейнера:/path/in/container/
Из контейнера на хост:

bash
docker cp ID_контейнера:/path/in/container/file.txt ./
## 3. Команды Docker Compose
Docker Compose используется для управления multi-контейнерными приложениями.

### 1) docker-compose build — Собрать образы
bash
docker-compose build
### 2) docker-compose up — Создать и запустить сервисы
bash
docker-compose up
Запуск в фоновом режиме:

bash
docker-compose up -d
### 3) docker-compose ps — Показать статус контейнеров
bash
docker-compose ps
### 4) docker-compose start — Запустить существующие контейнеры
bash
docker-compose start
В чем разница между up и start?
up создает и запускает контейнеры заново, start только запускает уже созданные.

### 5) docker-compose run — Запустить один конкретный сервис
bash
docker-compose run сервис
### 6) docker-compose rm — Удалить остановленные контейнеры
bash
docker-compose rm -f
### 7) docker-compose down — Остановить и удалить контейнеры
bash
docker-compose down
## 4. Команды для Работы с Docker Томами
### 1) docker volume create — Создать том
bash
docker volume create имя_тома
### 2) docker volume inspect — Показать информацию о томе
bash
docker volume inspect имя_тома
### 3) docker volume rm — Удалить том
bash
docker volume rm имя_тома
Удалить все неиспользуемые тома:

bash
docker volume prune
## 5. Команды для Работы с Docker Сетями
### 1) docker network create — Создать сеть
bash
docker network create --driver=bridge --subnet=192.168.100.0/24 br0
### 2) docker network ls — Показать список сетей
bash
docker network ls
### 3) docker network inspect — Показать информацию о сети
bash
docker network inspect bridge
##6. Команды для Логирования и Мониторинга Docker
### 1) docker ps -a — Показать все контейнеры
bash
docker ps -a
### 2) docker logs — Показать логи контейнера
bash
docker logs ID_контейнера
### 3) docker events — Показать события Docker
bash
docker events
### 4) docker top — Показать процессы в контейнере
bash
docker top ID_контейнера
### 5) docker stats — Показать статистику использования ресурсов
bash
docker stats ID_контейнера
### 6) docker port — Показать проброшенные порты
bash
docker port ID_контейнера
### 7. Команды Очистки Docker
Удалить все неиспользуемые (dangling) данные:

bash
docker system prune
Удалить ВСЕ неиспользуемые данные:

bash
docker system prune -a
Удалить неиспользуемые образы:

bash
docker image prune
Удалить неиспользуемые контейнеры:

bash
docker container prune
Удалить неиспользуемые тома:

bash
docker volume prune
Удалить неиспользуемые сети:

bash
docker network prune
## 8. Команды для Работы с Docker Hub
### 1) docker search — Найти образ в Docker Hub
bash
docker search ubuntu
### 2) docker pull — Скачать образ из Docker Hub
bash
docker pull ubuntu
### 3) docker login — Войти в Docker Hub
bash
docker login
### 4) docker push — Загрузить свой образ в Docker Hub
Сначала добавьте тег:

bash
docker tag мой_образ username/мой_образ:тег
Затем выполните push:

bash
docker push username/мой_образ:тег
## 5) docker logout — Выйти из Docker Hub
bash
docker logout
Заключение
В этом руководстве мы рассмотрели основные команды Docker для управления жизненным циклом контейнеров, работы с образами, контейнерами, Docker Compose, томами, сетями, мониторингом, очисткой и взаимодействием с Docker Hub. Эти команды составляют основу для эффективной работы с Docker.
