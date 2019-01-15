# 什么是RPC
RPC：Remote Produre Call-远程过程调用，像调用本地方法一样调用远程方法
# RPC原理
RPC采用客户端（服务调用方）/服务端（服务提供方）模式，各自独自运行。客户端需要通过引用需要使用的接口，接口的实现和运行都是在服务端。RPC主要的依赖的技术包括序列化、反序列化和数据传输协议。
![2019-01-14.17.45.32-image.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2019-01-14.17.45.32-image.png)
## 一些基础概念
- `RMI(Remote Method Invoke，远程方法调用)`代理模式通过代理对象将方法传递给实际对象；
- `stub(桩)`驻留客户端，承担远程对象实现者的角色；
- `skeleton(骨架)`帮助远程对象与stub连接进行通信。
## RPC时序图
![2019-01-15.12.45.18-image.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2019-01-15.12.45.18-image.png)
关于时序图的一些解释：
- 1.客户端（client）通过本地调用的方式调用服务
- 2.`client stub`接收到调用方法之后将参数、方法封装成能够网络传输的消息体（序列化）
- 3.`client stub`找到服务地址，并将消息发送到服务端
- 4.`server stub`接收消息并解码（反序列化）
- 5.`server stub`根据解码结果调用本地服务
- 6.本地服务奖调用结果返回给`server stub`
- 7.`server stub`将返回结果打包成消息（序列化）发送至调用方
- 8.`client stub`接收消息并解码（反序列化）
- 9.客户端得到最终结果
# 常见RPC框架
常见RPC框架方案有RMI（JDK自带），Hessian，Dubbo，Hprose，Thrift，HTTP等
## RMI
### RMI调用过程
![2019-01-15.15.54.19-image.png](https://raw.githubusercontent.com/jerryjoejj/yosoro-pic/master/img/2019-01-15.15.54.19-image.png)
- 1.服务端向RMI注册服务绑定自己的地址
- 2.客户端通过RMI注册服务获取目标地址
- 3.客户端调用本地stub对象上的方法，与调用本地方法一致
- 4.本地stub对象将调用信息打包（序列化），通过网络发送给服务端
- 5.服务端skeleton对象接收到网络请求，对调用信息进行解包（反序列化）
- 6.服务端真正服务调用发起调用，并将返回结果打包（序列化）发送给客户端
- 7.客户端解包（反序列化）结果，获取服务调用结果
### Java RMI基本概念
在RMI调用中，有以下几个核心概念
> 1.通过接口进行远程调用<br>
2.通过客户端stub对象和服务端skeleton对象帮助远程调用伪装成本地调用<br>
3.通过RMI注册服务完成服务的注册与发现
### Java RMI示例
#### 接口
HelloService
```java
// 必须继承Remote接口
public interface HelloService extends Remote {
    String sayHello() throws RemoteException;
}
```
#### 接口实现
```java
// 必须继承UnicastRemoteObject类
public class HelloServiceImpl extends UnicastRemoteObject implements HelloService {

	// 必须实现构造方法
    public HelloServiceImpl() throws RemoteException {
        super();
    }

    public String sayHello() throws RemoteException {
        return "Hello World";
    }
}
```
#### 服务端（Server）
```java
public class RmiServer {
    public static void main(String[] args) throws RemoteException {
    	// 初始化服务对象示例
        HelloService helloService = new HelloServiceImpl();
        // 创建一个本地RMI服务，监听1099端口
        Registry registry = LocateRegistry.createRegistry(1099);
        // 将示例对象注册绑定到服务商，客户端可以通过Hello这个名字查找远程对象
        registry.rebind("Hello", helloService);
    }
}
```
#### 客户端（Client）
```java
public class RmiClient {
    public static void main(String[] args) throws RemoteException, NotBoundException {
    	// 获取注册服务示例，这里假定获取的注册服务部署在本机上，并监听1099端口
        Registry registry = LocateRegistry.getRegistry();
        // 从注册中心查找服务名为Hello的远程对象
        HelloService helloService = (HelloService) registry.lookup("Hello");
        // 发起远程调用
        System.out.println(helloService.sayHello());
    }
}
```
## Dubbo
Dubbo架构
![Uploading 15475433271902]()