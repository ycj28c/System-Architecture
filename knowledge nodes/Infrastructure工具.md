罗列了一些见到的Infrastructure的工具，了解一下作为知识储备

**GraphQL:**   
call api来快速查询树/图结构，提供了很好的图形api query接口（用json格式管理）

**Docker:**   
容器技术,kubernates

**Hashistack:**   
提供了大量的DevOps基础设施自动化工具，集开发、运营和安全性于一体。部署数据中心的。 HashiCorp的技术栈，包括了vagrant（本地开发环境），packer，terraform（云迁移），consul，vault(Secrets Management.存放credential），Nomad（Applications Management），consul（Service Discovery），Traefik（反向代理）等等

**Traefik:**   
是一个反向代理，reverse proxy，用来负载均衡之类，java语言我们用apache，nginx

**Hubot:**   
chat bot聊天机器人，slack有这个插件，我做自动化测试是直接用slack来发消息，编写一些hook来执行脚本（google spreadsheet远程运行jenkins等等，只要开发了api就可以任意继承）

**InfluxDB:**   
典型的时序数据库time series database。尤其擅长记录时间为标志的指标啊，使用LSM tree做的，可以类比cassandra，hbase。在DB-Engines Ranking时序型数据库排行榜上排名第一，广泛应用于DevOps监控、IoT监控、实时分析等场景。无依赖，提供query。

iaas这样的infrastructure即是服务是怎么搭建的，vmware和docker，类似ec2的存在。不在cloud工作过肯定不行啊。
