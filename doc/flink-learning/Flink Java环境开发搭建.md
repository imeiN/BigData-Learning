# 新版Flink Java环境开发快速搭建

## Maven创建FLINK项目(三种方式选一种即可)
1. 基于 Maven Archetype 构建，直接使用下面的 mvn 语句来进行构建
直接使用下面的 mvn 语句来进行构建，然后根据交互信息的提示，依次输入 groupId , artifactId 以及包名等信息后等待初始化的完成
mvn archetype:generate \
  -DarchetypeGroupId=org.apache.flink \
  -DarchetypeArtifactId=flink-quickstart-java \
  -DarchetypeVersion=1.10.2 \
  -DgroupId=org.myorg.quickstart \
  -DartifactId=quickstart	\
  -Dversion=0.1 \
  -Dpackage=org.myorg.quickstart \
  -DinteractiveMode=false

2. 使用官方脚本快速构建
为了更方便的初始化项目，官方提供了快速构建脚本，可以直接通过以下命令来进行调用：

curl https://flink.apache.org/q/quickstart.sh | bash -s 1.10.2

该方式其实也是通过执行 maven archetype 命令来进行初始化，相比于第一种方式，该种方式只是直接指定好了 groupId ，artifactId ，version 等信息而已。

把生成的项目文件导入IDEA即可

3. 使用 IDEA 构建
直接在项目创建页面选择 Maven Flink Archetype 进行项目初始化：


如果你的 IDEA 没有上述 Archetype， 可以通过点击右上角的 ADD ARCHETYPE ，来进行添加，依次填入所需信息，这些信息都可以从上述的 archetype:generate 语句中获取。
点击  OK 保存后，该 Archetype 就会一直存在于你的 IDEA 中，之后每次创建项目时，只需要直接选择该 Archetype 即可：

选中 Flink Archetype ，然后点击 NEXT 按钮，之后的所有步骤都和正常的 Maven 工程相同。


## IDEA运行环境设置(二种方式选一种即可)

1. 使用 IDEA 创建项目pom.xml  添加profile 配置后在 id 为 add-dependencies-for-IDEA 的 profile 中，
所有的核心依赖都被标识为 compile，此时你可以无需改动任何代码，只需要在 IDEA 的 Maven 面板中勾选该 profile后再Reload ALL MAVEN PROJECTS，
即可直接在 IDEA 中运行 Flink 项目,或者会导致IDEA中启动项目时会抛出 ClassNotFoundException 异常。
   
        <!-- This profile helps to make things run out of the box in IntelliJ -->
        <!-- Its adds Flink's core classes to the runtime class path. -->
        <!-- Otherwise they are missing in IntelliJ, because the dependency is 'provided' -->
        <profile>
            <id>add-dependencies-for-IDEA</id>

            <activation>
                <property>
                    <name>idea.version</name>
                </property>
            </activation>

            <dependencies>
                <dependency>
                    <groupId>org.apache.flink</groupId>
                    <artifactId>flink-scala_${scala.binary.version}</artifactId>
                    <scope>compile</scope>
                </dependency>
                <dependency>
                    <groupId>org.apache.flink</groupId>
                    <artifactId>flink-streaming-scala_${scala.binary.version}</artifactId>
                    <scope>compile</scope>
                </dependency>
                <dependency>
                    <groupId>org.scala-lang</groupId>
                    <artifactId>scala-library</artifactId>
                    <scope>compile</scope>
                </dependency>
            </dependencies>
        </profile>
    </profiles>


2. 在IntelliJ IDEA中做相关设置 Include dependencies with “Provided” scope
RUN-->RUN/DEBUG  --> "Run/Debug Configurations"


## 外部数据源环境

###安装Docker
见java_learning 项目

###安装Kafka
$ docker search kafka
NAME                                    DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
wurstmeister/kafka                      Multi-Broker Apache Kafka Image                 1259                                    [OK]
spotify/kafka                           A simple docker image with both Kafka and Zo…   403                                     [OK]
sheepkiller/kafka-manager               kafka-manager                                   196                                     [OK]
bitnami/kafka                           Apache Kafka is a distributed streaming plat…   181                                     [OK]
ches/kafka                              Apache Kafka. Tagged versions. JMX. Cluster-…   117                                     [OK]

$ docker pull wurstmeister/kafka:2.12-2.5.0

安装完成之后，我们可以查看一下已经安装的镜像
$ docker images

###zookeeper
$ docker search zookeeper
NAME                               DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
zookeeper                          Apache ZooKeeper is an open-source server wh…   956                 [OK]                
jplock/zookeeper                   Builds a docker image for Zookeeper version …   166                                     [OK]
wurstmeister/zookeeper                                                             132                                     [OK]
mesoscloud/zookeeper               ZooKeeper                                       73                                      [OK]
bitnami/zookeeper                  ZooKeeper is a centralized service for distr…   45                                      [OK]
mbabineau/zookeeper-exhibitor                                                      24                                      [OK]

$ docker pull zookeeper:3.6.1

安装完成之后，查看一下已经安装的镜像
$ docker images

在容器的启动 zookeeper
$ docker run -d --name zookeeper  -p 2181:2181 -t zookeeper:3.6.1

启动后用docker ps查看启动状态
$ docker ps

在容器的启动 kafka
$ docker run -d --name kafka --publish 9092:9092 --link zookeeper --env KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 --env KAFKA_ADVERTISED_HOST_NAME=127.0.0.1 --env KAFKA_ADVERTISED_PORT=9092 wurstmeister/kafka:2.12-2.5.0

启动后用kafka ps查看启动状态
$ docker ps

进入Kafka命令行
$ docker exec -it kafka /bin/bash

尝试创建一个Topic
/opt/kafka/bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic test

向test Topic 发数据：
opt/kafka/bin/kafka-console-producer.sh --topic=test --broker-list localhost:9092
>hello
>welcome you

启动另一个Kafka的Shell来消费消息，如下：
$ docker exec -it kafka /bin/bash
/opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 -from-beginning --topic test
hello
welcome you
至此，我们Kafka的环境部署测试完成。

###安装MySQL
先搜索，再安装
$ docker search mysql
NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   10176               [OK]                
mariadb                           MariaDB is a community-developed fork of MyS…   3749                [OK]                
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   743                                     [OK]


$ docker pull mysql:5.7

安装完成之后，查看一下已经安装的镜像
$ docker images

配置mysql
mkdir -p $PWD/conf $PWD/logs $PWD/data
docker run -p 3306:3306 --name flink_mysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7

简单的解释一下命令参数含义如下：
-p 3306:3306：将容器的 3306 端口映射到主机的 3306 端口。
-v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql: 本机目录和容器目录进行挂载，比如：PWD/logs:/logs：将主机当前目录下的 logs 目录挂载到容器的 /logs。
-e MYSQL_ROOT_PASSWORD=123456：初始化 root 用户的密码。
-d: 后台运行容器，并返回容器ID

查看启动情况：
$ docker ps -a |grep flink_mysql

进入mysql命令行
docker exec -it flink_mysql bash
mysql -h localhost -u root -p

操作都成功后MySQL环境已经可用

###安装Flink
FLINK 主要部署方式
1. 本地集群
2. 独立集群

以下部署都有 Per Session 和 Per Job二种方式
3. cluster on YARN
4. cluster on Mesos
5. cluster on Docker
6. cluster on Kubernetes 

开发测试环境目前我们只需要配置方式1： 本地集群

Download and Start Flink
$ cd ~/Downloads        # Go to download directory
$ tar xzf flink-*.tgz   # Unpack the downloaded archive
$ cd flink-1.11.2

Start a Local Flink Cluster
$ ./bin/start-cluster.sh  # Start Flink

checking the log files in the logs directory:
$ tail log/flink-*-standalonesession-*.log

Check the Dispatcher’s web frontend
http://localhost:8081


##
https://www.cnblogs.com/toudoushaofeichang/p/11606255.html
https://juejin.im/post/6844903999703891976
https://segmentfault.com/a/1190000022205667
https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/ops/deployment/local.html
https://www.alibabacloud.com/blog/principles-and-practices-of-flink-on-yarn-and-kubernetes-flink-advanced-tutorials_596625


