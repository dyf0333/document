# java、php thrift

RPC的概念就不赘述了

公司项目需要 php与java 开发部分相互通讯，用RPC；

#### 选型：

    gRPC google的产品，不知道啥时候可能被墙
    thrift  虽然也算前facebook的产品，不过目前是apache，不至于把apache墙了吧
    
    
### java和php使用thrift
 
1.RPC:远程过程调用协议，它是一种通过网络从远程计算机程序上请求服务的方式.
RPC采用客户机/服务器模式。请求程序就是一个客户机，而服务提供程序就是一个服务器。
 
2.thrift:Apache Thrift 是 Facebook 实现的一种高效的、支持多种编程语言的远程服务调用的框架。

一种RPC框架。
```
 （1）可以跨语言相互调用，比如java调用php。
 （2）使用的4层网络模型。http是7层网络模型。
 （3）速度比http要快。
```

### 安装thrift
    
    brew install thrift

文件目录说明：

![|center](../master/src/thrift-1.png)

client目录：

```
  JavaClient：php调用java的client端
  PhpClient: php实现的client端。（php写的server端）
```

test.thrift 生成的client端通信协议文件
    
    lib目录：依赖的thrift的包
    
server目录：

```
 thrift:  test.thrift 生成的server端的通信协议文件
 TestServer.php  php实现的server端
 runserver.py  监控端口
```

通信协议的定义：文件名：test.thfit

```
//查询

 namespace java thrift.service  //生成java时是报名
 namespace php thrift.service    //生成的php的文件路径
 const string version = "1.0.0"

service TestService {    //类名

 // 获取版本号
 string getServiceVersion();

 //方法
  string getData(1:string str,2:i32 id);    //通信协议，传的参数是字符串和整型，返回字符串。

 }
```

Java作为server端，php和java作为客户端

```
 java通过thrift生成通信协议的文件。

 thrfit --gen java test.thrfit

 server端
       -- 实现thrift通信协议
```

![|center](../master/src/thrift-2.png)

server启动端口：

![|center](../master/src/thrift-3.png)

java作为client:

![|center](../master/src/thrift-4.png)

php作为client:

```
  （1）首先要生成php的客户端的通信文件：  

          thrift —gen php test.thrift 

  （2）导入依赖的lib文件。

  （3）实现client
```

![|center](../master/src/thrift-5.png)

php使用thrift

>用php作为server端时，依赖于服务器。实现方式有两种。

1.使用python 启动一个端口监控

![|center](../master/src/thrift-6.png)

2 把server端的文件放到apache服务器下面

>注意： 使用第一种方式时，client和server端的文件的第一行加//#!/usr/bin/env php   第二种方式不用加。

php生成server端时和生成client端的通信文件的命令是不一样的。

server端：thrift -r --gen php:server Test.thrift

![|center](../master/src/thrift-7.png)

client端：thrift —gen php test.thrift

![|center](../master/src/thrift-8.png)

(1)第一种启动方式：./runserver.py
>请求方式: server: php PhpClient.php

(2)第二种启动方式：把php项目放到apache下面
>请求方式server: php PhpClient.php

结果：
![|center](../master/src/thrift-9.png)


java服务器端和php服务器端区别：

```
（1）java直接打个jar包，就可以启动一个端口，接受thrift请求。不依赖于服务器，例如tomcat。
（2）php 作为thrift的server端，没有办法自己启动端口，需要依赖于服务器，需要部署在服务器下面。例如apache.
```

一些网上的其他要点，可以关注与使用：

```
thfit服务器端：

    1. 创建Handler，用于处理业务逻辑,数据处理接口.

    2. 基于Handler创建Processor,数据处理对象

    3. 创建Transport（通信方式）,数据传输方式

    4. 创建Protocol方式（设定传输格式）,数据传输协议

    5. 基于Processor, Transport和Protocol创建Server

    6. 运行Server

thrift客户端：

    客户端编写的一般步骤：

    1. 创建Transport

    2. 创建Protocol方式

    3. 基于Transport和Protocol创建Client

    4. 运行Client的方法

数据传输方式，即Protoco

    TBinaryProtocol – 二进制格式.

    TCompactProtocol – 压缩格式

    TJSONProtocol – JSON格式

支持的通信方式即，Transport

    THttpTransport：采用Http传输协议进行数据传输

    TSocket：采用TCP Socket进行数据传输

    下面几个类主要是对上面几个类地装饰（采用了装饰模式），以提高传输效率。

    TBufferedTransport：对某个Transport对象操作的数据进行buffer，即从buffer中读取数据进行传输，或者将数据直接写入buffer

    TFramedTransport：以frame(帧：按照固定大小来传输）为单位进行传输，非阻塞式服务中使用。同TBufferedTransport类似，也会对相关数据进行buffer，同时，它支持定长数据发送和接收。

    TMemoryBuffer：从一个缓冲区中读写数据

服务器端支持的模型。

    TSimpleServer – 简单的单线程服务模型，常用于测试

    TThreadedServer - 多线程服务模型，使用阻塞式IO，每个请求创建一个线程。

    TThreadPoolServer – 线程池服务模型，使用标准的阻塞式IO，预先创建一组线程处理请求。

    TNonblockingServer – 多线程服务模型，使用非阻塞式IO（需使用TFramedTransport数据传输方式）

    处理大量更新的话，主要是在TThreadedServer和TNonblockingServer中进行选择。TNonblockingServer能够使用少量线程处理大量并发连接，但是延迟较高；TThreadedServer的延迟较低。实际中，TThreadedServer的吞吐量可能会比TNonblockingServer高，但是TThreadedServer的CPU占用要比TNonblockingServer高很多。

阻塞IO:socket的阻塞意味着必须要做完Io包括错误才会返回。

    非阻塞io：无论操作是否完成都会立刻返回。

```