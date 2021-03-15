# OMS05

> 点餐管理系统 05

[![603Lb6.png](https://s3.ax1x.com/2021/03/14/603Lb6.png)](https://imgtu.com/i/603Lb6)

### 任务前瞻

在oms04中，大家编写了点餐系统中十分重要的一些角色类，同时完成了一些指令的实现，此时，我们就可以开始着手实现具体的点餐、服务等环节了。

## 任务

- 将所有List类（PersonList、DishList）改写为单例模式
  + [实现单例模式的常见方法](http://www.blogjava.net/kenzhh/archive/2013/03/15/357824.html)
- 给Dish类中添加可以存放点本菜品名单的属性（建议存储PID）
- 在实现oms04的基础上完善下列指令的完成（务必完成osm04）

### 一级指令集

> 一级指令表示在进入到系统中后的基础环境中时可以使用的指令

| 指令名 | [参数1] | [参数2] | [参数3] | 功能 |
| :---: | :---: | :---: | :---: | :---: |
| SUDO | | | | 在初始的情况下，输入指令`SUDO`进入超级用户模式（现实中只有经理等角色才可以使用，这里省略了权限的认证阶段），进入SUDO模式后，首先输出`Enter sudo mode`，接着可输入sudo模式下的指令，在二级指令集进行说明。 |
| login | -i | 用户ID（PID） | 用户密码 | 通过ID登录，如果当前PersonList不含有该PID，则输出`Pid not exist`，登录成功则输出`Login success`，密码错误输出`Password not match` |
| login | -n | 用户姓名 | 用户密码 | 通过姓名登录，如果当前PersonList不含有该姓名，则输出`Pname not exist`，登录成功则输出`Login success`，密码错误输出`Password not match` |
| QUIT | | | | 退出点餐系统，成功退出则输出`----- Good Bye! -----` |
| | | | | 在一级环境中：如果调用了一级指令集中不包含的指令，输出`Call 1-instruction illegal`；如果参数错误，输出`Params in 1-instruction illegal`；其他错误情况输出`Input illegal` |


### 二级指令集

#### SUDO级

`已存在指令集`

| 选项 | [参数 1] | [参数 2] | [参数 3] | [参数 4] | 功能描述 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| sncu | 新增顾客姓名 | 新增顾客性别 | 新增顾客手机号 | 新增顾客PID | 详见oms04 |
| snwa | 新增服务员姓名 | 新增服务员性别 | 新增服务员手机号 | 新增服务员PID | 详见oms04 |
| snco | 新增厨师姓名 | 新增厨师性别 | 新增厨师手机号 | 新增厨师PID | 详见oms04 |
| dcu | 目标顾客PID |  |  |  | 详见oms04 |
| dwa | 目标服务员PID |  |  |  | 详见oms04 |
| dco | 目标厨师PID |  |  |  | 详见oms04 |
| pp | | | | | 详见oms04 |
| SQ | | | | | 退出超级用户状态，输出`Quit sudo mode`，返回初始环境 |

`新增指令集`

| 选项 | [参数 1] | 功能描述 |
| :---: | :---: | :---: |
| pd | -c | 以数量为顺序打印当前所有被点菜品信息，格式化信息输出如下面说明所示 |
| pd | -t | 以类型名称为顺序打印当前所有被点菜品信息，格式化信息输出如下面说明所示 |

- 仅打印被点的菜品，打印顺序由[参数1]指定
  + -c 表示按照该菜品被点数量由多到少排序输出
  + -t 表示按照该菜品类型的顺序（H>C>O，每一种类型内按照DID由小到大）排序输出
- 输出样例（-c）
  ```
  # 假设当前的点餐菜品列表如下（非格式化表示）
  ↓
  DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,TOTAL:30,ORDERD:10
  DID:H000001,DISH:FoTiaoQiang,PRICE:200.0,TOTAL:15,ORDERD:1
  DID:C000000,DISH:IceCream,PRICE:2.5,TOTAL:100,ORDER:50
  ↓
  输入指令：
  [-] SUDO
  [+] Enter sudo mode
  [-] pd -c
  [+] 1. DID:C000000,DISH:IceCream,PRICE:2.5,TOTAL:100,ORDER:50
      2. DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,TOTAL:30,ORDERD:10
      3. DID:H000001,DISH:FoTiaoQiang,PRICE:200.0,TOTAL:15,ORDERD:1
  [-] pd -t
  [+] 1. DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,TOTAL:30,ORDERD:10
      2. DID:H000001,DISH:FoTiaoQiang,PRICE:200.0,TOTAL:15,ORDERD:1
      3. DID:C000000,DISH:IceCream,PRICE:2.5,TOTAL:100,ORDER:50
  ```
  
#### login级

在完善超级用户的指令集后，我们需要