 # 前言

在很早之前学习序列化的时候有写过一篇关于Java序列化的博客，不过那只是简单的使用，入门者欢迎移步：[http://blog.beifengtz.com/article/36](http://blog.beifengtz.com/article/36)。上周在工作时遇到了一个序列化的问题，就是父子类序列化对其值的保存问题，关于序列化有很多细节知识，这篇文章就仔细学习一下Java中的序列化吧。

# 一、为什么要序列化

现在企业中的系统大多都不是单语言编写的，一个平台可能有Java、Python、Cpp、Lua等语言编写而成，如果在其内部或者这个平台与其他平台进行数据交互时，必须要有统一的数据格式，各个语言的数据必须经过序列化成这些统一格式后才能被其他系统所识别，一般序列化的结果是一个二进制数据，当接收方系统收到这个二进制数据后必须经过反序列化转换成自己能识别的数据。当然除了网络传输外，序列化也是一种持久化的手段，你可以序列化成一个二进制文件来进行数据存储。多语言支持的序列化格式常见的有XML、JSON、ProtoBuf等。

Java语言中也有自己支持的序列化方式，一般使用序列化都是在对象持久化中，网络传输更多的是使用上面所说的那三种常见的序列化格式。

# 二、先看一个Demo

序列化的对象：
```java
/**
 * @author beifengtz
 * <a href='http://www.beifengtz.com'>www.beifengtz.com</a>
 * <p>location: serializ.javase_learning</p>
 * Created in 10:41 2019/10/13
 */
public class User implements Serializable {

    private static final long serialVersionUID = -6849794470754667710L;

    private String name;
    private transient String gender;
    private int age;
    private long regTime;

    public User(String name, String gender, int age, long regTime) {
        this.name = name;
        this.gender = gender;
        this.age = age;
        this.regTime = regTime;
    }

    public User(String name, int age, long regTime) {
        this.name = name;
        this.age = age;
        this.regTime = regTime;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", gender='" + gender + '\'' +
                ", age=" + age +
                ", regTime=" + regTime +
                '}';
    }
}
```

序列化与反序列化：
```java
/**
 * @author beifengtz
 * <a href='http://www.beifengtz.com'>www.beifengtz.com</a>
 * <p>location: serialization.javase_learning</p>
 * Created in 10:46 2019/10/13
 */
public class SerializationDemo {
    public static void main(String[] args) {
        User user = new User("beifengtz", "男", 100, System.currentTimeMillis() / 1000);

        System.out.println("序列化之前：");
        System.out.println(user);

        //  将对象序列化进文件
        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("serFile"));
            oos.writeObject(user);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                assert oos != null;
                oos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        //  从文件中反序列化成对象
        File file = new File("serFile");
        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream(file));
            User serUser = (User) ois.readObject();
            System.out.println("序列化之后：");
            System.out.println(serUser);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            try {
                assert ois != null;
                ois.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

最后控制台输出的结果：
```
序列化之前：
User{name='beifengtz', gender='男', age=100, regTime=1570935427}
序列化之后：
User{name='beifengtz', gender='null', age=100, regTime=1570935427}
```

然后我们再看中间生成的序列化文件`serFile`，直接打开之后是乱码，用二进制编辑器打开里面全是16进制字符。

# 三、序列化的ID

序列化方和反序列化方要想成功进行一次数据传输，必须要保证三个条件：
1. 类全路径必须一样，比如都是`com.beifengtz.User`
2. 类功能代码必须一样
3. 序列化ID必须一样，比如都是`private static final long serialVersionUID = 1L`

**以上三个条件缺少一个均无法成功序列化**，其中如果没有定义序列化ID虚拟机会随机生成一个ID，但是这样对于程序的可控性并不高，毕竟序列化是服务于多端的。下面举一个IBM Developer中一篇文章举的例子，在实际应用中的使用案例：

> Facade模式中，Facade Object是为应用程序提供统一的访问接口，案例程序中的 Client 客户端使用了该模式，案例程序结构图如图所示：
>
>![https://www.ibm.com/developerworks/cn/java/j-lo-serial/image003.gif](https://www.ibm.com/developerworks/cn/java/j-lo-serial/image003.gif)
>
> Client 端通过 Façade Object 才可以与业务逻辑对象进行交互。而客户端的 Façade Object 不能直接由 Client 生成，而是需要 Server 端生成，然后序列化后通过网络将二进制对象数据传给 Client，Client 负责反序列化得到 Façade 对象。该模式可以使得 Client 端程序的使用需要服务器端的许可，同时 Client 端和服务器端的 Façade Object 类需要保持一致。当服务器端想要进行版本更新时，只要将服务器端的 Façade Object 类的序列化 ID 再次生成，当 Client 端反序列化 Façade Object 就会失败，也就是强制 Client 端从服务器端获取最新程序。

# 四、父子类序列化

1. 序列化时，只对对象的状态进行保存，而不管对象的方法；
2. 父类实现Serializable，子类自动实现序列化，当序列化子类时，父类的属性值也会被保存，因此子类无需显示实现Serializable；
3. 父类未实现Serializable，子类实现Serializable，当序列化子类时，父类的属性值**不会被保存**，并且父类必须有无参构造（因为反序列化时不存在父类属性值，实例化对象时只有子类属性值）。

这里就不写实际的例子了，有兴趣可以自行去验证。

# 五、自定义序列化

在序列化过程中，虚拟机会试图调用对象类里的 writeObject 和 readObject 方法，进行用户自定义的序列化和反序列化，如果没有这样的方法，则默认调用是 ObjectOutputStream 的 defaultWriteObject 方法以及 ObjectInputStream 的 defaultReadObject 方法。用户自定义的 writeObject 和 readObject 方法可以允许用户控制序列化的过程，比如可以在序列化的过程中动态改变序列化的数值。

可以看看下面这个样例：

```java
public class Test implements Serializable {
    private static final long serialVersionUID = 1L;

    private String account = "beifengtz";
    private String password = "123456";

    public String getAccount() {
        return account;
    }

    public String getPassword() {
        return password;
    }

    private void readObject(ObjectInputStream in) {
        try {
            System.out.println("------开始反序列化------");
            ObjectInputStream.GetField readField = in.readFields();
            Object obj = readField.get("password", "");
            System.out.println("要解密的密码:" + obj);
            password = String.valueOf(Integer.parseInt((String) obj) / 1000);
            System.out.println("解密后的密码:" + password);
            System.out.println("------反序列化结束------");
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    private void writeObject(ObjectOutputStream out) {
        try {
            System.out.println("------开始序列化------");
            ObjectOutputStream.PutField putField = out.putFields();
            String encrypt = String.valueOf(Integer.parseInt(password) * 1000);
            putField.put("password", encrypt);
            putField.put("account", account);
            System.out.println("真实密码：" + password);
            System.out.println("加密后的密码：" + encrypt);
            out.writeFields();
            System.out.println("------序列化结束-----");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        try {

            ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("serFile"));
            out.writeObject(new Test());
            out.close();

            ObjectInputStream in = new ObjectInputStream(new FileInputStream("serFile"));
            Test test = (Test) in.readObject();
            in.close();
            System.out.println("反序列化后接收到的密码：" + test.getPassword());
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

看结果
```
------开始序列化------
真实密码：123456
加密后的密码：123456000
------序列化结束-----
------开始反序列化------
要解密的密码:123456000
解密后的密码:123456
------反序列化结束------
反序列化后接收到的密码：123456
```

# 六、多对象序列化的存储

对于JDK的序列化并不是简单的二进制文本追加存储，而是有一些优化的。其多对象序列化存储方式如下：
1. 如果多次存储的对象是不同类的对象，序列化后的二进制内容直接追加在文本中；
2. 如果多次存储的对象是同一个类的同一个对象，并且其属性完全相同，在第一次写入二进制之后，后面的序列化内容仅仅保存引用和控制信息，其余相同信息复用，并非文本追加；
3. 如果多次存储的对象是同一个类的同一个对象，但是在多次写入期间有改动其对象内容，虚拟机根据引用关系知道已经有一个相同对象已经写入文件，仅保存第一次写入的对象，第一次序列化之后的对象修改无法被保存；
4. 如果多次存储的对象是同一个类的不同对象，在序列化时也会复用类信息，仅保存这不同对象的不同属性的引用和控制信息，相同属性复用。

看一下下面四种不同场景，你是否有遇到过？

以下四种场景都基于下面两个类来进行序列化测试：

**Test1 类**
```java
public class Test1 implements Serializable {
    private Integer value;

    public Test1(Integer value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "Test1{" +
                "value=" + value +
                '}';
    }
}
```

**Test2 类**
```java
public class Test2 implements Serializable {
    private String content;

    public Test2(String content) {
        this.content = content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    @Override
    public String toString() {
        return "Test2{" +
                "content='" + content + '\'' +
                '}';
    }
}
```

## 6.1 多次写入同一个类的同一个对象
样例：
```java
public static void main(String[] args) {
    try {
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("serFile"));
        Test2 test = new Test2("1");
        out.writeObject(test);
        out.flush();
        System.out.println("二进制文件长度：" + new File("serFile").length());
        out.writeObject(test);
        out.flush();
        out.close();
        System.out.println("二进制文件长度：" + new File("serFile").length());
    } catch (IOException e) {
        e.printStackTrace();
    }


    try {
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("serFile"));
        Test2 test_1 = (Test2) in.readObject();
        Test2 test_2 = (Test2) in.readObject();

        System.out.println(test_1);
        System.out.println(test_2);
        System.out.println(test_1 == test_2);
    } catch (IOException | ClassNotFoundException e) {
        e.printStackTrace();
    }
}
```

执行结果：

```
二进制文件长度：75
二进制文件长度：80
Test2{content='1'}
Test2{content='1'}
true
```
## 6.2 多次写入同一个类的同一个对象（先后修改属性）

样例：
```java
public static void main(String[] args) {
    try {
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("serFile"));
        Test2 test = new Test2("1");
        out.writeObject(test);
        out.flush();
        System.out.println("二进制文件长度：" + new File("serFile").length());
        test.setContent("2");   //  修改test属性内容
        out.writeObject(test);
        out.flush();
        out.close();
        System.out.println("二进制文件长度：" + new File("serFile").length());
    } catch (IOException e) {
        e.printStackTrace();
    }


    try {
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("serFile"));
        Test2 test_1 = (Test2) in.readObject();
        Test2 test_2 = (Test2) in.readObject();

        System.out.println(test_1);
        System.out.println(test_2);
        System.out.println(test_1 == test_2);
    } catch (IOException | ClassNotFoundException e) {
        e.printStackTrace();
    }
}
```

执行结果：

```
二进制文件长度：75
二进制文件长度：80
Test2{content='1'}
Test2{content='1'}
true
```
## 6.3 多次写入同一个类的不同对象
样例：
```java
public static void main(String[] args) {
    try {
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("serFile"));
        out.writeObject(new Test2("1"));
        out.flush();
        System.out.println("二进制文件长度：" + new File("serFile").length());
        out.writeObject(new Test2("2"));
        out.flush();
        out.close();
        System.out.println("二进制文件长度：" + new File("serFile").length());
    } catch (IOException e) {
        e.printStackTrace();
    }


    try {
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("serFile"));
        Test2 test_1 = (Test2) in.readObject();
        Test2 test_2 = (Test2) in.readObject();

        System.out.println(test_1);
        System.out.println(test_2);
        System.out.println(test_1 == test_2);
    } catch (IOException | ClassNotFoundException e) {
        e.printStackTrace();
    }
}
```

执行结果：

```
二进制文件长度：75
二进制文件长度：85
Test2{content='1'}
Test2{content='2'}
false
```
## 6.4 多次写入不同类的对象

样例：
```java
public static void main(String[] args) {
    try {
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("serFile"));
        out.writeObject(new Test1(1));
        out.flush();
        System.out.println("二进制文件长度：" + new File("serFile").length());
        out.writeObject(new Test2("2"));
        out.flush();
        out.close();
        System.out.println("二进制文件长度：" + new File("serFile").length());
    } catch (IOException e) {
        e.printStackTrace();
    }


    try {
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("serFile"));
        Test1 test_1 = (Test1) in.readObject();
        Test2 test_2 = (Test2) in.readObject();

        System.out.println(test_1);
        System.out.println(test_2);
    } catch (IOException | ClassNotFoundException e) {
        e.printStackTrace();
    }
}
```

执行结果：
```
二进制文件长度：147
二进制文件长度：218
Test1{value=1}
Test2{content='2'}
```
# Java序列化知识总结

1. **需要被序列化的对象必须实现`Serializable`接口，这是一个声明式接口，无任何属性和方法，如果不实现该接口会报错：`NotSerializableException`**
2. **通过ObjectOutputStream和ObjectInputStream对对象进行序列化及反序列化**
3. **被transient关键字修饰的属性值不会被保存进序列化文件，故反序列化后的属性值是变量类型的默认值。比如这里String的gender就是`null`**
4. **序列化不保存静态变量**
5. **虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化 ID 是否一致（private static final long serialVersionUID）**
6. **对于多对象序列化的存储，并非简单的二进制内容追加，虚拟机对其有一定的优化，可减少磁盘空间占用或网络传输内容大小**
