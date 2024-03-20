<center>
    <font size="100%"><b>Java-Socket 网络编程</b></font>
</center>

# 一、Socket基础

## 概述

​		计算机网络中，两个进程或两台计算机可以通过一个网络通信连接实现数据交换，这种通信链路的端点就被称为**套接字**（英文名Socket），**Socket**是操作系统提供给应用程序进行网络通信的一个接口或一种机制。		

​		Java的Socket相关API是基于**计算机网络体系结构中传输层的TCP/UDP协议**进行网络通信的功能封装和实现抽象。Socket相关API底层十分复杂，但对于开发，我们只需将数据提交给Socket，Socket会根据我们指定的相关信息，经过一系列数据计算、IP绑定等操作，将数据交给操作系统来进行网络上的收发操作。

## 相关API（java.net.*）

1. java.net包提供了若干基于套接字的客户端/服务器通信的类

2. 常见的类有：

   * URL：统一资源定位符
   * InetAddress：网络上的硬件资源（IP地址）

   * ServerSocket：TCP协议的服务端类，用于监听客户端的请求并创建套接字（Socket）
   * Socket：TCP协议的套接字类
   * DatagramPacket：UDP数据报文容器（存储和解析UDP数据报）
   * DatagramSocket：UDP协议的套接字类

# 二、详细介绍
## URL

​		java.net.URL是Java对于统一资源定位符（*URL*）的封装，提供了一系列方法可以对*URL*进行解析和操作，比如（简单举例几个）：

| 方法                | 说明          |
| ------------------- | --------------- |
| Object getContent() | 获取此URL的内容 |
|String getFile()|获取此URL的文件名|
|String getHost()|获取此URL的主机名（如果适用）|
|InputStream openStream()|打开到此URL的连接并返回一个用于从该连接读入的InputStream|

​		举例：使用java.net.URL发起请求获取一个InputStream来读取网络数据

~~~java
public static void main(String[] args) throws IOException {
        URL url = new URL("http://www.baidu.com");
        /* 向指定URL发起请求，并将响应结果转为InpuStream */
        InputStream is = url.openStream();
        /* 字符流 */
        InputStreamReader isr = new InputStreamReader(is, "utf-8");
        /* 提供缓存功能 */
        BufferedReader br = new BufferedReader(isr);
        String line;
        while ((line = br.readLine()) != null) {
            System.out.println(line);
        }
        br.close();
    }
~~~

## InetAddress

​		java.net.InetAddress用于封装IP地址和DNS，该类没有公共的构造方法，可以使用静态工厂方法来创建InetAddress实例，如（简单举例两个）：

| 方法 | 说明 |
| ---- | ---- |
|getLocalHost()|返回表示本地主机的InetAddress对象      |
|getByName(String hostName)|返回指定主机名为hostName的InetAddress|

## TCP-Socket编程

​		基于TCP协议的**Socket和ServerSocket**满足面向连接，可靠交付的原则，遵循client-server模型进行通信，一般建立通信的主要步骤如下：

1. 服务端实例化ServerSocket对象，表示监听那个端口进行通信；
2. 服务端调用ServerSocket对象的accept()，阻塞等待客户端连接；
3. 客户端实例化Socket对象，指定服务器名称和端口，在构造函数中尝试与服务端建立连接，成功则返回Socket对象；
4. 服务端accept()返回与客户端Socket成功建立连接的Socket对象；
5. 连接成功后客户端和服务端就都可使用Socket对象的输入输出流进行数据交换。

### Socket

1. 构造方法

| 方法 | 说明 |
| ---- | ---- |
|Socket(String hostName,String port)|指定主机名和端口号来创建Socket对象|
|Socket(InetAddress address, int port)|指定InetAddress对象和端口号来创建Socket对象|

2. 常用方法

| 方法                           | 说明                               |
| ------------------------------ | ---------------------------------- |
| InetAddress getInetAddress()   | 返回与Socket对象关联的InetAddress  |
| int getPort()                  | 返回与Socket对象关联的远程端口     |
| int getLocalPort()             | 返回与Socket对象关联的本地端口     |
| InputStream getInputStream()   | 返回与Socket对象关联的InputStream  |
| OutputStream getOutputStream() | 返回与Socket对象关联的OutputStream |
| void close()                   | 关闭Socket                         |

### ServerSocket

1. 构造方法

| 方法                                | 说明                                                         |
   | ----------------------------------- | ------------------------------------------------------------ |
   | ServerSocket(int port)              | 指定端口号创建ServerSocket对象                               |
   | ServerSocket(int port, int backlog) | 指定端口号和最大队列队列长度来创建ServerSocket对象，最大队列长度表示系统在拒绝连接前可以拥有的最大客户端连接数 |

2. 常用方法

| 方法            | 说明                                               |
| --------------- | -------------------------------------------------- |
| Socket accept() | 阻塞等待客户端的连接，返回Socket对象进行进一步通信 |

### 实例演示

Server端：

~~~java
class Server {
    public static void main(String[] args) {
        //使用try-with-resource语法自动关闭ServerSocket资源
        try (ServerSocket serverSocket = new ServerSocket(8080)){
            System.out.println(">>>等待客户端连接<<<");
            //死循环等待客户端的连接
            while (true) {
                //阻塞等待客户端连接
                Socket socket = serverSocket.accept();

                //连接成功获取输入输出流
                System.out.println(">>>连接建立成功");
                InputStream inputStream = socket.getInputStream();
                OutputStream outputStream = socket.getOutputStream();

                //读客户端发送的内容
                System.out.print(">>>获取客户端发送的数据：");
                byte[] in = inputStream.readAllBytes();
                System.out.println(new String(in));

                //给客户端响应内容，注意：输出流用完一定要即刻手动关闭，不然对方无法判断连接是否结束，从而导致双方阻塞
                outputStream.write("连接成功".getBytes());
                outputStream.flush();
                //outputStream.close();
                //也可调用socket.shutdownOutput()来关闭输出流结束本次连接
                socket.shutdownOutput();

                //资源关闭
                inputStream.close();
                socket.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
~~~
> try-with-resource语法：一个类如果实现了AutoCloseable接口，并重写了close()，那这个类的实例创建就可以写在try后面的（）中，实现try-catch语句块执行完后其close()的自动调用
>

Client端：

~~~java
class Client {
    public static void main(String[] args) {
        try {
            //新建Socket对象尝试建立连接
            Socket socket = new Socket("localhost", 8080);

            //连接成功，开始发送数据
            OutputStream outputStream = socket.getOutputStream();
            outputStream.write("Hello World1".getBytes());
            outputStream.flush();
            //输出流用完一定要记得关，不然对方无法判断连接是否结束，导致双方都阻塞
            socket.shutdownOutput();

            //读取服务端的响应数据
            System.out.print(">>>获取服务端响应的数据：");
            InputStream inputStream = socket.getInputStream();
            byte[] bytes = inputStream.readAllBytes();
            System.out.println(new String(bytes));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
~~~

结果：

~~~text
（服务端）
>>>等待客户端连接<<<
>>>连接建立成功
>>>获取客户端发送的数据：Hello World
-----------------------------------------
（客户端）
>>>获取服务端响应的数据：连接成功
~~~

> 此处演示中的服务端是**单线程阻塞**式处理客户端连接，可改为主线程接受连接，然后分发给子线程来进行实际处理的模式（异步），也可改为非阻塞式（NIO的Selector那种），此处就不详细演示了。

## UDP-Socket编程

​		基于UDP协议的**DatagramPacket和DatagramSocket**满足无连接、尽最大努力交付的原则，发送方和接收方是对等的，无主次之分，不会事先建立连接，直接进行数据的发送和接收。常见开发步骤为：

1. 发送方利用DatagramPacket对象封装数据包；
2. 发送方利用DatagramSocket对象发送数据包；
3. 接收方利用DatagramSocket对象接收数据包；
4. 接收方利用DatagramPacket对象处理数据包。

### DatagramPacket

1、构造方法

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| DatagramPacket(byte buf[], int length)                       | 构造DatagramPacket对象，封装长度为length的数据包             |
| DatagramPacket(byte buf[], int length, InetAddress address, int port) | 构造DatagramPacket对象，封装长度为length的数据包并指定要发送的目的主机和端口 |

2、常用方法

| 方法                     | 说明                                   |
| ------------------------ | -------------------------------------- |
| byte[] getData()         | 返回接收到或要发送的数据报中的字节数组 |
| int getLength()          | 返回发送或接收到数据的长度             |
| InetAddress getAddress() | 返回发送或接受此数据报的主机IP         |
| int getPort()            | 返回发送或接受此数据报的主机端口号     |

### DatagramSocket

1、构造方法

| 方法                     | 说明                                                   |
| ------------------------ | ------------------------------------------------------ |
| DatagramSocket()         | 构建DatagramSocket对象，并于当前主机的所有可用端口绑定 |
| DatagramSocket(int port) | 构建DatagramSocket对象，并于当前主机的指定端口绑定     |

2、常用方法

| 方法                                        | 说明                                             |
| ------------------------------------------- | ------------------------------------------------ |
| void connect(InetAddress address, int port) | 将当前DatagramSocket对象连接到远程地址的指定端口 |
| void close()                                | 关闭当前DatagramSocket对象                       |
| void disconnect()                           | 断开当前DatagramSocket对象                       |
| int getLocalPort()                          | 返回当前DatagramSocket对象绑定的本地主机的端口号 |
| void send(DatagramPacket p)                 | 发送指定的数据报                                 |
| void recevie(DatagramPacket p)              | 接收数据报，并存于指定的DatagramPacket对象中     |

### 实例演示

接收方：

~~~java
class Receive {
    public static void main(String[] args) {
        //新建DatagramSocket
        try (DatagramSocket datagramSocket = new DatagramSocket(8080)) {
            //接收数据
            DatagramPacket requestPacket = new DatagramPacket(new byte[1024], 1024);
            datagramSocket.receive(requestPacket);
            System.out.printf(">>>对方发送：");
            System.out.println(new String(requestPacket.getData()));
            //响应数据
            byte[] res = "数据接收成功".getBytes();
            DatagramPacket responsePacket = 
                new DatagramPacket(res, res.length, requestPacket.getAddress()，requestPacket.getPort());
            datagramSocket.send(responsePacket);
        } catch (SocketException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
~~~

发送方：

~~~java
class Send{
    public static void main(String[] args) {
        //新建DatagramSocket
        try (DatagramSocket datagramSocket = new DatagramSocket(8081)) {
            //发送数据
            byte[] request = "Hello World".getBytes();
            DatagramPacket requestPacket = new DatagramPacket(request,request.length,InetAddress.getByName("localhost"),8080);
            datagramSocket.send(requestPacket);
            //接收响应
            System.out.print(">>>对方响应：");
            DatagramPacket responsePacket = new DatagramPacket(new byte[1024], 1024);
            datagramSocket.receive(responsePacket);
            System.out.println(new String(responsePacket.getData()));
        } catch (SocketException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
~~~

结果：

~~~text
（接收方）
>>>对方发送：Hello World            
----------------------------------------
（发送方）
>>>对方响应：数据接收成功                                                                                                           
~~~

