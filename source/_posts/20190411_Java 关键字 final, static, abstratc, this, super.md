---
title: Java 关键字 final, static, abstratc, this, super
date: 2019-04-11 14:08:08
tags:
    - Java
    - 面试
---

#### Java 有哪些关键字？

`final`, `static`, `abstract`, `this`, `super` 等等

#### *final 有什么作用和优缺点？*

`final` 是 `java` 语言中的保留关键字，可以修饰 `class`, `method`, `variable`。`final` 变量不可变，代表了可以安全的在多线程之间共享，`final` 关键字提高了性能

- `final` 修饰的类无法被继承。`final` 类中的所有成员方法都会被隐式地指定为 `final` 方法。

- `final` 修饰方法。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将 `final` 方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升（现在的Java版本已经不需要使用 final 方法进行这些优化了。类中所有的 `private` 方法都隐式地指定为 `final`。

- `final` 修饰的变量。如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。


##### final with class

`javase` 中有哪些类使用了 `final` 关键字？

`String`类：

![](https://blog-1251468774.cos.ap-shanghai.myqcloud.com/20190411_Java关键字_01.png)


使用 `final` 关键字防止类被篡改：

```java
/**
 * FinalClass.java
 * 
 * 下面这个类如果写成 extends Parent 会报错，因为 Parent 已经被final修饰，不能继承
 * 
 */
//public class FinalClass extends Parent {
//    private Integer sum(Integer num1, Integer num2) {
//        return num1 + num2;
//    }
//}

public class FinalClass {
    private Integer sum (Integer num1, Integer num2) {
        return num1 + num2;
    }
}

final class Parent {
    private String name = "zhangsan";

    public String getName() {
        return this.name;
    }
}
```

##### final with method

```java
/**
 * final 修饰方法时，方法无法被继承
 *
 */
public class FinalMethod {

    public int multiply (Integer num1, Integer num2) {
        return num1 * num2;
    }

    public final int sum (Integer num1, Integer num2) {
        return num1 + num2;
    }
}

class FinalMethodChild extends FinalMethod {

    @Override
    public int multiply(Integer num1, Integer num2) {
        return num1 * num2;
    }

    // 方法会报错，因为 final 方法无法被继承
//    public final int sum (Integer num1, Integer num2) {
//        return num1 + num2;
//    }
}
```
##### final with variable

- `final` 变量一般是大写
- `final` 变量如果引用的是基本类型，那么不可以修改重新赋值，
- `final` 如果引用的是对象，那么不可以修改引用，但是对象本身是可以变化的

```java
public class FinalVariable {

    public void change () {
        final String NAME = "zhangsan";
        // name = "lisi"; // 错误

        final ArrayList list;
        list = new ArrayList();

        list.add("zhangsan");
        // list = new ArrayList(); // 错误
    }
}
```

#### static 关键字

`static` 修饰的属性和方法都属于类，而不属于某个具体 `new` 出来的对象，所有对象共享相同的 `static` 属性和方法

`static` 修饰的属性和方法可以直接通过 `类名.变量` 或者 `类名.方法()` 调用


#### abstract 关键字

`abstract` 关键字可以修饰类或方法。

`abstract` 类可以扩展（增加子类），但不能直接实例化。

`abstract` 方法不在声明它的类中实现，但必须在某个子类中重写。

```java
public abstract class MyClass{}
  
public abstract String myMethod();
```

- 采用 `abstract` 方法的类本来就是抽象类，并且必须声明为 `abstract。`

- `abstract` 类不能实例化。

- 仅当 `abstract` 类的子类实现其超类的所有 `abstract` 方法时，才能实例化 `abstract` 类的子类。这种类称为具体类，以区别于 `abstract` 类。

- 如果 `abstract` 类的子类没有实现其超类的所有 `abstract` 方法，该子类也是 `abstract` 类。

- `abstract` 关键字不能应用于 `static`、`private` 或 `final` 方法，因为这些方法不能被重写，因此，不能在子类中实现。

- `final` 类的方法都不能是 `abstract`，因为 `final` 类不能有子类。


#### this 关键字

`this` 关键字代之当前对象，可以通过 `this` 引用当前对象的变量或者方法

#### super 关键字

super 关键字代指父对象，用来访问父对象的变量和方法

#### 关键字注意事项

- `abstract` 关键字和 `final` 关键字天生互斥，不能同时使用

- `super` 调用父类中的其他构造方法时，调用时要放在构造方法的首行！`this` 调用本类中的其他构造方法时，也要放在首行。

- `this`，`super`不能用在 `static` 方法中。`static` 中也不能调用非 `static` 修饰的变量或者方法，原理也是一样的。类范畴的无法调用对象范畴的东西，**`this` 和 `super` 是属于对象范畴的东西，而静态方法是属于类范畴的东西。**

- 静态代码快和非静态代码快的区别，静态代码块无论创建多少个实例都执行一次，非静态代码快每创建一个实例执行一次

- 静态代码块和构造方法的区别，不同的对象用不同的构造方法执行出来的对象可能不同，静态代码快就是所有对象相同的那部分

#### 参考链接

- [final,static,this,super 关键字总结](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Basis/final%E3%80%81static%E3%80%81this%E3%80%81super.md)

~

***