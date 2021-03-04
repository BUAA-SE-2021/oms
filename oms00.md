# OMS00

> 点餐管理系统 00

## 题目背景

餐厅中经常见到这样的场景：顾客根据菜单选择菜品，厨师接到订单之后开始制作，制作完成后，服务员为对应的客人上菜。顾客在就餐过程中，也可以继续点菜。当顾客就餐完毕后，服务员进行结账，打印账单，显示客人点过的所有菜，并统计菜品总价。餐厅老板可以对食材、服务员、厨师进行管理，也可以对消费流水进行统计。为了对餐厅的上述过程进行统一管理，提高服务效率，现开发一款餐厅管理系统。

## 题目描述

本次先实现java最基本的输入输出测试。

你的任务是编写一个Test类：

- 程序开始运行时，在终端打印一行字符：

```shell
----- Welcome to Our BUAA Order Management System! -----
```

- 当终端输入QUIT时，系统退出，并在终端打印一行字符：

```shell
----- Good Bye! -----
```

- 对于其他的输入，不做任何处理，等待下一行输入

## 参考实现

- java打印字符串与其他语言有所区别，不是直接调用print函数，可以用如下语句：

```java
String str = "Hello world!"
System.out.println(str);
```

- java连续读取输入行的一种实现：

```java
Scanner in = new Scanner(System.in);
String argStr;
while (true) {
    argStr = in.nextLine();
}
```
