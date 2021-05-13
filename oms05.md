# OMS05

> 点餐管理系统 05

## 任务前瞻

在 oms04 中，大家编写了点餐系统中十分重要的一些角色类，同时完成了一些指令的实现，此时，我们就可以开始着手实现具体的点餐、服务等环节了。 

当然，由于 oms并不采用多线程的写法，所以也无法还原一个真实的场景（例如用户点餐的同时厨师在做菜），所以本次 oms 需要频繁切换角色的登录并进行相关的操作。且对于现实世界中一些复杂的对应关系做了适当简化。

本次 oms 的编写计划完成所有的功能，因此也是有编码需求的最后一次 oms，望大家认真对待。

## 任务

- 将所有 List 类（`PersonList、DishList等`）改写为单例模式
- 实现用户点餐、厨师做菜、服务员上菜的功能
- 新建所需要的一些单例模式类

### 说明

> 由于本次涉及到多个对象间的关系和数据交互，因此需要大家以面向对象的角度和关系型数据库存放数据使用的关系表的角度来自行设计一些存储类，题中不会给出详细说明，但可能暗含需要新建存储类的信息，建议借助类图、顺序图、状态图等工具进行梳理分析。

### 单例模式改写

本次实验需要大家把前几次实现功能时引入的 List 类（存储对象）选择性地修改为单例模式，如果你觉得这个存储类十分契合单例模式的情形，那就可以按照下面的方式进行修改。

> Java Singleton 模式主要作用是保证在 Java 应用程序中，一个类 Class 只有一个实例存在。 使用 Singleton 好处还在于可以节省内存，因为它限制了实例的个数，有利于 Java 垃圾回收（garbage collection）
>
> [实现单例模式的常见方法](http://www.blogjava.net/kenzhh/archive/2013/03/15/357824.html)

```java
// 以Menu类为例
class Menu {
  private static Menu instance;

  private Menu() {
  }

  // 由于oms不需要考虑线程安全问题，因此可以使用不保证线程安全的懒汉式写法
  public static Menu getInstance() {
	if (instance == null) instance = new Menu();
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
- isConfirmed `顾客是否确认菜单(这个属性很重要)`
- isDelivered `服务员是否已送单`

**注意：**

- 当顾客在**初始情况**选择开始点餐时（见下，调用order指令），系统自动生成一张 Order 关系表，存储顾客的点餐情况（oms 餐厅具备数据长期存储功能）
- 我们**几乎所有的命令（login除外）**都是在 `SUDO` 环境下进行的，包括 `pm，gd` 等

#### Customer

> 在 oms4 中，顾客可以通过`login -i`和`login -n`两种方式进入 login 的环境，但是当前的环境中功能需要进一步扩展，在保留 oms4 的相关功能的前提下，增加下列模块：
>
> - 顾客在进行点餐前可以进行菜品和菜单的查询，因此继承了前些版本的查询功能

|   指令名    | [参数 1/子指令名] |   [参数 2]    | [参数 3] |   [参数 4]    | 功能                                                         |
| :---------: | :---------------: | :-----------: | :------: | :-----------: | :----------------------------------------------------------- |
|  错误输出   |                   |               |          |               | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；对于所有未提及的其他输入不合法的情况，输出`Input illegal` |
|     rc      |      金额 m       |               |          |               | 同 oms4                                                      |
|     gb      |                   |               |          |               | 同 oms4                                                      |
|   aplVIP    |                   |               |          |               | 同 oms4                                                      |
|     gd      |        -id        |   菜品编号    |          |               | 同 oms2                                                      |
|     gd      |       -key        |    关键字     |          |               | 同 oms2                                                      |
|     gd      |       -key        |    关键字     | 第 n 页  | 每页 m 个记录 | 同 oms3                                                      |
|     pm      |                   |               |          |               | 同 oms2                                                      |
|     pm      |      第 n 页      | 每页 m 个记录 |          |               | 同 oms3                                                      |
|  **order**  |                   |               |          |               | 进入点餐环境，详情见下                                       |
|   **co**    |                   |               |          |               | 查看已点菜品(check Order)，详情见下                          |
| **confirm** |                   |               |          |               | 提交菜单，详情见下                                           |

`order点餐环境`

> 和 login、sudo 等环境的组成类似，该环境是隶属于 login 的子环境，用户在该环境下进行点餐，并使用该环境下的指令

**在该环境中，循环读取指令，只接受 add 和 finish**

|  指令名  | [参数 1/子指令名] |         [参数 2]          |         [参数 3]          | [参数 4] | 功能                                                         |
| :------: | :---------------: | :-----------------------: | :-----------------------: | :------: | :----------------------------------------------------------- |
| 错误输出 |                   |                           |                           |          | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；对于所有未提及的其他输入不合法的情况，输出`Input illegal`。 |
|   add    |        -i         | Dish 主键 DID（保证合法） | m（正整数且保证数据合法） |          | 点餐，将选择的菜品（m 道）加入到对应的 Order 实例中，如果不存在这道菜，输出`Dish selected not exist`；如果菜品售空，输出`Dish selected is sold out`；如果点餐数量超过菜品数量，输出`Dish is out of stock`，本次点餐无效 |
|   add    |        -n         |      Dish 名称 name       | m（正整数且保证数据合法） |          | 点餐，将选择的菜品（m 道）加入到对应的 Order 实例中，如果不存在这道菜，输出`Dish selected not exist`；如果菜品售空，输出`Dish selected is sold out`；如果点餐数量超过菜品数量，输出`Dish is out of stock`，本次点餐无效 |
|  finish  |                   |                           |                           |          | 结束选菜，**检查此时 Order 的菜品数量**，如果菜品数量为 0，输出`Please select at least one dish to your order`，继续接收指令；否则退出到上一层环境中 |

##### 注意

> 在用户调用add的时候，会优先检查一下当前是否存在自己未confirm的Order，如果存在则可以继续追加菜品，否则系统需要自动生成新的order实例，分配对应的orderID
> - 在点菜的时候，不考虑余额是否足够的问题
> - 在点菜的同时，add成功一次，就需要在相应的位置把当前该Dish的总量减m（不存在取消订单的api）

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
- Tony 走进你的餐厅，点了一餐，**没有 confirm**，co(checkOrder) 了一下，觉得不够吃，接着追加了一份披萨，confirm
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

# 当前顾客已登录
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

`co指令：checkOrder`

> 顾客查看当前已经点的菜品列表，并输出当前菜品的总额（这里不考虑打折，保留 1 位小数）

| 指令名 |                             情况                             |
| :----: | :----------------------------------------------------------: |
|   co   | 输出该顾客的当前订单。输出格式见下文示例。若当前顾客没有下单，输出`No order` |

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

[-] co
[+]
1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1
2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:1
3.DID:C000000,DISH:IceCream,PRICE:4.0,NUM:2
4.DID:C000002,DISH:BoBoJi,PRICE:55.0,NUM:1
5.DID:O000100,DISH:Cake,PRICE:74.0,NUM:1
|
SUM:158.0
```

`confirm`

| 指令名  | 情况                                                         |
| :-----: | :----------------------------------------------------------- |
| confirm | 如果当前菜单有效（该Customer有未confirm的菜单），输出`Order Confirmed`，并退出顾客的 login 环境；如果不存在有效菜单，输出`No order can be confirmed`，不退出顾客的 login 环境 |

**方式**：当顾客confirm时，系统为当前订单（Order）自动分配一名服务员。分配算法为：系统检索所有的 Waiter 对象，按照该对象当前负责的 Order 数量（由小到大）以及 WID（由小到大）进行排序并选择第一个 Waiter 来服务当前订单，并将其WID填入订单属性中。

```text
[status : 排序后]
----[WID]----   ----[Order_Count]----
   Wa00009                5
   Wa00011                5
   Wa00007                6
   Wa00004                8

=> 选择Wa00009填入order.WaiterID
```

#### 一种可能的设计方案
```text
# 由于仅初始情况下调用order指令时系统才会生成Order的实例，因此建议：

order指令 -> return Order实例
                   ↓
                   confirm指令: 参数(Order 实例） 
```

#### Waiter

> 在 oms4 中，服务员可以通过`login -i`和`login -n`两种方式进入 login 的环境，但是当前的环境中服务员功能需要进一步扩展，在保留 oms4 的服务员登录后的相关功能前提下，增加下列模块：

|  指令名  | [参数 1/子指令名] |       [参数 2]       | [参数 3] | [参数 4] | 功能                                                         |
| :------: | :---------------: | :------------------: | :------: | :------: | :----------------------------------------------------------- |
| 错误输出 |                   |                      |          |          | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；对于所有未提及的其他输入不合法的情况，输出`Input illegal` |
|    gl    |                   |                      |          |          | 获取当前所服务的 Order 清单(get order list)，单个订单所有菜品的输出还是和原先针对菜品的输出顺序相同（打印的顺序有两个优先级，首先是 H>C>O，其次是 6 位编码从小到大）。详情见下 |
|    mo    |                   |                      |          |          | 分配订单(manage order)，详情见下                             |
|    sr    |   OID（订单ID）   |                      |          |          | 送餐并且收钱(serve and receive money)，完成后该服务员负责的订单数量减一，详情见下 |
|    rw    |   CID（顾客ID）   | money （充入的金额） |          |          | 充值(recharge by waiter)。具体逻辑参见 oms4。**注意**当这次充值达到 VIP 标准后，总金额应该按照八折重新计算 |

`gl指令：getOrderList`

> 查看服务员当前未送到厨房的各个订单（order），以列表形式输出

|  指令名  | 说明                                                         |
| :------: | :----------------------------------------------------------- |
| 错误输出 | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；对于所有未提及的其他输入不合法的情况，输出`Input illegal`。若当前服务员所服务的清单为空，则输出 `No serving order`；若清单不为空，按照**顾客confirm的顺序**进行输出（类似队列），具体输出格式见下 |

```
# 点餐Order输出示例

[-] co
[+]
1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1
2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:2
|
SUM:37.0

# 顾客登出，对应的服务员登录

# 服务员getOrderList输出示例
[-] gl
[+]
1. OID:Cu00000_1,DISH:[1 FanQieChaoDan,2 ChaoTuDouSi]

# 输出的DISH列表中的元素组成：点餐的份数 + 空格 + 菜名，以逗号分隔
```

`mo指令：manageOrder`

> 由于可能有多名顾客分配到当前服务员，因此一个waiter对象负责的order可能不止一个。考虑到服务员可能根据厨房忙碌程度进行流量控制，不会一次将所有订单送到厨房，而是每次一个order，按顾客的confirm的时间顺序送到厨房，即一次命令只能送一个 。Order。

| 指令名 | 说明                                                         |
| :----: | :----------------------------------------------------------- |
|   mo   | 若当前服务员所服务的顾客清单不为空，则将 Order 列表中的第一个 Order送到厨房，输出`Manage order success` |

| 错误输出 | 说明                                                         |
| :------: | :----------------------------------------------------------- |
|          | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；对于所有未提及的其他输入不合法的情况，输出`Input illegal` |
|    mo    | 若当前服务员所服务的顾客清单为空，则输出`No serving order`   |
```
# 顾客查看当前订单(check order)
[-] co
[+]
1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1
2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:1
|
SUM:21.0

# 顾客登出，对应的服务员登录

# mo(manageOrder)输出示例
[-] mo
[+] Manage order success

# 服务员查看Order订单列表(get order list)
[-] gl
[+] No serving order
```



`sr指令：serve and receive money`

> 厨房完成菜品制作后，由服务员送餐，并结账（顾客一拿到菜就要付钱）

若 Order 的状态为未送餐，则送餐成功，否则失败；每一次执行该命令，则向顾客打印订单明细，并结账。至此该订单终结，可从各个表中删除。

| 指令名 |        [参数 1]         | 说明                                                         |
| :----: | :---------------------: | :----------------------------------------------------------- |
|   sr   | OID 订单 ID（保证合法） | 若 Order 的状态为未送餐状态且余额充足，则为正常情况。具体输出格式见下 |

| 错误输出 | 说明                                                         |
| :------: | :----------------------------------------------------------- |
|          | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；对于所有未提及的其他输入不合法的情况，输出`Input illegal`。 |
|          | 若waiter并不负责该订单，输出`Order serve illegal`            |
|          | 若当前订单未被cook，输出`Order not cooked`                   |
|          | 请确保本订单的状态是已完成了cook但未进行结账，若订单的状态为已结账，输出`Order already checkout` |
|          | 计算总价格（普通用户全价，VIP 顾客**打八折**）。若余额不足，则输出`Insufficient balance` |

```
# 点餐Order输出示例
[-] co
[+]
1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1
2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:2
|
SUM:37.0

# 顾客登出，服务员登录，serve and receive money输出示例
[-] sr Cu00000_1
[+] OID:Cu00000_1,DISH:[FanQieChaoDan 5.0,ChaoTuDouSi 32.0],TOTAL:37.0,BALANCE:50.0

# 其中输出DISH中元素组成为：菜名 + 该菜名对应的总价格（菜的单价 * 菜的份数）
# BALANCE表示结账后的余额
```

`rw指令：recharge by waiter`

> 若用户结账时发现自己余额不足，可通过服务员帮助自己充钱

| 指令名 |        [参数 1]         |      [参数 2]      |                             说明                             |
| :----: | :---------------------: | :----------------: | :----------------------------------------------------------: |
|   rw   | CID 顾客 ID（保证合法） | money 具体充入的钱 | 具体逻辑与输出参见 oms4，注意：当这次充钱达到 VIP 标准后，总金额应该按照八折重新计算 |

```
# 点餐Order输出示例
[-] checkOrder
[+]
1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1
2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:2
|
SUM:37.0

# 顾客登出，服务员登录，receiveMoney输出示例
[-] sr Cu00000_1
[+] Insufficient balance

# 余额不足(假设只有20.0元)，服务员帮助充钱
[-] rw Cu00000 100.0
[-] sr Cu00000_1
[+] OID:Cu00000_1,DISH:[FanQieChaoDan 5.0,ChaoTuDouSi 32.0],TOTAL:37.0,BALANCE:83.0
```

#### Kitchen类

~~在实际的餐厅中，Order中每道菜与具体的厨师存在对应关系。也就是Cook对象与Order对象是多对一关系，且上菜顺序、烹饪时间存在差异，可能涉及到优先级调度等复杂问题。~~为了简化问题，降低实现难度， 我们将前面oms中构造的Cook类保留，但仅用于存储每位厨师的账户，不再与做菜直接联系。我们将全体厨师团队视为一个整体，不妨称之为Kitchen类，用于菜品制作。同样地，Order中的菜品不进行拆分，而是将所有菜一起送入厨房，烹饪完成后一起送给顾客。

简而言之，Kitchen的作用是将服务员送来的Order进行处理，**按照order送来的顺序，依次逐个进行处理**。处理后的order由服务员送给顾客，进行其他操作。

同样地，kitchen类通过Cook的账号以`login -i`或`login -n`两种方式进入 login 的环境进行登录。登录后添加以下指令：

> cook指令
>
> 从送来的一串order中拿取一个最早的进行制作。制作完成后，按以下格式输出。若当前厨房没有任务，则什么都不做。

| 指令名 |           成功输出            |
| :----: | :---------------------------: |
|  cook  | Finish order:[某个order的OID] |

```
# 厨房cook指令输出示例

# 假设当前waiter只向厨房送了一个order，为：OID:Cu00000_1,DISH:[1 FanQieChaoDan,2 ChaoTuDouSi]
# 厨房登录
[-] cook
[+] Finish order:Cu00000_1
[-] cook
# 此时厨房已经没有任务，无输出
```

