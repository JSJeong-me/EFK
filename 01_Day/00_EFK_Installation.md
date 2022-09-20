### Elasticsearch & Kibana Insatallation

    $ sudo apt-get update

    Elasticsearch

    https://www.elastic.co/kr/downloads/elasticsearch
    
    $ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.1-linux-x86_64.tar.gz



    Kibana

    https://www.elastic.co/kr/downloads/kibana
    
    $ wget https://artifacts.elastic.co/downloads/kibana/kibana-8.4.1-linux-x86_64.tar.gz


  
    $ tar xvfz elasticsearch-8.4.1-linux-x86_64.tar.gz
    
    $ tar xvfz kibana-8.4.1-linux-x86_64.tar.gz
    
-----

### 설치전 환경 설정

    /etc/security/limits.conf 파일에 (max file descriptord 4096 에서 65536 으로 증가)
    
    {리눅스계정} - nofile 65535
    
       
    
    /etc/sysctl.conf 파일에 아래 내용 추가 (virtual memory 영역을 65530 에서 262144 로 증가)
    
    
    
### Fluentd Docker 실행 테스트을 위해 fluentd.conf 준비


    $ /home/$(USER)/tmp/fluentd.conf

        <source>
          @type http
          port 9880
          bind 0.0.0.0
        </source>

        <match **>
          @type stdout
        </match>
    
    $ docker run -p 9880:9880 -v /home/me/tmp:/fluentd/etc fluent/fluentd:edge-debian -c /fluentd/etc/fluentd.conf
    
    
    Post Sample Logs via HTTP
    
    $ curl -X POST -d 'json={"json":"message"}' http://127.0.0.1:9880/sample.test
    
    $ docker ps -a
    
        CONTAINER ID   IMAGE                        COMMAND                  CREATED         STATUS                           PORTS                                         NAMES
    679c05ffaca3   fluent/fluentd:edge-debian   "tini -- /bin/entryp…"   2 minutes ago   Up 2 minutes                     5140/tcp, 24224/tcp, 0.0.0.0:9880->9880/tcp   vigilant_golick
    
    $ docker logs 775a8e192f2b | tail -n 1
    
    
    
    
    
    
    
    
    
    vm.max_map_count=262144
    
    
    (https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#limits.conf)

-----
###     [중요] https 설정 disable
        
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
        * $ curl -XGET localhost:9200
-----

### Fluentd Insatallation

# 설치전 작업

1. Increase the Maximum Number of File Descriptors

    $ ulimit -n
    65535

/etc/security/limits.conf file and reboot your machine:


    root soft nofile 65536
    root hard nofile 65536
    * soft nofile 65536
    * hard nofile 65536

2. Optimize the Network Kernel Parameters

/etc/sysctl.conf file:

        net.core.somaxconn = 1024
        net.core.netdev_max_backlog = 5000
        net.core.rmem_max = 16777216
        net.core.wmem_max = 16777216
        net.ipv4.tcp_wmem = 4096 12582912 16777216
        net.ipv4.tcp_rmem = 4096 12582912 16777216
        net.ipv4.tcp_max_syn_backlog = 8096
        net.ipv4.tcp_slow_start_after_idle = 0
        net.ipv4.tcp_tw_reuse = 1
        net.ipv4.ip_local_port_range = 10240 65535
        # If forward uses port 24224, reserve that port number for use as an ephemeral port.
        # If another port, e.g., monitor_agent uses port 24220, add a comma-separated list of port numbers.
        # net.ipv4.ip_local_reserved_ports = 24220,24224
        net.ipv4.ip_local_reserved_ports = 24224
    
    
        Use sysctl -p command or reboot your node
        
        
3. Use sticky bit symlink/hardlink protection


    Fluentd sometimes uses predictable paths for dumping, writing files, and so on. This default settings for the protections are in /etc/sysctl.d/10-link-restrictions.conf, or /usr/lib/sysctl.d/50-default.conf or elsewhere.
    For symlink attack protection, check the following parameters are set to 1:

    
    fs.protected_hardlinks = 1
    fs.protected_symlinks = 1


     Use sysctl -p command or reboot your node
     
     [출처: https://docs.fluentd.org/installation/before-install]
     
     
### td-agent: the stable distribution of Fluentd

(Ubuntu 설치)

### Step 1: Install from Apt Repository

    $ cat /etc/os-release  확인

    NAME="Ubuntu"
    VERSION="20.04.4 LTS (Focal Fossa)"
    ID=ubuntu
    ID_LIKE=debian
    PRETTY_NAME="Ubuntu 20.04.4 LTS"
    VERSION_ID="20.04"
    HOME_URL="https://www.ubuntu.com/"
    SUPPORT_URL="https://help.ubuntu.com/"
    BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
    PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
    VERSION_CODENAME=focal
    UBUNTU_CODENAME=focal
    
    
    For Ubuntu Focal:
    
    # td-agent 4
    curl -fsSL https://toolbelt.treasuredata.com/sh/install-ubuntu-focal-td-agent4.sh | sh
    

### Step 2: Launch Daemon

    Use /lib/systemd/system/td-agent script to start, stop, or restart the agent:
    
    
    $ sudo systemctl start td-agent.service
    $ sudo systemctl status td-agent.service
    
    
    ● td-agent.service - td-agent: Fluentd based data collector for Treasure Data
       Loaded: loaded (/lib/systemd/system/td-agent.service; disabled; vendor preset: enabled)
       Active: active (running) since Thu 2017-12-07 15:12:27 PST; 6min ago
         Docs: https://docs.treasuredata.com/articles/td-agent
      Process: 53192 ExecStart = /opt/td-agent/embedded/bin/fluentd --log /var/log/td-agent/td-agent.log --daemon /var/run/td-agent/td-agent.pid (code = exited, statu
     Main PID: 53198 (fluentd)
       CGroup: /system.slice/td-agent.service
               ├─53198 /opt/td-agent/embedded/bin/ruby /opt/td-agent/embedded/bin/fluentd --log /var/log/td-agent/td-agent.log --daemon /var/run/td-agent/td-agent
               └─53203 /opt/td-agent/embedded/bin/ruby -Eascii-8bit:ascii-8bit /opt/td-agent/embedded/bin/fluentd --log /var/log/td-agent/td-agent.log --daemon /v

    Dec 07 15:12:27 ubuntu systemd[1]: Starting td-agent: Fluentd based data collector for Treasure Data...
    Dec 07 15:12:27 ubuntu systemd[1]: Started td-agent: Fluentd based data collector for Treasure Data.
     
     
    * System has not been booted with systemd as init system (PID 1). Can't operate. Failed to connect to bus: Host is down
    
    sudo apt-get update && sudo apt-get install -yqq daemonize dbus-user-session fontconfig

    sudo daemonize /usr/bin/unshare --fork --pid --mount-proc /lib/systemd/systemd --system-unit=basic.target

    exec sudo nsenter -t $(pidof systemd) -a su - $LOGNAME
    
    
### 만약 -> Warning! D-Bus connection terminated.

        sudo apt install systemd-sysv
     



### Step 3: Post Sample Logs via HTTP(설치 확인)

    The default configuration (/etc/td-agent/td-agent.conf) is to receive logs at an HTTP endpoint and route them to stdout. For td-agent logs, see /var/log/td-agent/td-agent.log.

    $ curl -X POST -d 'json={"json":"message"}' http://localhost:8888/debug.test
    $ tail -n 1 /var/log/td-agent/td-agent.log
    2018-01-01 17:51:47 -0700 debug.test: {"json":"message"}
    
    [출처: https://docs.fluentd.org/installation/install-by-deb]
    
### Docker Image 설치

    $ docker pull fluent/fluentd:edge-debian
    
    # $(pwd)/tmp/fluentd.conf

    <source>
      @type http
      port 9880
      bind 0.0.0.0
    </source>

    <match **>
      @type stdout
    </match>


    $ docker run -p 9880:9880 -v $(pwd)/tmp:/fluentd/etc fluent/fluentd:edge-debian -c /fluentd/etc/fluentd.conf
    
    
    설치 확인
    
    $ curl -X POST -d 'json={"json":"message"}' http://127.0.0.1:9880/sample.test
    
    
    $ docker ps -a


    $ docker logs <docker ID> | tail -n 1
    
    [출처: https://docs.fluentd.org/container-deployment/install-by-docker]
    

 ### Docker Compose 설치
 
 
    https://github.com/digikin/fluentd-elastic-kibana
    
 
    [참조: https://docs.fluentd.org/container-deployment/docker-compose]
    
    
    
