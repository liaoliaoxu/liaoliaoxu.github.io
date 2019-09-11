---
layout:     post
title:      clean code
subtitle:   代码简洁之道
date:       2019-09-10
author:     Link
header-img: img/post-bg-keybord.jpg
catalog: true
tags:
    - Code
---
# clean code

## 命名

1. 名副其实
   能够表达变量的意思
2. 避免误导
   1. 不使用保留有特殊意义的名称 List, Set等
   2. 少使用l和O作为名称
3. 有意义区分
   1. 反例1: a1, a2, a3
   2. **反例2: apk, apkInfo**
4. 可朗读命名
   为了交流
5. 独特可搜索的命名
6. 别用俚语或者梗命名
7. 别用双关语，比如为了一致性用add表示append
8. 有意义的语境
   1. firstName, lastName, phoneNum, street, houseNum等，可设置Address类包装上述变量

## 函数

> 每个函数一个抽象层级

1. 参数
   1. 减少输入参数
   2. 不使用boolean值标示根据不同输入做不同的事情
   3. 避免使用输出参数（改变入参）
2. 无副作用，函数做的事情=函数名描述
3. 分割指令与询问避免 if(setAttr(attr))
4. 异常
   1. 异常代替错误码
   2. 异常处理单独函数

   ```java
   void delete(){
       try{
           doDelete();
       }
       catch(Exception e){
           log(e);
       }
   }
   ```

## 注释

1. 少用注释(需要额外精力阅读，且常忘记维护)

    ![comment](../img/bad_comment.jpg)

2. 适用注释的场景
   1. 法律信息
   2. 意图表示
   3. 无法改动的标准库阐释
   4. 警示
   5. 公共api的JavaDoc注释

## 格式

### 垂直格式

1. 调用与被调用方代码放在相邻位置
2. 相关类似概念的方法放在一起

### 水平格式