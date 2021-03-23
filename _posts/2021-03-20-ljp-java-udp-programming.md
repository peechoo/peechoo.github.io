---
title: "Java 网络编程之UDP通信"
date: 2021-03-20
author: pp
---

#### IP地址

使用类InetAddress来表示Internet协议（IP）地址

- 主要方法

| 方法名                                    | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| static InetAddress getByName(String host) | 确定主机名称的IP地址。主机名称可以是机器名称，也可以是IP地址 |
| String getHostName()                      | 获取此IP地址的主机名                                         |
| String getHostAddress()                   | 返回文本显示中的IP地址字符串                                 |

- 代码演示

```java
public class InetAddressDemo { 
    public static void main(String[] args) throws UnknownHostException { 
        //InetAddress address = InetAddress.getByName("itheima"); 
        InetAddress address = InetAddress.getByName("192.168.1.66"); 
        //public String getHostName()：获取此IP地址的主机名 
        String name = address.getHostName(); 
        //public String getHostAddress()：返回文本显示中的IP地址字符串 
        String ip = address.getHostAddress(); 
        System.out.println("主机名：" + name); 
        System.out.println("IP地址：" + ip); 
    } 
}
```

#### UDP发送数据

Java提供了DatagramSocket类作为基于UDP协议的Socket 

- 构造方法 

| 方法名                                        | 说明                                                 |
| --------------------------------------------- | ---------------------------------------------------- |
| DatagramSocket()                              | 创建数据报套接字并将其绑定到本机地址上的任何可用端口 |
| DatagramPacket(byte[] buf,int len,InetAddress | 创建数据包,发送长度为len的数据包到指定主机的指定端口 |

- 相关方法

| 方法名                         | 说明                   |
| ------------------------------ | ---------------------- |
| void send(DatagramPacket p)    | 发送数据报包           |
| void close()                   | 关闭数据报套接字       |
| void receive(DatagramPacket p) | 从此套接字接受数据报包 |

- 发送数据的步骤 
  - 创建发送端的Socket对象(DatagramSocket)创建数据，并把数据打包 
  - 调用DatagramSocket对象的方法发送数据 
  - 关闭发送端
- 代码演示

```java
public class SendDemo { 
    public static void main(String[] args) throws IOException {
    //创建发送端的Socket对象(DatagramSocket)
    // DatagramSocket() 构造数据报套接字并将其绑定到本地主机上的任何可用端口 
    DatagramSocket ds = new DatagramSocket(); //创建数据，并把数据打包 //DatagramPacket(byte[] buf, int length, InetAddress address, int port) 
    //构造一个数据包，发送长度为 length的数据包到指定主机上的指定端口号。 
    byte[] bys = "hello,udp,我来了".getBytes();
    DatagramPacket dp = new DatagramPacket(bys,bys.length,InetAddress.getByName("127.0.0.1"),10086); 
    //调用DatagramSocket对象的方法发送数据 
    //void send(DatagramPacket p) 从此套接字发送数据报包 
    ds.send(dp); 
    //关闭发送端 
    //void close() 关闭此数据报套接字 
    ds.close(); 
	} 
}
```

#### UDP接收数据

- 构造方法

| 方法名                              | 说明                                            |
| ----------------------------------- | ----------------------------------------------- |
| DatagramPacket(byte[] buf, int len) | 创建一个DatagramPacket用于接收长度为len的数据包 |

- 相关方法

| 方法名           | 说明                                     |
| ---------------- | ---------------------------------------- |
| byte[] getData() | 返回数据缓冲区                           |
| int getLength()  | 返回要发送的数据的长度或接收的数据的长度 |

- 演示代码

```java
public class ReceiveDemo { 
    public static void main(String[] args) throws IOException { 
        //创建接收端的Socket对象(DatagramSocket) 
        DatagramSocket ds = new DatagramSocket(12345); 
        
        //创建一个数据包，用于接收数据 
        byte[] bys = new byte[1024]; 
        DatagramPacket dp = new DatagramPacket(bys, bys.length); 
        //调用DatagramSocket对象的方法接收数据 
        ds.receive(dp);
        //解析数据包，并把数据在控制台显示 
        System.out.println("数据是：" + new String(dp.getData(), 0, dp.getLength())); 
	} 
}
```

