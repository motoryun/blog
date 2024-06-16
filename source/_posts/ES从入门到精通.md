---
title: ES从入门到精通
date: 2024-06-16 18:47:07
tags: ElasticSearch入门
categories: 面试
---

ElasticSearch 从入门到工业级使用
=======================

1.1 什么是全文检索
-----------

将非结构化数据中的一部分信息提取出来，重新组织，使其变得有一定结构，然后对此有一定结构的数据进行搜索，从而达到搜索相对较快的目的。这部分从非结构化数据中提取出的然后重新组织的信息，我们称之索引。

例如：字典。字典的拼音表和部首检字表就相当于字典的索引，对每一个字的解释是非结构化的，如果字典没有音节表和部首检字表，在茫茫辞海中找一个字只能顺序扫描。

然而字的某些信息可以提取出来进行结构化处理，比如读音，就比较结构化，分声母和韵母，分别只有几种可以一一列举，于是将读音拿出来按一定的顺序排列，每一项读音都指向此字的详细解释的页数。

我们搜索时按结构化的拼音搜到读音，然后按其指向的页数，便可找到我们的非结构化数据——也即对字的解释。 这种先建立索引，再对索引进行搜索的过程就叫全文检索(Full-text Search)。

虽然创建索引的过程也是非常耗时的，但是索引一旦创建就可以多次使用，全文检索主要处理的是查询，所以耗时间创建索引是值得的。
![](./2024/06/16/ES从入门到精通/1.png)
比如使用全文检索，所搜索“生化机”
![](./2024/06/16/ES从入门到精通/2.png)
（有可能是手抖打错了，本来是生化危机），但是期望需要出来右侧的 4条 记录

有 4条 数据将每条数据进行词条拆分。

如“生化危机电影”拆成：生化、危机、电影 关键词（拆分结果与策略算法有关）每个关键词将对应包含此关键词的数据 ID搜索的时候，直接匹配这些关键词，就能拿到包含关键词的数据这个过程就叫做全文检索。

而词条拆分和词条对应的 ID 这个就是倒排索引的的基本原理

**对比数据库的缺陷**

mysql如果没有索引的情况下，共有100万条,按照之前的思路,其实就要扫描100万次，而且每次扫描,都需要匹配那个文本所有的字符，确认是否包含搜索的关键词，而且还不能将搜索词拆解开来进行检索
![](./2024/06/16/ES从入门到精通/3.png)
**全文检索使用场景**

*   维基百科，类似百度百科，牙膏，牙膏的维基百科，全文检索，高亮，搜索推荐

*   The Guardian（国外新闻网站），类似搜狐新闻，用户行为日志（点击，浏览，收藏，评论）+社交网络数据（对某某新闻的相关看法），数据分析，给到每篇新闻文章的作者，让他知道他的文章的公众反馈（好，坏，热门，垃圾，鄙视，崇拜）

*   Stack Overflow（国外的程序异常讨论论坛），IT问题，程序的报错，提交上去，有人会跟你讨论和回答，全文检索，搜索相关问题和答案，程序报错了，就会将报错信息粘贴到里面去，搜索有没有对应的答案

*   GitHub（开源代码管理），搜索上千亿行代码（5）电商网站，检索商品

*   日志数据分析，logstash采集日志，ES进行复杂的数据分析（ELK技术，elasticsearch+logstash+kibana）

*   商品价格监控网站，用户设定某商品的价格阈值，当低于该阈值的时候，发送通知消息给用户，比如说订阅牙膏的监控，如果高露洁牙膏的家庭套装低于50块钱，就通知我，我就去买

*   BI系统，商业智能，Business Intelligence。比如说有个大型商场集团，BI，分析一下某某区域最近3年的用户消费金额的趋势以及用户群体的组成构成，产出相关的数张报表，\*\*区，最近3年，每年消费金额呈现100%的增长，而且用户群体85%是高级白领，开一个新商场。ES执行数据分析和挖掘，Kibana进行数据可视化


1.2 ES简介
--------

Elaticsearch，简称为es， es是一个开源的高扩展的分布式全文检索引擎，它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理PB级别的数据。

es也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。

Elasticsearch是面向文档(document oriented)的，这意味着它可以存储整个对象或文档(document)。然而它不仅仅是存储，还会索引(index)每个文档的内容使之可以被搜索。

在Elasticsearch中，你可以对文档（而非成行成列的数据）进行索引、搜索、排序、过滤。Elasticsearch比传统关系型数据库如下：

```


Relational DB -> Databases -> Tables -> Rows -> Columns  
Elasticsearch -> Indices   -> Types  -> Documents -> Fields  



```

Elasticsearch提供多种语言支持，其中Java的客户端为 Java REST Client 。

而它又分成两种：高级和低级的。高级包含更多的功能，如果把高级比作MyBatis的话，那么低级就相当于JDBC，是基于Netty和Server通讯相关。

高级的 Client类似Mybatis是对于Low Level的封装。
![](./2024/06/16/ES从入门到精通/4.png)
1.3 ES基本概念
----------

1.  **索引库**


ElasticSearch将它的数据存储在一个或多个索引（index）中。

用SQL领域的术语来类比，索引就像数据库，可以向索引写入文档或者从索引中读取文档，并通过ElasticSearch内部使用Lucene将数据写入索引或从索引中检索数据。

Elastic Search使用倒排索引（Inverted Index）来做快速的全文搜索，这点与数据库不同，一般数据库 的索引，用B+Tree来实现。

| **Relational DB** | **Databases** | **Tables** | **表结构** | **Rows** | **Columns** |
| --- | --- | --- | --- | --- | --- |
|ElasticSearch| Indices| Types| 映射mapping| Documents| Fields 字段|

索引库就是存储索引的保存在磁盘上的一系列的文件。里面存储了建立好的索引信息以及文档对象。一个索引库相当于数据库中的一张表。
![](./2024/06/16/ES从入门到精通/5.png)
2.  **document对象**


获取原始内容的目的是为了索引，在索引前需要将原始内容创建成文档（Document），文档中包括一个一个的域（Field），域中存储内容。

每个文档都有一个唯一的编号，就是文档id。

document对象相当于表中的一条记录。

文档（document）是ElasticSearch中的主要实体。

对所有使用ElasticSearch的案例来说，他们最终都可以归结为对文档的搜索。

文档由字段构成。

![](./2024/06/16/ES从入门到精通/6.png)
3.  **field对象**


如果我们把document看做是数据库中一条记录的话，field相当于是记录中的字段。field是索引库中存储数据的最小单位。field的数据类型大致可以分为数值类型和文本类型，一般需要查询的字段都是文本类型的，field的还有如下属性：

*   是否分词：是否对域的内容进行分词处理。前提是我们要对域的内容进行查询。

*   是否索引：将Field分析后的词或整个Field值进行索引，只有索引方可搜索到。比如：商品名称、商品简介分析后进行索引，订单号、身份证号不用分词但也要索引，这些将来都要作为查询条件。

*   是否存储：将Field值存储在文档中，存储在文档中的Field才可以从Document中获取。比如：商品名称、订单号，凡是将来要从Document中获取的Field都要存储。


4.  **term对象**


从文档对象中拆分出来的每个单词叫做一个Term，不同的域中拆分出来的相同的单词是不同的term。

term中包含两部分一部分是文档的域名，另一部分是单词的内容。

term是创建索引的关键词对象。

8.  **类型（type）**


每个文档都有与之对应的类型（type）定义。

这允许用户在一个索引中存储多种文档类型，并为不同文 档提供类型提供不同的映射。 type的版本迭代

*   5.x及以前版本一个index有一个或者多个type

*   6.X版本一个index只有一个index

*   7.X版本移除了type，type相关的所有内容全部变成Deprecated，为了兼容升级和过渡，所有的7.X版本es数据写入后type字段都默认被置为\_doc

*   8.X版本完全废弃type


9.  **映射（mapping）**


mapping是处理数据的方式和规则方面做一些限制，如某个字段的数据类型、默认值、分析器、是否被索引等等，这些都是映射里面可以设置的，其它就是处理es里面数据的一些使用规则设置也叫做映射，按着最优规则处理数据对性能提高很大，因此才需要建立映射，并且需要思考如何建立映射才能对性能更好。

10.  **分片（shards）**


代表索引分片，es可以把一个完整的索引分成多个分片，这样的好处是可以把一个大的索引拆分成多个，分布到不同的节点上。构成分布式搜索。分片的数量只能在索引创建前指定，并且索引创建后不能更改。

5.X默认不能通过配置文件定义分片 ES默认5:1 5个主分片，每个分片，1个副本分片

11.  **副本（replicas）**


代表索引副本，es可以设置多个索引的副本，副本的作用：

*   提高系统的容错性，当个某个节点某个分片损坏或丢失时可以从副本中恢复。

*   是提高es的查询效率，es会自动对搜索请求进行负载均衡。


12.  **集群（cluster）**


代表一个集群，集群中有多个节点（node），其中有一个为主节点，这个主节点是可以通过选举产生的，主从节点是对于集群内部来说的。es的一个概念就是去中心化，字面上理解就是无中心节点，这是对于集群外部来说的，因为从外部来看es集群，在逻辑上是个整体，你与任何一个节点的通信和与整个es集群通信是等价的。

2 安装和DSL的使用
===========

2.1 安装ES
--------

使用docker安装单点Elasticsearch，步骤如下：

```


docker network create elastic  
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.15.2  
docker run -di --name es --net elastic -p 192.168.56.181:9200:9200 -p 192.168.56.181:9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.15.2  



```

9200端口(Web管理平台端口)  9300(服务默认端口)

浏览器输入地址访问：`http://192.168.56.181:9200/`

![](./2024/06/16/ES从入门到精通/7.png)
3.  **系统参数配置**


es发现重启启动失败了，这时什么原因呢？

这与我们刚才修改的配置有关，因为elasticsearch在启动的时候会进行一些检查，比如最多打开的文件的个数以及虚拟内存区域数量等等，如果你放开了此配置，意味着需要打开更多的文件以及虚拟内存，所以我们还需要系统调优 修改vi /etc/security/limits.conf ，追加内容 (nofile是单个进程允许打开的最大文件个数 soft nofile 是软限制 hard nofile是硬限制 )

```


\* soft nofile 65536  
\* hard nofile 65536  



```

修改vi /etc/sysctl.conf，追加内容 (限制一个进程可以拥有的VMA(虚拟内存区域)的数量 )

```


vm.max\_map\_count\=655360  



```

执行下面命令 修改内核参数马上生效

```


sysctl \-p  



```

重新启动虚拟机，再次启动容器，发现已经可以启动并远程访问

```


reboot  



```

2.2 安装Kibana
------------

Kibana是一个针对Elasticsearch的开源分析及可视化平台，用来搜索、查看交互存储在Elasticsearch索引中的数据。使用Kibana，可以通过各种图表进行高级数据分析及展示。

Kibana让海量数据更容易理解。它操作简单，基于浏览器的用户界面可以快速创建仪表板（dashboard）实时显示Elasticsearch查询动态。

设置Kibana非常简单。

无需编码或者额外的基础架构，几分钟内就可以完成Kibana安装并启动Elasticsearch索引监测。

Query DSL是一个Java开源框架用于构建类型安全的SQL查询语句。

采用API代替传统的拼接字符串来构造查询语句。

目前QueryDSL支持的平台包括JPA，JDO，SQL，Java Collections，RDF，Lucene，Hibernate Search。elasticsearch提供了一整套基于JSON的DSL语言来定义查询。

```


docker pull docker.elastic.co/kibana/kibana:7.15.2  
docker run -di --name kb --net elastic -p 192.168.56.181:5601:5601 -e "ELASTICSEARCH\_HOSTS=http://192.168.56.181:9200" docker.elastic.co/kibana/kibana:7.15.2  



```

1.  拉取kibana镜像


```


docker pull docker.elastic.co/kibana/kibana:7.15.2  



```

2.  安装kibana容器


```


docker run -di --name kb --net elastic -p 192.168.56.181:5601:5601 -e "ELASTICSEARCH\_HOSTS=http://192.168.56.181:9200" docker.elastic.co/kibana/kibana:7.15.2  



```

3.  修改配置文件


```


docker exec -it kb /bin/bash  
vi config/kibana.yml  



```

好像不改也可以，因为上面docker启动有了ES地址

```


#修改elasticsarch.hosts: \[ "http://elasticsearch:9200" \]，如下：  
elasticsearch.hosts: \[ "http://192.168.56.181:9200" \]  



```

2.3通过脚本一键启动ES
-------------

通过提供的脚本，和配置文件，可以一键启动ES

这个非常的容易，非常的轻量级。 具体请参见视频。

![](./2024/06/16/ES从入门到精通/8.png)
启动完之后的效果

![](./2024/06/16/ES从入门到精通/9.png)
接下来，可以访问es

http://cdh1:9200

![](./2024/06/16/ES从入门到精通/10.png)
接下来，可以访问 Kibana

```


默认的地址     http://cdh1:5601               
  
的一键环境地址     http://cdh1:5601             


```

![](./2024/06/16/ES从入门到精通/11.png)
2.4使用 DSL 操作ES
--------------

在 Kibana的开发工具界面，可以执行 DSL 去进行ES的查询。

![](./2024/06/16/ES从入门到精通/12.png)
es开发，常常需要用到DSL语法去定义好 es的查询语句。

就像 myql开发，需要提前定义好 sql语句，并进行sql 的执行和测试一样。

### 2.4.1DSL 定义基本介绍

##### DSL（Domain Specific Language），一种特定领域的查询语言，用于构建复杂的查询和聚合操作。

在Elasticsearch中，可用DSL语法来定义查询和过滤条件，以及执行聚合操作。

DSL语法 具有JSON格式，因此它非常易于阅读和编写。

### 2.4.2DSL 定义语法说明

###### （1）关键字(Keywords)

*   DSL通常会定义一组关键字，这些关键字具有特殊含义，并在DSL中起到关键作用。关键字通常不能用作标识符或变量名。

*   示例：在一个简单的数学表达式DSL中，可能会定义关键字如"add"、"subtract"等来表示加法和减法操作。


###### （2）标识符(Identifiers)

*   标识符是用来表示变量名、函数名或其他用户定义的名称。它们需要遵循特定的命名规则，如大小写敏感、不包含特殊字符等。

*   示例：在一个配置文件DSL中，可以使用标识符来表示不同的配置项，如"username"、"password"等。


###### （3）表达式(Expressions)

*   表达式是DSL中最基本的构建块，用于计算或产生某个值。表达式可以包括变量、常量、运算符和函数调用。

*   示例：在一个数学表达式DSL中，可以将"2 + 3"作为一个表达式，计算结果为5。


###### （4）运算符(Operators)

*   运算符用于执行各种操作，例如算术运算、逻辑运算、比较运算等。DSL中的运算符根据所涉及的领域和需求而定。

*   示例：在一个布尔表达式DSL中，可以定义逻辑运算符如"and"、"or"用于连接多个条件。


###### （5）函数调用(Function Calls)

*   DSL可以支持函数调用，允许用户使用预定义或自定义的函数来完成特定的任务。函数调用通常由函数名称和传递给函数的参数组成。

*   示例：在一个日期处理DSL中，可以定义函数"formatDate(date, format)"，其中"date"是日期值，"format"是日期格式字符串。


###### （6）控制流(Control Flow)

*   控制流语句用于控制程序的执行流程，例如条件语句(if-else)和循环语句(while、for)等。DSL可以支持特定的控制流语句来满足领域特定需求。

*   示例：在一个工作流程DSL中，可以使用条件语句来判断某个条件是否满足并执行相应的操作。


###### （7）注释(Comments)

*   注释用于向DSL代码添加说明性文本，以便开发人员理解和维护代码。注释通常不会被编译或执行，仅用于阅读目的。

*   示例：在DSL中，可以使用双斜杠(//)或特定的注释标记来添加注释，如：“// 这是一个示例注释”。


### 2.4.3DSL常见语法

> 文章实在太长， 这里省略了500字+， 省略的内容请参见的免费电子书 PDF版本《ES学习圣经：从0到1, 精通 ElasticSearch 工业级使用 》

3 ES 的分词器
=========

3.1 倒排索引
--------

![](./2024/06/16/ES从入门到精通/13.png)
如上图

*   左边的是正排索引，通过文档的id如查找文档的内容

*   右边的是倒排索引，通过单词统计次数以及文档的位置，


如Elasticsearch出现的次数为3，在id=1，id=2，id=3都出现过，且位置分别为1，0，0

3.2 默认分词器
---------

默认分词器对于英文分词的效果如下

```


POST /\_analyze  
{  
  "text": "You can use Elasticsearch to store, search, and manage data",  
  "analyzer": "standard"  
}  
  
  



```

结果如下：
![](./2024/06/16/ES从入门到精通/14.png)
```


POST /\_analyze  
{  
  "text": "中华人民共和国人民大会堂",  
  "analyzer": "standard"  
}  



```

![](./2024/06/16/ES从入门到精通/15.png)
3.3 中文分词
--------

中文分词是中文文本处理的一个基础步骤，也是中文人机自然语言交互的基础模块。

不同于英文的是，中文句子中没有词的界限，因此在进行中文自然语言处理时，通常需要先进行分词，

分词效果将直接影响词性、句法树等模块的效果。

当然分词只是一个工具，场景不同，要求也不同。

部分分词工具如下：

*   中科院计算所NLPIR http://ictclas.nlpir.org/nlpir/

*   ansj分词器 https://github.com/NLPchina/ansj\_seg

*   哈工大的LTP https://github.com/HIT-SCIR/ltp

*   清华大学THULAC https://github.com/thunlp/THULAC

*   斯坦福分词器 https://nlp.stanford.edu/software/segmenter.shtml

*   Hanlp分词器 https://github.com/hankcs/HanLP

*   结巴分词 https://github.com/yanyiwu/cppjieba

*   KCWS分词器(字嵌入+Bi-LSTM+CRF) https://github.com/koth/kcws

*   ZPar https://github.com/frcchang/zpar/releases

*   IKAnalyzer https://github.com/wks/ik-analyzer


3.4 IK分词器
---------

IK分词器下载地址https://github.com/medcl/elasticsearch-analysis-ik/releases 将ik分词器上传到服务器上，然后解压，并改名字为ik

```


mkdir ~/ik  
mv elasticsearch-analysis-ik-7.15.2.zip ~/ik  
unzip elasticsearch-analysis-ik-7.15.2.zip  



```

将ik目录拷贝到docker容器的plugins目录下

```


docker cp ./ik es:/usr/share/elasticsearch/plugins  



```

> 特别说明，的一键启动版本，已经默认戴上了IK分词器

IKAnalyzer是一个开源的，基于java语言开发的轻量级的中文分词工具包，IK分词器分为两种模式

1.  **ik\_max\_word：会将文本做最细粒度的拆分**


比如会将“中华人民共和国人民大会堂”拆分为“中华人民共和国、中华人民、中华、华人、人民共和国、人民、共和国、大会堂、大会、会堂等词语。

```


POST /\_analyze  
{  
"text":"中华人民共和国人民大会堂",  
"analyzer":"ik\_max\_word"  
}  



```

![](./2024/06/16/ES从入门到精通/16.png)
2.  **ik\_smart：会做最粗粒度的拆分**


比如会将“中华人民共和国人民大会堂”拆分为中华人民共和国、人民大会堂。

```


POST /\_analyze  
{  
  "text": "中华人民共和国人民大会堂",  
  "analyzer": "ik\_smart"  
}  



```

![](./2024/06/16/ES从入门到精通/17.png)
3.5 自定义扩展字典
-----------

IK分词器的两种模式的最佳实践是：索引时用ik\_max\_word，搜索时用ik\_smart，索引时最大化的将文章内容分词，搜索时更精确的搜索到想要的结果。

举个例子：用户输入“华为手机”搜索，此时应该搜索出“华为手机”的商品，而不是“华为”和“手机”这两个词，这样会搜索出华为其它的商品，

此时使用ik\_smart和ik\_max\_word都会将华为手机拆分为华为和手机两个词，那些只包括“华为”这个词的信息也被搜索出来了，我的目标是搜索只包含华为手机这个词的信息，这没有满足我的目标。

ik\_smart默认情况下分词“华为手机”，依然会分成两个词“华为”和“手机”，这时需要使用自定义扩展字典

1.  **进入es**


```


docker exec -it es /bin/bash  



```

2.  **增加自定义字典文件**


如果容器编辑乱码，可以在宿主机编辑，然后拷贝到容器中

```


#进入ik配置目录  
cd plugins/ik/config/  
vi new\_word.dic  



```

内容如下：

```


老铁  
王者荣耀  
洪荒之力  
共有产权房  
一带一路  
java日知录  
华为手机  



```

3.  **修改配置文件**


```


vi IKAnalyzer.cfg.xml  



```

内容如下：

```


<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd"\>  
<properties>  
        <comment>IK Analyzer 扩展配置</comment>  
        <!--用户可以在这里配置自己的扩展字典 -->  
        <entry key="ext\_dict"\>new\_word.dic</entry>  
         <!--用户可以在这里配置自己的扩展停止词字典-->  
        <entry key="ext\_stopwords"\></entry>  
        <!--用户可以在这里配置远程扩展字典 -->  
        <!-- <entry key="remote\_ext\_dict"\>words\_location</entry> -->  
        <!--用户可以在这里配置远程扩展停止词字典-->  
        <!-- <entry key="remote\_ext\_stopwords"\>words\_location</entry> -->  
</properties>  



```

4.  **拷贝到宿主机**


```


docker cp es:/usr/share/elasticsearch/plugins/elasticsearch-analysis-ik/config ~/ik  



```

5.  **重启**


```


docker restart es  



```

4 高级的DSL 查询
===========

前面介绍了基础的DSL查询，接下来，介绍一下高级的DSL 查询

ES提供基于DSL(Domain Specific Language)的索引查询模式，

DSL查询基于JSON定义查询

Wikipedia对于DSL的定义"**为了解决某一类任务而专门设计的计算机语言"**

大师Martin Fowler对于DSL定义“**DSL 通过在表达能力上做的妥协换取在某一领域内的高效**”

我们在使用ES的时候，避免不了使用DSL语句去查询，就像使用关系型数据库的时候要学会SQL语法一样。

如果我们学习好了DSL语法的使用，那么在日后使用和使用Java Client调用时候也会变得非常简单。

![](./2024/06/16/ES从入门到精通/18.png)
> Elasticsearch provides a full Query DSL (Domain Specific Language) based on JSON to define queries. Think of the Query DSL as an AST (Abstract Syntax Tree) of queries

4.1 管理索引
--------

查看所有的索引

```


GET \_cat/indices?v  



```

![](./2024/06/16/ES从入门到精通/19.png)
1.  **删除某个索引**


```


DELETE /skuinfo  



```

2.  **新增索引**


```


PUT /user  



```

3.  **创建映射**


```


PUT /user/\_mapping  
{  
  "properties": {  
    "name":{  
      "type": "text",  
      "analyzer": "ik\_smart",  
      "search\_analyzer": "ik\_smart"  
    },  
    "city":{  
      "type": "text",  
      "analyzer": "ik\_smart",  
      "search\_analyzer": "ik\_smart"  
    },  
    "age":{  
      "type": "long"  
    },  
    "description":{  
      "type": "text",  
      "analyzer": "ik\_smart",  
      "search\_analyzer": "ik\_smart"  
    }  
  }  
}  



```

4.  **新增文档数据**


```


PUT /user/\_doc/1  
{  
  "name":"李四",  
  "age":22,  
  "city":"深圳",  
  "description":"李四来自湖北武汉！"  
}  
  



```

再增加几条记录：

```


#新增文档数据 id=2  
PUT /user/\_doc/2  
{  
  "name":"王五",  
  "age":35,  
  "city":"深圳",  
  "description":"王五家住在深圳！"  
}  
  
#新增文档数据 id=3  
PUT /user/\_doc/3  
{  
  "name":"张三",  
  "age":19,  
  "city":"深圳",  
  "description":"在深圳打工，来自湖北武汉"  
}  
  
#新增文档数据 id=4  
PUT /user/\_doc/4  
{  
  "name":"张三丰",  
  "age":66,  
  "city":"武汉",  
  "description":"在武汉读书，家在武汉！"  
}  
  
#新增文档数据 id=5  
PUT /user/\_doc/5  
{  
  "name":"赵子龙",  
  "age":77,  
  "city":"广州",  
  "description":"赵子龙来自深圳宝安，但是在广州工作！",  
  "address":"广东省茂名市"  
}  
  
#新增文档数据 id=6  
PUT /user/\_doc/6  
{  
  "name":"赵毅",  
  "age":55,  
  "city":"广州",  
  "description":"赵毅来自广州白云区，从事电子商务8年！"  
}  
  
#新增文档数据 id=7  
PUT /user/\_doc/7  
{  
  "name":"赵哈哈",  
  "age":57,  
  "city":"武汉",  
  "description":"武汉赵哈哈，在深圳打工已有半年了，月薪7500！"  
}  



```

5.  **修改数据**


**（1）操作1**

更新数据可以使用之前的增加操作,这种操作会将整个数据替换掉，代码如下：

```


#更新数据,id=4  
PUT /user/\_doc/4  
{  
  "name":"张三丰",  
  "description":"在武汉读书，家在武汉！在深圳工作！"  
}  



```

使用GET命令查看：

```


#根据ID查询  
GET /user/\_doc/4  



```

**（2）操作2**

我们先使用下面命令恢复数据：

```


#恢复文档数据 id=4  
PUT /user/\_doc/4  
{  
  "name":"张三丰",  
  "age":66,  
  "city":"武汉",  
  "description":"在武汉读书，家在武汉！"  
}  



```

使用POST更新某个列的数据

```


#使用POST更新某个域的数据  
POST /user/\_doc/4  
{  
  "doc":{  
    "name":"张三丰",  
    "description":"在武汉读书，家在武汉！在深圳工作！"  
  }  
}  



```

6.  **删除Document**


```


#删除数据  
DELETE user/userinfo/7  



```

4.2 数据查询
--------

1.  **查询所有数据**


```


#查询所有  
GET /user/\_search  



```

2.  **根据ID查询**


```


#根据ID查询  
GET /user/\_doc/2  



```

3.  **Sort排序**


```


#搜索排序  
GET /user/\_search  
{  
  "query":{  
    "match\_all": {}  
  },  
  "sort":{  
    "age":{  
      "order":"desc"  
    }  
  }  
}  



```

4.  **分页**


```


#分页实现  
GET /user/\_search  
{  
  "query":{  
    "match\_all": {}  
  },  
  "sort":{  
    "age":{  
      "order":"desc"  
    }  
  },  
  "from": 0,  
  "size": 2  
}  



```

*   from:从下N的记录开始查询

*   size:每页显示条数


4.3 查询模式
--------

1.  **term查询**


term主要用于分词精确匹配，如字符串、数值、日期等（不适合情况：1.列中除英文字符外有其它值 2.字符串值中有冒号或中文 3.系统自带属性如\_version）

```


#查询-term  
GET \_search  
{  
  "query":{  
    "term":{  
      "city":"武汉"  
    }  
  }  
}  



```

2.  **terms 查询**


terms 跟 term 有点类似，但 terms 允许指定多个匹配条件。 如果某个字段指定了多个值，那么文档需要一起去做匹配 。

```


#查询-terms 允许多个Term  
GET \_search  
{  
  "query":{  
    "terms":{  
      "city":  
        \[  
          "武汉",  
          "广州"  
        \]  
    }  
  }  
}  



```

3.  **match查询**


```


GET \_search  
{  
  "query": {  
    "match": {  
      "city": "广州武汉"  
    }  
  }  
}  



```

4.  **query\_string查询**


```


GET \_search  
{  
  "query": {  
    "query\_string": {  
      "default\_field": "city",  
      "query": "广州武汉"  
    }  
  }  
}  



```

5.  **range 查询**


range过滤允许我们按照指定范围查找一批数据。例如我们查询年龄范围

```


#-range 范围过滤  
#gt表示> gte表示=>  
#lt表示< lte表示<=  
GET \_search  
{  
  "query":{  
    "range": {  
      "age": {  
        "gte": 30,  
        "lte": 57  
      }  
    }  
  }  
}  



```

6.  **exists**


exists 过滤可以用于查找拥有某个域的数据

```


#搜索 exists：是指包含某个域的数据检索  
GET \_search  
{  
  "query": {  
    "exists":{  
      "field":"address"  
    }  
  }  
}  



```

7.  **bool 查询**


bool 可以用来合并多个条件查询结果的布尔逻辑，它包含以下操作符：

*   must : 多个查询条件的完全匹配,相当于 and。

*   must\_not : 多个查询条件的相反匹配，相当于 not。

*   should : 至少有一个查询条件匹配, 相当于 or。


这些参数可以分别继承一个过滤条件或者一个过滤条件的数组：

```


#过滤搜索 bool   
#must : 多个查询条件的完全匹配,相当于 and。  
#must\_not : 多个查询条件的相反匹配，相当于 not。  
#should : 至少有一个查询条件匹配, 相当于 or。  
GET \_search  
{  
  "query": {  
    "bool": {  
      "must": \[  
        {  
          "term": {  
            "city": {  
              "value": "深圳"  
            }  
          }  
        },  
        {  
          "range":{  
            "age":{  
              "gte":20,  
              "lte":99  
            }  
          }  
        }  
      \]  
    }  
  }  
}  



```

8.  **match\_all 查询**


可以查询到所有文档，是没有查询条件下的默认语句。

```


#查询所有 match\_all  
GET \_search  
{  
  "query": {  
    "match\_all": {}  
  }  
}  



```

9.  **match 查询**


match查询是一个标准查询，不管你需要全文本查询还是精确查询基本上都要用到它。如果你使用 match 查询一个全文本字段，它会在真正查询之前用分析器先分析match一下查询字符：

```


#字符串匹配  
GET \_search  
{  
  "query": {  
    "match": {  
      "description": "武汉广州"  
    }  
  }  
}  



```

10.  **prefix 查询**


以什么字符开头的，可以更简单地用 prefix ,例如查询所有以张开始的用户描述

```


#前缀匹配 prefix  
GET \_search  
{  
  "query": {  
    "prefix": {  
      "name": {  
        "value": "赵"  
      }  
    }  
  }  
}  



```

10.  **multi\_match 查询**


multi\_match查询允许你做match查询的基础上同时搜索多个字段，在多个字段中同时查一个

```


#多个域匹配搜索  
GET \_search  
{  
  "query": {  
    "multi\_match": {  
      "query": "深圳",  
      "fields": \[  
        "city",  
        "description"  
      \]  
    }  
  }  
}  



```

11.  **filter**


因为过滤可以使用缓存，同时不计算分数，通常的规则是，使用查询（query）语句来进行 全文 搜索或者其它任何需要影响 相关性得分 的搜索。除此以外的情况都使用过滤（filters)

```


GET /user/\_search  
{  
  "query": {  
    "bool": {  
      "filter": {  
         "range":{  
          "age":{  
            "gte":25,  
            "lte": 80  
          }  
        }  
      }  
    }  
  }  
}  



```

完整DSL案例代码如下：

```


  
GET \_cat/health?v  
  
GET \_cat/nodes?v  
  
GET \_cat/indices?v  
  
DELETE /user  
  
PUT /user  
  
  
PUT /user/\_mapping  
{  
  "properties": {  
    "name":{  
      "type": "text",  
      "analyzer": "ik\_smart",  
      "search\_analyzer": "ik\_smart"  
    },  
    "city":{  
      "type": "text",  
      "analyzer": "ik\_smart",  
      "search\_analyzer": "ik\_smart"  
    },  
    "age":{  
      "type": "long"  
    },  
    "description":{  
      "type": "text",  
      "analyzer": "ik\_smart",  
      "search\_analyzer": "ik\_smart"  
    }  
  }  
}  
  
PUT /user/\_doc/1  
{  
  "name":"李四",  
  "age":22,  
  "city":"深圳",  
  "description":"李四来自湖北武汉！"  
}  
  
  
#新增文档数据 id=2  
PUT /user/\_doc/2  
{  
  "name":"王五",  
  "age":35,  
  "city":"深圳",  
  "description":"王五家住在深圳！"  
}  
  
#新增文档数据 id=3  
PUT /user/\_doc/3  
{  
  "name":"张三",  
  "age":19,  
  "city":"深圳",  
  "description":"在深圳打工，来自湖北武汉"  
}  
  
#新增文档数据 id=4  
PUT /user/\_doc/4  
{  
  "name":"张三丰",  
  "age":66,  
  "city":"武汉",  
  "description":"在武汉读书，家在武汉！"  
}  
  
#新增文档数据 id=5  
PUT /user/\_doc/5  
{  
  "name":"赵子龙",  
  "age":77,  
  "city":"广州",  
  "description":"赵子龙来自深圳宝安，但是在广州工作！",  
  "address":"广东省茂名市"  
}  
  
#新增文档数据 id=6  
PUT /user/\_doc/6  
{  
  "name":"赵毅",  
  "age":55,  
  "city":"广州",  
  "description":"赵毅来自广州白云区，从事电子商务8年！"  
}  
  
#新增文档数据 id=7  
PUT /user/\_doc/7  
{  
  "name":"赵哈哈",  
  "age":57,  
  "city":"武汉",  
  "description":"武汉赵哈哈，在深圳打工已有半年了，月薪7500！"  
}  
  
  
GET /user/\_doc/4  
  
  
#更新数据,id=4  
PUT /user/\_doc/4  
{  
  "name":"张三丰",  
  "description":"在武汉读书，家在武汉！在深圳工作！"  
}  
  
PUT /user/\_doc/4  
{  
  "name":"张三丰",  
  "age":66,  
  "city":"武汉",  
  "description":"在武汉读书，家在武汉！"  
}  
  
#使用POST更新某个域的数据  
POST /user/\_doc/4  
{  
  "doc":{  
    "name":"张三丰",  
    "description":"在武汉读书，家在武汉！在深圳工作！"  
  }  
}  
  
GET /user/\_search  
  
GET /user/\_doc/2  
  
#搜索排序  
GET /user/\_search  
{  
  "query":{  
    "match\_all": {}  
  },  
  "sort":{  
    "age":{  
      "order":"desc"  
    }  
  }  
}  
  
#分页实现  
GET /user/\_search  
{  
  "query":{  
    "match\_all": {}  
  },  
  "sort":{  
    "age":{  
      "order":"desc"  
    }  
  },  
  "from": 0,  
  "size": 2  
}  
  
#查询-term  
GET \_search  
{  
  "query":{  
    "term":{  
      "city":"武汉"  
    }  
  }  
}  
  
#查询-terms 允许多个Term  
GET \_search  
{  
  "query":{  
    "terms":{  
      "city":  
        \[  
          "武汉",  
          "广州"  
        \]  
    }  
  }  
}  
  
GET \_search  
{  
  "query": {  
    "match": {  
      "city": "广州武汉"  
    }  
  }  
}    
  
GET \_search  
{  
  "query": {  
    "query\_string": {  
      "default\_field": "city",  
      "query": "广州武汉"  
    }  
  }  
}  
  
#-range 范围过滤  
#gt表示> gte表示=>  
#lt表示< lte表示<=  
GET \_search  
{  
  "query":{  
    "range": {  
      "age": {  
        "gte": 30,  
        "lte": 57  
      }  
    }  
  }  
}  
  
#搜索 exists：是指包含某个域的数据检索  
GET \_search  
{  
  "query": {  
    "exists":{  
      "field":"address"  
    }  
  }  
}  
  
GET /user/\_search  
#过滤搜索 bool   
#must : 多个查询条件的完全匹配,相当于 and。  
#must\_not : 多个查询条件的相反匹配，相当于 not。  
#should : 至少有一个查询条件匹配, 相当于 or。  
GET \_search  
{  
  "query": {  
    "bool": {  
      "must": \[  
        {  
          "term": {  
            "city": {  
              "value": "深圳"  
            }  
          }  
        },  
        {  
          "range":{  
            "age":{  
              "gte":20,  
              "lte":99  
            }  
          }  
        }  
      \]  
    }  
  }  
}  
  
#查询所有 match\_all  
GET \_search  
{  
  "query": {  
    "match\_all": {}  
  }  
}  
  
#字符串匹配  
GET \_search  
{  
  "query": {  
    "match": {  
      "description": "武汉广州"  
    }  
  }  
}  
  
#前缀匹配 prefix  
GET \_search  
{  
  "query": {  
    "prefix": {  
      "name": {  
        "value": "赵"  
      }  
    }  
  }  
}  
  
#多个域匹配搜索  
GET \_search  
{  
  "query": {  
    "multi\_match": {  
      "query": "深圳",  
      "fields": \[  
        "city",  
        "description"  
      \]  
    }  
  }  
}  
  
  
GET /user/\_search  
{  
  "query": {  
    "bool": {  
      "filter": {  
         "range":{  
          "age":{  
            "gte":25,  
            "lte": 80  
          }  
        }  
      }  
    }  
  }  
}  



```

4.4 使用别名
--------

在mysql中 我们经常遇到产品修改需求 我们可能会在原有数据库表基础上 对字段 索引 类型进行修改比如 增加一个字段 添加一个字段的索引 又或者修改某个字段的类型，这一切都看起来这么自然 不过在ES这里却是行不通的 ES的mapping一旦设置了之后，可以改，但是改了没有用，因为ES默认是对所有字段进行索引 如果你修改了mapping 那么已经索引过的数据就必须全部重新索引一遍 , ES没有提供这个机制, 只能利用别名手工刷数据，

1.  **添加索引别名**


```


PUT article1/\_alias/article  
{  
"acknowledged" : true  
}  



```

2.  **创建新article2索引（增加了一个owner字段）**


```


PUT article2  
{  
	"settings": {  
		"number\_of\_shards": 3,  
		"number\_of\_replicas": 1 ,  
		"analysis" : {  
			"analyzer" : {  
				"ik" : {  
					"tokenizer" : "ik\_max\_word"  
				}  
			}  
		}  
	},  
	"mappings": {  
		"properties": {  
			"id": {  
				"type": "keyword"  
			},  
			"title": {  
				"type": "text",  
				"analyzer": "ik\_max\_word"  
			},  
			"content": {  
				"type": "text",  
				"analyzer": "ik\_max\_word"  
			},  
			"viewCount": {  
				"type": "integer"  
			},  
			"creatDate": {  
				"type": "date",  
				"format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch\_millis"  
			},  
			"tags": {  
				"type": "keyword"  
			},   
			"owner": {  
				"type": "keyword"  
			}  
		}  
	}  
}  



```

3.  **重建索引 reindex**


```


POST \_reindex  
{  
	"source": {  
		"index": "article1"  
	},  
	"dest": {  
		"index": "article2"  
	}  
}  



```

4.  **修改别名映射**


```


POST /\_aliases  
{  
	"actions": \[  
	{  
		"remove": {  
			"index": "article1",  
			"alias": "article"  
		}  
	},  
	{  
		"add": {  
			"index": "article2",  
			"alias": "article"  
		}  
	}  
	\]  
}  



```

5.  使用别名搜索


```


GET /article/\_search  
{  
    "query": {  
    	"match\_all": {}  
    }  
}  



```

5 从0开始，ES工业级Java 应用开发
=====================

5.1 High Level Client基本用法
-------------------------

High Level Client客户端是构建于 Low Level Client之上的封装。

类似于Hibernate和JDBC的关系。

使用Spring Data ElasticSearch访问ElastiSearch,注意版本对应关系

![](./2024/06/16/ES从入门到精通/20.png)
spring boot 2.6.1对应的ES版本
![](./2024/06/16/ES从入门到精通/21.png)
High Level Client客户端测试案例

```


@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.NONE)  
public class RestHighLevelClientTest  
{  
  
    //用来操作 Elasticsearch 服务器的一个客户端对象类  
    @Autowired  
    private RestHighLevelClient restHLClient;  
  
    @ParameterizedTest  
    @ValueSource(ints = {1, 2, 3})  
    void testWithSimpleValueSource(int argument) {  
        System.out.println("Parameterized test with value: " + argument);  
    }  
  
    @Test  
    @SneakyThrows  
    void testWithSneakyThrows() {  
        throw new Exception("This exception is sneaky thrown!");  
    }  
  
    @ParameterizedTest  
    @CsvSource({"1, 1, 2", "2, 3, 5", "3, 5, 8"})  
    void testAddition(int a, int b, int result) {  
        int sum = a + b;  
        Assertions.assertEquals(result, sum, "Sum of " + a + " and " + b + " should be " + result);  
    }  
  
    //创建三个索引库  
    @ParameterizedTest //参数测试  
    @SneakyThrows  
    @ValueSource(strings = {"books", "items", "users"})  
    public void testCreateIndex(String indexName)  
    {  
  
        //指定分词器创建索引库的json格式的数据，每一行用双引号包起来，然后里面的每个双引号前面用反斜杠\\转义  
        String json = "{" +  
                "\\"settings\\": {" +  
                "    \\"analysis\\": {" +  
                "      \\"analyzer\\": {" +  
                "        \\"default\\": {" +  
                "           \\"tokenizer\\": \\"ik\_max\_word\\"" +  
                "        }" +  
                "      }" +  
                "    }" +  
                "  }" +  
                "}";  
  
        CreateIndexRequest request = new CreateIndexRequest(indexName)  
                //参数1：指定创建索引库时要传入的参数  ； 参数2：指定传入内容的类型  
                .source(json, XContentType.JSON);  
        //创建索引库后返回的响应类型--CreateIndexResponse  
        CreateIndexResponse resp = restHLClient.indices().create(request, RequestOptions.DEFAULT);  
  
        //获取Elasticsearch服务器的响应，就是响应索引库是否创建成功  
        System.err.println(resp.isAcknowledged());  
  
    }  
  
  
    //删除索引库  
    @SneakyThrows  
    @ParameterizedTest  
    @ValueSource(strings = {"items", "users"})  
    public void testDeleteIndex(String indexName)  
    {  
        //删除索引的请求数据  
        DeleteIndexRequest request = new DeleteIndexRequest(indexName);  
        //客户端调用操作索引的方法，然后再调用删除的方法  
        AcknowledgedResponse resp = restHLClient.indices().delete(request, RequestOptions.DEFAULT);  
        //查看删除后的响应  
        System.err.println(resp.isAcknowledged());  
    }  
  
    //查询所有的索引库  
    @SneakyThrows  
    @Test //这个测试不需要参数，直接用这个@Test注解即可  
    public void testGetIndex()  
    {  
        //参数 "\*" ： 表示匹配所有的索引库  
        GetIndexRequest request = new GetIndexRequest("\*");  
        //用rest客户端的方法来查询  
        GetIndexResponse resp = restHLClient.indices().get(request, RequestOptions.DEFAULT);  
        //返回的索引库是一个String类型的数组  
        String\[\] indices = resp.getIndices();  
        //把数组转成字符串  
        String s = Arrays.toString(indices);  
        System.err.println(s);  
  
  
    }  
  
    //往索引库添加文档  
  
    @ParameterizedTest  
    @SneakyThrows  
    //测试参数有多个值 ，用这个注解  
    @CsvSource({  
            "1,火影忍者,旋涡鸣人成长为第七代火影的故事,150",  
            "2,家庭教师,废材纲成长为十代首领的热血事迹,200",  
            "4,七龙珠Z,超级赛亚人贝吉塔来到地球后的热闹景象,400"  
    })  
    public void testSaveDocument(Integer id, String title, String description, Double price)  
    {  
        //表明向 books 索引库添加文档  
        IndexRequest request = new IndexRequest("books")  
                .id(id + "")  
                .source(  
                        "title", title,  
                        "description", description,  
                        "price", price  
                );  
  
        IndexResponse resp = restHLClient.index(request, RequestOptions.DEFAULT);  
        System.err.println(resp);  
  
    }  
  
  
    //根据文档的id获取文档  
    @SneakyThrows  
    @ParameterizedTest  
    @ValueSource(ints = {1, 3})  
    public void testGetDocumentById(Integer id)  
    {  
        //表明从 books 索引库获取文档  
        GetRequest request = new GetRequest("books")  
                //表明根据指定的文档的id获取文档  
                .id(id + "");  
  
        GetResponse resp = restHLClient.get(request, RequestOptions.DEFAULT);  
  
        System.err.println(resp);  
  
    }  
  
    //根据条件查询文档（普通关键字查询和通配符查询）  
  
    @SneakyThrows  
    @ParameterizedTest  
    @CsvSource({  
            "description,热\*",  
            "description,成长"  
    })  
    public void testSearchDocument(String field, String term)  
    {  
        // 构建查询条件的类  
        SearchSourceBuilder builder = new SearchSourceBuilder();  
  
        // 通过 SearchSourceBuilder 可以用面向对象的方式来构建查询的 JSON 字符串  
        // SearchSourceBuilder 需要传入 QueryBuilders，而 QueryBuilders 用于构建 QueryBuilder  
        if (term != null && term.contains("\*"))  
        {  
            //根据字段和通配符关键字查询  
            builder.query(QueryBuilders.wildcardQuery(field, term));  
        } else  
        {  
            //根据字段和普通关键字查询  
            builder.query(QueryBuilders.matchQuery(field,term));  
        }  
  
        //表明从 books 索引库查询文档  
        SearchRequest request = new SearchRequest("books")  
                // 此处的 builder 参数用于构建查询语法  
                .source(builder);  
  
        //客户端调用查询的方法 ， 参数1：查询条件语法  参数2：默认的请求选项，比如超时时间之类的  
        SearchResponse resp = restHLClient.search(request, RequestOptions.DEFAULT);  
  
        System.err.println(resp);  
  
    }  
  
  
    //根据 id 删除文档  
  
    @ParameterizedTest  
    @SneakyThrows  
    @ValueSource(ints = {3,4})  
    public void testDeleteDocumentById(Integer id)  
    {  
        //表明从 books 索引库删除文档  
        DeleteRequest request = new DeleteRequest("books")  
                //获取指定id的文档  
                .id(id+"");  
        //rest客户端调用删除文档的方法  
        DeleteResponse resp = restHLClient.delete(request, RequestOptions.DEFAULT);  
        System.err.println(resp);  
    }  
  
  
  
}  
  



```

5.2 聚合查询
--------

### 5.2.1 DSL聚合查询

1.  创建测试索引


```


PUT /jh\_test  
{  
	"settings": {},  
	"mappings": {  
		"properties": {  
			"name": {  
				"type": "text",  
                "fields": {  
                    "keyword": {  
                        "type": "keyword",  
                        "ignore\_above": 256  
                    }  
                }  
			},  
			"sex": {  
				"type": "keyword"  
			},  
			"buyCount": {  
				"type": "long"  
			},  
            "createMonth":{  
                "type":"keyword"  
            }  
		}  
	}  
}  
  



```

其中字段的含义为：name：姓名、buyCount：购买数量，sex：性别，createMonth：创建月 添加测试数据

```


POST /jh\_test/\_doc/1  
{"name":"张三","buyCount":5,"sex":"男","createMonth":"2021-01"}  
POST /jh\_test/\_doc/2  
{"name":"李四","buyCount":5,"sex":"男","createMonth":"2021-01"}  
POST /jh\_test/\_doc/3  
{"name":"小卷","buyCount":18,"sex":"女","createMonth":"2021-01"}  
POST /jh\_test/\_doc/4  
{"name":"小明","buyCount":6,"sex":"女","createMonth":"2021-01"}  
POST /jh\_test/\_doc/5  
{"name":"张三","buyCount":3,"sex":"男","createMonth":"2021-02"}  
POST /jh\_test/\_doc/6  
{"name":"王五","buyCount":8,"sex":"男","createMonth":"2021-02"}  
POST /jh\_test/\_doc/7  
{"name":"赵四","buyCount":4,"sex":"男","createMonth":"2021-02"}  
POST /jh\_test/\_doc/8  
{"name":"诸葛亮","buyCount":6,"sex":"男","createMonth":"2021-02"}  
POST /jh\_test/\_doc/9  
{"name":"黄忠","buyCount":9,"sex":"男","createMonth":"2021-03"}  
POST /jh\_test/\_doc/10  
{"name":"李白","buyCount":1,"sex":"男","createMonth":"2021-03"}  
POST /jh\_test/\_doc/11  
{"name":"赵四","buyCount":3,"sex":"男","createMonth":"2021-03"}  
POST /jh\_test/\_doc/12  
{"name":"张三","buyCount":2,"sex":"男","createMonth":"2021-03"}  
POST /jh\_test/\_doc/13  
{"name":"李四","buyCount":6,"sex":"男","createMonth":"2021-04"}  
POST /jh\_test/\_doc/14  
{"name":"王五","buyCount":9,"sex":"男","createMonth":"2021-04"}  
POST /jh\_test/\_doc/15  
{"name":"李四","buyCount":4,"sex":"男","createMonth":"2021-04"}  
POST /jh\_test/\_doc/16  
{"name":"王五","buyCount":2,"sex":"男","createMonth":"2021-04"}  
  



```

聚合查询语法：

```


"aggs" : {  
    "<aggregation\_name>" : {                                 <!--聚合名称 -->  
        "<aggregation\_type>" : {                             <!--聚合类型 -->  
            <aggregation\_body>                               <!--聚合具体字段 -->  
        }  
        \[,"meta" : {  \[<meta\_data\_body>\] } \]?                <!--元信息 -->  
        \[,"aggs" : { \[<sub\_aggregation>\]+ } \]?       <!--子聚合 -->  
    }  
}  
  



```

1.  **查询 buyCount 的总和**


```


GET /jh\_test/\_search  
{  
  "size":0,  
  "aggs":{  
    "buyCountSum":{  
      "sum": {  
        "field": "buyCount"  
      }  
    }  
  }  
}  



```

2.  **查询 2021-02 月 buyCount 的总和**


```


{  
  "size":0,  
   "query": {  
		"term": {   
            "createMonth": "2021-02"   
        }  
	},  
  "aggs":{  
    "buyCountSum":{  
      "sum": {  
        "field": "buyCount"  
      }  
    }  
  }  
}  



```

3.  **查询 2021-03 月 buyCount 的最大值：**


```


{  
  "size":0,  
   "query": {  
		"term": {   
            "createMonth": "2021-03"   
        }  
	},  
  "aggs":{  
    "buyCountMax":{  
      "max": {  
        "field": "buyCount"  
      }  
    }  
  }  
}  



```

4.  **查询 2021-03 月 buyCount 的最小值：**


```


{  
  "size":0,  
   "query": {  
		"term": {   
            "createMonth": "2021-03"   
        }  
	},  
  "aggs":{  
    "buyCountMin":{  
      "min": {  
        "field": "buyCount"  
      }  
    }  
  }  
}  
  



```

5.  **同时查询 2021-03 月 buyCount 的最大值和最小值：**


```


{  
  "size":0,  
   "query": {  
		"term": {   
            "createMonth": "2021-03"   
        }  
	},  
  "aggs":{  
    "buyCountMax":{  
      "max": {  
        "field": "buyCount"  
      }  
    },  
     "buyCountMin":{  
      "min": {  
        "field": "buyCount"  
      }  
    }  
  }  
}  



```

6.  **查询所有 name 的去重后的数量**


```


{  
  "size":0,  
  "aggs":{  
    "distinctName":{  
      "cardinality": {  
        "field": "name.keyword"  
      }  
    }  
  }  
}  



```

7.  **查询 2021-04 月 name 的去重后的数量**


```


{  
  "size":0,  
  "query": {  
		"term": {   
            "createMonth": "2021-04"   
        }  
	},  
  "aggs":{  
    "distinctName":{  
      "cardinality": {  
        "field": "name.keyword"  
      }  
    }  
  }  
}  
  



```

8.  **查询 BuyCount 的平均值**


```


{  
    "size":"0",  
    "aggs":{  
        "buyCountAvg":{  
            "avg":{  
                "field":"buyCount"  
            }  
        }  
    }  
}  
  



```

9.  **一次查询 总数，最大值，最小值，平均值，总和**


```


{  
  "size":0,  
  "aggs":{  
    "statsAll":{  
      "stats":{  
        "field":"buyCount"  
      }  
    }  
  }  
}  



```

10.  **根据 createMonth 分组查询每个月的最大 buyCount**


```


{  
    "size":0,  
    "aggs": {  
    "createMonthGroup": {  
      "terms": {  
        "field": "createMonth"  
      },  
      "aggs": {  
        "buyCountMax": {  
          "max": {  
            "field": "buyCount"  
          }  
        }  
      }  
    }  
  }  
}  



```

11.  **查询每 createMonth 下，根据 sex 区分，统计buyCount 的平均值**


```


{  
    "size":0,  
    "aggs": {  
    "createMonthGroup": {  
      "terms": {  
        "field": "createMonth"  
      },  
      "aggs": {  
        "sexGroup": {  
          "terms": {  
            "field": "sex"  
          },  
          "aggs": {  
                "buyCountAvg": {  
                    "avg": {  
                        "field": "buyCount"  
                    }  
                }  
            }  
        }  
      }  
    }  
  }  
}  



```

### 5.2.2 ES客户端实现聚合查询

测试代码如下：

```


@SpringBootTest  
@Slf4j  
public class AggregationTest {  
  
    @Resource  
    ElasticsearchRestTemplate elasticsearchRestTemplate;  
  
    /\*\*  
     \* 查询 buyCount 的总和  
     \*/  
    @Test  
    void aggs1() {  
        SumAggregationBuilder buyCountSum = AggregationBuilders.sum("buyCountSum").field("buyCount");  
        Query query = new NativeSearchQueryBuilder()  
                .withAggregations(buyCountSum)  
//                .addAggregation(buyCountSum)  
                .build();  
        SearchHits<JhTestEntity> search = elasticsearchRestTemplate.search(query, JhTestEntity.class);  
        if (search.hasAggregations()) {  
            Aggregations aggregations = (Aggregations) search.getAggregations().aggregations();  
            if (Objects.nonNull(aggregations)) {  
                Sum sum = aggregations.get("buyCountSum");  
                log.info("计算 buyCount 总数：{} ", sum.getValue());  
            }  
        }  
    }  
  
    /\*\*  
     \* 查询 2021-02 月 buyCount 的总和：  
     \*/  
    @Test  
    void aggs2() {  
        QueryBuilder queryBuilder = QueryBuilders.termQuery("createMonth", "2021-02");  
        SumAggregationBuilder buyCountSum = AggregationBuilders.sum("buyCountSum").field("buyCount");  
        Query query = new NativeSearchQueryBuilder()  
                .withQuery(queryBuilder)  
                .withAggregations(buyCountSum)  
//                .addAggregation(buyCountSum)  
                .build();  
        SearchHits<JhTestEntity> search = elasticsearchRestTemplate.search(query, JhTestEntity.class);  
        if (search.hasAggregations()) {  
            Aggregations aggregations = (Aggregations) search.getAggregations().aggregations();  
            if (Objects.nonNull(aggregations)) {  
                Sum sum = aggregations.get("buyCountSum");  
                log.info("计算 buyCount 总数：{} ", sum.getValue());  
            }  
        }  
    }  
  
    /\*\*  
     \* 查询 2021-03 月 buyCount 的最大值：  
     \*/  
    @Test  
    void aggs3() {  
        QueryBuilder queryBuilder = QueryBuilders.termQuery("createMonth", "2021-03");  
        MaxAggregationBuilder buyCountMax = AggregationBuilders.max("buyCountMax").field("buyCount");  
        Query query = new NativeSearchQueryBuilder()  
                .withQuery(queryBuilder)  
                .withAggregations(buyCountMax)  
//                .addAggregation(buyCountMax)  
                .build();  
        SearchHits<JhTestEntity> search = elasticsearchRestTemplate.search(query, JhTestEntity.class);  
        if (search.hasAggregations()) {  
            Aggregations aggregations = (Aggregations) search.getAggregations().aggregations();  
            if (Objects.nonNull(aggregations)) {  
                Max max = aggregations.get("buyCountMax");  
                log.info("计算 buyCount 最大值：{} ", max.getValue());  
            }  
        }  
    }  
  
    /\*\*  
     \* 查询 2021-03 月 buyCount 的最小值：  
     \*/  
    @Test  
    void aggs4() {  
        QueryBuilder queryBuilder = QueryBuilders.termQuery("createMonth", "2021-03");  
        MinAggregationBuilder buyCountMin = AggregationBuilders.min("buyCountMin").field("buyCount");  
        Query query = new NativeSearchQueryBuilder()  
                .withQuery(queryBuilder)  
                .withAggregations(buyCountMin)  
//                .addAggregation(buyCountMin)  
                .build();  
        SearchHits<JhTestEntity> search = elasticsearchRestTemplate.search(query, JhTestEntity.class);  
        if (search.hasAggregations()) {  
            Aggregations aggregations = (Aggregations) search.getAggregations().aggregations();  
            if (Objects.nonNull(aggregations)) {  
                Min min = aggregations.get("buyCountMin");  
                log.info("计算 buyCount 最小值：{} ", min.getValue());  
            }  
        }  
    }  
  
    /\*\*  
     \* 同时查询 2021-03 月 buyCount 的最大值和最小值：  
     \*/  
    @Test  
    void aggs5() {  
        QueryBuilder queryBuilder = QueryBuilders.termQuery("createMonth", "2021-03");  
        MaxAggregationBuilder buyCountMax = AggregationBuilders.max("buyCountMax").field("buyCount");  
        MinAggregationBuilder buyCountMin = AggregationBuilders.min("buyCountMin").field("buyCount");  
        Query query = new NativeSearchQueryBuilder()  
                .withQuery(queryBuilder)  
                .withAggregations(buyCountMax)  
                .withAggregations(buyCountMin)  
//                .addAggregation(buyCountMax)  
//                .addAggregation(buyCountMin)  
                .build();  
        SearchHits<JhTestEntity> search = elasticsearchRestTemplate.search(query, JhTestEntity.class);  
        if (search.hasAggregations()) {  
            Aggregations aggregations = (Aggregations) search.getAggregations().aggregations();  
            if (Objects.nonNull(aggregations)) {  
                Max max = aggregations.get("buyCountMax");  
                log.info("计算 buyCount 最大值：{} ", max.getValue());  
                Min min = aggregations.get("buyCountMin");  
                log.info("计算 buyCount 最小值：{} ", min.getValue());  
            }  
        }  
    }  
  
    /\*\*  
     \* 查询所有 name 的去重后的数量  
     \*/  
    @Test  
    void aggs6() {  
        CardinalityAggregationBuilder distinctName = AggregationBuilders.cardinality("distinctName").field("name.keyword");  
        Query query = new NativeSearchQueryBuilder()  
                .withAggregations(distinctName)  
//                .addAggregation(distinctName)  
                .build();  
        SearchHits<JhTestEntity> search = elasticsearchRestTemplate.search(query, JhTestEntity.class);  
        if (search.hasAggregations()) {  
            Aggregations aggregations = (Aggregations) search.getAggregations().aggregations();  
            if (Objects.nonNull(aggregations)) {  
                Cardinality cardinality = aggregations.get("distinctName");  
                log.info("计算 name 的去重后的数量：{} ", cardinality.getValue());  
            }  
        }  
    }  
  
    /\*\*  
     \*  查询 2021-04 月 name 的去重后的数量  
     \*/  
    @Test  
    void aggs7() {  
        QueryBuilder queryBuilder = QueryBuilders.termQuery("createMonth", "2021-04");  
        CardinalityAggregationBuilder distinctName = AggregationBuilders.cardinality("distinctName").field("name.keyword");  
        Query query = new NativeSearchQueryBuilder()  
                .withQuery(queryBuilder)  
                .withAggregations(distinctName)  
//                .addAggregation(distinctName)  
                .build();  
        SearchHits<JhTestEntity> search = elasticsearchRestTemplate.search(query, JhTestEntity.class);  
        if (search.hasAggregations()) {  
            Aggregations aggregations = (Aggregations) search.getAggregations().aggregations();  
            if (Objects.nonNull(aggregations)) {  
                Cardinality cardinality = aggregations.get("distinctName");  
                log.info("计算 name 的去重后的数量：{} ", cardinality.getValue());  
            }  
        }  
    }  
  
  
    /\*\*  
     \* 查询 BuyCount 的平均值  
     \*/  
    @Test  
    void aggs8() {  
        AvgAggregationBuilder buyCountAvg = AggregationBuilders.avg("buyCountAvg").field("buyCount");  
        Query query = new NativeSearchQueryBuilder()  
                .withAggregations(buyCountAvg)  
//                .addAggregation(buyCountAvg)  
                .build();  
        SearchHits<JhTestEntity> search = elasticsearchRestTemplate.search(query, JhTestEntity.class);  
        if (search.hasAggregations()) {  
            Aggregations aggregations = (Aggregations) search.getAggregations().aggregations();  
            if (Objects.nonNull(aggregations)) {  
                Avg avg = aggregations.get("buyCountAvg");  
                log.info("计算 buyCount 的平均值：{} ", avg.getValue());  
            }  
        }  
    }  
  
    /\*\*  
     \* 一次查询 总数，最大值，最小值，平均值，总和  
     \*/  
    @Test  
    void aggs9() {  
        StatsAggregationBuilder stats = AggregationBuilders.stats("stats").field("buyCount");  
        Query query = new NativeSearchQueryBuilder()  
                .withAggregations(stats)  
//                .addAggregation(stats)  
                .build();  
        SearchHits<JhTestEntity> search = elasticsearchRestTemplate.search(query, JhTestEntity.class);  
        if (search.hasAggregations()) {  
            Aggregations aggregations = (Aggregations) search.getAggregations().aggregations();  
            if (Objects.nonNull(aggregations)) {  
                Stats s = aggregations.get("stats");  
                log.info("计算 buyCount 的 count：{} ", s.getCount());  
                log.info("计算 buyCount 的 min：{} ", s.getMin());  
                log.info("计算 buyCount 的 max：{} ", s.getMax());  
                log.info("计算 buyCount 的 avg：{} ", s.getAvg());  
                log.info("计算 buyCount 的 sum：{} ", s.getSum());  
            }  
        }  
    }  
  
    /\*\*  
     \* 根据 createMonth 分组查询每个月的最大 buyCount  
     \*/  
    @Test  
    void aggs10() {  
        TermsAggregationBuilder createMonthGroup = AggregationBuilders.terms("createMonthGroup").field("createMonth")  
                .subAggregation(AggregationBuilders.max("buyCountMax").field("buyCount"));  
  
        Query query = new NativeSearchQueryBuilder()  
                .withAggregations(createMonthGroup)  
//                .addAggregation(createMonthGroup)  
                .build();  
        SearchHits<JhTestEntity> search = elasticsearchRestTemplate.search(query, JhTestEntity.class);  
        if (search.hasAggregations()) {  
            Aggregations aggregations = (Aggregations) search.getAggregations().aggregations();  
            if (Objects.nonNull(aggregations)) {  
                Terms terms = aggregations.get("createMonthGroup");  
                terms.getBuckets().forEach(bucket -> {  
                    String createMonth = bucket.getKeyAsString();  
                    Aggregations subAggs = bucket.getAggregations();  
                    if (Objects.nonNull(subAggs)) {  
                        Max max = subAggs.get("buyCountMax");  
                        log.info("计算 {} 月的最大值为：{} ", createMonth, max.getValue());  
                    }  
                });  
            }  
        }  
    }  
  
    /\*\*  
     \*  查询每 createMonth 下，根据 sex 区分，统计buyCount 的平均值  
     \*/  
    @Test  
    void aggs11() {  
        TermsAggregationBuilder createMonthGroup = AggregationBuilders.terms("createMonthGroup").field("createMonth")  
                .subAggregation(AggregationBuilders.terms("sexGroup").field("sex")  
                        .subAggregation(AggregationBuilders.avg("buyCountAvg").field("buyCount")));  
  
        Query query = new NativeSearchQueryBuilder()  
                .withAggregations(createMonthGroup)  
//                .addAggregation(createMonthGroup)  
                .build();  
        SearchHits<JhTestEntity> search = elasticsearchRestTemplate.search(query, JhTestEntity.class);  
        if (search.hasAggregations()) {  
            Aggregations aggregations = (Aggregations) search.getAggregations().aggregations();  
            if (Objects.nonNull(aggregations)) {  
                Terms terms = aggregations.get("createMonthGroup");  
                terms.getBuckets().forEach(bucket -> {  
                    String createMonth = bucket.getKeyAsString();  
                    Aggregations sexAggs = bucket.getAggregations();  
                    if (Objects.nonNull(sexAggs)) {  
                        Terms sexTerms = sexAggs.get("sexGroup");  
                        sexTerms.getBuckets().forEach(sexBucket -> {  
                            String sex = sexBucket.getKeyAsString();  
                            Aggregations avgAggs = sexBucket.getAggregations();  
                            if (Objects.nonNull(avgAggs)) {  
                                Avg avg = avgAggs.get("buyCountAvg");  
                                log.info("计算 {} 月，{} 性 的平均值为：{} ", createMonth, sex, avg.getValue());  
                            }  
                        });  
                    }  
                });  
            }  
        }  
    }  
  
}  
  



```

5.3 ES ES工业级Java 应用开发
---------------------

一般不会直接使用RestHighLevelClient对于ES操作，因为语法过于繁琐，重复，基于分离变和不变的原则可以进行工业级封装。

http://localhost:8080/doc.html

![](./2024/06/16/ES从入门到精通/22.png)
工业级封装后的Service：

> 文章实在太长， 这里省略了500字+， 省略的内容请参见的免费电子书 PDF版本《ES学习圣经：从0到1, 精通 ElasticSearch 工业级使用 》

6 ES集群架构
========

6.1 ES集群的5大角色
-------------

1.  **Master Node ：主节点**


主节点，该节点不和应用创建连接，每个节点都保存了集群状态。master节点控制整个集群的元数据。只有Master Node节点可以修改节点状态信息，及元数据(metadata)的处理。 元数据(metadata)，比如：

*   索引的分片信息 、主副本信息

*   分片的节点分配信息，路由分配

*   index 、type、Mapping

*   Setting 配置等等。


从资源占用的角度来说：master节点不占用磁盘IO和CPU，内存使用量一般， 没有data 节点高类似于kafa中的 controller，负责集群元数据的管理和维护

2.  **Master eligible nodes ：合格节点** 有资格成为Master节点但暂时并不是Master的节点被称为 eligible 节点，该节点可以参加选主流程，成为Master节点. 该节点只是与集群保持心跳，判断Master是否存活，如果Master故障则参加新一轮的Master选举。 从资源占用的角度来说：eligible节点比Master节点更节省资源，因为它还未成为 Master 节点， 只是有资格成功Master节点。

3.  **Data Node ：数据节点** 职责： 数据节点，用于建立文档索引，管理shard。 类似于rocket 中的 broker，负责数据的管理和维护，数据节点职责：


*   该节点用于建立文档索引， 接收 应用创建连接、接收索引请求

*   查询，接收用户的搜索请求


data节点的分片执行查询语句获得查询结果后将结果反馈给Coordinating节点，在查询的过程中非常消 耗硬件资源，如果在分片配置及优化没做好的情况下，进行一次查询非常缓慢(硬件配置也要跟上数据量)。 数据节点：保存包含索引文档的分片数据，执行CRUD、搜索、聚合相关的操作。属于：内存、CPU、IO密集型，对硬件资源要求高。 从资源占用的角度来说：data节点会占用大量的CPU、IO和内存

4.  **Coordinating Node ：协调节点(/路由节点/client节点)**


协调节点，该节点专用与接收应用的查询连接、接受搜索请求，但其本身不负责存储数据 协调节点，接受客户端搜索请求后将请求转发到与查询条件相关的多个data节点的分片上，然后多个data节点的分片执行查询语句或者查询结果再返回给协调节点，协调节点把各个data节点的返回结果进行整合、排序等一系列操作后再将最终结果返回给用户请求。 data节点的分片执行查询语句获得查询结果后将结果反馈给Coordinating节点，在查询的过程中非常消耗硬件资源，如果在分片配置及优化没做好的情况下，进行一次查询非常缓慢(硬件配置也要跟上数据量)。 搜索请求在两个阶段中执行（query 和 fetch），这两个阶段由接收客户端请求的节点 - 协调节点协调。

*   在请求query 阶段，协调节点将请求转发到保存数据的数据节点。每个数据节点在本地执行请求并将其结果返回给协调节点。

*   在收集fetch阶段，协调节点将每个数据节点的结果汇集为单个全局结果集。


从资源占用的角度来说：协调节点，可当负责均衡节点，该节点不占用io、cpu和内存 总结：Coordinating 大致的职责 ： 请求分发、结果的合并

5.  **Ingest Node ：ingest节点**


ingest 节点可以看作是数据前置处理转换的节点，支持 pipeline管道 设置，可以使用 ingest 对数据进 行过滤、转换等操作，类似于 logstash 中 filter 的作用，功能相当强大。 Ingest节点处理时机——在数据被索引之前，通过预定义好的处理管道对数据进行预处理。默认情况下，所有节点都启用Ingest，因此任何节点都可以处理Ingest任务。 当然，我们也可以创建专用的Ingest节点。

6.  **部落（tribe）**


接着说一下ES里面的部落：tribe， 可以在查询过程中链接两个集群的数据，查询的数据将会汇总到tribe节点，有tribe节点对数据进行整合再发送给client； tribe还可以写数据，但是这里有一个限制，就是写的索引只能是一个集群所有；如果写入两个集群同名索引，那么只能成功写入一个，至于写入哪一个可以通过配置偏好实现。 可以通过配置指明tribe只能读，不能写。

6.2 ES集群节点角色配置
--------------

```


1. node.master  
2. node.data  
3. node.ingest  



```

配置实例/usr/share/elasticsearch/config/elasticsearch.yml

```


bootstrap.memory\_lock: true  
cluster.name: "es-cluster"  
node.name: master1  
node.master: true  
node.data: true  
network.host: 0.0.0.0  
http.port: 9200  
transport.tcp.port: 9300  
cluster.initial\_master\_nodes: \["master1"\]  
discovery.zen.ping.unicast.hosts: master1, master2, master3  
#官方推荐 master-eligible nodes / 2 + 1 向下取整的个数  
discovery.zen.minimum\_master\_nodes: 2  
path.logs: /usr/share/elasticsearch/logs  
http.cors.enabled: true  
http.cors.allow-origin: "\*"  
xpack.security.audit.enabled: true  



```

默认情况下这三个属性的值都是true，实际上，一个节点在默认情况下会同时扮演：Master Node，Data Node 和 Ingest Node。

| **节点类型** | **配置参数** | **默认值** |
| --- | --- | --- |
|Master Eligible| node.master| true|
| Data| node.data| true|
| Coordinating only| 无| 

设置上面2 个参数全为 false，节点为专用协调节点

|| Ingest| node.ingest| true|

6.3 ES节点配置组合
------------

1.  **组合1**


```


node.master: true   
node.data: true   
node.ingest: true  



```
```


这种组合表示这个节点既有成为主节点的资格，又可以存储数据，还可以作为预处理节点这个时候如果某个节点被选举成为了真正的主节点，那么他还要存储数据，这样对于这个节点的压力就比较大了。  
elasticsearch 默认是：每个节点都是这样的配置，在测试环境下这样做没问题。实际工作中建议不要这  



```

样设置，这样相当于 主节点 和 数据节点 的角色混合到一块了。

2.  **组合2**


```


node.master: false   
node.data: true   
node.ingest: false  



```

这种组合表示这个节点没有成为主节点的资格，也就不参与选举，只会存储数据。这个节点我们称为 data(数据)节点。在集群中需要单独设置几个这样的节点负责存储数据。后期提供存储和查询服务

3.  **组合3**


```


node.master: true  
node.data: false  
node.ingest: false  



```

这种组合表示这个节点不会存储数据，有成为主节点的资格，可以参与选举，有可能成为真正的主节点。这个节点我们称为master节点

4.  **组合4**


```


node.master: false   
node.data: false   
node.ingest: true  



```

这种组合表示这个节点即不会成为主节点，也不会存储数据，这个节点的意义是作为一个 client(客户端)节点，主要是针对海量请求的时候可以进行负载均衡。 在新版 ElasticSearch5.x 之后该节点称之为：coordinate 节点，其中还增加了一个叫：ingest 节点，用 于预处理数据（索引和搜索阶段都可以用到）。 当然，作为一般应用是不需要这个预处理节点做什么额外的预处理过程，那么这个节点和我们称之为client 节点之间可以看做是等同的，我们在代码中配置访问节点就都可以配置这些 ingest 节点即可。‍

6.4 高可用ES的部署架构
--------------

![](./2024/06/16/ES从入门到精通/23.png)
1.  **小型的ES集群（<10）的节点架构**


![](./2024/06/16/ES从入门到精通/24.png)
*   对于Ingest转换节点，如果我们没有格式转换、类型转换等需求，直接设置为false。

*   3-5个节点属于轻量级集群，要保证主节点个数满足((节点数/2)+1)。

*   轻量级集群，节点的多重属性如：Master&Data设置为同一个节点可以理解的。

*   如果进一步优化，5节点可以将Master和Data再分离。


2.  **中型的ES集群（10-50）的节点架构**


*   三台服务器做master节点 （可选）

*   N（比如20）台服务器作为data节点（存储资源要大）

*   N（比如5）台做coodinate/ingest节点（用于搜索结果合并，可以提高ES查询效率）


![](./2024/06/16/ES从入门到精通/25.png)
3.  **超大型的ES集群的节点架构（150个节点+）**


可以按照100个节点为单位，分成多个集群，通过 tribenode连接，单个ES数据库最好的高可用集群部署架构为：每个集群，三台服务器做master节点

*   N（比如50）台服务器作为data节点（存储资源要大）

*   N（比如5）台做coodinate节点（用于搜索结果合并，可以提高ES查询效率）

*   N（比如2）台做ingest节点（用于数据转换，可以提高ES索引效率）


![](./2024/06/16/ES从入门到精通/26.png)
6.5 小型ES集群的安装
-------------

### 6.5.1 Image镜像

有外网环境，拉取镜像代码如下：

```


#下载elasticsearch,带中文分词器的版本  
docker pull andylsr/elasticsearch-with-ik-icu:7.14.0  
#下载kibana  
docker pull docker.elastic.co/kibana/kibana:7.14.0  



```

无外网环境，可以先从有公网的环境拉取镜像，然后导出镜像

```


docker save andylsr/elasticsearch-with-ik-icu:7.14.0 -o /root/elasticsearchwith-ik-icu.tar  
docker save docker.elastic.co/kibana/kibana:7.14.0 -o /root/kibana.tar  



```

然后上传导出的de镜像到dao目标虚拟机，然后导入镜像到docker

```


docker load -i /vagrant/3G-middleware/elasticsearch-with-ik-icu.tar  
docker load -i /vagrant/3G-middleware/kibana.tar  
docker load -i /vagrant/3G-middleware/haproxy.tar  



```

### 6.5.2 创建目录结构

```


mkdir -p /home/docker-compose/elasticsearch7/{coordinate1,coordinate2}-{logs,data}  
mkdir -p /home/docker-compose/elasticsearch7/{master1,master2,master3}-{logs,data}  
chmod -R 777 /home/docker-compose/elasticsearch7  



```

### 6.5.3 安装ES集群

把docker-compose和ES相应配置文件拷贝到`/home/docker-compose`目录下，执行如下命令安装ES集群

```


docker-compose --compatibility up -d # 兼容模式后台启动  



```

*   \--compatibility：表示已兼容模式启动容器

*   \-d：表示后台启动


6.6 使用kibana访问集群
----------------

1.  **查看集群健康情况**


```


GET \_cat/health?v  



```

结果：

```


epoch      timestamp cluster    status node.total node.data shards pri relo init unassign pending\_tasks max\_task\_wait\_time active\_shards\_percent  
1670418184 13:03:04  es-cluster green           5         3     16   8    0    0        0             0                  -                100.0%  
  



```

status选项的值

*   green : 所有primary shard和replica shard都已成功分配, 集群是100%可用的

*   yellow : 所有primary shard都已成功分配, 但至少有一个replica shard缺失. 此时集群所有功能都正常使用, 数据不会丢失, 搜索结果依然完整, 但集群的可用性减弱. —— 需要及时处理的警告

*   red : 至少有一个primary shard(以及它的全部副本分片)缺失 —— 部分数据不能使用, 搜索只能返回部分数据, 而分配到这个分配上的写入请求会返回一个异常. 此时虽然可以运行部分功能, 但为了索引数据的完整性, 需要尽快修复集群


2.  **查看集群中的节点个数**


```


GET \_cat/nodes?v  



```
```


ip         heap.percent ram.percent cpu load\_1m load\_5m load\_15m node.role   master name  
172.19.0.2           36          95  12    0.20    0.43     0.73 cdfhilmrstw -      master2  
172.19.0.7           75          95  12    0.20    0.43     0.73 cdfhilmrstw \*      master3  
172.19.0.5           29          95  12    0.20    0.43     0.73 cdfhilmrstw -      master1  
172.19.0.6           64          96  12    0.20    0.43     0.73 lr          -      coordinate2  
172.19.0.3           23          96  12    0.20    0.43     0.73 lr          -      coordinate1  



```

*   第一列（ip）：es节点ip

*   第二列（heap.percent）：堆内存占比

*   第三列（ram.percent）：内存使用占比

*   第四列（cpu）：cpu使用率

*   第五列（load\_1m）：1分钟内平均load情况

*   第六列（load\_5m）：5分钟内平均load情况

*   第七列（load\_15m）：15分钟内平均load情况

*   第八列（node.role）：节点权限

*   第九列（master）：是否master节点，\*为master节点

*   第十列（name）：节点名称


**（1）heap.percent** 表示ES使用的JVM内存情况，该值应该低于75，如果长时间大于75，表示JVM内存配置不够，如果JVM已经配置到30G，则表示该Datanode节点上的压力较大，需要考虑增加Datanode节点来分摊压力 **（2）ram.percent** 表示机器上内存的使用情况，实际对应linux上的 used+cache内存使用情况，如果该值接近100%，则表示机器上cache内存不够用，主要是由于ES检索中，lucene会消耗大量cache内存，如果cache不够，会导致lucene无法将部分文件加载到cache中，会频繁从磁盘中进行读取，导致查询延时加大

3.  **查看集群中的索引**


```


GET \_cat/indices?v  



```
```


health status index                           uuid                   pri rep docs.count docs.deleted store.size pri.store.size  
green  open   .kibana-event-log-7.14.0\-000001 1uw627v6T8KPBv2xrlPlPQ   1   1          9            0     97.8kb         48.9kb  
green  open   .kibana\_7.14.0\_001              1G2WGiXOT2Wvux7R\_5TuLA   1   1         86           14      4.9mb          2.7mb  
green  open   .apm-custom-link                dMukcWA2QVOFzsOhWiPOng   1   1          0            0       416b           208b  
green  open   .apm-agent-configuration        ReYFH-LaQbmWjGTwif0nHA   1   1          0            0       416b           208b  
green  open   .kibana\_task\_manager\_7.14.0\_001 H-3hQBgZRO2AfYsWLS6LQw   1   1         14         1275    622.9kb        337.1kb  
green  open   user                            HdrFzj7TQ\_ejXe\_9Xdx1LQ   1   1          6            2     37.4kb         17.4kb  
green  open   .tasks                          DudUNrFYTq24R7ZY9IFK5A   1   1         16            0     80.9kb         43.4kb  
  



```

7 数据类型和映射
=========

7.1 映射的创建
---------

和传统数据库不同，传统的数据库我们尝试向表中插入数据的前提是这个表已经存在数据结构的定义，且插入数据的字段要在表结构中被定义。 而ES的映射的创建支持主动和被动创建。

1.  **被动创建（动态映射）**


此时字段和映射类型不需要事先定义，只需要存在文档的索引，当向此索引添加数据的时候当遇到不存在的映射字段，ES会根据数据内容自动添加映射字段定义。

2.  **主动创建（显示映射）**


动态映射只能保证最基础的数据结构的映射，所以很多时候我们需要对字段除了数据结构定义更多的限制的时候，动态映射创建的内容很可能不符合我们的需求，所以可以使用 PUT {index}/mapping 来更新指定索引的映射内容。

7.2 动态映射Dynamic Mapping
-----------------------

写入文档的时候，索引不存在，会自动创建索引， 无需手动创建，ES会根据内容推断字段的类型，推断会不准确，可能造成某些功能无法使用，例如 范围查询。

```


POST /log2/\_doc/1  
{  
    "uid" : 1,  
    "ip" : "192.1.1.1",  
    "transTime" : "2018-01-01",  
    "content" : "中华人民共和国人民大会堂"  
}  



```
```


查看一个索引当前的mapping  



```
```


GET /log2/\_mapping  



```

结果如下

```


{  
    "log2": {  
        "mappings": {  
            "properties": {  
                "content": {  
                    "type": "text",  
                    "fields": {  
                        "keyword": {  
                            "type": "keyword",  
                            "ignore\_above": 256  
                        }  
                    }  
                },  
                "ip": {  
                    "type": "text",  
                    "fields": {  
                        "keyword": {  
                            "type": "keyword",  
                            "ignore\_above": 256  
                        }  
                    }  
                },  
                "transTime": {  
                    "type": "date"  
                },  
                "uid": {  
                    "type": "long"  
                }  
            }  
        }  
    }  
}  



```
```


动态映射规则


```

| **JSON中数据类型** | **Elasticsearch 数据类型** |
| --- | --- |
|null| 不添加任何字段|
| true或者false| boolean类型|
| 浮点数据| float类型|
| integer数据| long类型|
| object| object类型|
| array| 取决于数组中的第一个非空值的类型。|
| string| 匹配日期格式，设置为date；匹配数字，设置为float或者long，功能默认关闭；设置为text，并增加keyword子字段。|

7.3 显示的设置mapping
----------------

显示的设置mapping可以更灵活控制ES。 映射创建时，除了对字段的定义，Mapping创建的时候提供了一些对于查询策略和自身定义的参数配置。 下面只是简单介绍下映射支持的字段参数内容。

| 参数 | 说明 |
| --- | --- |
|analyzer| 定义此字段索引时使用的分词方式|
| normalizer| normalizer功能类似于analyzer，但是其可以使查询条件输出唯一的查询条件（可以认为其只是实现了条件小写等不会产生多个查询条件的相关操作）|
| boost| 定义当前字段的查询权重|
| coerce| 此字段控制是否尝试修复部分错误的数据格式，（比如对一个整数字段插入字符串比如"5"，此时此字符串可以被解析为数字），默认为true|
| copy\_to| 类似于别名，不同之处参数可以将此字段内容复制到指定字段中，多个字段可以复制到同一个字段中|
| doc\_values| 倒排索引虽然可以快速查询文档中内容，但是在进行排序或聚合操作的时候，倒排索引并不能获得文档内容，所以需要存储一份文档数据到doc\_values，而此参数控制字段是否需要存储在doc\_values中的开关。|
| dynamic| 是否开启动态映射，目前支持三个参数：true/false 开启和关闭，strict 当出现未定义的字段，抛出异常并拒绝添加文档|
| enabled| 此参数控制字段是否可以被索引，当被设置为false的时候表示此字段仅用来存储而无需索引，此时ES不会分析此字段内的数据，所以即使插入的非法的数据内容ES依旧允许执行|
| fielddata| 类似doc\_values都是单独存储额外的文档数据，这样通过倒排索引获取文档内容，从而实现在排序和聚合上的功能。不同的是doc\_values不支持text格式，text格式数据需要使用fielddata。此参数默认是禁止的，这是因为在第一次对字段进行排序或聚合的时候它会把这个列数据都加载到内存中，这样会带来大量的内存消耗。|
| eager\_global\_ordinals| 是否使用全局序号来进行聚合。主要在聚合分析构建hash的时候，使用序号来替代doc的值，这样在文档收集阶段根据需要收集到各个桶中，在计算结果时将序号转换为具体doc内容。但是此操作在每次查询时需要重建doc序号关系|
| format| 日期类型字段用来解析的日期格式|
| ignore\_above| 当插入字段长度超过此字段设置的值后，此内容将不被索引或存储。对于数组结构字段会作用到每一个元素|
| ignore\_malformed| 当向一个字段插入错误的数据类似时，会抛出异常并拒绝文档。但设置此参数后，对字段插入错误的数据时会忽略异常，此文档错误的数据将不被索引，但是其他字段则正常。|
| index\_options| 控制将哪些信息添加到反向索引中|
| index\_phrases| 主要将两个单词的组合索引到单独字段中，这样在进行精确的短语查询的时候会更有效。支持true和false参数。默认为false。此参数会使索引变大|
| index\_prefixes| 允许对字段的前缀进行索引，此参数用来提高查询的速度|
| index| 控制字段是否可以被索引，被设置为false的字段无法被索引到|
| fields| 此参数可以为同一个字段设置不同的索引方式，但是在\_source字段中只会保存一份，并不会实际增加存储。但是会增加索引大小|
| norms| norms里面存储的是各种各样的归一化因子，此内容会影响到文档的得分，在不需要对字段进行打分的时候可以禁用此参数，需要注意的是对于keyword字段默认为false|
| null\_value| 一般来说空值是无法被索引的，但是此参数允许使用指定的值替换空值，以对其进行索引|
| position\_increment\_gap| 增加近似值匹配|
| properties| 定义类型映射、对象字段和嵌套字段等数据|
| search\_analyzer| 定义此字段查询时使用的分词方式|
| similarity| 此参数可以配置用来计算字段相似性的算法|
| store| 默认情况下字段内容会被索引但是并不会存储字段中的值，想获取字段中的值则需要在source中获取对应字段的数据，当查询仅仅是尝试获取指定字段的内容的时候，可以设置此参数为true，那么系统可以直接获取此字段的内容，不再尝试获取source中的数据。|
| term\_vector| 术语向量的定义，存储一些术语向量，以便可以为特定文档检索它们|
| index| 控制字段是否可以被索引，被设置为false的字段无法被索引到|
| fields| 此参数可以为同一个字段设置不同的索引方式，但是在\_source字段中只会保存一份，并不会实际增加存储。但是会增加索引大小|
| norms| norms里面存储的是各种各样的归一化因子，此内容会影响到文档的得分，在不需要对字段进行打分的时候可以禁用此参数，需要注意的是对于keyword字段默认为false|
| null\_value| 一般来说空值是无法被索引的，但是此参数允许使用指定的值替换空值，以对其进行索引|
| position\_increment\_gap| 增加近似值匹配|
| properties| 定义类型映射、对象字段和嵌套字段等数据|
| search\_analyzer| 定义此字段查询时使用的分词方式|
| similarity| 此参数可以配置用来计算字段相似性的算法|
| store| 默认情况下字段内容会被索引但是并不会存储字段中的值，想获取字段中的值则需要在source中获取对应字段的数据，当查询仅仅是尝试获取指定字段的内容的时候，可以设置此参数为true，那么系统可以直接获取此字段的内容，不再尝试获取source中的数据。|

1.  **index** 表示字段是否索引。

2.  **index\_options**


index\_options 控制倒排索引记录的内容，一共有4种配置可选。

| index\_options | 含义 |
| --- | --- |
|docs| 只记录文档id（ doc id ）|
| freqs| 记录 doc id 和 term frequences|
| positions| doc id 、 term frequences 、 term position|
| offsets| doc id 、 term frequences 、 term position 、 character|
| offsets|    |

文本类型 text 默认的配置是 positions ，其他默认是docs。 需要注意的是，虽然index\_options提供了offsets这种内容较多的配置级别，但是记录的内容越多，占用的空间也会越多，在实际操作中还是要根据实际情况进行配置。 例如创建mapping,字段名为user\_name，字符串类型。user\_name不需要索引,info字段的倒排索引类型为positions。

```


PUT mapping\_test3  
{  
    "mappings": {  
        "properties": {  
            "user\_name": {  
                "index": false,  
                "type": "text"  
            },  
            "info": {  
                "index\_options": "positions",  
                "type": "text"  
            },  
            "doc": {  
                "type": "text",  
                "index\_options": "docs"  
            },  
            "freq": {  
                "type": "text",  
                "index\_options": "freqs"  
            },  
            "offset": {  
                "type": "text",  
                "index\_options": "offsets"  
            }  
        }  
    }  
}  



```

3.  **ANALYZER** 分词器。es有内置的分词器，也可以使用第三方的分词工具。如IK。


```


{  
    "mappings": {  
        "my\_type": {  
            "properties": {  
                "content": {  
                    "type": "text",  
                    "analyzer": "ik\_max\_word",//写入是的分词器  
                    "search\_analyzer": "ik\_max\_word"//搜索时的分词器  
                }  
            }  
        }  
    }  
}  



```

4.  **COPY\_TO**


允许将一个或者多个字段的值复制到某一个字段中。用来满足一些搜索需要，类似于数据库 title like "%a%" or title2 like "%a%" copy\_to的字段不会出现在\_source里面

```


{  
    "mappings": {  
        "my\_type": {  
            "properties": {  
                "first\_name": {  
                    "type": "text",  
                    "copy\_to": "full\_name"  
                },  
                "last\_name": {  
                    "type": "text",  
                    "copy\_to": "full\_name"  
                },  
                "full\_name": {  
                    "type": "text"  
                }  
            }  
        }  
    }  
}  
  
PUT my\_index/my\_type/1  
{  
    "first\_name": "John",  
    "last\_name": "Smith"  
}   
//full\_name = \["John","Smith"\]  



```

5.  **DOC\_VALUES**


为了加快排序、聚合操作，在建立倒排索引的时候，额外增加一个列式存储映射，是一个空间换时间的做法。 默认是开启的，对于确定不需要聚合或者排序的字段可以关闭。 注意：text类型没有doc\_values。

```


{  
    "mappings": {  
        "my\_type": {  
            "properties": {  
                "status\_code": {  
                    "type": "keyword"  
                },  
                "session\_id": {  
                    "type": "keyword",  
                    "doc\_values": false  
                }  
            }  
        }  
    }  
}  



```

6.  **ENABLED**


enabled默认为true,将搜索所有字段。如果设置为false，该字段将不会被搜索。但仍会随着\_source返回

7.  **FIELDDATA**


对非text类型的字段进行排序可以使用doc\_value来进行加速。但是对于，text类型的字段，却不能进行分组排序。更何况加速。 下面这个异常展示了，对text类型的字段进行分组排序的错误。
![](./2024/06/16/ES从入门到精通/27.png)
但是可以通过设置fielddata值来达到这一目的。它将字段加载到内存中，因此第一次肯定会很慢。而且将占用内存。

8.  **FORMAT**


对字段进行格式化。

```


{  
    "mappings": {  
        "my\_type": {  
            "properties": {  
                "date": {  
                    "type": "date",  
                    "format": "yyyy-MM-dd"  
                }  
            }  
        }  
    }  
}  



```

9.  **IGNORE\_ABOVE**


大小超过ignore\_above设置的字符串不会被索引或存储。

10.  FIELDS


可以为一个字段映射多个数据类型。比如，一个字符串，可以映射为text，满足全文搜索。同时可以映射为keyword,满足分组和排序。 也可以使用多个分词器来对同一个字段进行分词。

```


{  
    "mappings": {  
        "my\_type": {  
            "properties": {  
                "city": {  
                    "type": "text",  
                    "fields": {  
                        "raw": {  
                            "type": "keyword"  
                        }  
                    }  
                }  
            }  
        }  
    }  
}  



```

11.  **STORE**


我们知道，source字段存储了原始数据(默认)。当然可以通过设置其属性值来选择不存储。此外，还可以通过store选择是否额外存储某个字段。store属性默认为no,表示不存储。当设置为yes时，会在source之 外独立存储。此时，搜索时，会绕过\_source，单独进行一次IO得到该字段的值。 store会严重影响搜索效率，尽管如此，在以下两种情况下，还是可以选择使用：

*   字段很长，每次检索\_source代价很大。

*   需要单独对某些字段进行索引重建。


7.4 ES中常见数据类型
-------------

1.  **字符串**


字符串在之前的版本主要指的是 string 类型。但是在5.X版本已经不支持 string 类型。其被 text 和keyword 类型替代

2.  **text**


text字段需要被全文搜索的内容，它可以保存非常长的内容。查询的时候一般使用分词器器进⾏行行分词然后进行全文搜索。text类型的字段不用于排序，很少用于聚合。 （text类型的数据被用来索引长文本，例如电子邮件主体部分或者一款产品的介绍，这些文本会被分析，在建立索引文档之前会被分词器进行分词，转化为词组。经过分词机制之后es允许检索到该文本切分而成的词语，但是text类型的数据不能用来过滤、排序和聚合等操作。

3.  **keyword**


此字段不能使用分词器进行查询，只能搜索该字段的完整的值。所以其主要保存一些可以索引的结构化内容。此字段可以进行排序、聚合等操作。 keyword类型的数据可以满足电子邮箱地址、主机名、状态码、邮政编码和标签等数据的要求，不进行分词，常常被用来过滤、排序和聚合。 综上，可以发现text类型在存储数据的时候会默认进行分词，并生成索引。而keyword存储数据的时候，不会分词建立索引，显然，这样划分数据更加节省内存。）

8 底层知识：正排索引和倒排索引底层原理
====================

8.1 什么是正排索引
-----------

正排索引是按照文档编号或文档ID等有序的方式将每个文档存储在索引中，通过文档编号或ID进行检索。

这种方式类似于数据库表的行，可以很方便地根据文档ID检索到具体的文档，但是不适合处理大规模文档库的情况。

比如mysql的b+锁索引结构
![](./2024/06/16/ES从入门到精通/28.png)
比如书籍目录，可以根据页码找文档内容，就是正排索引
![](./2024/06/16/ES从入门到精通/29.png)
8.2 什么是倒排索引
-----------

倒排索引是按照单词或关键字将文档进行索引，并记录包含该词汇的文档列表。这种方式类似于数据库表的列，可以将具有相同属性的文档按照关键词进行分类，从而实现更加高效和精确的文本搜索。 倒排索引可以理解为Map< item, list< id>>，能够由查询词快速（时间复杂度O(1)）找到包含这个查询词的文件的数据结构。
![](./2024/06/16/ES从入门到精通/30.png)
比如书籍索引页根据关键词，找页码就是倒排索引
![](./2024/06/16/ES从入门到精通/31.png)
8.3 ES如何做到快速索引
--------------

假设有这么几条数据

```


| ID | Name | Age | Sex |  
| -- |:------------:| -----:| -----:|  
| 1 | Kate | 24 | Female  
| 2 | John | 24 | Male  
| 3 | Bill | 29 | Male  



```

ID是Elasticsearch自建的文档id，那么Elasticsearch建立的索引如下: Name：

```


| Term | Posting List |  
| -- |:----:|  
| Kate | 1 |  
| John | 2 |  
| Bill | 3 |  



```

Age：

```


| Term | Posting List |  
| -- |:----:|  
| 24 | \[1,2\] |  
| 29 | 3 |  



```

### 8.3.1 Posting List

Elasticsearch分别为每个field都建立了一个倒排索引，

*   Kate, John, 24, Female这些叫term，

*   而\[1,2\]就是Posting List。


Posting list就是一个int的数组，存储了所有符合某个term的文档id。 根据id查找的话，通过posting list这种索引方式似乎可以很快进行查找，比如要找age=24的同学，在id数组中查找即可。二分查找，但是，如果想通过name来查找呢？

### 8.3.2 Term Dictionary

Elasticsearch为了能快速找到某个term，将所有的term排个序，二分法查找term，logN的查找效， 就像通过字典查找一样. 这样我们可以用二分查找的方式，比全遍历更快地找出目标的term。 这个就是 term dictionary。 所以：反向索引分成两部分，如下图(来自《信息检索导论》)：
![](./2024/06/16/ES从入门到精通/32.png)
左面是词项词典(Term Dictionary)，右边是倒排记录表(Posting)。 在Lucene中，词典和倒排是分开存储的，词典存储在.tii和.tis文件中。 而倒排又分为两部分存储，第一部分是文档号和词频信息，存储在.frq中；另一部分是词的位置信息， 存储在.prx文件中。 有了term dictionary之后，可以用 logN 次磁盘查找得到目标。 问题是：现在再看起来，似乎和传统数据库通过B+Tree的方式类似啊，为什么说比B+Tree的查询快？

### 8.3.3 Term Index

B-Tree通过**减少磁盘寻道次数**来提高查询性能，Elasticsearch也是采用同样的思路

但是磁盘的随机读操作仍然是非常昂贵的（一次random access大概需要10ms的时间）。

所以尽量少的读磁盘，有必要把一些数据缓存到内存里。**Elasticsearch直接通过内存查找term，不读磁盘**

但是整个term dictionary本身又太大了，无法完整地放到内存里。

于是就有了term index。term index有点像一本字典的大的章节表，或者说，像一本书的目录。

比如：

```


A开头的term ……………. Xxx页  
C开头的term ……………. Xxx页  
E开头的term ……………. Xxx页  



```

如果所有的term都是英文字符的话，可能这个term index就真的是26个英文字符表构成的了。 但是实际的情况是，term未必都是英文字符，term可以是任意的byte数组。

而且26个英文字符也未必是每一个字符都有均等的term，比如x字符开头的term可能一个都没有，而s开头的term又特别多。

实际的term index，的内部结构，类似一棵 trie 树：
![](./2024/06/16/ES从入门到精通/33.png)
例子是一个包含 “A”, “to”, “tea”, “ted”, “ten”, “i”, “in”, 和 “inn” 的 trie 树。 Term Dictionary与Term Index存储，Term Dictionary文件的后缀名为tim，Term Index文件的后缀名是tip。

> Lucene为词典做了一层前缀索引(Term Index)，这个索引在Lucene4.0以后采用的数据结构是FST (Finite State Transducer)，一种前缀树的变种，可以称之为前缀索引。

这种数据结构占用空间很小，Lucene打开索引的时候将其全量装载到内存中，加快磁盘上词典查询速度的同时减少随机磁盘访问次数。

### 8.3.4 Trie树（前缀树，字典树）

Trie，又经常叫前缀树，字典树等等。

*   一个节点的所有子孙都有相同的前缀，也就是这个节点对应的字符串，而根节点对应空字符串。

*   一般情况下，不是所有的节点都有对应的值，只有叶子节点和部分内部节点所对应的键才有相关的值。


trie中的键通常是字符串，但也可以是其它的结构。它有很多变种，如后缀树，Radix Tree/Trie，PATRICIA tree，以及bitwise版本的crit-bit tree。

当然很多名字的意义其实有交叉。 在计算机科学中，trie，又称前缀树或字典树，是一种有序树，用于保存关联数组，其中的键通常是字 符串。

与二叉查找树不同，键不是直接保存在节点中，而是由节点在树中的位置决定。

trie的算法可以很容易地修改为处理其它结构的有序序列，比如一串数字或者形状的排列。

这棵树不会包含所有的 term，它包含的是 term 的一些前缀。

通过 term index 可以快速地定位到 term dictionary 的某个 offset，然后从这个位置再往后顺序查找。

再加上一些压缩技术（搜索 Lucene Finite State Transducers）, term index 的尺寸可以只有所有term 的尺寸的**几十分之一**，使得用内存缓存整个 term index 变成可能。整体上来说就是这样的效果：
![](./2024/06/16/ES从入门到精通/34.png)
### 8.3.5 为什么ES检索比Mysql快

现在我们可以回答**“为什么 Elasticsearch/Lucene 检索可以比 Mysql 快”** 了。 Mysql 只有 term dictionary 这一层，是以 b+tree 排序的方式存储在磁盘上的。

检索一个 term 需要若干次（1-3次）的 random access 的磁盘操作。

而ES/ Lucene 在 term dictionary 的基础上添加了 term index 来加速检索，term index 以类似前缀树的形式缓存在内存中。 从 term index 查到对应的 term dictionary 的 block 位置之后，再去磁盘上找 term，大大减少了磁盘的 random access 次数, 将3次 变成了1次。


