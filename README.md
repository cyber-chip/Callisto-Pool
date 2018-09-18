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

Установить NODE
Это установит последние узлы nodejs

$ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
$ sudo apt-get install -y nodejs

Установите мультигит

$ wget https://github.com/ethereumsocial/multi-geth/releases/download/v1.8.10/multi-geth-linux-v1.8.10.zip
$ unzip multi-geth-linux-v1.8.10.zip
$ sudo mv geth /usr/local/bin/geth

********************************************************
Запустить мультигит
Если вы используете Ubuntu, проще управлять услугами, используя сервисные услуги.

$ sudo nano /etc/systemd/system/callisto.service

Скопируйте следующий пример

[Unit]
Description=Callisto for Pool
After=network-online.target

[Service]
ExecStart=/usr/local/bin/geth --callisto --cache=1024 --rpc --extradata "Mined by <your-pool-domain>" --ethstats "<your-pool-domain>:Callisto@clostats.net"
User=<your-user-name>

[Install]
WantedBy=multi-user.target










