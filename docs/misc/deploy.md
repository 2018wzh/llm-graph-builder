# iCareer 部署指南 by 2018wzh
## Java类程序（dwsurvey;icareer-app/authserver/gateway/registry-center）
### 部署
- 新建data,bin目录
- 将对应的jar/war包放在bin目录下
- 将配置文件放在data目录下
- 创建docker-compose.yml，写入以下内容，并将<name>和<extra-args>替换成部署程序的名称
```yaml
version: '3'
services:
  <name>:
    image: openjdk:8-jdk-alpine
    container_name: <name>
    volumes:
      - /usr/share/fonts:/usr/share/fonts
      - ./bin/<name>.jar:/app/app.jar
      - ./data:/data
    environment:
      - TZ=Asia/Shanghai
    network_mode: host
    restart: always
    working_dir: /data
    command: java -Xmx2048M -Xms250M -jar /app/app.jar --spring.profiles.active=uat
```
- 在项目根目录下运行docker compose up -d启动
### 更新
- 将更新后的包替换到bin目录，并在backup文件夹下保留一份副本
- 在项目根目录下运行docker compose restart
## minio
### 部署/更新
- 新建data目录，将相关文件放入目录下
- 创建docker-compose.yml
```yaml
version: '3'
services:
  minio:
    image: minio/minio
    container_name: minio
    volumes:
      - ./data:/data
    environment:
      - MINIO_ROOT_USER=minioadmin
      - MINIO_ROOT_PASSWORD=minioadmin
      - TZ=Asia/Shanghai
    network_mode: host
    restart: always
    command: server /data
```
- 执行docker compose up -d
## NGINX
### 部署/更新
- 新建data目录，将相关文件放入目录下
- 创建docker-compose.yml
```yaml
version: '3'
services:
  nginx:
    restart: always
    container_name: nginx
    image: nginx
    network_mode: host
    volumes:
      - ./data/conf/nginx.conf:/etc/nginx/nginx.conf
      - ./data/html:/usr/share/nginx/html
      - ./data/log:/var/log/nginx
    environment:
      - NGINX_PORT=80
      - TZ=Asia/Shanghai
    privileged: true
```
- 执行docker compose up -d
## RabbitMQ
- 新建data目录，将相关文件放入目录下
```yaml
version: '3'
services:
  rabbitmq:
    image: rabbitmq:customer
    container_name: rabbitmq
    volumes:
      - ./data:/var/lib/rabbitmq
    network_mode: host
    restart: unless-stopped
```
- 执行docker compose up -d
## MySQL
- 新建data目录，将相关文件放入目录下
```yaml
version: '3'
services:
  mysql:
    image: mysql:latest
    container_name: mysql
    volumes:
      - ./data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=pl,okm123
    ports:
      - "3306:3306"
    restart: unless-stopped
    privileged: true
```
- 执行docker compose up -d
## resume-parser
### 部署
- 将代码文件复制到build目录下
- 新建docker-compose.yml
```yaml
version: '3'
services:
  parser:
    container_name: resume-parser
    volumes:
      - /usr/share/fonts:/usr/share/fonts
    environment:
      - TZ=Asia/Shanghai
    build:
      context: ./build
      dockerfile: Dockerfile
    network_mode: host
    restart: always
```
### 更新
- 将代码复制到build目录下并备份
- 运行docker compose build然后docker compose restart
## icareer-neo4j
### 部署
- 将代码文件复制到build目录下
- 新建docker-compose.yml
```yaml
version: "3"

services:
  backend-cv:
    container_name: lgb-backend-cv
    build:
      context: ./build/backend
      dockerfile: Dockerfile
    env_file:
      - ./data/cv.env
    volumes:
      - ./data/nltk_data:/root/nltk_data
      - ./data/backend-cv/chunks:/code/backend/chunks
      - ./data/backend-cv/merged_files:/code/backend/merged_files
    ports:
      - "8000:8000"
    networks:
      - net
    depends_on:
      - neo4j-cv
  backend-jd:
    container_name: lgb-backend-jd
    build:
      context: ./build/backend
      dockerfile: Dockerfile
    volumes:
      - ./data/nltk_data:/root/nltk_data
      - ./data/backend-jd/chunks:/code/backend/chunks
      - ./data/backend-jd/merged_files:/code/backend/merged_files
    env_file:
      - ./data/jd.env
    ports:
      - "8001:8000"
    networks:
      - net
    depends_on:
      - neo4j-jd
  neo4j-cv:
    image: neo4j:2025.03.0
    container_name: neo4j-cv
    volumes:
        - ./data/neo4j-cv/logs:/logs
        - ./data/neo4j-cv/config:/config
        - ./data/neo4j-cv/data:/data
    environment:
        - NEO4J_AUTH=neo4j/icareer-cv
        - NEO4J_apoc_export_file_enabled=true
        - NEO4J_apoc_import_file_enabled=true
        - NEO4J_apoc_import_file_use__neo4j__config=true
        - NEO4J_PLUGINS=["apoc","graph-data-science"]
        - TZ=Asia/Shanghai
    restart: always
    networks:
        - net
  neo4j-jd:
    container_name: neo4j-jd
    image: neo4j:2025.03.0
    volumes:
        - ./data/neo4j-jd/logs:/logs
        - ./data/neo4j-jd/config:/config
        - ./data/neo4j-jd/data:/data
    environment:
        - NEO4J_AUTH=neo4j/icareer-jd
        - NEO4J_apoc_export_file_enabled=true
        - NEO4J_apoc_import_file_enabled=true
        - NEO4J_apoc_import_file_use__neo4j__config=true
        - NEO4J_PLUGINS=["apoc","graph-data-science"]
        - TZ=Asia/Shanghai
    restart: always
    networks:
        - net
  frontend-cv:
    container_name: lgb-frontend-cv
    build:
      context: ./build/frontend
      dockerfile: Dockerfile
    env_file:
      - ./data/frontend-cv.env
    ports:
      - "8080:8080"
    networks:
      - net
  frontend-jd:
    container_name: lgb-frontend-jd
    build:
      context: ./build/frontend
      dockerfile: Dockerfile
    env_file:
      - ./data/frontend-jd.env
    ports:
      - "8081:8080"
    networks:
      - net
networks:
  net:
```
### 更新
- 将代码复制到build目录下并备份
- 运行docker compose build然后docker compose restart
## Changelog
* 2025/5/16
  - 添加了完整部署过程
  - 简化了java程序的部署
* 2025/5/15
  - 添加了icareer-app/authserver/gateway/registry-center和dwsurvey的docker-compose.yml，替换了原有的docker命令行方式，使升级更简便
  - 将mysql/rabbitmq/nginx/minio的相关文件从匿名volume分离出来，方便备份与迁移
  - 将Applications文件夹迁移到sdb