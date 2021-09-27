#### 1.java高版本中找不到@PostConstruct注解

---

解决方案：

在pom中添加依赖

~~~xml
<!-- https://mvnrepository.com/artifact/javax.annotation/javax.annotation-api -->
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
~~~



#### 2.java判断设备是否联网

---

方式一： InetAddress.isReachable（快，大概几毫秒）

**注：**InetAddress.isReachable方法通过TCP协议检测，连接设备端口，会因设备权限或防火墙之类的东西，端口有限制，容易连接不成功！

~~~java
    public boolean isNetworkConnected() {
        try {
            InetAddress address = InetAddress.getByName(ipAddress);
            return address.isReachable(500);
        } catch (IOException e) {
            log.warn("Failed to connect to the network : {}", e.getMessage());
        }
        return false;
    }
~~~

方式二：ping ip地址（慢，大概3000多毫秒）

~~~java
    public boolean isConnected(){
        try {
            return  0 == Runtime.getRuntime().exec("ping " + ipAddress).waitFor();
        } catch (InterruptedException | IOException e) {
            log.warn("Failed to connect to the network : {}", e.getMessage());
        }
        return false;
    }
~~~



[process的waitFor方法](https://blog.csdn.net/jimzhai/article/details/7864806)



#### 3.IDEA破解方法

---



1. 把jetbrains-agent.jar文件放在idea的bin目录下

2. 点击idea的"Help" -> "Edit Custom VM Options ..."，添加下方条件

   ~~~vm
   -javaagent:D:\IntelliJ IDEA 2019.3.3\bin\jetbrains-agent.jar
   ~~~

  3.重启idea

  4.选择Activation code方式离线激活

  [破解码1](http://www.xmpojie.com/3687.html)

  [破解码2](https://blog.csdn.net/weixin_39771614/article/details/110479556)



#### 4.stream的API

|       方法        |                参数                 |                             说明                             |
| :---------------: | :---------------------------------: | :----------------------------------------------------------: |
|     filter()      | 接收一个参数，需要返回一个boolean型 |          方法中需要返回一个boolean型，用于过滤数据           |
|       map()       | 接受一个参数，返回另一个类型的参数  |             可以将一种类型的值转换成另外一种类型             |
|      count()      |          无参数，返回long           |                        实现集合的计数                        |
|     collect()     |            返回集合类型             |                       转成集合，不解释                       |
|     foreach()     |       接收一个参数，无返回值        |                       循环遍历，不解释                       |
|    distinct()     |               无参数                |            根据对象的hashCode和equals方法进行去重            |
|     reduce()      |                                     | 接收两个类型一样的值，并返回同一个类型的值，可以用于求和，求乘机等等 |
|     sorted()      |                                     |                   排序，不解释，直接看代码                   |
|    allMatch()     |                                     | 判断集合中的元素是否都符合，都符合返回true，有一个元素不符合就会返回false |
|    anyMatch()     |                                     |         集合里的元素有一个元素符合判断条件就返回true         |
|    noneMatch()    |                                     | 和allMatch相反，如果集合元素都不符合判断条件，返回true，如果有一个元素符合判断条件，返回false |
|    dropWhile()    |                                     | 接收的lamdba表达式需要返回一个boolean值，如果是true，就drop掉该元素，当遇到第一个false时，保留其元素及后面所有元素。看下代码示例吧 |
|     findAny()     |                                     | 不接受参数，虽然是any，但是也是返回集合中的第一个元素，一般搭配filter方法使用 |
|    findFirst()    |                                     |              不接受参数，返回集合中的第一个元素              |
|     flatMap()     |                                     |             接收一个参数，返回结果是一个stream流             |
| flatMapToDouble() |                                     |                       返回DoubleStream                       |
|  flatMapToInt()   |                                     |                        返回IntStream                         |
|  flatMapToLong()  |                                     |                        返回LongStream                        |
| forEachOrdered()  |                                     | foreach在并行流下输出不一定是按照顺序的，如果想要按照顺序需要使用forEachOrdered |
|      limit()      |                                     |               接收参数n，获取集合中的前n个元素               |
|   mapToDouble()   |                                     |                  将输出一个double类型的结果                  |
|    mapToInt()     |                                     |                   将输出一个int类型的结果                    |
|    mapToLong()    |                                     |                   将输出一个long类型的结果                   |
|       max()       |                                     |                           求最大值                           |
|       min()       |                                     |                           求最小值                           |
|       sum()       |                                     |                             求和                             |
|      peek()       |                                     |   该方法主要是支持调试，能够查看元素流过管道中特定点的情况   |
|      skip()       |                                     |    接收参数n，跳过集合中的前n个元素，可以和limit实现分页     |
|    takeWhile()    |                                     |             和filter功能类似，还没查到有什么区别             |
|     toArray()     |                                     |                           输出数组                           |



##### **reduce()示例**

~~~java
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        list.add(5);
        Integer integer = list.stream().reduce((s1, s2) -> s1 * s2).get();
        System.out.println(integer);//120
~~~



##### **sorted()示例**

~~~java
        //按照Dict的id正向排序
        List<Dict> dicts1 = dicts.stream().sorted(Comparator.comparing(Dict::getId)).collect(Collectors.toList());
        //按照Dict的code倒序排序
        List<Dict> dicts2 = dicts.stream().sorted(Comparator.comparing(Dict::getCode).reversed()).collect(Collectors.toList());

~~~



##### **dropwhile()示例**

~~~java
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(2);
        list.add(5);
        // 1 和 2小于 3，返回false， 所有返回 [1 2 3 4 5]
        System.out.println(list.stream().dropWhile(n -> n > 3).collect(Collectors.toList()));
        // 1和 2 小于 3，前两个元素返回true，所以drop掉，返回 [3, 2, 5]虽然第四个元素 2 也小于3，但是 3  < 3已经返回了false，后面的元素就不在判断了
        System.out.println(list.stream().dropWhile(n -> n < 3).collect(Collectors.toList()));
~~~



##### **findAny()和findFirst()**

- 区别：findFirst()和findAny()存在并行上的区别，findFirst并行限制较多，findAny并行限制较少，如果不在乎哪个值，用findAny。

- 细节：使用orElse，不使用get方法

  ~~~java
          List<Integer> list = List.of(1, 2, 3, 4, 5);
          //输出 4
          System.out.println(list.stream().filter( n -> n > 3).findAny().get());
          //集合中没有大于6的元素，会抛错
          System.out.println(list.stream().filter(n -> n > 6).findAny().get());
          //使用orElse避免上面这种报错，如果集合没有元素，返回null
          System.out.println(list.stream().filter(n -> n > 6).findAny().orElse(null));
  ~~~



##### **map()和flatMap()**

- 区别：map只是一维 1对1 的映射，而flatmap可以将一个2维的集合映射成一个一维,相当于他映射的深度比map深了一层

- 示例：

  ~~~java
          /**获取单词，并且去重**/
          List<String> list = Arrays.asList("beijing changcheng", "beijing gugong", "beijing tiantan", "gugong tiananmen");
          //使用map只能转为 string数组的list集合
          List<String[]> collect1 = list.stream().map(s -> s.split(" ")).collect(Collectors.toList());
          //使用flatMap可以将字符串分割成各自的list之后直接合并成一个List
          List<String> collect = list.stream().flatMap(s -> Arrays.stream(s.split(" "))).collect(Collectors.toList());
          System.out.println(collect);
  ~~~



##### **foreachOrdered()示例**

~~~java
        List<Integer> list = List.of(1, 2, 3, 4, 5);
        //list.stream().forEach(System.out::println);
        //foreach在并行流下输出不一定是按照顺序的，如果想要按照顺序需要使用forEachOrdered
        list.parallelStream().forEach(System.out::println);
        list.parallelStream().forEachOrdered(System.out::println);
~~~





#### 5.System.lineSeparator()的意义

System.lineSeparator()返回的是行分隔符，它会根据当前的电脑系统返回对应的行分隔符

在UNIX系统下，System.lineSeparator()方法返回 "\n"

在Windows系统下，System.lineSeparator()方法返回 "\r\n"

System.lineSeparator()方法会根据当前的系统返回对应的行分隔符。从而避免了你编写的程序在windows系统上可以运行，linux/unix系统上无法运行的情况。

 

#### 6.windows安装mysql8.0.13教程

---

[下载地址](https://downloads.mysql.com/archives/community/)

[教程](https://www.jianshu.com/p/3b57c2856fc5)



#### 7.linux上传下载文件

---



首先使用下面命令安装

~~~bash
yum install  lrzsz -y
~~~

结果

~~~bash
Total download size: 78 k
Installed size: 181 k
Downloading packages:
lrzsz-0.12.20-36.el7.x86_64.rpm                                     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : lrzsz-0.12.20-36.el7.x86_64                          
  Verifying  : lrzsz-0.12.20-36.el7.x86_64                          

Installed:
  lrzsz.x86_64 0:0.12.20-36.el7                                     

Complete!

~~~

验证，输入：

~~~bash
rpm -qa |grep lrzsz
~~~

结果：

~~~bash
lrzsz-0.12.20-36.el7.x86_64
~~~



- 下载到本地：

  sz 文件名

- 上传到服务器：

  rz
