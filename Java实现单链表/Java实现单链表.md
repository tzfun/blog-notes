单链表是最基本的数据结构之一，之前在学数据结构的时候用C语言写过单链表，但是还从来没用Java写过，尝试之后才发现Java用对象实现起来虽然没有C那么简便快捷，但是却更加灵活。
# 定义Link链表类
这里我是用内部类来定义每一个链表的节点Node，先定义基本的框架，其余的方法下面一一添加。
```
class Link{
    /**
     * 定义链表节点，为了方便数据访问所以我用内部类
     * */
    private class Node{
        private Object data;
        private Node next;
        public Node(Object data){
            this.data = data;
        }
    }

    private Node root;  // 链表根节点
    private int count = 0; // 保存元素的个数
    private int foot = 0;   // 保存链表的索引
    private Object[] resArray; // 返回对象数组
}
```

# 链表添加数据
定义Node类中添加void addNode(Node newNode)方法
```
    /**
     * 增加新节点
     * @param newNode 新的Node对象
     */
    public void addNode(Node newNode){
        if(this.next == null){
            this.next = newNode;
        }else{  // 向后继续保存
            this.next.addNode(newNode);
        }
    }
```
在Link类添加void add(Object obj)方法
```
    /**
     * 向链表中添加元素
     * @param data 添加的数据
     * */
    public void add(Object data) {
        if (data == null) {
            return;
        } else {
            Node newNode = new Node(data);
            if (this.root == null) {
                this.root = newNode;
            } else {
                this.root.addNode(newNode);
            }
        }
        this.count ++;
    }
```
# 获取链表长度
Link类中添加方法int size();
```
    /**
     * 获取链表的长度
     * @return int
     * */
    public int size(){
        return this.count;
    }
```
# 判断链表是否为空
Link类中添加方法boolean isEmpty();
```
    /**
     * 判断链表是否为空
     * @return boolean
     * */
    public boolean isEmpty() {
        if (root == null) return true;
        return false;
    }
```
# 查找数据是否存在
在Node类中添加方法boolean containsNode(Object obj)
```
    /**
     * 查询数据是否存在于节点中
     * @param data
     * @return boolean
     */
    public boolean containsNode(Object data){
        if(this.data.equals(data)){
            return true;
        }else{
            if(this.next == null){
                return false;
            }else {
                return this.next.containsNode(data);
            }
        }
    }
```
在Link类中添加方法boolean contains(Object obj)
```
    /**
     * 根据节点内容判断某一个数据是否存在
     * @return boolean
     * */
    public boolean contains(Object data){
        if (data == null || this.root == null) return false;
        return this.root.containsNode(data);
    }
```
# 根据索引取得数据
在Node类中添加方法Object getNode(int index)
```
    /**
     * 获取某一索引的节点数据
     * @param index 索引
     * @return Object
     */
    public Object getNode(int index) {
        // 使用当前节点索引与index比较，随后自增，目的是为了下一步查询
        if(Link.this.foot ++ == index){ // 如果当前节点索引与index相等就返回当前节点的data
            return this.data;
        }else{
            return this.next.getNode(index);
        }
    }
```
在Link类中添加方法Object get(int index)
```
    /**
     * 根据索引取得数据
     * */
    public Object get(int index) {

        if(index+1 > this.count){
            try {
                throw new Exception("索引值超出链表长度！");
            } catch (Exception e) {
                e.printStackTrace();
            }
            return null;
        }else{
            this.foot = 0;
            return this.root.getNode(index);
        }
    }
```
# 根据索引值修改链表内容
在Node类添加方法void setNode(int index, Object data)
```
    /**
     * 根据索引值设置节点内容
     * @param index 索引
     * @param data 数据
     */
    public void setNode(int index, Object data) {
        if(Link.this.foot ++ == index){
            this.data = data;
        }else {
            this.next.setNode(index,data);
        }
    }
```
在Link类添加方法void set(int index,Object data)
```
    /**
     * 根据索引值修改节点内容
     * @param index 节点索引
     * @param data 节点数据
     */
    public void set(int index,Object data){
        if(index+1 > this.count){
            try {
                throw new Exception("索引值超出链表长度！");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }else {
            this.foot = 0;
            this.root.setNode(index,data);
        }
    }
```
# 打印链表所有内容
在Node类里添加void printNode()
```
    /**
     * 打印出节点数据内容
     * @return
     */
    public void printNode() {
        System.out.println(this.data);
        if(this.next != null){
            this.next.printNode();
        }
    }
```
在Link类里添加void print()
```
    /**
     * 打印链表所有内容
     */
    public void print(){
        this.foot =0;
        if (this.count != 0){
            this.root.printNode();
        }
    }
```
# 根据数据内容删除节点
在Node类中添加方法void removeNode(Node previous, Object data)
```
    /**
     * 根据索引值删除节点
     * @param index
     */
    public void removeNode(Node previous, Object data) {
        if(this.data == data){
            previous.next = this.next;
        }else {
             this.next.removeNode(this,data);
        }
    }
```
在Link类中添加方法void remove(Object data)
```
    /**
     * 根据数据内容删除节点
     * @param index
     */
    public void remove(Object data){
        if (!this.contains(data)){
            try {
                throw new Exception("链表内不存在该数据！");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }else if(data.equals(this.root.data)){
            this.root = this.root.next;
        }else{
            this.root.next.removeNode(this.root,data);
        }
        this.count --;
    }
```
# 返回对象数组
在Link类中添加方法Object[] toArray()
```
    /**
     * 将链表以对象数组形式返回
     * @return
     */
    public Object[] toArray(){
        if(this.root == null){
            return null;
        }
        this.resArray = new Object[this.count];
        this.foot = 0;
        while(this.foot != this.count){
            resArray[this.foot] = this.get(this.foot);
        }
        return resArray;
    }
```
# 测试链表
我们在main方法中对上面链表的增、删、改、查进行测试
```
public class LinkDemo {
    public static void main(String[] args) {
//        new 一个链表对象
        Link link = new Link();
        System.out.println("--------------------添加数据前-----------------");
        System.out.println("链表是否为空:"+link.isEmpty());
        System.out.println("链表的长度:"+link.size());

//        向链表中添加一个字符串对象
        link.add("A");
//        向链表中添加一个char型数据
        link.add('B');
//        向链表中添加一个Boolean对象
        link.add(true);
//        向链表中添加一个int型数据
        link.add(123);
//        向链表中添加一个集合对象
        ArrayList arrayList = new ArrayList<String>();
        arrayList.add("hello");
        arrayList.add("world");
        arrayList.add("!");
        link.add(arrayList);
        System.out.println("--------------------添加数据后-----------------");
//        判断是否为空
        System.out.println("链表是否为空:"+link.isEmpty());
//        获取链表长度
        System.out.println("链表的长度:"+link.size());
//        根据索引值获取某一个节点数据
        System.out.println("获取第2个节点的数据:"+link.get(1));
//        修改第一个节点数据为tianzhi
        link.set(0,"tianzhi");
        System.out.println("修改第1个节点数据为tianzhi后第1个节点:"+link.get(0));
        System.out.println("************删除节点内容值为true之前************");
        link.print();
//        删除节点内容值为true的节点
        link.remove(true);
        System.out.println("************删除节点内容值为true之后************");
        link.print();
//        返回对象数组
        System.out.println("++++++++++++以下是对象数组输出++++++++++++");
        for (int i = 0;i<link.toArray().length;i++){
            System.out.println(link.toArray()[i]);
        }
//        测试节点长度越界情况
        System.out.println("############以下是测试链表越界############");
        link.get(100);
    }
}
```
最终输出结果:
```
--------------------添加数据前-----------------
链表是否为空:true
链表的长度:0
--------------------添加数据后-----------------
链表是否为空:false
链表的长度:5
获取第2个节点的数据:B
修改第1个节点数据为tianzhi后第1个节点:tianzhi
************删除节点内容值为true之前************
tianzhi
B
true
123
[hello, world, !]
************删除节点内容值为true之后************
tianzhi
B
123
[hello, world, !]
++++++++++++以下是对象数组输出++++++++++++
tianzhi
B
123
[hello, world, !]
############以下是测试链表越界############
java.lang.Exception: 索引值超出链表长度！
	at language.Link.get(LinkDemo.java:157)
	at language.LinkDemo.main(LinkDemo.java:277)
```
# 单链表全部代码
```
package language;

import java.util.ArrayList;
import java.util.List;

/**
 * @Author beifengtz
 * @Date Created in 23:07 2018/9/22
 * @Description:
 */
class Link{

    /**
     * 定义链表节点
     * */
    private class Node{
        private Object data;
        private Node next;
        public Node(Object data){
            this.data = data;
        }

        /**
         * 增加新节点
         * @param newNode 新的Node对象
         */
        public void addNode(Node newNode){
            if(this.next == null){
                this.next = newNode;
            }else{  // 向后继续保存
                this.next.addNode(newNode);
            }
        }

        /**
         * 查询数据是否存在于节点中
         * @param data
         * @return boolean
         */
        public boolean containsNode(Object data){
            if(this.data.equals(data)){
                return true;
            }else{
                if(this.next == null){
                    return false;
                }else {
                    return this.next.containsNode(data);
                }
            }
        }

        /**
         * 获取某一索引的节点数据
         * @param index 索引
         * @return Object
         */
        public Object getNode(int index) {
//            使用当前节点索引与index比较，随后自增，目的是为了下一步查询
            if(Link.this.foot ++ == index){ // 如果当前节点索引与index相等就返回当前节点的data
                return this.data;
            }else{
                return this.next.getNode(index);
            }
        }

        /**
         * 根据索引值设置节点内容
         * @param index 索引
         * @param data 数据
         */
        public void setNode(int index, Object data) {

            if(Link.this.foot ++ == index){
                this.data = data;
            }else {
                this.next.setNode(index,data);
            }
        }

        /**
         * 打印出节点数据内容
         * @return
         */
        public void printNode() {
            System.out.println(this.data);
            if(this.next != null){
                this.next.printNode();
            }
        }

        /**
         * 根据索引值删除节点
         * @param index
         */
        public void removeNode(Node previous, Object data) {
            if(this.data == data){
                previous.next = this.next;
            }else {
                this.next.removeNode(this,data);
            }
        }
    }

    private Node root;  // 链表根节点
    private int count = 0; // 保存元素的个数
    private int foot = 0;   // 保存链表的索引
    private Object[] resArray; // 返回的对象数组
    /**
     * 向链表中添加元素
     * @param data 添加的数据
     * */
    public void add(Object data) {
        if (data == null) {
            return;
        } else {
            Node newNode = new Node(data);
            if (this.root == null) {
                this.root = newNode;
            } else {
                this.root.addNode(newNode);
            }
        }
        this.count ++;
    }
    /**
     * 获取链表的长度
     * @return int
     * */
    public int size(){
        return this.count;
    }

    /**
     * 判断链表是否为空
     * @return boolean
     * */
    public boolean isEmpty() {
        if (root == null) return true;
        return false;
    }

    /**
     * 根据节点内容判断某一个数据是否存在
     * @return boolean
     * */
    public boolean contains(Object data){
        if (data == null || this.root == null) return false;
        return this.root.containsNode(data);
    }
    /**
     * 根据索引取得数据
     * */
    public Object get(int index) {

        if(index+1 > this.count){
            try {
                throw new Exception("索引值超出链表长度！");
            } catch (Exception e) {
                e.printStackTrace();
            }
            return null;
        }else{
            this.foot = 0;
            return this.root.getNode(index);
        }
    }

    /**
     * 根据索引值修改节点内容
     * @param index 节点索引
     * @param data 节点数据
     */
    public void set(int index,Object data){
        if(index+1 > this.count){
            try {
                throw new Exception("索引值超出链表长度！");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }else {
            this.foot = 0;
            this.root.setNode(index,data);
        }
    }

    /**
     * 打印链表所有内容
     */
    public void print(){
        this.foot =0;
        if (this.count != 0){
            this.root.printNode();
        }
    }

    /**
     * 根据数据内容删除节点
     * @param index
     */
    public void remove(Object data){
        if (!this.contains(data)){
            try {
                throw new Exception("链表内不存在该数据！");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }else if(data.equals(this.root.data)){
            this.root = this.root.next;
        }else{
            this.root.next.removeNode(this.root,data);
        }
        this.count --;
    }

    /**
     * 将链表以对象数组形式返回
     * @return
     */
    public Object[] toArray(){
        if(this.root == null){
            return null;
        }
        this.resArray = new Object[this.count];
        this.foot = 0;
        while(this.foot != this.count){
            resArray[this.foot] = this.get(this.foot);
        }
        return resArray;
    }
}
public class LinkDemo {
    public static void main(String[] args) {
//        new 一个链表对象
        Link link = new Link();
        System.out.println("--------------------添加数据前-----------------");
        System.out.println("链表是否为空:"+link.isEmpty());
        System.out.println("链表的长度:"+link.size());

//        向链表中添加一个字符串对象
        link.add("A");
//        向链表中添加一个char型数据
        link.add('B');
//        向链表中添加一个Boolean对象
        link.add(true);
//        向链表中添加一个int型数据
        link.add(123);
//        向链表中添加一个集合对象
        ArrayList arrayList = new ArrayList<String>();
        arrayList.add("hello");
        arrayList.add("world");
        arrayList.add("!");
        link.add(arrayList);
        System.out.println("--------------------添加数据后-----------------");
//        获取链表长度
        System.out.println(link.size());
//        判断是否为空
        System.out.println("链表是否为空:"+link.isEmpty());
        System.out.println("链表的长度:"+link.size());
//        根据索引值获取某一个节点数据
        System.out.println("获取第2个节点的数据:"+link.get(1));
//        修改第一个节点数据为tianzhi
        link.set(0,"tianzhi");
        System.out.println("修改第1个节点数据为tianzhi后第1个节点:"+link.get(0));
        System.out.println("************删除节点内容值为true之前************");
        link.print();
//        删除节点内容值为true的节点
        link.remove(true);
        System.out.println("************删除节点内容值为true之后************");
        link.print();
//        返回对象数组
        System.out.println("++++++++++++以下是对象数组输出++++++++++++");
        for (int i = 0;i<link.toArray().length;i++){
            System.out.println(link.toArray()[i]);
        }
//        测试节点长度越界情况
        System.out.println("############以下是测试链表越界############");
        link.get(100);
    }
}

```