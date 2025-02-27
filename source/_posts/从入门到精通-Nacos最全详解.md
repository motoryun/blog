---
title: 从入门到精通 Nacos最全详解
date: 2025-02-27 10:27:12
tags: Nacos
categories: 架构
---



1 简介
====

Nacos `/nɑ:kəʊs/` 是 Dynamic Naming and Configuration Service的首字母简称，一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。

服务（Service）是 Nacos 世界的一等公民。Nacos 支持几乎所有主流类型的“服务”的发现、配置和管理：

*   Kubernetes Service

*   gRPC & Dubbo RPC Service

*   Spring Cloud RESTful Service


Nacos生态：

Nacos 几乎支持所有主流语言，其中 Java/Golang/Python 已经支持 Nacos 2.0 长链接协议，能最大限度发挥 Nacos 性能。

阿里微服务 **DNS**（Dubbo+Nacos+Spring-cloud-alibaba/Seata/Sentinel）最佳实践，是 Java 微服务生态最佳解决方案；除此之外，Nacos 也对微服务生态活跃的技术做了无缝的支持，如目前比较流行的 Envoy、Dapr 等，能让用户更加标准获取微服务能力。

![](./2025/02/27/从入门到精通-Nacos最全详解/1.png)

### Nacos的三大核心能力：

![](./2025/02/27/从入门到精通-Nacos最全详解/2.png)

2 安装Nacos
=========

2.1 单机模式安装
----------

### 2.1.1 安装步骤

在使用Nacos之前需要先下载Nacos Server，下载地址：https://github.com/alibaba/nacos/releases/download/1.4.1/nacos-server-1.4.1.zip。

Nacos Server有两种运行模式

*   standanlone：单节点模式

*   cluster：集群模式


单机模式（standanlone）此模式一般用于demo和测试。命令如下。

```


bin/startup.sh -m standalone # linux  
bin/startup.cmd -m standalone # windows  



```

或者修改配置文件startup.cmd，代码如下，默认为cluster，然后直接运行startup.cmd

```


set MODE="standalone"  



```

然后访问http://localhost:8848/nacos，进入nacos管控台，默认账号密码为nacos/nacos，如图8-1所示。

![](./2025/02/27/从入门到精通-Nacos最全详解/3.png)

### 2.1.2 单机模式的数据库

问题来了： Nacos Server 的配置数据是存在哪里呢？我们没有对 Nacos Server 做任何配置，那么数据只有两个位置可以存储：

*   内存

*   本地数据库


随便创建一个配置文件，重启nacos，配置文件还在，说明不是内存存储的。

这时候我们打开NACOS\_PATH/data，会发现里边有个derby-data目录，我们的配置数据现在就存储在这个库中。

> Derby 是 Java 编写的数据库，属于 Apache 的一个开源项目

Nacos Server 的数据源是用 Derby 还是 MySQL 完全是由其运行模式决定的：

*   standalone 的话仅会使用 Derby，即使在 application.properties 里边配置 MySQL 也照样无视；

*   cluster 模式会自动使用 MySQL，这时候如果没有 MySQL 的配置，是会报错的。


> 注意：不支持 MySQL 8.0 版本

2.2 集群模式安装
----------

如果我们要搭建集群的话，那么肯定是不能用内嵌的数据库，不然数据无法共享。所以，集群搭建的时候我们需要将Nacos对接Mysql进行数据存储。

集群模式跟我们平时进行扩容是一样的，可以通过Nginx转发到多个节点，最前面挂一个域名即可

![](./2025/02/27/从入门到精通-Nacos最全详解/4.png)

### 2.2.1 准备虚拟机环境

准备虚拟机安装Nacos集群需要的软件

*   Mysql

*   JDK

*   Nginx



### 2.2.2 Nacos持久化配置

拷贝nacos-server-1.4.1.tar.gz，解压到/user/local/nacos下

```


tar -zxvf nacos-server-1.4.1.tar.gz -C /usr/local  



```

创建nacos数据库，导入nacos\\conf\\nacos-mysql.sql脚本

修改conf\\application.properties配置文件，如下：

```


#\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* Config Module Related Configurations \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*#  
\### If use MySQL as datasource:  
spring.datasource.platform=mysql  
  
\### Count of DB:  
db.num=1  
  
\### Connect URL of DB:  
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC  
db.user.0=root  
db.password.0=123456  
  
\### Connection pool configuration: hikariCP  
db.pool.config.connectionTimeout=30000  
db.pool.config.validationTimeout=10000  
db.pool.config.maximumPoolSize=20  
db.pool.config.minimumIdle=2  



```

让3个Nacos节点都使用同一个数据源

修改nacos\\conf\\cluster文件：

```


10.0.2.15:8848  
10.0.2.15:8858  
10.0.2.15:8868  



```

拷贝2个nacos，修改端口号，分别为8848、8858、8868

```


server.port=8858  



```

启动3个Nacos节点

从之前Nacos导入配置，修改如下

![](./2025/02/27/从入门到精通-Nacos最全详解/5.png)

### 2.2.3 Nginx反向代理配置

修改/usr/local/nginx/conf配置如下：

```


    #gzip  on;  
    upstream nacos\_cluster {  
       server 127.0.0.1:8848;  
       server 127.0.0.1:8858;  
       server 127.0.0.1:8868;  
    }  
    server {  
        listen       8838;  
        server\_name  localhost;  
        location / {  
                proxy\_pass http://nacos\_cluster;  
                proxy\_set\_header Host $host:$server\_port;  
        }  
    }


```

bootstrap.yml配置文件指定nginx地址代码如下。

```


spring:  
  application:  
    name: order-service  
  profiles:  
    active: dev  
  cloud:  
    nacos:  
      config:  
        server-addr: 192.168.56.110:1111  
        file-extension: yaml  
        extension-configs\[0\]:  
          data-id: common.yaml  
          refresh: true  



```

启动测试，关闭一个Nacos节点，测试高可用，Nacos其他两个节点依然可用

启动集群脚本：startupall.sh

```


#!/bin/bash  
echo "starting nacos 8848"  
/home/nacos-cluster/nacos-8848/bin/startup.sh  
echo "starting nacos 8858"  
/home/nacos-cluster/nacos-8858/bin/startup.sh  
echo "starting nacos 8868"  
/home/nacos-cluster/nacos-8868/bin/startup.sh  



```

关闭集群脚本shutdown.sh

```


#!/bin/bash  
echo "starting nacos 8848"  
/home/nacos-cluster/nacos-8848/bin/shutdown.sh  
echo "starting nacos 8858"  
/home/nacos-cluster/nacos-8858/bin/shutdown.sh  
echo "starting nacos 8868"  
/home/nacos-cluster/nacos-8868/bin/shutdown.sh  



```

### 2.2.4 Nacos 2.x集群配置

上述配置在Nacos 1.x下，没有问题，

但是，2.x下Nginx配置，需要对于grpc做反向代理，先看下2.x的RPC通讯的的变化

![](./2025/02/27/从入门到精通-Nacos最全详解/6.png)

怎么配置？

```


stream {  
    upstream nacos-server-grpc {  
        server 127.0.0.1:9848;  
        server 127.0.0.1:9858;  
        server 127.0.0.1:9868;  
    }  
    server {  
        listen 9838;  
        proxy\_pass nacos-server-grpc;  
    }  
}  



```

其中9848 9858 9868端口是grpc请求服务端端口,

由nacos端口号+1000得出, 9838是由8838端口+1000得出

```


\# 端口规则  
server.port(默认8848）  
raft port: ${server.port} - 1000  
grpc port: ${server.port} + 1000  
grpc port for server: ${server.port} + 1001  



```

可以看到grpc用于服务间同步的端口需要+1001,

这就导致在虚拟机跑三台nacos时,三台nacos的端口是不能连续的,否则会出现端口冲突问题


3 注册中心实战
========

使用订单和支付服务，在订单微服务中调用支付微服务，演示Nacos作为注册中心

*   Nacos Server：Nacos注册中心。

*   支付服务：服务提供者。

*   订单服务：服务消费者。


![](./2025/02/27/从入门到精通-Nacos最全详解/7.png)

3.1 父工程
-------

父工程统一管理spring boot,spring cloud和spring cloud alibaba版本号，参考上面版本对应关系

*   Spring Cloud 2020.0.3

*   Spring Cloud Alibaba 2021.1

*   Spring Boot 2.4.9


注意：spring-cloud-starter-bootstrap组件，在Spring Boot 2.4.X版本中，不在加载bootstrap.yml，需要spring-cloud-starter-bootstrap组件，才能加载bootstrap.yml配置文件。

```


<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
  
    <groupId>com.lxs.demo</groupId>  
    <artifactId>cloud-alibaba-demo</artifactId>  
    <version>1.0-SNAPSHOT</version>  
    <modules>  
        <module>payment</module>  
        <module>order</module>  
    </modules>  
  
    <packaging>pom</packaging>  
  
    <parent>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-parent</artifactId>  
        <version>2.4.9</version>  
        <relativePath/> <!-- lookup parent from repository -->  
    </parent>  
  
    <properties>  
        <java.version>1.8</java.version>  
        <alibaba-cloud.version>2021.1</alibaba-cloud.version>  
        <springcloud.version>2020.0.3</springcloud.version>  
    </properties>  
  
    <dependencyManagement>  
        <dependencies>  
            <dependency>  
                <groupId>org.springframework.cloud</groupId>  
                <artifactId>spring-cloud-dependencies</artifactId>  
                <version>${springcloud.version}</version>  
                <type>pom</type>  
                <scope>import</scope>  
            </dependency>  
  
            <dependency>  
                <groupId>com.alibaba.cloud</groupId>  
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>  
                <version>${alibaba-cloud.version}</version>  
                <type>pom</type>  
                <scope>import</scope>  
            </dependency>  
        </dependencies>  
    </dependencyManagement>  
  
    <dependencies>  
  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-actuator</artifactId>  
        </dependency>  
  
        <dependency>  
            <groupId>org.springframework.cloud</groupId>  
            <artifactId>spring-cloud-starter-bootstrap</artifactId>  
        </dependency>  
  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-devtools</artifactId>  
            <scope>runtime</scope>  
            <optional>true</optional>  
        </dependency>  
        <dependency>  
            <groupId>org.projectlombok</groupId>  
            <artifactId>lombok</artifactId>  
            <optional>true</optional>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
            <scope>test</scope>  
        </dependency>  
  
        <dependency>  
            <groupId>org.apache.commons</groupId>  
            <artifactId>commons-lang3</artifactId>  
            <version>3.9</version>  
        </dependency>  
    </dependencies>  
  
    <build>  
        <plugins>  
            <plugin>  
                <groupId>org.springframework.boot</groupId>  
                <artifactId>spring-boot-maven-plugin</artifactId>  
                <configuration>  
                    <excludes>  
                        <exclude>  
                            <groupId>org.projectlombok</groupId>  
                            <artifactId>lombok</artifactId>  
                        </exclude>  
                    </excludes>  
                </configuration>  
            </plugin>  
        </plugins>  
    </build>  
  
  
</project>  



```

3.2 支付微服务（Provider）
-------------------

1.  pom.xml


```


<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns\="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi\="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation\="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"\>  
    <parent\>  
        <artifactId\>cloud-alibaba-demo</artifactId\>  
        <groupId\>com.lxs.demo</groupId\>  
        <version\>1.0-SNAPSHOT</version\>  
    </parent\>  
    <modelVersion\>4.0.0</modelVersion\>  
  
    <artifactId\>payment</artifactId\>  
  
    <dependencies\>  
        <dependency\>  
            <groupId\>org.springframework.boot</groupId\>  
            <artifactId\>spring-boot-starter-web</artifactId\>  
        </dependency\>  
  
        <!--nacos-->  
        <dependency\>  
            <groupId\>com.alibaba.cloud</groupId\>  
            <artifactId\>spring-cloud-starter-alibaba-nacos-discovery</artifactId\>  
        </dependency\>  
  
    </dependencies\>  
  
  
</project\>  



```

2.  application.yml


```


server:  
  port: ${port:9001}  
  
spring:  
  application:  
    name: payment-service  
  cloud:  
    nacos:  
      discovery:  
        server-addr: localhost:8848 #配置Nacos地址  



```

3.  PaymentApplication


```


@SpringBootApplication  
@EnableDiscoveryClient  
public class PaymentApplication  
{  
    public static void main(String\[\] args) {  
        SpringApplication.run(PaymentApplication.class, args);  
    }  
}  



```

4.  PaymentController


```


@RestController  
@RequestMapping("/payment")  
public class PaymentController {  
  
    @Value("${server.port}")  
    private String serverPort;  
  
    @GetMapping("/{id}")  
    public ResponseEntity<String> payment(@PathVariable("id") Long id) {  
        return ResponseEntity.ok("订单号 = " + id + "，支付成功，server.port" + serverPort);  
    }  
  
  
  
}  
  



```

5） 启动支付服务

启动两个服务实例端口号分别为9001和9002，注册到nacos中

配置文件port: ${port:9001}表示，没有port参数，使用9001端口，有port参数则使用port参数指定的端口，使用9002端口的支付服务。

![](./2025/02/27/从入门到精通-Nacos最全详解/8.png)

使用9001端口支付服务。

![](./2025/02/27/从入门到精通-Nacos最全详解/9.png)

访问nacos查看服务列表

![](./2025/02/27/从入门到精通-Nacos最全详解/10.png)

3.3 订单微服务(Consumer)
-------------------

1.  pom.xml


注意：必须依赖spring-cloud-starter-loadbalancer组件，spring-cloud-starter-alibaba-nacos-discovery，不在默认继承ribbon，而是使用Spring Cloud Common总的loadbalancer组件，实现负载均衡

```


<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <parent>  
        <artifactId>cloud-alibaba-demo</artifactId>  
        <groupId>com.lxs.demo</groupId>  
        <version>1.0-SNAPSHOT</version>  
    </parent>  
    <modelVersion>4.0.0</modelVersion>  
  
    <artifactId>order</artifactId>  
  
    <dependencies>  
  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>  
  
        <!--nacos-->  
        <dependency>  
            <groupId>com.alibaba.cloud</groupId>  
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>  
        </dependency>  
        <!--open feign-->  
        <dependency>  
            <groupId>org.springframework.cloud</groupId>  
            <artifactId>spring-cloud-starter-openfeign</artifactId>  
        </dependency>  
          
        <dependency>  
            <groupId>org.springframework.cloud</groupId>  
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>  
        </dependency>  
         
  
    </dependencies>  
  
  
</project>  



```

2.  application.yml


```


server:  
  port: 84  
  
spring:  
  application:  
    name: order-service  
  cloud:  
    nacos:  
      discovery:  
        server-addr: localhost:8848  



```

3.  启动器


```


@EnableDiscoveryClient  
@SpringBootApplication  
@EnableFeignClients  
public class OrderApplication  
{  
    public static void main(String\[\] args) {  
        SpringApplication.run(OrderApplication.class, args);  
    }  
}  



```

4.  ApplicationContextConfig


Nacos底层使用Spring Cloud Common中的Spring Cloud LoadBalancer组件实现负载均衡，注入RestTemplate，使用注解@LoadBalanced开启负载均衡功能。

```


@Configuration  
public class ApplicationContextConfig {  
  
    @Bean  
    @LoadBalanced  
    public RestTemplate getRestTemplate(){  
        return new RestTemplate();  
    }  
}  



```

5.  FeignClient


（1）PaymentFallbackService

```


@Component  
public class PaymentFallbackService implements PaymentService {  
  
    @Override  
    public ResponseEntity<String> payment(Long id) {  
        return new ResponseEntity<String>("feign调用，异常降级方法", HttpStatus.INTERNAL\_SERVER\_ERROR);  
    }  
  
  
}  



```

（2）PaymentService

```


@FeignClient(value = "payment-service", fallback = PaymentFallbackService.class)  
public interface PaymentService {  
  
    @GetMapping("/payment/{id}")  
    public ResponseEntity<String> payment(@PathVariable("id") Long id);  
  
}  



```

6.  OrderController


```


@RestController  
@RequestMapping("/order")  
public class OrderController {  
      
    public static final String SERVICE\_URL = "http://payment-service";  
  
    @Resource  
    private RestTemplate restTemplate;  
  
    @GetMapping("/lb/{id}")  
    public ResponseEntity<String> consumer\_ribbon(@PathVariable("id") Integer id){  
        String result = restTemplate.getForObject("http://payment-service" + "/payment/" + id, String.class);  
        return ResponseEntity.ok(result);  
    }  
      
    //OpenFeign  
    @Resource  
    private PaymentService paymentService;  
  
    @GetMapping(value = "/feign/{id}")  
    public ResponseEntity<String> consumer\_feign(@PathVariable("id") Long id) {  
        return paymentService.payment(id);  
    }  
      
}  



```

5） 启动

访问http://localhost:9003/order/lb/2地址，可以看到9001和9002端口交替执行，因为nacos底层也是采用Spring Cloud Loadbalancer进行负载均衡处理。

![](./2025/02/27/从入门到精通-Nacos最全详解/11.png)

4 配置中心实战
========

配置文件想必大家都不陌生。

在Spring Boot项目中，默认会提供一个application.properties或者application.yml文件，

可以把一些全局性的配置或者需要动态维护的配置写入改文件，比如数据库连接，功能开关，限流阈值，服务地址等。

为了解决不同环境下服务连接配置等信息的差异，Spring Boot还提供了基于spring.profiles.active={profile}的机制来实现不同的环境的切换。

### 配置中心的 起源

随着单体架构向微服务架构的演进，各个应用自己独立维护本地配置文件的方式开始显露出它的不足之处。

主要有下面几点。

*   配置的动态更新：在实际应用会有动态更新位置的需求，比如修改服务连接地址、限流配置等。在传统模式下，需要手动修改配置文件并且重启应用才能生效，这种方式效率太低，重启也会导致服务暂时不可用。

*   配置集中式管理：在微服务架构中某些核心服务为了保证高性能会部署上百个节点，如果在每个节点中都维护一个配置文件，一旦配置文件中的某个属性需要修改，可想而知，工作量是巨大的。

*   不同部署环境下配置的管理：前面提到通过profile机制来管理不同环境下的配置，这种方式对于日常维护来说也比较繁琐。


统一配置管理，就是弥补上述不足的方法。

简单说，是把各个应用系统中的某些配置放在一个第三方中间件上进行统一维护。

然后，对于统一配置中心上的数据的变更需要推送到相应的服务节点实现动态跟新，所以微服务架构中，配置中心也是一个核心组件。

4.1 基本概念
--------

1.  **Profile**


Java项目一般都会有多个Profile配置，用于区分开发环境，测试环境，准生产环境，生成环境等，

每个环境对应一个properties文件（或是yml/yaml文件），然后通过设置 spring.profiles.active 的值来决定使用哪个配置文件。

```


spring:  
  application:  
    name: order-service  
  profiles:  
    active: dev  



```

Nacos Config的作用就把这些文件的内容都移到一个统一的配置中心，即方便维护又支持实时修改后动态刷新应用。

2.  **Data ID**


当使用Nacos Config后，Profile的配置就存储到Data ID下，即一个Profile对应一个Data ID

*   Data ID的拼接格式：`${prefix}-${spring.profiles.active}.${file-extension}`

*   prefix 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix` 来配置

*   `spring.profiles.active` 取 `spring.profiles.active` 的值，即为当前环境对应的 profile，当 `spring.profiles.active` 为空时，对应的连接符 `-`也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`

*   file-extension 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。


3.  **Group**


当配置项太多或者有重名时，可以通过分组来方便管理 ， 这就是 Group

Group 默认为 DEFAULT\_GROUP，

可以通过 spring.cloud.nacos.config.group 来配置，

4.2 配置中心 基础配置
-------------

nacos可以作为配置中心使用，在payment工程中如下步骤，启动nacos配置中心

1.  引入依赖


```


<!--nacos config-->  
        <dependency\>  
            <groupId\>com.alibaba.cloud</groupId\>  
            <artifactId\>spring-cloud-starter-alibaba-nacos-config</artifactId\>  
        </dependency\>  
        <dependency\>  
            <groupId\>org.springframework.cloud</groupId\>  
            <artifactId\>spring-cloud-starter-bootstrap</artifactId\>  
        </dependency\>  



```

2.  配置文件


（1）bootstrap.yml

注意：Sprnig Boot 2.4.X版本后需要手动添加spring-cloud-starter-bootstrap组件后，才能加载bootstrap.yml配置文件

```


spring:  
  application:  
    name: payment-service  
  profiles:  
    active: dev  
  cloud:  
    nacos:  
      config:  
        server-addr: localhost:8848  
        file-extension: yaml  



```

注意：spring.cloud.nacos.config配置必须放到bootstrap.yml配置文件中，保证在优先读取配置文件再启动，否则配置无效

（2）application.yml

```


server:  
  port: ${port:9001}  
spring:  
  cloud:  
    nacos:  
      discovery:  
        server-addr: localhost:8848 #配置Nacos地址  



```

3.  nacos中的配置DataID


访问nacos在配置列表中增加如下配置

![](./2025/02/27/从入门到精通-Nacos最全详解/12.png)

当使用Nacos Config后，Profile的配置就存储到Data ID下，即一个Profile对应一个Data ID

*   Data ID的拼接格式：`${prefix}-${spring.profiles.active}.${file-extension}`

*   prefix 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix` 来配置

*   `spring.profiles.active` 取 `spring.profiles.active` 的值，即为当前环境对应的 profile，当 `spring.profiles.active` 为空时，对应的连接符 `-`也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`

*   file-extension 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。


对应关系

![](./2025/02/27/从入门到精通-Nacos最全详解/13.png)

4.  业务中读取配置属性


```


@RestController  
@RequestMapping("/payment")  
@RefreshScope  
public class PaymentController {  
  
    @Value("${config.info}")  
    private String configInfo;  
  
    @GetMapping("/config/info")  
    public String getConfigInfo() {  
        return configInfo;  
    }  
}  



```

5.  测试


访问地址http://localhost:9001/payment/config/info，显示`config info public default group`

当修改配置值，会结果已经改变，Nacos自带自动刷新功能。

4.3 配置隔离
--------

通常，企业研发的流程是这样的：先在开发测试环境开发和测试功能，然后灰度，最后发布到生产环境。

并且，为了生产环境的稳定，需要将测试环境和生产环境进行隔离，Nacos可以通过三种方式进行配置隔离：

*   Nacos的服务器

*   namespace命名空间

*   group分组，


在bootstrap.yml文件中可以通过配置Nacos的server-addr、namespace和group来区分不同的配置信息。

*   Nacos的服务器 spring.cloud.nacos.config.server-addr

*   Nacos的命名空间 spring.cloud.nacos.config.namespace，注意，这里使用命名空间的ID不是名称

*   Nacos的分组 spring.cloud.nacos.config.group


Nacos配置隔离具体步骤如下。

1.  命名空间


创建dev,test,prod

![](./2025/02/27/从入门到精通-Nacos最全详解/14.png)

2.  DataID


在不同命名空间下创建如下DataID（克隆即可）

![](./2025/02/27/从入门到精通-Nacos最全详解/15.png)

3.  bootstrap.yml


配置读取相应的配置，bootstrap.yml代码如下。

```


spring:  
  application:  
    name: payment-service  
  profiles:  
    active: dev  
  cloud:  
    nacos:  
      config:  
        server-addr: localhost:8848  
        file-extension: yaml  
        namespace: f521b45b-e0bb-4c4d-b97c-3d0158970001  
        group: MY\_GROUP  



```

注意：如果不配置namespace默认为public，不配置group默认为DEFAULT\_GROUP

此时读取的配置为namespace=prod，group=MY\_GROUP的配置页面显示`config info prod public group`

4.  service隔离


同样注册service时可以指定指定namespace隔离注册到哪一个命名空间。application.yml如下。

```


server:  
  port: ${port:9001}  
spring:  
  cloud:  
    nacos:  
      discovery:  
        server-addr: localhost:8848 #配置Nacos地址  
        namespace: f521b45b-e0bb-4c4d-b97c-3d0158970001  



```

4.4 配置拆分
--------

1.  配置拆分策略


项目中会有很多的微服务，必然会存在很多具体配置，和重复配置，可以采用如下方案管理配置

![](./2025/02/27/从入门到精通-Nacos最全详解/16.png)

根据上面分析在nacos中配置

![](./2025/02/27/从入门到精通-Nacos最全详解/17.png)

2.  DataID配置


payment-service-dev.yaml

![](./2025/02/27/从入门到精通-Nacos最全详解/18.png)

common.yaml

![](./2025/02/27/从入门到精通-Nacos最全详解/19.png)

3.  配置文件


bootstrap.yml，配置代码如下所示

```


spring:  
  application:  
    name: payment-service  
  profiles:  
    active: dev  
  cloud:  
    nacos:  
      config:  
        server-addr: localhost:8848  
        file-extension: yaml  
        namespace: f521b45b-e0bb-4c4d-b97c-3d0158970001  
        group: MY\_GROUP  
        extension-configs\[0\]:  
          data-id: common.yaml  
          refresh: true  



```

*   注：extension-configs配置属性和shared-configs配置属性功能一致，都是读取配置文件，这里我们把重复配置放到common.yaml中，这样在不同的工程中就可以重复使用\[n\]的值越大，优先级越高。

*   注：`spring.application.name`和`spring.profiles.active`配置必须放到bootstrap.yml中，否则影响配置自动刷新功能。


按照上面方案，整合拆分订单微服务的配置文件

![](./2025/02/27/从入门到精通-Nacos最全详解/20.png)

order微服务bootstrap.yml

```


spring:  
  application:  
    name: order-service  
  profiles:  
    active: dev  
  cloud:  
    nacos:  
      config:  
        server-addr: localhost:8848 #nacos 配置中心的地址  
        file-extension: yaml  
        extension-configs\[0\]:  
          data-id: common.yaml  
          refresh: true  



```

注意：这里两个工程使用重用的配置文件common.yaml

5 Nacos架构和 底层原理
===============

**思考下面几个面试高频问题？**

*   临时实例和永久实例是什么？有什么区别？

*   服务实例是如何注册到服务端的？

*   服务实例和服务端之间是如何保活的？

*   服务订阅是如何实现的？

*   集群间数据是如何同步的？CP还是AP？

*   Nacos的数据模型是什么样的？


后续原理部分内容，主要解释以下Nacos面试高频问题：

5.1 Nacos架构图
------------

整体架构分为用户层、业务层、内核层和插件，‘

*   用户层主要解决用户使用的易用性问题，

*   业务层主要解决服务发现和配置管理的功能问题，

*   内核层解决分布式系统一致性、存储、高可用等核心问题，插件解决扩展性问题。
![](./2025/02/27/从入门到精通-Nacos最全详解/21.png)


1.  用户层


*   OpenAPI：暴露标准Rest风格HTTP接口，简单易用，方便多语言集成

*   Console：易用控制台，做服务管理、配置管理等操作

*   SDK：多语言 SDK，目前几乎支持所有主流编程语言

*   Agent：Sidecar 模式运行，通过标准 DNS 协议与业务解耦

*   CLI：命令行对产品进行轻量化管理，像 git 一样好用


2.  业务层


*   服务管理：实现服务 CRUD，域名 CRUD，服务健康状态检查，服务权重管理等功能

*   配置管理：实现配置管 CRUD，版本管理，灰度管理，监听管理，推送轨迹，聚合数据等功能

*   元数据管理：提供元数据 CURD 和打标能力，为实现上层流量和服务灰度非常关键


3.  内核层


*   插件机制：实现三个模块可分可合能力，实现扩展点 SPI 机制，用于扩展自己公司定制

*   事件机制：实现异步化事件通知，SDK 数据变化异步通知等逻辑，是Nacos高性能的关键部分

*   日志模块：管理日志分类，日志级别，日志可移植性（尤其避免冲突），日志格式，异常码+帮助文档

*   回调机制：SDK 通知数据，通过统一的模式回调用户处理。接口和数据结构需要具备可扩展性

*   寻址模式：解决 Server IP 直连，域名访问，Nameserver 寻址、广播等多种寻址模式，需要可扩展

*   推送通道：解决 Server 与存储、Server 间、Server 与 SDK 间高效通信问题

*   容量管理：管理每个租户，分组下的容量，防止存储被写爆，影响服务可用性

*   流量管理：按照租户，分组等多个维度对请求频率，长链接个数，报文大小，请求流控进行控制

*   缓存机制：容灾目录，本地缓存，Server 缓存机制，是 Nacos 高可用的关键

*   启动模式：按照单机模式，配置模式，服务模式，DNS 模式，启动不同的模块

*   一致性协议：解决不同数据，不同一致性要求情况下（包括AP 协议和CP协议）的诉求

*   存储模块：解决数据持久化、非持久化存储，解决数据分片问题


4.  插件


*   Nameserver：解决 Namespace 到 ClusterID 的路由问题，解决用户环境与 Nacos 物理环境映射问题

*   CMDB：解决元数据存储，与三方 CMDB 系统对接问题，解决应用，人，资源关系

*   Metrics：暴露标准 Metrics 数据，方便与三方监控系统打通

*   Trace：暴露标准 Trace，方便 SLA 系统打通，日志白平化，推送轨迹等能力，并且可以和计量计费系统打

*   接入管理：相当于阿里云开通服务，分配身份、容量、权限过程

*   用户管理：解决用户管理，登录，SSO 等问题

*   权限管理：解决身份识别，访问控制，角色管理等问题

*   审计系统：扩展接口方便与不同公司审计系统打通

*   通知系统：核心数据变更，或者操作，方便通过SMS系统打通，通知到对应人数据变更


5.2 配置模型
--------

在单体架构的时候我们可以将配置写在配置文件中，但有一个缺点就是每次修改配置都需要重启服务才能生效。  
当应用程序实例比较少的时候还可以维护。

如果转向微服务架构有成百上千个实例，每修改一次配置要将全部实例重启，不仅增加了系统的不稳定性，也提高了维护的成本。

![](./2025/02/27/从入门到精通-Nacos最全详解/22.png)

那么如何能够做到服务不重启就可以修改配置？所有就产生了四个基础诉求：

*   需要支持动态修改配置

*   需要动态变更实时生效

*   变更快了之后如何管控控制变更风险，如灰度、回滚等

*   敏感配置如何做安全配置


![](./2025/02/27/从入门到精通-Nacos最全详解/img.png)

### 5.2.1 基础模型



![](./2025/02/27/从入门到精通-Nacos最全详解/img_1.png)


上图是 Nacos 配置管理的基础模型：

1.  Nacos 提供可视化的控制台，可以对配置进行发布、更新、删除、灰度、版本管理等功能。

2.  SDK 可以提供发布配置、更新配置、监听配置等功能。

3.  SDK 通过 GRPC 长连接监听配置变更，Server 端对比 Client 端配置的 MD5 和本地 MD5 是否相等，不相等推送配置变更。

4.  SDK会保存配置的快照，当服务端出现问题的时候从本地获取。


### 5.2.2 配置资源模型

Namespace 的设计就是用来进行资源隔离的，我们在进行配置资源的时候可以从以下两个角度来看：

*   从单个租户的角度来看，我们要配置多套环境的配置，可以根据不同的环境来创建 Namespace 。比如开发环境、测试环境、线上环境，我们就创建对应的 Namespace（dev、test、prod），Nacos 会自动生成对应的 Namespace Id 。如果同一个环境内想配置相同的配置，可以通过 Group 来区分。如下图所示：


![](./2025/02/27/从入门到精通-Nacos最全详解/img_2.png)

*   从多个租户的角度来看，每个租户都可以有自己的命名空间。我们可以为每个用户创建一个命名空间，并给用户分配对应的权限，比如多个租户（zhangsan、lisi、wangwu），每个租户都想有一套自己的多环境配置，也就是每个租户都想配置多套环境。那么可以给每个租户创建一个 Namespace （zhangsan、lisi、wangwu）。同样会生成对应的 Namespace Id。然后使用 Group 来区分不同环境的配置。如下图所示：

* ![](./2025/02/27/从入门到精通-Nacos最全详解/img_3.png)


### 5.2.3 配置存储模型（ER图）

![](./2025/02/27/从入门到精通-Nacos最全详解/img_4.png)


Nacos存储配置有几个比较重要的表分别是:

*   config\_info 存储配置信息的主表，里面包含 dataId、groupId、content、tenantId、encryptedDataKey 等数据。

*   config\_info\_beta 灰度测试的配置信息表，存储的内容和 config\_info 基本相似。有一个 beta\_ips 字段用于客户端请求配置时判断是否是灰度的ip。

*   config\_tags\_relation 配置的标签表，在发布配置的时候如果指定了标签，那么会把标签和配置的关联信息存储在该表中。

*   his\_config\_info 配置的历史信息表，在配置的发布、更新、删除等操作都会记录一条数据，可以做多版本管理和快速回滚。


5.3 分布式Nacos 节点之间，数据一致性设计
-------------------------

集群模式下，需要考虑如何保障各个节点之间的数据一致性以及数据同步，

而要解决这个问题，就不得不引入共识算法，通过算法来保障各个节点之间的数据的一致性。

### 5.3.1 CAP定理和BASE理论

谈到数据一致性问题，一定离不开两个著名分布式理论

*   CAP定理

*   BASE理论


CAP定理中，三个字母分别代表这些含义：

*   C，Consistency单词的缩写，代表一致性，指分布式系统中各个节点的数据保持强一致，也就是每个时刻都必须一样，不一样整个系统就不能对外提供服务

*   A，Availability单词的缩写，代表可用性，指整个分布式系统保持对外可用，即使从每个节点获取的数据可能都不一样，只要能获取到就行

*   P，Partition tolerance单词的缩写，代表分区容错性。


所谓的CAP定理，就是指在一个分布式系统中，CAP这三个指标，最多同时只能满足其中的两个，不可能三个都同时满足

![](./2025/02/27/从入门到精通-Nacos最全详解/img_5.png)

为什么三者不能同时满足？

对于一个分布式系统，网络分区是一定需要满足的

而所谓分区指的是系统中的服务部署在不同的网络区域中

![](./2025/02/27/从入门到精通-Nacos最全详解/img_6.png)

比如，同一套系统可能同时在北京和上海都有部署，那么他们就处于不同的网络分区，就可能出现无法互相访问的情况

当然，你也可以把所有的服务都放在一个网络分区，但是当网络出现故障时，整个系统都无法对外提供服务，那这还有什么意义呢？

所以分布式系统一定需要满足分区容错性，把系统部署在不同的区域网络中

此时只剩下了一致性和可用性，它们为什么不能同时满足？

其实答案很简单，就因为可能出现网络分区导致的通信失败。

比如说，现在出现了网络分区的问题，上图中的A网络区域和B网络区域无法相互访问

此时假设往上图中的A网络区域发送请求，将服务中的一个值 i 属性设置成 1

![](./2025/02/27/从入门到精通-Nacos最全详解/img_7.png)

如果保证可用性，此时由于A和B网络不通，此时只有A中的服务修改成功，B无法修改成功，此时数据AB区域数据就不一致性，也就没有保证数据一致性

如果保证一致性，此时由于A和B网络不通，所以此时A也不能修改成功，必须修改失败，否则就会导致AB数据不一致

虽然A没修改成功，保证了数据一致性，AB还是之前相同的数据，但是此时整个系统已经没有写可用性了，无法成功写数据了。

所以从上面分析可以看出，在有分区容错性的前提下，可用性和一致性是无法同时保证的。

虽然无法同时一致性和可用性，但是能不能换种思路来思考一下这个问题

首先我们可以先保证系统的可用性，也就是先让系统能够写数据，将A区域服务中的i修改成1

之后当AB区域之间网络恢复之后，将A区域的i值复制给B区域，这样就能够保证AB区域间的数据最终是一致的了

这不就皆大欢喜了么

这种思路其实就是BASE理论的核心要点，优先保证可用性，数据最终达成一致性。

BASE理论主要是包括以下三点：

*   基本可用（Basically Available）：系统出现故障还是能够对外提供服务，不至于直接无法用了

*   软状态（Soft State）：允许各个节点的数据不一致

*   最终一致性，（Eventually Consistent）：虽然允许各个节点的数据不一致，但是在一定时间之后，各个节点的数据最终需要一致的


BASE理论其实就是妥协之后的产物。

### 5.3.2 Nacos是AP还是CP

Nacos是AP还是CP？

为什么 Nacos 会在单个集群中同时运行 CP 协议以及 AP 协议呢？

这其实要从 Nacos 的场景出发的：Nacos 是一个集服务注册发现以及配置管理于一体的组件，因此对于集群下，各个节点之间的数据一致性保障问题，需要拆分成两个方面

1.  从服务注册发现来看


服务发现注册中心，在当前微服务体系下，是十分重要的组件，服务之间感知对方服务的当前可正常提供服务的实例信息，必须从服务发现注册中心进行获取，

因此对于服务注册发现中心组件的可用性，提出了很高的要求，需要在任何场景下，尽最大可能保证服务注册发现能力可以对外提供服务；

同时 Nacos 的服务注册发现设计，采取了心跳可自动完成服务数据补偿的机制。如果数据丢失的话，是可以通过该机制快速弥补数据丢失。

因此，为了满足服务发现注册中心的可用性，强一致性的共识算法这里就不太合适了，因为强一致性共识算法能否对外提供服务是有要求的，如果当前集群可用的节点数没有过半的话，整个算法直接“罢工”，而最终一致共识算法的话，更多保障服务的可用性，并且能够保证在一定的时间内各个节点之间的数据能够达成一致。

上述的都是针对于Nacos服务发现注册中的非持久化服务而言（即需要客户端上报心跳进行服务实例续约）。

而对于Nacos服务发现注册中的持久化服务，因为所有的数据都是直接使用调用Nacos服务端直接创建，因此需要由Nacos保障数据在各个节点之间的强一致性，故而针对此类型的服务数据，选择了强一致性共识算法来保障数据的一致性。

2.  从配置管理来看


配置数据，是直接在 Nacos 服务端进行创建并进行管理的，必须保证大部分的节点都保存了此配置数据才能认为配置被成功保存了，否则就会丢失配置的变更，

如果出现这种情况，问题是很严重的，如果是发布重要配置变更出现了丢失变更动作的情况，那多半就要引起严重的现网故障了，因此对于配置数据的管理，是必须要求集群中大部分的节点是强一致的，而这里的话只能使用强一致性共识算法。

**总结：Nacos是AP还是CP？**

*   Nacos其实目前是同时支持AP和CP的

*   具体使用AP还是CP得取决于Nacos内部的具体功能，并不是有的文章说的可以通过一个配置自由切换。

*   服务注册场景，


*   临时实例，Nacos会优先保证可用性，也就是AP

*   永久实例，Nacos会优先保证数据的一致性，也就是CP


*   就配置管理而言是CP


### 5.3.3 Nacos一致性协议

为什么选择 Raft 和 Distro 呢？

对于强一致性共识算法，当前工业生产中，最多使用的就是 Raft 协议，Raft 协议更容易让人理解，并且有很多成熟的工业算法实现，比如蚂蚁金服的 JRaft、Zookeeper 的 ZAB、Consul 的 Raft、百度的 braft、Apache Ratis；因为Nacos是Java技术栈，

因此只能在 JRaft、ZAB、Apache Ratis 中选择，但是 ZAB 因为和Zookeeper 强绑定，再加上希望可以和 Raft 算法库的支持团队随时沟通交流，因此选择了 JRaft，选择 JRaft 也是因为 JRaft 支持多 RaftGroup，为 Nacos 后面的多数据分片带来了可能。

而 Distro 协议是阿里巴巴自研的一个最终一致性协议，而最终一致性协议有很多，比如 Gossip、Eureka 内的数据同步算法。

而 Distro 算法是集 Gossip 以及 Eureka 协议的优点并加以优化而出来的，对于原生的Gossip，由于随机选取发送消息的节点，也就不可避免的存在消息重复发送给同一节点的情况，增加了网络的传输的压力，也给消息节点带来额外的处理负载，

而 Distro 算法引入了权威 Server 的概念，每个节点负责一部分数据以及将自己的数据同步给其他节点，有效的降低了消息冗余的问题。

#### 早期的一致性协议

我们先来看看早期的Naocs版本的架构

![](./2025/02/27/从入门到精通-Nacos最全详解/img_8.png)

在早期的 Nacos 架构中，服务注册和配置管理一致性协议是分开的，没有下沉到 Nacos 的内核模块作为通用能力演进，服务发现模块一致性协议的实现和服务注册发现模块的逻辑强耦合在一起，并且充斥着服务注册发现的一些概念。

这使得 Nacos 的服务注册发现模块的逻辑变得复杂且难以维护，耦合了一致性协议层的数据状态，难以做到计算存储彻底分离，以及对计算层的无限水平扩容能力也有一定的影响。

因此为了解决这个问题，必然需要对 Nacos 的一致性协议做抽象以及下沉，使其成为 Core 模块的能力，彻底让服务注册发现模块只充当计算能力，同时为配置模块去外部数据库存储打下了架构基础。

#### 当前的一致性协议层

正如前面所说，在当前的 Nacos 内核中，我们已经做到了将一致性协议的能力，

完全下沉到了内核模块作为Nacos 的核心能力，很好的服务于服务注册发现模块以及配置管理模块，我们来看看当前 Nacos 的架构。

![](./2025/02/27/从入门到精通-Nacos最全详解/img_9.png)

可以发现，在新的 Nacos 架构中，已经完成了将一致性协议从原先的服务注册发现模块下沉到了内核模块当中，

并且尽可能的提供了统一的抽象接口，使得上层的服务注册发现模块以及配置管理模块，不再需要耦合任何一致性语义，解耦抽象分层后，每个模块能快速演进，并且性能和可用性都大幅提升。

#### 一致性协议下沉

既然 Nacos 已经做到了将 AP、CP 协议下沉到了内核模块，而且尽可能的保持了一样的使用体验。

那么这个一致性协议下沉，Nacos 是如何做到的呢？

1.  一致性协议抽象


其实，一致性协议，就是用来保证数据一致的，而数据的产生，必然有一个写入的动作；

同时还要能够读数据，并且保证读数据的动作以及得到的数据结果，并且能够得到一致性协议的保障。因此，一致性协议最最基础的两个方法，就是写动作和读动作

```


public interface ConsistencyProtocol<T extends Config, P extends RequestProcessor> extends CommandOperations {  
  
  
    ...  
  
  
    /\*\*  
     \* Obtain data according to the request.  
     \*  
     \* @param request request  
     \* @return data {@link Response}  
     \* @throws Exception {@link Exception}  
     \*/  
    Response getData(ReadRequest request) throws Exception;  
  
  
    /\*\*  
     \* Data operation, returning submission results synchronously.  
     \*  
     \* @param request {@link com.alibaba.nacos.consistency.entity.WriteRequest}  
     \* @return submit operation result {@link Response}  
     \* @throws Exception {@link Exception}  
     \*/  
    Response write(WriteRequest request) throws Exception;  
  
  
    ...  
  
  
}  



```

任何使用一致性协议的，都只需要使用 getData 以及 write 方法即可。

同时，一致性协议已经被抽象在了consistency 的包中，Nacos 对于 AP、CP 的一致性协议接口使用抽象都在里面，并且在实现具体的一致性协议时，采用了插件可插拔的形式，进一步将一致性协议具体实现逻辑和服务注册发现、配置管理两个模块达到解耦的目的。

```


public class ProtocolManager extends MemberChangeListener implements DisposableBean {  
  
  
    ...  
    private void initAPProtocol() {  
        ApplicationUtils.getBeanIfExist(APProtocol.class, protocol -> {  
            Class configType = ClassUtils.resolveGenericType(protocol.getClass());  
            Config config = (Config) ApplicationUtils.getBean(configType);  
            injectMembers4AP(config);  
            protocol.init((config));  
            ProtocolManager.this.apProtocol = protocol;  
        });  
    }  
  
  
    private void initCPProtocol() {  
        ApplicationUtils.getBeanIfExist(CPProtocol.class, protocol -> {  
            Class configType = ClassUtils.resolveGenericType(protocol.getClass());  
            Config config = (Config) ApplicationUtils.getBean(configType);  
            injectMembers4CP(config);  
            protocol.init((config));  
            ProtocolManager.this.cpProtocol = protocol;  
        });  
    }  
    ...  
}  



```

其实，仅做完一致性协议抽象是不够的，如果只做到这里，那么服务注册发现以及配置管理，还是需要依赖一致性协议的接口，在两个计算模块中耦合了带状态的接口；

并且，虽然做了比较高度的一致性协议抽象，服务模块以及配置模块却依然还是要在自己的代码模块中去显示的处理一致性协议的读写请求逻辑，以及需要自己去实现一个对接一致性协议的存储，这其实是不好的，服务发现以及配置模块，更多应该专注于数据的使用以及计算，而非数据怎么存储、怎么保障数据一致性，数据存储以及多节点一致的问题应该交由存储层来保证。

为了进一步降低一致性协议出现在服务注册发现以及配置管理两个模块的频次以及尽可能让一致性协议只在内核模块中感知，Nacos这里又做了另一份工作——数据存储抽象。

2.  数据存储抽象


正如前面所说，一致性协议，就是用来保证数据一致的，如果利用一致性协议实现一个存储，那么服务模块以及配置模块，就由原来的依赖一致性协议接口转变为了依赖存储接口，而存储接口后面的具体实现，就比一致性协议要丰富得多了，并且服务模块以及配置模块也无需为直接依赖一致性协议而承担多余的编码工作（快照、状态机实现、数据同步）。

使得这两个模块可以更加的专注自己的核心逻辑。对于数据抽象，这里仅以服务注册发现模块为例

```


public interface KvStorage {  
  
  
    enum KvType {  
        /\*\*  
         \* Local file storage.  
         \*/  
        File,  
  
  
        /\*\*  
         \* Local memory storage.  
         \*/  
        Memory,  
  
  
        /\*\*  
         \* LSMTree storage.  
         \*/  
        LSMTree,  
  
  
        AP,  
  
  
        CP,  
    }  
  
  
    // 获取一个数据  
    byte\[\] get(byte\[\] key) throws KvStorageException;  
    // 存入一个数据  
    void put(byte\[\] key, byte\[\] value) throws KvStorageException;  
    // 删除一个数据  
    void delete(byte\[\] key) throws KvStorageException;  
  
  
    ...  
  
  
}  



```

由于 Nacos 的服务模块存储，更多的都是根据单个或者多个唯一key去执行点查的操作，因此 Key-Value 类型的存储接口最适合不过。

而 Key-Value 的存储接口定义好之后，其实就是这个 KVStore 的具体实现了。可以直接将 KVStore 的实现对接 Redis，也可以直接对接 DB ，或者直接根据 Nacos 内核模块的一致性协议，再此基础之上，实现一个内存或者持久化的分布式强（弱）一致性 KV。

通过功能边界将 Nacos 进一步分离为计算逻辑层和存储逻辑层，计算层和存储层之间的交互仅通过一层薄薄的数据操作胶水代码，这样就在单个Nacos 进程里面实现了计算和存储二者逻辑的彻底分离。



![](./2025/02/27/从入门到精通-Nacos最全详解/img_10.png)

同时，针对存储层，进一步实现插件化的设计，对于中小公司且有运维成本要求的话，可以直接使用 Nacos 自带的内嵌分布式存储组件来部署一套 Nacos 集群，而如果服务实例数据以及配置数据的量级很大的话，并且本身有一套比较好的 Paas 层服务，那么完全可以复用已有的存储组件，实现 Nacos 的计算层与存储层彻底分离。

### 5.3.4 Distro协议（AP实现）

对于AP来说，Nacos使用的是阿里自研的Distro协议

Distro 协议是 Nacos 社区自研的一种 AP 分布式协议，是面向临时实例设计的一种分布式协议，其保证了在某些 Nacos 节点宕机后，整个临时实例处理系统依旧可以正常工作。作为一种有状态的中间件应用的内嵌协议，Distro 保证了各个 Nacos 节点对于海量注册请求的统一协调和存储。

#### 设计思想

Distro 协议的主要设计思想如下：

*   Nacos 每个节点是平等的都可以处理写请求，同时把新数据同步到其他节点。

*   每个节点只负责部分数据，定时发送自己负责数据的校验值到其他节点来保持数据一致性。

*   每个节点独立处理读请求，及时从本地发出响应。


下面几节将分为几个场景进行 Distro 协议工作原理的介绍。

#### 数据初始化

新加入的 Distro 节点会进行全量数据拉取。具体操作是轮询所有的 Distro 节点，通过向其他的机器发送请求拉取全量数据。

![](./2025/02/27/从入门到精通-Nacos最全详解/img_11.png)

在全量拉取操作完成之后，Nacos 的每台机器上都维护了当前的所有注册上来的非持久化实例数据。

#### 数据校验

在 Distro 集群启动之后，各台机器之间会定期的发送心跳。心跳信息主要为各个机器上的所有数据的元信息（之所以使用元信息，是因为需要保证网络中数据传输的量级维持在一个较低水平）。这种数据校验会以心跳的形式进行，即每台机器在固定时间间隔会向其他机器发起一次数据校验请求。

![](./2025/02/27/从入门到精通-Nacos最全详解/img_12.png)

一旦在数据校验过程中，某台机器发现其他机器上的数据与本地数据不一致，则会发起一次全量拉取请求，将数据补齐。

#### 写操作

对于一个已经启动完成的 Distro 集群，在一次客户端发起写操作的流程中，当注册非持久化的实例的写请求打到某台 Nacos 服务器时，Distro 集群处理的流程图如下。

![](./2025/02/27/从入门到精通-Nacos最全详解/img_13.png)


整个步骤包括几个部分（图中从上到下顺序)：

*   前置的 Filter 拦截请求，并根据请求中包含的 IP 和 port 信息计算其所属的 Distro 责任节点，并将该请求转发到所属的 Distro 责任节点上。

*   责任节点上的 Controller 将写请求进行解析。

*   Distro 协议定期执行 Sync 任务，将本机所负责的所有的实例信息同步到其他节点上。


#### 读操作

由于每台机器上都存放了全量数据，因此在每一次读操作中，Distro 机器会直接从本地拉取数据。快速响应。



![](./2025/02/27/从入门到精通-Nacos最全详解/img_14.png)

对于读操作都进行及时的响应。在网络分区的情况下，对于所有的读操作也能够正常返回；当网络恢复时，各个 Distro 节点会把各数据分片的数据进行合并恢复。

#### Distro 协议 的总结

Distro 协议是 Nacos 对于临时实例数据开发的一致性协议。其数据存储在缓存中，并且会在启动时进行全量数据同步，并定期进行数据校验。

在 Distro 协议的设计思想下，每个 Distro 节点都可以接收到读写请求。所有的 Distro 协议的请求场景主要分为三种情况：

1、当该节点接收到属于该节点负责的实例的写请求时，直接写入。

2、当该节点接收到不属于该节点负责的实例的写请求时，将在集群内部路由，转发给对应的节点，从而完成读写。

3、当该节点接收到任何读请求时，都直接在本机查询并返回（因为所有实例都被同步到了每台机器上）。

Distro 协议作为 Nacos 的内嵌临时实例一致性协议，保证了在分布式环境下每个节点上面的服务信息的状态都能够及时地通知其他节点，可以维持数十万量级服务实例的存储和一致性。

### 5.3.5 Nacos中CP的实现

Nacos的CP实现是基于Raft算法来实现的

在1.x版本早期，Nacos是自己手动实现Raft算法

在2.x版本，Nacos移除了手动实现Raft算法，转而拥抱基于蚂蚁开源的JRaft框架

在Raft算法，每个节点主要有三个状态

*   Leader，负责所有的读写请求，一个集群只有一个

*   Follower，从节点，主要是负责复制Leader的数据，保证数据的一致性

*   Candidate，候选节点，最终会变成Leader或者Follower


集群启动时都是节点Follower，经过一段时间会转换成Candidate状态，再经过一系列复杂的选择算法，选出一个Leader，后面在介绍这个算法

![](./2025/02/27/从入门到精通-Nacos最全详解/img_15.png)

当有写请求时，如果请求的节点不是Leader节点时，会将请求转给Leader节点，由Leader节点处理写请求

比如，有个客户端连到的上图中的`Nacos服务2`节点，之后向`Nacos服务2`注册服务

`Nacos服务2`接收到请求之后，会判断自己是不是Leader节点，发现自己不是

此时`Nacos服务2`就会向Leader节点发送请求，Leader节点接收到请求之后，会处理服务注册的过程

为什么说Raft是保证CP的呢？主要是因为Raft在处理写的时候有一个判断过程

*   首先，Leader在处理写请求时，不会直接数据应用到自己的系统，而是先向所有的Follower发送请求，让他们先处理这个请求

*   当超过半数的Follower成功处理了这个写请求之后，Leader才会写数据，并返回给客户端请求处理成功

*   如果超过一定时间未收到超过半数处理成功Follower的信号，此时Leader认为这次写数据是失败的，就不会处理写请求，直接返回给客户端请求失败


所以，一旦发生故障，导致接收不到半数的Follower写成功的响应，整个集群就直接写失败，这就很符合CP的概念了。

不过这里还有一个小细节需要注意

Nacos在处理查询服务实例的请求直接时，并不会将请求转发给Leader节点处理，而是直接查当前Nacos服务实例的注册表

这其实就会引发一个问题

如果客户端查询的Follower节点没有及时处理Leader同步过来的写请求（过半响应的节点中不包括这个节点），此时在这个Follower其实是查不到最新的数据的，这就会导致数据的不一致

所以说，虽然Raft协议规定要求从Leader节点查最新的数据，但是Nacos至少在读服务实例数据时并没有遵守这个协议

当然对于其它的一些数据的读写请求有的还是遵守了这个协议。

JRaft对于读请求其实是做了很多优化的，其实从Follower节点通过一定的机制也是能够保证读到最新的数据

### 5.3.6 Raft协议

Raft 适用于一个管理日志一致性的协议，相比于 Paxos 协议， Raft 更易于理解和去实现它。

#### 基础概念

Raft 算法是通过一切以领导者为准的方式，实现一系列数据的共识和各节点日志的一致。Raft是强领导模型，集群中只能有一个领导者。

成员身份：领导者（Leader）、跟随者（Follower）和候选者（Candidate）。

![](./2025/02/27/从入门到精通-Nacos最全详解/img_16.png)

*   领导者：集群中霸道总裁，一切以我为准，处理写请求、管理日志复制和不断地发送心跳信息。

*   跟随者：普通成员，处理领导者发来的消息，发现领导者心跳超时，推荐自己成为候选人。

*   候选人：先给自己投一票，然后请求其他集群节点投票给自己，得票多者成为新的领导者。

*   任期编号：每任领导者都有任期编号。当领导者心跳超时，跟随者会变成候选人，任期编号 +1，然后发起投票。任期编号小的服从编号大的。

*   心跳超时：每个跟随者节点都设置了随机心跳超时时间，目的是避免跟随者们同时成为候选人，同时发起投票。

*   选举超时：每轮选举的结束时间，随机选举超时时间可以避免多个候选人同时结束选举未果，然后同时发起下一轮选举


#### 领导选举

1.  **选举规则**


*   领导者周期性地向跟随者发送心跳消息，告诉跟随者我是领导者，阻止跟随者发变成候选人发起新选举；

*   如果跟随者的随机心跳超时了，那么认为没有领导者了，推荐自己为候选人，发起新的选举；

*   在一轮选举中，赢得一半以上选票的候选人，即成为新的领导者；

*   在一轮选举中，每个节点对每个任期编号的选举只能投出一票，先来先服务原则；

*   日志完整性高（也就是最后一条日志项对应的任期编号值更大，索引号更大）的跟随者A拒绝给完整性低的候选人B投票，即使B的任期编号大；


2.  **选举动画**


![](./2025/02/27/从入门到精通-Nacos最全详解/img_17.png)

初始选举

![](./2025/02/27/从入门到精通-Nacos最全详解/img_18.png)

领导者宕机/断网

![](./2025/02/27/从入门到精通-Nacos最全详解/img_19.png)

第一轮选举未果，发起第二轮选举

#### 日志复制

日志复制（Log Replication）的主要目的是确保节点的一致性，在此阶段执行的操作都是为了确保一致性和高可用性。

当 Leader 选举产生后，它开始负责处理客户端的请求。所有的事务（更新操作）请求都必须先由 Leader 处理。日志复制（Log Replication）就是为了确保执行相同的操作序列所做的工作。

在 Raft 中，当接收到客户端的日志（事务请求）后，先把该日志追加到本地的Log中，然后通过heartbeat把该Entry同步给其他Follower，Follower接收到日志后记录日志然后向Leader发送ACK，当Leader收到大多数（n/2+1）Follower的ACK信息后将该日志设置为已提交并追加到本地磁盘中，通知客户端并在下个heartbeat中Leader将通知所有的Follower将该日志存储在自己的本地磁盘中。

![](./2025/02/27/从入门到精通-Nacos最全详解/img_20.png)

简单日志复制

![](./2025/02/27/从入门到精通-Nacos最全详解/img_21.png)

复杂日志复制

### 5.3.7 配置一致性模型 的底层原理

Nacos 配置管理一致性协议分为两个大部分，

*   第一部分是 Server 间一致性协议，一个是 SDK 与 Server 的一致性协议，配置作为分布式系统中非强一致数据，在出现脑裂的时候可用性高于一致性，因此阿里配置中心是采用 AP 一致性协议。

*   第二部分Server 间采用 Raft 协议保证数据一致性，行业大部分产品才用此模式，Nacos 提供此模式，是方便用户本机运行，降低对存储依赖。


#### Server 间的一致性协议

一致性的核心是 Server 与 DB 保持数据一致性，从而保证 Server 数据一致；Server 之间都是对等的。

数据写任何一个 Server，优先持久化，持久化成功后异步通知其他节点到数据库中拉取最新配置值，并且通知写入成功。

![](./2025/02/27/从入门到精通-Nacos最全详解/img_22.png)

Server 间采用 Raft 协议保证数据一致性，行业大部分产品才用此模式。

#### SDK 与 Server 的一致性协议

SDK 与 Server 一致性协议的核心是通过 MD5 值是否一致，如果不一致就拉取最新值。

1.  Nacos 1.X


Nacos 1.X 采用 Http 1.1 短链接模拟长链接，每 30s 发一个心跳跟 Server 对比 SDK 配置 MD5 值是否跟 Server 保持一致，如果一致就 hold 住链接，如果有不一致配置，就把不一致的配置返回，然后 SDK 获取最新配置值。

![](./2025/02/27/从入门到精通-Nacos最全详解/img_23.png)

2.  Nacos 2.X


Nacos 2.x 相比上面 30s 一次的长轮训，升级成长链接模式，配置变更，启动建立长链接，配置变更服务端推送变更配置列表，然后 SDK 拉取配置更新，因此通信效率大幅提升。

5.4 服务注册 的底层原理
--------------

作为一个服务注册中心，服务注册肯定是一个非常重要的功能

所谓的服务注册，就是通过注册中心提供的客户端SDK（或者是控制台）将服务本身的一些元信息，比如ip、端口等信息发送到注册中心服务端

服务端在接收到服务之后，会将服务的信息保存到前面提到的服务注册表中

![](./2025/02/27/从入门到精通-Nacos最全详解/img_24.png)

### 1.x版本的服务注册实现

在Nacos在1.x版本的时候，服务注册是通过Http接口实现的

![](./2025/02/27/从入门到精通-Nacos最全详解/img_25.png)

代码如下

![](./2025/02/27/从入门到精通-Nacos最全详解/img_26.png)

整个逻辑比较简单，因为Nacos服务端本身就是用SpringBoot写的

但是在2.x版本的实现就比较复杂了

### 2.x版本的服务注册实现

1.  通信协议的改变


2.x版本相比于1.x版本最主要的升级就是客户端和服务端通信协议的改变，由1.x版本的Http改成了2.x版本gRPC

gRPC是谷歌公司开发的一个高性能、开源和通用的RPC框架，Java版本的实现底层也是基于Netty来的

之所以改成了gRPC，主要是因为Http请求会频繁创建和销毁连接，白白浪费资源

所以在2.x版本之后，为了提升性能，就将通信协议改成了gRPC

根据官网显示，整体的效果还是很明显，相比于1.x版本，注册性能总体提升至少2倍

虽然通信方式改成了gRPC，但是2.x版本服务端依然保留了Http注册的接口，所以用1.x的Nacos SDK依然可以注册到2.x版本的服务端

2.  具体的实现


Nacos客户端在启动的时候，会通过gRPC跟服务端建立长连接

![](./2025/02/27/从入门到精通-Nacos最全详解/img_27.png)

这个连接会一直存在，之后客户端与服务端所有的通信都是基于这个长连接来的

当客户端发起注册的时候，就会通过这个长连接，将服务实例的信息发送给服务端

服务端拿到服务实例，跟1.x一样，也会存到服务注册表

3.  Redo操作


除了注册之外，当注册的是临时实例时，2.x还会将服务实例信息存储到客户端中的一个缓存中，供Redo操作，也就是保活机制

所谓的Redo操作，其实就是一个补偿机制，本质是个定时任务，默认每3s执行一次

这个定时任务作用是，当客户端与服务端重新建立连接时（因为一些异常原因导致连接断开）

那么之前注册的服务实例肯定还要继续注册服务端（断开连接服务实例就会被剔除服务注册表）

所以这个Redo操作一个很重要的作用就是重连之后的重新注册的作用

> 除了注册之外，比如服务订阅之类的操作也需要Redo操作，当连接重新建立，之前客户端的操作都需要Redo一下

### 总结：

1.x版本是通过Http协议来进行服务注册的

2.x由于客户端与服务端的通信改成了gRPC长连接，所以改成通过gRPC长连接来注册

2.x比1.x多个Redo操作，当注册的服务实例是临时实例是，出现网络异常，连接重新建立之后，客户端需要将服务注册、服务订阅之类的操作进行重做

这里你可能会有个疑问

> 既然2.x有Redo机制保证客户端与服务端通信正常之后重新注册，那么1.x有类似的这种Redo机制么？

当然也会有，接下往下看。

5.5 心跳机制 的底层原理
--------------

心跳机制，也可以被称为保活机制，它的作用就是服务实例告诉注册中心我这个服务实例还活着

![](./2025/02/27/从入门到精通-Nacos最全详解/img_28.png)

在正常情况下，服务关闭了，那么服务会主动向Nacos服务端发送一个服务下线的请求

Nacos服务端在接收到请求之后，会将这个服务实例从服务注册表中剔除

但是对于异常情况下，比如出现网络问题，可能导致这个注册的服务实例无法提供服务，处于不可用状态，也就是不健康

而此时在没有任何机制的情况下，服务端是无法知道这个服务处于不可用状态

所以为了避免这种情况，一些注册中心，就比如Nacos、Eureka，就会用心跳机制来判断这个服务实例是否能正常

在Nacos中，心跳机制仅仅是针对临时实例来说的，临时实例需要靠心跳机制来保活

心跳机制在1.x和2.x版本的实现也是不一样的

### 1.x心跳实现

在1.x中，心跳机制实现是通过客户端和服务端各存在的一个定时任务来完成的

在服务注册时，发现是临时实例，客户端会开启一个5s执行一次的定时任务

![](./2025/02/27/从入门到精通-Nacos最全详解/img_29.png)

这个定时任务会构建一个Http请求，携带这个服务实例的信息，然后发送到服务端

![](./2025/02/27/从入门到精通-Nacos最全详解/img_30.png)

在Nacos服务端也会开启一个定时任务，默认也是5s执行一次，去检查这些服务实例最后一次心跳的时间，也就是客户端最后一次发送Http请求的时间

*   当最后一次心跳时间超过15s，但没有超过30s，会把这服务实例标记成不健康

*   当最后一次心跳超过30s，直接把服务从服务注册表中剔除


![](./2025/02/27/从入门到精通-Nacos最全详解/img_31.png)

这就是1.x版本的心跳机制，本质就是两个定时任务

其实1.x的这个心跳还有一个作用，就是跟上一节说的gRPC时Redo操作的作用是一样的

服务在处理心跳的时候，发现心跳携带这个服务实例的信息在注册表中没有，此时就会添加到服务注册表

所以心跳也有Redo的类似效果

### 底层原理：2.x 版本心跳实现

在2.x版本之后，由于通信协议改成了gRPC，客户端与服务端保持长连接，所以2.x版本之后它是利用这个gRPC长连接本身的心跳来保活

一旦这个连接断开，服务端就会认为这个连接注册的服务实例不可用，之后就会将这个服务实例从服务注册表中提出剔除

除了连接本身的心跳之外，Nacos还有服务端的一个主动检测机制

Nacos服务端也会启动一个定时任务，默认每隔3s执行一次

这个任务会去检查超过20s没有发送请求数据的连接

一旦发现有连接已经超过20s没发送请求，那么就会向这个连接对应的客户端发送一个请求

如果请求不通或者响应失败，此时服务端也会认为与客户端的这个连接异常，从而将这个客户端注册的服务实例从服务注册表中剔除

所以对于2.x版本，主要是两种机制来进行保活：

*   连接本身的心跳机制，断开就直接剔除服务实例

*   Nacos主动检查机制，服务端会对20s没有发送数据的连接进行检查，出现异常时也会主动断开连接，剔除服务实例


### 总结

*   心跳机制仅仅针对临时实例而言

*   1.x心跳机制是通过客户端和服务端两个定时任务来完成的，客户端定时上报心跳信息，服务端定时检查心跳时间，超过15s标记不健康，超过30s直接剔除

*   1.x心跳机制还有类似2.x的Redo作用，服务端发现心跳的服务信息不存在会，会将服务信息添加到注册表，相当于重新注册了

*   2.x是基于gRPC长连接本身的心跳机制和服务端的定时检查机制来的，出现异常直接剔除


5.6 健康检查 的底层原理
--------------

临时实例的健康检查，采用心跳机制

![](./2025/02/27/从入门到精通-Nacos最全详解/img_32.png)

那么问题来了，永久实例的健康检查如何实现？

针对永久实例的情况，Nacos通过一种叫健康检查的机制去判断服务实例是否活着

健康检查跟心跳机制刚好相反，心跳机制是服务实例向服务端发送请求

而所谓的健康检查就是服务端主动向服务实例发送请求，去探测服务实例是否活着

![](./2025/02/27/从入门到精通-Nacos最全详解/img_33.png)

健康检查机制在1.x和2.x的实现机制是一样的

![](./2025/02/27/从入门到精通-Nacos最全详解/img_34.png)

Nacos服务端在会去创建一个健康检查任务，这个任务每次执行时间间隔会在2000~7000毫秒之间

当任务触发的时候，会根据设置的健康检查的方式执行不同的逻辑，目前主要有以下三种方式：

*   TCP

*   HTTP

*   MySQL


TCP的方式就是根据服务实例的ip和端口去判断是否能连接成功，如果连接成功，就认为健康，反之就任务不健康

HTTP的方式就是向服务实例的ip和端口发送一个Http请求，请求路径是需要设置的，如果能正常请求，说明实例健康，反之就不健康

MySQL的方式是一种特殊的检查方式，他可以执行下面这条Sql来判断数据库是不是主库

![](./2025/02/27/从入门到精通-Nacos最全详解/img_35.png)

默认情况下，都是通过TCP的方式来探测服务实例是否还活着

5.7 服务发现 的底层原理
--------------

所谓的服务发现就是指当有服务实例注册成功之后，其它服务可以发现这些服务实例

Nacos提供了两种发现方式：

*   主动查询（pull）

*   服务订阅（push）


主动查询就是指客户端主动向服务端查询需要关注的服务实例，也就是拉（pull）的模式

服务订阅就是指客户端向服务端发送一个订阅服务的请求，当被订阅的服务有信息变动就会主动将服务实例的信息推送给订阅的客户端，本质就是推（push）模式

![](./2025/02/27/从入门到精通-Nacos最全详解/img_36.png)

在我们平时使用时，一般来说都是选择使用订阅的方式，这样一旦有服务实例数据的变动，客户端能够第一时间感知

并且Nacos在整合SpringCloud的时候，默认就是使用订阅的方式

对于这两种服务发现方式，1.x和2.x版本实现也是不一样

服务查询其实两者实现都很简单

1.x整体就是发送Http请求去查询服务实例，2.x只不过是将Http请求换成了gRPC的请求

服务端对于查询的处理过程都是一样的，从服务注册表中查出符合查询条件的服务实例进行返回

不过对于服务订阅，两者的机制就稍微复杂一点

在Nacos客户端，不论是1.x还是2.x都是通过SDK中的`NamingService#subscribe`方法来发起订阅的



![](./2025/02/27/从入门到精通-Nacos最全详解/img_37.png)

当有服务实例数据变动的时，客户端就会回调`EventListener`，就可以拿到最新的服务实例数据了

虽然1.x还是2.x都是同样的方法，但是具体的实现逻辑是不一样的

### 1.x服务订阅实现

在1.x版本的时候，服务订阅的处理逻辑大致会有以下三步：

1.  **第一步，客户端在启动的时候，会去构建一个叫PushReceiver的类**


这个类会去创建一个UDP Socket，端口是随机的



![](./2025/02/27/从入门到精通-Nacos最全详解/img_38.png)

其实通过名字就可以知道这个类的作用，就是通过UDP的方式接收服务端推送的数据的

2.  **第二步，调用**`**NamingService#subscribe**`**发起订阅时，先去服务端查询需要订阅服务的所有实例信息**


之后会将所有服务实例数据存到客户端的一个内部缓存中



![](./2025/02/27/从入门到精通-Nacos最全详解/img_39.png)

并且在查询的时候，会将这个UDP Socket的端口作为一个参数传到服务端

服务端接收到这个UDP端口后，后续就通过这个端口给客户端推送服务实例数据

3.  **第三步，服务器会为这次订阅开启一个不定时执行的任务**


之所以不定时，是因为这个当执行异常的时候，下次执行的时间间隔就会变长，但是最多不超过60s，正常是10s，这个10s是查询服务实例是服务端返回的

这个任务会去从服务端查询订阅的服务实例信息，然后更新内部缓存

这里你可能会有个疑问

既然有了服务变动推送的功能，为什么还要定时去查询更新服务实例信息呢?

其实很简单，那就是因为UDP通信不稳定导致的

虽然有Push，但是由于UDP通信自身的不确定性，有可能会导致客户端接收变动信息失败

所以这里就加了一个定时任务，弥补这种可能性，属于一个兜底的方案。

这就是1.x版本的服务订阅的实现

![](./2025/02/27/从入门到精通-Nacos最全详解/img_40.png)

### 2.x服务订阅的实现

由于2.x版本换成了gRPC长连接的方式，所以2.x版本服务数据变更推送已经完全抛弃了1.x的UDP做法

当有服务实例变动的时候，服务端直接通过这个长连接将服务信息发送给客户端

客户端拿到最新服务实例数据之后的处理方式就跟1.x是一样了

除了处理方式一样，2.x也继承了1.x的其他的东西

比如客户端依然会有服务实例的缓存

定时对比机制也保留了，只不过这个定时对比的机制默认是关闭状态

之所以默认关闭，主要还是因为长连接还是比较稳定的原因

当客户端出现异常，接收不到请求，那么服务端会直接跟客户端断开连接

当恢复正常，由于有Redo操作，所以还是能拿到最新的实例信息的

所以2.x版本的服务订阅功能的实现大致如下图所示

![](./2025/02/27/从入门到精通-Nacos最全详解/img_41.png)

这里还有个细节需要注意

在1.x版本的时候，任何服务都是可以被订阅的

但是在2.x版本中，只支持订阅临时服务，对于永久服务，已经不支持订阅了

### 总结

*   服务查询1.x是通过Http请求；2.x通过gRPC请求

*   服务订阅1.x是通过UDP来推送的；2.x就基于gRPC长连接来实现的

*   1.x和2.x客户端都有服务实例的缓存，也有定时对比机制，只不过1.x会自动开启；2.x提供了一个开关，可以手动选择是否开启，默认不开启


5.8 临时实例和永久实例
-------------

临时实例和永久实例在Nacos中是一个非常非常重要的概念

为什么Nacos要将服务实例分为临时实例和永久实例?

主要还是因为应用场景不同

1.  **临时实例**


*   就比较适合于业务服务，服务下线之后可以不需要在注册中心中查看到

*   临时实例在注册到注册中心之后仅仅只保存在服务端内部一个缓存中，不会持久化到磁盘

*   这个服务端内部的缓存在注册中心届一般被称为服务注册表

*   当服务实例出现异常或者下线之后，就会把这个服务实例从服务注册表中剔除


2.  **永久实例**


*   就比较适合需要运维的服务，这种服务几乎是永久存在的，比如说MySQL、Redis等等

*   永久服务实例不仅仅会存在服务注册表中，同时也会被持久化到磁盘文件中

*   当服务实例出现异常或者下线，Nacos只会将服务实例的健康状态设置为不健康，并不会对将其从服务注册表中剔除


所以从这可以看出Nacos跟你印象中的注册中心不太一样，他不仅仅可以注册平时业务中的实例，还可以注册像MySQL、Redis这个服务实例的信息到注册中心

在SpringCloud环境底下，一般其实都是业务服务，所以默认注册服务实例都是临时实例

当然如果你想改成永久实例，可以通过下面这个配置项来完成

```


spring  
  cloud:  
    nacos:  
      discovery:  
        #ephemeral单词是临时的意思，设置成false，就是永久实例了  
        ephemeral: false  



```

这里还有一个小细节

在1.x版本中，一个服务中可以既有临时实例也有永久实例，服务实例是永久还是临时是由服务实例本身决定的

但是2.x版本中，一个服务中的所有实例要么都是临时的要么都是永久的，是由服务决定的，而不是具体的服务实例

所以在2.x可以说是临时服务和永久服务

![](./2025/02/27/从入门到精通-Nacos最全详解/img_42.png)

为什么2.x把临时还是永久的属性由实例本身决定改成了由服务决定?

其实很简单，你想想，假设对一个MySQL服务来说，它的每个服务实例肯定都是永久的，不会出现一些是永久的，一些是临时的情况吧

所以临时还是永久的属性由服务本身决定其实就更加合理了

**临时实例和永久实例总结：**

*   健康检查机制不同


*   临时实例，使用心跳机制

*   永久实例，server端主动发起探测请求，http,tcp,mysql


*   注册数据一致性方案不同


*   临时实例，使用自研Distro协议，属于AP

*   永久实例，使用jRaft协议，属于CP


*   临时实例存储在内存，永久实例还会持久化到硬盘
    

