---
title:       "Базовая работа с Nginx и Apache2"
subtitle:    ""
description: "nginx — это HTTP-сервер и обратный прокси-сервер, почтовый прокси-сервер, а также TCP/UDP прокси-сервер общего назначения, изначально написанный Игорем Сысоевым."
date:        2023-09-01T22:05:11+03:00
author:      "Георгий Кузора"
image:       "img/nginx_basic.webp"
tags:        ["Nginx", "Apache2", "Linux", "Server", "PHP", "HTTP"]
categories:  ["Tech"]
toc:         true
draft:       false
---
## Установка Nginx и настройка работы с PHP-FPM

Для установки Nginx и php-fpm используем менеджер пакетов `apt`:

```shell
sudo apt install nginx php8.1 php8.1-fpm
```

Проверим что nginx и php-fpm запустились:

```shell
ps auxf
```

Перейдем в папку `/etc/nginx/`:

```shell
cd /etc/nginx/
```

Отредактируем файл конфигурации nginx нашего сайта `/etc/nginx/sites-enabled/default`:

```shell
vi /etc/nginx/sites-enabled/default
```

Добавим конфигурацию в файл `/etc/nginx/sites-enabled/default`:

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;
    server_name _;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

Добавим также конфигурацию для работы сервера php-fpm в файл `/etc/nginx/sites-enabled/default`:

```nginx
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    root /var/www/html;
    fastcgi_pass unix:/run/php/php8.1-fpm.sock;
}
```

Проверим что новая конфигурация не имеет ошибок:

```shell
sudo nginx -t
```

Ошибок нет. Перезапустим сервер:

```shell
sudo systemctl reload nginx
```

Проверим что сервер возвращает страницу по умолчанию:

```shell
curl localhost:80
```

Для проверки работы сервера php-fpm создадим файл `info.php` в директории `/var/www/html`. Содежимое файла:

```php
<?php
phpinfo();
?>
```

Для проверки того что сервер php-fpm работает как надо запросим страницу `info.php`:

```shell
curl localhost/info.php
```

## Установка Apache2. Настроить обработки PHP

Установим сервер `apache2` и модуль `libapache2-mod-php8.1`:

```shell
sudo apt install apache2 libapache2-mod-php8.1
```

Изменим порт на котором работает сервер `apache2`, так как порт 80 уже занят сервером nginx. Перейдем в директорию `/etc/apache2`:

```shell
cd /etc/apache2
```

Изменим файл конфигурации портов `ports.conf`. Внесем в него следующую конфигурацию:

```xml
Listen 8080
```

Проверим что конфигурация работает и запустим сервер `apache2`:

```shell
sudo apachectl -t
sudo systemctl start apache2
sudo systemctl status apache2
```

Проверим что сервер `apache2` возвращает страницу по умолчанию на порту 8080:

```shell
curl localhost:8080
```

Проверим что сервер `apache2` обрабатывает php и возвращает страницу `info.php` на порту 8080:

```shell
curl localhost:8080/info.php
```

## Настройка схемы обратного прокси для Nginx (динамика — на Apache2)

Чтобы настроить схему обратного прокси для Nginx добавим кофигурацию в файл `/etc/nginx/sites-enabled/default`:

```shell
vi `/etc/nginx/sites-enabled/default`
```

```nginx
location / {
    proxy_pass http://localhost:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;
}
location ~* ^.+.(jpg|jpeg|gif|png|ico|css|zip|pdf|txt|tar|js)$ {
    root /var/www/html;
}
```

Закоментируем предыдущую конфигурацию для сервера php-fpm:

```nginx
# location ~ \.php$ {
#       include snippets/fastcgi-php.conf;
#       root /var/www/html;
#       fastcgi_pass unix:/run/php/php8.1-fpm.sock;
# }
```

Закоментируем предыдущую конфигурацию запроса файлов:

```nginx
# location / {
#       # First attempt to serve request as file, then
#       # as directory, then fall back to displaying a 404.
#       try_files $uri $uri/ =404;
# }
```

Проверим что новая конфигурация не имеет ошибок:

```shell
sudo nginx -t
```

Ошибок нет. Перезапустим сервер:

```shell
sudo systemctl reload nginx
```

Проверим что сервер `nginx` возвращает страницу по умолчанию:

```shell
curl localhost
```

Проверим что сервер `nginx` возвращает страницу `info.php`:

```shell
curl localhost/info.php
```

## Настройка схемы балансировки трафика между несколькими серверами Apache на стороне Nginx с помощью модуля `ngx_http_upstream_module`

Укажем порт, на котором будет работать второй сервер Apache. Для этого откроем файл конфигурации Apache:

```shell
sudo vi /etc/apache2/ports.conf
```

Добавим в него строку

```
Listen 8081
```

Создадим новый файл конфигурации для второго сервера Apache:

```shell
sudo vi /etc/apache2/sites-available/second-server.conf
```

Внесем следующий код в файл конфигурации:

```xml
   <VirtualHost *:8081>
       ServerAdmin webmaster@localhost
       DocumentRoot /var/www/second-server
       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
```

Активируем файл конфигурации второго сервера apache2:

```shell
sudo a2ensite second-server.conf
```

Проверим что конфигурация работает и перезапустим веб-сервер:

```shell
sudo apachectl -t
sudo systemctl reload apache2
```

Добавим простой html документ в директорию `/var/www/second-server`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>Hello world!</h1>
</body>
</html>
```

Определим блок upstream в файле `etc/nginx/sites-enabled/default`. Для определения серверов Apache, которые будут балансироваться:

```nginx
upstream apache_servers {
    server localhost:8080;
    server  localhost:8081;
}
```

Определим блок `server` для определения порта и хоста `nginx`, который будет использоваться для балансировки трафика:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://apache_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Проверим конфигурацию и перезапустим Nginx для применения изменений:

```shell
sudo nginx -t
sudo systemctl reload nginx
```

Теперь Nginx будет балансировать трафик между серверами Apache, которые были определены в блоке upstream.
