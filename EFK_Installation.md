Insatallation

Elasticsearch

https://www.elastic.co/kr/downloads/elasticsearch



Kibana

https://www.elastic.co/kr/downloads/kibana


    $ sudo apt-get update

    $ sudo apt-get install gzip
    
    $ gzip -d elasticsearch-8.4.1-linux-x86_64.tar.gz
    
    $ tar xvf elasticsearch-8.4.1-linux-x86_64.tar
    
    
    $ gzip -d kibana-8.4.1-linux-x86_64.tar.gz
    
    $ tar xvf kibana-8.4.1-linux-x86_64.tar


-----

        elasticsearch.yml

        # Enable security features
        xpack.security.enabled: false

        xpack.security.enrollment.enabled: false

        # Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
        xpack.security.http.ssl:
          enabled: false
          keystore.path: certs/http.p12

        # Enable encryption and mutual authentication between cluster nodes
        xpack.security.transport.ssl:
          enabled: false
          verification_mode: certificate
          keystore.path: certs/transport.p12
          truststore.path: certs/transport.p12
        # Create a new cluster with the current node only
        # Additional nodes can still join the cluster later
        cluster.initial_master_nodes: ["USER"]

-----
        ### 설치확인

        * http://localhost:5601/ - Kibana Web UI interface
        * http://localhost:9200/ - Elastic Search API

