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

### Fluentd Tag Test : Matching

    <source>
      @type http
      port 8888
      bind 0.0.0.0
    </source>


    <match test.cycle>
      @type stdout
    </match>


    $ docker run -p 8888:8888 -v /home/me/tmp:/fluentd/etc fluent/fluentd:edge-debian -c /fluentd/etc/fluentd.conf

    $ curl -i -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.cycle

    # Log 확인

--------

### Fluentd Filters 테스트

<source>
  @type http
  port 8888
  bind 0.0.0.0
</source>

<filter test.cycle>
  @type grep
  <exclude>
    key action
    pattern ^logout$
  </exclude>
</filter>

<match test.cycle>
  @type stdout
</match>


    $ docker run -p 8888:8888 -v /home/me/tmp:/fluentd/etc fluent/fluentd:edge-debian -c /fluentd/etc/fluentd.conf

    $ curl -i -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.cycle

    # Log 확인


    [참조: https://docs.fluentd.org/quickstart/life-of-a-fluentd-event]
--------

### EFK (Elasticsearch + Fluentd + Kibana) - HTTP logging

    
    $ docker-compose up -d
    
    $ docker-compose ps
    
    # Checking logs for service fluentd
    $ docker-compose logs fluentd

    # Checking logs for service kibana
    $ docker-compose logs kibana
    
    #  IP address 확인
    $ docker inspect efk_elasticsearch_1
    
    
    $ curl 172.18.0.2:9200
     http://localhost:5601

    # Configuring Kibana Index Pattern
    
    $ docker pull nginx:alpine
    
    # Docker Container with Fluentd Log Driver
    $ docker run --name nginx_container -d --log-driver=fluentd -p 8080:80 nginx:alpine
    
    
    # NGINX Log 생성
    $ curl localhost:8080
       
  
    # set up the index name pattern for Kibana. Specify fluentd-* to Index name or pattern and click Create
    
    # container_name : nginx_container 검색 KQL Kibana
    
    
    [참조:  https://adamtheautomator.com/efk-stack/]
    
    
    
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
