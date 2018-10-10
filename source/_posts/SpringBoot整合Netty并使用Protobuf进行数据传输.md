---
title: SpringBoot整合Netty并使用Protobuf进行数据传输
date: 2018-10-10 22:49:36
tags:
---

![logo](/home/james/%E5%9B%BE%E7%89%87/logo.png)



# 前言

本篇文章主要介绍的是SpringBoot整合Netty以及使用Protobuf进行数据传输的相关内容。Protobuf会介绍下用法，至于Netty在[netty 之 telnet HelloWorld 详解](https://www.cnblogs.com/sanshengshui/p/9726306.html)中已经介绍过了，这里就不再过多细说了。

# Protobuf

## 介绍

> Protocol Buffer是Google的语言中立的，平台中立的，可扩展机制的，用于序列化结构化数据 - 对比XML，但更小，更快，更简单。您可以定义数据的结构化，然后可以使用特殊生成的源代码轻松地在各种数据流中使用各种语言编写和读取结构化数据。

官网地址: https://developers.google.com/protocol-buffers/

## 使用

这里的使用就只介绍Java相关的使用。具体protobuf3的使用可以看[Protobuf 语言指南(proto3)](https://www.cnblogs.com/sanshengshui/p/9739521.html)。
首先我们需要在src/main文件夹下建立一个proto文件夹，然后在该文件夹新建一个**user.proto**文件，此文件定义我们需要传输的文件。

![选区_002](/home/james/%E5%9B%BE%E7%89%87/%E9%80%89%E5%8C%BA_002.png)

**注**：使用grpc方式编译**.proto**时，会默认扫描src/main/proto文件夹下的protobuf文件。

例如我们需要定义一个用户的信息，包含的字段主要有编号、名称、年龄。
那么该**protobuf**文件的格式如下:
**注**：这里使用的是**proto3**，相关的注释我已写了，这里便不再过多讲述了。需要注意一点的是**proto**文件和生成的**Java**文件名称不能一致!

```protobuf
//proto3语法注解:如果您不这样做，protobuf编译器将假定您正在使用proto2,这必须是文件的第一个非空的非注释行。
syntax = "proto3";
//生成的包名
option java_package = "com.sanshengshui.netty.protobuf";
//生成的java名
option java_outer_classname = "UserMsg";

message User{
    //ID
    int32 id = 1;
    //姓名
    string name = 2;
    //年龄
    int32 age = 3;
    //状态
    int32 state = 4;
}
```

创建好该文件之后，我们**cd**到该工程的根目录下，执行**mvn clean compile**,输入完之后，回车即可在target文件夹中看到已经生成好的Java文件，然后直接在工程中使用此protobuf文件就可以了。因为能自动扫描到此类。详情请看下图:

![选区_003](/home/james/%E5%9B%BE%E7%89%87/%E9%80%89%E5%8C%BA_003.png)

**注：生成protobuf的文件软件和测试的protobuf文件我也整合到该项目中了，可以直接获取的。**

Java文件生成好之后，我们再来看怎么使用。
这里我就直接贴代码了，并且将注释写在代码中，应该更容易理解些吧。。。
**代码示例:**

```
@RunWith(JUnit4.class)
@Slf4j
public class NettySpringbootProtostuffApplicationTests {
    @Test
    public void ProtobufTest() throws IOException {
        UserMsg.User.Builder userInfo = UserMsg.User.newBuilder();
        userInfo.setId(1);
        userInfo.setName("mushuwei");
        userInfo.setName("24");
        UserMsg.User user = userInfo.build();
        // 将数据写到输出流
        ByteArrayOutputStream output = new ByteArrayOutputStream();
        user.writeTo(output);
        // 将数据序列化后发送
        byte[] byteArray = output.toByteArray();
        // 接收到流并读取
        ByteArrayInputStream input = new ByteArrayInputStream(byteArray);
        // 反序列化
        UserMsg.User userInfo2 = UserMsg.User.parseFrom(input);
        log.info("id:" + userInfo2.getId());
        log.info("name:" + userInfo2.getName());
        log.info("age:" + userInfo2.getAge());

    }
}
```

注：这里说明一点，因为**protobuf**是通过二进制进行传输，所以需要注意下相应的编码。还有使用**protobuf**也需要注意一下一次传输的最大字节长度。

**输出结果：**

```
17:28:07.914 [main] INFO com.sanshengshui.nettyspringbootprotostuff.NettySpringbootProtostuffApplicationTests - id:1
17:28:07.919 [main] INFO com.sanshengshui.nettyspringbootprotostuff.NettySpringbootProtostuffApplicationTests - name:24
17:28:07.919 [main] INFO com.sanshengshui.nettyspringbootprotostuff.NettySpringbootProtostuffApplicationTests - age:0
```

# Netty整合springboot并使用protobuf进行数据传输

### 说明：如果想直接获取工程那么可以直接跳到底部，通过[链接](https://github.com/sanshengshui/netty-learning-example/tree/master/netty-springboot-protobuf)下载工程代码。

## 开发准备

### **环境要求**

**JDK**:：1.8
**Netty**:：4.0或以上(不包括5)
**Protobuf**：3.0或以上

如果对Netty不熟的话，可以看看我之前写的netty 之 telnet HelloWorld 详解。大神请无视~。~
地址：https://www.cnblogs.com/sanshengshui/p/9726306.html

首先还是Maven的相关依赖:

```xml
 <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <netty-all.version>4.1.29.Final</netty-all.version>
        <protobuf.version>3.6.1</protobuf.version>
        <grpc.version>1.15.0</grpc.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--netty jar包导入-->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>${netty-all.version}</version>
        </dependency>

        <!--使用grpc优雅的编译protobuf-->
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java</artifactId>
            <version>${protobuf.version}</version>
        </dependency>

        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-netty</artifactId>
            <version>${grpc.version}</version>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-protobuf</artifactId>
            <version>${grpc.version}</version>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-stub</artifactId>
            <version>${grpc.version}</version>
        </dependency>

        <!--lombok用于日志,实体类的重复代码书写-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

添加了相应的maven依赖之后！我们还需要添加**grpc优雅的编译protobuf的插件**:

```xml
 <build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.5.0.Final</version>
            </extension>
        </extensions>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.5.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.7</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>2.2.1</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.0.2</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.0.0</version>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy-protoc</id>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>copy</goal>
                        </goals>
                        <configuration>
                            <artifactItems>
                                <artifactItem>
                                    <groupId>com.google.protobuf</groupId>
                                    <artifactId>protoc</artifactId>
                                    <version>${protobuf.version}</version>
                                    <classifier>${os.detected.classifier}</classifier>
                                    <type>exe</type>
                                    <overWrite>true</overWrite>
                                    <outputDirectory>${project.build.directory}</outputDirectory>
                                </artifactItem>
                            </artifactItems>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.5.0</version>
                <configuration>
                    <!--
                      The version of protoc must match protobuf-java. If you don't depend on
                      protobuf-java directly, you will be transitively depending on the
                      protobuf-java version that grpc depends on.
                    -->
                    <protocArtifact>com.google.protobuf:protoc:${protobuf.version}:exe:${os.detected.classifier}
                    </protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.0.0:exe:${os.detected.classifier}
                    </pluginArtifact>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                            <goal>test-compile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

此外我们还需要对**application.yml**配置文件作一点修改:

```xml
server:
  enabled: true
  bind_address: 0.0.0.0
  bind_port: 9876
  netty:
    #不进行内存泄露的检测
    leak_detector_level: DISABLED
    boss_group_thread_count: 1
    worker_group_thread_count: 12
    #最大负载大小
    max_payload_size: 65536
```

### 项目结构

```
  netty-springboot-protobuf
    ├── client
      ├── NettyClient.class -- 客户端启动类
      ├── NettyClientHandler.class -- 客户端逻辑处理类
      ├── NettyClientHandler.class -- 客户端初始化类
    ├── server 
      ├── NettyServer.class -- 服务端启动类
      ├── NettyServerHandler -- 服务端逻辑处理类
      ├── NettyServerInitializer -- 服务端初始化类
    ├── proto
      ├── user.proto -- protobuf文件
```

### 代码编写

代码模块主要分为服务端和客户端。
主要实现的业务逻辑：
服务端启动成功之后，客户端也启动成功，这时服务端会发送一条**protobuf**格式的信息给客户端，然后客户端给予相应的应答。客户端与服务端连接成功之后，客户端每个一段时间会发送心跳指令给服务端，告诉服务端该客户端还存过中，如果客户端没有在指定的时间发送信息，服务端会关闭与该客户端的连接。当客户端无法连接到服务端之后，会每隔一段时间去尝试重连，只到重连成功!

#### 服务端

首先是编写服务端的启动类，相应的注释在代码中写得很详细了，这里也不再过多讲述了。不过需要注意的是，在之前的我写的Netty文章中，是通过main方法直接启动服务端，因此是直接new一个对象的。而在和SpringBoot整合之后，我们需要将Netty交给springBoot去管理，所以这里就用了相应的注解。
**代码如下:**

```java
@Service("nettyServer")
@Slf4j
public class NettyServer {
    /**
     * 通过springboot读取静态资源,实现netty配置文件的读写
     */

    @Value("${server.bind_port}")
    private Integer port;

    @Value("${server.netty.boss_group_thread_count}")
    private Integer bossGroupThreadCount;

    @Value("${server.netty.worker_group_thread_count}")
    private Integer workerGroupThreadCount;

    @Value("${server.netty.leak_detector_level}")
    private String leakDetectorLevel;

    @Value("${server.netty.max_payload_size}")
    private Integer maxPayloadSize;

    private  ChannelFuture channelFuture;
    private  EventLoopGroup bossGroup;
    private  EventLoopGroup workerGroup;


    @PostConstruct
    public void init() throws Exception {
            log.info("Setting resource leak detector level to {}",leakDetectorLevel);
            ResourceLeakDetector.setLevel(ResourceLeakDetector.Level.valueOf(leakDetectorLevel.toUpperCase()));

            log.info("Starting Server");
            //创建boss线程组 用于服务端接受客户端的连接
            bossGroup = new NioEventLoopGroup(bossGroupThreadCount);
            // 创建 worker 线程组 用于进行 SocketChannel 的数据读写
            workerGroup = new NioEventLoopGroup(workerGroupThreadCount);
            // 创建 ServerBootstrap 对象
            ServerBootstrap b = new ServerBootstrap();
            //设置使用的EventLoopGroup
            b.group(bossGroup, workerGroup)
                    //设置要被实例化的为 NioServerSocketChannel 类
                    .channel(NioServerSocketChannel.class)
                    // 设置 NioServerSocketChannel 的处理器
                    .handler(new LoggingHandler(LogLevel.INFO))
                    // 设置连入服务端的 Client 的 SocketChannel 的处理器
                    .childHandler(new NettyServerInitializer());
            // 绑定端口，并同步等待成功，即启动服务端
            channelFuture = b.bind(port).sync();

            log.info("Server started!");

    }

    @PreDestroy
    public void shutdown() throws InterruptedException {
        log.info("Stopping Server");
        try {
            // 监听服务端关闭，并阻塞等待
            channelFuture.channel().closeFuture().sync();
        } finally {
            // 优雅关闭两个 EventLoopGroup 对象
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
        log.info("server stopped!");

    }

}

```

服务端主类编写完毕之后，我们再来设置下相应的过滤条件。
这里需要继承Netty中**ChannelInitializer**类，然后重写**initChannel**该方法，进行添加相应的设置，如心跳超时设置，传输协议设置，以及相应的业务实现类。
**代码如下:**

```java
  public class NettyServerInitializer extends ChannelInitializer<SocketChannel> {


    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline ph = ch.pipeline();

        //入参说明: 读超时时间、写超时时间、所有类型的超时时间、时间格式
        ph.addLast(new IdleStateHandler(5, 0, 0, TimeUnit.SECONDS));
        // 解码和编码，应和客户端一致
        //传输的协议 Protobuf
        ph.addLast(new ProtobufVarint32FrameDecoder());
        ph.addLast(new ProtobufDecoder(UserMsg.User.getDefaultInstance()));
        ph.addLast(new ProtobufVarint32LengthFieldPrepender());
        ph.addLast(new ProtobufEncoder());

        //业务逻辑实现类
        ph.addLast("nettyServerHandler", new NettyServerHandler());
    }
}

```

服务相关的设置的代码写完之后，我们再来编写主要的业务代码。
使用Netty编写业务层的代码，我们需要继承**ChannelInboundHandlerAdapter** 或**SimpleChannelInboundHandler**类，在这里顺便说下它们两的区别吧。
继承**SimpleChannelInboundHandler**类之后，会在接收到数据后会自动**release**掉数据占用的**Bytebuffer**资源。并且继承该类需要指定数据格式。
而继承**ChannelInboundHandlerAdapter**则不会自动释放，需要手动调用**ReferenceCountUtil.release()**等方法进行释放。继承该类不需要指定数据格式。
所以在这里，个人推荐服务端继承**ChannelInboundHandlerAdapter**，手动进行释放，防止数据未处理完就自动释放了。而且服务端可能有多个客户端进行连接，并且每一个客户端请求的数据格式都不一致，这时便可以进行相应的处理。
客户端根据情况可以继承**SimpleChannelInboundHandler**类。好处是直接指定好传输的数据格式，就不需要再进行格式的转换了。

**代码如下:**

```java
@Slf4j
public class NettyServerHandler extends ChannelInboundHandlerAdapter {
    /** 空闲次数 */
    private AtomicInteger idle_count = new AtomicInteger(1);
    /** 发送次数 */
    private AtomicInteger count = new AtomicInteger(1);


    /**
     * 建立连接时，发送一条消息
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        log.info("连接的客户端地址:" + ctx.channel().remoteAddress());
        UserMsg.User user = UserMsg.User.newBuilder().setId(1).setAge(24).setName("穆书伟").setState(0).build();
        ctx.writeAndFlush(user);
        super.channelActive(ctx);
    }

    /**
     * 超时处理 如果5秒没有接受客户端的心跳，就触发; 如果超过两次，则直接关闭;
     */
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object obj) throws Exception {
        if (obj instanceof IdleStateEvent) {
            IdleStateEvent event = (IdleStateEvent) obj;
            // 如果读通道处于空闲状态，说明没有接收到心跳命令
            if (IdleState.READER_IDLE.equals(event.state())) {
                log.info("已经5秒没有接收到客户端的信息了");
                if (idle_count.get() > 1) {
                    log.info("关闭这个不活跃的channel");
                    ctx.channel().close();
                }
                idle_count.getAndIncrement();
            }
        } else {
            super.userEventTriggered(ctx, obj);
        }
    }

    /**
     * 业务逻辑处理
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        log.info("第" + count.get() + "次" + ",服务端接受的消息:" + msg);
        try {
            // 如果是protobuf类型的数据
            if (msg instanceof UserMsg.User) {
                UserMsg.User user = (UserMsg.User) msg;
                if (user.getState() == 1) {
                    log.info("客户端业务处理成功!");
                } else if(user.getState() == 2){
                    log.info("接受到客户端发送的心跳!");
                }else{
                    log.info("未知命令!");
                }
            } else {
                log.info("未知数据!" + msg);
                return;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            ReferenceCountUtil.release(msg);
        }
        count.getAndIncrement();
    }

    /**
     * 异常处理
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

还有个服务端的启动类，之前是通过main方法直接启动， 不过这里改成了通过springBoot进行启动，差别不大。
**代码如下：**

```java
@SpringBootApplication
@ComponentScan({"com.sanshengshui.netty.server"})
public class NettyServerApp {
    /**
     * @param args
     */
    public static void main(String[] args) {
       SpringApplication.run(NettyServerApp.class);
    }
}

```

到这里服务端相应的代码就编写完毕了💡。

#### 客户端

客户端这边的代码和服务端的很多地方都类似，我就不再过多细说了，主要将一些不同的代码拿出来简单的讲述下。
首先是客户端的主类，基本和服务端的差不多，也就是多了监听的端口和一个监听器(用来监听是否和服务端断开连接，用于重连)。
主要实现的代码逻辑如下:

```
     /**
     * 重连
     */
    public void doConnect(Bootstrap bootstrap, EventLoopGroup eventLoopGroup) {
        try {
            if (bootstrap != null) {
                bootstrap.group(eventLoopGroup);
                bootstrap.channel(NioSocketChannel.class);
                bootstrap.option(ChannelOption.SO_KEEPALIVE, true);
                bootstrap.handler(new NettyClientInitializer());
                bootstrap.remoteAddress(host, port);
                f = bootstrap.connect().addListener((ChannelFuture futureListener) -> {
                    final EventLoop eventLoop = futureListener.channel().eventLoop();
                    if (!futureListener.isSuccess()) {
                        log.info("与服务端断开连接!在10s之后准备尝试重连!");
                        eventLoop.schedule(() -> doConnect(new Bootstrap(), eventLoop), 10, TimeUnit.SECONDS);
                    }
                });
                if(initFalg){
                    log.info("Netty客户端启动成功!");
                    initFalg=false;
                }
            }
        } catch (Exception e) {
            log.info("客户端连接失败!"+e.getMessage());
        }

    }
```

**注：监听器这块的实现用的是JDK1.8的写法。**

客户端过滤其这块基本和服务端一直。不过需要注意的是，传输协议、编码和解码应该一致，还有心跳的读写时间应该小于服务端所设置的时间。
改动的代码如下:

```
    ChannelPipeline ph = ch.pipeline();
        /*
         * 解码和编码，应和服务端一致
         * */
        //入参说明: 读超时时间、写超时时间、所有类型的超时时间、时间格式
        ph.addLast(new IdleStateHandler(0, 4, 0, TimeUnit.SECONDS));
```

客户端的业务代码逻辑。
主要实现的几点逻辑是心跳按时发送以及解析服务发送的**protobuf**格式的数据。
这里比服务端多个个注解， 该注解**Sharable**主要是为了多个handler可以被多个channel安全地共享，也就是保证线程安全。
废话就不多说了，代码如下：

```
    @ChannelHandler.Sharable
@Slf4j
public class NettyClientHandler extends ChannelInboundHandlerAdapter {
    @Autowired
    private NettyClient nettyClient;

    /** 循环次数 */
    private AtomicInteger fcount = new AtomicInteger(1);

    /**
     * 建立连接时
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        log.info("建立连接时：" + new Date());
        ctx.fireChannelActive();
    }

    /**
     * 关闭连接时
     */
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        log.info("关闭连接时：" + new Date());
        final EventLoop eventLoop = ctx.channel().eventLoop();
        nettyClient.doConnect(new Bootstrap(), eventLoop);
        super.channelInactive(ctx);
    }

    /**
     * 心跳请求处理 每4秒发送一次心跳请求;
     *
     */
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object obj) throws Exception {
        log.info("循环请求的时间：" + new Date() + "，次数" + fcount.get());
        if (obj instanceof IdleStateEvent) {
            IdleStateEvent event = (IdleStateEvent) obj;
            // 如果写通道处于空闲状态,就发送心跳命令
            if (IdleState.WRITER_IDLE.equals(event.state())) {
                UserMsg.User.Builder userState = UserMsg.User.newBuilder().setState(2);
                ctx.channel().writeAndFlush(userState);
                fcount.getAndIncrement();
            }
        }
    }

    /**
     * 业务逻辑处理
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        // 如果不是protobuf类型的数据
        if (!(msg instanceof UserMsg.User)) {
            log.info("未知数据!" + msg);
            return;
        }
        try {

            // 得到protobuf的数据
            UserMsg.User userMsg = (UserMsg.User) msg;
            // 进行相应的业务处理。。。
            // 这里就从简了，只是打印而已
            log.info(
                    "客户端接受到的用户信息。编号:" + userMsg.getId() + ",姓名:" + userMsg.getName() + ",年龄:" + userMsg.getAge());

            // 这里返回一个已经接受到数据的状态
            UserMsg.User.Builder userState = UserMsg.User.newBuilder().setState(1);
            ctx.writeAndFlush(userState);
            log.info("成功发送给服务端!");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            ReferenceCountUtil.release(msg);
        }
    }

}
```

那么到这里客户端的代码也编写完毕了💡。

### 功能测试

#### protobuf传输

首先启动服务端，然后再启动客户端。
我们来看看结果是否如上述所说。

**服务端输出结果:**

```
2018-10-03 19:58:41.098  INFO 23644 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 第1次,服务端接受的消息:state: 1

2018-10-03 19:58:41.098  INFO 23644 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 客户端业务处理成功!
2018-10-03 19:58:45.058  INFO 23644 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 第2次,服务端接受的消息:state: 2

2018-10-03 19:58:45.059  INFO 23644 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 接受到客户端发送的心跳!
2018-10-03 19:58:49.060  INFO 23644 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 第3次,服务端接受的消息:state: 2

2018-10-03 19:58:49.061  INFO 23644 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 接受到客户端发送的心跳!
2018-10-03 19:58:53.063  INFO 23644 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 第4次,服务端接受的消息:state: 2

2018-10-03 19:58:53.064  INFO 23644 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 接受到客户端发送的心跳!
2018-10-03 19:58:57.066  INFO 23644 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 第5次,服务端接受的消息:state: 2
```

**客户端输入结果:**

```
2018-10-03 19:58:40.733  INFO 23737 --- [           main] c.sanshengshui.netty.client.NettyClient  : Netty客户端启动成功!
2018-10-03 19:58:40.897  INFO 23737 --- [ntLoopGroup-2-1] c.s.netty.client.NettyClientHandler      : 建立连接时：Wed Oct 03 19:58:40 CST 2018
2018-10-03 19:58:41.033  INFO 23737 --- [ntLoopGroup-2-1] c.s.netty.client.NettyClientHandler      : 客户端接受到的用户信息。编号:1,姓名:穆书伟,年龄:24
2018-10-03 19:58:41.044  INFO 23737 --- [ntLoopGroup-2-1] c.s.netty.client.NettyClientHandler      : 成功发送给服务端!
2018-10-03 19:58:41.053  INFO 23737 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2018-10-03 19:58:41.067  INFO 23737 --- [           main] com.sanshengshui.netty.NettyClientApp    : Started NettyClientApp in 1.73 seconds (JVM running for 2.632)
2018-10-03 19:58:45.054  INFO 23737 --- [ntLoopGroup-2-1] c.s.netty.client.NettyClientHandler      : 循环请求的时间：Wed Oct 03 19:58:45 CST 2018，次数1
2018-10-03 19:58:49.057  INFO 23737 --- [ntLoopGroup-2-1] c.s.netty.client.NettyClientHandler      : 循环请求的时间：Wed Oct 03 19:58:49 CST 2018，次数2
2018-10-03 19:58:53.060  INFO 23737 --- [ntLoopGroup-2-1] c.s.netty.client.NettyClientHandler      : 循环请求的时间：Wed Oct 03 19:58:53 CST 2018，次数3
2018-10-03 19:58:57.063  INFO 23737 --- [ntLoopGroup-2-1] c.s.netty.client.NettyClientHandler      : 循环请求的时间：Wed Oct 03 19:58:57 CST 2018，次数4
2018-10-03 19:59:01.066  INFO 23737 --- [ntLoopGroup-2-1] c.s.netty.client.NettyClientHandler      : 循环请求的时间：Wed Oct 03 19:59:01 CST 2018，次数5

```

通过打印信息可以看出如上述所说。

#### 断线重连

接下来我们再来看看客户端是否能够实现重连。
先启动客户端，再启动服务端。

**客户端输入结果:**

```repl
2018-10-03 20:02:33.549  INFO 23990 --- [ntLoopGroup-2-1] c.sanshengshui.netty.client.NettyClient  : 与服务端断开连接!在10s之后准备尝试重连!
2018-10-03 20:02:43.571  INFO 23990 --- [ntLoopGroup-2-1] c.s.netty.client.NettyClientHandler      : 建立连接时：Wed Oct 03 20:02:43 CST 2018
2018-10-03 20:02:43.718  INFO 23990 --- [ntLoopGroup-2-1] c.s.netty.client.NettyClientHandler      : 客户端接受到的用户信息。编号:1,姓名:穆书伟,年龄:24
2018-10-03 20:02:43.727  INFO 23990 --- [ntLoopGroup-2-1] c.s.netty.client.NettyClientHandler      : 成功发送给服务端!
2018-10-03 20:02:47.733  INFO 23990 --- [ntLoopGroup-2-1] c.s.netty.client.NettyClientHandler      : 循环请求的时间：Wed Oct 03 20:02:47 CST 2018，次数1
2018-10-03 20:02:51.735  INFO 23990 --- [ntLoopGroup-2-1] c.s.netty.client.NettyClientHandler      : 循环请求的时间：Wed Oct 03 20:02:51 CST 2018，次数2

```

**服务端输出结果:**

```
2018-10-03 20:02:43.661  INFO 24067 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 连接的客户端地址:/127.0.0.1:55690
2018-10-03 20:02:43.760  INFO 24067 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 第1次,服务端接受的消息:state: 1

2018-10-03 20:02:43.760  INFO 24067 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 客户端业务处理成功!
2018-10-03 20:02:47.736  INFO 24067 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 第2次,服务端接受的消息:state: 2

2018-10-03 20:02:47.737  INFO 24067 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 接受到客户端发送的心跳!
2018-10-03 20:02:51.736  INFO 24067 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 第3次,服务端接受的消息:state: 2
```

结果也如上述所说！

#### 读写超时

**服务端输出结果:**

```
2018-10-03 20:12:19.193  INFO 24507 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 连接的客户端地址:/127.0.0.1:56132
2018-10-03 20:12:24.173  INFO 24507 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 已经5秒没有接收到客户端的信息了
2018-10-03 20:12:29.171  INFO 24507 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 已经5秒没有接收到客户端的信息了
2018-10-03 20:12:29.172  INFO 24507 --- [ntLoopGroup-3-1] c.s.netty.server.NettyServerHandler      : 关闭这个不活跃的channel
```

**telnet输出结果:**

如下图:

![选区_004]()

## 其它

关于netty整合springboot并使用protobuf进行数据传输到这里就结束了。

netty整合springboot并使用protobuf进行数据传输 项目工程地址: <https://github.com/sanshengshui/netty-learning-example/tree/master/netty-springboot-protobuf>

对了，也有Netty整合的其他中间件项目工程地址: <https://github.com/sanshengshui/netty-learning-example>

原创不易，如果感觉不错，希望给个推荐！您的支持是我写作的最大动力！

版权声明: 作者：穆书伟 

博客园出处：<https://www.cnblogs.com/sanshengshui> 

github出处：<https://github.com/sanshengshui>　　　　

 个人博客出处：<https://sanshengshui.github.io/>