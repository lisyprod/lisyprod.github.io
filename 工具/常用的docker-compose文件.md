## docker-compose相关

## 一、docker-compose相关

- [docker教程](https://yeasy.gitbook.io/docker_practice/)
- [docker-compose教程](https://yeasy.gitbook.io/docker_practice/compose)

常用命令

```
// 启动
docker-compose up -d
// 停止
docker-compose stop
// 停止并删除容器
docker-compose down
```

## 二、docker-compose 文件
运行jar文件

```
version: "3"
services:
  platform:
    # 指定容器名称
    container_name: platform
    # 重启机制
    restart: always
    # 镜像名称
    image: java:8
    # 指定工作空间（为了方便jar包直接用外部的配置文件）
    working_dir: /usr/local/myapp
    volumes:
      # 将当前目录映射到容器内部的/usr/local/myapp
      - ./:/usr/local/myapp
      # 指定容器时间为宿主机时间
      - /etc/localtime:/etc/localtime
    # 访问端口
    ports:
      - 18284:8284
      - 19876:19876
    environment:
      # 指定时区
      - TZ="Asia/Shanghai"
    #启动容器后执行的命令
    entrypoint: java -Xms1024m -Xmx1024m -jar app.jar

```
Dockerfile( docker build -t lisyjavaapp:v1 .)
```
FROM openjdk:11-jre
COPY app.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]

```

nginx

```
mkdir -p ./html ./conf ./conf.d ./logs


version: '3'
services:
  nginx:
    image: nginx
    container_name: nginx-main
    restart: always
    ports:
      - 80:80
    volumes:
    - ./html:/usr/share/nginx/html
    - ./logs:/var/log/nginx
    - ./conf.d:/etc/nginx/conf.d
    - ./conf/nginx.conf:/etc/nginx/nginx.conf



// conf文件里面新建nginx.conf
events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  300;
    server {
        listen       80;
        server_name  localhost;


        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }


        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}



```

sql server
```
mssql:
    image: 'mcr.microsoft.com/mssql/server:2019-latest'
    ports:
        - '1433:1433'
    environment:
        - ACCEPT_EULA=Y
        - SA_PASSWORD=12345678
    volumes:
       - ./data/data:/var/opt/mssql/data
       - ./data/logs:/var/opt/mssql/log
       - ./data/secrets:/var/opt/mssql/secrets

```



graylog

```
version: '3'
services:
  # MongoDB: https://hub.docker.com/_/mongo/
  mongodb:
    image: mongo:4.2
    volumes:
      - ./mongo_data:/data/db
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 512M
    command: /bin/bash -c "docker-entrypoint.sh mongod"
  # Elasticsearch: https://www.elastic.co/guide/en/elasticsearch/reference/7.10/docker.html
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch-oss:7.10.2
    volumes:
      - ./es_data:/usr/share/elasticsearch/data
    environment:
      - http.host=0.0.0.0
      - transport.host=localhost
      - network.host=0.0.0.0
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    command: /bin/bash -c "/usr/local/bin/docker-entrypoint.sh eswrapper"
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 1024M
  # Graylog: https://hub.docker.com/r/graylog/graylog/
  graylog:
    image: graylog/graylog:4.0
    volumes:
      - ./graylog_data:/usr/share/graylog/data
    environment:
      # CHANGE ME (must be at least 16 characters)!
      - GRAYLOG_PASSWORD_SECRET=somepasswordpepper
      # Password: admin
      - GRAYLOG_ROOT_PASSWORD_SHA2=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
      - GRAYLOG_HTTP_EXTERNAL_URI=http://192.168.6.175:9000/
    entrypoint: /usr/bin/tini -- wait-for-it elasticsearch:9200 --  /docker-entrypoint.sh
    links:
      - mongodb:mongo
      - elasticsearch
    restart: always
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 512M
    depends_on:
      - mongodb
      - elasticsearch
    ports:
      # Graylog web interface and REST API
      - 9000:9000
      # Syslog TCP
      - 1514:1514
      # Syslog UDP
      - 1514:1514/udp
      # GELF TCP
      - 12201:12201
      # GELF UDP
      - 12201:12201/udp

```

es 

```
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch-oss:7.10.2
    volumes:
      - /root/sky/es/es_data:/usr/share/elasticsearch/data
    command: /bin/bash -c "/usr/local/bin/docker-entrypoint.sh eswrapper"
    environment:
      - http.host=0.0.0.0
      - transport.host=localhost
      - network.host=0.0.0.0
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.type=single-node
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 512M
    restart: always
    ports:
     - 9200:9200
     - 9300:9300
```

elk:

```
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.10.2
    volumes:
      - ./es_data:/usr/share/elasticsearch/data
      - ./es_data/plugins:/usr/share/elasticsearch/plugins
    command: /bin/bash -c "/usr/local/bin/docker-entrypoint.sh eswrapper"
    environment:
      - http.host=0.0.0.0
      - transport.host=localhost
      - network.host=0.0.0.0
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.type=single-node
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 512M
    restart: always
    ports:
     - 9200:9200
     - 9300:9300
  kibana:
     image: docker.elastic.co/kibana/kibana:7.10.2
     container_name: kibana
     depends_on:
       - elasticsearch
     deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 512M
     restart: always
     ports:
       - 5601:5601
  logstash:
     image: logstash:7.10.1
     container_name: logstash
     deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 512M
     restart: always
     ports:
      - "5000:5000/tcp"
      - "5010:5010/tcp"
      - "5000:5000/udp"
      - "9600:9600"
     environment:
       discovery.type: single-node
       ES_JAVA_OPTS: "-Xmx256m -Xms256m"
     volumes:
       - ./logstash_data/conf:/usr/share/logstash/config
       - ./logstash_data/plugins:/tmp/plugins
       - /etc/localtime:/etc/localtime
       - ./logstash_data/logstash.yml:/usr/share/logstash/config/logstash.yml
     depends_on:
       - elasticsearch

```



portainer
```
version: '3'
services:
  portainer:
    image: portainer/portainer-ce:2.9.3
    #command: -H unix:///var/run/docker.sock
    restart: always
    ports:
      - 52000:9000
      - 8000:8000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./portainer_data:/data


```

gitea

```
version: '3'
services:
  web:
    image: gitea/gitea:1.14.2
    volumes:
      - ./data:/data
    environment:
      - DB_TYPE=mysql
      - DB_HOST=db:3306
      - DB_NAME=gitea
      - DB_USER=gitea
      - DB_PASSWD=wzrNewDB
      - HTTP_PORT=16989
    ports:
      - "16988:16989"
      - "16222:22"
    depends_on:
      - db
    restart: always
    deploy:
      resources:
        limits:
          cpus: '0.10'
          memory: 512M
  db:
    image: mariadb:10
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=wzrNewDB
      - MYSQL_DATABASE=gitea
      - MYSQL_USER=gitea
      - MYSQL_PASSWORD=wzrNewDBA
    volumes:
      - ./db/:/var/lib/mysql
    deploy:
      resources:
        limits:
          cpus: '0.10'
          memory: 512M

```

mysql8

```
version: '3.1'
services:
  db:
    container_name: mysql-master
    image: mysql:8.0.23
    hostname: server-1
    network_mode: host
    environment:
      MYSQL_ROOT_PASSWORD: 123456
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 512M
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
    ports:
      - 3306:3306
      - 33061:33061
    volumes:
      - ./data:/var/lib/mysql
      - ./config/my.cnf:/etc/mysql/conf.d/docker.cnf
      - /etc/hosts:/etc/hosts


```

redis

```
version: '3.1'
services:
    redis:
      image: redis:5.0.12
      container_name: redis
      command: redis-server --requirepass 123456
      ports:
        - "6379:6379"
      volumes:
        - ./data:/data
      restart: always

```


nacos

```
version: '3.1'
services:
  nacos-server:
    image: nacos/nacos-server:1.4.2
    container_name: nacos-server
    environment:
      - MODE=standalone
      - SPRING_DATASOURCE_PLATFORM=mysql
      - MYSQL_SERVICE_HOST=mysql8.0.23
      - MYSQL_SERVICE_PORT=3306
      - MYSQL_SERVICE_DB_NAME=nacos
      - MYSQL_SERVICE_USER=root
      - MYSQL_SERVICE_PASSWORD=123456
      - JVM_XMS=512m
      - JVM_XMX=512m
    volumes:
      - ./standalone-logs/:/home/nacos/logs
      - ./custom.properties:/home/nacos/init.d/custom.properties
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 512M
    ports:
      - "8848:8848"
      - "9848:9848"
      - "9555:9555"
    restart: on-failure
    networks:
      - default
      - mysql8_default
networks:
  mysql8_default:
    external: true

```

rocketmq

```
version: '3.5'
services:
  namesrv:
    image: rocketmqinc/rocketmq
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    volumes:
      - ./rocketmq/logs:/home/rocketmq/logs
      - ./rocketmq/store:/home/rocketmq/store
    command: sh mqnamesrv
    environment:
      - JAVA_OPTS=-Xmx512m -Xms512m -Xss256k
    restart: on-failure
    deploy:
      resources:
        limits:
          cpus: '0.10'
          memory: 512M
  broker:
    image: rocketmqinc/rocketmq
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
      - 10912:10912
    volumes:
      - ./rocketmq/logs:/home/rocketmq/logs
      - ./rocketmq/store:/home/rocketmq/store
      - ./rocketmq/conf/broker.conf:/opt/rocketmq-4.4.0/conf/broker.conf
    #command: sh mqbroker -n namesrv:9876
    command: sh mqbroker -n namesrv:9876 -c ../conf/broker.conf
    depends_on:
      - namesrv
    environment:
      - JAVA_HOME=/usr/lib/jvm/jre
      - JAVA_OPTS=-Xmx512m -Xms512m -Xss256k
    restart: on-failure
    deploy:
      resources:
        limits:
          cpus: '0.10'
          memory: 512M
  console:
    image: styletang/rocketmq-console-ng
    container_name: rocketmq-console-ng
    ports:
      - 8087:8080
    depends_on:
      - namesrv
    restart: on-failure
    deploy:
      resources:
        limits:
          cpus: '0.10'
          memory: 512M
    environment:
      - JAVA_OPTS= -Xmx512m -Xms512m -Xss256k -Dlogging.level.root=info -Drocketmq.namesrv.addr=rmqnamesrv:9876 
      - Dcom.rocketmq.sendMessageWithVIPChannel=false

```
broker.conf

```
brokerClusterName = DefaultCluster  
brokerName = broker-a  
brokerId = 0  
deleteWhen = 04  
fileReservedTime = 48  
brokerRole = ASYNC_MASTER  


flushDiskType = ASYNC_FLUSH  
# 如果是本地程序调用云主机 mq，这个需要设置成 云主机 IP,docker 的话，要写宿主的ip
brokerIP1=192.168.23.131 

```

postgresql

```
version: '3'
services:
  postgres:
    image: postgres:latest
    container_name: postgres_dc
    volumes:
      - ./pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: lisy #在此填写postgres的用户名
      POSTGRES_DB: qzptmis #在此填写postgres的数据库名，默认是postgres
      POSTGRES_PASSWORD: 123456 #在此填写posgres的数据库密码
    ports:
      - "5432:5432"
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: pgadmin_dc
    environment: 
      PGADMIN_DEFAULT_EMAIL: lisywork@qq.com #在此填写pgAdmin登录账户邮箱
      PGADMIN_DEFAULT_PASSWORD: 123456 #在此填写pgAdmin密码
    ports:
      - "5050:80"
volumes:
  pgdata:

```

emqx
```
version: '3.1'
services:
  emqx:
    image: emqx/emqx
    container_name: emqx
    restart: always
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 512M
    ports:
      - 18083:18083
      - 1883:1883
      - 4369:4369
```
minio
```
version: '3.1'
services:
  minio:
    #image: minio/minio:RELEASE.2020-09-08T23-05-18Z
    image: minio/minio:RELEASE.2022-01-27T03-53-02Z
    container_name: minio
    restart: always
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 512M
    volumes:
      - ./data:/data
      - ./config:/root/.minio
    #command: server /data --address ":9120"
    command: server /data --console-address ":9120" --address ":9000"
    environment:
      MINIO_ACCESS_KEY: minioadmin
      MINIO_SECRET_KEY: minioadmin
    ports:
      - 19120:9120
      - 19000:9000
```

prometheus

```
version: '3.1'
services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./config/hoststats-alert.rules:/etc/prometheus/rules/hoststats-alert.rules
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 1024M
    ports:
      - 19090:9090
  grafana:
    image: grafana/grafana
    container_name: grafana
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 1024M
    ports:
      - 3001:3000
  mysqld-exporter:
    image: prom/mysqld-exporter
    container_name: mysqld-exporter
    environment:
      DATA_SOURCE_NAME: "root:123456@(192.168.6.216:3306)/"
    ports:
      - 9104:9104
  redis-exporter:
    image: oliver006/redis_exporter
    container_name: redis-exporter
    environment:
       REDIS_ADDR: redis://192.168.6.216:6379
       REDIS_PASSWORD: 123456
    ports:
      - 9121:9121
  alertmanager:
    image: prom/alertmanager
    container_name: alertmanager
    volumes:
      - ./config/alertmanager.yml:/etc/prometheus/rules.yml
    command:
      - "--config.file=/etc/prometheus/rules.yml"
    ports:
      - 9093:9093






// config/prometheus.yml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['192.168.6.216:9093']

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - /etc/prometheus/rules/*.rules
  #- "server_other.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "lisymac"
    static_configs:
      - targets: ["192.168.6.216:9100"]
  - job_name: "mysqld"
    static_configs:
      - targets: ["192.168.6.216:9104"]
  - job_name: "redis"
    static_configs:
      - targets: ["192.168.6.216:9121"]
  - job_name: minio-job
    bearer_token: eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJleHAiOjQ3OTY4NzAyMDEsImlzcyI6InByb21ldGhldXMiLCJzdWIiOiJtaW5pb2FkbWluIn0.JSHo843-AMFrBREpeF0PgGMywrhQCUIbz_jryiHDytMWvOcybOub_ivT_KWPi0BeuVTH7PuYDANJ9na21eGPSw
    metrics_path: /minio/v2/metrics/cluster
    static_configs:
      - targets: ['192.168.6.216:9120']


        #- job_name: "nacos"
        #metrics_path: '/nacos/actuator/prometheus'
        #static_configs:
        #- targets: ["192.168.111.133:8848"]


//config/alertmanager.yml
global:
  # 告警超时时间
  resolve_timeout: 5m
  # 发送者邮箱地址
  smtp_from: 'test@163.com'
  # 邮箱smtp服务器地址及端口
  smtp_smarthost: 'smtp.163.com:465'
  # 发送者邮箱账号
  smtp_auth_username: 'test@163.com'
  # 发送者邮箱密码，这里填入第一步中获取的授权码
  smtp_auth_password: '123456'
  # 是否使用tls
  smtp_require_tls: false
  smtp_hello: '163.com'
# 路由配置,设置报警的分发策略，它是一个树状结构，按照深度优先从左向右的顺序进行匹配。
route:
  # 用于将传入警报分组在一起的标签。
  # 基于告警中包含的标签，如果满足group_by中定义标签名称，那么这些告警将会合并为一个通知发送给接收器。
  group_by: ['alertname']
  # 发送通知的初始等待时间
  group_wait: 30s
  # 在发送有关新警报的通知之前需要等待多长时间
  group_interval: 5m
  # 如果已发送通知，则在再次发送通知之前要等待多长时间，通常约3小时或更长时间
  repeat_interval: 30s
  # 接受者名称
  receiver: '163.email'
# 配置告警消息接受者信息，例如常用的 email、wechat、slack、webhook 等消息通知方式
receivers:
- name: '163.email'
  email_configs:
  # 配置接受邮箱地址
  - to : 'lisywork@qq.com'



//config/hoststats-alert.rules

groups:
- name: server_status # 组的名字，在这个文件中必须要唯一
  rules:
  - alert: redisDown # 告警的名字，在组中需要唯一
    expr: redis_up == 0 # 表达式, 执行结果为true: 表示需要告警
    for: 1m # 超过多少时间才认为需要告警(即up==0需要持续的时间)
    labels:
      severity: warning # 定义标签
    annotations:
      summary: "服务 {{ $labels.instance }} 监控的redis下线了"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes."
  - alert: mysqlDown # 告警的名字，在组中需要唯一
    expr: mysql_up == 0 # 表达式, 执行结果为true: 表示需要告警
    for: 1m # 超过多少时间才认为需要告警(即up==0需要持续的时间)
    labels:
      severity: warning # 定义标签
    annotations:
      summary: "服务 {{ $labels.instance }} 监控的mysql下线了"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes."


```
```
version: '3.1'
services:
  registry:
    restart: always
    image: registry:2
    ports:
      - 15000:5000
        #environment:
      #REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
      #REGISTRY_HTTP_TLS_KEY: /certs/domain.key
      #REGISTRY_AUTH: htpasswd
      #REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
      #REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
    volumes:
      - ./data:/var/lib/registry
        #- ./certs:/certs
        #- ./auth:/auth
  frontend:
    image: konradkleine/docker-registry-frontend:v2
    ports:
      - 8082:80
    # volumes:
    #   - ./certs/frontend.crt:/etc/apache2/server.crt:ro
    #   - ./certs/frontend.key:/etc/apache2/server.key:ro
    environment:
      - ENV_DOCKER_REGISTRY_HOST=192.168.111.133
      - ENV_DOCKER_REGISTRY_PORT=15000
  
```