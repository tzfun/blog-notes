上一篇博客写了字节流和字符流的基本知识，这一篇将对上一篇的内容进行一个简单的应用。这学期在学嵌入式的时候用c语言写了一个文件操作的小程序，但是从基本实现上c语言和java是有所不同的。今天就用java实现一下dos命令中的文件复制命令。

> copy \[源文件路径\] \[目的文件路径\]
> * copy命令的参数需为两个
> * 源文件路径必须存在
> * 如果目的文件目录不存在，先创建目录再复制文件

为了方便测试，我在D盘下放了一个11.1MB的图片test.jpg，先用cmd测试一下，输入命令*copy d:/test.jpg d:test-copy.jpg*之后在不到**一秒内**就实现了文件复制

思考：

如何实现文件复制命令？？？

对于命令的逻辑判断比较常规，最核心的问题就是如何实现文件复制，主要有两种方式：
* 方式一：将文件全部读取到内存，然后再输出到目的目录；
* 方式二：边读取边输出。

常规选择的方式肯定是方式二，因为方式一不够安全，如果我们需要复制的文件非常大，比如300GB，如果全部读取到内存程序直接会崩掉，因为没有那么大的内存去存如此大的字节数据，所以不可取。同时读取的方式应该使用字节流来进行，至于为什么使用字节流上篇博客也有所说明，这里就不多赘述，有疑问参见上篇博文。

其实实现文件复制的过程我们可以这样模拟一下，文件复制如同是将一百吨的沙搬运到另一个地方，搬运的工具就是我们的程序，要找到一个一次性能装载一百吨沙的货车几乎是没有的，而且就算找到也很耗费资源。接下来用程序实现一下copy的过程：

> 在测试时需要对主函数参数进行设置，eclipse和idea都在运行处设置，此处我设置的是 d:/test.jpg d:/test-copy.jpg 如果是在cmd中用javac命令执行的话在后面跟上参数即可

一次单个字节读取和存储：
```
package language;

import java.io.*;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 20:14 2019/1/24
 * @Description:
 */
public class CopyDemo {
    public static void main(String[] args) throws Exception{

        long start = System.currentTimeMillis();    // 开始时时间戳

        if (args.length!= 2){   //  判断传入参数
            System.out.println("命令参数错误！");
            System.exit(1); //  终止程序运行
        }

        File fileA = new File(args[0]);     // 源文件
        File fileB = new File(args[1]);     // 目的文件

        if (!fileA.exists()){               // 判断源文件是否存在
            System.out.println("源文件不存在！");
            System.exit(1);
        }

        if ( !fileB.getParentFile().exists()){  // 如果目的文件的目录不存在
            fileB.getParentFile().mkdirs();     // 创建目的文件目录
        }

        InputStream inputStream = new FileInputStream(fileA);
        OutputStream outputStream = new FileOutputStream(fileB);
        System.out.println("复制文件开始......");
        int temp = 0;
        while ((temp = inputStream.read()) != -1){
            outputStream.write(temp);
        }

        inputStream.close();
        outputStream.close();
        System.out.println("复制文件结束......");
        long end = System.currentTimeMillis(); // 结束时时间戳

        System.out.println("复制总耗时：" + (end - start));
    }
}
```
运行结果如下：
```
复制文件开始......
复制文件结束......
复制总耗时：46230
```
我们发现这个复制过程耗费了46秒，仅仅11MB的文件复制过程就耗费了46秒，这样的程序性能肯定是不佳的。

分析一下原因：
> 上面程序的过程相当于在搬运沙子的过程时一粒一粒的搬运，而上面程序的io又是阻塞式io，相当于全程只有一个人在搬运，而且搬运的量是一粒沙，很明显这是非常慢速的。

下面对上面程序进行一下优化，使用另一种输入输出方式：
```
package language;

import java.io.*;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 20:14 2019/1/24
 * @Description:
 */
public class CopyDemo {
    public static void main(String[] args) throws Exception{

        long start = System.currentTimeMillis();    // 开始时时间戳

        if (args.length!= 2){   //  判断传入参数
            System.out.println("命令参数错误！");
            System.exit(1); //  终止程序运行
        }

        File fileA = new File(args[0]);     // 源文件
        File fileB = new File(args[1]);     // 目的文件

        if (!fileA.exists()){               // 判断源文件是否存在
            System.out.println("源文件不存在！");
            System.exit(1);
        }

        if ( !fileB.getParentFile().exists()){  // 如果目的文件的目录不存在
            fileB.getParentFile().mkdirs();     // 创建目的文件目录
        }

        InputStream inputStream = new FileInputStream(fileA);
        OutputStream outputStream = new FileOutputStream(fileB);
        System.out.println("复制文件开始......");
        int temp = 0;
        byte[] tempData = new byte[1024];   // 开辟一个字节数组
        while ((temp = inputStream.read(tempData)) != -1){  // temp为一次读取字节长度
            outputStream.write(tempData,0,temp);
        }

        inputStream.close();
        outputStream.close();
        System.out.println("复制文件结束......");
        long end = System.currentTimeMillis(); // 结束时时间戳

        System.out.println("复制总耗时：" + (end - start));
    }
}
```
运行结果如下：
```
复制文件开始......
复制文件结束......
复制总耗时：74
```
可以看到现在运行的时间是0.07秒，速度提升了600多倍，这样的程序速度优化了很多。

分析一下为什么会提升：
> 上面程序将一次一个字节的传输改为了1024个字节的传输，如同在运输沙子的时候由一粒沙改为了一吨载重量的货车，效率上有显著的提升。

那么是不是开辟的数组越大越忧？
> 答案是：NO！！开辟数组的大小取决于你的资源条件有多充足，假设你是搬运沙子这个项目的老板，去购买多大载重量的货车取决于你的资金有多少。程序也是一样，一次开辟多大的数组取决于服务器的性能及内存有多强，因为数组开辟之后会占用内存资源。而且一般如果程序是服务类的程序还得考虑另一个问题，就是使用你程序的用户不止一个，如果多个用户同时使用，又如何分配你的内存资源，考虑因素增加了许多。

java.io包中的输入输出是阻塞式的io，如果做服务器需要运用到另一种io方式就是NIO，NIO的内容后面再学习吧~~