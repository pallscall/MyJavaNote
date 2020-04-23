# Java8新特性

## 1.Lambda表达式

### 1.1 什么是Lambda？

Lambda是 Java 8 添加的新的特性。说白了，Lambda就是一个匿名函数

### 1.2 为什么要使用Lambda

使用Lambda表达式可以对一个接口进行非常简单的实现。

### 1.3 Lambda对接口的要求

虽然i使用Lambda表达式对某些接口进行简单的实现，但是并不是所有的接口都可以用Lambda表达式来实现，要求接口中定义的必须要实现的抽象方法只能是一个。

> 在Java 8 对接口加了一个新的特性： default。由default修饰的方法在接口中有默认实现

@FunctionInterface 函数式接口注解，保证接口中只有一个方法。



### 1.4 基础语法

```java
()：描述参数列表

{}：描述方法体

->：Lambda运算符，读作 goes to
```



基础demo

```java
// 无参无返回值
interface noReturnNoParam{
    void test();
}

//单参无返回值
interface noReturnSingleParam{
    void test(int n);
}

//无参有返回值
interface SingleReturnNoParam{
    int test();
}

//多参无返回值
interface noReturnMultiParam{
    void test(int a,int b);
}

//单参有返回值
interface SingleReturnSingleParam{
    int test(int a);
}

//多参有返回值
interface SingleReturnMultiParam{
    int test(int a,int b);
}
```

```java
public class Program {
    public static void main(String[] args) {
        // 无参无返回值
       noReturnNoParam lambda1=()->{
           System.out.println("lambda1：hello world");
       };
       lambda1.test();

        //单参无返回值
       noReturnSingleParam lambda2=(int a)->{
           System.out.println("lambda2: "+a);
       };
       lambda2.test(10);

        //多参无返回值
       noReturnMultiParam lambda3=(a,b)->{
           System.out.println("lambda3: "+(a+b));
       };
       lambda3.test(10,20);

        //无参有返回值
       SingleReturnNoParam lambda4=()->{
           return 100;
       };
       System.out.println("lambda4: "+lambda4.test());

        //单参有返回值
       SingleReturnSingleParam lambda5=(a)->{
           return a*2;
       };
        System.out.println("lambda5: "+lambda5.test(10));

        //多参有返回值
        SingleReturnMultiParam lambda6=(a,b)->{
            return a*b;
        };
        System.out.println("lambda6: "+lambda6.test(2,3));
    }
}
```

```java
//结果
lambda1：hello world
lambda2: 10
lambda3: 30
lambda4: 100
lambda5: 20
lambda6: 6
```

### 1.5 语法精简

1. 参数

   由于在接口的抽象方法中，已经定义了参数的数量和类型，所以在Lambda表达式中，参数的类型可以省略。

   备注：如果需要省略类型，则每一个参数的类型都要省略。

   `(int a,int b)->{} `=> `(a,b)->{}`

2. 参数小括号

   如果参数列表中，参数的数量只有一个，此时小括号可以省略

   `(a)->{}` => `a->{}`

3. 方法大括号

   如果方法体中只有一条语句，此时大括号可以省略

   `a->{System.out.println("hello!");}`=>`a->System.out.println("hello!");`

4. 省略return

   如果方法体中的唯一一条语句是一条返回语句，则在省略大括号的同时也必须省略return

   `a->{return 10;}`=>`a->10`;

### 1.6 语法进阶

#### 1.6.1 方法引用

可以快速地将一个Lambda表达式的实现指向一个已经实现的方法

语法： 方法的隶属者::方法名

```java
public class Program {
    public static void main(String[] args) {
        SingleReturnSingleParam lambda1 = a->change(a);
    }

    private static int change(int a) {
        return a<<1;
    }
}
```

其中

```java
SingleReturnSingleParam lambda1 = a->change(a);
等价于
SingleReturnSingleParam lambda2 = Program::change;
```



注意：

1.方法引用的参数的数量和类型一定要和接口中定义的方法一致

2.方法引用的返回值类型一定要和接口中定义的方法一致



#### 1.6.2 构造方法引用

```java
class Person{
    public String name;
    public int age;
    public Person(){
        System.out.println("无参构造方法执行了。");
    }
    public Person(String name,int age){
        this.name = name;
        this.age = age;
        System.out.println("有参构造方法执行了。");
    }
}
interface PersonCreater{
    Person getPerson();
}
```

```java
public class Program {
    public static void main(String[] args) {
        PersonCreater personCreater = ()-> new Person();

        //构造方法的引用
        PersonCreater personCreater1 = Person::new;

        Person a = personCreater1.getPerson();
    }
}
```



#### 1.6.3 集合排序

```java
        //需求：按照年龄排序
        ArrayList<Person> list = new ArrayList<>();
        list.add(new Person("A",10));
        list.add(new Person("B",11));
        list.add(new Person("C",12));
        list.add(new Person("D",13));
        list.add(new Person("E",14));
        list.add(new Person("F",15));

        list.sort((o1,o2)-> o2.age-o1.age);
        System.out.println(list);
```

通过Lambda表达式实现了Comparator接口。



### 1.7 综合

####  1.7.1 TreeSet排序

```java
    public static void main(String[] args) {
        //需求：按照年龄排序
        TreeSet<Person> set = new TreeSet<>((o1,o2)->{
            if(o1.age >= o2.age){
                return -1;
            }
            return 1;
        });
        set.add(new Person("A",10));
        set.add(new Person("B",11));
        set.add(new Person("C",12));
        set.add(new Person("D",13));
        set.add(new Person("E",14));
        set.add(new Person("F",15));

        System.out.println(set);
    }
```



#### 1.7.2 集合遍历

forEach：

```java
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>();
        Collections.addAll(list,1,2,3,4,5,6);

        list.forEach(ele->{
            if(ele % 2 == 0)
                System.out.println(ele);
        });
    }
```



#### 1.7.3 集合删除

```
public static void main(String[] args) {
    //需求：按照年龄排序
    ArrayList<Person> list = new ArrayList<>();
    list.add(new Person("A",10));
    list.add(new Person("B",11));
    list.add(new Person("C",12));
    list.add(new Person("D",13));
    list.add(new Person("E",14));
    list.add(new Person("F",15));

    list.removeIf(ele->ele.age<12);
    System.out.println(list);
}
```

#### 1.7.4 线程实例化

```java
new Thread(()->{
    System.out.println(Thread.currentThread().getName()); 
}).start();
```

实现了Runnable接口



### 1.8 系统内置函数式接口

```java
Predicate<T> : 参数T，返回值boolean
            IntPredicate    int->boolean
            LongPredicate    Long->boolean
            DoublePredicate    Double->boolean
        Comsumer<T> : 参数T， 返回值void
        Function<T,R> : 参数T， 返回值R
        Supplier<T> : 参数无， 返回值T
        UnaryOperator<T>: 参数T，返回值T
        BinaryOperator<T> : 参数T，T，返回值T
        BiFunction<T,U,R> : 参数T，U，返回值R
        BiPredicate<T,U> : 参数T，U，返回值boolean
        BiConsumer<T,U> : 参数T，U，返回值void
```

### 1.9 闭包

```java
    public static void main(String[] args) {
        int n = getNumber().get();
        System.out.println(n);
    }

    private static Supplier<Integer> getNumber(){
        int num = 10;
        return ()->num;
    }
```

输出n为10，看上去似乎没什么问题。



但是执行getNumber后，num变量应该直接销毁了，再执行get()时应该为null。实际上使用Lambda闭包，能够延长局部变量的生命周期。



备注：在闭包中引用的变量应该为常量，就算没有加final修饰，系统也会自动加final。



## 2. Stream流式编程

### 2.1 Stream流的创建

操作的数据源：集合、数组

产生 结果：新的流

将数据源转换成流，经过一系列中间操作，转换成新的流。

#### 2.1.1 什么是Stream

流式数据渠道，用于操作数据源（集合、数组等）所生产的元素序列。

<span style="color:Blue">集合讲的是数据，流讲的是计算</span>

<span style="color:red">注意：</span>

① Stream自己不会存储元素

② Stream不会改变源对象。相反，他们会返回一个持有结果的新Stream

③ Stream操作时延迟执行的。这意味着他们会等到需要结果的时候才执行。



#### 2.1.2 Stream创建的几种方式

```java
//1. 通过Collection系列集合提供的stream()或parallelStream()
        List<String> list = new ArrayList<>();
        Stream<String> stream = list.stream();

        //2. 通过Arrays中的静态方法stream()获取数组流
        Integer[] emps = new Integer[10];
        Stream<Integer> stream1 = Arrays.stream(emps);

        //3. 通过Stream中的静态方法of(可变参数)
        Stream<String> stream2 = Stream.of("aa", "bb", "cc");

        //4. 创建无限流
        //迭代
        Stream<Integer> stream4 = Stream.iterate(0, x -> x + 2);
        stream4.forEach(System.out::println);

        //生成
        Stream.generate(()->Math.random()).forEach(System.out::println);
```



### 2.2 中间操作

中间操作不会执行任何操作，只有当终止操作执行，才会一次性执行全部内容。这叫做“惰性求值”

#### 2.2.1筛选与切片

##### filter

接收Lambda，从流中排除某些元素。

```java
@Test
public void test01(){
    //中间操作，过滤掉年龄大于2的元素
    Stream<Person> personStream = list.stream().filter((e) -> e.age > 2);
    //终止操作
    personStream.forEach(System.out::println);
}
```



##### limit

截断流，使其元素不超过给定数量

```java
@Test
public void test02(){
    list.stream().filter(e->e.age < 4)
        		.limit(2)
        		.forEach(System.out::println);
}
```

此时Stream API内部迭代次数为2次，只要找到满足`limit` 数量的数据后，后续迭代操作无需进行。

##### skip(n)

跳过元素，返回一个扔掉了前n个元素的流。若流中元素不足n个	，则返回一个空流。与limit互补

```java
@Test
    public void test03(){
        list.stream().filter(e->e.age < 4)
            		.skip(2)
            		.limit(3)
            		.forEach(System.out::println);
    }
```



##### distinct

筛选，通过流所生产元素的**hashCode()**和**equals()**去除重复的元素

```java
@Test
public void test04(){
    list.stream().filter(e->e.age < 4)
        		.distinct()
        		.forEach(System.out::println);
}
```



#### 2.2.2映射

##### map

接收lambda，将元素转换成其他形式或提取信息。接收一个函数作为参数，该函数会被**应用到每个元素上**，并将其映射成一个新的元素。

```java
@Test
public void test05(){
    List<String> list = Arrays.asList("aa","bb","cc","dd","ee");

    list.stream()
            .map((str)->str.toUpperCase())
            .forEach(System.out::println);
}
```

##### flatMap

接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流

```java
@Test
public void test05(){
    List<String> list = Arrays.asList("aa","bb","cc","dd","ee");

    //每个字符串中的字符单独提取出来
    Stream<Character> characterStream = list.stream()
        .flatMap(StreamTest01::filterCharacter);
    characterStream.forEach(System.out::println);
}

public static Stream<Character> filterCharacter(String str){
    List<Character> list = new ArrayList<>();
    for(Character ch: str.toCharArray()){
        list.add(ch);
    }
    return list.stream();
}
```

#### 2.2.3 排序

##### sorted

自然排序

```java
@Test
public void test07(){
    List<String> list = Arrays.asList("aa","bb","cc","dd","ee");
    list.stream().sorted().forEach(System.out::println);
}
```

##### sorted(Comparator com)

定制排序

```java
@Test
public void test07(){
    list.stream().sorted((o1,o2)->{
        if(o1.age > o2.age) return 1;
        else return -1;
    }).forEach(System.out::println);
}
```



### 2.3 终止操作

#### 2.3.1 查找与匹配

##### allMatch

检查是否匹配所有元素

```java
@Test
public void test01(){
    boolean b1 = list.stream().allMatch(e -> e.getName().equals("E"));
    System.out.println(b1);
}
```

集合list内的元素的名字是否都是“E”。

##### anyMatch

检查是否至少匹配一个元素

```java
@Test
public void test01(){
    boolean a = list.stream().anyMatch(e -> e.getName().equals("A"));
    System.out.println(a);
}
```

集合list内的元素的名字是否有至少一个是“A”。

##### noneMatch

检查是否没有匹配所有元素

```java
@Test
public void test01(){
    boolean a = list.stream().noneMatch(e -> e.getName().equals("F"));
    System.out.println(a);
}
```

list中的元素的名字是否都不等于“F”

##### findFirst

返回第一个元素

```java
@Test
public void test01(){
    Optional<Person> first = list.stream().sorted(((o1, o2) -> Integer
                            .compare(o1.age, o2.age)))
                            .findFirst();
    System.out.println(first.get());
}
```

按年龄从小到大排序返回第一个元素。

由于可能为空，所以返回一个Optional类。

##### findAny

返回当前流中任意元素

```java
@Test
public void test01(){
    Optional<Person> any = list.stream()
            .filter(e -> e.getAge() == 6)
            .findAny();

    System.out.println(any);
}
```

##### count

返回流中元素的总个数

```java
@Test
public void test01(){
    long count = list.stream().count();
    System.out.println(count);
}
```

##### max

返回流中最大值

##### min

返回流中最小值



```java
@Test
public void test01(){
    Optional<Person> max = list.stream().max((e1, e2) -> Integer.compare(e1.getAge(), e2.getAge()));
    System.out.println(max.get());

    Optional<Integer> max1 = list.stream()
            .map(Person::getAge) //先把年龄都映射出来
            .max(Double::compare);  //取个min
    System.out.println(max1.get());
}
```



#### 2.3.2 归约

`reduce(T identity, BinaryOperator) / reduce(BinaryOperator)`

可以将流中元素反复结合起来，得到一个值

```java
@Test
public void test01(){
    List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9);

    Integer sum = list.stream()
            .reduce(0, (x, y) -> x + y);
    System.out.println(sum);
}
```

求集合元素内所有元素之和。

过程：初始值设定为0。 从集合中取一个值，先作为x，再取一个作为y，求和x+y，然后将x+y作为x，再取一个作为y，求和。以此类推。



#### 2.3.3 收集

collect是将流转换成其他形式，接收一个Collector接口的实现，用于给Stream中元素做汇总的方法。

```java
@Test
public void test01(){
    List<String> collect = list.stream()
            .map(Person::getName)
            .collect(Collectors.toList());

    System.out.println(collect);
}
```

提取出名字放到 list 中



```java
@Test
public void test01(){
    HashSet<String> collect = list.stream()
            .map(Person::getName)
            .collect(Collectors.toCollection(HashSet::new));

    System.out.println(collect);
}
```

收集到指定集合中 `Collectors.toCollection(XX)`



还有很多其它功能，有待学习。	



#### 2.3.4 并行流与串行流

##### 并行流

把一个内容分成多个数据块，并用不同线程分别处理每个数据块的流。

###### fork/join框架

将一个大任务分成（fork）若干个小任务，再将一个个小任务的运算结果汇总（join），类似分治。

若干个小任务的执行过程，就是并行。



```java
@Test
public void test01(){
    OptionalLong sum = LongStream.rangeClosed(0, 1000000000L)
            .parallel()
            .reduce(Long::sum);
    System.out.println(sum);
}
```

并行流`parallel()`计算 0+....+1000000000

##### 串行流

相当于单线程。



## 3. Optional容器类

Optional< T > 类 (java.util.Optional) 是一个容器类，代表一个值存在或不存在。 原来用null表示一个值不存在，现在Optional可以更好地表示这个概念。并且可以避免空指针异常。

### 3.1 常用方法

`Optional.of(T t)` : 创建一个Optional 实例

```java
@Test
public void test01(){
    Optional<Person> person = Optional.of(new Person());
    Person per = person.get();
    System.out.println(per);
}
```

`Optional.empty()`: 创建一个空的Optional实例

`Optional.ofNullable(T t)`: 若t 不为null，创建Optional实例，否则创建空实例。

`isPresent()`: 判断是否包含值

`orElse(T t)` 如果调用对象包含值，返回该值，否则返回 t

```java
@Test
public void test01(){
    Optional<Person> person = Optional.ofNullable(null);

    Person dd = person.orElse(new Person("dd", 50));
    System.out.println(dd);
}
```

如果person有值，那么输出person的值，否则输出orElse里的值。



`orElseGet(Supplier s)`: 如果调用对象包含值，返回该值，否则返回 s 获取的值。

`map(Function f)`: 如果有值对其处理，返回处理后的Optional，否则返回Optional.empty()

```java
@Test
public void test01(){
    Optional<Person> person = Optional.ofNullable(new Person("qqq",70));
    Optional<String> s = person.map(e -> e.getName());
    System.out.println(s.get());
}
```

`flatMap(Function mapper)`: 与map类似，要求返回值必须是Optional

```java
@Test
public void test01(){
    Optional<Person> person = Optional.ofNullable(new Person("qqq",70));
    Optional<String> s = person.flatMap(e -> Optional.of(e.getName()));
    System.out.println(s.get());
}
```



## 4. 接口的新特性

### 4.1 默认方法

java8 允许接口中定义有实现的方法，无需创建实现类去实现它。

### 4.2 静态方法

 java8 允许接口中定义有实现的静态方法，无需创建实现类去实现它。



```java
interface myInter{
    default void print() {
        System.out.println("默认方法");
    }

    static void print2(){
        System.out.println("静态方法");
    }
}
```

### 4.3 为什么要有这个特性

首先，之前的接口是个双刃剑，好处是面向抽象而不是面向具体编程，缺陷是，当需要修改接口时候，需要修改全部实现该接口的类，目前的java 8之前的集合框架没有foreach方法，通常能想到的解决办法是在JDK里给相关的接口添加新的方法及实现。然而，对于已经发布的版本，是没法在给接口添加新方法的同时不影响已有的实现。所以引进的默认方法。**他们的目的是为了解决接口的修改与现有的实现不兼容的问题。**

## 5. 新的时间日期API

### 5.1 传统日期API缺点

+ **非线程安全** − java.util.Date 是非线程安全的，所有的日期类都是可变的，这是Java日期类最大的问题之一。
+ **设计不规范** − Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。
+ **时区处理麻烦** − 日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题。

### 5.2 新的日期API

`LocalDate` 日期

`LocalTime` 时间

`LocalDateTime` 日期时间

都是不可变的，线程安全。

> 区别
>
> LocalDate 只能定义日期
>
> LocalTime 只能定义时间
>
> LocalDateTime 既能定义日期，也能定义时间

`Instant` 时间戳（以Unix元年：1970年1月1日 00:00:00到某个时间之间的毫秒值）。



### 5.3 使用



1 获取本地日期时间

```java
@Test
public void test01(){
    LocalDateTime ldt = LocalDateTime.now();
    System.out.println(ldt);
}
```

2 自定义日期时间

```java
@Test
public void test01(){
    LocalDateTime ldt2 = LocalDateTime.of(2020,12,12,23,59,59);
    System.out.println(ldt2);
}
```

3 日期加减

```java
@Test
public void test01(){
    LocalDateTime ldt2 = LocalDateTime.of(2020,12,12,23,59,59);
    LocalDateTime ldt3 = ldt2.plusYears(2);
    System.out.println(ldt3);

    LocalDateTime ldt4 = ldt2.minusYears(2);
    System.out.println(ldt4);
}
```

4 还有各种get方法

5 Instant

```java
@Test
public void test01(){
    Instant ins1 = Instant.now();
    System.out.println(ins1);

    OffsetDateTime odt = ins1.atOffset(ZoneOffset.ofHours(8));
    System.out.println(odt);

    //转换成毫秒
    System.out.println(ins1.toEpochMilli());
}
```

## 6. 重复注解与类型注解

