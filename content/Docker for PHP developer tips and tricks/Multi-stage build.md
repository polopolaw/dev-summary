Данная концепция в ходу уже не первый год, но до сих пор не так часто используется.
```
FROM golang:1.13-alpine3.12 as builder
COPY ./go.mod go.mod
RUN go mod download
COPY ./main.go .
RUN go build -v -o rr main.go

FROM php:7.4-cli-alpine3.11
COPY --chown=app-user:app-user --from=builder /usr/bin/rr /var/server/
```

Что тут происходит.
Мы берем как базовый образ `golang:1.13-alpine3.12` нарекаем его `builder`

В нем мы выполняем какую-то логику:
```
COPY ./go.mod go.mod
RUN go mod download
COPY ./main.go .
RUN go build -v -o rr main.go
```
 Далее берем уже образ php копируем в него бинарник сервера из `builder` вот так:
```
 --from=builder /usr/bin/rr /var/server/
```

### Более распространенный пример:

```
FROM node:16-alpine3.16 as builder
COPY ./package.json /var/www
COPY ./yarn.lock /var/www
RUN yarn install --silent
COPY ./ /var/www
RUN yarn build

FROM nginx:1.23-alpine as fe-server
COPY --from=builder /var/www/build /usr/share/nginx/html
```

1. Берем node образ, в нем билдим статику для проекта `yarn build`
2. Собранную статику помещаем в чистый образ nginx

Как итог у нас получается легкий образ nginx который хранит только статитку без установки в него node.


#### Данная схема работает и для публичных образов и их тегов

```
COPY --from=composer:2.3 /usr/bin/composer /usr/bin/composer
```
 Данная конструкция скопирует уже установленный composer  в наш образ. Без этого ранее приходилось делать что-то похожее на это:

```
ENV PATH "/composer/vendor/bin:$PATH"
ENV COMPOSER_HOME /composer
ENV COMPOSER_VERSION 1.10.19
RUN EXPECTED_COMPOSER_SIGNATURE=$(wget -q -O - https://composer.github.io/installer.sig) \
&& php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
&& php -r "if (hash_file('SHA384', 'composer-setup.php') === '${EXPECTED_COMPOSER_SIGNATURE}') { echo
'Composer.phar Installer verified'; } else { echo 'Composer.phar Installer corrupt';
unlink('composer-setup.php'); } echo PHP_EOL;" \
&& php composer-setup.php --no-ansi --install-dir=/usr/bin --filename=composer
--version=${COMPOSER_VERSION} \
&& php -r "unlink('composer-setup.php');"&& composer global require hirak/prestissimo && \
composer clear-cache
```
Плохо читаемые команды и лишние слои в нашем образе.
 