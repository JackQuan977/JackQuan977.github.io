---
layout:     post
title:      ElasticSearch学习笔记
subtitle:   概述
date:       2021-07-12
author:     QuanLi
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - ElasticSearch
---

# ElasticSearch学习笔记

### ElasticStack技术栈

​	如果你没有听说过 **Elastic Stack**，那你一定听说过 **ELK** ，实际上 **ELK** 是三款软件的简称，分别是**Elasticsearch**、 **Logstash**、**Kibana** 组成，在发展的过程中，又有新成员 **Beats** 的加入，所以就形成了**Elastic Stack**。所以说，**ELK** 是旧的称呼，**Elastic Stack** 是新的名字。

![image-20210712205325964](https://i.loli.net/2021/07/12/9rxeNRIAXM2JKQn.png)

#### Elasticsearch

​	Elasticsearch 基于 **Java**，是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，**restful** 风格接口，多数据源，自动搜索负载等。

#### Logstash

​	**Logstash** 基于 **Java**，是一个开源的用于收集,分析和存储日志的工具。

#### Kibana

​	**Kibana** 基于 **nodejs**，也是一个开源和免费的工具，**Kibana** 可以为 **Logstash** 和 **ElasticSearch** 提供的日志分析友好的 **Web** 界面，可以汇总、分析和搜索重要数据日志。

#### Beats

​	**Beats** 是 **elastic** 公司开源的一款采集系统监控数据的代理 **agent**，是在被监控服务器上以客户端形式运行的数据收集器的统称，可以直接把数据发送给 **Elasticsearch** 或者通过 **Logstash** 发送给 **Elasticsearch**，然后进行后续的数据分析活动。

​	Beats和Logstash其实都可以进行数据的采集，但是**目前主流的是使用Beats进行数据采集，然后使用 Logstash进行数据的分割处理**等，早期没有Beats的时候，使用的就是Logstash进行数据的采集。

### ElasticSearch快速入门

​	**ElasticSearch** 是一个基于 **Lucene** 的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于**RESTful Web** 接口。**Elasticsearch** 是用 **Java** 开发的，并作为 **Apache** 许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。

​	我们建立一个网站或应用程序，并要添加搜索功能，但是想要完成搜索工作的创建是非常困难的。我们希望搜索解决方案要运行速度快，我们希望能有一个零配置和一个完全免费的搜索模式，我们希望能够简单地使用JSON通过HTTP来索引数据，我们希望我们的搜索服务器始终可用，我们希望能够从一台开始并扩展到数百台，我们要实时搜索，我们要简单的多租户，我们希望建立一个云的解决方案。因此我们利用Elasticsearch来解决所有这些问题及可能出现的更多其它问题。

​	**ElasticSearch** 是 **Elastic Stack** 的核心，**同时 Elasticsearch 是一个分布式、RESTful 风格的搜索和数据分析引擎**，能够解决不断涌现出的各种用例。作为 **Elastic Stack** 的核心，它集中存储您的数据，帮助您发现意料之中以及意料之外的情况。

### ElasticSearch中的基本概念

#### 索引

​	**索引是 Elasticsearch 对逻辑数据的逻辑存储**，所以它可以分为更小的部分。

​	**可以把索引看成关系型数据库的表**，索引的结构是为快速有效的全文索引准备的，特别是它不存储原始值。

​	**Elasticsearch** 可以把索引存放在一台机器或者分散在多台服务器上，每个索引有一或多个分片（**shard**），每个分片可以有多个副本	（**replica**）。

#### 文档

- 存储在 **Elasticsearch** 中的主要实体叫文档（**document**）。用关系型数据库来类比的话，**一个文档相当于数据库表中的一行记录**。

- 文档由多个字段组成，每个字段可能多次出现在一个文档里，这样的字段叫多值字段（**multivalued**）。 每个字段的类型，可以是文本、数值、日期等。字段类型也可以是复杂类型，一个字段包含其他子文档或者数 组。

#### 映射

​	所有文档写进索引之前都会先进行分析，**如何将输入的文本分割为词条、哪些词条又会被过滤**，这种行为叫做 映射（**mapping**）。一般由用户自己定义规则。

### 文档类型

- 在 **Elasticsearch** 中，一个索引对象可以存储很多不同用途的对象。例如，一个博客应用程序可以保存文章和评论。
- 每个文档可以有不同的结构。
- 不同的文档类型不能为相同的属性设置不同的类型。例如，在同一索引中的所有文档类型中，一个叫 **title** 的字段必须具有相同的类型。

### RESTful API

​	在 **Elasticsearch** 中，提供了功能丰富的 **RESTful API** 的操作，包括基本的 **CRUD**、创建索引、删除索引等操作。

#### 创建非结构化索引

​	在 **Lucene** 中，创建索引是需要定义字段名称以及字段的类型的，在 **Elasticsearch** 中提供了**非结构化的索引**，就是不需要创建索引结构，即可写入数据到索引中，实际上在 **Elasticsearch** 底层会进行结构化操作，此操作对用户是透明的。

#### 创建空索引

~~~javascript
PUT /haoke
{
    "settings": {
        "index": {
        "number_of_shards": "2", #分片数
        "number_of_replicas": "0" #副本数
        }
    }
}
~~~

#### 删除索引

~~~javascript
#删除索引
DELETE /haoke
{
    "acknowledged": true
}
~~~

#### 插入数据

​	**URL** 规则： **POST** /{索引}/{类型}/{id}

~~~javascript
POST /haoke/user/1001
#数据
{
"id":1001,
"name":"张三",
"age":20,
"sex":"男"
}
~~~

使用 **postman** 操作成功后

![image-20210712211632549](https://i.loli.net/2021/07/12/atcS3UTNqlY9BwF.png)

我们通过 **ElasticSearchHead** 进行数据预览就能够看到我们刚刚插入的数据了

![image-20210712211712298](https://i.loli.net/2021/07/12/Fypt65uXNoOmYQk.png)

​	说明：非结构化的索引，不需要事先创建，直接插入数据默认创建索引。不指定id插入数据：

![image-20210712211929635](https://i.loli.net/2021/07/12/RCBEnzdPLVK2SIA.png)

#### 更新数据

​	在 **Elasticsearch** 中，文档数据是不能修改的，但是可以通过覆盖的方式进行更新。

~~~javascript
PUT /haoke/user/1001
{
"id":1001,
"name":"张三",
"age":21,
"sex":"女"
}
~~~

覆盖成功后的结果如下：

![image-20210712212218415](https://i.loli.net/2021/07/12/NSpiw5al7qjxzRd.png)

​	可以看到数据已经被覆盖了。问题来了，可以局部更新吗？ -- 可以的。前面不是说，文档数据不能更新吗？ 其实是这样的：在内部，依然会查询到这个文档数据，然后进行覆盖操作

![image-20210712212249882](https://i.loli.net/2021/07/12/tyrL6PUxhTWvDG4.png)

#### 删除索引

​	在 **Elasticsearch** 中，删除文档数据，只需要发起 **DELETE** 请求即可，不用额外的参数

~~~javascript
DELETE 1 /haoke/user/1001
~~~

![image-20210712212646762](https://i.loli.net/2021/07/12/Etru7jkHPg86AOi.png)

​	需要注意的是，**result** 表示已经删除，**version** 也增加了。如果删除一条不存在的数据，会响应 **404**

​	删除一个文档也不会立即从磁盘上移除，它只是被标记成已删除。Elasticsearch将会在你之后添加更多索引的时候才会在后台进行删除内容的清理。【相当于批量操作】

#### 关键字搜索数据

~~~
#查询年龄等于20的用户
GET /haoke/user/_search?q=age:20
~~~

结果如下：

![image-20210712212919150](https://i.loli.net/2021/07/12/FAfIpgY8ZcJGndw.png)

#### DSL搜索

​	Elasticsearch提供丰富且灵活的查询语言叫做DSL查询(Query DSL),它允许你构建更加复杂、强大的查询。 DSL(Domain Specific Language特定领域语言)以JSON请求体的形式出现。

~~~javascript
POST /haoke/user/_search
#请求体
{
    "query" : {
        "match" : { #match只是查询的一种
            "age" : 20
        }
    }
}
~~~

实现：查询年龄大于30岁的男性用户。

![image-20210712213058871](C:\Users\16227\AppData\Roaming\Typora\typora-user-images\image-20210712213058871.png)

### ElasticSearch核心详解

​	在Elasticsearch中，文档以JSON格式进行存储，可以是复杂的结构，如：

~~~javascript
{
    "_index": "haoke",
    "_type": "user",
    "_id": "1005",
    "_version": 1,
    "_score": 1,
    "_source": {
        "id": 1005,
        "name": "孙七",
        "age": 37,
        "sex": "女",
        "card": {
            "card_number": "123456789"
         }
    }
}
~~~

其中，card是一个复杂对象，嵌套的Card对象

#### 元数据（metadata）

一个文档不只有数据。它还包含了元数据(metadata)——关于文档的信息。三个必须的元数据节点是：

![image-20210712213754307](https://i.loli.net/2021/07/12/5JagEfwUdeO72Xh.png)

##### index

​	索引(index)类似于关系型数据库里的“数据库”——它是我们存储和索引关联数据的地方

##### _type

​	在应用中，我们使用对象表示一些“事物”，例如一个用户、一篇博客、一个评论，或者一封邮件。每个对象都属于一个类(class)，这个类定义了属性或与对象关联的数据。user 类的对象可能包含姓名、性别、年龄和Email地址。 在关系型数据库中，我们经常将相同类的对象存储在一个表里，因为它们有着相同的结构。同理，**在Elasticsearch 中，我们使用相同类型(type)的文档表示相同的“事物”，因为他们的数据结构也是相同的。**

​	**每个类型(type)都有自己的映射(mapping)或者结构定义，就像传统数据库表中的列一样。**所有类型下的文档被存储在同一个索引下，但是类**型的映射(mapping)会告诉Elasticsearch不同的文档如何被索引。**

_type 的名字可以是大写或小写，不能包含下划线或逗号。我们将使用blog 做为类型名。

##### _id

​	id仅仅是一个字符串，它与_index 和_type 组合时，就可以在Elasticsearch中唯一标识一个文档。当创建一个文 档，你可以自定义_id ，也可以让Elasticsearch帮你自动生成（32位长度）



### 查询响应

#### pretty

可以在查询url后面添加pretty参数，使得返回的json更易查看。

![image-20210712214251604](https://i.loli.net/2021/07/12/NIaE7Mu5KzmycJZ.png)

#### 指定响应字段

​	在响应的数据中，如果我们不需要全部的字段，可以**指定某些需要的字段进行返回**。通过添加 _source

~~~
GET /haoke/user/1005?_source=id,name
#响应
{
    "_index": "haoke",
    "_type": "user",
    "_id": "1005",
    "_version": 1,
    "found": true,
    "_source": {
        "name": "孙七",
        "id": 1005
     }
}
~~~

如不需要返回元数据，仅仅返回原始数据，可以这样：

~~~
GET /haoke/1 user/1005/_source
~~~

![image-20210712214415673](https://i.loli.net/2021/07/12/nc41mDLqKJ7brRP.png)



还可以这样：

~~~
GET /haoke/user/1005/_source?_1 source=id,name
~~~

#### 判断文档是否存在

如果我们只需要判断文档是否存在，而不是查询文档内容，那么可以这样：

通过发送一个head请求，来判断数据是否存在

![image-20210712214540414](https://i.loli.net/2021/07/12/F1iP2I5KkZywUBL.png)

### 分页

和 **SQL** 使用 **LIMIT** 关键字返回只有一页的结果一样，**Elasticsearch** 接受 **from** 和 **size** 参数：

- **size**: 结果数，默认10
- **from**: 跳过开始的结果数，默认0

如果你想每页显示5个结果，页码从1到3，那请求如下：

~~~
GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10
~~~

### 映射

前面我们创建的索引以及插入数据，都是由 **Elasticsearch** 进行自动判断类型，有些时候我们是需要进行明确字段类型的，否则，自动判断的类型和实际需求是不相符的。

自动判断的规则如下：

![image-20210712215104752](https://i.loli.net/2021/07/12/DFXEsdueLQY5P7S.png)

Elasticsearch中支持的类型如下：

![image-20210712215134690](https://i.loli.net/2021/07/12/HUkTB6A5myrbM4Y.png)

#### 创建明确类型的索引：

~~~
put http://202.193.56.222:9200/itcast?include_type_name=true
{
    "settings":{
        "index":{
            "number_of_shards":"2",
            "number_of_replicas":"0"
        }
    },
    "mappings":{
        "person":{
            "properties":{
                "name":{
                    "type":"text"
                },
                "age":{
                    "type":"integer"
                },
                "mail":{
                    "type":"keyword"
                },
                "hobby":{
                    "type":"text"
                }
            }
        }
    }
}
~~~

查看映射

![image-20210712215332063](https://i.loli.net/2021/07/12/1YL2S5Do9xnWyfj.png)

### 中文分词

#### 什么是分词

​	**分词就是指将一个文本转化成一系列单词的过程**，也叫文本分析，在Elasticsearch中称之为Analysis。

举例：我是中国人 --> 我/是/中国人

#### 分词api

指定分词器进行分词

~~~
POST /_analyze
{
    "analyzer":"standard",
    "text":"hello world"
}
~~~

结果如下

![image-20210712215806585](https://i.loli.net/2021/07/12/LGeNHd2VfFIkR9C.png)

在结果中不仅可以看出分词的结果，还返回了该词在文本中的位置。

### 中文分词难点

​	中文分词的难点在于，在汉语中没有明显的词汇分界点，如在英语中，空格可以作为分隔符，如果分隔不正确就会造成歧义。如：

- **我/爱/炒肉丝**

- **我/爱/炒/肉丝**

  常用中文分词器，**IK**、**jieba**、**THULAC**等，推荐使用 **IK分词器**。

**IK Analyzer** 是一个开源的，基于java语言开发的轻量级的中文分词工具包。从2006年12月推出1.0版开始，**IKAnalyzer** 已经推出了3个大版本。最初，它是以开源项目Luence为应用主体的，结合词典分词和文法分析算法的中文分词组件。新版本的IK Analyzer 3.0则发展为面向Java的公用分词组件，独立于Lucene项目，同时提供了对 **Lucene** 的默认优化实现。

采用了特有的“正向迭代最细粒度切分算法“，具有80万字/秒的高速处理能力 采用了多子处理器分析模式，支持：英文字母（IP地址、Email、URL）、数字（日期，常用中文数量词，罗马数字，科学计数法），中文词汇（姓名、地名处理）等分词处理。 优化的词典存储，更小的内存占用。

### 全文搜索

全文搜索两个最重要的方面是：

- 相关性（**Relevance**） 它是评价查询与其结果间的相关程度，并根据这种相关程度对结果排名的一种能力，这 种计算方式可以是 **TF/IDF** 方法、地理位置邻近、模糊相似，或其他的某些算法。
- 分词（**Analysis**） 它是将文本块转换为有区别的、规范化的 **token** 的一个过程，目的是为了创建倒排索引以及查询倒排索引。

### 单词搜索

~~~
POST /itcast/person/_search
{
    "query":{
        "match":{
            "hobby":"音乐"
        }
    },
    "highlight":{
        "fields":{
            "hobby":{

            }
        }
    }
}
~~~

查询出来的结果如下，并且还带有高亮

![image-20210712220156563](https://i.loli.net/2021/07/12/q1P3scRKaCLEZr9.png)

- 检查字段类型

- - 爱好 hobby 字段是一个 text 类型（ 指定了IK分词器），这意味着查询字符串本身也应该被分词。

- 分析查询字符串 。

- - 将查询的字符串 “音乐” 传入IK分词器中，输出的结果是单个项 音乐。因为只有一个单词项，所以 match 查询执行的是单个底层 term 查询。

- 查找匹配文档 。

- - 用 term 查询在倒排索引中查找 “音乐” 然后获取一组包含该项的文档，本例的结果是文档：3 、5 。

- 为每个文档评分 。

- - 用 term 查询计算每个文档相关度评分 _score ，这是种将 词频（term frequency，即词 “音乐” 在相关文档的hobby 字段中出现的频率）和 反向文档频率（inverse document frequency，即词 “音乐” 在所有文档的hobby 字段中出现的频率），以及字段的长度（即字段越短相关度越高）相结合的计算方式。

### 多词搜索

~~~
POST /itcast/person/_search
{
    "query":{
        "match":{
            "hobby":"音乐 篮球"
            "operator":"and"
        }
    },
    "highlight":{
        "fields":{
            "hobby":{

            }
        }
    }
}
~~~

通过这样的话，就会让两个关键字之间存在and关系了

![image-20210712220456103](https://i.loli.net/2021/07/12/ijNs6gTXcnvC1Bk.png)

### 组合搜索

在搜索时，也可以使用过滤器中讲过的 **bool** 组合查询，示例：

~~~
POST /itcast/person/_search
{
    "query":{
        "bool":{
            "must":{
                "match":{
                    "hobby":"篮球"
                }
            },
            "must_not":{
                "match":{
                    "hobby":"音乐"
                }
            },
            "should":[
                {
                    "match":{
                        "hobby":"游泳"
                    }
                }
            ]
        }
    },
    "highlight":{
        "fields":{
            "hobby":{

            }
        }
    }
}
~~~

上面搜索的意思是： 搜索结果中必须包含篮球，不能包含音乐，如果包含了游泳，那么它的相似度更高。

### 权重

有些时候，我们可能需要对某些词增加权重来影响该条数据的得分。如下：

搜索关键字为“游泳篮球”，如果结果中包含了“音乐”权重为10，包含了“跑步”权重为2。

~~~
POST /itcast/person/_search
{
    "query":{
        "bool":{
            "must":{
                "match":{
                    "hobby":{
                        "query":"游泳篮球",
                        "operator":"and"
                    }
                }
            },
            "should":[
                {
                    "match":{
                        "hobby":{
                            "query":"音乐",
                            "boost":10
                        }
                    }
                },
                {
                    "match":{
                        "hobby":{
                            "query":"跑步",
                            "boost":2
                        }
                    }
                }
            ]
        }
    },
    "highlight":{
        "fields":{
            "hobby":{

            }
        }
    }
}
~~~

### ElasticSearch集群

#### 集群节点

​	ELasticsearch的集群是由多个节点组成的，通过cluster.name设置集群名称，并且用于区分其它的集群，每个节点通过node.name指定节点的名称。

​	在Elasticsearch中，节点的类型主要有4种：

- **master节点**

- - 配置文件中node.master属性为true(默认为true)，就有资格被选为master节点。**master节点用于控制整个集群的操作。比如创建或删除索引，管理其它非master节点等。

- **data节点**

- - 配置文件中node.data属性为true(默认为true)，就有资格被设置成data节点。**data节点主要用于执行数据相关的操作。比如文档的CRUD。**

- **客户端节点**

- - 配置文件中node.master属性和node.data属性均为false。
  - 该节点不能作为master节点，也不能作为data节点。
  - **可以作为客户端节点，用于响应用户的请求，把请求转发到其他节点**

- **部落节点**

- - 当一个节点配置tribe.*的时候，它是一个特殊的客户端，**它可以连接多个集群，在所有连接的集群上执行 搜索和其他操作**。
  - 集群中有三种颜色

![image-20210712221315408](https://i.loli.net/2021/07/12/5x3fX2qQz1tcYsZ.png)

### 分片和副本

为了将数据添加到 **Elasticsearch**，我们需要索引(**index**)——一个存储关联数据的地方。实际上，索引只是一个用来指向一个或多个分片(**shards**)的 逻辑命名空间 (logical namespace).

- 一个分片(**shard**)是一个最小级别“工作单元(**worker unit**)，它只是保存了索引中所有数据的一部分。
- 我们需要知道是分片就是一个Lucene实例，并且它本身就是一个完整的搜索引擎。应用程序不会和它直接通 信。
- 分片可以是主分片(**primary shard**)或者是复制分片(**replica shard**)。
- 索引中的每个文档属于一个单独的主分片，所以主分片的数量决定了索引最多能存储多少数据。
- 复制分片只是主分片的一个副本，它可以防止硬件故障导致的数据丢失，同时可以提供读请求，比如搜索或者从别的 **shard** 取回文档。
- 当索引创建完成的时候，主分片的数量就固定了，但是复制分片的数量可以随时调整。

### 故障转移

#### 将data节点停止

这里选择将 **node02** 停止：

![image-20210712221552692](https://i.loli.net/2021/07/12/3PizM2XKRolDBCx.png)

​	当前集群状态为黄色，表示主节点可用，副本节点不完全可用，过一段时间观察，发现节点列表中看不到node02，副本节点分配到了 **node01** 和 **node03**，集群状态恢复到绿色。

![image-20210712221742785](https://i.loli.net/2021/07/12/gJieu9PXRpH6hKS.png)

可以看到，node02恢复后，重新加入了集群，并且重新分配了节点信息。

### 将master节点停止

接下来，测试将 **node01** 停止，也就是将主节点停止。

![image-20210712221820080](https://i.loli.net/2021/07/12/dRjPuNcw3BioWpa.png)

![image-20210712221845138](https://i.loli.net/2021/07/12/WL6XTAmCRqGdlsE.png)

### 分布式文档

#### 路由

首先，来看个问题：

![image-20210712221937101](https://i.loli.net/2021/07/12/jsb9PDFcqAInglL.png)

如图所示：当我们想一个集群保存文档时，文档该存储到哪个节点呢？ 是随机吗？ 是轮询吗？实际上，在**ELasticsearch** 中，会采用计算的方式来确定存储到哪个节点，计算公式如下：

~~~
shard = hash(routing) % number_1 of_primary_shards
~~~

其中：

- **routing** 值是一个任意字符串，它默认是 **_id** 但也可以自定义。
- 这个 **routing** 字符串通过哈希函数生成一个数字，然后除以主切片的数量得到一个余数(remainder)，余数 的范围永远是0到 **number_of_primary_shards - 1**，这个数字就是特定文档所在的分片

这就是为什么创建了主分片后，不能修改的原因。

### Java客户端

​	在Elasticsearch中，为java提供了2种客户端，一种是REST风格的客户端，另一种是Java API的客户端

#### REST客户端

Elasticsearch提供了2种REST客户端，一种是低级客户端，一种是高级客户端。

- Java Low Level REST Client：官方提供的低级客户端。该客户端通过http来连接Elasticsearch集群。用户在使 用该客户端时需要将请求数据手动拼接成Elasticsearch所需JSON格式进行发送，收到响应时同样也需要将返回的JSON数据手动封装成对象。虽然麻烦，不过该客户端兼容所有的Elasticsearch版本。
- Java High Level REST Client：官方提供的高级客户端。该客户端基于低级客户端实现，它提供了很多便捷的 API来解决低级客户端需要手动转换数据格式的问题。