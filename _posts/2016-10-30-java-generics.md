---
layout: post
title:  "Java Generics"
date:   2016-10-30 14:49:52
categories: java generics
published: true
comments: true
thread: 20161030145055555
---
Java 泛型
---
## 为什么要使用泛型
使用泛型的好处：
- 编译期的强类型检查
- 去除类型的强制转换
- 使程序员能实现通用算法

## 泛型类型(Generic Types)
泛型类型是class／interface的类型参数化。

```java
public class Box {
  private Object object;

  public void set(Object object) { this.object = object; }
  public Object get() { return object; }
}
```
泛型版本的代码：

```java
public class Box<T>{
  private T t;

  public void set(T t) {this.t = t;}
  public T get() { return t;}
}
```

## 泛型类型参数命名规约(Type Parameter Naming Conventions)
类型参数命名是单个大写的字母，常用的命名：

- E 元素(Java 集合框架广泛使用)
- K key
- N Number
- T Type
- V Value
- S,U V etc. 2nd, 3rd, 4th types

## 调用&实例化一个泛型
代码中引用Box泛型类，你必须执行一个泛型类型调用，使用具体值替换T，比如Integer。

```java
Box<Integer> integerBox;

Terminology:
- Type Parameter = 'T'
- Type Argument = 'Integer'
- Parameterized type = 'Box<Integer>'  泛型类型的一次调用通常被称为一个 parameterized type.
- Raw Type= 'Box raw = new Box();'
```

## 泛型，继承和子类型（Generics, Inheritance, and Subtypes）
考虑一下方法的定义：

```java
public void boxTest(Box<Number> n) {
  /**
   *
   */
}
```
调用该方法是，传入Box<Integer>/Box<Double>是变异不通过的，因为， Box<Integer>和Box<Double>不是Box<Number>的子类。
