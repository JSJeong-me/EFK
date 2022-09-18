### Docker-compose EFK

### EFK 실행 Docker-compose

    $ docker-compose up -d
    
    # Checking logs for service fluentd
    $ docker-compose logs fluentd
    
    # Checking logs for service kibana
    $ docker-compose logs kibana
    
    # Elasticsearch + Fluentd + Kibana 컨테이너 ID 확인
    $ docker-compose ps
    
    
-----

### KQL

    https://www.elastic.co/guide/en/kibana/current/kuery-query.html

### Visulization

    https://www.elastic.co/guide/en/kibana/current/dashboard.html
