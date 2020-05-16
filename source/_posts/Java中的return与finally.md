---
title: Java 中的 return 与 finally
id: return-and-finally-in-java
date: 2019-09-22 17:07:45
updated: 2019-09-22 17:07:45
categories:
  - 如何精致地拧螺丝
tags:
  - java
  - exception
---
这周出去面试，笔试题中有一道涉及 return 和 finally 的题，当时只依稀记得上课时老师讲的 finally 会在 return 前执行，但是如何执行的没有仔细研究过，故写篇文章好好梳理一下。

此文章只对代码和运行结果做合理猜测与总结，具体运行原理可看参考资料部分，因为涉及到JVM的堆栈之类的，这部分我还不太熟悉，因此此文章中不会对此过多阐述。

<!-- MORE -->

### finally 是否一定会执行

两种情况下 finally 不会执行

1. try 模块没有运行。
2. 使用`System.exit(0)`终止JVM。

### finally 和 return 谁先执行

运行下面代码：
```java
public static void main(String[] args) {
    System.out.println(test());
}

public static int test() {
    int a = 0;
    try {
        int b = 1/0;                    
        return b;                       
    } catch (Exception e) {
        System.out.println("catch");    
        return a;                       
    } finally {
        System.out.println("finally");  
    }
}
```
得到结果

```
catch
finally
0
```
我们大概可以猜到，执行顺序是 ，即<font color="red">在 finally 中不含 return 语句时，先执行 catch 里的非 return 语句，再执行 finally 里的语句，再执行 catch 里的 return 语句</font>。

那么 finally 里含 return 语句时的顺序呢？finally 能否修改 catch 里要 return 的结果呢？

### finally 对 return 结果的影响

#### 示例1

首先我们让 catch 和 finally 都对返回结果 a 进行修改
```java
public static int test() {
    int a = 0;
    try {
        int b = 1/0;
        return b;
    } catch (Exception e) {
        System.out.println("catch-1, a = " + a);   
        a = 1;                                     
        System.out.println("catch-2, a = " + a);   
        return a;                                  
    } finally {
        System.out.println("finally-1, a = " + a); 
        a = 2;                                     
        System.out.println("finally-2, a = " + a); 
    }
}
```
结果：

```
catch-1, a = 0
catch-2, a = 1
finally-1, a = 1
finally-2, a = 2
1
```
看起来执行顺序为 ，但是 finally 的修改结果并没有影响到 catch 的返回结果。

#### 示例2

我们仍然让 catch 和 finally 都对返回结果 s 进行修改，但是这次和 示例1 不同的是，s 是一个内部可变的对象，而不是一个基本类型
```java
public static StringBuilder test() {
    StringBuilder s = new StringBuilder("test ");
    try {
        int b = 1/0;
        return s;
    } catch (Exception e) {
        System.out.println("catch-1, s = " + s.toString());    
        s.append("catch ");                                    
        System.out.println("catch-2, s = " + s.toString());    
        return s;                                              
    } finally {
        System.out.println("finally-1, s = " + s.toString());  
        s.append("finally ");                                  
        System.out.println("finally-2, s = " + s.toString());  
    }
}
```
结果：
```
catch-1, s = test 
catch-2, s = test catch 
finally-1, s = test catch 
finally-2, s = test catch finally 
test catch finally 
```
看起来执行顺序为 ，这次 finally 的修改结果影响到了 catch 的返回结果。

结合示例1，我们大致能猜测到这样情况：<font color="red">在进入 finally 语句块之前，catch 语句块已经计算好了自己要 return 的数据，并把这个数据的地址存储在了某个地方。 </font>

示例1中，finally 中对a重新赋值，相当于把变量a的值的地址从A改到了B，但是 catch 语句中存储的地址指向的还是地址A，它不是指向变量a，所以变量a的值的修改和 catch 语句的返回结果没有任何关系。

示例2中，finally 修改的不是变量s的值的地址，而是变量s指向的地址里面存储的值。

#### 示例3

此示例在示例1的基础上多加了个 change() 方法，修改了 
```java
public static int test() {
    int a = 0;
    try {
        int b = 1/0;
        return b;
    } catch (Exception e) {
        System.out.println("catch-1, a = " + a);   
        a = 1;                                     
        System.out.println("catch-2, a = " + a);   
        return change(a);                          
    } finally {
        System.out.println("finally-1, a = " + a); 
        a = 2;                                     
        System.out.println("finally-2, a = " + a); 
    }
}

public static int change(int a) {
    System.out.println("change, a = " + a);
    a = 666;
    return a;
}

```
结果：
```
catch-1, a = 0
catch-2, a = 1
change, a = 1
finally-1, a = 1
finally-2, a = 2
666
```
此示例表明，在进 finally 之前，return 后面的 change() 也已经执行完了，已经算出了要 return 的最终结果，只是要等 finally 完结后再执行 return 动作。

#### 示例4

此示例在示例1的基础上多加了 
```java
public static int test() {
    int a = 0;
    try {
        int b = 1/0;
        return b;
    } catch (Exception e) {
        System.out.println("catch-1, a = " + a);   
        a = 1;                                     
        System.out.println("catch-2, a = " + a);   
        return a;                                  
    } finally {
        System.out.println("finally-1, a = " + a); 
        a = 2;                                     
        System.out.println("finally-2, a = " + a); 
        return a;                                  
    }
}
```
结果：

```
catch-1, a = 0
catch-2, a = 1
finally-1, a = 1
finally-2, a = 2
2
```
此示例表明，finally 中的 return 语句可修改最终的要返回的数据的地址。

#### 结论

1. 在进入 finally 语句块之前，catch 已经计算好了最终要 return 的数据，并缓存了该数据的地址
2. 对于可变对象，finally 修改可变对象内部的值，能够影响到 return 的结果
3. 对于不可变对象，finally 中修改不能影响到 return 的结果
4. finally 中的 return 语句可以修改最终要 return 的数据的地址。

补充：根据二面面试官反馈，finally 中不建议使用 return，因为会造成返回结果的不稳定性，即有时返回的是 catch 里的 return，有时返回的是 finally 里的 return，具体原因本人尚不得知，待后续研究。

### 参考资料

[【参考资料1】](https://blog.csdn.net/weixin_41005006/article/details/80643681)
[【参考资料2】](https://www.jianshu.com/p/841c63511b74)




