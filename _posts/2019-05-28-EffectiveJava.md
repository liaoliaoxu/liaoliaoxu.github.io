---
layout:     post
title:      EffectiveJava
subtitle:   EffectiveJava读书笔记
date:       2019-05-28
author:     Link
header-img: img/post-bg-keybord.jpg
catalog: true
tags:
    - Java
---

# 创建销毁对象

## 静态工厂方法与构造器

### 静态工厂方法替代公有构造器的优势

1. 静态工厂方法有名称
2. 不必每次调用都创建实例
3. 可返回原返回类型的子类
4. 泛型实例代码简洁（8解决了？）

### 静态工厂方法替代公有构造器的缺点

1. 不能子类化
2. 难以查明该静态函数为静态工厂方法

## 多个构造器参数时使用构造器Builder模式

### 多参时的三种方法

1. 重叠构造器：可读性差，容易弄错参数顺序

   ```java
   public class NutritionFacts{
       private final int servingSize;
       private final int servings;
       private final int calories;
       private final int sodium;

       public NutritionFacts(int servingSize, int serving){
           this(servingSize, serving, 0);
       }

        public NutritionFacts(int servingSize, int serving,
                int calories){
           this(servingSize, serving, calories,0);
       }

        public NutritionFacts(int servingSize, int serving,
                int calories, sodium){
           this.servingSize = servingSize;
           this.servings = servings;
           this.calories = calories;
           this.sodium = sodium;
       }
   }
   ```

2. JavaBeans(getter, setter)：类状态不稳定（有的参数有，有的为null），破坏类的不可变性
3. Builder 比较好，追求极致性能时有问题

   ```java
    public class NutritionFacts{
       private final int servingSize;
       private final int servings;
       private final int calories;
       private final int sodium;

       public static class Builder{
            private final int servingSize;
            private final int servings;
            private int calories;
            private int sodium;

            public Builder(){
                this.servingSize = servingSize;
                this.servings = servings;
            }

            public Builder calories(int val){
                calories = val;
                return this;
            }

            public Builder sodium(int val){
                sodium = val;
                return this;
            }

            public NutritionFacts build(){
                return new NutritionFacts(this);
            }
       }

        public NutritionFacts(Builder builder){
           servingSize = builder.servingSize;
           servings = builder.servings;
           calories = builder.calories;
           sodium = sbuilder.odium;
       }
   }

   //client
        NutritionFacts cocaCola = new NutritionFacts.Builer(240, 8)
            .calories(100).sodium(35).build();
   ```

## 单例模式实现方法

1. 公有域 简单
2. 静态工厂方法 安全灵活
3. 单元素枚举 最好

```java
public enum Elvis{
    INSTANCE;

    public void leaveTheBuilding(){}
}
```

## 避免不必要对象的创建

1. 可重用对象，String常量，用于比较的边界常量等
2. 多用基本类型，提高程序运行效率

## 保护性拷贝

1. 构造器在构造私有域时创建新对象而不是直接引用输入的对象
2. 访问器返回时，重新创建等值对象，而不是直接返回私有域对象

## 提供static函数的工具类

1. 显示私有构造，防止被实例化

## 清除过期对象引用

### 内存泄漏

1. 维护过期引用，pop返回之后，元素还被elements数组引用

    ```java
    public Object pop(){
        if(size==0)
            throw new EmptyStackException;
        //error
        return elements[--size];
        //replace
        Object element = elements[--size];
        elements[size] == null;
        return element
    }
    ```

2. 缓存。过期时间清除，或者新增项目时显示清除

3. 监听器与其他回调。API回调与弱引用

## 终结方法finalize()

### 大部分情况下不应使用

1. 不能保证被及时执行，甚至不能保证被执行
2. 终结过程抛出的异常会被忽略，且对象会处于被破坏状态
3. 严重的性能损失

### 应使用显示的终结方法

### 终结方法适用

1. 安全网，当显示终止方法未被调用时使用
2. 终结本地对等体

### 使用时应注意

1. 调用super.finalize
2. 记录终结方法的非法用法
3. 终结方法守卫者？

# 对象通用方法(equals, hashcode, toString, clone)

## equals

### 需要覆盖的场景

1. 有逻辑相等的概念
2. 父类的equals没有实现期望的行为

### 性质

1. 自反性
2. 对称性
3. 传递性 超类与子类混用是会出现问题

## hashcode
