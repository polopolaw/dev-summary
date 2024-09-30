![[Pasted image 20240925122342.png]]

Одна из базовых концепций докера - cache in layers.
Каждая операция в докер файле которая ведет к изменению системы будь то `RUN` `COPY` они будут задавать новые слои у каждого будет вычислен хэш и если хэш меняется допустим на рисунке выше у команды `COPY`то в результате сборки все следующие слои инвалидируются, то есть собираются заново.

Учитывая эту особенность мы должны всегда следовать принципу пирамиды:
![[Pasted image 20240925123254.png]]
В начале Dockerfile помещаем медленные операции и то что редко меняется. В конце быстрые операции или часто меняющиеся. 

**Пример Layers Order**
```
FROM php:7.4-cli-alpine3.11

RUN apk update \
	&& apk add --no-cache git wget imagemagick-dev \
		${PHPIZE_DEPS} \
&& docker-php-ext-install pgsql pdo_pgsql gd
&& docker-php-source delete && \
apk del ${BUILD_DEPENDS} && rm -rf /var/lib/apt/lists/*


# Host system user id
ARG UID=1000
ARG GID=1000
# Add user so we don't need --no-sandbox.
RUN addgroup -g ${GID} -S app-user && adduser --uid ${UID} --ingroup
app-user -S -g app-user app-user
RUN mkdir -p && chown -R app-user:app-user /home/app-user \
&& chown -R app-user:app-user /app


# Copy rr server requirements
COPY --from=builder /usr/bin/rr /var/server/
COPY ./.rr.yaml /var/server/

COPY ./.docker/php/entrypoint.sh /var/www

COPY --from=composer:2.3 /usr/bin/composer /usr/bin/composer
COPY ./composer.* /var/www


RUN composer install --no-autoloader --no-suggest --no-scripts && \
composer clear-cache && \
composer dump --optimize --no-scripts

COPY --chown=app-app:app-user ./ /var/www
USER app-user
EXPOSE 8133
CMD ["/var/www/entrypoint.sh"]
```
В первой части мы устанавливает через apk add  некоторые приложения в том числе для работы с изображениями. Далее устанавливаем php-extensions и удаляем все лишнее в конце.

На следующем шаге создаем нового пользователя, чтобы не использовать стандартного.
Выдаем все необходимые права.

После, копируем бинарный файл roadrunner и его конфиг, копируем entrypoint который будет запускаться в корне контейнера.

Далее ставим composer и копируем файлы compose-lock.json и composer.json.

И в конце
Устанавливаем зависимости, копируем остальные файлы приложения.


Почему мы делаем сначала установку зависимостей и только потом копирование.  Дело в том что с большой вероятностью кеш будет инвалидирован и зависимости будут установлены снова. Если `composer.lock` и `composer.json` не поменялись то установки зависимостей не будет и слой загрузится из кеша.
![[Pasted image 20240925124805.png]]
