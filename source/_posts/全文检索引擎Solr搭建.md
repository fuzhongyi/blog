---
title: 全文检索引擎Solr搭建
date: 2017-10-18 09:43:31
tags: solr
categories : 技术水波文
---

{% fi /全文检索引擎Solr搭建/header-img.png %}

{% cq %} Solr是一个高性能，采用Java5开发，基于Lucene的全文搜索服务器。同时对其进行了扩展，提供了比Lucene更为丰富的查询语言，同时实现了可配置、可扩展并对查询性能进行了优化，并且提供了一个完善的功能管理界面，是一款非常优秀的全文搜索引擎。 {% endcq %}

<!-- more -->

## 准备

### 配置环境

+ window
+ [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
+ [Solr 7.0.1](https://mirrors.tuna.tsinghua.edu.cn/apache/lucene/solr/7.0.1/solr-7.0.1.zip)
+ oracle

### 目录说明

下载Solr后解压得到目录

``` text
├─bin               // 脚本的启动目录
├─contrib           // 第三方 jar 包存放目录
├─dist              // 编译打包后存放目录，即构建后的输出产物存放目录
├─docs              // solr 的 API 文档文档存放目录
├─example           // 示范例子的存放目录，example-DIH 目录下的是一些 solr 索引 core 样例
├─licenses          // solr 相关的一些许可信息
└─server            // solr 服务端工作目录，自带集成 jetty 插件方式启动 solr 服务器
    ├─solr          // solr 搜索引擎工作目录，即 SOLRHOME
    ├─solr-webapp   // solr 后台管理页面 webapp
    └─start.jar     // 服务端启动 jar 包
```

## 启动

进入 /bin 目录， 按住 shift + 鼠标右键打开命令窗口

![打开命令窗口](/全文检索引擎Solr搭建/2-1.png)

输入

```text
solr start
```

启动 Solr 服务

![启动 Solr 服务](/全文检索引擎Solr搭建/2-2.png )

在浏览器中输入 [localhost:8983](http://localhost:8983) 访问 Solr 管理后台

![管理后台](/全文检索引擎Solr搭建/2-3.png)

## 新建索引库core

输入

```text
solr create -c my_core
```

新建名为 my_core 的 core

![my_core](/全文检索引擎Solr搭建/3-1.png)

创建成功后，在管理页面选择 Core Admin 可以看到刚创建的 core
      
![my_core](/全文检索引擎Solr搭建/3-2.png)

在 SOLRHOME （目录说明中有提到）下会生成 my_core 目录

![my_core](/全文检索引擎Solr搭建/3-3.png)

``` text
├─conf               // 存放core的配置文件
│  ├─solrconfig.xml  // 定义了这个 core 的配置信息
│  └─start.jar       // 定义索引库的字段及分词器等，这个配置文件是核心文件
└─data               // 存放core的数据，即index-索引文件和log-日志记录
```

## core 配置

### 数据导入

导入 oracle 表 news 中的数据

![table_news](/全文检索引擎Solr搭建/4-1.png)

在 %SOLRHOME%/my_core/conf 目录下创建 data-config.xml 文件，并写入以下内容

```xml
<dataConfig>
  <dataSource user="[user]" password="[password]" url="jdbc:oracle:thin:@[host]:[port]:[SID]" driver="oracle.jdbc.driver.OracleDriver"/>
  <document >
	<!-- 表中字段映射 
	     pk: 主键
	     transformer： clob 转换
	-->
    <entity name="news" pk="ID" query="select * from news" transformer="ClobTransformer">
      <field name="id" column="N_ID"/> 
      <field name="title" column="N_TITLE"/>
      <field name="content" column="N_CONTENT" clob="true"/>
      <field name="time" column="N_TIME"/>
    </entity>
  </document>
</dataConfig>
```

在 %SOLRHOME%/my_core 下创建 lib 文件夹，并将 oracle 驱动包 [ojdbc6.jar](http://www.datanucleus.org/downloads/maven2/oracle/ojdbc6/11.2.0.3/ojdbc6-11.2.0.3.jar) 放到该目录下
编辑 %SOLRHOME%/my_core/conf/solrconfig.xml，添加类库和数据库配置：

```xml
<lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-dataimporthandler-\d.*\.jar" />
```

```xml
<requestHandler name="/dataimport" class="solr.DataImportHandler">  
  <lst name="defaults">  
      <str name="config">data-config.xml</str>  
  </lst>  
</requestHandler>
```

选择当前 my_core ,选择 Schema, 添加索引字段 **（字段名称和 solrconfig.xml 中 field 相同，忽略主键）**

![Add Filed](/全文检索引擎Solr搭建/4-2.png)

添加完成后，打开 %SOLRHOME%/my_core/conf/managed-schema，可以看到，此时添加索引字段已写入该文件中

![managed-schema](/全文检索引擎Solr搭建/4-3.png)

在命令台中输入
```
solr restart -p 8983
```

重启服务

![managed-schema](/全文检索引擎Solr搭建/4-4.png)

进入管理界面，选择 my_core --> Dataimport --> Execute 导入数据，可以点击 Refresh Status 查看导入状态

![Dataimport](/全文检索引擎Solr搭建/4-5.png)

## 查询数据

选择 Query

![Dataimport](/全文检索引擎Solr搭建/4-6.png)

点击 Execute Query 查询数据

![Execute Query](/全文检索引擎Solr搭建/4-7.png)