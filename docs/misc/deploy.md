# 部署与更新指南 by 2018wzh
## icareer-app/authserver/gateway/registry-center，dwsurvey
* 将更新的jar/war放在backup/{日期}目录下备份
* 将更新的jar/war放在bin目录下替换现有的文件
* 若要修改配置文件，在data下修改
* 进行更新或修改文件后在对应目录下执行docker compose restart
* 查看日志或排错执行docker compose logs
## mysql/minio/nginx/rabbitmq
* 若要更新岛最新版本则直接执行docker compose pull后进行docker compose up -d
* 数据备份：备份对应目录下的data目录
## resume-parser/icareer-neo4j
* 将更新后的源码放在build目录下，然后运行docker compose up -d
## Changelog
* 2025/5/15
  - 添加了icareer-app/authserver/gateway/registry-center和dwsurvey的docker-compose.yml，替换了原有的docker命令行方式，使升级更简便
  - 将mysql/rabbitmq/nginx/minio的相关文件从匿名volume分离出来，方便备份与迁移
  - 将Applications文件夹迁移到sdb
