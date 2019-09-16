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
   1. 不使用保留有特殊意义的名称如List, Set等
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
2. 无副作用，即函数做的事情=函数名描述，不产生额外的效果
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

    ![comment](/img/bad_comment.jpg)

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

1. 缩进
2. 与团队同一

## 对象数据结构

1. 数据结构，暴露数据；可以此构造过程类代码，便于添加方法
2. 对象，隐藏数据暴露函数；面向对象代码，便于添加新类型
3. 避免混杂以上两种抽象方式

```java
//data and process
public class Point{
    public Integer x;
    public Integer y;
}

public class PointAction{

    public static double getDistance(Point a, Point b){
        //...
    }
}

//object
public class Point{
    public Integer x;
    public Integer y;

    public double getDistance(Point anotherPoint){
        //...
    }
}
```

## 异常

1. 使用异常而非状态码
2. TDD测试驱动开发
3. 业务开发减少可控异常使用；可控异常即函数显示抛出的异常
4. 定义常规流程，特例返回，比如版本管理中的无更新，外层调用就无需判null

    ```java
    public class UpdateCheckResult{
        //other fields
        public static final UpdateCheckResult NO_UPDATE = noUpdate();

        //methods
    }
    ```

5. 自定义异常包装第三方库，用统一的异常PortDeviceFailure，防止业务代码凌乱

    ```java
    public class LocalPort{
        private ACMEPort innerPort;

        public LocalPort(int portNumber){
            innerPort = new ACMEPort(portNumber);
        }

        public void open(){
            try{
                innerPort.open();
            } catch(DeviceResponseException e){
                throw new PortDeviceFailure(e);
            } catch(ATM1212UnlockedException e){
                throw new PortDeviceFailure(e);
            } catch(GMXError e){
                throw new PortDeviceFailure(e);
            }
        }
    }
    ```

## 边界

1. 封装三方库的使用
2. 构建学习性测试以应对三方库的变化
3. 虚拟接口针对尚未启动的依赖

## 测试

1. 测试先于开发
2. 原则：
   1. 快速：快才会想运行
   2. 独立：测试之间不互相依赖
   3. 可重复：任何环境任何场景下都可以通过（实际上不太可能，比如一些对数据库操作的如何满足这个要求）
   4. 自足验证：测试应有成功与否的输出，不应手动干预查看
   5. 及时：及时修改维护

## 类

1. 单一权责原则
2. 拆分类（保持单一权责原则），易于拓展
3. 接口和抽象类抽象实现
4. 类依赖于抽象而非具体实现
