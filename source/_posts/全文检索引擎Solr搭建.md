---
title: 全文检索引擎Solr搭建
date: 2017-10-18 09:43:31
tags: solr
categories : 技术水波文
---

{% fi /全文检索引擎Solr搭建/header-img.png %}

{% cq %} Solr是一个高性能，采用Java5开发，基于Lucene的全文搜索服务器。同时对其进行了扩展，提供了比Lucene更为丰富的查询语言，同时实现了可配置、可扩展并对查询性能进行了优化，并且提供了一个完善的功能管理界面，是一款非常优秀的全文搜索引擎。 {% endcq %}

## 准备

### 配置环境

+ window
+ [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
+ [Solr 7.0.1](https://mirrors.tuna.tsinghua.edu.cn/apache/lucene/solr/7.0.1/solr-7.0.1.zip)
+ oracle

### 目录说明

下载Solr后解压得到目录

```text
├─bin               // 脚本的启动目录
├─contrib           // 第三方 jar 包存放目录
├─dist              // 编译打包后存放目录，即构建后的输出产物存放目录
├─docs              // solr 的 API 文档文档存放目录
├─example           // 示范例子的存放目录，example-DIH 目录下的是一些 solr 索引 core 样例
├─licenses          // solr 相关的一些许可信息
└─server            // solr 服务端工作目录，自带集成 jetty 插件方式启动 solr 服务器
    │ start.jar     // 服务端启动 jar 包
    ├─solr          // solr 搜索引擎工作目录，即 SOLRHOME
    └─solr-webapp   // solr 后台管理页面 webapp
     
```
<!-- more -->
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

![create](/全文检索引擎Solr搭建/3-1.png)

创建成功后，在管理页面选择 Core Admin 可以看到刚创建的 core
      
![Core Admin](/全文检索引擎Solr搭建/3-2.png)

在 SOLRHOME （目录说明中有提到）下会生成 my_core 目录

![my_core](/全文检索引擎Solr搭建/3-3.png)

``` text
├─conf               // 存放core的配置文件
│    solrconfig.xml  // 定义了这个 core 的配置信息
│    start.jar       // 定义索引库的字段及分词器等，这个配置文件是核心文件
└─data               // 存放core的数据，即index-索引文件和log-日志记录
```

## 导入数据

导入 oracle 表 <a href="{% asset_path news.sql %}">news</a> 中的数据

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
此时 my_core 文件结构如下

```text
│  core.properties
│  
├─conf
│  │  data-config.xml
│  │  dataimport.properties
│  │  managed-schema
│  │  params.json
│  │  protwords.txt
│  │  solrconfig.xml
│  │  stopwords.txt
│  │  synonyms.txt
│  └─lang        
├─data        
└─lib
        ojdbc6.jar
```

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

![restart](/全文检索引擎Solr搭建/4-4.png)

进入管理界面，选择 my_core --> Dataimport --> Execute 导入数据，可以点击 Refresh Status 查看导入状态

![Dataimport](/全文检索引擎Solr搭建/4-5.png)

## 查询数据

选择 Query

![Query](/全文检索引擎Solr搭建/5-1.png)

点击 Execute Query 查询数据

![Execute Query](/全文检索引擎Solr搭建/5-2.png)

## 分词器
`smartcn` 、 `IKAnalyzer` 、 `mmseg4j`，选择其中任意一种即可

### smartcn
smartcn 是 Solr 同步发行的一个中文分词包，将 solr-7.0.1/contrib/analysis-extras/lucene-libs/lucene-analyzers-smartcn-7.0.1.jar 拷贝至 %SOLRHOME%/my_core/lib，编辑 %SOLRHOME%/my_core/conf/managed-schema 文件，添加配置

```xml
<!-- smartcn -->
<fieldType name="text_smartcn" class="solr.TextField" positionIncrementGap="0">
  <analyzer type="index">
    <tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/>
  </analyzer>
  <analyzer type="query">
    <tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/>
  </analyzer>
</fieldType>
<!-- smartcn -->
```

### IKAnalyzer

>IKAnalyzer 是一个开源的，基于 java 语言开发的轻量级的中文分词工具包。

将 [ik-analyzer-solr5](https://github.com/EugenePig/ik-analyzer-solr5) 打包后放至 %SOLRHOME%/my_core/lib 下
编辑 %SOLRHOME%/my_core/conf/managed-schema 文件，添加配置

```xml
<!-- IKAnalyzer -->
<fieldType name="text_ik" class="solr.TextField">
  <analyzer type="index" isMaxWordLength="false" class="org.wltea.analyzer.lucene.IKAnalyzer"/>
  <analyzer type="query" isMaxWordLength="true" class="org.wltea.analyzer.lucene.IKAnalyzer"/>
</fieldType>
<!-- IKAnalyzer -->
```

### mmseg4j

>+ mmseg4j 用 Chih-Hao Tsai 的 MMSeg [算法](http://technology.chtsai.org/mmseg) 实现的中文分词器，并实现 lucene 的 analyzer 和 solr 的TokenizerFactory 以方便在Lucene和Solr中使用。
>+ MMSeg 算法有两种分词方法： Simple 和 Complex，都是基于正向最大匹配。
>+ mmseg4j 有三种分词模式 simple | complex | max-word，默认是 max-word。
>+ mmseg4j 的词库强制使用 UTF-8。

下载需要的包
+ [mmseg4j-core-1.10.0.jar](http://central.maven.org/maven2/com/chenlb/mmseg4j/mmseg4j-core/1.10.0/mmseg4j-core-1.10.0.jar)
+ [mmseg4j-solr-2.4.0.jar](http://central.maven.org/maven2/com/chenlb/mmseg4j/mmseg4j-solr/2.4.0/mmseg4j-solr-2.4.0.jar)

放至 %SOLRHOME%/my_core/lib 中
编辑 %SOLRHOME%/my_core/conf/managed-schema 文件，添加配置

```xml
<!-- mmseg4j-->
<fieldType name="text_mmseg4j_complex" class="solr.TextField" positionIncrementGap="100" >
  <analyzer>
    <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="complex"/>
  </analyzer>
</fieldType>
<fieldType name="text_mmseg4j_maxword" class="solr.TextField" positionIncrementGap="100" >
  <analyzer>
    <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="max-word" />
  </analyzer>
</fieldType>
<fieldType name="text_mmseg4j_simple" class="solr.TextField" positionIncrementGap="100" >
  <analyzer>
    <tokenizer class="com.chenlb.mmseg4j.solr.MMSegTokenizerFactory" mode="simple" />
  </analyzer>
</fieldType>
<!-- mmseg4j-->
```
重启服务

### Analysis

分词器配置完成后，选择 Analysis，输入要分词的内容（Field Value），选择字段类型（Analyse Fieldname / FieldType [与配置信息 fieldType name 同步]），点击 Analyse Values 获取分词结果。

![Analyse Values](/全文检索引擎Solr搭建/6-1.png)

编辑 %SOLRHOME%/my_core/conf/managed-schema，修改索引字段类型为新添加分词类型其中任意一种即可，这里我们将 text_general 修改为 text_ik，重启服务后重新导入数据

![Modify Type](/全文检索引擎Solr搭建/6-2.png)

这样我们在对该字段行进搜索时就能达到分词搜索的效果了😛