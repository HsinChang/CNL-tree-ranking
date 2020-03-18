- [For interviews](#for-interviews)
  - [Java](#java)
    - [Singleton](#singleton)
    - [Factory](#factory)


# For interviews
## Java
### Singleton
饿汉式
```java
public class SingleObject {
 
   //创建 SingleObject 的一个对象
   private static SingleObject instance = new SingleObject();
 
   //让构造函数为 private，这样该类就不会被实例化
   private SingleObject(){}
 
   //获取唯一可用的对象
   public static SingleObject getInstance(){
      return instance;
   }
 
   public void showMessage(){
      System.out.println("Hello World!");
   }
}
```
```java
public class SingletonPatternDemo {
   public static void main(String[] args) {
 
      //不合法的构造函数
      //编译时错误：构造函数 SingleObject() 是不可见的
      //SingleObject object = new SingleObject();
 
      //获取唯一可用的对象
      SingleObject object = SingleObject.getInstance();
 
      //显示消息
      object.showMessage();
   }
}
```
### Factory
定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。<br>
https://www.runoob.com/design-pattern/factory-pattern.html <br>
使用场景： 1、日志记录器：记录可能记录到本地硬盘、系统事件、远程服务器等，用户可以选择记录日志到什么地方。 2、数据库访问，当用户不知道最后系统采用哪一类数据库，以及数据库可能有变化时。 3、设计一个连接服务器的框架，需要三个协议，"POP3"、"IMAP"、"HTTP"，可以把这三个作为产品类，共同实现一个接口。
### Builder
将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。<br>
https://www.runoob.com/design-pattern/builder-pattern.html <br>
应用实例： 1、去肯德基，汉堡、可乐、薯条、炸鸡翅等是不变的，而其组合是经常变化的，生成出所谓的"套餐"。 2、JAVA 中的 **StringBuilder**。
### Adapter 适配器
意图：将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作 <br>
https://www.runoob.com/design-pattern/adapter-pattern.html <br>
应用实例： 1、美国电器 110V，中国 220V，就要有一个适配器将 110V 转化为 220V。 2、JAVA JDK 1.1 提供了 Enumeration 接口，而在 1.2 中提供了 Iterator 接口，想要使用 1.2 的 JDK，则要将以前系统的 Enumeration 接口转化为 Iterator 接口，这时就需要适配器模式。 3、在 LINUX 上运行 WINDOWS 程序。 4、JAVA 中的 jdbc。 <br>
注意事项：适配器不是在详细设计时添加的，而是解决正在服役的项目的问题

### Decorator 装饰器
意图：动态地给一个对象添加一些额外的职责。就增加功能来说，装饰器模式相比生成子类更为灵活。<br>

### Strategy
意图：定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。<br>
使用场景： 1、如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为。 2、一个系统需要动态地在几种算法中选择一种。 3、如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现。

### Observer
意图：定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。<br>

### JVM
Java虚拟机是一个可以执行Java字节码的虚拟机进程。Java源文件被编译成能被Java虚拟机执行的字节码文件。 Java被设计成允许应用程序可以运行在任意的平台，而不需要程序员为每一个平台单独重写或者是重新编译。Java虚拟机让这个变为可能，因为它知道底层硬件平台的指令长度和其他特性。
#### garbage collection 垃圾回收
https://liuchi.coding.me/2017/08/05/%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90Java%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E6%9C%BA%E5%88%B6/

## 握手 handshake
https://blog.csdn.net/hyg0811/article/details/102366854?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task

## Hive
### Hive数据倾斜问题：

倾斜原因： map输出数据按Key Hash分配到reduce中,由于key分布不均匀、或者业务数据本身的特点。等原因造成的reduce上的数据量差异过大。

1.1)key分布不均匀

1.2)业务数据本身的特性

1.3)SQL语句造成数据倾斜

解决方案：

1>参数调节：

    hive.map.aggr=true

    hive.groupby.skewindata=true

有数据倾斜的时候进行负载均衡，当选项设定为true,生成的查询计划会有两个MR Job。第一个MR Job中，Map的输出结果集合会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；第二个MR Job在根据预处理的数据结果按照 Group By Key 分布到Reduce中(这个过程可以保证相同的 Group By Key 被分布到同一个Reduce中)，最后完成最终的聚合操作。

2>SQL语句调节：

   1)选用join key 分布最均匀的表作为驱动表。做好列裁剪和filter操作，以达到两表join的时候，数据量相对变小的效果。

   2)大小表Join： 使用map join让小的维度表（1000条以下的记录条数）先进内存。在Map端完成Reduce。

   3)大表Join大表：把空值的Key变成一个字符串加上一个随机数，把倾斜的数据分到不同的reduce上，由于null值关联不上，处理后并不影响最终的结果。

   4)count distinct大量相同特殊值：count distinct时，将值为空的情况单独处理，如果是计算count distinct，可以不用处理，直接过滤，在做后结果中加1。如果还有其他计算，需要进行group by，可以先将值为空的记录单独处理，再和其他计算结果进行union.
### Hive中的排序关键字有哪些
sort by ，order by ，cluster by ，distribute by

sort by ：不是全局排序，其在数据进入reducer前完成排序

order by ：会对输入做全局排序，因此只有一个reducer(多个reducer无法保证全局有序).只有一个reducer,会导致当输入规模较大时，需要较长的计算时间。

cluster by ： 当distribute by 和sort by的字段相同时，等同于cluster by.可以看做特殊的distribute + sort

distribute by ：按照指定的字段对数据进行划分输出到不同的reduce中

https://blog.csdn.net/wxfghy/article/details/81361400
 
### Hive Join 的实现
https://blog.csdn.net/u013668852/article/details/79768266
