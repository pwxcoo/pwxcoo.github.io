---
title: 《阿里巴巴Java开发手册》注意点
date: 2019-01-23 16:21:10
categories: memo
tags:
- java
---


大概把这本书里之前没注意到的整理了一下。

## prerequisites

- POJO: plain ordinary java object
    - PO: persistent object
    - BO: business object
    - VO: value/view object
    - DTO: data Transfer object 
    - DAO: data access object


## 编程规约

### （一）命名风格

- 抽象类命名使用 Abstract 或 Base 开头 ;
- POJO 类中布尔类型的变量，都不要加 is 前缀 ，否则部分框架解析会引起序列化错误。
- 枚举类名建议带上 Enum 后缀


### （二）常量定义

- 在 long 或者 Long 赋值时，数值后使用大写的 L ，不能是小写的 l ，小写容易跟数字 1 混淆，造成误解。


### （三）代码格式

- 采用 4 个空格缩进，禁止使用 tab 字符。


### （四）OOP 规约

- Object 的 equals 方法容易抛空指针异常，应使用常量或确定有值的对象来调用 equals 。推荐使用 java . util . Objects # equals(JDK 7 引入的工具类 )
- 关于基本数据类型与包装数据类型的使用标准如下: 
    1. 【强制】所有的 POJO 类属性必须使用包装数据类型。
    2. 【强制】 RPC 方法的返回值和参数必须使用包装数据类型。
    3. 【推荐】所有的局部变量使用基本数据类型。
- 禁止在 POJO 类中，同时存在对应属性 xxx 的 isXxx() 和 getXxx() 方法。框架在调用属性 xxx 的提取方法时，并不能确定哪个方法一定是被优先调用到。
- 使用索引访问用 String 的 split 方法得到的数组时，需做最后一个分隔符后有无内容的检查，否则会有抛 IndexOutOfBoundsException 的风险。（源码的实现会舍去末尾的空字符）
    ```java
    // Construct result
    int resultSize = list.size();
    if (limit == 0) {
        while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
            resultSize--;
        }
    }
    String[] result = new String[resultSize];
    return list.subList(0， resultSize).toArray(result);
    ```
- 循环体内，字符串的连接方式，使用 StringBuilder 的 append 方法进行扩展。


### （五）集合处理

- ArrayList 的 subList 结果不可强转成 ArrayList 。subList 返回的是 ArrayList 的内部类 SubList ，并不是 ArrayList 而是 ArrayList
的一个视图，对于 SubList 子列表的所有操作最终会反映到原列表上。
- 使用集合转数组的方法，必须使用集合的 toArray(T[] array) ，传入的是类型完全一样的数组，大小就是 `list.size()` 。直接使用 toArray 无参方法存在问题，此方法返回值只能是 Object[] 类，若强转其它类型数组将出现 ClassCastException 错误。
- 使用工具类 Arrays . asList() 把数组转换成集合时，不能使用其修改集合相关的方法。sList 的返回对象是一个 Arrays 内部类，并没有实现集合的修改方法。 Arrays . asList 体现的是适配器模式，只是转换接口，后台的数据仍是数组。
- 泛型通配符<? extends T >来接收返回的数据，此写法的泛型集合不能使用 add 方法，而 <? super T> 不能使用 get 方法，作为接口调用赋值时易出错。扩展说一下 PECS(Producer Extends Consumer Super) 原则:第一、频繁往外读取内容的，适合用<? extends T >。第二、经常往里插入的，适合用 <? super T> 。
- 集合初始化时,指定集合初始值大小。
- 使用 entrySet 遍历 Map 类集合 KV ,而不是 keySet 方式进行遍历。
- Map 类对 null 值的处理。

    | 集合类            | Key | Value | Super       | 说明                |
    |-------------------|-----|-------|-------------|---------------------|
    | Hashtable         | no  | no    | Dictionary  | thread safe         |
    | ConcurrentHashMap | no  | no    | AbstractMap | 锁分段（JDK8: CAS） |
    | TreeMap           | no  | yes   | AbstractMap | thread unsafe       |
    | HashMap           | yes | yes   | AbstractMap | thread unsafe       |



### （六） 并发处理

- 【强制】创建线程或线程池时请指定有意义的线程名称,方便出错时回溯。
- 【强制】线程池不允许使用 Executors 去创建,而是通过 ThreadPoolExecutor 的方式,这样的处理方式让写的同学更加明确线程池的运行规则,规避资源耗尽的风险。
    1. FixedThreadPool 和 SingleThreadPool :
    允许的请求队列长度为 Integer.MAX_VALUE ,可能会堆积大量的请求,从而导致 OOM 。
    2. CachedThreadPool 和 ScheduledThreadPool :
    允许的创建线程数量为 Integer.MAX_VALUE ,可能会创建大量的线程,从而导致 OOM 。
- 并发修改同一记录时,避免更新丢失,需要加锁。要么在应用层加锁,要么在缓存加锁,要么在数据库层使用乐观锁,使用 version 作为更新依据。如果每次访问冲突概率小于 20%,推荐使用乐观锁,否则使用悲观锁。乐观锁的重试次数不得小于 3 次。
- 多线程并行处理定时任务时, Timer 运行多个 TimeTask 时,只要其中之一没有捕获抛出的异常,其它任务便会自动终止运行,使用 ScheduledExecutorService 则没有这个问题。
- 使用 CountDownLatch 进行异步转同步操作,每个线程退出前必须调用 countDown 方法,线程执行代码注意 catch 异常,确保 countDown 方法被执行到,避免主线程无法执行至 await 方法,直到超时才返回结果。
- HashMap 在容量不够进行 resize 时由于高并发可能出现死链,导致 CPU 飙升, 在开发过程中可以使用其它数据结构或加锁来规避此风险。
- ThreadLocal 无法解决共享对象的更新问题, ThreadLocal 对象建议使用 static修饰。这个变量是针对一个线程内所有操作共享的,所以设置为静态变量,所有此类实例共享此静态变量 ,也就是说在类第一次被使用时装载,只分配一块存储空间,所有此类的对象 ( 只要是这个线程内定义的 ) 都可以操控这个变量。



### （七）控制语句

- 【参考】下列情形,需要进行参数校验:
    1. 调用频次低的方法。
    2. 执行时间开销很大的方法。此情形中,参数校验时间几乎可以忽略不计,但如果因为参
    数错误导致中间执行回退,或者错误,那得不偿失。
    3. 需要极高稳定性和可用性的方法。
    4. 对外提供的开放接口,不管是 RPC / API / HTTP 接口。
    5. 敏感权限入口。
- 【参考】下列情形,不需要进行参数校验:
    1. 极有可能被循环调用的方法。但在方法说明里必须注明外部参数检查要求。
    2. 底层调用频度比较高的方法。毕竟是像纯净水过滤的最后一道,参数错误不太可能到底
    层才会暴露问题。一般 DAO 层与 Service 层都在同一个应用中,部署在同一台服务器中,所
    以 DAO 的参数校验,可以省略。
    3. 被声明成 private 只会被自己代码所调用的方法,如果能够确定调用方法的代码传入参
    数已经做过检查或者肯定不会有问题,此时可以不校验参数。

### （八） 注释规约



### （九）其它

- 【强制】后台输送给页面的变量必须加 $!{var} ——中间的感叹号。
- 在 JDK 8 中, 针对统计时间等场景,推荐使用 Instant 类。


## 异常日志

### (一) 异常处理

- 【推荐】防止 NPE ,是程序员的基本修养,注意 NPE 产生的场景:
    1. 返回类型为基本数据类型, return 包装数据类型的对象时,自动拆箱有可能产生 NPE 。
    2. 数据库的查询结果可能为 null 。
    3. 集合里的元素即使 isNotEmpty ,取出的数据元素也可能为 null 。
    4. 远程调用返回对象时,一律要求进行空指针判断,防止 NPE 。
    5. 对于 Session 中获取的数据,建议 NPE 检查,避免空指针。
    6. 级联调用 `obj.getA().getB().getC()`; 一连串调用,易产生 NPE 。

### (二) 日志规约

- 应用中不可直接使用日志系统 (Log 4 j 、 Logback) 中的 API ,而应依赖使用日志框架 SLF4J（Simple Logging Facade for Java）中的 API ,使用门面模式的日志框架,有利于维护和各个类的日志处理方式统一。


## 单元测试

## 安全规约

- 【强制】表单、 AJAX 提交必须执行 CSRF 安全验证。

## MySQL 数据库

### (一) 建表规约

- 【强制】表达是与否概念的字段,必须使用 is_xxx 的方式命名,数据类型是 unsigned tinyint(1 表示是,0 表示否)。
- 【强制】主键索引名为 pk_ 字段名;唯一索引名为 uk _字段名 ; 普通索引名则为 idx _字段名。
- 【强制】小数类型为 decimal ,禁止使用 float 和 double 。
- 【强制】如果存储的字符串长度几乎相等,使用 char 定长字符串类型。
- 【强制】表必备三字段: id , gmt _ create , gmt _ modified 。
- 【推荐】字段允许适当冗余,以提高查询性能,但必须考虑数据一致。冗余字段应遵循:
    1. 不是频繁修改的字段。
    2. 不是 varchar 超长字段,更不能是 text 字段。
- 【推荐】单表行数超过 500 万行或者单表容量超过 2 GB ,才推荐进行分库分表。

### (二) 索引规约

- 【强制】业务上具有唯一特性的字段,即使是多个字段的组合,也必须建成唯一索引。
- 【强制】在 varchar 字段上建立索引时,必须指定索引长度,没必要对全字段建立索引,根据实际文本区分度决定索引长度即可。索引的长度与区分度是一对矛盾体,一般对字符串类型数据,长度为 20 的索引,区分度会高达 90% 以上,可以使用 count(distinct left( 列名, 索引长度 )) / count( * ) 的区分度来确定。
- 【强制】页面搜索严禁左模糊或者全模糊
- 【推荐】如果有 order by 的场景,请注意利用索引的有序性。
- 【推荐】利用覆盖索引来进行查询操作,避免回表。
- 【推荐】利用延迟关联或者子查询优化超多分页场景。
- 【推荐】建组合索引的时候,区分度最高的在最左边。


### (三) SQL 语句

- 【强制】不要使用 count( 列名 ) 或 count( 常量 ) 来替代 count( * ) , count( * ) 是 SQL 92 定义的标准统计行数的语法,跟数据库无关,跟 NULL 和非 NULL 无关。count( * ) 会统计值为 NULL 的行,而 count( 列名 ) 不会统计此列为 NULL 值的行。
- 【强制】 count(distinct col) 计算该列除 NULL 之外的不重复行数,注意 count(distinct col 1, col 2 ) 如果其中一列全为 NULL ,那么即使另一列有不同的值,也返回为 0。
- 【强制】当某一列的值全是 NULL 时, count(col) 的返回结果为 0,但 sum(col) 的返回结果为 NULL ,因此使用 sum() 时需注意 NPE 问题。
- 【强制】使用 ISNULL() 来判断是否为 NULL 值。NULL 与任何值的直接比较都为 NULL。
- 【强制】不得使用外键与级联,一切外键概念必须在应用层解决。


### (四) ORM 映射

- 【强制】 POJO 类的布尔属性不能加 is ,而数据库字段必须加 is _,要求在 resultMap 中进行字段与属性之间的映射。
- 【强制】不要用 resultClass 当返回参数,即使所有类属性名与数据库字段一一对应,也需要定义 ; 反过来,每一个表也必然有一个 POJO 类与之对应。
- 【强制】 sql. xml 配置参数使用: #{}, # param # 不要使用${} 此种方式容易出现 SQL 注入。
- 【参考】< isEqual >中的 compareValue 是与属性值对比的常量,一般是数字,表示相等时带上此条件 ; < isNotEmpty >表示不为空且不为 null 时执行 ; < isNotNull >表示不为 null 值时执行。


## 工程结构

### (一) 应用分层

### (二) 二方库依赖

- 【强制】二方库里可以定义枚举类型,参数可以使用枚举类型,但是接口返回值不允许使用枚举类型或者包含枚举类型的 POJO 对象。


### (三) 服务器

- 【推荐】高并发服务器建议调小 TCP 协议的 time _ wait 超时时间。操作系统默认 240 秒。在 linux 服务器上请通过变更/ etc / sysctl . conf 文件去修改该缺省值 ( 秒 ) :
    ```
    net.ipv4.tcp_fin_timeout = 30
    ```
- 【推荐】调大服务器所支持的最大文件句柄数 (File Descriptor ,简写为 fd) 。
- 【推荐】给 JVM 环境参数设置- XX :+ HeapDumpOnOutOfMemoryError 参数,让 JVM 碰到 OOM 场景时输出 dump 信息。
- 【推荐】在线上生产环境, JVM 的 Xms（JVM初始分配的堆内存 ） 和 Xmx（JVM最大允许分配的堆内存，按需分配 ） 设置一样大小的内存容量,避免在 GC 后调整堆大小带来的压力。