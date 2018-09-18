# Callisto-Pool
Будем поднимать Pool Callisto

Руководство по созданию собственного пула шахт Callisto

Основываясь на Linux
зависимости:

go >= 1.10

redis-server >= 2.8.0

nodejs >= 4 LTS

nginx

geth (multi-geth)

Я настоятельно рекомендую использовать Ubuntu 16.04 LTS.

Установить go lang

$ sudo apt-get install -y build-essential golang-1.10-go unzip
$ sudo ln -s /usr/lib/go-1.10/bin/go /usr/local/bin/go

Установка redis-сервера

$ sudo apt-get install redis-server

Рекомендуется привязать ваш DB address 127.0.0.1 или к внутреннему ip. Также, пожалуйста, настройте пароль для повышенной безопасности!

Установить nginx

$ sudo apt-get install nginx

образец конфигурации, расположенный в configs/nginx.default.example (HINT, отредактируйте и перейдите в /etc/nginx/sites-available/default)
