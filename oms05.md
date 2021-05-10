# OMS05

> 点餐管理系统 05

## 任务前瞻

在 oms04 中，大家编写了点餐系统中十分重要的一些角色类，同时完成了一些指令的实现，此时，我们就可以开始着手实现具体的点餐、服务等环节了。 当然，由于 oms
并不采用多线程的写法，所以也无法还原一个真实的场景（例如用户点餐的同时厨师在做菜），所以本次 oms 需要频繁切换角色的登录并进行相关的操作。

本次 oms 的编写计划完成所有的功能，因此也是有编码需求的最后一次 oms，望大家认真对待。

## 任务

- 将所有 List 类（`PersonList、DishList等`）改写为单例模式
- 实现用户点餐、厨师做菜、服务员上菜等功能
- 新建所需要的一些单例模式类

### 说明

> 由于本次涉及到多个对象间的关系和数据交互，因此需要大家以面向对象的角度和关系型数据库存放数据使用的关系表的角度来自行设计一些存储类，题中不会给出详细说明，但可能暗含需要新建存储类的信息，请大家务必仔细分析

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

> 为了完善餐厅的运营流程，使得整个流程更加贴近现实，需要增加一些类

#### Order 类

> 大家在学数据库的时候应该已经接触过两个类之间的关系表，Order 就是这样一个的存在，作为`Customer`和`Menu`的一个关系表，存储着顾客本次的点菜

Order 的基本属性如下：

- orderID `Order的主键，具体格式见下[order指令]`
- CustomerID `表明是哪位顾客所点的菜品`
- WaiterID `表明本次是由哪位服务员进行接单以及送餐服务的（类似海底捞）`
- DishList `顾客所点菜品列表`
- isConfirm `顾客是否确认菜单`

**注意：**

- 当顾客在**初始情况**选择开始点餐时（见下，调用order指令），系统自动生成一张 Order 关系表，存储顾客的点餐情况（oms 餐厅具备数据长期存储功能）

#### Customer

> 在 oms4 中，顾客可以通过`login -i`和`login -n`两种方式进入 login 的环境，但是当前的环境中功能需要进一步扩展，在保留 oms4 的相关功能的前提下，增加下列模块：
>
> - 顾客在进行点餐前可以进行菜品和菜单的查询，因此继承了前些版本的查询功能

| 指令名 | [参数 1/子指令名] | [参数 2] | [参数 3] | [参数 4] | 功能 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 错误输出 | | | | | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；对于所有未提及的其他输入不合法的情况，输出`Input illegal` |
| rc | 金额 m | | | | 同 oms4 |
| gb | | | | | 同 oms4 |
| aplVIP | | | | | 同 oms4 |
| gd | -id | 菜品编号 | | | 同 oms2 |
| gd | -key | 关键字 | | | 同 oms2 |
| gd | -key | 关键字 | 第 n 页  | 每页 m 个记录 | 同 oms3 |
| pm | | | | | 同 oms2 |
| pm | 第 n 页 | 每页 m 个记录 | | | 同 oms3 |
| **order** | | | | | 进入点餐环境，详情见下 |
| **checkOrder** | | | | | 查看已点菜品，详情见下 |
| **confirm** | | | | | 提交菜单，详情见下 |

`order点餐环境`

> 和 login、sudo 等环境的组成类似，该环境是隶属于 login 的子环境，用户在该环境下进行点餐，并使用该环境下的指令

**在该环境中，循环读取指令，只接受 add 和 finish**

| 指令名 | [参数 1/子指令名] | [参数 2] | [参数 3] | [参数 4] | 功能 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 错误输出 | | | | | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；对于所有未提及的其他输入不合法的情况，输出`Input illegal`。|
| add | -i | Dish 主键 DID | m（正整数且保证数据合法） | | 点餐，将选择的菜品（m 道）加入到对应的 Order 实例中，如果不存在这道菜，输出`Dish selected is not exist`；如果菜品售空，输出`Dish selected is sold out`；如果点餐数量超过菜品数量，输出`Dish is out of stock`，本次点餐无效 |
| add | -n | Dish 名称 name（不考虑名称的重复和非法） | m（正整数且保证数据合法） | | 点餐，将选择的菜品（m 道）加入到对应的 Order 实例中，如果不存在这道菜，输出`Dish selected is not exist`；如果菜品售空，输出`Dish selected is sold out`；如果点餐数量超过菜品数量，输出`Dish is out of stock`，本次点餐无效 |
| finish | | | | | 结束选菜，**检查此时 Order 的菜品数量**，如果菜品数量为 0，输出`Please select at least one dish to your order`，继续接收指令；否则退出到上一层环境中 |

##### 注意

> 在用户调用add的时候，会优先检查一下当前是否存在自己未confirm的Order，如果存在则可以继续追加菜品，否则系统需要自动生成新的order实例，分配对应的orderID
> - 在点菜的时候，不考虑余额是否足够的问题
> - 在点菜的同时，add成功一次，就需要在相应的位置把当前该Dish的总量减一（不存在取消订单的api）

##### orderID 的格式

`orderID: CustomerID + '_' + (orderCount+1)`

> 这里的`orderCount`是指`Order-Customer关系表`中该用户已经完成（confirm）的 order 数量

示例：

```text
· Tom的CustomerID为: Cu00001
· 已经完成了3次order
       ↓
  那么他这次调用order指令生成的orderID为: Cu00001_4
```

##### 情景提示

- Samshui 走进你的餐厅，点了一餐，confirm
- Tony 走进你的餐厅，点了一餐，**没有 confirm**，checkOrder 了一下，觉得不够吃，接着追加了一份披萨，confirm
- Ami 前天来过了你的餐厅吃了一顿，觉得不错，今天又来点了一餐，confirm

##### 点餐示例

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

> 顾客查看当前已经点的菜品列表，并输出当前菜品的总额（保留 1 位小数）

输出还是和原先针对菜品的输出顺序相同（打印的顺序有两个优先级，首先是 H>C>O，其次是 6 位编码从小到大）

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
Customer doesn't confirm
No other waiters take orders
1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1
2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:1
3.DID:C000000,DISH:IceCream,PRICE:4.0,NUM:2
4.DID:C000002,DISH:BoBoJi,PRICE:55.0,NUM:1
5.DID:O000100,DISH:Cake,PRICE:74.0,NUM:3
|
SUM:158.0
```

`confirm`

| 指令名 | 情况 |
| :---: | :---: |
| confirm | 如果当前菜单有效（该Customer有未confirm的菜单），输出`Order Confirmed`，并退出顾客的 login 环境；如果不存在有效菜单，输出`No order can be confirmed`，不退出顾客的 login 环境 |

**方式**：当顾客confirm时，由系统自动分配当前菜单（Order），此时系统会检索所有的 Waiter 对象，按照该对象当前负责的 Order 数量（由小到大）以及 WID（由小到大）进行排序并选择第一个 Waiter 的
WID，填入该菜单属性中。

```text
[status : 排序后]
----[WID]----   ----[Order_Count]----
   Wa00009                5
   Wa00011                5
   Wa00007                6
   Wa00004                8

=> 选择Wa00009填入order.WaiterID
```

#### 一个推荐的设计方案
```text
# 由于仅初始情况下调用order指令时系统才会生成Order的实例，因此建议：

order指令 -> return Order实例
                   ↓
                   confirm指令: 参数(Order 实例） 
```

#### Waiter

> 在 oms4 中，服务员可以通过`login -i`和`login -n`两种方式进入 login 的环境，但是当前的环境中服务员功能需要进一步扩展，在保留 oms4 的服务员登录后的相关功能前提下，增加下列模块：

| 指令名 | [参数 1/子指令名] | [参数 2] | [参数 3] | [参数 4] | 功能  |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 错误输出 | | | | | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；对于所有未提及的其他输入不合法的情况，输出`Input illegal` |
| getOrderList | | | | | 获取当前所服务的 Order 清单，详情见下 |
| manageOrder | | | | | 分配订单，详情见下 |
| DeliverAndReceive | | | | | 送餐并且收钱，详情见下 |
| rw | CID（顾客ID） | money （具体充入的金额） | | | 具体逻辑参见 oms4，注意：当这次充钱达到 VIP 标准后，总金额应该按照八折重新计算 |

`getOrderList`

> 当前服务员所服务的 Order 清单

| 指令名 | 成功输出 |
| :---: | :---: |
| getOrderList | 若当前服务员所服务的顾客清单为空，则输出 `The list of customers served is empty` |
| getOrderList | 若当前服务员所服务的顾客清单不为空，按照接单的顺序进行输出（类似队列），具体输出格式见下 |
| 错误输出 | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；对于所有未提及的其他输入不合法的情况，输出`Input illegal`。 |

```
# 点餐Order输出示例

[-] checkOrder
[+]
Customer has confirmed
No other waiters take orders
1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1,SERVE:NO
2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:2,SERVE:NO

# 顾客登出，对应的服务员登录

# 服务员getOrderList输出示例
[-] getOrderList
[+]
1. OID:Cu00000_1,DISH:[1 FanQieChaoDan,2 ChaoTuDouSi]

```

`manageOrder`

> 服务员接单后分配 Order，即将 Order 清单中第一个 Order 送入厨房

每当服务员输入一次 manageOrder 指令，则将 Order 清单的第一个 Order 送个厨房，即一个命令只能送一个 Order。

| 指令名| 成功输出 |
| :---: | :---: |
| manageOrder | 若当前服务员所服务的顾客清单不为空，则送入 Order 清单中的第一个 Order，具体输出格式见下 |

| 指令名 | 错误输出 |
| :---: | :---: |
| 错误输出 | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；对于所有未提及的其他输入不合法的情况，输出`Input illegal`。 |
| manageOrder | 若当前服务员所服务的顾客清单为空，则输出`Can not manage empty OrderList` |

```
# 点餐Order输出示例
[-] checkOrder
[+]
Customer has confirmedNo
other waiters take orders
1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1,SERVE:NO
2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:1,SERVE:NO

# 顾客登出，对应的服务员登录

# manageOrder输出示例
[-] manageOrder
[+] Deliver order success

```

`DeliverAndReceive`

> 厨房完成菜品制作后，由服务员送餐，并结账

服务员送餐，若 Order 的状态为未送餐，则送餐成功，否则失败；每一次执行该命令，则送一个订单 Order，并结账

| 指令名 | [参数 1] | 成功输出 |
| :---: | :---: | :---: |
| DeliverAndReceive | OID 订单 ID | 若 Order 的状态为未送餐且余额充足，具体输出格式见下 |

| 指令名 | [参数 1] | 失败输出 |
| :---: | :---: | :---: |
| 错误输出 | OID 订单 ID | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；对于所有未提及的其他输入不合法的情况，输出`Input illegal`。 |
| DeliverAndReceive | OID 订单ID | 若订单的状态为已送餐，输出`Have Served order` |
| DeliverAndReceive | OID 订单ID | 计算总价格（普通用户全价，VIP 顾客打八折）。若余额不足，则输出`Insufficient balance` |

```
# 点餐Order输出示例
[-] checkOrder
[+]
Customer has confirmed
No other waiters take orders
1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1,SERVE:YES
2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:1,SERVE:YES

# 顾客登出，服务员登录，receiveMoney输出示例
[-] DeliverAndReceive
[+] OID:Cu00000_1,Total:21.0,Balance:70.0
```

`rw`

> 若用户结账时发现自己余额不足，需要让服务员帮助自己充钱

| 指令名 | [参数 1] | [参数 2] | 成功输出 |
| :---: | :---: | :---: | :---: |
| rw | CID 顾客 ID | money 具体充入的钱 | 具体逻辑参见 oms4，注意：当这次充钱达到 VIP 标准后，总金额应该按照八折重新计算 |
