# 本资源由微信公众号：光明顶一号，分享
Neo4j 对于大多数人来说，可能是比较陌生的。其实，Neo4j 是一个图形数据库，就像传统的关系数据库中的 Oracel 和
MySQL一样，用来持久化数据。Neo4j 是最近几年发展起来的新技术，属于 NoSQL 数据库中的一种。

本文主要从 Neo4j 为什么被用来做知识图谱，Neo4j 的简单安装，在 Neo4j 浏览器中创建节点和关系，Neo4j 的 Python 接口操作以及用
Neo4j 构建一个简单的农业知识图谱五个方面来讲。

### Neo4j 为什么被用来做知识图谱

从第19课《知识挖掘与知识图谱概述》中，我们已经明白，知识图谱是一种基于图的数据结构，由节点和边组成。其中节点即实体，由一个全局唯一的 ID
标示，关系（也称属性）用于连接两个节点。通俗地讲，知识图谱就是把所有不同种类的信息连接在一起而得到一个关系网络，提供了从“关系”的角度去分析问题的能力。

而 Neo4j 作为一种经过特别优化的图形数据库，有以下优势：

  * **数据存储** ：不像传统数据库整条记录来存储数据，Neo4j 以图的结构存储，可以存储图的节点、属性和边。属性、节点都是分开存储的，属性与节点的关系构成边，这将大大有助于提高数据库的性能。

  * **数据读写** ：在 Neo4j 中，存储节点时使用了 `Index-free Adjacency` 技术，即每个节点都有指向其邻居节点的指针，可以让我们在时间复杂度为 O(1) 的情况下找到邻居节点。另外，按照官方的说法，在 Neo4j 中边是最重要的，是 `First-class Entities`，所以单独存储，更有利于在图遍历时提高速度，也可以很方便地以任何方向进行遍历。

  * **资源丰富** ：Neo4j 作为较早的一批图形数据库之一，其文档和各种技术博客较多。

  * **同类对比** ：Flockdb 安装过程中依赖太多，安装复杂；Orientdb，Arangodb 与 Neo4j 做对比，从易用性来说都差不多，但是从稳定性来说，neo4j 是最好的。

综合上述以及因素，我认为 Neo4j 是做知识图谱比较简单、灵活、易用的图形数据库。

### Neo4j 的简单安装

Neo4j 是基于 Java 的图形数据库，运行 Neo4j 需要启动 JVM 进程，因此必须安装 Java SE 的 JDK。从 Oracle
官方网站下载 [Java SE
JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)，选择版本
JDK8 以上版本即可。

下面简单介绍下 Neo4j 在 Linux 和 Windows
的安装过程。首先去[官网](https://neo4j.com/download/other-
releases/#releases)下载对应版本。解压之后，Neo4j 应用程序有如下主要的目录结构：

  * bin 目录：用于存储 Neo4j 的可执行程序；
  * conf 目录：用于控制 Neo4j 启动的配置文件；
  * data 目录：用于存储核心数据库文件；
  * plugins 目录：用于存储 Neo4j 的插件。

#### Linux 系统下的安装

通过 tar 解压命令解压到一个目录下：

    
    
    tar -xzvf neo4j-community-3.3.1-unix.tar.gz
    

然后进入 Neo4j 解压目录：

    
    
    cd /usr/local/neo4j/neo4j-community-3.1.0
    

通过启动命令，可以实现启动、控制台、停止服务：

    
    
    bin/neo4j  start/console/stop（启动/控制台/停止）
    

通过 `cypher-shell` 命令，可以进入命令行：

    
    
    bin/cypher-shell
    

#### Windows 系统下的安装

启动 DOS 命令行窗口，切换到解压目录 bin 下，以管理员身份运行命令，分别为启动服务、停止服务、重启服务和查询服务的状态：

    
    
    bin\neo4j start
    bin\neo4j stop
    bin\neo4j restart
    bin\neo4j status
    

把 Neo4j 安装为服务（Windows Services），可通过以下命令：

    
    
    bin\neo4j install-service
    bin\neo4j uninstall-service
    

Neo4j 的配置文档存储在 conf 目录下，Neo4j 通过配置文件 neo4j.conf
控制服务器的工作。默认情况下，不需要进行任意配置，就可以启动服务器。

下面我们在 Windows 环境下启动 Neo4j：

![enter image description
here](https://images.gitbook.cn/a76e8bd0-a0fd-11e8-8279-afd456e25292)

Neo4j 服务器具有一个集成的浏览器，在一个运行的服务器实例上访问： http://localhost:7474/，打开浏览器，显示启动页面：

![enter image description
here](https://images.gitbook.cn/d39a6080-a0fd-11e8-aa70-d319955d5dbe)

默认的 Host 是 `bolt://localhost:7687`，默认的用户是 neo4j，其默认的密码是 neo4j，第一次成功登录到 Neo4j
服务器之后，需要重置密码。访问 Graph Database 需要输入身份验证，Host 是 Bolt 协议标识的主机。登录成功后界面：

![enter image description
here](https://images.gitbook.cn/456d52d0-a0fe-11e8-aa70-d319955d5dbe)

到此为止，我们就完成了 Neo4j 的基本安装过程，更详细的参数配置，可以参考官方文档。

### 在 Neo4j 浏览器中创建节点和关系

下面，我们简单编写 Cypher 命令，Cypher 命令可以通过 [Neo4j
教程](https://www.w3cschool.cn/neo4j/)学习，在浏览器中通过 Neo4j 创建两个节点和两个关系。

在 `$` 命令行中，编写 Cypher 脚本代码，点击 Play 按钮完成创建，依次执行下面的语句：

    
    
    CREATE (n:Person { name: 'Andres', title: 'Developer' }) return n;
    

作用是创建一个 Person，并包含属性名字和职称。

![enter image description
here](https://images.gitbook.cn/eb544300-a100-11e8-8279-afd456e25292)

下面这条语句也创建了一个 Person 对象，属性中只是名字和职称不一样。

    
    
    CREATE (n:Person { name: 'Vic', title: 'Developer' }) return n;
    

紧接着，通过下面两行命令进行两个 Person 的关系匹配：

    
    
    match(n:Person{name:"Vic"}),(m:Person{name:"Andres"}) create (n)-[r:Friend]->(m) return r;
    
    match(n:Person{name:"Vic"}),(m:Person{name:"Andres"}) create (n)<-[r:Friend]-(m) return r;
    

最后，在创建完两个节点和关系之后，查看数据库中的图形：

    
    
    match(n) return n;
    

如下图，返回两个 Person 节点，以及其关系网，两个 Person 之间组成 Friend 关系：

![enter image description
here](https://images.gitbook.cn/fbbae950-a101-11e8-82f2-9d33eeaf0679)

### Neo4j 的 Python 操作

既然 Neo4j 作为一个图库数据库，那我们在项目中使用的时候，必然不能通过上面那种方式完成任务，一般都要通过代码来完成数据的持久化操作。其中，对于
Java 编程者来说，可通过 [Spring Data Neo4j](https://docs.spring.io/spring-
data/neo4j/docs/current/reference/html/) 达到这一目的。

而对于 Python 开发者来说，Py2neo 库也可以完成对 Neo4j 的操作，操作过程如下。

首先 安装 Py2neo。Py2neo 的安装过程非常简单，在命令行通过下面命令即可安装成功。

    
    
    pip install py2neo
    

安装好之后，我们来看一下简单的图关系构建，看下面代码：

    
    
    from py2neo.data import Node, Relationship
    a = Node("Person", name="Alice")
    b = Node("Person", name="Bob")
    ab = Relationship(a, "KNOWS", b)
    

第一行代码，首先引入 Node 和 Relationship 对象，紧接着，创建 a 和 b 节点对象，最后一行匹配 a 和 b
之间的工作雇佣关系。接着来看看 ab 对象的内容是什么：

    
    
    print(ab)
    

通过 print 打印出 ab 的内容：

    
    
    (Alice)-[:KNOWS {}]->(Bob)
    

通过这样，就完成了 Alice 和 Bob 之间的工作关系，如果有多组关系将构建成 Person 之间的一个关系网。

了解更多 Py2neo 的使用方法，建议查看官方文档。

### 用 Neo4j 构建一个简单的农业知识图谱

我们来看一个基于开源语料的简单农业知识图谱，由于过程比较繁杂，数据和知识图谱数据预处理过程这里不再赘述，下面，我们重点看基于 Neo4j
来创建知识图谱的过程。

整个过程主要包含以下步骤：

  * 环境准备
  * 语料准备
  * 语料加载
  * 知识图谱查询展示

#### Neo4j 环境准备。

根据上面对 Neo4j 环境的介绍，这里默认你已经搭建好 Neo4j 的环境，并能正常访问，如果没有环境，请自行搭建好 Neo4j 的可用环境。

#### 数据语料介绍。

本次提供的语料是已经处理好的数据，包含6个 csv 文件，文件内容和描述如下。

  * attributes.csv：文件大小 2M，内容是通过互动百科页面得到的部分实体的属性，包含字段：Entity、AttributeName、Attribute，分别表示实体、属性名称、属性值。文件前5行结构如下：

    
    
    Entity,AttributeName,Attribute
    密度板,别名,纤维板
    葡萄蔓枯病,主要为害部位,枝蔓
    坎德拉,性别,男
    坎德拉,国籍,法国
    坎德拉,场上位置,后卫
    

  * `hudong_pedia.csv`：文件大小 94.6M，内容是已经爬好的农业实体的百科页面的结构化数据，包含字段：title、url、image、openTypeList、detail、baseInfoKeyList、baseInfoValueList，分别表示名称、百科 URL 地址、图片、分类类型、详情、关键字、依据来源。文件前2行结构如下：

    
    
    "title","url","image","openTypeList","detail","baseInfoKeyList","baseInfoValueList"
    "菊糖","http://www.baike.com/wiki/菊糖","http://a0.att.hudong.com/72/85/20200000013920144736851207227_s.jpg","健康科学##分子生物学##化学品##有机物##科学##自然科学##药品##药学名词##药物中文名称列表","[药理作用] 诊断试剂 人体内不含菊糖，静注后，不被机体分解、结合、利用和破坏，经肾小球滤过，通过测定血中和尿中的菊糖含量，可以准确计算肾小球的滤过率。菊糖广泛存在于植物组织中,约有3.6万种植物中含有菊糖,尤其是菊芋、菊苣块根中含有丰富的菊糖[6,8]。菊芋(Jerusalem artichoke)又名洋姜,多年生草本植物,在我国栽种广泛,其适应性广、耐贫瘠、产量高、易种植,一般亩产菊芋块茎为2 000～4 000 kg,菊芋块茎除水分外,还含有15%～20%的菊糖,是加工生产菊糖及其制品的良好原料。","中文名：","菊糖"
    "密度板","http://www.baike.com/wiki/密度板","http://a0.att.hudong.com/64/31/20200000013920144728317993941_s.jpg","居家##巧克力包装##应用科学##建筑材料##珠宝盒##礼品盒##科学##糖果盒##红酒盒##装修##装饰材料##隔断##首饰盒","密度板（英文：Medium Density Fiberboard (MDF)）也称纤维板，是以木质纤维或其他植物纤维为原料，施加脲醛树脂或其他适用的胶粘剂制成的人造板材。按其密度的不同，分为高密度板、中密度板、低密度板。密度板由于质软耐冲击，也容易再加工，在国外是制作家私的一种良好材料，但由于国家关于高密度板的标准比国际标准低数倍，所以，密度板在中国的使用质量还有待提高。","中文名：##全称：##别名：##主要材料：##分类：##优点：","密度板##中密度板纤维板##纤维板##以木质纤维或其他植物纤维##高密度板、中密度板、低密度板##表面光滑平整、材质细密性能稳定"
    

  * `hudong_pedia2.csv`：文件大小 41M，内容结构和 `hudong_pedia.csv` 文件保持一致，只是增加数据量，作为 `hudong_pedia.csv` 数据的补充。

  * `new_node.csv`：文件大小 2.28M，内容是节点名称和标签，包含字段：title、lable，分别表示节点名称、标签，文件前5行结构如下：

    
    
    title,lable
    药物治疗,newNode
    膳食纤维,newNode
    Boven Merwede,newNode
    亚美尼亚苏维埃百科全书,newNode
    

  * `wikidata_relation.csv`：文件大小 1.83M，内容是实体和关系，包含字段 HudongItem1、relation、HudongItem2，分别表示实体1、关系、实体2，文件前5行结构如下：

    
    
    HudongItem1,relation,HudongItem2
    菊糖,instance of,化合物
    菊糖,instance of,多糖
    瓦尔,instance of,河流
    菊糖,subclass of,食物
    瓦尔,origin of the watercourse,莱茵河
    

  * `wikidata_relation2.csv`：大小 7.18M，内容结构和 `wikidata_relation.csv` 一致，作为 `wikidata_relation.csv` 数据的补充。

#### 语料加载。

语料加载，利用 Neo4j 的 `LOAD CSV WITH HEADERS FROM...` 功能进行加载，具体操作过程如下。

首先，依次执行以下命令：

    
    
    // 将hudong_pedia.csv 导入
    LOAD CSV WITH HEADERS  FROM "file:///hudong_pedia.csv" AS line  
    CREATE (p:HudongItem{title:line.title,image:line.image,detail:line.detail,url:line.url,openTypeList:line.openTypeList,baseInfoKeyList:line.baseInfoKeyList,baseInfoValueList:line.baseInfoValueList})
    

执行成功之后，控制台显示成功：

![enter image description
here](https://images.gitbook.cn/160bd4e0-a1ca-11e8-aa70-d319955d5dbe)

上面这张图，表示数据加载成功，并显示加载的数据条数和耗费的时间。

    
    
    // 新增了hudong_pedia2.csv
    LOAD CSV WITH HEADERS  FROM "file:///hudong_pedia2.csv" AS line  
    CREATE (p:HudongItem{title:line.title,image:line.image,detail:line.detail,url:line.url,openTypeList:line.openTypeList,baseInfoKeyList:line.baseInfoKeyList,baseInfoValueList:line.baseInfoValueList})
    
    // 创建索引
    CREATE CONSTRAINT ON (c:HudongItem)
    ASSERT c.title IS UNIQUE
    

以上命令的意思是，将 `hudong_pedia.csv` 和 `hudong_pedia2.csv` 导入 Neo4j 作为结点，然后对 titile
属性添加 UNIQUE（唯一约束/索引）。

**注意：** 如果导入的时候出现 Neo4j JVM 内存溢出错误，可以在导入前，先把 Neo4j 下的 `conf/neo4j.conf` 中的
`dbms.memory.heap.initial_size` 和 `dbms.memory.heap.max_size`
调大点。导入完成后再把值改回去即可。

下面继续执行数据导入命令：

    
    
    // 导入新的节点
    LOAD CSV WITH HEADERS FROM "file:///new_node.csv" AS line
    CREATE (:NewNode { title: line.title })
    
    //添加索引
    CREATE CONSTRAINT ON (c:NewNode)
    ASSERT c.title IS UNIQUE
    
    //导入hudongItem和新加入节点之间的关系
    LOAD CSV  WITH HEADERS FROM "file:///wikidata_relation2.csv" AS line
    MATCH (entity1:HudongItem{title:line.HudongItem}) , (entity2:NewNode{title:line.NewNode})
    CREATE (entity1)-[:RELATION { type: line.relation }]->(entity2)
    
    LOAD CSV  WITH HEADERS FROM "file:///wikidata_relation.csv" AS line
    MATCH (entity1:HudongItem{title:line.HudongItem1}) , (entity2:HudongItem{title:line.HudongItem2})
    CREATE (entity1)-[:RELATION { type: line.relation }]->(entity2)
    

执行完这些命令后，我们导入 `new_node.csv` 新节点，并对 titile 属性添加 UNIQUE（唯一约束/索引），导入
`wikidata_relation.csv` 和 `wikidata_relation2.csv`，并给节点之间创建关系。

紧接着，继续导入实体属性，并创建实体之间的关系：

    
    
    LOAD CSV WITH HEADERS FROM "file:///attributes.csv" AS line
    MATCH (entity1:HudongItem{title:line.Entity}), (entity2:HudongItem{title:line.Attribute})
    CREATE (entity1)-[:RELATION { type: line.AttributeName }]->(entity2);
    
    LOAD CSV WITH HEADERS FROM "file:///attributes.csv" AS line
    MATCH (entity1:HudongItem{title:line.Entity}), (entity2:NewNode{title:line.Attribute})
    CREATE (entity1)-[:RELATION { type: line.AttributeName }]->(entity2);
    
    LOAD CSV WITH HEADERS FROM "file:///attributes.csv" AS line
    MATCH (entity1:NewNode{title:line.Entity}), (entity2:NewNode{title:line.Attribute})
    CREATE (entity1)-[:RELATION { type: line.AttributeName }]->(entity2);
    
    LOAD CSV WITH HEADERS FROM "file:///attributes.csv" AS line
    MATCH (entity1:NewNode{title:line.Entity}), (entity2:HudongItem{title:line.Attribute})
    CREATE (entity1)-[:RELATION { type: line.AttributeName }]->(entity2)  
    

这里注意，建索引的时候带了 label，因此只有使用 label 时才会使用索引，这里我们的实体有两个 label，所以一共做 `2*2=4`
次。当然也可以建立全局索引，即对于不同的 label 使用同一个索引。

以上过程，我们就完成了语料加载，并创建了实体之间的关系和属性匹配，下面我们来看看 Neo4j 图谱关系展示。

#### 知识图谱查询展示

最后通过 cypher 语句查询来看看农业图谱展示。

首先，展示 HudongItem 实体，执行如下命令：

    
    
    MATCH (n:HudongItem) RETURN n LIMIT 25
    

对 HudongItem 实体进行查询，返回结果的25条数据，结果如下图：

![enter image description
here](https://images.gitbook.cn/2608f910-a1cd-11e8-a041-3fe38238552b)

接着，展示 NewNode 实体，执行如下命令：

    
    
    MATCH (n:NewNode) RETURN n LIMIT 25
    

对 NewNode 实体进行查询，返回结果的25条数据，结果如下图：

![enter image description
here](https://images.gitbook.cn/2ee13980-a1cd-11e8-aeb1-374a1fc4569b)

之后，展示 RELATION 直接的关系，执行如下命令：

    
    
    MATCH p=()-[r:RELATION]->() RETURN p LIMIT 25
    

展示实体属性关系，结果如下图：

![enter image description
here](https://images.gitbook.cn/36cbbe90-a1cd-11e8-aa70-d319955d5dbe)

### 总结

本节内容到此结束，回顾下整篇文章，主要讲了以下内容：

  1. 解释了 Neo4j 被用来做知识图谱的原因；
  2. Neo4j 的简单安装以及在 Neo4j 浏览器中创建节点和关系；
  3. Neo4j 的 Python 接口操作及使用；
  4. 从五个方面讲解了如何使用 Neo4j 构建一个简单的农业知识图谱。

最后，强调一句，知识图谱未来会通过自然语言处理技术和搜索技术结合应用会越来越广，工业界所出的地位也会越来越重要。

**参考文献及推荐阅读**

  1. [Neo4j 官网](https://neo4j.com/download/other-releases/#releases)
  2. [The Neo4j Developer Manual](https://neo4j.com/docs/operations-manual/3.2/)
  3. [Neo4j 教程](https://www.w3cschool.cn/neo4j/)
  4. [Py2neo——Neo4j & Python 的配合使用](https://www.jianshu.com/p/a2497a33390f)


## 更多资源下载交流请加微信：Morstrong
# 本资源由微信公众号：光明顶一号，分享,一个用技术共享精品付费资源的公众号！