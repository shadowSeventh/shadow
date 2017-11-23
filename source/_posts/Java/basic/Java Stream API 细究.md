---
title: Java Stream API 细究
date: 2017-11-21 10:33:22
categories: Java
tags: [Java]
description: 昨天突然看到一篇关于JDK1.8 Stream 讲解的文章，之前一直是简单的使用，并没有仔细分析过其中的细节。仅仅做到了知其然，看了那位大牛的博客后，打算花一些时间，尽力做到知其所以然。
---

## Lambda表达式和匿名内部类

### 取代某些匿名内部类
Java Lambda表达式的一个重要用法是简化某些匿名内部类（`Anonymous Classes`）的写法。实际上Lambda表达式并不仅仅是匿名内部类的语法糖，JVM内部是通过invokedynamic指令来实现Lambda表达式的。
#### demo1: 无参函数的简写
如果需要新建一个线程，一种常见的写法是这样：
```java
// JDK7 匿名内部类写法
new Thread(new Runnable(){// 接口名
    @Override
    public void run(){// 方法名
        System.out.println("Thread run()");
    }
}).start();
```
上述代码给`Tread`类传递了一个匿名的`Runnable`对象，重载`Runnable`接口的`run()`方法来实现相应逻辑。这是JDK7以及之前的常见写法。匿名内部类省去了为类起名字的烦恼，但还是不够简化，在Java 8中可以简化为如下形式：
```java
// JDK8 Lambda表达式代码块写法
new Thread(
        () -> {
            System.out.print("Hello");
            System.out.println(" Hoolee");
        }
).start();
```
#### demo2: 带参函数的简写
如果要给一个字符串列表通过自定义比较器，按照字符串长度进行排序，Java 7的书写形式如下：
```java
// JDK7 匿名内部类写法
List<String> list = Arrays.asList("I", "love", "you", "too");
Collections.sort(list, new Comparator<String>(){// 接口名
    @Override
    public int compare(String s1, String s2){// 方法名
        if(s1 == null)
            return -1;
        if(s2 == null)
            return 1;
        return s1.length()-s2.length();
    }
});
```
上述代码通过内部类重载了`Comparator`接口的`compare()`方法，实现比较逻辑。采用Lambda表达式可简写如下：
```java
// JDK8 Lambda表达式写法
List<String> list = Arrays.asList("I", "love", "you", "too");
Collections.sort(list, (s1, s2) ->{// 省略参数表的类型
    if(s1 == null)
        return -1;
    if(s2 == null)
        return 1;
    return s1.length()-s2.length();
});
```
上述代码跟匿名内部类的作用是一样的。除了省略了接口名和方法名，代码中把参数表的类型也省略了。这得益于javac的**类型推断机制**，编译器能够根据上下文信息推断出参数的类型，当然也有推断失败的时候，这时就需要手动指明参数类型了。注意，Java是强类型语言，每个变量和对象都必需有明确的类型。

### 简写的依据
也许你已经想到了，能够使用**Lambda**的依据是必须有相应的函数接口（函数接口，是指内部只有一个抽象方法的接口）。这一点跟Java是强类型语言吻合，也就是说你并不能在代码的任何地方任性的写Lambda表达式。实际上**Lambda**的类型就是对应函数接口的类型。Lambda表达式另一个依据是类型推断机制，在上下文信息足够的情况下，编译器可以推断出参数表的类型，而不需要显式指名。**Lambda**表达更多合法的书写形式如下：
```java
// Lambda表达式的书写形式
Runnable run = () -> System.out.println("Hello World");// 1
ActionListener listener = event -> System.out.println("button clicked");// 2
Runnable multiLine = () -> {// 3 代码块
    System.out.print("Hello");
    System.out.println(" Hoolee");
};
BinaryOperator<Long> add = (Long x, Long y) -> x + y;// 4
BinaryOperator<Long> addImplicit = (x, y) -> x + y;// 5 类型推断
```
上述代码中，1展示了无参函数的简写；2处展示了有参函数的简写，以及类型推断机制；3是代码块的写法；4和5再次展示了类型推断机制。

### 自定义函数接口
自定义函数接口很容易，只需要编写一个只有一个抽象方法的接口即可。
```java
// 自定义函数接口
@FunctionalInterface
public interface ConsumerInterface<T>{
    void accept(T t);
}
```
上面代码中的**@FunctionalInterface**是可选的，但加上该标注编译器会帮你检查接口是否符合函数接口规范。**就像加入@Override标注会检查是否重载了函数一样。**
有了上述接口定义，就可以写出类似如下的代码：

`ConsumerInterface<String> consumer = str -> System.out.println(str);`

进一步的，还可以这样使用：
```java
class MyStream<T>{
    private List<T> list;
    ...
    public void myForEach(ConsumerInterface<T> consumer){// 1
        for(T t : list){
            consumer.accept(t);
        }
    }
}
MyStream<String> stream = new MyStream<String>();
stream.myForEach(str -> System.out.println(str));// 使用自定义函数接口书写Lambda表达式
```
### 不是匿名内部类的简写
经过上面的介绍，我们看到Lambda表达式似乎只是为了简化匿名内部类书写，这看起来仅仅通过语法糖在编译阶段把所有的Lambda表达式替换成匿名内部类就可以了。但实时并非如此。在JVM层面，Lambda表达式和匿名内部类有着明显的差别。
#### 匿名内部类实现
匿名内部类仍然是一个类，只是不需要程序员显示指定类名，编译器会自动为该类取名。因此如果有如下形式的代码，编译之后将会产生两个class文件：
```java
public class MainAnonymousClass {
	public static void main(String[] args) {
		new Thread(new Runnable(){
			@Override
			public void run(){
				System.out.println("Anonymous Class Thread run()");
			}
		}).start();;
	}
}
```
进一步分析主类MainAnonymousClass.class的字节码，可发现其创建了匿名内部类的对象：
```java
// javap -c MainAnonymousClass.class
public class MainAnonymousClass {
  ...
  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class java/lang/Thread
       3: dup
       4: new           #3                  // class MainAnonymousClass$1 /*创建内部类对象*/
       7: dup
       8: invokespecial #4                  // Method MainAnonymousClass$1."<init>":()V
      11: invokespecial #5                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
      14: invokevirtual #6                  // Method java/lang/Thread.start:()V
      17: return
}
```
#### Lambda表达式实现
**Lambda**表达式通过**invokedynamic**指令实现，书写**Lambda**表达式不会产生新的类。如果有如下代码，编译之后只有一个class文件：
```java
public class MainLambda {
	public static void main(String[] args) {
		new Thread(
				() -> System.out.println("Lambda Thread run()")
			).start();;
	}
}
```
通过javap反编译命名，我们更能看出Lambda表达式内部表示的不同：
```java
// javap -c -p MainLambda.class
public class MainLambda {
  ...
  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class java/lang/Thread
       3: dup
       4: invokedynamic #3,  0              // InvokeDynamic #0:run:()Ljava/lang/Runnable; /*使用invokedynamic指令调用*/
       9: invokespecial #4                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
      12: invokevirtual #5                  // Method java/lang/Thread.start:()V
      15: return

  private static void lambda$main$0();  /*Lambda表达式被封装成主类的私有方法*/
    Code:
       0: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: ldc           #7                  // String Lambda Thread run()
       5: invokevirtual #8                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: return
}
```
反编译之后我们发现Lambda表达式被封装成了主类的一个私有方法，并通过**invokedynamic**指令进行调用。
#### 推论，this引用的意义
既然Lambda表达式不是内部类的简写，那么Lambda内部的this引用也就跟内部类对象没什么关系了。在Lambda表达式中this的意义跟在表达式外部完全一样。因此下列代码将输出两遍Hello Hoolee，而不是两个引用地址。
```java
public class Hello {
	Runnable r1 = () -> { System.out.println(this); };
	Runnable r2 = () -> { System.out.println(toString()); };
	public static void main(String[] args) {
		new Hello().r1.run();
		new Hello().r2.run();
	}
	public String toString() { return "Hello Hoolee"; }
}
```
## Lambda表达式和Java集合框架
Java8为容器新增一些有用的方法，这些方法有些是为完善原有功能，有些是为引入函数式编程（Lambda表达式），学习和使用这些方法有助于我们写出更加简洁有效的代码．本文分别以ArrayList和HashMap为例，讲解Java8集合框架（Java Collections Framework）中新加入方法的使用．
### Collection中的新方法
#### forEach()
该方法的签名为`void forEach(Consumer<? super E> action)`，作用是对容器中的每个元素执行`action`指定的动作，其中`Consumer`是个函数接口，里面只有一个待实现方法`void accept(T t)`（后面我们会看到，这个方法叫什么根本不重要，你甚至不需要记忆它的名字）。

需求：假设有一个字符串列表，需要打印出其中所有长度大于3的字符串.

Java7及以前我们可以用增强的for循环实现：
```java
// 使用曾强for循环迭代
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
for(String str : list){
    if(str.length()>3)
        System.out.println(str);
}
```
现在使用`forEach()`方法结合匿名内部类，可以这样实现：
```java
// 使用forEach()结合匿名内部类迭代
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.forEach(new Consumer<String>(){
    @Override
    public void accept(String str){
        if(str.length()>3)
            System.out.println(str);
    }
});
```
上述代码调用`forEach()`方法，并使用匿名内部类实现`Comsumer`接口。到目前为止我们没看到这种设计有什么好处，但是不要忘记Lambda表达式，使用Lambda表达式实现如下：
```java
// 使用forEach()结合Lambda表达式迭代
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.forEach( str -> {
        if(str.length()>3)
            System.out.println(str);
    });
```
上述代码给`forEach()`方法传入一个Lambda表达式，我们不需要知道`accept()`方法，也不需要知道`Consumer`接口，类型推导帮我们做了一切。

#### removeIf()
该方法签名为`boolean removeIf(Predicate<? super E> filter)`，作用是删除容器中所有满足`filter`指定条件的元素，其中`Predicate`是一个函数接口，里面只有一个待实现方法`boolean test(T t)`，同样的这个方法的名字根本不重要，因为用的时候不需要书写这个名字。

需求：假设有一个字符串列表，需要删除其中所有长度大于3的字符串。

我们知道如果需要在迭代过程冲对容器进行删除操作必须使用迭代器，否则会抛出`ConcurrentModificationException`，所以上述任务传统的写法是：
```java
// 使用迭代器删除列表元素
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
Iterator<String> it = list.iterator();
while(it.hasNext()){
    if(it.next().length()>3) // 删除长度大于3的元素
        it.remove();
}
```
现在使用`removeIf()`方法结合匿名内部类，我们可是这样实现：
```java
// 使用removeIf()结合匿名名内部类实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.removeIf(new Predicate<String>(){ // 删除长度大于3的元素
    @Override
    public boolean test(String str){
        return str.length()>3;
    }
});
```
上述代码使用`removeIf()`方法，并使用匿名内部类实现`Precicate`接口。相信你已经想到用Lambda表达式该怎么写了：
```java
// 使用removeIf()结合Lambda表达式实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.removeIf(str -> str.length()>3); // 删除长度大于3的元素
```
使用Lambda表达式不需要记忆`Predicate`接口名，也不需要记忆`test()`方法名，只需要知道此处需要一个返回布尔类型的Lambda表达式就行了。
#### replaceAll()
该方法签名为`void replaceAll(UnaryOperator<E> operator)`，作用是对每个元素执行`operator`指定的操作，并用操作结果来替换原来的元素。其中`UnaryOperator`是一个函数接口，里面只有一个待实现函数`T apply(T t)`。

需求：假设有一个字符串列表，将其中所有长度大于3的元素转换成大写，其余元素不变。

Java7及之前似乎没有优雅的办法：
```java
// 使用下标实现元素替换
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
for(int i=0; i<list.size(); i++){
    String str = list.get(i);
    if(str.length()>3)
        list.set(i, str.toUpperCase());
}
```
使用`replaceAll()`方法结合匿名内部类可以实现如下：
```java
// 使用匿名内部类实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.replaceAll(new UnaryOperator<String>(){
    @Override
    public String apply(String str){
        if(str.length()>3)
            return str.toUpperCase();
        return str;
    }
});
```
上述代码调用`replaceAll()`方法，并使用匿名内部类实现`UnaryOperator`接口。我们知道可以用更为简洁的Lambda表达式实现：
```java
// 使用Lambda表达式实现
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.replaceAll(str -> {
    if(str.length()>3)
        return str.toUpperCase();
    return str;
});
```
#### sort()
该方法定义在List接口中，方法签名为`void sort(Comparator<? super E> c)`，该方法根据c指定的比较规则对容器元素进行排序。`Comparator`接口我们并不陌生，其中有一个方法`int compare(T o1, T o2)`需要实现，显然该接口是个函数接口。

需求：假设有一个字符串列表，按照字符串长度增序对元素排序。

由于Java7以及之前sort()方法在Collections工具类中，所以代码要这样写：
```java
// Collections.sort()方法
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
Collections.sort(list, new Comparator<String>(){
    @Override
    public int compare(String str1, String str2){
        return str1.length()-str2.length();
    }
});
```
现在可以直接使用`List.sort()`方法，结合Lambda表达式，可以这样写：
```java
// List.sort()方法结合Lambda表达式
ArrayList<String> list = new ArrayList<>(Arrays.asList("I", "love", "you", "too"));
list.sort((str1, str2) -> str1.length()-str2.length());
```