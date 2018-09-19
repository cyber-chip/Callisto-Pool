# Callisto-Pool

Будем поднимать Pool Callisto

Руководство по созданию собственного пула шахт Callisto

Основываясь на Linux

*************************************
Зависимости:

go >= 1.10

redis-server >= 2.8.0

nodejs >= 4 LTS

nginx

geth (multi-geth)

*************************************

Я настоятельно рекомендую использовать Ubuntu 16.04 LTS.
Перед установкой что либо на новый сервер нужно обновить сервак.

sudo apt-get update && sudo apt-get -y upgrade

sudo apt-get -y install software-properties-common libzmq3-dev pwgen

sudo apt-get -y install git libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-program-options-dev libboost-test-dev libboost-thread-dev libboost-all-dev unzip libminiupnpc-dev python-virtualenv

sudo apt-get -y install build-essential libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils

sudo apt-get -y install virtualenv

sudo add-apt-repository ppa:bitcoin/bitcoin
 
sudo apt-get update 

sudo apt-get -y install libdb4.8-dev libdb4.8++-dev 

sudo adduser User
*************************************
1) Установить go lang

$ sudo apt-get install -y build-essential golang-1.10-go unzip

$ sudo ln -s /usr/lib/go-1.10/bin/go /usr/local/bin/go

*************************************

2) Установка redis-сервера

$ sudo apt-get install redis-server

Проверим как работает redis server, для этого запустим его.

$ redis-server

Рекомендуется привязать ваш DB address 127.0.0.1 или к внутреннему ip. Также, пожалуйста, настройте пароль для повышенной безопасности!

*************************************

3) Установить nginx

$ sudo apt-get install nginx

образец конфигурации, расположенный в configs/nginx.default.example (HINT, отредактируйте и перейдите в /etc/nginx/sites-available/default)

*************************************

4) Установить NODE

Это установит последние узлы nodejs

$ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -

$ sudo apt-get install -y nodejs

*************************************

5) Установите мультигит

$ wget https://github.com/ethoxy/multi-geth/releases/download/v1.8.15/multi-geth-linux.zip

$ unzip multi-geth-linux.zip

$ sudo mv geth /usr/local/bin/geth

*************************************

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

*************************************

Затем запустите multi-geth следующими командами

$ sudo systemctl enable callisto

$ sudo systemctl start callisto

Если вы хотите отладить команду узла

$ sudo systemctl status callisto

*************************************

Запустить консоль

$ geth attach

Зарегистрировать учетную запись пула и открыть кошелек для транзакции. Этот процесс всегда требуется, когда узел кошелька перезапускается.

> personal.newAccount()
> personal.unlockAccount(eth.accounts[0],"password",40000000)

*************************************

Установка Callisto Pool

$ git clone https://github.com/chainkorea/open-callisto-pool
$ cd open-callisto-pool
$ make all

Если вы столкнулись с  ls ~/open-callisto-pool/build/bin/, установка завершена.

$ ls ~/open-callisto-pool/build/bin/

*************************************

Настройка пула Callisto

$ mv config.example.json config.json
$ nano config.json

*************************************

Настройтесь на основе приведенных ниже команд.

{
  // The number of cores of CPU.
  "threads": 2,
  // Prefix for keys in redis store
  "coin": "clo",
  // Give unique name to each instance
  "name": "main",
  // PPLNS rounds
  "pplns": 9000,

  "proxy": {
    "enabled": true,

    // Bind HTTP mining endpoint to this IP:PORT
    "listen": "0.0.0.0:8888",

    // Allow only this header and body size of HTTP request from miners
    "limitHeadersSize": 1024,
    "limitBodySize": 256,

    /* Set to true if you are behind CloudFlare (not recommended) or behind http-reverse
      proxy to enable IP detection from X-Forwarded-For header.
      Advanced users only. It's tricky to make it right and secure.
    */
    "behindReverseProxy": false,

    // Stratum mining endpoint
    "stratum": {
      "enabled": true,
      // Bind stratum mining socket to this IP:PORT
      "listen": "0.0.0.0:8008",
      "timeout": "120s",
      "maxConn": 8192
    },

    // Try to get new job from geth in this interval
    "blockRefreshInterval": "120ms",
    "stateUpdateInterval": "3s",
    // If there are many rejects because of heavy hash, difficulty should be increased properly.
    "difficulty": 2000000000,

    /* Reply error to miner instead of job if redis is unavailable.
      Should save electricity to miners if pool is sick and they didn't set up failovers.
    */
    "healthCheck": true,
    // Mark pool sick after this number of redis failures.
    "maxFails": 100,
    // TTL for workers stats, usually should be equal to large hashrate window from API section
    "hashrateExpiration": "3h",

    "policy": {
      "workers": 8,
      "resetInterval": "60m",
      "refreshInterval": "1m",

      "banning": {
        "enabled": false,
        /* Name of ipset for banning.
        Check http://ipset.netfilter.org/ documentation.
        */
        "ipset": "blacklist",
        // Remove ban after this amount of time
        "timeout": 1800,
        // Percent of invalid shares from all shares to ban miner
        "invalidPercent": 30,
        // Check after after miner submitted this number of shares
        "checkThreshold": 30,
        // Bad miner after this number of malformed requests
        "malformedLimit": 5
      },
      // Connection rate limit
      "limits": {
        "enabled": false,
        // Number of initial connections
        "limit": 30,
        "grace": "5m",
        // Increase allowed number of connections on each valid share
        "limitJump": 10
      }
    }
  },

  // Provides JSON data for frontend which is static website
  "api": {
    "enabled": true,
    "listen": "0.0.0.0:8080",
    // Collect miners stats (hashrate, ...) in this interval
    "statsCollectInterval": "5s",
    // Purge stale stats interval
    "purgeInterval": "10m",
    // Fast hashrate estimation window for each miner from it's shares
    "hashrateWindow": "30m",
    // Long and precise hashrate from shares, 3h is cool, keep it
    "hashrateLargeWindow": "3h",
    // Collect stats for shares/diff ratio for this number of blocks
    "luckWindow": [64, 128, 256],
    // Max number of payments to display in frontend
    "payments": 50,
    // Max numbers of blocks to display in frontend
    "blocks": 50,
    // Frontend Chart related settings
    "poolCharts":"0 */20 * * * *",
    "poolChartsNum":74,
    "minerCharts":"0 */20 * * * *",
    "minerChartsNum":74

    /* If you are running API node on a different server where this module
      is reading data from redis writeable slave, you must run an api instance with this option enabled in order to purge hashrate stats from main redis node.
      Only redis writeable slave will work properly if you are distributing using redis slaves.
      Very advanced. Usually all modules should share same redis instance.
    */
    "purgeOnly": false
  },

  // Check health of each geth node in this interval
  "upstreamCheckInterval": "5s",

  /* List of geth nodes to poll for new jobs. Pool will try to get work from
    first alive one and check in background for failed to back up.
    Current block template of the pool is always cached in RAM indeed.
  */
  "upstream": [
    {
      "name": "main",
      "url": "http://127.0.0.1:8545",
      "timeout": "10s"
    },
    {
      "name": "backup",
      "url": "http://127.0.0.2:8545",
      "timeout": "10s"
    }
  ],

  // This is standard redis connection options
  "redis": {
    // Where your redis instance is listening for commands
    "endpoint": "127.0.0.1:6379",
    "poolSize": 10,
    "database": 0,
    "password": ""
  },

  // This module periodically remits ether to miners
  "unlocker": {
    "enabled": false,
    // Pool fee percentage
    "poolFee": 1.0,
    // the address is for pool fee. Personal wallet is recommended to prevent from server hacking.
    "poolFeeAddress": "",
    // Amount of donation to a pool maker. 10 percent of pool fee is donated to a pool maker now. If pool fee is 1 percent, 0.1 percent which is 10 percent of pool fee should be donated to a pool maker.
    "donate": true,
    // Unlock only if this number of blocks mined back
    "depth": 120,
    // Simply don't touch this option
    "immatureDepth": 20,
    // Keep mined transaction fees as pool fees
    "keepTxFees": false,
    // Run unlocker in this interval
    "interval": "10m",
    // Geth instance node rpc endpoint for unlocking blocks
    "daemon": "http://127.0.0.1:8545",
    // Rise error if can't reach geth in this amount of time
    "timeout": "10s"
  },

  // Pay out miners using this module
  "payouts": {
    "enabled": true,
    // Require minimum number of peers on node
    "requirePeers": 5,
    // Run payouts in this interval
    "interval": "12h",
    // Geth instance node rpc endpoint for payouts processing
    "daemon": "http://127.0.0.1:8545",
    // Rise error if can't reach geth in this amount of time
    "timeout": "10s",
    // Address with pool coinbase wallet address.
    "address": "0x0",
    // Let geth to determine gas and gasPrice
    "autoGas": true,
    // Gas amount and price for payout tx (advanced users only)
    "gas": "21000",
    "gasPrice": "50000000000",
    // The minimum distribution of mining reward. It is 1 CLO now.
    "threshold": 1000000000,
    // Perform BGSAVE on Redis after successful payouts session
    "bgsave": false
    "concurrentTx": 10
  }
}

*************************************

Для Продвинутых пользователий *********************************

Если вы распространяете развертывание пула на несколько серверов или процессов, создайте несколько конфигураций и отключите ненужные модули на каждом сервере. 

Я рекомендую эту стратегию развертывания:

Mining Интерфейс - 1x (это зависит, вы можете запустить один узел для ЕС, один для США, один для Азии)

Интерфейс Разблокировки и выплаты строго по 1 раз!

API Интерфейс - 1x

Запуск пула

За Пулом требуется сервисное обслуживание. Если не следить, терминал может быть остановлен, а пул не работает.

$ sudo nano /etc/systemd/system/etherpool.service

Скопируйте следующий пример

[Unit]
Description=Etherpool
After=callisto.target

[Service]
Type=simple
ExecStart=/home/<your-user-name>/open-callisto-pool/build/bin/open-callisto-pool /home/<your-user-name>/open-callisto-pool/config.json

[Install]
WantedBy=multi-user.target


*************************************






















