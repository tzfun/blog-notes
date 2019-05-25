今天的内容是Java内存流，同样是基于字节流和字符流操作。

java.io包中提供两种内存流的操作：
* 字节内存流：ByteArrayInputStream、ByteArrayOutputStream
* 字符内存流：CharArrayReader、CharArrayWriter

因为字节内存流和字符内存流的操作方法相似，所以这里以字节内存流为例，先来看一下其继承结构和构造方法：

ByteArrayInputStream：
> 继承结构：
> ```
> java.lang.Object
>    java.io.InputStream
>        java.io.ByteArrayInputStream
> ```
> 构造方法：
> ```
> public ByteArrayInputStream(byte[] buf)   //  将要操作的数据设置到输入流
> public ByteArrayInputStream(byte[] buf, int offset, int length)   //  将要操作的部分数据设置到输入流
> ```

ByteArrayOutputStream：
> 继承结构：
> ```
> java.lang.Object
>   java.io.OutputStream
>       java.io.ByteArrayOutputStream
> ```
> 构造方法：
> ```
> public ByteArrayOutputStream()    //  从内存输出数据
> public ByteArrayOutputStream(int size)    //  从内存输出部分数据，size为初始化大小
> ```

从其中的继承结构看发现它们都是InputStream和OutputStream的直接子类。但是我们发现其构造方法和InputStream和OutputStream有所不一样，特别是参数上，下面我简单的画个图来理解一下其工作流程：

文件操纵：
![1.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E6%B5%81/20190125200158.png)

内存操作：
![2.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E6%B5%81/20190125200207.png)

它们之间的区别就是使用的流不同，一个是文件输入输出流一个是字节内存流；然后就是操作终端不同，一个是文件一个是内存。

ByteArrayOutputStream类中重要方法：
* public int size() //  返回当前内存使用大小
* <font color=red>**public byte[] toByteArray()**</font>   //  将内存中的所有数据转换为字节数组（重要）
* public String toString()  //  使用平台默认的字符集，通过解码字节将缓冲区内容转换为字符串。
* <font color=red>**public void writeTo(OutputStream out) throws IOException**  //  将内存中所有数据输入到另一个输出流，可以是文件流也可以是网络流等。（重要）</font>

根据上面的方法我用一个简单的例子来呈现，实现文件合并并由内存流转换为文件流输出：
> 先在D盘创建两个文件，testA.txt和testB.txt，其内容如下：
> * testA.txt
> ```
> 大家好！我是beifengtz，欢迎来到北风个人博客！
> ```
> * testB.txt
> ```
> Hello everyone! I'm beifengtz. Welcome to Beifeng Personal Blog!
> ```

```
package language;

import java.io.*;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 19:24 2019/1/25
 * @Description:
 */
public class MemoryStream {
    public static void main(String[] args) throws Exception{
        File fileA = new File("D:"+File.separator+"testA.txt");
        File fileB = new File("D:"+File.separator+"testB.txt");
        File fileC = new File("D:"+File.separator+"testC.txt");     // 将合并输出的文件
        InputStream inputStreamA = new FileInputStream(fileA);  // 创建testA的文件输入流
        InputStream inputStreamB = new FileInputStream(fileB);  //  创建testB的文件输入流
        OutputStream outputStream = new ByteArrayOutputStream();    //  创建内存输入流（针对于内存是输入，针对于程序是输出）
        int temp = 0;
        while ((temp = inputStreamA.read()) != -1){
            outputStream.write(temp);   //  将A文件的数据写入内存
        }
        while ((temp = inputStreamB.read()) != -1){
            outputStream.write(temp);   //  将B文件的数据写入内存
        }
        System.out.println("内存中的数据：" + outputStream);   //  调用toString方法将内存中所有的数据转化成字符串输出
        byte data[] = ((ByteArrayOutputStream) outputStream).toByteArray();     // 调用toByteArray方法将内存中所有数据转化为字节数组
        System.out.println("手动编码后的数据：" + new String(data,"GBK"));   //  将内存中的数据手动编码输出
        System.out.println("内存占用大小："+((ByteArrayOutputStream) outputStream).size());  //  获取内存占用大小
        System.out.println("转换后字节数组的长度："+data.length);
        if (!fileC.getParentFile().exists()){   // 如果新输出的文件目录不存在
            fileC.getParentFile().mkdirs();     // 创建新文件路径
        }
        //  将内存中的数据全部以文件输出流的形式写入，相当于调用FileOutputStream的write方法
        ((ByteArrayOutputStream) outputStream).writeTo(new FileOutputStream(fileC));
    }
}
```
输出结果：
```
内存中的数据：��Һã�����beifengtz����ӭ����������˲��ͣ�
Hello everyone! I'm beifengtz. Welcome to Beifeng Personal Blog!
手动编码后的数据：大家好！我是beifengtz，欢迎来到北风个人博客！
Hello everyone! I'm beifengtz. Welcome to Beifeng Personal Blog!
内存占用大小：111
转换后字节数组的长度：111
```
最后打开testC.txt文件之后发现两个文件的内容也已经合并在一起了：
```
大家好！我是beifengtz，欢迎来到北风个人博客！
Hello everyone! I'm beifengtz. Welcome to Beifeng Personal Blog!
```

在编写上面程序过程中遇到了几个问题需要注意一下：
* ByteArrayOutputStream中toString()方法是以默认编码进行字符串的转换，所以输出结果中会有乱码。之所以文件里没有乱码，是因为在输出的时候都是以字节进行输出，不存在编码问题。在平时开发中如果要讲字节转化为字符一定要注意编码的问题！！！
* 输出流转换用处非常多。内存中的计算是非常快的，所以平时一般使用内存中的io读取更高效，但是如果要将内存流转换为文件流或网络流，writeTo方法是最好的选择。
* 读入内存的数据不宜太大，否则会造成程序崩溃，在开发时要对写入大小进行限制，一般会用到size()方法。