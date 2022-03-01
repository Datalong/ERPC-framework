          

[Chinese](https://github.com/Datalong/ERPC-framework/new/master/readme.md) | English

## To talk about🏆 Design a lightweight distributed RPC framework from scratch
### 📰 Write in front

Based on Spring + netty + Nacos + kyro, this project designs and implements a lightweight distributed RPC framework from scratch, including detailed design ideas and development tutorials,  learn by building wheels, so as to deeply understand the underlying principle of RPC framework.Compared with the same XXXX system on the resume, making wheels can obviously win the favor of interviewers💖

Of course, we should make fewer wheels in the actual project and try to use the ready-made excellent framework to save a lot of trouble

In the spirit of open source, the English version of readme of this project has been synchronized.In addition, most of the comments of the source code of the project are also modified to English.

If the access speed is poor, it can be placed at gitee address:https://gitee.com/Datalong/erpc-framework。If you want to submit issue or PR, please submit it on GitHub:https://github.com/Datalong/erpc-framework/issues。

Project recommendation👍：

1.[LeetCode_Offer](https://gitee.com/Datalong/leet-code--offer）(Graphic high frequency algorithm is being updated...)

2.[CodeGuide](https://github.com/Datalong/CodeGuide）: knowledge management and resource site, welcome to visit and exchange comments.

3.[BuildSkill](https://gitee.com/Datalong/build-skill）: design a high-performance distributed second kill system from scratch with detailed tutorials. Welcome to star!


To talk about👝 preface
Although the principle of RPC is not difficult in practice, I have encountered many problems in the process of implementation.At present, ERPC framework only realizes the most basic functions of RPC framework. I will mention some optimization points below. Interested partners can improve it by themselves.

Through this simple wheel, you can learn the underlying principles and principles of RPC and the application of various java coding practices.

You can even use ERPC framework as your choice of design / project experience, which is also very good!Compared with other job seekers, the project experience is all kinds of systems, making wheels is certainly more able to win the favor of interviewers.

If you want to take ERPC framework as your design / project experience, I hope you must stick to it and understand the code instead of directly copying and pasting my ideas.Then you can fork my project and optimize it.If you think the optimization is valuable, you can submit PR to me and I will deal with it as soon as possible.

## What features are implemented
-Two network transmission modes based on Java Native socket transmission and netty transmission are realized
-Four serialization algorithms are implemented, including Jason method, kryo algorithm, Hessian algorithm and Google protobuf method (kryo method serialization is adopted by default here)
-Two load balancing algorithms are implemented: random algorithm and rotation algorithm
-Use Nacos as the registry to manage the service provider information, including the remote address
-If the customer adopts netty mode, it will reuse the channel to avoid multiple connections
-For example, both consumers and providers adopt netty mode, and the heartbeat mechanism of netty will be adopted to ensure continuous connection
-Good interface abstraction, low module coupling, configurable network transmission, serializer and load balancing algorithm
-Implement custom communication protocol
-Service provider side automatic registration service

## Project module overview
` ` ` `

ROC API -- General Interface
RPC common -- common classes such as entity objects and tool classes
RPC core -- the core implementation of the framework
Test client -- consumer side for test
Test server - Test side

` ` ` `
### 2 transmission protocol (MRF protocol)
The transfer of call parameters and return values adopts the following ERF protocol (ERPC framework initials) to prevent packet sticking:

```
+---------------+---------------+-----------------+-------------+
|  Magic Number |  Package Type | Serializer Type | Data Length |
|    4 bytes    |    4 bytes    |     4 bytes     |   4 bytes   |
+---------------+---------------+-----------------+-------------+
|                          Data Bytes                           |
|                   Length: ${Data Length}                      |
+---------------------------------------------------------------+
```

### 3 field interpretation
Magic number indicates an ERF protocol package, 0xcafebabe

Package type = package type, indicating whether this is a call request or a call response

Serializer type = serializer type, indicating the serialization method of the data of this package

Data length length of data bytes

The object transmitted by data bytes is usually an rpcrequest or rpcclient object, which depends on the package type field, and the serialization method of the object depends on the serializer type field.

## To talk about🎨 What do you need

### 1. A basic RPC framework design idea

>Note: the RPC framework we mentioned here refers to a framework that allows the client to call the server method directly, just like calling the local method, such as Dubbo, Motan and grpc.If you need to deal with HTTP protocol, parse and encapsulate HTTP request and response.Such frameworks are not "RPC frameworks", such as feign.

The use diagram of the simplest RPC framework is shown in the figure below, which is also the current architecture of ERPC framework:

![]https://gitee.com/Datalong/picture/raw/master/2022-2-11/1644574339818-2.png)

The service provider server registers the service with the registry, and the service consumer client gets the service related information through the registry, and then requests the service provider server through the network.

As a leader in the field of RPC framework, Dubbo's architecture is shown in the figure below, which is roughly the same as what we drew above.

![]https://gitee.com/Datalong/picture/raw/master/2022-2-11/1644574566797-3.png)

Generally, RPC framework should not only provide service discovery function, but also provide load balancing, fault tolerance and other functions. Only such RPC framework can be truly qualified.

### 2 here is a brief description of what you need

![]https://gitee.com/Datalong/picture/raw/master/2022-2-11/1644574603800-rpc-architure-detail.png)

- 1.Registration Center: first of all, there should be a registration center. The registration center is responsible for the registration and search of service address, which is equivalent to directory service.When the server starts, it registers the service name and its corresponding address (IP + port) in the registration center, and the service consumer finds the corresponding service address according to the service name.With the service address, the service consumer can request the server through the network.

👍Nacos (produced by ALI) is recommended as the registration center

Nacos provides us with distributed data consistency solutions with high availability, high performance and strong compatibility, which are usually used to realize functions such as data publish / subscribe, load balancing, naming service, distributed coordination / notification, cluster management, master election, distributed lock and distributed queue.In addition, * * zookeeper saves data in memory, and its performance is very good * *.This is particularly high performance in applications that "read" more than "write", because "write" causes synchronization between all servers.(reading more than writing is a typical scenario for coordinating services).

- 2.Network transmission: since you want to call a remote method, you need to send a request. The request should at least include the class name, method name and relevant parameters you call!Netty framework based on NiO is recommended.The way the consumer calls the provider depends on the choice of the consumer's client. If the native socket is selected, bio will be used in this step. If netty is selected, NiO will be used in this step.If the call has a return value, the provider sends the return value to the consumer in the same way.

1) Socket: the most primitive and basic network communication mode in Java.However, socket is Io blocking, low performance and single function

2) NiO: synchronous non blocking I / O model, but it's really troublesome to use it for network programming

3) Netty: NiO based client server framework, which can be used to quickly and simply develop network applications.It greatly simplifies and simplifies network programming such as TCP and UDP socket server, and even better in many aspects such as performance and security.Support a variety of protocols, such as FTP, SMTP, HTTP and various binary and text-based traditional protocols.

👍Netty highly available network application tool is recommended

In other words, netty is a client-side and server-side programming framework based on NiO. Using netty can ensure that you can quickly and simply develop a network application, such as a client and server-side application that implements a certain protocol.Netty is equivalent to simplifying and streamlining the programming and development process of network applications, such as the development of socket services based on TCP and UDP.Among them, "fast" and "simple" do not cause maintainability or performance problems.Netty is a carefully designed project that absorbs the implementation experience of a variety of protocols (including FTP, SMTP, HTTP and other binary text protocols).Finally, netty successfully found a way to ensure the performance, stability and scalability of its application while ensuring easy development.


- 3.Serialization: since network transmission is involved, serialization must be involved. In some other scenarios, we also need to store objects in files, databases, etc.You can't directly use the serialization provided by JDK!Of course, you can. Just implement 'JavaIo.Serializable 'interface is OK, but the serialization efficiency of JDK is low, there are security vulnerabilities, and it cannot be called across languages.Because the data transmitted by the network must be binary.Therefore, our Java objects cannot be transmitted directly in the network.In order to enable Java objects to be transmitted in the network, we need to * * serialize * * them into binary data.What we ultimately need is the target Java object, so we also need to "parse" the binary data into the target Java object, that is to * * deserialize * * the binary data again.Therefore, you should also consider which serialization protocol to use.

👍Hession2, kyro and protostuff are commonly used. Let's choose kryo serialization framework.

- 4.Dynamic proxy: in addition, dynamic proxy is also required.Because the main purpose of RPC is to make calling remote methods as simple as calling local methods. Using dynamic proxy can mask the details of remote method calls, such as network transmission.In other words, when you call a remote method, you will actually transmit the network request through the proxy object. Otherwise, how can you call the remote method directly?Here I also want to explain what dynamic proxy means: provide a proxy object for an object and let the proxy object do something instead of the real object.You can understand the proxy object as a tool behind the scenes.For example: when we call methods from real objects, we can do some things through proxy objects, such as security verification, log printing and so on.However, this process is completely shielded from real objects.

👍Dynamic proxy mechanisms include JDK dynamic proxy, cglib dynamic proxy, javassist dynamic proxy, etc

- 5.Load balancing: load balancing is also needed.Why?For example, the access volume of a service in our system is particularly large. We deploy this service on multiple servers. When the client initiates a request, multiple servers can process the request.Therefore, how to correctly select the server to process the request is very important.If you need a server to handle the request of the service, the significance of deploying the service on multiple servers will no longer exist.Load balancing is to prevent a single server from responding to the same request, which is easy to cause server downtime, crash and other problems. We can clearly feel its significance from the four words of load balancing.

-  6. Transmission / communication protocol

We also need to design a private RPC Protocol (communication / transmission protocol), which is the basis for the communication between the client (service consumer) and the server (service provider).

Simply put: by designing the transmission protocol, we define which types of data need to be transmitted, and also specify how many bytes each type of data should occupy.In this way, after receiving the secondary system data, we can correctly analyze the data we need.

Generally, some standard RPC protocols include the following contents:

- Magic Number: usually 4 bytes.This magic number is mainly used to filter the data packets coming to the server. After having this magic number, the server first takes out the first four bytes for comparison, which can identify that the data packet does not follow the user-defined protocol, that is, invalid data packets. For security reasons, you can directly close the connection to save resources.
-Serializer number: identifies the serialization method, for example, whether to use the serialization provided by Java or JSON, kyro and other serialization methods.
-Message body length: calculated at runtime.
- ..........

## To talk about🎂 Technical foreshadowing required

This project is based on netty + kyro + Spring + Nacos. To learn this project, you need the following technical reserves:

-🔸 Java Foundation
See [codeguide] for related tutorials（https://github.com/Datalong/CodeGuide)

-Dynamic agent mechanism
-Java I / O system
-Serialization and serialization framework (kyro...)Basic use of
-Java network programming (socket programming)
-Multithreading, Java concurrency
-Java reflection
-Java annotation
- ..........
-🔸 Netty 4.x: It makes NiO programming easier and shields the NiO details at the bottom of Java

See [netty tutorial] for related tutorials（https://www.bilibili.com/video/BV1DJ411m7NR?from=search&seid=11475802238117137240&spm_id_from=333.337.0.0)
  
-🔸 Nacos: it provides service registration and discovery functions, is a necessary choice for developing distributed systems, and has natural clustering ability

See [Turing Nacos tutorial] for relevant tutorials（https://www.bilibili.com/video/BV1fR4y1M7ZK?from=search&seid=11385608076957522410&spm_id_from=333.337.0.0)

-🔸 Spring framework (spring): the most powerful dependency injection framework, which is widely selected in the industry

See [Raytheon springboot2] for related tutorials（https://www.bilibili.com/video/BV19K4y1L7MT?from=search&seid=17207560233934484749&spm_id_from=333.337.0.0)


### To talk about🎁 Project address | embrace open source

I have to say that open source has really greatly improved our productivity and learning efficiency (at least for me)

Project source code address:

- Recommended visit gitee: [ERPC framework]（https://gitee.com/Datalong/erpc-framework)
- Next, visit GitHub: [ERPC framework]（https://github.com/Datalong/ERPC-framework)

## 🤔 basic information and optimization points of the project
In order to step by step, at first, I used socket for network transmission based on the traditional bio method, and then used the serialization mechanism of JDK to implement this RPC framework.Later, I optimized the original version. I listed the completed optimization points and the optimization points that can be completed below👇。

Why list the optimization points?I mainly want to give some ideas to those small partners who want to optimize this RPC framework.Welcome to fork this warehouse and optimize it yourself.

- Use netty (NiO based) instead of bio to realize network transmission;
 
- Use the open source serialization mechanism kyro (or others) to replace the serialization mechanism of JDK;

- Use Nacos to manage relevant service address information
 
- Netty reuses the channel to avoid repeated connection to the server

- Use 'completabilefuture' to wrap and accept the results returned by the client (the previous implementation was implemented by binding 'attributemap' to the channel). See details: use completabilefuture to optimize the results returned by the receiving service provider

- Add netty heartbeat mechanism: ensure that the connection between the client and the server is not broken and avoid reconnection.

- Load balancing when the client calls the remote service: when calling the service, select a service address from many service addresses according to the corresponding load balancing algorithm.PS: at present, random load balancing algorithm and consistent hash algorithm are implemented.

- deal with the situation that an interface has multiple class implementations: group services and add a group parameter when publishing services.
 
- integrate spring to register services through annotations
 
- integrate spring to consume services through annotations.
 
- increase the service version number: it is recommended to use a two digit version, such as 1.0. Generally, the version number needs to be upgraded only when the interface is incompatible.Why increase the service version number?Make it possible for subsequent incompatible upgrades. For example, adding methods to the service interface or adding fields to the service model can be backward compatible. Deleting methods or fields will be incompatible, and the new fields of enumeration types will also be incompatible. It needs to be upgraded by changing the version number.
 
- Application of SPI mechanism
 
- Add configurable methods, such as serialization and registry implementation, to avoid hard coding: configure through API. If spring is integrated later, it is recommended to use configuration file for configuration
 
- The communication protocol (packet structure) between the client and the server is redesigned. The original rpcrequest and rpcreuqest objects can be used as the message body, and then the following fields are added (refer to the design of this part in the introduction to netty and Dubbo framework):
- Magic Number: usually 4 bytes.This magic number is mainly used to filter the data packets coming to the server. After having this magic number, the server first takes out the first four bytes for comparison, which can identify that the data packet does not follow the user-defined protocol, that is, invalid data packets. For security reasons, you can directly close the connection to save resources.
- Serializer number: identifies the serialization method, for example, whether to use the serialization provided by Java or JSON, kyro and other serialization methods.
- Message body length: calculated at runtime.

- Writing tests provides confidence in refactoring code
 
- Service monitoring center (similar to Dubbo admin)
 
- Set gzip compression


## 🛠 formal operation project
### 1 import project
Fork the project to your own warehouse, and then clone the project to your local: ` git clonegit@github.com:username/erpc-framework. Git `, open it with idea and wait for the project initialization to complete.
###Initialize git hooks
This step is mainly to run check style before committing the code to ensure that there is no problem with the code format. If there is a problem, it cannot be submitted.

>The following shows the corresponding operation of MAC / Linux. Window users need to manually copy the 'pre commit' file under the 'config / git hooks' directory to the' pre commit 'file under the projectGit / hooks / ` directory.

Execute these commands:
```sh

➜  erpc-framework git:(master) ✗ chmod +x ./init.Sh
➜  erpc-framework git:(master) ✗ ./init.Sh

```

`init.SH ` the main function of this script is to copy the GIT commit hook to the projectGit / hooks / ` directory, so that you will execute it every time you commit.
###Checkstyle plug-in download and configuration
`IntelliJ idea - > Preferences - > plugins - > search and download checkstyle plug-in , and then configure it as follows.

![]https://gitee.com/Datalong/picture/raw/master/2022-2-11/1644575555001-4.png)

After configuration, use the plug-in as follows!

![]https://gitee.com/Datalong/picture/raw/master/2022-2-11/1644575597601-5.png)

### 2 download and run Nacos
Docker is used here to download and install.
####Search for images of Nacos

```sh

# docker search nacos

```
#### The stable version is recommended (official recommendation 1.3.1). If the version is not specified, it is the latest version (corresponding to Nacos version 1.4)
```sh

# docker pull nacos/nacos-server

```
#### View all downloaded image packages in docker:
```sh

# docker images

```

![]https://gitee.com/Datalong/picture/raw/master/2022-2-11/1644576669508-Si.png)

#### Start command:
```sh

docker run -d -e prefer_host_mode=127.0.0.1 -e MODE=standalone -v /nacos/logs:/opt/software/nacos/logs -p 8848:8848 --name nacos --restart=always nacos/nacos-server

```
- Detailed explanation of parameters:

- D background operation
- E environment variable setting
- V directory of a container: map a directory on CentOS (according to the actual settings)
- P external access port: internal mapped port (according to actual settings)
- Name the name of the container
-Restart restart policy

####Check the startup status and log of Nacos:
To view the containers that docker has started:
`#Docker PS ` # to view all containers, add the parameter ` - A`

![]https://gitee.com/Datalong/picture/raw/master/2022-2-11/1644576751371-6.png)

To view the output log of the container specified by docker:

```sh

#Docker logs -- since 10m, Nacos # 10m is the time parameter, and Nacos is the container name or ID

```
Open Nacos 8848 port external connection:

Firewall open '8848' port:

Check whether a port of the firewall is open
` firewall-cmd --query-port=8848/tcp`

![]https://gitee.com/Datalong/picture/raw/master/2022-2-11/1644576889050-7.png)

Open firewall port 8848 and restart the firewall to take effect

```sh

# firewall-cmd --zone=public --add-port=8848/tcp --permanent

```

service iptables restart 
```sh

# systemctl restart firewalld

```

### 3. Access the Nacos management interface:
http://xxx.xxx.xx.xxx:8848/nacos/#/login

User name / password: Nacos / Nacos

>At this point, the download and installation of Nacos in docker have been completed.The following describes how to modify the Nacos configuration

###4. Modify the configuration of Nacos:

Docker PS # found Nacos container ID

Docker exec - it 1f392d60d21c / bin / bash # enters the container of nacso

Exit # exits the container

![]https://gitee.com/Datalong/picture/raw/master/2022-2-11/1644577320705-8.png)


To talk about🎨Official use
###1. Service provider
Implementation interface:
```java

@Slf4j
@RpcService(group = "test1", version = "version1")
public class HelloServiceImpl implements HelloService {
    static {
        System.out.Println ("helloserviceimpl created");
    }

    @Override
    public String hello(Hello hello) {
        log.Info ("helloserviceimpl received: {}.",hello.getMessage());
        String result = "Hello description is " + hello.getDescription();
        log.Info ("helloserviceimpl returns: {}.",result);
        return result;
    }
}
	
@Slf4j
public class HelloServiceImpl2 implements HelloService {

    static {
        System.out.Println ("helloserviceimpl2 is created");
    }

    @Override
    public String hello(Hello hello) {
        log.Info ("helloserviceimpl2 received: {}.",hello.getMessage());
        String result = "Hello description is " + hello.getDescription();
        log.Info ("helloserviceimpl2 returns: {}.",result);
        return result;
    }
}

```
Publishing service (transport using netty):
```java

* *
 * Server: Automatic registration service via @RpcService annotation
*
 * @author shuang.Kou
* @ createtime 07:25:00, May 10, 2020
 */
@RpcScan(basePackage = {"github.javaguide.serviceimpl"})
public class NettyServerMain {
    public static void main(String[] args) {
        // Register service via annotation
        new AnnotationConfigApplicationContext(NettyServerMain.class);
        NettyServer nettyServer = new NettyServer();
        // Register service manually
        HelloService helloService2 = new HelloServiceImpl2();
        RpcServiceProperties rpcServiceConfig = RpcServiceProperties.builder()
                .group("test2").version("version2").build();
        nettyServer.registerService(helloService2, rpcServiceConfig);
        nettyServer.start();
    }
}

``` 
### 2. Service consumer
```java

@Component
public class HelloController {

    @RpcReference(version = "version1", group = "test1")
    private HelloService helloService;

    public void test() throws InterruptedException {
        String hello = this.helloService.hello(new Hello("111", "222"));
/ / to use the assert assertion, add the parameter: - EA in VM options
        assert "Hello description is 222".equals(hello);
        Thread.sleep(12000);
        for (int i = 0; i < 10; i++) {
            System.out.println(helloService.hello(new Hello("111", "222")));
        }
    }
}

``` 
```java
ClientTransport rpcRequestTransport = new SocketRpcClient();
RpcServiceProperties rpcServiceConfig = RpcServiceProperties.builder()
        .group("test2").version("version2").build();
RpcClientProxy rpcClientProxy = new RpcClientProxy(rpcRequestTransport, rpcServiceConfig);
HelloService helloService = rpcClientProxy.getProxy(HelloService.class);
String hello = helloService.hello(new Hello("111", "222"));
System.out.println(hello);
```
To talk about📢 Related issues
##### Why build this wheel?Doesn't Dubbo smell good?

I wrote this RPC framework mainly to study deeply by making wheels and test my application of the knowledge I have mastered.

It's actually easier to implement a simple RPC framework, but it's still more difficult than handwritten AOP and IOC. The premise is that you understand the basic principle of RPC.

I have previously shared how to implement an RPC from a theoretical level.However, the theoretical level is only support. If you understand the theory, you may only fool the interviewer.Our programmer industry still needs hands-on ability most, even if you are an architect level person.When you start to practice something and put the theory into practice, you will find many pits waiting for you.

In practical projects, we should try to build fewer wheels and use them as soon as possible after there is an excellent framework. Dubbo has done a good and perfect job in all aspects.

## To talk about🙋 Wechat communication group

Underneath scan code, I will pay attention to the official account reply to `ERPC`. There is my contact information. Note "ERPC" plus my WeChat. I pull you into WeChat WeChat communication group, follow up the project schedule in real time, get the tutorial updates, share your thoughts, and help you solve the problems, but there is a lot of them.

<img width="220px" src="https://gitee.com/Datalong/picture/raw/master/2022-3-1/1646127555917-gongzhon.jpg"  /> 

### To talk about😁 Acknowledge

Bloggers have limited level and do not have good architecture ability. If you find logical errors in the code after fork, you can actively contact me or mention PR / issue. After adoption, you will appear in the list below.Thanks to the following small partners for their contributions to the project. The ranking is in chronological order.


Friendship link (if you want to appear here, you can scan wechat QR code above to contact me):
 
- [LeetCode_Offer](https://gitee.com/Datalong/leet-code--offer）(Graphic high frequency algorithm is being updated...)

- [CodeGuide](https://github.com/Datalong/CodeGuide）Knowledge management and resource site, welcome to visit and exchange comments.

- [BuildSkill](https://gitee.com/Datalong/build-skill）Design a high-performance distributed second kill system from scratch with detailed tutorials. Welcome to star!

- [Furion](https://gitee.com/dotnetchina/Furion）: letNet development is simpler, more general and more popular

- [Free-Fs](https://gitee.com/dh_free/free-fs）: spring boot open source cloud file management system, convenient and fast management of cloud stored files


