# 分布式与分布式协调服务Zookeeper

==**Frank				带图答疑QQ：2338247381**==

## 一、课程目标

1. 学习大数据的要点总结自己的方法

   - 大数据知识点重点偏向理论【一定要提前预习】
   - 难度较高：分布式技术
     - 多而杂
     - |
     - 规律

2. ==**重点掌握分布式思想【重点】**==

   - 决定了学习大数据的难度

3. 大数据应用以及学习的技术【了解】

4. ==**Zookeeper【重点】**==

   - 功能以及应用场景
   - 设计思想以及架构
   - 操作比较简单

5. 大数据基础课程

   - Java

     - 大部分的大数据的软件工具都是Java开发的
       - 使用包括运维
     - 自己开发程序处理数据
     - JavaSE：重点
       - 常用的工具类
         - 日期处理
         - 字符串
         - 集合
       - 数据结构以及类型
         - 集合：List、Set、Map
       - 多线程、IO流
       - 设计模式
       - JVM：内存管理、垃圾回收
     - JavaWeb：了解
       - 了解网站开发以及实现的过程

   - MySQL

     - SQL语句

     - 大数据中会用MySQL存储分析处理的结果

     - 大数据中60%的程序开发都是写SQL开发

     - 重点

       - DDL[create/drop/show]/DML[insert/delete/update]

         - 比较简单

       - DQL：select【重点】

         ```
         select 1 from 2 where 3 group by 4 having 5 order by 6 limit 7
         ```

       - 函数

         - 聚和函数
         - 字符串处理函数
         - 日期处理函数
         - 与数据处理有关的函数

   - Linux

     - 所有大数据的技术都是部署在Linux上
     - 重点
       - 学会基本的操作命令即可

   - 问题

     - 技术操作：多练习就可以解决

       - JAVA开发
       - SQL开发
       - Linux操作

     - 如何能将需求转换为技术？

       - 深刻理解每个技术的背景和特点以及原理

     - 不懂就问，切记产生依赖

       

       

       

## 二、分布式概念

### 1、传统应用中存在的问题

- 为什么会有分布式？

  - 大数据
  - 人工智能
  - 区块链
  - JavaEE
  - ……

- 传统Java平台设计

  ![image-20200723095112462](分布式基础与分布式协调服务Zookeeper.assets/image-20200723095112462.png)

  - 问题：如果一旦产生高并发，大数据量的请求的情况下，单台机器是无法处理大量请求的
  - 请求：100万/s
  - 会导致服务故障甚至机器宕机

### 2、分布式解决方案

- **定义**：将多台机器的资源进行整合，构建成一个整体，对外提供服务

  - |
  - 分布式集群

- **思想**：分而治之

  - 对外：用户看到的是一个整体
  - 对内：将一个大的任务进行拆分，拆分成若干个小的任务，将小任务分配给每台内部的机器来实现

  - 问题1：其中任何一条请求，如何决定给哪一台机器进行处理？
    - 必然有一个角色按照一定的规则负责将大的任务变成小的任务分配给每台机器
    - 解决：
      - Nginx：作为反向代理
  - 问题2：如果Nginx宕机怎么解决？
    - 解决
      - 用两个Nginx实现主备

  ![image-20200723101938667](分布式基础与分布式协调服务Zookeeper.assets/image-20200723101938667.png)

- **功能**

  - 解决单台机器节点资源有限，无法实现大量数据高并发的负载问题
  - 分布式的性能是基于高并发多线程的
    - 假设一台机器能处理100万请求：需要花费100ms
      - 1万/ms
    - 五台机器处理100万请求：每台机器1万/ms
      - 每台机器20万请求：20ms

- 应用场景

  - 所有高并发，高负载的场景

  - 大数据

    - 大量数据存储和计算
      - 100TB数据

  - 人工智能

    - 分布式存储和计算

  - 区块链

    - 分布式去中心化的结构

    

### 3、分布式分类

- 第一类：自己本身不是分布式的工具，需要依赖别的工具来构建成一个整体的分布式结构

  - 多台Tomcat + Nginx：分布式网站架构
  - Nginx：实现将大的请求任务划分为多个小的请求，分配给每一台Tomcat

- **==第二类：自己本身就是分布式的，大数据中的技术主要是第二种==**

  - 架构：分布式主从架构
  - 这种工具本身有两种角色
  - 主节点：一般只有一个
    - 负责管理
    - 接客：接受所有客户端的请求
    - 管理从节点：监控所有从节点的死活
    - 任务的分配
  - 从节点 ：一般有多个
    - 负责执行小任务

  ![image-20200723103740849](分布式基础与分布式协调服务Zookeeper.assets/image-20200723103740849.png)

### 4、分布式存在的问题

- ==**问题1：**==第一类分布式架构中，Nginx+Tomcat，==**分布式多台机器的数据一致性问题**==

  - 如果有5台Tomcat，公司后台的数据库原来用的MySQL集群太老了，换一个新的集群

  - 5台Tomcat代码中的JDBC的连接地址都要更换，怎么换？

    - 第一种方案：将代码修改好以后，重新停服发布五台机器新的代码

      - 不可能出现

    - 问题：多台机器要使用共同的一个JDBC连接？

    - 第二种方案：将JDBC地址放在一个公共的地方，让五台机器都请求这个公共的位置获取JDBC地址

      ![image-20200723105507067](分布式基础与分布式协调服务Zookeeper.assets/image-20200723105507067.png)

  - 问题：分布式架构中，如何解决多台机器共享资源问题，如何保证多台机器获取的资源是一致的？

    - 将JDBC配置放入一个properties文件中
      - 希望让整个程序中多个代码文件都可以共享这一个JDBC配置
      - test1.java
      - test2.java
      - |
      - jdbc：properties文件中
    - 将JDBC配置放入一个共享的存储中
      - 多台机器要读取共享存储中的数据，保证每台机器读的都是一致的
      - node1
      - node2
      - node3
      - node4
      - |
      - jdbc：共享存储

- ==**问题2：**==第二类分布式主从架构中：==**主节点单点故障**==

  - 主节点只有1个，如果主节点故障，整个分布式服务将不可用
  - 如何解决？
    - 构建两个或者多个主节点
  - 新的问题1：两个或者多个主节点，谁负责处理请求、管理？
    - 两个主节点，保证只有一个是工作状态，另外一个是备份状态
    - 如果主节点1是工作状态，主节点2是备份状态
    - 主节点1故障，主节点2代替主节点1成为工作状态
  - 新的问题2：如何决定谁是工作的，谁是备份的呢？
    - ==**通过选举来决定**==
  - 新的问题3：如何避免同时出现两个工作的主节点？
    - ==**脑裂问题：**==一个系统出现了两个工作的主节点



## 三、大数据的应用及技术架构

### 1、大数据业务流程

- 本质：通过一系列的大数据的软件工具对数据根据需求进行处理，得到想要的结果【数据中的价值】
  - 存储：Excel
  - 处理：简单的统计分析
    - 函数、宏编程
- 应用
  - 数据分析：分析我们产品的好坏、广告的投放，合理的决策和调整运营策略等等
    - 分析我们今天所有访问我们网站的用户都是如何进入的？
      - 直接输入网站
      - 导航网站
      - 广告：有多少人是通过点击广告进来的
    - 分析广告的投放效果
      - 百度：100万
        - 用户量：50万
        - 注册率：20% = 10万
        - 下单率：50% = 5万
        - 收益率：20万 / 100万 = 20%
      - 谷歌：100万
        - 用户量
        - 注册率
        - 下单率
        - 收益率
      - 搜狗：100万
        - 用户量
        - 注册率
        - 下单率
        - 收益率
  - 用户画像：对用户的数据进行分析，给用户打标签，得到用户的喜好，行为习惯
    - 针对用户的喜好来进行营销转换
    - 收集用户在平台上所有的行为，进行分析
      - 收藏/添加购物车：喜欢
      - 浏览了以后立即关闭：不喜欢
  - 推荐系统：根据用户画像来实现推荐
    - 根据用户的喜好
    - 根据用户的相似度
    - 根据用户之间的关系
      - 收获地址的关联
      - 通讯录
  - 风控系统
    - 金融风控
- 大数据业务流程
  - 数据生成
    - 业务数据
    - 用户行为数据
    - 爬虫数据
    - ……
  - 数据采集
  - 数据存储
  - 数据计算
  - 数据应用

### 2、大数据技术选型

- 数据采集：Flume/Sqoop/Datax/Logstash……
- 数据存储：HDFS、Hbase、Kafka、ES、Redis
  - 分布式存储
- 数据计算：MapReduce+YARN、Spark、Flink、Kylin、Impala
  - 分布式计算
- 辅助技术
  - 协调服务：Zookeeper
  - 调度工具：Azkaban
  - 可视化工具：Hue、Kibana
  - 集群管理：CM

### 3、大数据分布式架构

- ==分布式存储==：将多台机器的磁盘进行整合，构建成了整体

  ![image-20200723113501116](分布式基础与分布式协调服务Zookeeper.assets/image-20200723113501116.png)

  - 优点
    - 解决单台机器容量不足
    - 性能更好

- ==分布式计算==：将多台机器的CPU和内存进行整合，构建成了整体

  ![image-20200723114056294](分布式基础与分布式协调服务Zookeeper.assets/image-20200723114056294.png)

  

  - 优点：性能更快







回顾

1. 分布式是什么？
   - 分布式将多台机器的资源进行整合，变成一个整体，对外提供统一的服务
2. 分布式的功能是什么？
   - 解决单台机器资源不足，无法实现应用
   - 解决单台机器处理的性能问题
3. 分布式的应用场景？
   - 所有需要对大量的数据负载进行处理或者需要高性能的处理场景中
4. 分布式的架构？
   - 第一种：工具本身不是分布式的，依赖于别的工具组合成一个分布式结构
     - 多台Tomcat + Nginx
   - 第二种：工具本身就是分布式结构
     - 分布式主从结构
     - 主：管理
       - 接客
       - 管理从节点
       - 分配任务
     - 从：执行
       - 处理每个小任务
5. 分布式设计思想：分而治之
   - 将一个大的任务拆分成多个小的任务，多台机器并行处理所有的小任务
6. 分布式设计中问题
   - 多台机器一起工作：数据信息的一致性，共同一致性的配置
   - 选举
     - 主节点只有一个 ，存在单点故障
     - 多个主节点：工作的，备份的
7. 大数据中分布式应用
   - 本质：就是使用大数据软件工具实现对数据的处理
   - 业务流程
     - 数据生成
     - 数据采集
     - 数据存储
     - 数据处理
     - 数据应用
   - 技术选型
     - 数据存储：分布式存储
       - 将一个大的存储需求，切分为若干个小的存储，分布式存储在多台机器上
       - HDFS、Hbase、Kafka、ElasticSearch、Redis
     - 数据处理：分布式计算
       - 将一个大的计算需求，切分成若干个小的计算，分布式运行在多台机器上
       - MapReduce、YARN、Spark、Impala、Kylin、Flink





## 四、分布式协调服务Zookeeper

### 1、分布式存在的问题

- 多台机器之间如何保证数据一致性问题
- 多个主节点如何选举得到工作和备份状态

### 2、Zookeeper的功能

- Zookeeper：动物园管理员
  - Hadoop：大象
  - Hive：蜜蜂
  - Hbase：鲸鱼
  - Flink：松鼠
  - ……

- 功能：专门用于解决分布式存在的问题的

- 应用：整个IT领域中
  - 只要是分布式的工具
  - 要么 依赖于Zookeeper帮它解决问题
  - 要么自己实现了类似于Zookeeper的功能
  
- 场景

  - 一致性服务

    ![image-20200723142232001](分布式基础与分布式协调服务Zookeeper.assets/image-20200723142232001.png)

    - 利用zookeeper作为共享存储

  - 选举

    - 为什么要选举？

      - 主从架构中，主节点只有一个，如果主节点故障，就会导致整个分布式服务故障
      - 会有两个主节点
      - 问题：谁是Active【工作状态】，谁是StandBy【备份状态】？
        - 进行选举

    - 如何实现选举？

      - 利用Zookeeper来实现选举

    - 方案一：让所有的主节点【1和2】都到Zookeeper中创建一个同名临时节点【文件】

      - 1和2一起去创建，谁创建成功，谁就是Active，另外一个就是StandBy
        - 1创建成功了，就是Active
        - 2创建失败了，就是StandBy
      - Standby的节点需要监听这个P，如果这个P消失了，说明Active的节点发生故障了
      - StandBy就会创建这个P，就变成了Active的状态

      ![image-20200723143522232](分布式基础与分布式协调服务Zookeeper.assets/image-20200723143522232.png)

      

    - 方案二：让所有的主节点【1和2】都到Zookeeper中创建一个顺序节点，只要给定名称，会自动编号

      - 1和2一起去创建，都创建成功了，序号不一样
        - 1-P0
        - 2-P1
        - 谁的id最小谁就是Active
      - Standby的节点需要监听这个P0，如果这个P0消失了，说明Active的节点发生故障了
      - StandBy就会创建这个P，谁的id最小就变成了Active的状态

    ![image-20200723143900640](分布式基础与分布式协调服务Zookeeper.assets/image-20200723143900640.png)

    - 设计：HA【故障转移、高可用】

      - JavaWeb中：底层存储用MySQL

        - 工作中会有两套MYSQL，存储的数据是一模一样
        - 主MySQL：对外提供读写
        - 备MySQL：负责与主MySQL同步数据

      - Linux：网卡

        - 一台服务器：8个网卡
        - 工作中会用2个网卡：绑定一个IP地址
        - 避免一个网卡故障了，另外一个网卡可以正常的工作

        - 两个电源接口
          - 一个电源接口就可以工作
          - 一个电源：接交流电源
          - 另一个电源：接备份UPS

  - 注册中心与命名服务

    - 注册中心：所有主节点都会在Zookeeper中进行注册，然后才能参加选举

      ![image-20200723145205415](分布式基础与分布式协调服务Zookeeper.assets/image-20200723145205415.png)

      

    - 命名服务：将所有主节点封装成一个整体

      - 只要请求访问主节点
      - 根据当前active的状态，返回对应的active的地址

      ![image-20200723145301802](分布式基础与分布式协调服务Zookeeper.assets/image-20200723145301802.png)

      

  - 分布式锁

    - 只会有一个Active的状态

  - 订阅发布中心

    - 监听zookeeper中的数据，只要修改了 Zookeeper中的信息
    - 分布式中所有机器都可以读取到最新的信息

- ==**如何 理解Zookeeper这个工具**==

  - 类似于一个文件系统：可以存储数据
  - 通过存储数据的方式来解决了分布式中存在的问题
  - 学习
    - 操作：实现数据读写
    - 难点
      - 功能以及应用场景
      - zookeeper怎么自己不出问题

### 3、Zookeeper的架构

- 本质：zookeeper就是一个特殊的分布式文件系统

- 区别 ：与普通文件系统区别

  - Zookeeper中不适合存储大量数据
  - 单个数据不允许超过1M
  - 存储关键性数据
    - 统一化的配置
    - 关键性信息

- 架构：分布式主从架构

  - 主：Leader

    - 负责接受所有读写请求

  - 从：Follower

    - 负责接收所有的读的请求
    - 如果接受到写请求会转发给Leader来实现

  - ==**主和从是公平节点**==：与别的分布式主从架构不一样

    - 所有的主和从都可以接受客户端的请求
      - 主：实现读和写
      - 从：实现读，写的操作只能由主节点来做，会自动转发给主节点
      - 为什么这么设计呢？
        - 传统：只有主节点可以接受请求，主节点存在单点故障
          - 解决：用zookeeper实现多个主节点的选举
        - zookeeper必须自己解决，如果主节点故障，其他节点也可以提供正常的读
    - 主节点和从节点上存储的内容是一模一样的

    ![image-20200723152453576](分布式基础与分布式协调服务Zookeeper.assets/image-20200723152453576.png)

    

  - ==**问题1：如果主节点故障，Zookeeper集群就不可以写入了**==

    - 如果Leader故障，Zookeeper会让所有的Follower重新选举一个新的Leader

    - 与上面讲到的帮别人选举不一样
      - 这里是Zookeeper自己内部选举

  - ==**问题2：为什么只让Leader来实现写，其他的Follower中的数据怎么与Leader保持一致？**==

    - 所有写请求都只能由Leader来执行，Follower即使接受了写，也会转发给Leader

      ![image-20200723152657557](分布式基础与分布式协调服务Zookeeper.assets/image-20200723152657557.png)

    - Leader会将这个写的请求广播写入所有节点

      ![image-20200723152808341](分布式基础与分布式协调服务Zookeeper.assets/image-20200723152808341.png)

    - 如果你想要所有的节点的数据都保持一致，只能由一个人来操作这个过程

      - leader来广播：保证一致性问题
      - 如果超过半数写入成功，整体写入就成功了
      - 如果超过半数写入失败，整体写入失败了

    - 为什么要让所有节点的数据都一样呢？

      - 为了避免任何一台机器都可能故障，只要有一台机器存活，就可以对外提供服务

### 4、Zookeeper中的概念

- 类似于文件系统

  - 正常的文件系统

    - 目录节点：不允许存储，但是可以拥有子节点
      - 可以在目录下面创建子目录或者文件
      - 但是不能编辑目录的内容
    - 文件节点：允许存储，但是不能拥有子节点
      - 可以编辑文件的内容
      - 文件下面不能创建目录或者文件

    ![image-20200723154226884](分布式基础与分布式协调服务Zookeeper.assets/image-20200723154226884.png)

  - Zookeeper中的节点

    - 数据节点znode：拥有目录和文件的特点
      - 既可以有子节点，也可以存储内容
    - 结构：树形结构
      - 第一级数据节点：/



## 五、分布式部署

![image-20200723154846690](分布式基础与分布式协调服务Zookeeper.assets/image-20200723154846690.png)



### 1、准备环境

- 准备三台机器

- 配置好IP以及主机名映射

- 关闭防火墙及seilnux

- 安装好JDK

- SSH免秘钥登录，三台机器之间互相可以免秘钥登录

- 时钟同步

  

### 2、下载安装

- Apache版本： http://archive.apache.org/dist/ 

- Cloudera版本： http://archive.cloudera.com/cdh5/cdh/5/ 

- 为了后面很多个大数据软件放在一起，不需要考虑兼容性的问题，选择Cloudera版本

- **第一步**：上传到第一台机器的/export/software目录下

  ```
  cd /export/software/
  rz
  ```

  - 如果没有rz命令：command not found

    ```
    yum install -y lrzsz
    ```

  ![image-20200723155322413](分布式基础与分布式协调服务Zookeeper.assets/image-20200723155322413.png)

  

- 第二步：解压安装：/export/server

  ```shell
   cd /export/software/
   tar -zxvf zookeeper-3.4.5-cdh5.14.0.tar.gz -C /export/servers/
  ```

- 第三步：检查是否解压完成

  ```
  cd /export/servers/
  cd zookeeper-3.4.5-cdh5.14.0/
  ll
  ```
  
  ![image-20200723161043114](分布式基础与分布式协调服务Zookeeper.assets/image-20200723161043114.png)
  
  - bin：放的是程序命令，客户端命令、服务端命令
  - conf：放的是配置文件
  - lib：放的是依赖包

### 3、配置

- 节点规划：构建分布式zookeeper

  - node-01
  - node-02
  - node-03

- 先配置第一台机器的zookeeper

  - 先切换到zookeeper的家目录中

    ```shell
    cd /export/servers/zookeeper-3.4.5-cdh5.14.0/
    ```

  - 复制一份配置文件
  
    ```shell
    cp conf/zoo_sample.cfg conf/zoo.cfg
    ```

  - 创建一个目录，用于存储zookeeper中的数据

    ```shell
    mkdir /export/servers/zookeeper-3.4.5-cdh5.14.0/zkData
    ```

  - 修改zoo.cfg：注意改成自己的主机名

    ```shell
    vim  conf/zoo.cfg
    #修改第12行
    dataDir=/export/servers/zookeeper-3.4.5-cdh5.14.0/zkData
    
    #在文件的最后添加以下的行，主机名要改成自己的，用于定义zookeeper集群中有哪些机器
    server.1=node-01:2888:3888
    server.2=node-02:2888:3888
    server.3=node-03:2888:3888
    ```
  
  
  
  - 创建每一台zookeeper自己的myid
  
    - 第一台机器的myid就是1，创建一个文件，里面就写1
  
    ```shell
  vim /export/servers/zookeeper-3.4.5-cdh5.14.0/zkData/myid
    1
    ```
  
- 将第一台机器上的zookeeper发送到第二台机器和第三台机器

  ```shell
  cd /export/servers/
  scp -r zookeeper-3.4.5-cdh5.14.0 node-02:$PWD
  scp -r zookeeper-3.4.5-cdh5.14.0 node-03:$PWD
  ```

- 修改第二台机器的myid为2

  ![image-20200723161958832](分布式基础与分布式协调服务Zookeeper.assets/image-20200723161958832.png)

- 修改第三台机器的myid为3

  ![image-20200723162023796](分布式基础与分布式协调服务Zookeeper.assets/image-20200723162023796.png)
  
  

### 4、启动服务

- 启动Zookeeper的服务端

  - 每台Zookeeper都要执行

    ```
    cd /export/servers/zookeeper-3.4.5-cdh5.14.0/
    bin/zkServer.sh start
    
    bin/zkServer.sh start | status | stop
    ```

  - 检查每台机器有没有对应的zookeeper的进程

    ![image-20200723162225384](分布式基础与分布式协调服务Zookeeper.assets/image-20200723162225384.png)

    

- 查看三台机器的zookeeper的状态

  - Zookeeper是有主动之分

    ```
    cd /export/servers/zookeeper-3.4.5-cdh5.14.0/
    bin/zkServer.sh status
    ```

    ![image-20200723162319494](分布式基础与分布式协调服务Zookeeper.assets/image-20200723162319494.png)
    
    ![image-20200723162335122](分布式基础与分布式协调服务Zookeeper.assets/image-20200723162335122.png)
    
    ![image-20200723162354738](分布式基础与分布式协调服务Zookeeper.assets/image-20200723162354738.png)
    
    

- 关闭Zookeeper的服务端

  ```
  bin/zkServer.sh stop
  ```



- 出现问题怎么办？

  - step1：哪台机器有问题，就找到那台机器上zookeeper的日志文件

    ```
    /export/servers/zookeeper-3.4.5-cdh5.14.0/zookeeper.out
    ```

    - 如果你启动了zookeeper，找不到这个文件

      ```
      find /  -name  'zookeeper.out'
      ```

      

  - step2：查看这个文件

    ```
    tail -100f  /export/servers/zookeeper-3.4.5-cdh5.14.0/zookeeper.out
    ```

  - step3：找到抛出Exception的地方，截图发给老师

    - 截图：截全一点，建议截全屏

    - 不要就截图那么两三行

      

## 六、Zookeeper节点操作

### 1、理解Zookeeper的使用

- 类似于Linux文件系统：通过命令实现节点的增删改查

### 2、终端命令

- 服务端开放连接端口：2181

- 客户端：bin/zkCli.sh

  - 默认连接客户端所在机器的zookeeper的服务

  - 指定连接到某一台zookeeper

    ```
    -server  hostname:port
    ```

- 指定某台zookeeper服务实现连接

  ```
  bin/zkCli.sh  -server node-02:2181
  ```

- 查看帮助文档

  ```
  help
  ```

- 退出客户端

  ```
  quit
  ```

- 列举当前所有节点

  ```
  ls  /
  ```

  - 没有切换，只能通过/一级一级的访问

- 创建一个节点

  - 语法：create [-s] [-e] path data

  - 测试

    ```
    create  /itcast  heima
    ```

    ![image-20200723163944730](分布式基础与分布式协调服务Zookeeper.assets/image-20200723163944730.png)

    ```
    create /itcast/hadoop  hadoop
    ```

    ![image-20200723164036116](分布式基础与分布式协调服务Zookeeper.assets/image-20200723164036116.png)

- 查看某个节点的数据内容

  - 语法：get path [watch]

  - 测试

    ```
    get /itcast
    ```

    ![image-20200723164222617](分布式基础与分布式协调服务Zookeeper.assets/image-20200723164222617.png)

    ```
    get /itcast/hadoop
    ```

    ![image-20200723164238921](分布式基础与分布式协调服务Zookeeper.assets/image-20200723164238921.png)

- 修改节点的内容

  - 语法：set path data [version]

  - 测试

    ```
    set  /itcast/hadoop  hive
    ```

    ![image-20200723164414961](分布式基础与分布式协调服务Zookeeper.assets/image-20200723164414961.png)

- 删除节点

  - 语法：rmr path

  - 测试

    ```
    rmr /itcast/hadoop
    ```

    ![image-20200723164509243](分布式基础与分布式协调服务Zookeeper.assets/image-20200723164509243.png)

    

### 3、Zookeeper的节点类型

- 临时节点：如果客户端断开，临时节点会自动消失
- 顺序节点：创建的节点是有序

### 4、Zookeeper中的Watch机制

- 监听节点的动作







## 附录一：Zookeeper的Maven依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.apache.curator</groupId>
		<artifactId>curator-framework</artifactId>
		<version>2.12.0</version>
	</dependency>
	<dependency>
		<groupId>org.apache.curator</groupId>
		<artifactId>curator-recipes</artifactId>
		<version>2.12.0</version>
	</dependency>
	<dependency>
		<groupId>com.google.collections</groupId>
		<artifactId>google-collections</artifactId>
		<version>1.0</version>
	</dependency>
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.11</version>
	</dependency>
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>slf4j-simple</artifactId>
		<version>1.7.25</version>
	</dependency>
</dependencies>
```

