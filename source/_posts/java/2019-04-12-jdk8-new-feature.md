---
title: jdk8 新特性
date: 2019-04-12 13:41:24
tags: java
categories: java
toc: true
summary: jdk8相比之前版本有了新的特性
top: true
typora-root-url: 2019-04-12-jdk8-new-feature
---

## 1.  接口的默认方法和静态方法

>​        jdk8新增的接口默认方法，接口可以有默认的实现方法。而不需要接口的实现类去实现该方法。我们只需要在方法前加上default关键字就可以了。

### 1. 接口默认方法

#### 1. 语法

```java
  public interface DefaultInterface {
    default void print() {
      System.out.println("default method");
    }
  }
```

#### 2. 为什么需要该特性？

```java
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
}
```

>​        接口的好处是面向抽象而不是面向具体实现编程。但是如果我们在已有的接口新增接口方法，那么所有的实现类都需要去新增该方法的实现。比如上面的集合源码的foreach方法，在 jdk 8 之前的集合类中没有该方法。如果我们在该接口新增实现方法的话，就会改动之前版本的所有实现类。所以该特性是为了解决接口的修改和现有实现不兼容的情况出现。

#### 3. 多个接口默认方法

​        在一个接口里面我们可以定义多个接口的默认方法。同时在接口的实现类我们可以不需要再去实现该方法。但是会继承该方法。我们也可以在实现类中去重写该方法。如果一个实现类C去实现了两个接口A、B，B接口继承了A接口。AB接口中都有同一个默认方法。实现类C所继承的默认方法会是最具体的那一个，也就是B接口的默认实现方法。

```java
interface DefaultInterfaceA {
  default void print1() {
    System.out.println("default method1 A");
  }
  default void print2() {
    System.out.println("default method2 A");
  }
}
interface DefaultInterfaceB extends DefaultInterfaceA {
  default void print1() {
    System.out.println("default method1 B");
  }
  default void print2() {
    System.out.println("default method2 B");
  }
}
class ClassC implements DefaultInterfaceA, DefaultInterfaceB {

  public static void main(String...args) {
    ClassC classC = new ClassC();
    classC.print1(); // default method1 B
    classC.print2(); // default method2 B
  }
}
```

#### 4. 接口不能提供对Object类的任何方法的默认实现

​        每个类都默认是Object类的子类，也都继承了Object类的equals()、hashCode()、toString()等方法。根据类优先原则，在接口中定义Object类的方法是没有意思的。也不会被编译。

### 2. 静态方法

​        jdk 8 可以申明一个和多个静态方法。但只能通过接口类调用接口中的静态方法。

```java
public class TestStaticMethod {
  public static void main(String...args) {
    DefaultStaticInterface.print(); // static method
  }
}
interface DefaultStaticInterface {
  static void print() {
    System.out.println("static method");
  }
  static void print1() {
    System.out.println("static method");
  }
}
```

## 2. lambda表达式

​        lambda表达式（也可称为闭包），是jdk8最重要的特性。lambda表达式允许把函数作为一个方法的参数。函数可作为参数传递进方法。使用lambda表达式可以使代码变得紧凑。

### 1. 语法

​       一个`Lambda`表达式可以由用逗号分隔的参数列表、`–>`符号与函数体三部分表示。

```java
(parameters) -> expression;
(parameters) -> {expressions;}
```

​      **lambda的特征:**

- **可选类型申明**：  可以不写出参数的类型，编译器会自动的识别参数类型。
- **可选的参数括号**：当只有一个参数的时候可以不写出参数括号。
- **可选的执行语句大括号**：当只有一句执行语句时，可以不需要大括号。
- **可选的返回关键字**：当只有一句执行语句同时没有大括号的时候，可以不加上返回的关键字。编译器会自动的返回值。

### 2.Lambda 表达式实例

我们可以看一下两种不同的创建TreeSet的定制排序方式。

```java
    Set<Integer> oldTreeSet = new TreeSet<>(new Comparator<Integer>() {
      @Override
      public int compare(Integer o1, Integer o2) {
        return o2 - o1;
      }
    });
    oldTreeSet.add(1);
    oldTreeSet.add(3);
    oldTreeSet.add(2);
    oldTreeSet.add(4);
    System.out.println("oldTreeSet:" + oldTreeSet); // oldTreeSet:[4, 3, 2, 1]
    Set<Integer> newTreeSet = new TreeSet<>((Integer o1, Integer o2) -> {return o2 -o1;});
    newTreeSet.add(1);
    newTreeSet.add(3);
    newTreeSet.add(2);
    newTreeSet.add(4);
    System.out.println("newTreeSet:" + newTreeSet); // newTreeSet:[4, 3, 2, 1]
  }
```

其实在使用lambda表达式还可以更加的简洁。当只有一句执行语句时，可以不需要大括号。

```java
Set<Integer> newTreeSet = new TreeSet<>((Integer o1, Integer o2) ->  o2 -o1);
```

同时编译器还可自动推导出参数的类型，我们还可以这样写。

```java
Set<Integer> newTreeSet = new TreeSet<>((o1, o2) ->  o2 -o1);
```

### 3.lambda只能引用final标记的外层局部变量

```java
class TestLambdaFinal {
  public static void main(String args[]) {
    final int num = 1; 
    Multiply<Integer, Integer> s = (param) -> System.out.println(String.valueOf(param * num));
    s.multiply(2);  // 输出结果为 2
  }
}
interface Multiply<T1, T2> {
  void multiply(Integer i);
}
```

 上面的代码如果我们去掉final的话，也是可以执行的，但是num被默认成的是final的变量。在之后的代码中就不能带num的值进行修改了。如果修改就会编译不通过。报Variable used in lambda expression should be final or effectively final的错误提示。

```java
class TestLambdaFinal {
  public static void main(String args[]) {
    int num = 1;
    Multiply<Integer, Integer> s = (param) -> System.out.println(String.valueOf(param * num));
    s.multiply(2);
    num ++;
    // Variable used in lambda expression should be final or effectively final
  }
}
interface Multiply<T1, T2> {
  void multiply(Integer i);
}
```

同时在 Lambda 表达式当中不允许声明一个与局部变量同名的参数或者局部变量。

```java
String param = 1;
int num = 1;
Multiply<Integer, Integer> s = (param) -> System.out.println(String.valueOf(param + num)); 
//编译会出错
```

## 3. 函数式接口

​      `Lambda`表达式是如何在Java的类型系统中表示的呢？每一个Lambda表达式都对应一个类型，通常是接口类型。函数式接口就是只有一个抽象方法但是可以有多个默认实现方法的接口。如果不写上@FunctionalInterface也同样可以。JDK 1.8 新增加的函数接口：java.util.function 它包含了很多类，用来支持 Java的 函数式编程。

```java
public class TestFunctionalInterface {
  public static void main(String args[]) {
    FunctionalInterfaceA<Integer, Integer, Integer> functionalInterfaceA = (i, m) -> i + m;
    Integer add = functionalInterfaceA.add(2, 2);
    System.out.println(multiply); // 4
  }
}
@FunctionalInterface
interface FunctionalInterfaceA<T1, T2,  T3> {
  T3 multiply(T1 i, T2 m);
  boolean equals(Object i); // 是Object的public方法，不会编译所以没有报错
  default void print() {
    System.out.println("default method");
  }
  default void print1() {
    System.out.println("default method1");
  }
}
```

## 4. 方法引用

### 1. 概述

方法引用是通过方法的名字来指向一个方法。它可以使代码更加紧凑，减少代码冗余。

```java
List<String> list = Arrays.asList("ada","123123","cfsd");
list.forEach(item -> System.out.println(item));
```

在Java8中，我们可以直接通过方法引用来**简写**lambda表达式中已经存在的方法。

```java
List<String> list = Arrays.asList("ada","123123","cfsd");
list.forEach(System.out::println);
```

>简单地说，方法引用就是一个Lambda表达式。在Java 8中，我们会使用Lambda表达式创建匿名方法，但是有时候，我们的Lambda表达式可能仅仅调用一个已存在的方法，而不做任何其它事，对于这种情况，通过一个方法名字来引用这个已存在的方法会更加清晰，Java 8的方法引用允许我们这样做。方法引用是一个更加紧凑，易读的Lambda表达式，注意方法引用是一个Lambda表达式，其中方法引用的操作符是双冒号"::"。

### 2. 实例

```java
public class Person{

  private String name;

  private Integer age;

  public Person(String name, Integer age) {
    this.name = name;
    this.age = age;
  }

  public Integer getAge() {
    return age;
  }

  public static int compareByAge(Person a, Person b) {
    return a.age.compareTo(b.age);
  }

  @Override
  public String toString() {
    return this.name;
  }
}
```

```java
public class TestMethodReference {
  public static void main(String args[]) {
    //test();
    Person[] pers = new Person[] {
        new Person("mfl", 27),
        new Person("zym", 29),
        new Person("123", 28),
    };
    // 使用匿名内部类
    Arrays.sort(pers, new Comparator<Person>() {
      @Override
      public int compare(Person o1, Person o2) {
        return o1.getAge().compareTo(o2.getAge());
      }
    });
    System.out.println(Arrays.asList(pers));
    // 使用lambda表达式和静态方法
    Arrays.sort(pers, (o1, o2) -> Person.compareByAge(o1, o2));
    System.out.println(Arrays.asList(pers));
    // 使用方法引用
    Arrays.sort(pers, Person::compareByAge);
    System.out.println(Arrays.asList(pers));
  }   
}
```

### 3. 四种方法引用类型

方法引用的标准形式是：`类名::方法名`。（**注意：只需要写方法名，不需要写括号**）

有以下四种形式的方法引用：

| **类型**                         | **示例**                             |
| -------------------------------- | ------------------------------------ |
| 引用静态方法                     | ContainingClass::staticMethodName    |
| 引用某个对象的实例方法           | containingObject::instanceMethodName |
| 引用某个类型的任意对象的实例方法 | ContainingType::methodName           |
| 引用构造方法                     | ClassName::new                       |

## 5. Stream

> jdk8 新增的Stream API 将生成环境的函数式编程引入了java库中，是目前为止对java最大的补充，使开发者写出更有效，更加简洁、紧凑的代码。Stream API 极大简化了对集合的操作。同时又不只是集合。

### 1. 部分API

#### 1. 生成流

在 Java 8 中, 集合接口有两个方法来生成流：**stream() 和 parallelStream()** 

#### 2. map

map 方法用于映射每个元素到对应的结果，以下代码片段使用 map 输出了元素对应的平方数：

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5); // 获取对应的平方数 
List<Integer> squaresList = numbers.stream()
    .map(i -> i*i).distinct()
    .collect(Collectors.toList());
```

#### 4. filter

filter 方法用于通过设置的条件过滤出元素。以下代码片段使用 filter 方法过滤出空字符串：

```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); 
// 获取空字符串的数量 
int count = strings.stream().filter(string -> string.isEmpty()).count();
```

#### 5. limit

limit 方法用于获取指定数量的流。 以下代码片段使用 limit 方法打印出 10 条数据：

```java
Random random = new Random(); 
random.ints().limit(10).forEach(System.out::println); // 不加上的话会一直打印下去
```

#### 6. sorted

sorted 方法用于对流进行排序。以下代码片段使用 sorted 方法对输出的 10 个随机数进行排序：

```java
Random random = new Random();
random.ints().limit(10).sorted().forEach(System.out::println);
```

#### 7. 并行（parallel）程序

parallelStream 是流并行处理程序的代替方法。以下实例我们使用 parallelStream 来输出空字符串的数量：

```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); // 获取空字符串的数量 
int count = strings.parallelStream()
    .filter(string -> string.isEmpty())
    .count();
```

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
numbers.parallelStream().forEach(out::println);
```

这里得到的并不一定是有序的，有可能是1,3,4,5,2,6,8,9,7。就forEach()这个操作来讲，如果平行处理时，希望最后顺序是按照原来Stream的数据顺序，那可以调用forEachOrdered()。例如：

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
numbers.parallelStream().forEachOrdered(out::println);
```

>**注意:**如果forEachOrdered()中间有其他如filter()的中介操作，会试着平行化处理，然后最终forEachOrdered()会以原数据顺序处理，因此，使用forEachOrdered()这类的有序处理,可能会（或完全失去）失去平行化的一些优势，实际上中介操作亦有可能如此，例如sorted()方法。

#### 8. Collectors

Collectors 类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors 可用于返回列表或字符串：

```java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); 
List<String> filtered = strings.stream()
    .filter(string -> !string.isEmpty())
    .collect(Collectors.toList());   
System.out.println("筛选列表: " + filtered); // 筛选列表: [abc, bc, efg, abcd, jkl]

String mergedString = strings.stream()
    .filter(string -> !string.isEmpty())
    .collect(Collectors.joining(", ")); 
System.out.println("合并字符串: " + mergedString); // 合并字符串: abc, bc, efg, abcd, jkl
```

#### 9. 统计

另外，一些产生统计结果的收集器也非常有用。它们主要用于int、double、long等基本类型上，它们可以用来产生类似如下的统计结果。

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);   
IntSummaryStatistics stats = numbers.stream()
    .mapToInt((x) -> x)
    .summaryStatistics();
System.out.println("列表中最大的数 : " + stats.getMax()); 
System.out.println("列表中最小的数 : " + stats.getMin()); 
System.out.println("所有数之和 : " + stats.getSum()); 
System.out.println("平均数 : " + stats.getAverage());
```

### 2. 简单示例

现在我们有一个叫TestStream的类和一个Status的枚举类。

```java
public class TestStream {

  private Status status;

  private Integer point;

  public TestStream(Status status, Integer point) {
    this.status = status;
    this.point = point;
  }

  public TestStream() {
  }

  public Status getStatus() {
    return status;
  }

  public void setStatus(Status status) {
    this.status = status;
  }

  public Integer getPoint() {
    return point;
  }

  public void setPoint(Integer point) {
    this.point = point;
  }

  @Override
  public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    TestStream that = (TestStream) o;
    return status == that.status &&
            Objects.equals(point, that.point);
  }

  @Override
  public int hashCode() {
    return Objects.hash(status, point);
  }

  @Override
  public String toString() {
    return "TestStream{" +
            "status=" + status +
            ", point=" + point +
            '}';
  }
}

enum Status {
  OPEN,CLOSE
}
```

 现在假设我们有个该类的集合。

```java
Collection<TestStream> testStreams = Arrays.asList(new TestStream(Status.OPEN, 13),
     new TestStream(Status.OPEN, 8),
     new TestStream(Status.CLOSE, 12));
```

​      现在我们要看OPEN状态的点数和为多少？在jdk8之前我们需要foreach循环testStreams集合。在循环的时候加上if判断。以前的写法。

```java
int sum = 0;
for (TestStream testStream : testStreams) {
   if (Status.OPEN == testStream.getStatus()) {
      sum += testStream.getPoint();
   }
}
System.out.println(sum); // 21
```

但是在Java 8中可以利用steams解决：包括一系列元素的列表，并且支持顺序和并行处理。

```java
int total = testStreams.stream()
            .filter(item -> item.getStatus() == Status.OPEN)
            .mapToInt(TestStream::getPoint)
            .sum();
System.out.println(total); // 21
```

​       在这几步中，首先，testStreams集合被转换成steam表示；其次，在steam上的**filter**操作会过滤掉所有CLOSED的testStream；第三，**mapToInt**操作基于每个testStream实例的**TestStream::getPoint**方法将task流转换成IntStream；最后，通过**sum**方法计算总和，得出最后的结果。

​        Steam之上的操作可分为中间操作和晚期操作。中间操作会返回一个新的Steam——执行一个中间操作（例如**filter**）并不会执行实际的过滤操作，而是创建一个新的Steam，并将原Steam中符合条件的元素放入一个新创建的Steam。晚期操作（例如**forEach**或者**sum**或者**max**），会遍历Steam并得出结果或者附带结果；在执行晚期操作之后，Steam处理线已经处理完毕，就不能使用了。在几乎所有情况下，晚期操作都是立刻对Steam进行遍历。

​       在这里还有两个例子，如何计算集合中TestStream的点数在集合中所占的比重，具体处理的代码如下：

```java
Collection<String> result = testStreams.stream()
            .mapToDouble(testStream -> testStream.getPoint())
            .map(point -> point / totalPoint)
            .map(weigth -> (long) (weigth * 100))
            .mapToObj(item -> item + "%")
            .collect(Collectors.toList());
System.out.println(result); // [39.0%, 24.0%, 36.0%]
```

​        对于一个集合，经常需要根据某些条件对其中的元素分组。利用steam提供的API可以很快完成这类任务，代码如下：

```java
Map<Status, List<TestStream>> map = testStreams.stream()
            .collect(Collectors.groupingBy(TestStream::getStatus));
System.out.println(map);
// {OPEN=[TestStream{status=OPEN, point=13}, TestStream{status=OPEN, point=8}], CLOSE=[TestStream{status=CLOSE, point=12}]}
```

## 6. optional

> Java应用中最常见的bug就是空值异常。在Java 8之前，[Google Guava](http://code.google.com/p/guava-libraries/)引入了**Optionals**类来解决**NullPointerException**，从而避免源码被各种**null**检查污染，以便开发者写出更加整洁的代码。Java 8也将**Optional**加入了官方库。**Optional** 是个容器。它可以保存类型T的值或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。Optional 类的引入很好的解决空指针异常。

```java
public static void main(String[] args) {
    Integer nullValue = null;
    Integer value = 10;
    Optional<Integer> optionalInteger = Optional.ofNullable(nullValue); // 进行了是否为空的判断
    Optional<Integer> optional = Optional.of(value); // 如果value为空就会报空指针异常
    System.out.println("和：" + sum(optionalInteger, optional)); // 和：10
}
public static int sum(Optional a, Optional b) {
    System.out.println("a参数不为空:" + a.isPresent()); // a参数不为空:false
    System.out.println("b参数不为空:" + b.isPresent()); // b参数不为空:true
    Integer value1 = (Integer) a.orElseGet(() -> new Integer(0)); // 当a为空时返回回调函数返回的值
    Integer value3 = (Integer) a.orElse(new Integer(0)); // 当a为空时返回设置的值
    Integer value2 = (Integer) b.get(); //值不能为空 否则报 new NoSuchElementException("No value present");
    return value1 + value2;
}
```

​       如果`Optional`类的实例为非空值的话，`isPresent()`返回`true`，否从返回`false`。为了防止Optional为空值，`orElseGet()`方法通过回调函数来产生一个默认值。`map()`函数对当前`Optional`的值进行转化，然后返回一个新的`Optional`实例。`orElse()`方法和`orElseGet()`方法类似，但是`orElse`接受一个默认值而不是一个回调函数。

## 7. 重复注解

元注解是指注解的注解，包括@Retention @Target @Document @Inherited四种。

1. @Retention: 定义注解的保留策略

```java
@Retention(RetentionPolicy.SOURCE)   //注解仅存在于源码中，在class字节码文件中不包含
@Retention(RetentionPolicy.CLASS)    // 默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得，
@Retention(RetentionPolicy.RUNTIME)  // 注解会在class字节码文件中存在，在运行时可以通过反射获取到
```

​        首先要明确生命周期长度 SOURCE < CLASS < RUNTIME ，所以前者能作用的地方后者一定也能作用。一般如果需要在运行时去动态获取注解信息，那只能用 RUNTIME 注解；如果要在编译时进行一些预处理操作，比如生成一些辅助代码，就用 CLASS注解；如果只是做一些检查性的操作，比如 @Override 和 @SuppressWarnings，则可选用 SOURCE 注解。

2. @Target：定义注解的作用目标

```java
@Target(ElementType.TYPE)   //接口、类、枚举、注解
@Target(ElementType.FIELD) //字段、枚举的常量
@Target(ElementType.METHOD) //方法
@Target(ElementType.PARAMETER) //方法参数
@Target(ElementType.CONSTRUCTOR)  //构造函数
@Target(ElementType.LOCAL_VARIABLE)//局部变量
@Target(ElementType.ANNOTATION_TYPE)//注解
@Target(ElementType.PACKAGE) //包
```

3. @Document：说明该注解将被包含在javadoc中

4. @Inherited：说明子类可以继承父类中的该注解

> 重复注解机制本身必须用`@Repeatable`注解。事实上，这并不是语言层面上的改变，更多的是编译器的技巧，底层的原理保持不变。让我们看一个快速入门的例子：

```java
public class TestRepeatingAnnotations {
  public static void main(String...args) {
    Num[] annotationsByType = Numable.class.getAnnotationsByType(Num.class);
    for (int i = 0; i< annotationsByType.length; i++) {
      System.out.print(annotationsByType[i]); // @Num(value=10) @Num(value=21)
    }
  }
}
@Target(ElementType.TYPE) 
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Repeatable(Nums.class)
@interface Num {
  int value() default 10;
}
@Target(ElementType.TYPE) 
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@interface Nums {
  Num[] value();
}
@Num
@Num(value = 21)
interface Numable {
}
```

​       正如我们所见，这里的**Num**类使用@Repeatable(Num.class)注解修饰，而**Nums**是存放**Num**注解的容器，编译器尽量对开发者屏蔽这些细节。这样，**Numable**接口可以用两个**Num**注解注释（这里并没有提到任何关于Nums的信息）。另外，反射API提供了一个新的方法：**getAnnotationsByType()**，可以返回某个类型的重复注解，例如`Filterable.class.getAnnoation(Num.class)`将返回两个Num实例。

## 8. 新的时间API

> Java 8通过发布新的Date-Time API (JSR 310)来进一步加强对日期与时间的处理。
>
> 在旧版的 Java 中，日期时间 API 存在诸多问题，其中有：
>
> - **非线程安全** − java.util.Date 是非线程安全的，所有的日期类都是可变的，这是Java日期类最大的问题之一。
> - **设计很差** − Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。
> - **时区处理麻烦** − 日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题。
>
> Java 8 在 **java.time** 包下提供了很多新的 API。以下为两个比较重要的 API：
>
> - **Local(本地)** − 简化了日期时间的处理，没有时区的问题。
> - **Zoned(时区)** − 通过制定的时区处理日期时间。
>
> 新的java.time包涵盖了所有处理日期，时间，日期/时间，时区，时刻（instants），过程（during）与时钟（clock）的操作。

### 1. Clock 

**Clock**类使用时区来返回当前的纳秒时间和日期。**Clock**可以替代**System.currentTimeMillis()**和**TimeZone.getDefault()**。

```java
Clock clock = Clock.systemDefaultZone();
System.out.println(clock.instant()); // 获得时间 2019-04-17T06:46:19.513Z
System.out.println(clock.getZone()); // 获得当前时区 Asia/Shanghai
System.out.println(clock.millis()); // 获得时间戳 1555483579543
```

### 2. TimeZones

​        在新API中时区使用`ZoneId`来表示。时区可以使用静态方法`of`来获取。时区定义了到UTS时间的时间差，在`Instant`时间点对象到本地日期对象之间转换的时候是极其重要的。代码如下:

```java
System.out.println(ZoneId.getAvailableZoneIds()); // 得到所有可用的时区
ZoneId zoneId1 = ZoneId.of("Europe/Berlin");// 指定时区
ZoneId zoneId2 = ZoneId.of("Brazil/East"); // 指定时区
System.out.println(zoneId1.getRules()); // ZoneRules[currentStandardOffset=+01:00]
System.out.println(zoneId2.getRules()); // ZoneRules[currentStandardOffset=-03:00]
```

### 3. LocalTime

​     **LocalTime**则仅仅包含该日历系统中的时间部分。可以得到该时区额本地时间，还可以计算两个时区的时间差。LocalTime提供了多种工厂方法来简化对象的创建，包括解析时间字符串。代码如下:

```java
ZoneId zoneId1 = ZoneId.of("Europe/Berlin");
ZoneId zoneId2 = ZoneId.of("Brazil/East");
LocalTime now1 = LocalTime.now(zoneId1);
LocalTime now2 = LocalTime.now(zoneId2);
System.out.println(now1.isBefore(now2));  // false
long hoursBetween = ChronoUnit.HOURS.between(now1, now2);
long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);
System.out.println(hoursBetween);       // -4
System.out.println(minutesBetween);     // -299
LocalTime localTime = LocalTime.now();
System.out.println(localTime); // 当前本地时间

LocalTime localTime1 = LocalTime.of(13, 32, 32);
System.out.println(localTime1); // 13:32:32
```

### 4. LocalDate

   **LocalDate**仅仅包含ISO-8601日历系统中的日期部分。该对象值是不可变的，用起来和`LocalTime`基本一致。

```java
LocalDate today = LocalDate.now();
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
LocalDate yesterday = today.minusDays(1);

LocalDate independenceDay = LocalDate.of(2019, Month.APRIL, 17);
DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
System.out.println(dayOfWeek);    // WEDNESDAY

LocalDate localDate = LocalDate.of(2019, 4, 17);
System.out.println(localDate); // 2019-04-17

DateTimeFormatter germanFormatter = DateTimeFormatter
            .ofLocalizedDate(FormatStyle.MEDIUM)
    .withLocale(Locale.GERMAN);
LocalDate xmas = LocalDate.parse("17.04.2019", germanFormatter);
System.out.println(xmas);   // 2019-04-17
```

### 5.LocalDateTime

`LocalDateTime`同时表示了时间和日期，相当于前两节内容合并到一个对象上了。`LocalDateTime`和`LocalTime`还有`LocalDate`一样，都是不可变的。`LocalDateTime`提供了一些能访问具体字段的方法。代码如下:

```java
LocalDateTime localDateTime = LocalDateTime.of(2019, Month.APRIL, 17, 16, 59, 59);
DayOfWeek dayOfWeek = localDateTime.getDayOfWeek();
System.out.println(dayOfWeek);      // WEDNESDAY
Month month = localDateTime.getMonth();
System.out.println(month);          // APRIL
```

只要附加上时区信息，就可以将其转换为一个时间点`Instant`对象，`Instant`时间点对象可以很容易的转换为老式的`java.util.Date`。代码如下:

```java
Instant instant = localDateTime.atZone(ZoneId.systemDefault()).toInstant();
Date legacyDate = Date.from(instant);
System.out.println(legacyDate);     // Wed Apr 17 16:59:59 CST 2019
```

