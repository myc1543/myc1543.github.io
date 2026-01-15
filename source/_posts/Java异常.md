---
title: Java异常

date: 2026-01-13 23:54:52
tags: [Java]
categories: [Java]
description: 这是一篇关于Java异常文章主要讲解了Java异常的分类、异常的处理以及生产过程中的一些建议。

---



## 1.认识异常

程序运行过程中可能会出现很多的问题，这些问题可能是代码写的有错误，又或者是输入的数据导致的一些问题......

异常就是对这些问题的表现。Java中的异常体系如下

![](java-basic-exception-1.png)  

所有的不正常类都继承于`Throwable`，分为Error和Exception两类。

- Error
- Exception
  - 受检异常
  - 非受检异常

Error表示JVM发生的错误，例如内存溢出、线程死锁。一旦出现Error，就代表程序崩溃了，一般来说对于开发者来说是没有办法在程序中处理这些Error的。

Exception表示程序运行过程中发生的不期望的事件，通过异常处理机制可以进行恢复。Exception下分为受检异常和非受检异常。

非受检异常：指的是Error和RuntimeException以及它们的子类。在编译时，不会提示和发现这些异常，不要求处理这些异常。例如数组越界异常、除0异常、空指针异常等等。

受检异常：除了非受检异常(Error和RuntimeException以及它们的子类)的异常都为受检异常。Javac强制开发这对受检异常做处理，否则编译不通过。例如IO异常、SQL异常

![](image-20260114112614995.png)

## 2. 异常处理机制

### 2.1 try-catch-finally

从一个例子中学习异常处理

```
 try {
            System.out.println("try 块：开始执行");
            // 异常
            int result = 10 / 0;
            System.out.println("这行不会被执行");
        } catch (ArithmeticException e) {
            System.out.println("catch 块：捕获到算术异常 - " + e.getMessage());
        } finally {
            System.out.println("finally 块：无论是否发生异常，这里总会执行");
        }
```

- try块：负责捕获异常，一旦try中发现异常，程序的控制权将被移交给catch块中的异常处理程序。在可能发生异常的代码块，应该包含在try块中。

　　**【try语句块不可以独立存在，必须与 catch 或者 finally 块同存】**

- catch块：如何处理？比如发出警告：提示、检查配置、网络连接，记录错误等。执行完catch块之后程序跳出catch块，继续执行后面的代码。

　  【**编写catch块的注意事项：多个catch块处理的异常类，要按照先catch子类后catch父类的处理方式，因为会【就近处理】异常（由上自下）。**】

- finally：最终执行的代码，用于关闭和释放资源。

**多重捕获**

一个 try 代码块后面跟随多个 catch 代码块的情况就叫多重捕获。

```
 public static void multipleCatchExample() {
        String str = null;
        int[] array = {1, 2, 3};

        try {
            System.out.println("try 块：尝试访问空字符串");
            int length = str.length(); // NullPointerException
            System.out.println("尝试访问数组越界");
            int value = array[10]; // ArrayIndexOutOfBoundsException
        } catch (NullPointerException e) {
            System.out.println("catch 块1：捕获到空指针异常 - " + e.getMessage());
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("catch 块2：捕获到数组越界异常 - " + e.getMessage());
        } catch (Exception e) {
            System.out.println("catch 块3：捕获到其他异常 - " + e.getMessage());
        } finally {
            System.out.println("finally 块：清理资源");
        }
    }
```



#### finally的注意事项

在try中，即使有return,break,continue等改变流程的语句，finally最终也会执行

```
public static void main(String[] args)
{
    int re = bar();
    System.out.println(re);
}
private static int bar() 
{
    try{
        return 5;
    } finally{
        System.out.println("finally");
    }
}
/*输出：
finally
5
*/
```



由于最终会执行finally的代码，所以如果finally也使用了return，会导致finally后renturn的值会覆盖之前return的值

```
public static void main(String[] args)
{
    int re = bar();
    System.out.println(re);
}
private static int bar() 
{
    try{
        return 5;
    } finally{
        System.out.println("finally");
        return 10;
    }
}
/*输出：
finally
1
*/
```



#### finally中的return会消除前面try-catch块中的异常

```
public static void main(String[] args) {
        try{
            tryMethod();
        }catch (Exception e){
            System.out.println("捕获到异常");
        }

        try{
            catchMethod();
        }catch (Exception e){
            System.out.println("捕获到异常");
        }
    }


    public static int tryMethod() {
        try {
            int i = 5 / 0;
            return 0;
        } finally {
            System.out.println("finally");
            return 1;
        }
    }

    public static int catchMethod(){
        try {
            int i = 5 / 0;
            return 0;
        }catch (Exception e){
            throw new RuntimeException("catch块中的异常");
        }finally {
            System.out.println("finally");
            return 1;
        }
    }
```



#### finally中的异常会覆盖前面try-catch块中的异常

```
public static void main(String[] args) {
        try{
            tryMethod();
        }catch (Exception e){
            System.out.println(e.getMessage());
        }

        try{
            catchMethod();
        }catch (Exception e){
            System.out.println(e.getMessage());
        }
    }


    public static int tryMethod() {
        try {
            int i = 5 / 0;
            return 0;
        } finally {
            throw new RuntimeException("finally的异常");
        }
    }

    public static int catchMethod(){
        try {
            int i = 5 / 0;
            return 0;
        }catch (Exception e){
            throw new RuntimeException("catch块中的异常");
        }finally {
            throw new RuntimeException("finally的异常");
        }
    }

```



### 2.2 throw & throws

Java中抛出异常有两种方式， `throw`和`throws`

#### thow关键字

使用throw将抛出一个异常对象，如果不进行处理，将由上层调用者进行处理，程序会在这里停止，不会执行后续代码。

![](image-20260114113935810.png)



#### throws关键字

throws关键字申明将抛出何种类型的异常，作用于方法。当某个方法可能会抛出某种异常时用于throws 声明可能抛出的异常，然后交给上层调用它的方法程序处理。

```
 public static void readFile(String filename) throws IOException {
        System.out.println("尝试读取文件: " + filename);
        // FileReader 的构造方法声明了 throws FileNotFoundException
        // 我们不在这里处理，而是继续向上抛出
        FileReader reader = new FileReader(filename);
        System.out.println("文件读取成功");
        reader.close();
    }
```



### **throw与throws的比较**

1、throws出现在方法函数头；而throw出现在函数体。
2、throws表示出现异常的一种可能性，并不一定会发生这些异常；throw则是抛出了异常，执行throw则一定抛出了某种异常对象。
3、两者都是消极处理异常的方式（这里的消极并不是说这种方式不好），只是抛出或者可能抛出异常，但是不会由函数去处理异常，真正的处理异常由函数的上层调用处理。



## 3.自定义异常

代码中我们可以自定义异常。需要注意以下三点

- 所有异常都必须是 Throwable 的子类。
- 如果希望写一个检查性异常类，则需要继承 Exception 类。
- 如果你想写一个运行时异常类，那么需要继承 RuntimeException 类。

```
public class BusinessException extends Exception{
    /*
    错误码
     */
    String errorCode;

    /*
    构造方法
     */
    public BusinessException(){}

    public BusinessException(String message){
        super(message);
    }

    public String getErrorCode() {
        return errorCode;
    }

    public void setErrorCode(String errorCode) {
        this.errorCode = errorCode;
    }
}

```

```
public static void main(String[] args) {
    try {
        service();
    }catch (Exception e){
        System.out.println("系统错误:" + e.getMessage());
    }
}

public static void service() throws BusinessException {
    List<Long> idList = new ArrayList<>();
    idList.add(1L);
    idList.add(2L);
    idList.add(0L);
    for(Long id : idList){
        if(id == 0){
            throw new BusinessException("查询结果错误");
        }
    }
}
```



## 4.使用异常的实践

### 4.1 处理异常

在业务中处理异常主要有两种

- 记录问题，跳过当前问题继续向后执行
- 抛出异常，返回错误、降级处理等等

在处理流程时，应该要注意以下几个点

- 不要通过异常来进行流程控制，因为捕获异常会比正常流程花费更多的时间(创建一个异常对象需要填写堆栈信息)
- 非受检异常尽量通过预先检查来规避，例如
  - 空指针提前判空
  - 数组越界提前判断
- 除非是在最外层的地方如Controller，应该要捕获自己需要的异常，而不是一股脑Exception
- catch住的异常，如果不处理，也要记录日志，便于排查问题

### 4.2 阿里巴巴开发手册对于异常处理的建议

【强制】Java 类库中定义的可以通过预检查方式规避的RuntimeException异常不应该通过catch 的方式来处理，比如：NullPointerException，IndexOutOfBoundsException等等。

【强制】catch时请分清稳定代码和非稳定代码，稳定代码指的是无论如何不会出错的代码。对于非稳定代码的catch尽可能进行区分异常类型，再做对应的异常处理。
说明：对大段代码进行try-catch，使程序无法根据不同的异常做出正确的应激反应，也不利于定位问题，这是一种不负责任的表现。

【强制】finally块必须对资源对象、流对象进行关闭，有异常也要做try-catch。

【强制】捕获异常与抛异常，必须是完全匹配，或者捕获异常是抛异常的父类。

【推荐】方法的返回值可以为null，不强制返回空集合，或者空对象等，必须添加注释充分

【推荐】防止NPE，是程序员的基本修养，注意NPE产生的场景：
1）返回类型为基本数据类型，return包装数据类型的对象时，自动拆箱有可能产生NPE。
反例：public int f() { return Integer对象}， 如果为null，自动解箱抛NPE。
2） 数据库的查询结果可能为null。
3） 集合里的元素即使isNotEmpty，取出的数据元素也可能为null。
4） 远程调用返回对象时，一律要求进行空指针判断，防止NPE。
5） 对于Session中获取的数据，建议NPE检查，避免空指针。
6） 级联调用obj.getA().getB().getC()；一连串调用，易产生NPE。

【推荐】定义时区分unchecked / checked 异常，避免直接抛出new RuntimeException()，更不允许抛出Exception或者Throwable，应使用有业务含义的自定义异常。推荐业界已定义过的自定义异常，如：DAOException / ServiceException等。

【参考】对于公司外的http/api开放接口必须使用“错误码”；而应用内部推荐异常抛出；跨应用间RPC调用优先考虑使用Result方式，封装isSuccess()方法、“错误码”、“错误简短信息”。



## 5.深入Excetion本质

### 5.1 创建异常的开销？

```
    public static int time;
    public static void main(String[] args) {
       time = 10000;
       createObject();
       createException();
       createAndCatchException();
    }

    public static void createObject(){
        long start = System.nanoTime();
        for(int i = 0;i < time;i++){
            new Object();
        }
        System.out.println("创建对象:" + (System.nanoTime() - start));
    }

    public static void createException(){
        long start = System.nanoTime();
        for(int i = 0;i < time;i++){
            new Exception();
        }
        System.out.println("创建异常对象:" + (System.nanoTime() - start));
    }

    public static void createAndCatchException(){
        long start = System.nanoTime();
        for(int i = 0;i < time;i++){
            try{
                throw new Exception();
            }catch (Exception e){

            }
        }
        System.out.println("创建并捕获异常对象:" + (System.nanoTime() - start));
    }
```

结果：

```
创建对象:252083
创建异常对象:6987208
创建并捕获异常对象:11605084
```

可以看到建立一个异常对象，是创建普通对象耗时的数十倍，而抛出、接住一个一个异常的时间又是创建一个异常对象的数倍。

### 5.2 创建异常时发生了什么？

（1）调用构造方法

调用构造方法，调用父类的构造方法Exception，Throwable的调用方法

```
public BusinessException(String message){
    super(message); // 调用Exception的构造方法
    }
    
public Exception(String message) {
    super(message); // 调用Throwable的构造方法
    }
    
public Throwable(String message) {
    fillInStackTrace();
    detailMessage = message;
    }
```



（2）fillInStackTrace() 本地方法

![image-20260114153247929](/image-20260114153247929.png)

在这一步，会保存调用栈信息。
调用栈信息包括了

- 类名
- 方法名
- 文件名
- 行号

捕获调用栈信息，会遍历线程栈帧，创建 StackTraceElement 数组。这一步耗时很大，也就是为什么创建异常会耗时很大的原因。

### 5.3 优化

可以通过禁用堆栈追踪的方式来提高性能，但是一般不建议这么做，因为异常本来就是用于追踪问题的。

```
public class MyException extends RuntimeException {
    public MyException(String message) {
        // writableStackTrace = false 不捕获堆栈
        super(message, null, false, false);
    }
}
```

