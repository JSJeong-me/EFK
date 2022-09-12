### File log 연동

Reading logs from a file we need an application that writes logs to a file. <br/>

```
cd monitoring\logging\fluentd\introduction\

docker-compose up -d file-myapp

```

To collect the logs, lets start fluentd

```
docker-compose up -d fluentd
```

## Collecting logs over HTTP (incoming)

```
cd monitoring\logging\fluentd\introduction\

docker-compose up -d http-myapp

```
-----

### HTTP log 연동




-----

### Fluentd MySQL slow log 연동

### 설치 및 사전 준비

    $ gpasswd mysql -a td-agent

    $ apt install -y ruby ruby-dev libc6-dev

    # 현재 서버에서 사용중인 gem 레포지가 
    # td-agent-gem 인지 fluentd-gem 인지 gem 인지 먼저 확인하세요 
    $ td-agent-gem install fluent-plugin-mysqlslowquery

### 설정

    # INPUT
    <source>
    @type mysql_slow_query
    @id MYSQL_SLOW_LOG

    tag mysql.slow
    path /mnt/vdb/mysql/data/slow.log
    pos_file /var/log/td-agent/mysql-slow.log.pos

    <parse>
        @type none
    </parse>
    </source>


### 실행

    /etc/td-agent/td-agent-db-*.conf 설정 파일 수정

    service td-agent stop
    service td-agent start
