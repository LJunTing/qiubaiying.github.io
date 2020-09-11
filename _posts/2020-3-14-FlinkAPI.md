---
layout:     post                    # 使用的布局（不需要改）
title:      Filnk            # 标题 
subtitle:   flink简单学习 #副标题
date:       2020-3-13              # 时间
author:     JT                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Flink
    - 大数据
    - Scala
    - 实时数据
---
[TOC]

#### 介绍
```
是一个框架和分布式的处理引擎,用于对有界和无界数据流的状态计算

```
#### 安装配置
```
[下载](https://archive.apache.org/dist/flink/flink-1.7.2/)
# 配置:
    集群模式
    standalone 不需要过多配置
        slaves 子节点配置   
    yarn 
        启动yarn-session
        -n taskmanager数量
        -s 每个task的slot数量
        -jm jobmanager的内存(MB)
        -tm taskmanager的内存(MB)
        -nm yarn appname
        -d 后台执行
         1. 启动yarn-session
          ./yarn-session.sh -n 2 -s 6 -jm 1024 -tm 1024 -na test -d
         2. flink on yarn  
         ./flink run -m yarn-cluster -c 类的包名 -p 1 /../*.jar --host mini1 --port 9999
    Kubernetes 部署
        1. 搭建Kubernetes 集群
        2. 配置各组件的yaml文件
            在k8s上构建flink session cluster ,需要将flink集群的组件对应的docker镜像分别在k8s上启动,包括job,task ,jobmanagerservice三个镜像服务.每个镜像服务够可以从中央镜像仓库中获取
        3. 启动flinksession cluster
            kubectl create -f jobmanager-service.yaml
            kubectl create -f jobmanager-deployment.yaml
            kubectl create -f taskmanager-deployment.yaml
        4. 访问flinkUi页面
            
命令行一些查看:
    1. 运行提交
        ./flink run -c 类的包名 -p 1 /../*.jar --host mini1 --port 9999
    2. 查看job
        ./flink list
    3. 取消job
        ./flink cancle job_id
    4. 
        
    5. 

```

#### 框架
```
# 架构
1. 运行时的组件
    a. 作业管理器
    b. 任务管理器 
    c. 资源管理器
    d. 分发器(Dispacher)

2. 任务提交流程


3. 任务调度原理



# slot 和 并行度
    task里slot的个数* task 个数  并行度最优
    如果要写出文件要把并行度调低,不然多个任务操作一个文件内容会乱
    把sink 的算子.setParallelism(1)设置为1
# 算子的传输数据的形式
    one-to-one      一对一
    redistributing   重新分区
# 任务链(Operator chain)
    通过本地转发local forward 的方式进行链接
    并行度相同,并且是one-to-one 操作,两个条件缺一不可
# 任务链 分割操作
    .disableChaining()  禁掉任务链操作 独立的一个任务
    .startNewChain()    和前面任务链断开 ,开启新的任务链
    environment.disableOperatorChaining() 全局禁用任务链
    
# 与其他主流区别 
 flink和spark streamming区别
    a. flink流式, spark微批处理
    b. 模型
        数据模型
            1. spark采用RDD模型,streamming的Dstream 一组组小数据RDD
            2. Flink 基本数据模型是数据流,以及事件序列
        运行时架构:
            1. spark是微批计算,将DGA划分不同的stage,计算完这个才执行下个
            2. Flink是标准的流执行模式,一个事件在一个节点处理完后可以直接发到下个节点进行处理

```

#### 特点
```
1. 支持事件时间和处理时间
2. 精确一次的状态保证数据的一直性
3. 低延迟,每秒处理百万事件,毫秒级延迟
4. 能与众多存储系统连接
5. 高可用,动态扩展,全天运行

```
#### scala API
```

### 简单入手
# 文件
    //创建一个批处理的执行环境
    val environment: ExecutionEnvironment = ExecutionEnvironment.getExecutionEnvironment
    var path: String = "/Users/ljt/Downloads/apache-hive-2.3.6-src/flink/src/main/resources/hello"

    val lines = environment.readTextFile(path)
    val word  = lines.flatMap(x=>x.split(" "))
      .map((_,1))
      .groupBy(0)
      .sum(1)
      .print()
      
### StreamExecutionEnvironment 流式执行环境
# socket 
    //解析args参数
    val tool: ParameterTool = ParameterTool.fromArgs(args)
    val host: String = tool.get("host")
    val port: Int = tool.getInt("port")
    //创建执行环境
    val environment: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    val line: DataStream[String] = environment.socketTextStream(host, port)
    import org.apache.flink.streaming.api.scala.createTypeInformation
    line.flatMap(_.split(" "))
      .map((_, 1))
      .keyBy(0)
      .sum(1)
      .print()
    //任务执行
    environment.execute("streamming wordcont test")

#### API

# Environment
    一般全局的设置
  StreamExecutionEnvironment
    1. getExecutionEnvironment
    2. createLocalEnvironment()   本地环境
    3. createRemoteEnvironment()  远程环境
# Source
    1. 从集合里面读取数据
    2. 从文件读取数据
    3. 以kafka消息队列的数据作为来源
        //先配置kafka的参数
        val properties = new Properties()
        properties.setProperty("bootstrap.servers", "mini1:9092")
        properties.setProperty("group.id", "consumer-group")
        properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
        properties.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
        properties.setProperty("auto.offset.reset", "latest")
        environment.addSource(new FlinkKafkaConsumer011[String]("sensor", new SimpleStringSchema(), properties))
            .print().setParallelism(1)
        environment.execute("source test")
    4. 自定义source

# Transform(算子使用)
    1. map
    2. flatmap
    3. filter
    4. keyby
    5. 滚动聚合算子
    6. reduce
    # 多流转换算子
    7. split select
         val splitstream: SplitStream[String] = lines.split(x => {
          if (x.startsWith("h")) {
            Seq("h_start")
          } else {
            Seq("f_start")
          }
        })
        //分多条流打印
        val h_start: DataStream[String] = splitstream.select("h_start")
        val f_start: DataStream[String] = splitstream.select("f_start")
        h_start.print("h")
        f_start.print("f")
    8. connect coMap
        两条流合并一条
        val h_start: DataStream[String] = splitstream.select("h_start")
        val f_start: DataStream[String] = splitstream.select("f_start")
    
        val merge: ConnectedStreams[String, String] = h_start.connect(f_start)
        val value: DataStream[(String, Int)] = merge.map(
          h_s => (h_s, 1),
          f_s => (f_s, 2)
        )
        value.print("comap")

    9. Union
        多条流合并,数据类型一致才行
        val value: DataStream[String] = h_start.union(f_start)
# 支持的数据类型
    1. java scala 基本数据类型
    2. 元组
    3. 样例类  (bean对象相似)
    4. 简单的java对象
    
    
# 函数类

    //自定义预聚合函数  累加器
        class CountAgg extends AggregateFunction[UserBehavior, Long, Long] {
          override def createAccumulator(): Long = 0L
        
          override def add(in: UserBehavior, acc: Long): Long = acc + 1
        
          override def getResult(acc: Long): Long = acc
        
          //合并分区的累加器
          override def merge(acc: Long, acc1: Long): Long = acc + acc1
        }
        //平均数
        class CountAgg extends AggregateFunction[UserBehavior, (Long, Int), Double] {
          override def createAccumulator(): (Long, Int) = (0, 0)
        
          override def add(in: UserBehavior, acc: (Long, Int)): (Long, Int) = (acc._1 + in.timestamp, acc._2 + 1)
        
          override def getResult(acc: (Long, Int)): Double = acc._1 / acc._2
        
          override def merge(acc: (Long, Int), acc1: (Long, Int)): (Long, Int) = (acc._1 + acc1._1, acc._2 + acc1._2)
        }
        
        //定义窗口函数 输出ItemViewcount
        [输入,输出,key的类型,窗口类型]
        class WindowReult extends WindowFunction[Long, ItemViewCount, Long, TimeWindow] {
          override def apply(key: Long, window: TimeWindow, input: Iterable[Long], out: Collector[ItemViewCount]): Unit = {
            //key, 窗口关闭的时间 
            out.collect(ItemViewCount(key, window.getEnd, input.iterator.next()))
          }
        }
        
        
        //topn处理函数
        class TopNAllFunction(topN: Int) extends KeyedProcessFunction[Long, ItemViewCount, String] {
        
          var itemState: ListState[ItemViewCount] = _
        
          override def open(parameters: Configuration): Unit = {
            itemState = getRuntimeContext.getListState(new ListStateDescriptor[ItemViewCount]("list all data", classOf[ItemViewCount]))
          }
        
          override def processElement(i: ItemViewCount, context: KeyedProcessFunction[Long, ItemViewCount, String]#Context, collector: Collector[String]): Unit = {
            //把每条数据存入状态列表
            itemState.add(i)
            //注册定时器
            context.timerService().registerEventTimeTimer(i.windowEnd + 1)
          }
        
          override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[Long, ItemViewCount, String]#OnTimerContext, out: Collector[String]): Unit = {
            val items = new ListBuffer[ItemViewCount]()
        
            import scala.collection.JavaConversions._
            for (item <- itemState.get()) {
              items += item
            }
            //按照count 大小排序
            val sortitems: ListBuffer[ItemViewCount] = items.sortBy(_.count)(Ordering.Long.reverse).take(topN)
            //释放状态
            itemState.clear()
            //将排名结果格式化输出
            val result = new StringBuilder
            result.append("时间: ").append(new Timestamp(timestamp - 1))
            //输出每一个商品的信息
            for (elem <- sortitems.indices) {
              val currentitem: ItemViewCount = sortitems(elem)
              result.append("NO ").append(elem + 1).append(":")
                .append(" 商品ID=").append(currentitem.itemid)
                .append(" 浏览量=").append(currentitem.count)
                .append("\n")
            }
            out.collect(result.toString())
        
            Thread.sleep(1000)
          }
        }


```
#### Sink
```
    kafka
        new FlinkKafkaProducer011[String]("host:9092","topic",new SimpleStringSchame)
        
        # 依赖 官方接口
            <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-connector-kafka-0.11 -->
            <dependency>
                <groupId>org.apache.flink</groupId>
                <artifactId>flink-connector-kafka-0.11_2.11</artifactId>
                <version>1.7.2</version>
            </dependency>
    readis
    
         # 依赖 官方接口 bahir项目解决大数据的接口
        <!-- https://mvnrepository.com/artifact/org.apache.bahir/flink-connector-redis -->
        <dependency>
            <groupId>org.apache.bahir</groupId>
            <artifactId>flink-connector-redis_2.11</artifactId>
            <version>1.0</version>
        </dependency>
        
        class MyRedisMapper() extends RedisMapper[SensorReading] {
          //定义保存到redis的命令
          override def getCommandDescription: RedisCommandDescription = {
            //保存成 id = value 哈希表  HSET  key field value
            new RedisCommandDescription(RedisCommand.HSET, "sensor_temperrature")
          }
          //定义保存到redis的value
          override def getKeyFromData(t: SensorReading): String = {
            t.temperature.toString
          }
          //保存到redis的key
          override def getValueFromData(t: SensorReading): String = {
            t.id
          }
        }
        
        val lines: DataStream[SensorReading] = environment.fromCollection(List(
            SensorReading("Sensor_1", 1547718199, 35.5646461516546),
            SensorReading("Sensor_6", 1547718201, 15.5646461516546),
            SensorReading("Sensor_7", 1547718202, 6.5646461516546),
            SensorReading("Sensor_10", 1547718205, 38.10106545445151)
          ))
          //  lines.print("stream1").setParallelism(1)
          val config: FlinkJedisPoolConfig = new FlinkJedisPoolConfig.Builder()
            .setHost("mini1")
            .setPort(6379)
            .build()
          private val mapper = new MyRedisMapper
          lines.addSink(new RedisSink[SensorReading](config, mapper))
        
    Elasticsearch
        
        依赖  官方提供
        <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-connector-elasticsearch6 -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-elasticsearch6_2.11</artifactId>
            <version>1.7.2</version>
        </dependency>
        
        private val list = new util.ArrayList[HttpHost]()
          list.add(new HttpHost("mini1", 9200))
          //创建一个esSink 的builder
          private val esbuilder = new ElasticsearchSink.Builder[String](list, new ElasticsearchSinkFunction[String] {
            override def process
            (t: String, runtimeContext: RuntimeContext, requestIndexer: RequestIndexer): Unit = {
              println("save es")
              val json = new util.HashMap[String, String]()
              val fields: Array[String] = t.split(" ")
              json.put("id", fields(0))
              json.put("value", fields(1))
              json.put("ts", fields(2))
              //创建index request,准备发送数据
              val request: IndexRequest = Requests.indexRequest()
                .index("sensor")
                //        .`type`("readingdata")
                .source(json)
              //利用requestIndexer发送请求, 写入数据
              requestIndexer.add(request)
            }
          })
          environment.socketTextStream("mini1", 9999)
            .addSink(esbuilder.build())
            
            
    JDBC (Mysql)
    
       # 自定义Sink 
        
        class MyJdbcSink extends RichSinkFunction[String] {
          var connection: Connection = _
          var updatestatement: PreparedStatement = _
          var insertstatement: PreparedStatement = _
          val updatesql: String = ""
          val insertsql: String = ""
          //初始化 创建链接和预编译语句
          override def open(parameters: Configuration): Unit = {
            super.open(parameters)
            connection = DriverManager.getConnection("jdbc:mysql://mini1:3306/", "root", "123456")
            insertstatement = connection.prepareStatement(insertsql)
            updatestatement = connection.prepareStatement(updatesql)
          }
          //调用链接 执行sql
          override def invoke(value: String, context: SinkFunction.Context[_]): Unit = {
            //执行更新语句
            val fields: Array[String] = value.split(" ")
            updatestatement.setDouble(1, fields(0).toDouble)
            updatestatement.setString(2, fields(1))
            updatestatement.execute()
            //没有更新的话 插入新值
            if (updatestatement.getUpdateCount == 0) {
              insertstatement.setString(1, fields(1))
              insertstatement.setDouble(2, fields(0).toDouble)
              insertstatement.execute()
            }
          }
          override def close(): Unit = {
            insertstatement.close()
            updatestatement.close()
            connection.close()
          }
        }
    
```
#### window API

```
1. 一般真实的流都是无界的,
2. 可以把无限的数据流进行切分,得到有限的数据集进行处理--也就是得到有界流
3. 窗口就是将无限流切割为有限流的一种方式,它会将流数据分发到有限大小的桶(bucket)中进行分析

# 时间窗口(time window)
    1. 滚动
    2. 滑动
    3. 会话
        无数据进来就会形成新的窗口 

# 计数窗口(count window)
    1. 滚动
    2. 滑动
# 窗口分配器(assigner)

 全局窗口 无界流
# 窗口函数
    1. 增量聚合函数
    2. 全窗口函数
    # 窗口开始点计算
        //滚动
        getWindowStartWithOffset(timestamp, offset, slide){
             timestamp - (timestamp - offset + windowSize) % windowSize;
        }
        //滑动  lastart(195)    不断的减去 slide(5)   > (184)  timestamp(199) - size(15)  循环创建windows 195-210 190-205 185-200(watermark 延迟关闭窗口)
        180< 184不创建新的窗口
        long lastStart = TimeWindow.getWindowStartWithOffset(timestamp, offset, slide);
			for (long start = lastStart;
				start > timestamp - size;
				start -= slide) {
				windows.add(new TimeWindow(start, start + size));
			}
# 可选api
    .trigger                触发器
    .evitor                 移除器
    .allowedlateness        允许处理迟到的数据
    .slideoutputlatedata    将迟到的数据放入侧输入流
    .getsideoutput          获取侧输入流
# 时间语义 & watermark
        
    1. 时间语义
        # 设置时间语义
            environment.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
        
        event time
        
        process time(默认)
        
            //只有在10秒窗口内  统计大小  窗口小练习
            val minTemp: DataStream[(String, Double)] = sr_ds.map(x => (x.id, x.temp))
              .keyBy(_._1)
              //窗口默认的时间语义  processtime
              .timeWindow(Time.seconds(10))
              .reduce((x, y) => (x._1, x._2.max(y._2)))
        
            minTemp.print("min temp")
            sr_ds.print("all data")
        
    2. watermark (水印)
        1. 从调用时刻开始,给env创建的每一个stream追加时间特性
            import org.apache.flink.streaming.api.TimeCharacteristic
            environment.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
        2.  抽取时间戳  和设置延迟的水印
            .assignAscendingTimestamps(_.time*1000)     只是抽取时间戳       //对于排好序的数据,不需要延迟处理直接给定时间戳就行   不延迟
        
            .assignTimestampsAndWatermarks(new MyAssigner())   //如何从事件中抽取时间戳   和  生成watermark  延迟
            
            //使用 继承自 AssignerWithPeriodicWatermarks<>
            lines.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[SensorReading](Time.milliseconds(1000)) {
              override def extractTimestamp(t: SensorReading): Long = {
                t.time * 1000
              }
            })
        3. MyAssigner
            AssignerWithPeriodicWatermarks<>   //周期性的将watermark 插入到流中 默认200毫秒, ExecutionConfig.setAutoWatermarkInterval(100) 修改周期
            //设置水印延迟 
            environment.getConfig.setAutoWatermarkInterval(100)
            AssignerWithPunctuatedWatermarks<>  //断点式 
            都继承自 TimestampAssigner<>
       
# API使用
    
 # ProcessFuntion 底层API:
    a. 上下文  ctx
    b. 计时器
 
 
    1. ProcessFunction
        
    2. KeyedProcessFunction
    
        //自定义  处理函数
        class MyProcess() extends KeyedProcessFunction[String, SensorReading, String] {
          //定义一个状态  用来保存上个数据的温度值
          lazy val lastTemp = getRuntimeContext.getState(new ValueStateDescriptor[Double]("lastTemp", classOf[Double]))
          //定义一个状态  保存定时器的而时间戳
          lazy val currentTimer = getRuntimeContext.getState(new ValueStateDescriptor[Long]("currentier", classOf[Long]))
        
          override def processElement(i: SensorReading, context: KeyedProcessFunction[String, SensorReading, String]#Context, collector: Collector[String]): Unit = {
            //从状态中获取上次的温度
            val preTemp: Double = lastTemp.value()
            //更新状态中的温度
            lastTemp.update(i.temperature)
            val preTimer: Long = currentTimer.value()
            //温度上升其没有设置过定时器 则注册定时器
            if (i.temperature > preTemp && preTimer == 0) {
              //10000L  给定一个时间段 如果定时器没有被删除 就会触发onTimer() 回调
              val timerTs: Long = context.timerService().currentProcessingTime() + 10000L
              context.timerService().registerProcessingTimeTimer(timerTs)
              //存入数据
              currentTimer.update(timerTs)
            } else if (preTemp > i.temperature || preTemp == 0.0) {
              //删除定时器  并清空状态
              context.timerService().deleteProcessingTimeTimer(preTimer)
              currentTimer.clear()
            }
          }
          override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[String, SensorReading, String]#OnTimerContext, out: Collector[String]): Unit = {
            //输出报警信息
            out.collect(ctx.getCurrentKey + "温度连续上升")
          }
        }
    3. ProcessJoinFunction
    4. BroadcastProcessFunction
    5. KeyedBroadcastProcessFunction
    6. ProcessWindowFunction
    7. ProcessAllWindowFunction
 侧输出流    
 SideOutput   
 
          val value: DataStream[String] = SrData
              .process(new MyProcess)
              //获取侧输出流
              .getSideOutput(new OutputTag[String]("alter output"))
            SrData.print("all data")
            value.print("alter_output")
                
        //自定义  处理函数   [SensorReading, SensorReading] 主输出流 :  输入  输出类型
        class MyProcess() extends ProcessFunction[SensorReading, SensorReading] {
          lazy val alterOutput: OutputTag[String] = new OutputTag[String]("alter output")
          override def processElement(i: SensorReading, context: ProcessFunction[SensorReading, SensorReading]#Context, collector: Collector[SensorReading]): Unit = {
            if (i.temperature < 32.0) {
              //低于32 走侧输出流
              context.output(alterOutput, "alter for" + i.id)
            } else {
              //没有低于32度   正常走主输出流
              collector.collect(i)
            }
          }
        }

```

####  状态管理

```
算子状态
    列表状态
    联合列表状态
    广播状态


键控状态 (keyed statte 常用的)
    数据结构
        值状态
            将状态表示为单个的值
        列表状态
            将状态表示为一组数据的列表
        映射状态
            状态表示为一组key-value对
        聚合状态
            将状态表示为一个用于聚合操作的列表

状态后端
    memoryStateBackend
        会将键控状态作为内存中的对象进行管理,将他们存储在TaskManager的jvm堆上, 而将checkpoint存储在jobmanager的内存中
        特点: 快速低延迟, 不稳定
    FsStateBackend
        将checkpoint存到远程的持久化文件系统上,而对本地状态,跟memoryStateBackend一样,也会存在Taskmanager的jvm堆上
        同时拥有内存级的本地访问速度和更好的容错保证
    RocksDBStateBackend
        将所有状态序列化后存入本地的RocksDB中存储

```
####  容错机制(checkpoint)
```
- 一致性检查点
flink 故障恢复机制的核心,就是应用状态的一致性检查点
有状态流应用的一直检查点,其实就是所有任务的状态,在某个时间点的一份拷贝,这个时间点应该是所有任务都恰好处理完一个相同的输入数据的时候

检查点分界线(checkpoint barrier)


检查点算法
jobmanager -> 发送保存checkpoint消息-> source 接到消息后向checkpoint写入状态, 暂停接收,发送数据-> 成功后反馈给jobmanager-> checkpoint_id 发送到下游任务(通过广播方式通知所有)

 1. 状态一致性
    有状态的流处理,内部内个算子任务都可以有自己的状态
    对于流处理器内部来说,所谓的状态一致性,其实就是我们所说的计算结果要保证准确
    一条数据不应该丢失,也不应该重复计算
    在遇到故障时可以恢复状态,恢复后的重新计算,结果应该也是完全正确的
    分类:
        AT-MOST-ONCE(最多一次)
        AT-LEAST-ONCE(至少一次)
        EXACTLY-ONCE(精确一次)
 2. 一致性检查点
    使用了一种轻量级快照机制-检查点来保证exactly-once语义
    有状态流应用的一直检查点,其实就是所有任务的状态,在某个时间点的一份拷贝,这个时间点应该是所有任务都恰好处理完一个相同的输入数据的时候
    是flink故障恢复机制的核心
 3. 端到端(end-to-end) 状态一致性
    意味着结果的正确性贯穿了整个流处理应用的始终,每个组件都保证了它的一致性
    整个端到端的一致性级别取决于所有组件中一致性最弱的组件
 4. 端到端的精确一次(exactly-once) 保证
    a. 内部保证-checkpoint
    b. source - 可重设数据的读取位置
    c. sink   -  从故障恢复时,数据不会重复写入外部系统
        幂等写入(idemotent writes)
            一个操作,可以重复执行很多次,但只要导致一次结果更改,也就是说,后面再重复执行就不起作用了
        事务写入
            构建的事务对应着checkpoint,等到checkpoint真正完成的时候,才把所有对应的结果写入sink系统中
            实现方式
                预写日志
                    把结果数据先当成状态保存,然后收到checkpoint完成的通知时,一次性写入sink系统
                    DataStream API提供了一个模版类: GenericWriteAheadSink 实现这种事务性sink
                两阶段提交(Two-phase-Commit,2pc 实现exactly-once)
                    对于每一个checkpoint,sink任务会启动一个事务,并将接下来所有接收的数据添加到事务里
                    然后将这些数据写入外部sink系统中,但不提交,
                    当收到checkpoint完成的通知时,它才正式提交事务,实现结果的真正写入
                    实现exactly-once,但是需要提供事务支持的外部sink系统,flink提供了 TwoPhaseCommitSinkFunction接口
                
 5. fink+kafka  端到端状态一致性保证
        source -kafkaconsumer 作为source,可以将偏移量保存下来,如果后续任务出现了故障,恢复的时候可以由连接器重置偏移量,重新消费数据,保证一致性
        sink- kafka produce 作为sink,采用两阶段提交sink,需要实现一个TwoPhaseCommitSinkFunction接口
```

#### table API & sql
```
# 依赖引入
    <!-- https://mvnrepository.com/artifact/org.apache.flink/flink-table -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-table_2.11</artifactId>
        <version>1.7.2</version>
        <scope>provided</scope>
    </dependency>

动态表
字段

 table 转换stream
    addapend
    toRetractStream
    

```