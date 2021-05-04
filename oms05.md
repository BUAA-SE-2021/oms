<div style="font-family: 'Fira Code', 'PingFang SC',serif">

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

  private DishList() {
  }

  // 由于oms不需要考虑线程安全问题，因此可以使用不保证线程安全的懒汉式写法
  public static DishList getInstance() {
	if (instance == null) instance = new DishList();
	return instance;
  }
}
```

### 新增

> 为了完善餐厅的运营流程，使得整个流程更加贴近现实，需要对一些基础类进行属性的扩写

#### Dish类

- 加入`CookList`属性，存储和Cook相关的主键（推荐存储ID，当然直接存储一个对象也是可以的，取决于个人的实现，只要保证**数据的一致性**即可），不要求新建CookList类并使用单例模式
- 加入`time`属性，为double类的数据，代表该菜品的耗时（单位：小时），默认值0.0（保留一位小数）

#### Order类

> 大家在学数据库的时候应该已经接触过两个类之间的关系表，Order就是这样一个的存在，作为`Customer`和`DishList`的一个关系表，存储着顾客本次的点菜

Order的基本属性如下：
- CustomerID `表明是哪位顾客所点的菜品`
- WaiterID `表明本次是由哪位服务员进行接单以及送餐服务的（类似海底捞）`
- DishList `顾客所点菜品列表`

**注意：**
- Order是有一定的生存周期的，当顾客选择开始点餐时（见下），系统自动生成一张Order关系表，存储顾客的点餐情况，在顾客结账和服务员进行找零后自动删除（oms餐厅不具备数据长期存储功能）

#### Customer

> 在oms4中，顾客可以通过`login -i`和`login -n`两种方式进入login的环境，但是当前的环境中功能需要进一步扩展，在保留oms4的相关功能的前提下，增加下列模块：
> - 顾客在进行点餐前可以进行菜品和菜单的查询，因此继承了前些版本的查询功能

| 指令名 | [参数1/子指令名] |[参数2] |[参数3] |[参数4] | 功能 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 错误输出 | | | | | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；**对于所有未提及的其他输入不合法的情况，输出`Input illegal`**。 |
| rc | 金额m | | | | 同oms4 |
| gb | | | | | 同oms4 |
| aplVIP | | | | | 同oms4 |
| gd | -id | 菜品编号 | | | 同oms2 |
| gd | -key | 关键字 | | | 同oms2 |
| gd | -key | 关键字 | 第n页 | 每页m个记录 | 同oms3 |
| pm | | | | | 同oms2 |
| pm | 第n页 | 每页m个记录 |  |  | 同oms3 |
| order | | | | | 进入点餐环境，详情见下 |
| checkOrder | | | | | 查看已点菜品，详情见下 |
| confirm | | | | | 提交菜单，详情见下 |
| checkLeft | | | | | 查看未上菜品，详情见下 |
| checkOut | | | | | 结账，详情见下 |

`order点餐环境`

> 和login、sudo等环境的组成类似，该环境是隶属于login的子环境，用户在该环境下进行点餐，并使用该环境下的指令

**在该环境中，循环读取指令，只接受add和finish**

| 指令名 | [参数1/子指令名] |[参数2] |[参数3] |[参数4] | 功能 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 错误输出 | | | | | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；**对于所有未提及的其他输入不合法的情况，输出`Input illegal`**。 |
| add | -i | Dish主键DID | m（正整数且保证数据合法） | | 点餐，将选择的菜品（m道）加入到对应的Order实例中，如果不存在这道菜，输出`Dish selected is not exist`；如果菜品售空，输出`Dish selected is sold out`；如果点餐数量超过菜品数量，输出`Dish is out of stock`，本道菜点餐无效  |
| add | -n | Dish名称name（不考虑名称的重复和非法） | m（正整数且保证数据合法） | | 点餐，将选择的菜品（m道）加入到对应的Order实例中，如果不存在这道菜，输出`Dish selected is not exist`；如果菜品售空，输出`Dish selected is sold out`；如果点餐数量超过菜品数量，输出`Dish is out of stock`，本道菜点餐无效 |
| finish | | | | | 结束选菜，检查此时Order的菜品数量，如果菜品数量为0，输出`Please select at least one dish to your order`，继续接收指令；否则退出到上一层环境中 |

**注意**：当顾客第一次进入到order环境中，由系统自动生成一张菜单（Order实例），此时系统会检索所有的Waiter对象，按照该对象当前负责的Order数量（由小到大）以及WID（由小到大）进行排序并选择第一个Waiter的WID，填入该菜单中。

```text
[status : 排序后]
----[WID]----   ----[Order_Count]----
   Wa00009                5
   Wa00011                5
   Wa00007                6
   Wa00004                8

=> 选择Wa00009填入Order
```

点餐示例
```text
[Menu]
1. DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,TOTAL:2
2. DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,TOTAL:80
3. DID:H000213,DISH:MaoXueWang,PRICE:15.0,TOTAL:10
4. DID:C000000,DISH:IceCream,PRICE:4.0,TOTAL:100
5. DID:C000001,DISH:PiDan,PRICE:15.0,TOTAL:150
6. DID:C000002,DISH:BoBoJi,PRICE:55.0,TOTAL:70
7. DID:O000000,DISH:Cola,PRICE:5.0,TOTAL:200
8. DID:O000100,DISH:Cake,PRICE:74.0,TOTAL:50

# 当前已登录
[-] order
[-] finish
[+] Please select at least one dish to your order
[-] add -i H000000 1 => 成功点菜后，菜品的余量就发生了变化，这也是数据一致性的体现
[-] add -i H000009 2
[+] Dish selected is not exist
[-] add -n FanQieChaoDan 2
[+] Dish is out of stock
[-] add -n FQCD 1
[+] Dish selected is not exist
[-] add -n FanQieChaoDan 1
[-] add -i H000000 1
[+] Dish selected is sold out
[-] finish
```

`checkOrder`

> 顾客查看当前已经点的菜品列表

输出还是和原先针对菜品的输出顺序相同（打印的顺序有两个优先级，首先是H>C>O，其次是6位编码从小到大）
```text
[Menu]
1. DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,TOTAL:30
2. DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,TOTAL:80
3. DID:H000213,DISH:MaoXueWang,PRICE:15.0,TOTAL:10
4. DID:C000000,DISH:IceCream,PRICE:4.0,TOTAL:100
5. DID:C000001,DISH:PiDan,PRICE:15.0,TOTAL:150
6. DID:C000002,DISH:BoBoJi,PRICE:55.0,TOTAL:70
7. DID:O000000,DISH:Cola,PRICE:5.0,TOTAL:200
8. DID:O000100,DISH:Cake,PRICE:74.0,TOTAL:50

# 点餐Order输出示例

[-] checkOrder
[+] 
1. DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1
2. DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:1
3. DID:C000000,DISH:IceCream,PRICE:4.0,NUM:2
4. DID:C000002,DISH:BoBoJi,PRICE:55.0,NUM:1
5. DID:O000100,DISH:Cake,PRICE:74.0,NUM:3
```

`confirm`

> 确认菜单，交给服务员进行下一步的操作，此时**默认退出顾客的login环境**
</div>