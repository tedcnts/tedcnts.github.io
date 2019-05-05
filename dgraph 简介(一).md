# Dgraph

Fast, Transactional, Distributed Graph Database.
Dgraph是分布式，支持事务，快速的图形数据库。

Dgraph的目标是提供谷歌生产水平的规模和吞吐量，具有足够低的延迟，可以提供超过数TB的结构化数据的实时用户查询。DGraph组件支持GraphQL的查询语法，以及响应JSON和协议缓冲区超过GRPC和HTTP。

## 社区状态
截止2019/5/1，Dgraph版本是1.X。已经有500多公司使用其产品。
github： https://github.com/dgraph-io/dgraph

## 选择
本人目前已知图形化数据库Neo4j。但是目前好像Neo4J只是支持单机版，分布式版需要企业级，且闭源。

官方对比url：https://docs.dgraph.io/dgraph-compared-to-other-databases/
## 安装
本人使用docker安装，其他的安装方式请查看官网。

### docker

#### image： 
``` bash
docker pull dgraph/dgraph:latest
```
因为Dpragh是天生支持分布书，所以用docker-compose方式管理容器。
#### docker-compose.yml
``` bash
version: "3.2"
services:
  # Dgraph Zero控制Dgraph集群，将服务器分配给组并在服务器组之间重新平衡数据。
  zero:
    image: dgraph/dgraph:latest
    volumes:
      - type: volume
        source: dgraph
        target: /dgraph
        volume:
          nocopy: true
    ports:
      - 5080:5080
      - 6080:6080
    restart: on-failure
    command: dgraph zero --my=zero:5080

  # Dgraph Alpha托管谓词和索引。
  server:
    image: dgraph/dgraph:latest
    volumes:
      - type: volume
        source: dgraph
        target: /dgraph
        volume:
          nocopy: true
    ports:
      - 8080:8080
      - 9080:9080
    restart: on-failure
    command: dgraph alpha --my=server:7080 --lru_mb=2048 --zero=zero:5080
    
  # Dgraph Ratel为UI提供运行查询，突变和更改架构的功能。
  ratel:
    image: dgraph/dgraph:latest
    volumes:
      - type: volume
        source: dgraph
        target: /dgraph
        volume:
          nocopy: true
    ports:
      - 8000:8000
    command: dgraph-ratel
# 数据卷 
volumes:
  dgraph:

```
#### 启动&关闭
``` bash
cd ${docker-compose.yml 目录}
docker-compose up -d // 启动
docker-compose down // 关闭
```
tips：

访问 http://localhost:8000 查看图形操作话界面
