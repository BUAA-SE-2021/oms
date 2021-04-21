# OMS05

> 点餐管理系统 05

## 任务前瞻

在 oms04 中，大家编写了点餐系统中十分重要的一些角色类，同时完成了一些指令的实现，此时，我们就可以开始着手实现具体的点餐、服务等环节了。
当然，由于oms并不采用多线程的写法，所以也无法还原一个真实的场景（例如用户点餐的同时厨师在做菜），所以本次oms需要频繁切换角色的登录并进行相关的操作。

本次oms的编写计划完成所有的功能，因此也是有编码需求的最后一次oms，望大家认真对待。

## 任务

- 将所有 List 类（`PersonList、DishList等`）改写为单例模式
- 实现用户点餐、厨师做菜、服务员上菜等功能
- 新建所需要的一些单例模式类


### 单例模式改写

本次实验需要大家把前几次实现功能时引入的 List 类（存储对象）修改为单例模式。

> Java Singleton 模式主要作用是保证在 Java 应用程序中，一个类 Class 只有一个实例存在。 使用 Singleton 好处还在于可以节省内存，因为它限制了实例的个数，有利于 Java 垃圾回收（garbage collection）
>  
> [实现单例模式的常见方法](http://www.blogjava.net/kenzhh/archive/2013/03/15/357824.html)

```java
// 以DishList类为例
class DishList {
  private static DishList instance;
  private DishList(){}

  // 由于oms不需要考虑线程安全问题，因此可以使用不保证线程安全的懒汉式写法
  public static DishList getInstance(){
    if (instance == null) instance = new DishList();
    return instance;
  }
}
```

### 基本功能

