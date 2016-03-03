---
layout: post
title:   Java静态嵌套类和非静态嵌套类的区别
date:   2016-03-02 09:04:52
categories: Java
published: true
comments: true
thread: 201603020202425555
---

在Java中不能Top class定义为static， 只有Nested classes才可以为static。
在Java里有四种，主要使用两种Nested Class，一种是Static Nested也被叫做静态嵌套类；另一种是Non Static Nested也被叫做普通内部类（[Inner Class](http://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.1.3)） 。




## Static nested classes
获取方式：
 - OuterClass.StaticNestedClass nestedObject = new OuterClass.StaticNestedClass();
OuterClass的一个实例中可以有多个Static nested classes存在。

```Java
public class OuterClass {
    private static int a = 1;
    private int b=2;

    /**
     * public 修饰后可以在OuterClass的外部使用;
     * private 后只能在OuterClass中使用
     */
    private static class InnerClass {
        private static int b=3;
        public void execute() {
            System.out.println(a + b);
        }
    }

    public static void main(String[] args) {
        InnerClass a = new InnerClass();
        a.execute();

    }
}
```

## Non-Static nested classes（inner classes）
获取方式：
 - OuterClass.InnerClass innerClass = outerObject.new InnerClass();
OuterClass一个实例只能存在一个innerClass的实例。也叫做enclosing instance

```Java
public class OuterClass {
    private int a=1;
    private int b=2;
     class Inner{
         private int a=3;

         public void method() {
            System.out.println(a);
            System.out.println(OuterClass.this.a);
            System.out.println(b);
         }
    }

    public static void main(String[] args) {
        Inner inner = new OuterClass().new Inner();
        inner.method();
    }
}
```

## local inner classes
这种是定义在方法内部的, 所以也被叫做Method local inner class。
类似于局部变量，不能定义为public，protected，private或者static类型。
定义方法中，只能方法中声明为final类型的变量。

```Java
public class OuterClass {
    public static void main(String[] args) {
        new OuterClass().method();
    }

    void method() {
        /**
         * 如果在OuterClass有'Local'的inner class, 要想使用该处的Local必须在声明后实例使用
         */
        class Local {
            void method() {
                System.out.println("Local");
            }
        }
        new Local().method();
    }
}
```

## [anonymous inner classes](http://docs.oracle.com/javase/specs/jls/se7/html/jls-15.html#jls-15.9.5)
    - 永远不能为abstract
    - 隐含是final的
    - 是一个local inner class并且不是静态的。

获取方式：
 - new *ParentClassName*(*constructorArgs*) {*members*}
 - new *InterfaceName*() {*members*}

```Java
public class OuterClass {

    public void print( Date date ) {
        System.out.println( date );
    }

    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();
        outerClass.print( new Date() {
            public String toString() {
                return "ParentClassName anonymous ";
            }
        });
        new Runnable() {
            @Override
            public void run() {
                System.out.println("InterfaceName anonymous");
            }
        }.run();
    }
}
```
