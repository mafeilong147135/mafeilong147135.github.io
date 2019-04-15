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

#### 1. 概述

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

#### 2. 实例

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

#### 3. 四种方法引用类型

方法引用的标准形式是：`类名::方法名`。（**注意：只需要写方法名，不需要写括号**）

有以下四种形式的方法引用：

| **类型**                         | **示例**                             |
| -------------------------------- | ------------------------------------ |
| 引用静态方法                     | ContainingClass::staticMethodName    |
| 引用某个对象的实例方法           | containingObject::instanceMethodName |
| 引用某个类型的任意对象的实例方法 | ContainingType::methodName           |
| 引用构造方法                     | ClassName::new                       |

 

 

 

 

 