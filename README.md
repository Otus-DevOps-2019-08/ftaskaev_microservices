[![Build Status](https://travis-ci.com/Otus-DevOps-2019-08/ftaskaev_microservices.svg?branch=master)](https://travis-ci.com/Otus-DevOps-2019-08/ftaskaev_microservices)
# [Otus-DevOps-2019-08] Fedor Taskaev homeworks
Выполнение домашних заданий по курсу [DevOps практики и инструменты](https://otus.ru/lessons/devops-praktiki-i-instrumenty/) (август 2019 - март 2020).

## Памятка

Очистить место после выполнения работ:

```console
$ docker rm $(docker ps -a -q)
$ docker rmi $(docker images -q)
```

Проверить занимаемое место:

```console
$ docker system df
```

Удалить VM с docker в GCE:

```console
$ docker-machine rm docker-host
```

## Lesson 15: homework 12
Docker: Запуск VM с установленным Docker Engine при помощи Docker Machine. Написание Dockerfile и сборка образа с тестовым приложением. Сохранение образа на DockerHub.  
PR: [Otus-DevOps-2019-08/ftaskaev_microservices#1](https://github.com/Otus-DevOps-2019-08/ftaskaev_microservices/pull/1)

Устанавливаем ПО:

```console
$ docker -v
Docker version 19.03.5, build 633a0ea
```
```console
$ docker-compose --version
docker-compose version 1.25.1, build unknown
```
```console
$ docker-machine -v
docker-machine version 0.16.2, build bd45ab1
```

Запускаем Docker Machine:

```console
$ docker-machine create -d virtualbox docker-machine
$ eval $(docker-machine env docker-machine)
```

Проверяем работу Docker Engine:

```console
$ docker version
Client: Docker Engine - Community
 Version:           19.03.5
 API version:       1.40
 Go version:        go1.13.4
 Git commit:        633a0ea
 Built:             Thu Nov 14 23:51:40 2019
 OS/Arch:           darwin/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.5
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.12
  Git commit:       633a0ea838
  Built:            Wed Nov 13 07:28:45 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.2.10
  GitCommit:        b34a5c8af56e510852c35414db4c1f4fa6172339
 runc:
  Version:          1.0.0-rc8+dev
  GitCommit:        3e425f80a8c931f88e6d94a8c831b9d5aa481657
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

Создаем образ из контейнера:

```console
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
5e7e55456117        ubuntu:16.04        "/bin/bash"         6 minutes ago       Exited (0) 3 seconds ago                       loving_wilson
fa6ec3721a34        hello-world         "/hello"            8 minutes ago       Exited (0) 8 minutes ago                       great_lovelace
```
```console
$ docker commit 5e7e55456117 ftaskaev/ubuntu-tmp-file
sha256:eba1598ab8b5a878bf29514db3b851b16d0db7ef2ba63878862b366941401a58
```
```console
$ docker images
REPOSITORY                 TAG                 IMAGE ID            CREATED              SIZE
ftaskaev/ubuntu-tmp-file   latest              eba1598ab8b5        About a minute ago   123MB
ubuntu                     16.04               c6a43cd4801e        3 weeks ago          123MB
hello-world                latest              fce289e99eb9        12 months ago        1.84kB
```

### CGE

Переключаем glocud на новый проект:

```console
$ gcloud config set project docker-264821
$ export GOOGLE_PROJECT=docker-264821
```

Запускаем VM с docker в GCE:

```console
$ docker-machine create --driver google \
 --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
 --google-machine-type n1-standard-1 \
 --google-zone europe-west1-b \
 docker-host
```
```console
$ eval $(docker-machine env docker-host)
```
```console
$ docker-machine ls
NAME             ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER     ERRORS
docker-host      *        google       Running   tcp://35.187.55.129:2376            v19.03.5
docker-machine   -        virtualbox   Running   tcp://192.168.99.101:2376           v19.03.5
```

Создаем образ из Dockerfile и запускаем контейнер на удалённом хосте:

```console
$ docker build -t reddit:latest .
$ docker run --name reddit -d --network=host reddit:latest
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
c3125f11d978        reddit:latest       "/start.sh"         3 minutes ago       Up 3 minutes                            reddit
```

### Docker Hub

Публикуем собранный образ в Docker Hub:

```console
$ docker tag reddit:latest sumkinstein/otus-reddit:1.0
$ docker push sumkinstein/otus-reddit:1.0
```

Теперь можно удалить все ранее созданные образы и заново запустить контейнер из Docker Hub:

```console
$ docker run --name reddit -d --network=host sumkinstein/otus-reddit:1.0
```

## Lesson 16: homework 13
Docker: Разбиение приложения на несколько микросервисов. Выбор базового образа. Подключение volume к контейнеру.  
PR: [Otus-DevOps-2019-08/ftaskaev_microservices#2](https://github.com/Otus-DevOps-2019-08/ftaskaev_microservices/pull/2)

Собираем образы:

```console
$ docker build -t sumkinstein/post:1.0 ./post-py
$ docker build -t sumkinstein/comment:1.0 ./comment
$ docker build -t sumkinstein/ui:1.0 ./ui
```

При сборке образа ui используется кеш от предыдущей сборки:

```console
$ docker build -t sumkinstein/comment:1.0 ./comment
Sending build context to Docker daemon  14.85kB
Step 1/11 : FROM ruby:2.2
 ---> 6c8e6f9667b2
Step 2/11 : RUN apt-get update -qq && apt-get install -y build-essential
 ---> 364261c482c4
Step 3/11 : ENV APP_HOME /app
 ---> f383419fb60c
Step 4/11 : RUN mkdir $APP_HOME
 ---> 2cb6a2cc83ac
Step 5/11 : WORKDIR $APP_HOME
 ---> f524c26183d6
```
```console
$ docker build -t sumkinstein/ui:1.0 ./ui
Sending build context to Docker daemon  30.72kB
Step 1/13 : FROM ruby:2.2
 ---> 6c8e6f9667b2
Step 2/13 : RUN apt-get update -qq && apt-get install -y build-essential
 ---> Using cache
 ---> 364261c482c4
Step 3/13 : ENV APP_HOME /app
 ---> Using cache
 ---> f383419fb60c
Step 4/13 : RUN mkdir $APP_HOME
 ---> Using cache
 ---> 2cb6a2cc83ac
Step 5/13 : WORKDIR $APP_HOME
 ---> Using cache
 ---> f524c26183d6
```

### Дополнительное задание #1

При запуске контейнера можно переопределить переменные, заданные в Dockerfile.  
Запустим mongo с `--network-alias=puma_db` и при запуске остальных контейнеров переопределим переменные, хранящие адрес сервера БД:

```console
docker run -d --network=reddit --network-alias=reddit_db mongo:latest
docker run -d --network=reddit --network-alias=post -e POST_DATABASE_HOST=reddit_db sumkinstein/post:1.0
docker run -d --network=reddit --network-alias=comment -e COMMENT_DATABASE_HOST=reddit_db sumkinstein/comment:1.0
```

### Дополнительное задание #2

Используя в качестве базового образ `ruby:2.2-alpine` итоговый образ уменьшился до ~300MB:

```console
$ docker images
REPOSITORY            TAG                 IMAGE ID            CREATED              SIZE
sumkinstein/comment   3.0                 59c0d38f254a        4 seconds ago        305MB
sumkinstein/ui        3.0                 a03eb25462e6        About a minute ago   308MB
sumkinstein/comment   2.0                 6dae48d0587a        About an hour ago    456MB
sumkinstein/ui        2.0                 2e75a60f2b30        About an hour ago    459MB
sumkinstein/comment   1.0                 0a7ad278f1b3        2 hours ago          782MB
sumkinstein/ui        1.0                 4919b19e441e        2 hours ago          784MB
```

### Docker volumes

Создаём volume:

```console
$ docker volume create reddit_db
```

Запускаем контейнер с подключенным volume:

```console
$ docker run -d --network=reddit --network-alias=reddit_db -v reddit_db:/data/db mongo:latest
```

