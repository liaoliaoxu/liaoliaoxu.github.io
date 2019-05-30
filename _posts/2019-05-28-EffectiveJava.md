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

## 提供static函数的工具类

1. 显示私有构造，防止被实例化

## 一些小点与疑问

1. 类不可见与线程安全？
2. 服务提供者框架

## butler中的一个教训

1. 使用最新版本有风险，在最初尝试是应该套用原有的成熟的版本和配置（mybatisplus）