# 问题引入
首先我们先回顾一下一些数据类型的比较，Number类型的数据比较是最常见的，比如2>1,20=20等等，但是之前常用的类String和Date类都可以进行比较，他们比较的方法都有一个compareTo()方法来比较，大于返回1、等于返回0、小于返回-1。我们以String为例
```
package language;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 15:55 2019/1/19
 * @Description:
 */
public class ComparatorTest{
    public static void main(String[] args) {
        String strListA = "hello";
        String strListB = "hello1";
        System.out.println(strListA.compareTo(strListB));
    }
}
```
最后输出结果是-1，因为“hello”字符串比“hello1”字符串少了一个1，所以字符串A小于字符串B。对于这种对象的比较jdk是如何完成的呢？下面是源码的实现代码：
```
public int compareTo(String anotherString) {
        int len1 = value.length;
        int len2 = anotherString.value.length;
        int lim = Math.min(len1, len2);
        char v1[] = value;
        char v2[] = anotherString.value;

        int k = 0;
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        return len1 - len2;
    }
```
它的实现方法很容易理解，以小的那个字符串长度作为循环次数进行遍历，从第一个字符开始比较ASCII码，一旦不相等就返回他们ASCII码的差值，如果前面都相同了，就返回长度的差值。嗯嗯，很nice我们读懂了String类的比较过程，但是其他类对象如何进行比较呢？它是怎样实现比较的呢？

在之前看String类和Date类定义的时候我们发现，他们都实现了Comparable接口，而compareTo方法就定义在这个接口里面。Java中有两大比较器，分别是Comparable和Comparator，这两个比较器都能实现对象的比较，接下来就来学习一下这两个接口，在学习之前先回顾一下Arrays类。
# Arrays类
Arrays类是一个提供数组操作的相关方法的类，常用的方法如下：
```
Arrays.sort(array);  // 数组排序，支持对象数组也支持基本数据类型数组，如果是对象数组必须实现Comparable接口
Arrays.toString(array);  // 将数组以字符串的形式输出（并不是Object类中的toString，切记！！！）
```
如果一个自定义类没有实现Comparable接口将会抛出异常（类型转换异常），在sort方法中如果是对象比较它会调用ComparableTimSort类的sort方法，而sort方法又是调用的binarySort二叉树排序方法，在binarySort中有强制转换，是将你传入的Object类强转为Comparable类，向下强转是可能抛出异常的，所以如果需要排序的对象没有实现Comparable接口就无法使用Arrays.sort()方法。
# Comparable接口
我们以人作为研究对象，先定义一个People对象。
```
/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 16:42 2019/1/19
 * @Description:
 */
public class People {

    private String name;

    private double height;

    public People(){}

    public People(String name, double height){
        this.name = name;
        this.height = height;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getHeight() {
        return height;
    }

    public void setHeight(double height) {
        this.height = height;
    }

    @Override
    public String toString() {
        return "name: " + this.name + ", Height " + this.height;
    }
}

```
如果没有实现Comparable接口直接进行排序（排序的基础就是比较）
```
package language;

import java.util.Arrays;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 15:55 2019/1/19
 * @Description:
 */
public class ComparatorTest{

    public static void main(String[] args) {
        People peopleA = new People("小智",1.80);
        People peopleB = new People("小明",1.70);
        People[] peoples = new People[]{peopleA,peopleB};
        Arrays.sort(peoples);
        System.out.println(Arrays.toString(peoples));
    }
}
```
最后一运行，得到了我们预期的ClassCastException异常
```
Exception in thread "main" java.lang.ClassCastException: language.People cannot be cast to java.lang.Comparable
	at java.util.ComparableTimSort.countRunAndMakeAscending(ComparableTimSort.java:320)
	at java.util.ComparableTimSort.sort(ComparableTimSort.java:188)
	at java.util.Arrays.sort(Arrays.java:1246)
	at language.ComparatorTest.main(ComparatorTest.java:17)
```
错误栈里面我们可以看到是ComparableTimSort的binarySort方法报错，正如上面所说。如果要想一个对象具有可比较性，我们必须定义它的比较规则，比如人比较有很多种比较，是比较身高？还是年龄？还是其他，比如书本是比较出版时间？还是价格等等，所以在不同的应用场景有不同的比较规则，这里我们设定比较人的身高的规则，改后的People类如下：
```
package language;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 16:42 2019/1/19
 * @Description:
 */
public class People implements Comparable<People>{

    private String name;

    private double height;

    public People(){}

    public People(String name, double height){
        this.name = name;
        this.height = height;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getHeight() {
        return height;
    }

    public void setHeight(double height) {
        this.height = height;
    }

    @Override
    public String toString() {
        return "name: " + this.name + ", Height " + this.height;
    }

    @Override
    public int compareTo(People o) {
        if (this.height > o.height){
            return 1;
        }else if(this.height < o.height){
            return -1;
        }else{
            return 0;
        }
    }
}
```
运行之后我们可以看到，实现了排序，小智在小明后面
```
[name: 小明, Height 1.7, name: 小智, Height 1.8]
```
# Comparator接口
在开发过程中难免会遇到这样的情况：在写代码时没有排序的需求，对象都已经定义好了并且不可以修改（对象没有实现Comparable接口）。突然某天要改需要，要实现一个对象的排序，但又不能改这个对象的代码，这时就需要增强比较Comparator接口了。

现在我把People类改回到原来的定义，也就是没有实现Comparable接口。Comparator的出现也就是为了解决上述难题，有了它之后解决方案是新定义一个可比较的People类，我取名为PeopleComparator，然后让它实现Comparator<T>接口
```
class PeopleComparator implements Comparator<People> {

    @Override
    public int compare(People o1, People o2) {
        if (o1.getHeight() > o2.getHeight()){
            return 1;
        }else if(o1.getHeight() < o2.getHeight()){
            return -1;
        }else{
            return 0;
        }
    }
}
```
之前实现Comparable接口的时候我们使用的是Arrays.sort()方法进行排序，而现在是实现Comparator接口，那么我们可以使用另一个被重载的Arrays.sort()方法：
```
public static <T> void sort(T[] a, Comparator<? super T> c);
```
我们看到传入的Comparator类型约束的泛型是设定有下限的，也就是说只允许T类及其父类才允许传入，这一点需要注意。
```
package language;

import java.util.Arrays;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 15:55 2019/1/19
 * @Description:
 */
public class ComparatorTest{

    public static void main(String[] args) {
        People peopleA = new People("小智",1.80);
        People peopleB = new People("小明",1.70);
        People peopleC = new People("小方",1.83);
        People[] peoples = new People[]{peopleA,peopleB,peopleC};
        Arrays.sort(peoples,new PeopleComparator());
        System.out.println(Arrays.toString(peoples));
    }
}
```
输出结果：
```
[name: 小明, Height 1.7, name: 小智, Height 1.8, name: 小方, Height 1.83]
```
# 总结
> Comparable和Comparator的区别：
> * 如果对象数组要进行排序那么必须设置排序规则，可以使用Comparable或Comparator接口实现；
> * java.lang.Comparable是在一个类定义的时候实现好接口，这样本类的数组对象就可以进行排序，在Comparable接口下定义有public int compareTo()方法；
> * java.util.Comparator是专门定义一个指定类的比较规则，属于挽救的比较操作，里面有两个方法：public int compare()、public boolean equals() 。