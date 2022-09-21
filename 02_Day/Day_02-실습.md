### Fluentd Install:  Docker Image 테스트->  records from http, and outputs to stdout

    $ docker pull fluent/fluentd:edge-debian
    
    $ ls -l /home/me/tmp/fluentd.conf
    
        <source>
          @type http
          port 9880
          bind 0.0.0.0
        </source>

        <match **>
          @type stdout
        </match>    


    # Docker 실행
    
    $ docker run -p 9880:9880 -v /home/me/tmp:/fluentd/etc fluent/fluentd:edge-debian -c /fluentd/etc/fluentd.conf
    
    
    # Post Sample Logs via HTTP
    
    $ curl -X POST -d 'json={"json":"message"}' http://127.0.0.1:9880/sample.test
    
    $ docker ps -a
    
    $ docker logs <docker container ID> | tail -n 1

    $ docker stop <docker container ID>


    [참조: https://docs.fluentd.org/container-deployment/install-by-docker]
-------

### EFK (Elasticsearch + Fluentd + Kibana) 

    https://github.com/JSJeong-me/EFK/tree/main/01_Day/fluentd-elastic-kibana
    
    [참조: https://docs.fluentd.org/container-deployment/docker-compose]
    
    
-----
    
    
    









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
