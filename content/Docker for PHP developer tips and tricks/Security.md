Представьте себе, что мы хотим поработать с каким-нибудь секретом во время build и потом от него избавиться.
```
FROM php:8.1-cli-alpine AS app_php
ARG SECRET
WORKDIR /var/www
RUN echo $SECRET > secret.json
RUN echo "Perform some work with it"
RUN rm secret.json
```

```
$ docker build --tag=meetup:1.0.1 --target=app_php --build-arg SECRET="I love PHP" .
```

Используя команду `RUN echo $SECRET > secret.json` записываем секрет в файл делаем какую-то работу с ним и после удаляем. `RUN rm secret.json`. Secret  у нас передается в команде выше.

Теперь если посмотреть результат работы команды
```
$ dive meetup:1.0.1
```
![[Pasted image 20240928143447.png]]

В истории сборки образа секрет уже совсем не секрет 😱
Так же на одном из слоев до удаления файла этот файл у нас лежит в файлах слоя.(нижняя правая часть скриншота).

Исправить одну из проблем можно объединив удаление и использование секрета в один слой примерно так:

```
Dockerfile:
FROM php:8.1-cli-alpine AS app_php
ARG SECRET
WORKDIR /var/www
RUN echo $SECRET > secret.json && echo "Perform some work with it" && rm secret.json
```

Но данное решение не убирает секрет из истории.
![[Pasted image 20240928143937.png]]

Здесь на помощь приходит новая фича в Docker она будет актальна в чуть-чуть изменившемся Dockerfile. Рассмотрим на более приближенном к реальности примере.

```
FROM php:8.1-cli-alpine AS app_php_plain
COPY --from=composer:2.1.14 /usr/bin/composer /usr/bin/composer
ENV COMPOSER_HOME=/tmp/composer
COPY composer.json .
RUN --mount=type=secret,id=auth.json,target=$COMPOSER_HOME/auth.json,required \
composer install --no-autoloader --no-suggest --no-scripts
```

Что тут происходит - добавлен флаг mount=type=secret в который мы передаем файл `auth.json`.

Секрет во время билда нужно передать так:
```
docker build --secret id=auth.json,src=auth.json --tag=meetup:1.0.1
--target=app_php_plain .
```
Теперь история у нас чистая:
![[Pasted image 20240928144838.png]]


Это не единственый mount=type= флаг

![[Pasted image 20240928145120.png]]

### RUN --mount=type=cache

```
FROM ubuntu:18.04
ENV PIP_CACHE_DIR=/var/cache/buildkit/pip
RUN mkdir -p $PIP_CACHE_DIR
RUN rm -f /etc/apt/apt.conf.d/docker-clean
RUN --mount=type=cache,target=/var/cache/apt \
apt-get update &&
apt-get install -yqq --no-install-recommends \
$YOUR_PACKAGES && rm -rf /var/lib/apt/lists/*
```

Флаг `--mount=type=cache` - предназначен для ускорения работы пакетных менеджеров применяя промежуточное кэширование.  Может работать и для composer, apt, apk

### RUN --mount=type=ssh

```
RUN mkdir -p -m 0700 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts
RUN --mount=type=ssh bundle install

FROM alpine
RUN apk add --no-cache openssh-client
RUN mkdir -p -m 0700 ~/.ssh && ssh-keyscan gitlab.com >> ~/.ssh/known_hosts
RUN --mount=type=ssh \
ssh -q -T git@gitlab.com 2>&1 | tee /hello
# "Welcome to GitLab, @GITLAB_USERNAME_ASSOCIATED_WITH_SSHKEY" should be printed here
```
Позволяет использовать ssh агента с хост машины чтобы не настраивать ключи в контейнере. В таком случае ключи и другие данные авторизации не ототобразятся в истории образа.