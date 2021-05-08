# OMS05

> 点餐管理系统 05

## 任务前瞻

在 oms04 中，大家编写了点餐系统中十分重要的一些角色类，同时完成了一些指令的实现，此时，我们就可以开始着手实现具体的点餐、服务等环节了。
当然，由于 oms 并不采用多线程的写法，所以也无法还原一个真实的场景（例如用户点餐的同时厨师在做菜），所以本次 oms 需要频繁切换角色的登录并进行相关的操作。

本次 oms 的编写计划完成所有的功能，因此也是有编码需求的最后一次 oms，望大家认真对待。

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

#### Dish 类

- 加入`CookList`属性，存储和 Cook 相关的主键（推荐存储 ID，当然直接存储一个对象也是可以的，取决于个人的实现，只要保证**数据的一致性**即可），不要求新建 CookList 类并使用单例模式
- 加入`time`属性，为 double 类的数据，代表该菜品的耗时（单位：小时），默认值 0.0（保留一位小数）

#### Order 类

> 大家在学数据库的时候应该已经接触过两个类之间的关系表，Order 就是这样一个的存在，作为`Customer`和`Menu`的一个关系表，存储着顾客本次的点菜

Order 的基本属性如下：

- CustomerID `表明是哪位顾客所点的菜品`
- WaiterID `表明本次是由哪位服务员进行接单以及送餐服务的（类似海底捞）`
- DishList `顾客所点菜品列表`
- isTakeOrder `是否有服务员接单`
- isConfirm `顾客是否确认菜单`
- isServe `菜品制作完成后，是否送餐`

**注意：**

- Order 是有一定的生存周期的，当顾客选择开始点餐时（见下），系统自动生成一张 Order 关系表，存储顾客的点餐情况，在顾客结账和服务员进行找零后自动删除（oms 餐厅不具备数据长期存储功能）

#### Customer

> 在 oms4 中，顾客可以通过`login -i`和`login -n`两种方式进入 login 的环境，但是当前的环境中功能需要进一步扩展，在保留 oms4 的相关功能的前提下，增加下列模块：
>
> - 顾客在进行点餐前可以进行菜品和菜单的查询，因此继承了前些版本的查询功能

|   指令名   | [参数 1/子指令名] |   [参数 2]    | [参数 3] |   [参数 4]    |                                                                                 功能                                                                                  |
| :--------: | :---------------: | :-----------: | :------: | :-----------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|  错误输出  |                   |               |          |               | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；**对于所有未提及的其他输入不合法的情况，输出`Input illegal`**。 |
|     rc     |      金额 m       |               |          |               |                                                                                同 oms4                                                                                |
|     gb     |                   |               |          |               |                                                                                同 oms4                                                                                |
|   aplVIP   |                   |               |          |               |                                                                                同 oms4                                                                                |
|     gd     |        -id        |   菜品编号    |          |               |                                                                                同 oms2                                                                                |
|     gd     |       -key        |    关键字     |          |               |                                                                                同 oms2                                                                                |
|     gd     |       -key        |    关键字     | 第 n 页  | 每页 m 个记录 |                                                                                同 oms3                                                                                |
|     pm     |                   |               |          |               |                                                                                同 oms2                                                                                |
|     pm     |      第 n 页      | 每页 m 个记录 |          |               |                                                                                同 oms3                                                                                |
|   order    |                   |               |          |               |                                                                        进入点餐环境，详情见下                                                                         |
| checkOrder |                   |               |          |               |                                                                        查看已点菜品，详情见下                                                                         |
|  confirm   |                   |               |          |               |                                                                          提交菜单，详情见下                                                                           |
| checkLeft  |                   |               |          |               |                                                                        查看未上菜品，详情见下                                                                         |
|  checkOut  |                   |               |          |               |                                                                            结账，详情见下                                                                             |

`order点餐环境`

> 和 login、sudo 等环境的组成类似，该环境是隶属于 login 的子环境，用户在该环境下进行点餐，并使用该环境下的指令

**在该环境中，循环读取指令，只接受 add 和 finish**

|  指令名  | [参数 1/子指令名] |                 [参数 2]                 |         [参数 3]          | [参数 4] |                                                                                                             功能                                                                                                             |
| :------: | :---------------: | :--------------------------------------: | :-----------------------: | :------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| 错误输出 |                   |                                          |                           |          |                            如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；**对于所有未提及的其他输入不合法的情况，输出`Input illegal`**。                             |
|   add    |        -i         |              Dish 主键 DID               | m（正整数且保证数据合法） |          | 点餐，将选择的菜品（m 道）加入到对应的 Order 实例中，如果不存在这道菜，输出`Dish selected is not exist`；如果菜品售空，输出`Dish selected is sold out`；如果点餐数量超过菜品数量，输出`Dish is out of stock`，本道菜点餐无效 |
|   add    |        -n         | Dish 名称 name（不考虑名称的重复和非法） | m（正整数且保证数据合法） |          | 点餐，将选择的菜品（m 道）加入到对应的 Order 实例中，如果不存在这道菜，输出`Dish selected is not exist`；如果菜品售空，输出`Dish selected is sold out`；如果点餐数量超过菜品数量，输出`Dish is out of stock`，本道菜点餐无效 |
|  finish  |                   |                                          |                           |          |                                       结束选菜，检查此时 Order 的菜品数量，如果菜品数量为 0，输出`Please select at least one dish to your order`，继续接收指令；否则退出到上一层环境中                                       |

**注意**：当顾客第一次进入到 order 环境中，由系统自动生成一张菜单（Order 实例），此时系统会检索所有的 Waiter 对象，按照该对象当前负责的 Order 数量（由小到大）以及 WID（由小到大）进行排序并选择第一个 Waiter 的 WID，填入该菜单中。

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
1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1,SERVE:NO,SERVE_NUM:0
2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:1,SERVE:NO,SERVE_NUM:0
3.DID:C000000,DISH:IceCream,PRICE:4.0,NUM:2,SERVE:NO,SERVE_NUM:0
4.DID:C000002,DISH:BoBoJi,PRICE:55.0,NUM:1,SERVE:NO,SERVE_NUM:0
5.DID:O000100,DISH:Cake,PRICE:74.0,NUM:3,SERVE:NO,SERVE_NUM:0

# SERVE表示是否送餐，值为NO表示没有送餐，YES表示已送餐
```

`confirm`

> 确认菜单，交给服务员进行下一步的操作，此时**默认退出顾客的 login 环境**

---

#### Waiter

> 在 oms4 中，服务员可以通过`login -i`和`login -n`两种方式进入 login 的环境，但是当前的环境中服务员功能需要进一步扩展，在保留 oms4 的服务员登录后的相关功能前提下，增加下列模块：

|     指令名     | [参数 1/子指令名] | [参数 2] | [参数 3] | [参数 4] |                                                                                 功能                                                                                  |
| :------------: | :---------------: | :------: | :------: | :------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|    错误输出    |                   |          |          |          | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；**对于所有未提及的其他输入不合法的情况，输出`Input illegal`**。 |
|   takeOrder    |                   |          |          |          |                                                                         服务员接单，详情见下                                                                          |
| getServeCuList |                   |          |          |          |                                                                  获取当前所服务的顾客清单，详情见下                                                                   |
|   manageDish   |                   |          |          |          |                                                                          分配菜品，详情见下                                                                           |
|   serveDish    |                   |          |          |          |                                                                            送餐，详情见下                                                                             |
|  receiveMoney  |                   |          |          |          |                                                                            收钱，详情见下                                                                             |

`takeOrder`

> 当顾客点单完成后，服务员接单

当服务员接单时，需要查看 oms 系统中的 Order 对象的相关状态。只有在顾客已经确认菜单并且没有其他服务员接单的情况下，才可以接单成功。

|  指令名   |       成功输出       |
| :-------: | :------------------: |
| takeOrder | `Take order success` |

> 注意：错误输出的优先级如下先后顺序所示

|  指令名   |                                                                               错误输出                                                                                |
| :-------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| 错误输出  | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；**对于所有未提及的其他输入不合法的情况，输出`Input illegal`**。 |
| takeOrder |                                                       顾客还未确认菜单，输出 `Customer didn't confirm the menu`                                                       |
| takeOrder |                                                    已经有其他服务员接单，输出 `Other waiters Already taken orders`                                                    |

- 情况 1

  ```
  # 点餐Order输出示例

  [-] checkOrder
  [+]
  Customer doesn't confirm
  No other waiters take orders
  1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1,SERVE:NO,SERVE_NUM:0
  2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:1,SERVE:NO,SERVE_NUM:0

  # 顾客登出，服务员登录，接单输出示例
  [-] takeOrder
  [+]
  Customer didn't confirm the menu
  ```

- 情况 2

  ```
  # 点餐Order输出示例

  [-] checkOrder
  [+]
  Customer has confirmed
  Other waiters have taken orders
  1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1,SERVE:NO,SERVE_NUM:0
  2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:1,SERVE:NO,SERVE_NUM:0

  # 顾客登出，服务员登录，接单输出示例
  [-] takeOrder
  [+]
  Other waiters Already taken orders
  ```

- 情况 3

  ```
  # 点餐Order输出示例

  [-] checkOrder
  [+]
  Customer has confirmed
  No other waiters take orders
  1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1,SERVE:NO,SERVE_NUM:0
  2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:1,SERVE:NO,SERVE_NUM:0

  # 顾客登出，服务员登录，接单输出示例
  [-] takeOrder
  [+]
  Take order success
  ```

`getServeCuList`

> 当前服务员所服务的顾客清单

|     指令名     |                                         成功输出                                         |
| :------------: | :--------------------------------------------------------------------------------------: |
| getServeCuList |     若当前服务员所服务的顾客清单不空，则输出 `The list of customers served is empty`     |
| getServeCuList | 若当前服务员所服务的顾客清单不为空，按照顾客 ID 从小到大的顺序进行输出，具体输出格式见下 |

|  指令名  |                                                                               错误输出                                                                                |
| :------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| 错误输出 | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；**对于所有未提及的其他输入不合法的情况，输出`Input illegal`**。 |

```
# 点餐Order输出示例

[-] checkOrder
[+]
Customer has confirmed
No other waiters take orders
1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1,SERVE:NO,SERVE_NUM:0
2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:1,SERVE:NO,SERVE_NUM:0

# 顾客登出，服务员登录，接单输出示例
[-] takeOrder
[+]
Take order success

# getServeCuList输出示例
[-] getServeCuList
[+]
1. PID:Cu00000,Name:zhangSan,Phone:13766660310,Ordered:[1 FanQieChaoDan Li,1 ChaoTuDouSi Zhang]

# Ordered列表中的元素表示：数字（点餐份数）+ 字符串（菜名）+ 字符串（厨师名）
# Ordered列表中的元素输出顺序也应该和原先针对菜品的输出顺序一样
```

`manageDish`

> 服务员接单后按照厨师来分配菜品

每当服务员接到一单时，根据顾客所点的菜品中所对应的厨师来进行分配，这样方便交给各个厨师来制作。

|   指令名   |                                            成功输出                                            |
| :--------: | :--------------------------------------------------------------------------------------------: |
| manageDish | 若当前服务员所服务的顾客清单不为空，按照顾客所点的菜品中的厨师来进行分配输出，具体输出格式见下 |

|   指令名   |                                                                               错误输出                                                                                |
| :--------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|  错误输出  | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；**对于所有未提及的其他输入不合法的情况，输出`Input illegal`**。 |
| manageDish |                                                若当前服务员所服务的顾客清单为空，则输出`Can not manage empty dishList`                                                |

```
# 点餐Order输出示例
[-] checkOrder
[+]
Customer has confirmedNo
other waiters take orders
1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1,SERVE:NO,SERVE_NUM:0
2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:1,SERVE:NO,SERVE_NUM:0

# 顾客登出，服务员登录，接单输出示例
[-] takeOrder
[+] Take order success

# manageDish输出示例
[-] manageDish
[+]
1.PID:Cu00000,Name:zhangSan,Phone:13766660310,Dish:FanQieChaoDan,NUM:1,COOK:Li
2.PID:Cu00000,Name:zhangSan,Phone:13766660310,Dish:ChaoTuDouSi,NUM:1,COOK:Zhang

# 注意：输出顺序按照厨师名字的先后顺序进行输出
```

`serveDish`

> 厨师完成菜品制作后，由服务员送餐

服务员送餐，若菜品的状态为未送餐，则送餐成功，否则失败

|  指令名   |  [参数 1]   |  [参数 2]   | [参数 3]         | 成功输出             |
| :-------: | :---------: | :---------: | ---------------- | -------------------- |
| serveDish | PID 顾客 ID | DID 菜品 ID | Num 所送餐的数量 | `Serve dish success` |

> 注意：若成功送餐，则需要修改 Order 中已上菜品数量 SERVE_NUM

|  指令名   |  [参数 1]   |  [参数 2]   | [参数 3]         |                                                                               失败输出                                                                                |
| :-------: | :---------: | :---------: | ---------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| 错误输出  | PID 顾客 ID | DID 菜品 ID | Num 所送餐的数量 | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；**对于所有未提及的其他输入不合法的情况，输出`Input illegal`**。 |
| serveDish | PID 顾客 ID | DID 菜品 ID | Num 所送餐的数量 |                                                             若菜品的状态为已送餐，输出`Have Served dish`                                                              |
| serveDish | PID 顾客 ID | DID 菜品 ID | Num 所送餐的数量 |                                                        若所送餐的数量大于需要的菜品数量，输出`Serve Much dish`                                                        |

`receiveMoney`

> 当所服务的顾客清单中某个顾客所点的菜品的已上菜的数量等于所点的数量，则收钱

|    指令名    |  [参数 1]   |   成功输出   |
| :----------: | :---------: | :----------: |
| receiveMoney | PID 顾客 ID | Total:总金额 |

|    指令名    |  [参数 1]   |                                                                               失败输出                                                                                |
| :----------: | :---------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|   错误输出   | PID 顾客 ID | 如果输入的指令名不属于该环境，输出`Command not exist`；若参数数量不正确，输出`Params' count illegal`；**对于所有未提及的其他输入不合法的情况，输出`Input illegal`**。 |
| receiveMoney | PID 顾客 ID |                                                     若顾客所点的菜还没有上完，则输出`The dish is not served yet`                                                      |

```
# 点餐Order输出示例
[-] checkOrder
[+]
Customer has confirmed
No other waiters take orders
1.DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,NUM:1,SERVE:YES,SERVE_NUM:1
2.DID:H000001,DISH:ChaoTuDouSi,PRICE:16.0,NUM:1,SERVE:YES,SERVE_NUM:1

# 顾客登出，服务员登录，receiveMoney输出示例
[-] receiveMoney
[+] Total:21.0
```
