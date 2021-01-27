#  ELK(一) Elasticsearch

>项目中使用到了Elasticsearch，花了点时间学习整理。在我能够熟练使用Es之后，无意中发现了一个没见过的名词“**ELK**”，于是乎，又花时间学习整理了一番，特此总结一下。

* **Elasticsearch** 
* **Logstash**
* **Kibana** 



## Elasticsearch 

>Elasticsearch 是一个**分布式**、**可扩展**、**实时**的**搜索与数据分析引擎**。
>
>Elasticsearch 是一个**开源**的搜索引擎，建立在一个全文搜索引擎库 [Apache Lucene™](https://lucene.apache.org/core/) 基础之上。
>
>Elasticsearch 也是使用 **Java** 编写的，它的内部使用 **Lucene** 做索引与搜索，但是它的目的是使全文检索变得简单， 通过**隐藏** Lucene 的复杂性，取而代之的提供一套简单一致的 **RESTful API**。



### 安装并运行Elasticsearch 

方便起见，使用**docker**的安装方式

可以在[Docker Hub](https://hub.docker.com/search/?q=&type=image)上搜索镜像

拉取Es镜像

```
docker pull elasticsearch:7.9.3
```

后台运行

```
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" --name Es -d elasticsearch:7.9.3
```

* -p 9200:9200 将elasticsearch容器的9200端口映射到宿主机的9200端口上（RESTful请求方式默认使用9200端口，即Http方式）
* -p 9300:9300 将elasticsearch容器的9300端口映射到宿主机的9300端口上（分布式情况下，各个节点之间通信默认使用9300端口，即Tcp方式）
* -d 后台运行
* --name 容器名称



### Kibana

docker版本要和Es的一致

拉取Kibana镜像

```
docker pull kibana:7.9.3
```

