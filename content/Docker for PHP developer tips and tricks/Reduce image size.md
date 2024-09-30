Проведем мысленный эксперимент. Мы хотим уменьшить размер образа описанного ниже.

```
FROM php:7.4.0-cli-alpine

RUN apk update && apk add --no-cache wget libmcrypt-dev
RUN docker-php-ext-install iconv
RUN rm -rf /var/lib/apt/lists/ && \
docker-php-source delete \
&& apk del ${BUILD_DEPENDS}
```

На первый взгляд команда 

```
RUN rm -rf /var/lib/apt/lists/ && \
docker-php-source delete \
&& apk del ${BUILD_DEPENDS}
```

Удаляет лишнее в образе, но если посмотреть историю через `dive`
![[Pasted image 20240928150209.png]]

Мы видим что, размер не только не уменьшился, но и добавился еще один слой и дополнительный вес.

Чтобы по настоящему уменьшить, нужно использовать один общий слой для удаления.

```
FROM php:7.4.0-cli-alpine

RUN apk update && apk add --no-cache wget libmcrypt-dev && \
docker-php-ext-install iconv && \
rm -rf /var/lib/apt/lists/ && \
docker-php-source delete \
&& apk del ${BUILD_DEPENDS}
```

![[Pasted image 20240928150437.png]]
Мы получили вместо 3 слоев получили 1. И немного уменьшили общий размер.
![[Pasted image 20240928150454.png]]

Может показаться что это совсем небольшая экономия, но если будет более сложный пример с большим количеством установок и зависимостей, результат будет больше.