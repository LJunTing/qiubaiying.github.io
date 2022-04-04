---
layout:     post                    # 使用的布局（不需要改）
title:      Android 高级            # 标题 
subtitle:   高级知识	 #副标题
date:       2022-4-5             # 时间
author:     JT                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android 
    - Android 高级面试
    - 高级
    - 待完善
    - 学习
---

[TOC]

### arraylist 效率与linkedlist性能对比

> arraylist : 
>
> > 查询修改快,增加删除的话,会copy从position到end的数组,添加删除后,在赋值到源数组
>
> linkedlist: 
>
> > 增加删除快,查询慢 : 插入前面的节点指向他,他指向后面的节点.  查询的要遍历链表才能拿到值

**如果一个数组要反复插入删除怎么优化降低时间复杂度**

>对数组要删除的进行标记(如:null),在插入到这个地放就不会copy了,
>
>不然删除 插入就要copy两次

### hashmap

>  hashmap基础结构还是： 数组 + 链表 ，称作哈希表 或 散列表 

#### 插入节点的流程

> jdk1.8之前头部插入,之后尾部插入
>
> 先将key hash&(length-1) ,根据得出的数据下标拿到对应entry对象
>
> (少于=8个)对像又是个双向链表结构 ,在链表大于8个转为红黑树结构
>
> 红黑树构建时占用性能大

#### 如何解决hash碰撞问题

> 碰撞的都存入一个链表
>
> hash&(length-1)     16-1=15 (二进制: 1111)   
>
> h &1111会均匀分布,    H&1100 不管h=1 or 0 都是0,分布集中,链长性能差
>
> 所以lenght=2的n次幂

#### HashMap与SparseArray对比

> hashmap: (key(object))
>
> sparseArray: key(int)  二分查找, 对数组要删除的进行标记(如:null),在插入到这个地放直接赋值,就不会copy了,不然删除 插入就要copy两次.  频繁删除插入效率很高.  (性能,内存都成倍优于hashmap) 







### Contant

### activity View window关系
### Android 设备启动流程

> android 系统初始化-->bootloader进程--> linux kernel->init.rc-->
>
> zygote进程->jvm jni Systemserver-->(binder线程池 systemserviceManager)
>
> zy,sysS,SSM--共同作用 启动-->AMS,WMS,PMS...(共八十多种)

### app启动流程

> 

### dex文件结构

> 

### apk打包流程

> R.java(aapt),app源码 ,aidl转换java接口---java complier-->打包.class文件+加三方库一起打包-->dex文件-->将未打包的资源(complied R  and other R)和dex---apk builder->打包成apk-->然后签名(jarsinger签名工具)---xx.keystore->打出签名apk-->对齐

### 事件分发
### measurespec

**mainactivity oncreate()里面开线程更新UI不报错**

- 只有activity.onresume() 关联window UI线程检查才开启

**生命周期**
- getmeasuredwidth  在measure()过程结束就可以获取到对应的值,通过setMeasuredDimension()设置的
- getwidth  在layout结束才能获取到,通过右减去左坐标
- 父类直接调childermeasure() 不走初始化,数据在measure()

### 自定义view布局原理

> onmeasure()--> chidleview.onmeasure()
>
> > 1. viewgroup 走onmeasure()会先测量所有子childview.onmeasure()
> > 2. 所以veiwgroup也可能是父类调用的onmeasure(), 初始化不在构造中,不走构造方法,直接childview.onmeasure()
>
> onlayout()
>
> > 布局相关的都在这, 父view的左上坐标,屏幕左上坐标
>
> 
>
> ondraw()
>
> > 画图形,动画(内存泄漏在这里)

### xml解析过程原理

-layoutinflater inflate() -> xml->creatView-包名反射创建view实例-->解析成java code布局
- creatViewFromTag-->createview()都要在这逐层创建-->可以拿到所有view
- 直接在内存中不需要重启的状态下: 换肤 适配 沉浸式

- activity->window(phonewindow)->decorview(framelayout)-veiwstub-contentparent-->contentview

### 如何解决项目中viewpagerUI卡顿问题
>一般是嵌套有fragment,又有预加载,  使用空白布局, fragment懒加载

### viewpager的缓存机制

> 适配器模式: 将fragment放入arraylist中,用的时候拿出来,缓存机制
>
> offscreenpagelimit的值就是缓存的数量(FragmentStatePagerAdapter)

### viewpager+fragment嵌套使用,fragment如何管理

> fragment 事务管理 commit(里面handler提交)
>
> instantiateItem (用户可见的) viewpager对缓存fragmentde 管理
>
>  缓存的 -- 当前的--目标的


### recycleview的复用  缓存机制


### 内存泄漏OOM如何发生的?减少OOM的概率

### 内存抖动的影响原理 
>频繁GC 线程暂停 用户操作卡顿 (循环大量创建对象)

### 如何解决应用崩溃
>集成三方的日志捕捉,日志上传后台分析解决

### ANR是什么 怎么避免

> 1. 主线程尽量只做UI相关的操作,避免耗时操作 比如过度发杂的UI绘制,网络操作,文件IO等
>
> 2. 避免主线程跟工作线程发生锁的竞争,减少系统耗时binder的调用,谨慎使用SP,注意主线程执行provider query操作
>
> 3. 减少主线程负载 , 可以随时响应用户操作
>
>    
>
> 输入事件(点击,触摸)   5
>
> contantprovider        10  
>
> BroadcastReceiver    10           60
>
> servevice                      20        200

### app启动速度如何优化

> 绘制优化: 根本原因:层级深,数据处理
>
> 内存:
>
> 存储:
>
> 稳定性:
>
> 耗电:
>
> apk瘦身:
>
> 工具: 

### andfix修复原理
>补丁包-类加载->fixfile--反射类-->拿到实例等--反射注解-->拿到要修改的(类的包名,method()名字)-->把类的方法替换(成第一次反射获取的实例)
### robost原理

> 对每个函数都在编译打包阶段自动插入了一段代码.(字节码增强 插桩)
> 类似于代理,将方法执行的代码重定向到其中
> 补丁包-类加载->fixfile--反射类-->拿到实例等--反射注解-->拿到要修改的(类的包名,method()名字)-->把类的方法替换(成第一次反射获取的实例)

### tinker

> 差异包(增量更新)
> dex差异包,运行时将baseapk的dex与差异包进行合成,重启后加载全新的合成后的dex文件

- Android classloader
```
1. bootclassloader (加载framwork层的class文件)
2. pathclassloader(Android 应用程序类加载器)
3. dexclassloader (额外提供的动态类加载器)

2,3加载指定的dex,以及jar,zip,apk中的classes.dex
```
> application获取类加载器(pathclassloder)
> findclass()-->从dexElements遍历出dexfile1能找出要加载的class文件就加载,否则继续找第二个dexfile2
> 在dexElements前面塞入差异包dex,如果能找到A类直接return,后面的文件也有A类也不会找
> application获取类加载器(pathclassloder)--反射-->patlist-->dexelements-->塞入

### Android 类加载机制的原理

- bootclassloder(framwork) pathclassloder(用户应用)

- 获取当前应用的pathclassloader
- 反射获取到DexPathList属性对象pathList
- 反射修改pathList的dexElements
    + 把补丁包patch.dex转化为Element[] 
    + 获得pathList的dexElements属性(old)
    + patch+dexElements合并,并反射赋值给pathList的dexElements


### rxjava
- 链式调用解决嵌套访问的,存在嵌套网络请求就会陷入 "回调陷阱"
- 

**线程如何切换**

**适配器**

- 将okhttp call()封装返回了(retorfit call(), rxjava Observable)


### dagger2如何实现注入,全局单例, 多组件依赖


### leakcanary 如何自动检测内存对象泄漏的
>继承了contentProvider,而在应用启动过程中,先加载的contentprovider,再加载application

### glide

**主线流程**
**为何监听fragment-activity**

**with主线流程**

>空白的fragment管理生命周期,主线程才有fragment

**load主线流程**

>最终构建requestBuilder

**into主线流程**

- 等待队列 执行队列
- 活动缓存
- 内存缓存
- httpurlconnection

>glide构建->给每一个requestM绑定一个空白的fragment来管理生命周期->
>request对像的构建(请求宽高 采样)->请求之前先检查缓存->
>engine缓存机制的检测:先活动缓存,内存缓存->缓存都没有在enginejob构建一个新的异步任务->执行request之前,先检测discache中有没有本地磁盘缓存->没有,就通过网络请求,返回输入流(inputStream)->解析输入流进行采样压缩,最终拿到bitmap->bitmap转换为drawable->构建磁盘缓存->构建内存缓存->回到ImageViweTarget(DrawableImageViewTarget)

### okhttp
>clientbuilder.build->oClient->request->new call();

**使用流程-分发器与拦截器**
- 分发器: 内部维护队列与线程池,完成请求调配
- 拦截器: 完成整个请求

**分发器原理**
- 同步请求 分发器只记录请求,用于判断idlerunnable是否选哟执行
- 异步请求,向分发器中提交请求

**分发器线程池**

**分发器线程池的工作行为**
- 无等待 最大并发 Squeue
  

**如何决定请求放入ready还是running**
- 如果当前正在请求数为64,则将请求放入ready等待执行,如果小于64,但是已经存在同一域名主机的请求5个也会放入ready,否则放入running队列立即执行

**从ready移动过running的条件是什么**
- 每个请求执行完成就会从running移除,同时进行第一步相同逻辑的判断决定是否移动

**拦截器责任链设计模式**
- 重试拦截器,桥接拦截器(头信息的补全),缓存拦截器,
- 连接拦截器(找到或新建一个链接,获取socket),请求服务器拦截进行真正的与服务器的通信,向服务器发送数据,解析读取的响应数据

**使用存在的问题**

### retrofit 

> 创建retrofit对象-->配置baseurl--callfactory(网络请求)--callbackexecutor(回调方法执行器)
>
> --adapterfactories(网络请求适配器)--converterfactories(数据转换)

#### **中create为什么用动态代理**

- 动态代理可以获取到接口所有信息(注解,参数等),自动构建request对象,不用手动配置这些请求每次都要设置的东西
- 动态代理可以代理所有的接口,让所有的接口都走invoke()函数,这样就可以拦截调用函数的执行,从而将网络接口的参数配置归一化

#### **相对于okhttp解决的问题明白吗**

**实现思想和设计原理**

- 用户网络请求的接口配置繁琐,尤其是需要配置复杂请求body,请求头,参数的时候
- 数据解析过程需要用户手动拿到responsbody进行解析,不能复用
- 无法适配自动进行线程的切换
- 万一我们的网络请求存在嵌套就会陷入 "回调陷阱"

#### serviceMethod设计理念

> 先从缓存中获取SM对象,如果缓存有就去出来,没有构建一个出来在加入到缓存中
>
> 设计思想 :  
>
> 如果不缓存,那么每次解析接口注解,解析很多东西,都要通过反射获取,耗费大量性能
>
> 这样不用每次都要解析接口,直接返回缓存中第一次构建的对象,

### rxjavaCallAdapterFactory的设计模式

> 网络请求适配工厂
>
> 适配成不同的网络请求对象  经常使用的:
>
> > retrofit: 特有的call
> >
> > rxjava:   observeable

**粘性事件原理**

****
****
****

### databinding 
**初始化做了什么事情**
**双向绑定的原理**

### workmanager 

**主线流程**
**有条件约束的流程**
### serializable

#### app启动流程

> luncher-->ASM--发送创建进程请求-->zygote---fork新进程-->app---通知-->AMS---realstartActivitylocked---applicationThread---lunch_activity--activityThread---handlelunchActivity--Activity.oncreate

### 内存共享机制是如何完成过handler跨线程的

- 子线程: ahndler.sendM->enqueueM->queue.enqueueM
- 主线程: activityThread->looper.loop->queue.next->dispatchM-->hadler.handlerM
- handler:(androd 生命周期都是)
- UI线程 ,界面显示,统一管理UI

### handler导致内存泄漏


- 非静态内部类和匿名内部类会持有外部类(ac)的引用。

- handler发送消息<-持有Message<-持有messagequeue<-持有<-looper()--持有<-threadlocal(静态对象 此次的GCroot)



> 如过有延迟消息,还未处理,activity销毁了,那么会造成threadlocal一直持有activity对象


> 解决:
> 1. activity销毁时移除所有消息
> 2. 2.static 修饰handler 全局变量不持有activity


### sychronized锁机制 与 wait notify原理

- HandlerThread的实现子线程创建handler
```
@Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();//唤醒对象的等待池中的所有线程，进入锁池。
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }
    public Looper getLooper() {
        if (!isAlive()) {
            return null;
        }
        
        // If the thread has been started, wait until the looper has been created.
        synchronized (this) {
            while (isAlive() && mLooper == null) {//防止其他地方唤醒,唤醒再判断下
                try {
                    wait(); //释放锁 可能多个地方调用 对应notifyall
                    //不阻塞线程 CPU交给别人用
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }

synchronized( this )    
synchronized( object )   
synchronized( 方法 )   
static synchronized{
    
}
    
    
    
```
### handlerthread没有消息做什么, 有什么用

- quit()-->looper.quit-->mquitting=true-移除所有消息-->next() returen null-->loop()msg=queue.next=null,loop结束
- 如果loop结束,那么handler不会释放,启动handler的Thread不会释放,内存泄漏


### handler如何发送延迟消息

- messagequeue.next()->nativepollonce(ptr,nextPollTimeoutMIllis)->native 的方法 epoll具体实现

### AMS中如何应用handler进行管理


### 享元设计模式应用原理解析

- handler.obtain
- 共享内存池 
- 地图 股票软件 recycleview(recyclepool) 对象频繁创建
- 
```java
//从全局池中返回一个新的 Message 实例。允许我们在很多情况下避免分配新对象。
public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0; // clear in-use flag
                sPoolSize--;
                return m;
            }
        }
        return new Message();
    }
    
    //将消息标记为正在使用，同时保留在回收对象池中。清除所有其他细节。
    //回收消息
     void recycleUnchecked() {
        // Mark the message as in use while it remains in the recycled object pool.
        // Clear out all other details.
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = UID_NONE;
        workSourceUid = UID_NONE;
        when = 0;
        target = null;
        callback = null;
        data = null;

        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }
```


### handler的阻塞机制为什么不会导致ANR


- 阻塞 , 没有事情做了,sleep 没有启动定时器
- epoll机制,主线程阻塞并进入休眠,释放cpu资源

### AIDL通信机制的原理

- stub 存根(binder驱动--stub---serviece)融合
- Proxy(代理Ibinder) implements IAIDLInterface
- 同一包名(service,client) 
- client-->服务接口-->服务Proxy->binder proxy->
- binder Driver-->binder->服务stub->服务server接口-->server

### binder

#### **binder机制如何夸进程**
- 共享内存  无需copy
- 传统c/s socket 通信(server--copy信息--内核---copy-(两次copy)-client)
- binderc/s   servser-->在内核空间 binder(一次copy)--client
- 8之前共享空间有个数(越多占用内存越大),之后优化用完释放

#### **如何做到只需一次拷贝**
#### **MMap 的原理与学习价值**

- 虚拟内存 : 多个地方需要的内存(可能比物理大) -> 都映射到真实的物理内存
- MMU
- binder内存--虚拟内存--地址,MMU--指向->物理内存
- zygote-->init()-->processtate-->mmap()(初始化进程时为进程准备的空间1M-8k 物理内存)-->bindermmap使用空间
- 将用户进程空间(vm_area_stuct)和内核地址(binderbuffer)都与mmap()那块空间映射 (虚拟内存)

#### **binder通信**

-  c->s 所以用户进程 和内核空间 通信只需在 真实物理内存改下地址映射(到进程地址)就能拿到数据
- s->c  c的进程也有一块物理内存 binder也和这块内存映射 同理

#### **总结 :** 
- MMap对文件的读写操作只需要从磁盘到用户主存的一次数据拷贝过程,减少了数据的拷贝次数,提高呃文件曹组效率.
- MMap使用逻辑内存对磁盘文件进行映射,操作内存就相当与操作文件,不需要开启线程,操作MMAp的速度和操作内存的速度一样快

#### **设计日志系统**

- [美团loagn设计思想](https://tech.meituan.com/2018/02/11/logan.html),微信mmkv

- app挂了没有log, 日志产生快,stream 硬盘太慢,抛弃


#### **相对于其他IPC机制的优点**

- 共享内存  无需copy  (控制复杂,自行处理并发同步问题)(访问接入点不安全)
- 传统c/s socket 通信(server--copy信息--内核---copy-(两次copy)-client)  (c/s)   (访问接入点开放不安全)
- binderc/s   servser-->在内核空间 binder(一次copy)--client  (c/s相对独立,稳定性好)  (内核(对进程)添加身份标识(可靠)) 
- 一个牛人创建(openbinder)---入职谷歌->binder机制加入Android

#### **四大组件的通信**

```

### 同进程activity service 通信,   AMS不知道AC和service是否同一进程  6次binder通信

                 1    2次binder通信   2             3
ac-->serviceM ------------>AMS --------->serviceM----->ac (拿到ASM)

                     4  2次binder通信     5                                   6  bindre交给      
service--->serviceM-------->AMS---------------->serviceM-->service (拿到ASM)-------------->AMS()

ac-->ASM<--Server   -->ac和service关联起来

```

- AIDL是对binder的封装,AIDL是个标准,AMS就是手写,自己也可以按照AIDL标准写



#### **intent能够传递的最大数据内存是多大**

- 进程 or binder ---映射-->  (mmap(1M-8k))物理内存就这么大

### sp原理 优化

- SharedPreferencesImpl
- 优先从全局缓存map中取到spi对象,没有则新建一个spi对象(并在子线程中执行读取xml文件) ,-->将xml数据取到spi的map中,mloaded=true,唤醒其他线程
- mloaded=false,对sp文件的操作都要判断状态,false当前线程等待
- 每次edit都会创建editor对象,数据保存到Editor对象内部的临时容器map中,且clear操作也只是改变其内部clear字段值为true-->commit() or apply map与spi内部的map进行数据对比,--->map写入到sp文件中
- commit() 直接在当前线程提交,apply()放入一个线程池中执行,ActivityThread handleStopActivity方法中会检查这个线程池中的任务，如果任务未完成则会等待。
- SharedPreferences并不支持多进程，对于mode是多进程模式也仅仅是重新加载文件到内存中。
另外SharedPreferences 内部是通过加锁保证线程安全的。


- 优化:
- 不要存放大的key和value在SharedPreferences中，否则会一直存储在内存中得- 不到释放，内存使用过高会频发引发GC，导致界面丢帧甚至ANR。
- 不相关的配置选项最好不要放在一起，单个文件越大读取速度则越慢。
- 读取频繁的key和不频繁的key尽量不要放在一起（如果整个文件本身就较小则忽略，为了这点性能添加维护得不偿失）。
- 不要每次都edit，因为每次都会创建一个新的EditorImpl对象，最好是批量处理统一提交。否则edit().commit每次创建一个EditorImpl对象并且进行一次IO操作，严重影响性能。
- commit发生在UI线程中，apply发生在工作线程中，对于数据的提交最好是批量操作统一提交。虽然apply发生在工作线程（不会因为IO阻塞UI线程）但是如果添加任务较多也有可能带来其他严重后果（参照ActivityThread源码中handleStopActivity方法实现）
- 尽量不要存放json和html，这种可以直接文件缓存。
- 不要指望它能够跨进程通信 Context.PROCESS
- 最好提前初始化SharedPreferences，避免SharedPreferences第一次创建时读取文件线程未结束而出现等待情况。