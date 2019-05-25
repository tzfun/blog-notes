今天的内容是Java基本的网络编程，实现一个基本的Server、client程序（嵌入式的网络编程Java版）。
# 两种网络编程模式
* C/S模型（Client/Server）：又称客户机/服务器模式，此类模式开发需要两套程序，一套是客户端代码，一套是服务器端代码，程序开发及维护成本较高，但是安全性相对较高。
* B/S模型（Browser/Server）：浏览器/服务器模式，不需要单独开发客户端代码，只需要开发一套服务器端代码即可，程序开发及维护相对方便，但是安全性较低，大多数使用的是80端口或443端口。

相对于这两种模型，现在本人更多使用的是B/S模型，也就是平常说的网站，同时更偏向与后端。对于安全性其实现在都相对差不多，只要能抓到网络传输过程的数据包，对于服务器的ip、端口这些都将暴露无遗。而对于开发成本我个人觉得都差不多，因为它要开发各种操作系统的程序，还需要兼容不同屏幕大小的设备，但是C/S模型其实也差不多，它需要兼容不同类型、不同版本的浏览器，以及不同屏幕大小的设备，所以两者其实也差不多。这两种模式的优与劣不是今天讨论的重点，不同的场景需求有不同的优势与缺点，今天的讨论重点是如何实现其中的数据传输。
# 类库介绍
对于C/S模型有两种网络形式的连接方式：
* 基于TCP：采用可靠的方式进行连接，面向连接。
* 基于UDP：不可靠的连接，数据报协议。
 
这里以TCP连接作为介绍，在Java中要实现网络编程需要两种类的支持：
* 服务器端：java.net.ServerSocket，工作在服务器端，用于接收客户端请求；
* 客户端：java.net.Socket，工作在客户端，用于发起请求。

# ServerSocket
继承结构：
```
java.lang.Object
    java.net.ServerSocket
```
类定义：
```
public class ServerSocket
extends Object
implements Closeable
```
构造函数：
```
public ServerSocket() throws IOException
public ServerSocket(int port) throws IOException
public ServerSocket(int port, int backlog) throws IOException
public ServerSocket(int port, int backlog, InetAddress bindAddr) throws IOException
```
# Socket
继承结构：
```
java.lang.Object
    java.net.Socket
```
类定义：
```
public class Socket
extends Object
implements Closeable
```
构造函数：
```
public Socket()
public Socket(InetAddress address, int port) throws IOException
```

# 基于TCP连接的编程实现
服务端（支持多个客户端访问，每访问一个创建一个线程去与之回话）：
```
package language;

import java.io.PrintStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.Scanner;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 21:21 2019/1/29
 * @Description:
 */
public class NetServerTest {
    static class ServerThread implements Runnable{
        Socket client;
        public ServerThread (Socket client){
            this.client = client;
        }
        @Override
        public void run() {
            try{
                System.out.println(client.getInetAddress()+"已成功连接");
                PrintStream printStream = new PrintStream(client.getOutputStream());    //  获取客户端的输出流（客户端向服务器端发送的流）
                Scanner scanner = new Scanner(client.getInputStream());     //  获取客户端的输入流（服务器端向客户端发送的流）
                boolean flag = true;    //循环的标志
                while(flag){
                    if (scanner.hasNext()){
                        String str = scanner.next();
                        if (str.equalsIgnoreCase("byebye")){ //  如果客户端发来“byebye”则终止会话
                            flag = false;
                            printStream.println("再见！本次回话结束");
                        }else{  //  否则就将客户端的信息进行返回
                            printStream.println("Hello！我是服务器。客户端的输入是："+str);
                        }
                    }
                }
                scanner.close();
                client.close();
                printStream.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
    public static void main(String[] args) throws Exception{
        ServerSocket serverSocket = new ServerSocket(9000);     //  绑定并监听9000接口
        System.out.println("等待客户请求中......");
        boolean flag = true;
        while (flag){
            Socket client = serverSocket.accept();      //  接收客户端连接
            new Thread(new ServerThread(client)).start();   //  创建新的线程并进行数据处理
        }
        serverSocket.close();
    }
}
```
客户端（与服务器创建连接并进行回话，输入byebye结束回话）：
```
package language;

import java.io.PrintStream;
import java.net.Socket;
import java.util.Scanner;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 21:25 2019/1/29
 * @Description:
 */
public class NetClientTest {
    public static void main(String[] args) throws Exception{
        Socket client = new Socket("localhost",9000);   //  设置连接的地址和端口
        Scanner scanner = new Scanner(client.getInputStream());     //  设置客户端的输入流（从服务器发送到客户端的流）
        PrintStream printStream = new PrintStream(client.getOutputStream());//  注册客户端传输信息的输出流（从客户端发送到服务器的流）
        Scanner input = new Scanner(System.in);                     //  用户键盘输入打印流注册
        scanner.useDelimiter("\n"); //  以换行为分隔符
        input.useDelimiter("\n"); //  以换行为分隔符
        boolean flag = true;        //  循环的标识
        while (flag){
            System.out.print("请输入内容（输入byebye退出）：");
            String str = input.next();
            if (str.equalsIgnoreCase("byebye")){
                flag = false;
            }
            printStream.println(str);   // 客户端向服务器端发送数据
            System.out.println("发送数据成功！！");
            if (scanner.hasNext()){ //  如果服务器端有回应结果
                System.out.println(scanner.next());
            }
        }
        client.close();
        scanner.close();
        input.close();
    }
}
```
服务器端输出：
```
等待客户请求中......
/127.0.0.1已成功连接
/127.0.0.1已成功连接
```
客户端1输出：
```
请输入内容（输入byebye退出）：哈哈
发送数据成功！！
Hello！我是服务器。客户端的输入是：哈哈
请输入内容（输入byebye退出）：
```
客户端2输出：
```
请输入内容（输入byebye退出）：beifengtz
发送数据成功！！
Hello！我是服务器。客户端的输入是：beifengtz
请输入内容（输入byebye退出）：hahhahaha
发送数据成功！！
Hello！我是服务器。客户端的输入是：hahhahaha
请输入内容（输入byebye退出）：byebye
发送数据成功！！
再见！本次回话结束
```

以上程序简单的实现了客户端和服务器端的数据传输，服务器可以同时接收多个客户端的连接并处理其会话，客户端只能与一个固定地址的固定端口的某个服务器连接并发起会话，每发起一个会话请求收到一个回复，输入byebye则会话结束。

现在的服务器都不是上面这种形式去处理了，因为不会有哪个服务器有那么多资源去开启多个线程去处理会话，因为多个线程同时工作，在某些时刻总有大部分的线程是闲置的，并没有工作，这样就造成了大量的资源浪费，所以后来Tomcat等服务器都将这种服务器模式改成了新的io模式即NIO，这是一种单线程运作的高可用的io模式，一个线程轮流地为多个客户端服务，将资源利用得到了最大化，具体NIO的学习后面再看吧，今天就到这里啦~~