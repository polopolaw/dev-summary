Рассмотрим пример:
```
FROM php:8.1-fpm-alpine AS app_php
COPY --from=mlocati/php-extension-installer --link /usr/bin/install-php-extensions /usr/local/bin/
COPY --from=composer/composer:2-bin --link /composer /usr/bin/composer
# LINES
FROM app_php AS app_php_dev
# LINES
FROM caddy:2.6-builder-alpine AS app_caddy_builder
# LINES
FROM caddy:2.6-alpine AS app_caddy
COPY --from=app_caddy_builder --link /usr/bin/caddy /usr/bin/caddy
COPY --from=app_php --link /srv/app/public public/
```
Берем alpine образ `php:8.1-fpm-alpine` и нарекаем его `app_php`. Устанавливаем в него все что будет общим для все окружений
```
COPY --from=mlocati/php-extension-installer --link /usr/bin/install-php-extensions /usr/local/bin/
COPY --from=composer/composer:2-bin --link /composer /usr/bin/composer
```

Далее мы можем написать вот так `FROM app_php AS app_php_dev` - используем базовый образ и ниже описываем только то что нужно сделать в dev окружении, например, установить Xdebug.

Командой `FROM caddy:2.6-builder-alpine AS app_caddy_builder` мы компилируем бинарник caddy веб-сервера.

И в последнем шаге копируем бинарник веб сервера и php файлы с vendor и другими необходимыми файлами. Копируем через команду [[Copy --link]]

## Что это дает

Для примера выше мы можем собрать не менее 4 разных образов:
Не используя никаких переменных и if

```
docker build -–target=app_caddy_builder .

docker build -–target=app_php .

docker build -–target=app_caddy .

docker build -–target=app_php_dev .
```

Если какой-то из образов зависит от других, то сначала будут собраны родительские и интегрированы в дочерние.

### Анти пример без использования Targets vs Targets

```
ARG XDEBUG_ENABLED=false
RUN if [ "${XDEBUG_ENABLED}" == "true" ]; then pecl install xdebug && docker-php-ext-enable
xdebug; fi
```
**VS**
```
FROM php:8.1-fpm-alpine AS app_php
# Install all common for all envs
FROM app_php AS app_php_dev
RUN pecl install xdebug && docker-php-ext-enable xdebug
```
```

$ docker build --tag=meetup:1.0.1 --target=app_php .
$ docker build --tag=meetup:1.0.1 --target=app_php_dev .
```
Во втором случае мы легко можем собрать образ под разные окружения.