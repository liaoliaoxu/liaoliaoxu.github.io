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