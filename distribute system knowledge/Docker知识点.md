Portainer.io作为docker容器管理

[01 尚硅谷 Docker教程 前提知识要求和课程简介](https://www.youtube.com/watch?v=37b3cWIIxUg&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33)

[02 尚硅谷 Docker 为什么会出现](https://www.youtube.com/watch?v=e9PDl3Kk-nU&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=2)
```
主要解决开发和运维中间的“在我的机器上好使”的问题，保证环境的一致性。
代码/配置/系统/数据全部打包为一个整体。
```

[03 尚硅谷 Docker 理念](https://www.youtube.com/watch?v=jKKzkwyUq-8&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=3)
```
一次封装，到处使用，和集装箱的突破性理念是一样的。
```

[04 尚硅谷 Docker 是什么](https://www.youtube.com/watch?v=OIoVwmHNfr8&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=4)


[05 尚硅谷 Docker 能干什么](https://www.youtube.com/watch?v=T8iPSVjQpks&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=5)
```
虚拟机 vs Docker
虚拟机也会模拟硬件，Docker只保留软件工作所需要的资源和设置，容器没有自己的内核，就是浓缩的linux。
促使了Devops职业的诞生，然后就没纯运维的事了。
CentOS本身大概4G，但是Docker基础镜像只有170M，很轻便。
```

[06 尚硅谷 Docker 三要素](https://www.youtube.com/watch?v=8hCQ-sxfBIA&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=6)
```
安装docker是有一系列要求的。比如CentOS和Windows上安装都有具体的内核或者版本要求。
Docker Hub和Github的概念很像，Client基本命令:
docker build
docker pull
docker run

三要素：
1.镜像image：就是模板。
2.容器containter：就是模板的一个实例，可以启动，停止，删除。
3.仓库repository：就是存放镜像的地方。分为公开库（hub.docker.com）和私有库。
```

7-9 跳过

[10 尚硅谷 Docker helloworld镜像](https://www.youtube.com/watch?v=cFlszZTbQ_4&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=10)
```
$ docker run hello-world
先去本地Docker domain找container，没有就找镜像，再没有就去远程repository去pull。
注意这个命令是全局的，在哪运行都可以。

因为第一次跑，所以会自动下载到本地的docker客户端（我用win10）
```

[11 尚硅谷 Docker 运行底层原理](https://www.youtube.com/watch?v=BW9471j-2qM&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=11)
```
docker是CS结构，docker守护进程运行在主机上，客户端通过守护进程管理各个容器。

虚拟机：Infrastructure -> Operating System -> Hypervisor(实现引荐资源虚拟化） -> CentOS -> Bins/Libs各种依赖库 -> App
Docker： Infrastructure -> Operating System -> Docker System -> App

Docker没有Hypervisor层和CentOS层面和Lib层，利用宿主机的内核(操作系统)，使用实际物理机硬件资源，有明显优势。
```

[12 尚硅谷 Docker 帮助命令](https://www.youtube.com/watch?v=DdJDKAqW1YQ&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=12)
```
# 看版本
$ docker version
# 看详细docker信息
$ docker info
# 帮助
$ docker --help
```

[13 尚硅谷 Docker 镜像命令](https://www.youtube.com/watch?v=jFgJCKDlnAs&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=13)
```
# 看本地所有images
$ docker images
$ docker images -a
$ docker images -q

# 到docker hub上搜索tomcat的image
$ docker search tomcat
# 显示点赞数/stars超过30的
$ docker search -s 30 tomcat

# 下载tomcat镜像image
$ docker pull tomcat
# 注意默认是下载latest(docker pull tomcat:latest)，特定版本请修改如下
$ docker pull tomcat:8.0

# 删除镜像hello-world
$ docker rmi hello-world
# 注意如果有多个版本也需要加上版本号，默认是tomcat:latest
$ docker pull tomcat:8.0
# 强制删除，即使in use
$ docker rmi -f hello-world
# 删除多个，这里是同时删除hell-world,nginx的例子
$ docker rmi -f hello-workd nginx
```

[14 尚硅谷 Docker 容器命令上](https://www.youtube.com/watch?v=zw7uGug9sDw&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=14)
```
# 使用centos做例子
$ docker pull centos
# 生成实例，可以使用--name来设置别名
# -i 表示用"交互"模式运行容易，-t 表示为容器分配一个伪输入中断，通常两者配合使用
$ docker run -it --name mycentos2021 centos
# 这样就启动并且自动进入到了docker容器的centos镜像

# 退出交互中断（容器停止退出），Ctrl + P + Q容器不停止退出 
[root@7653941ada2f /]# exit

# 看所有container
$ docker ps
# 看历史上运行情况
$ docker ps -a
# 就看两个
$ docker ps -n 2

# 重启已经创建的容器
$ docker restart 7653941ada2f
# 停止容器
$ docker stop 7653941ada2f
# 强制停止容器
$ docker kill 7653941ada2f
# 删除已经停止的容器
$ dokcer rm 7653941ada2f
# 批量删除（类似liunx命令）
$ docker rm -f(docker ps -a -q)
$ docker ps -a -q | xargs docker rm
```

[15 尚硅谷 Docker 容器命令下](https://www.youtube.com/watch?v=OaP9uYf_QBw&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=15)
```
# 后台运行mini版本centos的docker
$ docker run -d centos
# 注意这个不会实际启动，因为centos需要交互，不进行交互直接就关闭了
# 所以可以用下面这种方式让centos一直运行
$ docker run -d centos /bin/sh -c "while true;do echo hello zzyy; sleep 2;done"
# 查看日志
$ docker logs -f -t --tail 7653941ada2f
# 查看某个container内部运行的进程
$ docker top 7653941ada2f
# 显示某个container内部的具体细节
$ docker inspect 7653941ada2f

# 演示退出交互后继续进入容器，先启动centos
$ docker run -it centos /bin/bash
# 就是在centos的容器7653941ada2f内运行 "ls -l /tmp" 命令
$ docker exec -t 7653941ada2f ls -l /tmp
# 重新进入终端了
$ docker exec -t 7653941ada2f /bin/bash

# 从容器内拷贝文件到主机上
# 注意容器停止了内部的文件就没了，所以可能会需要拷贝出来
# 这是将容器中/tmp/yum.log拷贝到本机的/root下面
$ docker cp 7653941ada2f:/tmp/yum.log /root
```

[16 尚硅谷 Docker 镜像原理](https://www.youtube.com/watch?v=B8c1ui1hDTw&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=16)
```
docker镜像是一种轻量级，可执行的独立软件包，打包了软件的运行环境和基于运行环境的软件。

UnionFS（联合文件系统）：是一种分层，轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加。Union文件系统是Docker镜像的基础。

必要组成（TODO 不是很理解）：
bootfs(boot file system)主要包括了bootloader和kernel，bootloader用于引导加载kernel，linux刚启动时候会加载bootfs文件系统，在Docker镜像的最底层是bootfs。当boot加载完成之后整个内核都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。
rootfs（root file system），在bootfs之上。包含就是典型的linux系统中的/dev,/proc,/bin,/etc等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。

一个centos的镜像只有200M，但是Tomcat却需要近500M，这是什么原理？
因为Tomcat一个容器需要能独立运行，所以需要依赖。对于tomcat来说，它的container大致是Kernel -> 叠加CentOS -> 叠加JDK8.0 -> 叠加Tomcat，因为堆了这么多文件，所以tomcat镜像就大了。

用UnionFS的好处是每一层的文件系统可以被共享。
```

[17 尚硅谷 Docker 镜像commit](https://www.youtube.com/watch?v=ddk5jb85G-M&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=17)
```
一个Tomcat的例子：
1.从Hub下载Tomcat到本地
$ docker pull tomcat

2.启动docker tomcat
# （-p表示docker container内的端口8080暴露到外部的8888端口）
$ docker run -it -p 8888:8080 tomcat 
# （大P表示随机端口）
$ docker run -it -P tomcat 

3.故意删除tomcat中的文档
$ docker exec -it 94682dg34s /bin/bash
# 进入webapp目录
$ rm -rf docs

4.创建自己的镜像
# 生成atguigu/mytomcat 1.2版本镜像文件，提交到docker hub
$ docker commit -a="author" -m="commit message" 94682dg34s atguigu/mytomcat:1.2

# 启动自己的镜像
$ docker run -it -p 7777:8080 atguigu/mytomcat:1.2
# 浏览器打开本地7777端口 localhost:7777/docs/ 已经被删除
```

[18 尚硅谷 Docker 容器数据卷介绍](https://www.youtube.com/watch?v=TLl1EXQzM58&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=18)
```
主要是持久化的考虑。因为docker的container一旦删除，里面的数据就没了。
容器数据卷可以进行容器和容器，容器到主机，和主机到容器之间的数据共享。
```

[19 尚硅谷 Docker 容器数据卷用V命令添加](https://www.youtube.com/watch?v=27WmFTlpcnE&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=19)
```
1.命令直接
# 主机建立hostData文件夹
# 连接本地的hostData文件夹和docker文件夹脸上
$ docker run -it -v /hostData:/dockerData centos
# 可以在inspect里面看到valume被关上，有读写权限
$ docker inspect

主机上：
$ cd hostData
$ touch host.txt
在容器内:
$ docker run -it centos /bin/bash
# 在容器内看到host.txt
[root@7653941ada2f /]# vi host.txt
# 同理，在容器内增加文件，主机也会同步显示

# 重启docker（退出容器）
$ docker start f8935dfgs
$ docker attach f8935dfgs
# docker的数据还是保留的，这就是mount是一样的

# 写保护挂在valume
$ docker run -it -v /hostData:/dockerData：ro centos
# 容器无法修改host.txt了，但是主机的修改没有问题
```

[20 尚硅谷 Docker 容器数据卷用DockerFile添加](https://www.youtube.com/watch?v=NcRkN7kHkTE&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=20)
```
看DockerFile就可以看到这个包是怎么集成的。
可以在hub.docker.com上找包的dockerFile

# 创建dockerFile
$ vi Dockerfile
# 写入内容
FROM centos
VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
CMD echo "finished, success1"
CMD /bin/bash
# 相当于
docker run -it -v /host1:/dataVolumeContainer1 -v /host2:/dataVolumeContainer2 centos /bin/bash

# 创建docker镜像
# 获得一个镜像 zzyy/centos
$ docker build -f /mydocker/dockerfile2 -t zzyy/centos .
$ docker images zzyy/centos
$ run -it zzyy/centos 
# 看到容器卷dataVolumeContainer1和dataVolumeContainer2已经创建在容器内
[root@7653941ada2f /]# ll
# 创建一个临时文件tmp.txt观察和主机的交互
[root@7653941ada2f /]# touch tmp.txt

$ docker ps
$ docker inspect 7653941ada2f
# 看到会在宿主机上生成一个默认的文件目录
```

[尚硅谷 Docker 容器数据卷volumes from](https://www.youtube.com/watch?v=LszNIOWV4KQ&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=21)
```
# 启动刚才自己做的那个zzyy/centos镜像，叫它dc01
$ docker run -it --name dc01 zzyy/centos
# 通过valume继承的方式依次建立dc02, dc03
$ docker run -it --name dc02 --volumes-from dc01 zzyy/centos
$ docker run -it --name dc03 --volumes-from dc01 zzyy/centos
# 发现这个容器卷可是在3个docker container内都是共享使用的

# 此时删除dc01父镜像
$ docker rm -f dc01
# 发现dc02和dc03完成不受影响，仍然volume共享
$ docker attach dc02
[root@7653941ada2f /] ll
```

[22 尚硅谷 Docker Dockerfile是什么](https://www.youtube.com/watch?v=3rF2eOV8sYU&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=22)
```
就是docker镜像的构建文件
```

[23 尚硅谷 Docker DockerFile构建过程解析](https://www.youtube.com/watch?v=O134IztsiMc&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=23)
```
大致构建过程：
就是根据dockerfile里头写的呗，注意必须有from一个基础镜像
```

[24 尚硅谷 Docker DockerFile保留字指令](https://www.youtube.com/watch?v=G1Dr1081L9c&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=24)
```
# dockerfile基础指令介绍
FROM 基础镜像来源
MAINTAINER 镜像维护者的姓名邮箱
RUN 容器构建是需要运行的命令
EXPOSE 暴露端口号
WORKDIR 指定登录container以后的落脚点，默认是/根目录
ENV 设置环境变量
ADD 拷贝 + 解压，比如ADD centos-7-docker.tar，将宿主机目录下文件拷贝到镜像
COPY 拷贝
VOLUME 就是挂硬盘卷
CMD 指定一个容器启动时要运行的命令，如果有多个只有最后一个生效，所以加上/bin/bash会覆盖它
ENTRYPOINT 和CMD一样，不会覆盖，是追加
ONBUILD 继承dockerfile时运行，父镜像被子继承后父镜像onbuild会触发
```

[25 尚硅谷 Docker DockerFile案例 自定义镜像mycentos](https://www.youtube.com/watch?v=xifL_E8cS8s&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=25)
```
默认的centos镜像只有内核，很精简不包含vim或者ifconfig指令，这里的例子就是进行重构一个包含这两项指令的image

# 详细dockerfile指令，假设这个文件path在/mydocker/Dockerfile2
FROM centos

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "success ------------ ok"
CMD /bin/bash

# 进行编译
$ docker build -f /mydocker/Dockerfile2 -t mycentos:1.3 .
# 查看本地镜像
$ docker images mycentos
# 运行该镜像
$ docker run -it mycentos:1.3
# 发现container进行以后就包含了vim和ifconfig了
# 此外可以用history看镜像编译的详细流程
$ docker history 5235df423df
```

[26 尚硅谷 Docker DockerFile案例 CMD ENTRYPOINT命令案例](https://www.youtube.com/watch?v=BYuxlnEo9Mc&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=26)
```
# 让docker能够进行curl，演示ENTRYPOINT
# /mydocker/Dockerfile3 
FROM centos
RUN yum install -y curl
ENTRYPOINT ["curl", "-s", "http://ip.cn"]

$ docker build -f /mydocker/Dockerfile3 -t myip .
$ docker run -it myip
# 显示 当前IP:218.22.22.33，也就是curl ip.cn返回的结果

# 显示curl头文件，需要-i，类似 curl -s http://ip.cn -i
$ docker run -it myip -i
```

[27 尚硅谷 Docker DockerFile案例 ONBUILD命令案例](https://www.youtube.com/watch?v=H6XHVFdVeeU&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=27)
```
# 演示ONBUILD的使用，就是
# /mydocker/Dockerfile4 
FROM centos
RUN yum install -y curl
ENTRYPOINT ["curl", "-s", "http://ip.cn"]
ONBUILD RUN echo "Father image is triggered by ONBUILD"
# 创建myip_father这个image
$ docker build -f /mydocker/Dockerfile4 -t myip_father .

# /mydocker/Dockerfile5，注意继承父类
FROM myip_father
RUN yum install -y curl
ENTRYPOINT ["curl", "-s", "http://ip.cn"]
# 创建myip_son这个image
$ docker build -f /mydocker/Dockerfile5 -t myip_son .
# 注意创建的时候就出发了父类的触发器
```

[28 尚硅谷 Docker DockerFile案例 自定义的tomcat9](https://www.youtube.com/watch?v=ATgPQervmQw&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=28)
```
# 演示ADD和COPY的使用
# 先准备好3个文件：apache-tomcat-9.0.8.tar.gz, c.txt, jdk-8u171-linux-x64.tar.gz
# /mydockerfile/tomcat9 内容

FROM centos 
MAINTAINER zzyy<xxx@gmail.com>
# 把宿主机当前上下文的c.txt拷贝到容器/usr/local/路径下
COPY c.txt /usr/local/cincontainer.txt
# 把java与tomcat添加到容器中
ADD jdk-8u171-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.8.tar.gz /usr/local/
# 安装vim编辑器
RUN yum -y install vim
# 设置工作访问时候的WORKDIR路径，登录落脚点
ENV MYPATH /usr/local
WORKDIR $MYPATH
# 配置java与tomcat环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_171
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.8
ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.8
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
# 容器运行时监听的端口
EXPOSE 8080
# 启动时候运行tomcat
# ENTRYPOINT ["/usr/local/apache-tomcat-9.0.8/bin/startup.sh"]
# CMD ["/usr/local/apache-tomcat-9.0.8/bin/catalina.sh","run"]
CMD /usr/local/apache-tomcat-9.0.8/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.8/bin/logs.catalina.out 

# 创建镜像
$ docker build -t zzyytomcat9 .
# 多多熟悉下列的命令
$ docker run -d -p 9080:8080 --name myt9 -v /mydockerfile/tomcat9/test:/usr/local/apache-tomcat-9.0.8/webapps/test -v /mydockerfile/tomcat9/tomcat9logs/:/usr/local/apache-tomcat-9.0.8/logs --privileged=true tomcat9
```

[29 尚硅谷 Docker DockerFile案例 自定义的tomcat9上发布演示](https://www.youtube.com/watch?v=2WnoW3dJijg&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=29)
```
自己在tomcat9的容器内加个页面，就正常使用了
```

[30 尚硅谷 Docker DockerFile小总结](https://www.youtube.com/watch?v=EX4htYS9T14&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=30)
```
就是复习下
```

[31 尚硅谷 Docker 安装mysql](https://www.youtube.com/watch?v=5VQ7bTyznmU&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=31)
```
# 找mysql5.6最多的镜像
$ docker search mysql:5.6
$ docker pull mysql:5.6
# -d 是后台运行的意思， -e 是配置环境
$ docker run -p 12345:3306 --name mysql -v zzyy/mysql/conf:/etc/mysql/conf.d -v zzyy/mysql/logs:logs -v zzyy/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
$ docker ps
$ docker exec -it s32dg4234 /bin/bash
# 登录到mysql
[root@s32dg4234]# ps -ef
[root@s32dg4234]# mysql -uroot -p
# 进入mysql并创建库和表
mysql > show databases
mysql > create database db01;
mysql > use db01;
mysql > create table t_book(id int not null primary key, bookName varchar(20);
mysql > insert into t_book values(1, 'java');
mysql > select * from t_book;

# 将docker镜像内的mysql dump文件输出到宿主机的all-databases.sql里去
$ docker exec s32dg4234 sh -c ' exec mysqldump --all-databases -uroot -p"123456" ' > /zzyy/all-databases.sql
```

[32 尚硅谷 Docker 安装Redis](https://www.youtube.com/watch?v=IdM_2fshLog&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=32)
```
$ docker pull redis:3.2
# appendonly yes 是开始redis的aof备份的意思
$ docker run -p 6379:6379 /zzyy/myredis/data:data -v /zzyy/myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf -d redis:3.2 redis-server /usr/local/etc/redis/redis.conf --appendonly yes
# 增加redis的配置文件
[root@s32dg4234]# vim /zzyy/myredis/conf/redis.conf/redis.conf
[root@s32dg4234]# cd /zzyy/myredis/conf/redis
# 登录redis使用
$ docker exec -it s32dg4234 redis-cli
127.0.0.1 > set k1 v1
OK
127.0.0.1 > set k2 v2
OK
127.0.0.1 > SHUTDOWN
```

[33 尚硅谷 Docker 本地镜像推送到阿里云](https://www.youtube.com/watch?v=d-alo1DXgbo&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=33)
```
docker的registry也有共有云(docker hub)和私有云的区别

$ docker run -it mycentos:1.3
# 使用1.3的container生成了1.4版本
$ docker commt -a zzyy -m "new mycentos with vim and ifconfig" df25sdg53 mycentos:1.4

# 推送到阿里云的例子
$ sudo docker login --username= registry.cn-hangzhou.aliyuncs.com
$ sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/zzyy/mycentos:[镜像版本号]
$ sudo docker push registry.cn-hangzhou.aliyuncs.com/zzyy/mycentos:[镜像版本号]
```

[34 尚硅谷 Docker CentOS7安装Docker（补充知识）](https://www.youtube.com/watch?v=FKRK-3hC148&list=PLmOn9nNkQxJFX0YVLDw5EMUL-4cVzXL33&index=34)
```
跳过
```
