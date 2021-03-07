# OMS04

> 点餐管理系统 04

## 写在前面
⼤家对Java编程已经逐渐熟悉，现在我们开始减少对于实现⽅式的要求，只对测试接⼝进⾏严格要求。所有⽅法我们只规定需要实现，实现⽅式等不做要求，但请将代码写规范，因为**有⼀部分分会给在代码⻛格上**。接⼝和⽂件格式⼀定要写正确，⽆法测试会直接导致分数较低。

## 题目背景
经过大家辛苦的开发，目前我们的点餐系统已经实现了较为成熟的菜单的查看以及菜品的查找，当然，只有菜品菜单是不够的，饭店的正常运营，一些角色是无法或缺的，此次oms的实现主要侧重于这些类的编写。

## 任务

### Person

此次功能新增会对oms01已经编写的`Person`类进行修改，说明如下：

该类是下面几个角色类的父类。为Person类新增主键属性PID（String类型），对应的密码为PWD（String类型），两者的组成结构有严格的要求，具体要求如下：

- 对于PID：
```
PID = 两位身份符号（Cu/Wa/Bo/Co） + 5位整数编号
例如：Cu00000、Wa01000
```
- 对于PWD：
```
  1. PWD的长度限制在[8,18]
  2. PWD中至少含有英文字母以及罗马数字（即不允许只出现数字或字母）
  3. 只允许出现ASCII码表中序号在[33,126]中的字符
  4. 在角色被新创建时，为其生成默认密码oms1921
```

### Customer
`继承Person类`

- 添加属性isVIP（boolean类型），表示当前顾客是否为VIP用户
- 添加属性balance（double类型），表示当前该顾客账户的余额
- 添加属性isDining（boolean类型），表示当前顾客是否在店用餐
- 添加属性当前点餐清单（列表）
- 添加属性当前未上菜清单（列表）

### Waiter
`继承Person类`

- 添加属性当前所服务的顾客清单（列表，具体存储内容请自行判断，但必须保证含有Person类的主键）

### Cook
`继承Person类`

- 添加属性当前完成菜品清单（Dish对象列表）

### PersonList
编写PersonList类存放当前点餐系统的所有人的信息，包括当前系统中所有的Customer、Waiter、Cook。

## 功能新增

此次功能新增依旧是在前面的oms的基础上进行，请务必完成oms03后再编写此次实验。同时，大家可能在这次实验对自己的程序进行一小部分的重构，请务必重视。

### SUDO模式的引入

为了引入权限模式，此次功能更新会引入SUDO指令，来模拟有权限的终端系统。在初始的情况下，输入指令`SUDO`进入超级用户模式（现实中只有经理等角色才可以使用，这里省略了权限的认证阶段），进入SUDO模式后，首先输出`Enter sudo mode`，其次输入sudo模式下的指令，如下所示：

基本格式：

```shell
选项 [参数1] [参数2] [参数3] [参数4]
```

| 选项 | [参数 1] | [参数 2] | [参数 3] | [参数 4] | 功能描述 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| sncu | 新增顾客姓名 | 新增顾客性别 | 新增顾客手机号 | 新增顾客PID | 详见新增功能描述1 |
| snwa | 新增服务员姓名 | 新增服务员性别 | 新增服务员手机号 | 新增服务员PID | 详见新增功能描述1 |
| snco | 新增厨师姓名 | 新增厨师性别 | 新增厨师手机号 | 新增厨师PID | 详见新增功能描述1 |
| dcu | 目标顾客PID |  |  |  | 详见新增功能描述2 |
| dwa | 目标服务员PID |  |  |  | 详见新增功能描述2 |
| dco | 目标厨师PID |  |  |  | 详见新增功能描述2 |
| pp | | | | | 详见新增功能描述3 |
| SQ | | | | | 退出超级用户状态，输出`Quit sudo mode`，返回初始环境 |

#### 新增功能描述1
本组新增功能的目的是实现超级用户向点餐系统中录入各类角色信息（顾客、服务员、厨师），在录入信息时，需要对各参数的合法性进行检查，同时保证主键的不重复性（目前可以认为姓名、手机号、PID均不可重复）。

| 指令名 | 成功输出 |
| :---: | :---: |
| sncu | Add new customer success |
| snwa | Add new waiter success |
| snco | Add new cook success |

错误情况的输出如下，且相同指令的错误判断优先级由上到下进行

| 指令名 | 错误情况 | 错误输出 |
| :---: | :---: | :---: |
| 错误指令 | | Call sudo method illegal |
| sncu | 参数错误 | Arguments illegal |
| sncu | 顾客输入性别错误（非M/F） | Sex illegal |
| sncu | 顾客输入手机号格式错误 | Phone number illegal |
| sncu | 顾客输入性别与手机号不匹配 | Phone number doesn't match sex |
| sncu | 新增顾客PID格式错误 | Customer PID illegal |
| sncu | 新增顾客PID已存在 | Customer PID exists |
||||
| snwa | 参数错误 | Arguments illegal |
| snwa | 服务员输入性别错误（非M/F） | Sex illegal |
| snwa | 服务员输入手机号格式错误 | Phone number illegal |
| snwa | 服务员输入性别与手机号不匹配 | Phone number doesn't match sex |
| snwa | 新增服务员PID格式错误 | Waiter PID illegal |
| snwa | 新增服务员PID已存在 | Waiter PID exists |
||||
| snco | 参数错误 | Arguments illegal |
| snco | 厨师输入性别错误（非M/F） | Sex illegal |
| snco | 厨师输入手机号格式错误 | Phone number illegal |
| snco | 厨师输入性别与手机号不匹配 | Phone number doesn't match sex |
| snco | 新增厨师PID格式错误 | Cook PID illegal |
| snco | 新增厨师PID已存在 | Cook PID exists |

建议大家把重复的判断封装成一个方法，进行调用而不是简单的if-else的复制粘贴。

#### 新增功能描述2
本组新增功能的目的是实现超级用户对系统角色信息的删除操作，在删除时需要指定删除对象的PID，自然也需要对该PID进行合法性的检查（格式是否正确、是否存在）。

| 指令名 | 成功输出 |
| :---: | :---: |
| dcu | Delete customer success |
| dwa | Delete waiter success|
| dco | Delete cook success |


| 指令名 | 错误情况 | 错误输出 |
| :---: | :---: | :---: |
| 错误指令 | | Call sudo method illegal |
| dcu | 参数错误 | Arguments illegal |
| dcu | 指定PID错误 | D-Customer PID illegal |
| dcu | 指定PID不存在 | D-Customer PID doesn't exist |
||||
| dwa | 参数错误 | Arguments illegal |
| dwa | 指定PID错误 | D-Waiter PID illegal |
| dwa | 指定PID不存在 | D-Waiter PID doesn't exist |
||||
| dco | 参数错误 | Arguments illegal |
| dco | 指定PID错误 | D-Cook PID illegal |
| dco | 指定PID不存在 | D-Cook PID doesn't exist |

#### 新增功能描述3
为了方便超级用户查看当前系统的所有用户信息，新增指令pp用来输出系统角色信息列表（格式化）。输出的顺序依旧有两层优先级，首先按照Customer > Waiter > Cook排列，其次则是PID的排序，序号从1开始。

注意，如果当前角色列表为空，则输出`Empty person list`。

输出的格式如下所示：
```
[-] SUDO
[+] Enter sudo mode
[-] pp
[+] Empty person list
[-] sncu ZhangSan M 13766660310 Cu00000
[+] Add new customer success
[-] pp
[+] 1. PID:Cu00000,Name:ZhangSan,Sex:M,Phone:13766660310,PWD:oms1921
```