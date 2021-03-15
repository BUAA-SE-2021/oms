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
|  | | | | |


### 二级指令集