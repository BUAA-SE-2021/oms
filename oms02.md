# OMS02

> 点餐管理系统 02

## 题目背景

干饭人上线！既然是点餐系统，当然要有菜单啦~（大家在做实验之前记得先仔细阅读实验文档，在开始动手，以免返工）

## 提示
由于点餐系统逐渐复杂，输出也变得更多了起来，因此在大家完成本次任务时，请认真阅读任务部分以及功能部分，防止遗漏输出情况。

## 任务

注意：以下编写的方法是否为静态，可根据自己实现的方式进行选择，我们不作限制，但请选择最契合面向对象编程的逻辑方式进行。

### Dish

编写一个 Dish 类，作为该餐厅的菜品：

- 属性包括菜品名称（String 类型）、菜品编号 DID（String类型，**菜品主键**，其组成后面会详细说明）、价格（Double / double 类型）、总量（Integer / int类型，因为食材是有限的）

- 编写方法 `String toString()`，输出 Dish 的格式化信息，要求格式如下：

  - 注意冒号和逗号使用英文字符 ':' 和 ','

  - 注意：输出时标点符号间不要有空格，例如： 
    ```shell
    DID:H000011,DISH:FISH,PRICE:38.5,TOTAL:100
    ```
  - 保证所有信息合法，比如：该菜品的总量不能为负数等

- 添加任何你认为所需要的属性和方法

- 以上所有属性均为**私有**属性，需提供相应的 getter 和 setter 方法

### Menu

编写一个 Menu 类，作为该餐厅的菜单，包括所有的菜品（封装对菜品的操作），并且菜单分为热菜 H、凉菜 C 和其他 O 三个部分（菜品编号组成为：菜品类型 + 6 位整数编号，并且类型字母为大写）：
  ```shell
  例如：H000000
  ```
- 提供默认构造方法，构造一个空的菜单

- 编写查询菜品相关信息的方法
  - #### Dish getDishById(String did)
    - 根据菜品的编号 DID ，查询对应的菜品（需要自行判断DID的合法性） 
    - 如果查找到指定的菜品，返回目标Dish对象，并输出该菜品的格式化信息
    - 如果不存在该菜品，则返回 `null`，输出对应信息
  - #### Vector\<Dish\> getDishByKeyWord(String keyword)
    - 根据关键字检索，返回所有菜品名字中包含有该关键字的菜品（注意：检索时不区分大小写）
    - 返回类型为 Dish 的容器（数组、List、Map 等），同时输出格式化信息
    - 如果不存在这样的菜品，则返回一个空容器，并输出对应信息
    - 合法的关键字是一个连续的没有空白符的字符串
- 添加新的菜品 `nd`
  - 如果这种菜品不存在于列表中（用菜品编号DID进行检验），则将其所有的信息添加进菜单
  - 如果这种菜品已经存在（DID或名称存在），则输出`Dish exists` （详细说明见下方表格）
  - 请保证录入的数据是合法的，例如：
    - 修改后菜单中的菜品总数不能为负数
    - 菜品的价格不能为负数
    - 信息不合法时，要保证数据不被修改，并输出`New dish input illegal`（详细说明见下方表格）
- 修改指定的菜品信息 `udd`
  - 修改的菜品应当是已经存在于菜单中的
  - 请保证修改的信息是合法的
  - 如果输入的修改信息不合法，请输出`Change dish illegal`
  - 如果被修改的是能唯一标识某一种菜品的属性（比如菜品的编号、名称）,那么需要保证修改这个属性之后不与已经存在的其他菜品重复，如果信息重复则判定为不合法，输出`Change dish repeated`
  - 关于修改的信息的合法状态的详细情况请**查看表格**
- 概览整本菜单的信息 `pm`
  - 打印目前菜单里所有菜品的信息
  - 打印的顺序有两个优先级，首先是H>C>O，其次是6位编码从小到大
  - 打印序号从1开始，注意形式为`"1." + [空格] + 具体信息` 
  - 如果当前菜单内容为空，则输出`Empty Menu`
- 以上功能都视作菜单 Menu 对外提供的获取/修改数据的 API，是后续实验的基础

### 功能新增

在**上次的 Test 类基础上**（请先完成点餐系统 1），增加以下功能：

- 在 main 方法中创建⼀个空的 Menu 对象

- 在 main 方法中实现⼀个简易命令行终端对上述 Menu 对象进行操作，要求：

  
  输入的基本格式如下

  ```shell
  选项 [参数1] [参数2] [参数3] [参数4]
  ```

  注意：所有输出中存在的标点符号都是英文符号，且各种情况的优先判断次序按照说明进行

  | 选项 | [参数 1] | [参数 2] | [参数 3]     | [参数 4] | 功能描述                                                                                                                                                                        |
  | ---- | -------- | :------: | ------------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  |      |          |          |              |          | 若输入指令不存在（选项名或者选项中的选择性参数不存在均认为指令不存在，例如 *gd -not* 不存在），输出`Command not exists`；若参数数量不正确，输出`Params' count illegal`；对于所有未提及的其他输入格式错误以及不合法的情况，输出`Input illegal`。                                                                                                                              |
  | gd   | -id      | 菜品编号 |              |          | 在参数（数量、格式）合法的前提下，根据菜品编号查找菜品，如果Did格式错误，输出`Did input illegal`；如果菜品不存在，则输出`Dish does not exist`；如果存在，则调用该菜品的 toString 方法进行输出。                                                             |
  | gd   | -key     |  关键字  |              |          | 在参数（数量、格式）合法的前提下，根据菜品名字查找菜品（测试保证key合法），如果菜品不存在，则输出`Dish does not exist`；如果存在，则根据菜品类型热菜 H、凉菜 C 和其他 O 的顺序输出，并且这三个分类中的编号应按照从小到大的格式输出，编号从1开始，具体格式请看文末示例。 |
  | udd  | 菜品编号 |    -n    | 名字         |          | 在参数（数量、格式）合法的前提下，修改指定菜品编号的名称，如果Did格式错误，输出`Did input illegal`；如果没有菜品存在，则输出`Dish does not exist`。注意，名称的合法情况仅包括`数字 + 英文字母大小写`（位置不作强制要求），其他情况均非法，输出`New name input illegal`；如果新名称与原名称重复，输出`New name repeated`；成功修改后输出`Update dish's name success`。                                                                        |
  | udd  | 菜品编号 |    -t    | 该菜品的总量 |          | 在参数（数量）合法的前提下，修改指定菜品编号的总量，如果Did格式错误，输出`Did input illegal`；如果没有菜品存在，则输出`Dish does not exist`；如果输入修改后总量非法（**合法情况仅为非负整数**），输出`Change dish's total illegal`；成功后输出`Update dish's total success`。                                                                        |
  | udd  | 菜品编号 |    -p    | 该菜品的价格 |          | 在参数（数量）合法的前提下，修改指定菜品编号的价格，如果Did格式错误，输出`Did input illegal`；如果没有菜品存在，则输出`Dish does not exist`；如果输入修改后价格非法（**合法情况仅为非负浮点数**），输出`Change dish's price illegal`；成功后输出`Update dish's price success`。                                                                        |
  | nd   | 菜品编号 | 菜品名字 | 菜品价格 | 菜品总量 | 在参数（数量、格式）合法的前提下，添加⼀个菜品到对应类型的 Menu 中，如果Did格式错误，输出`Did input illegal`；如果菜品已经存在,则输出`Dish exists`。如果不存在则判断数据是否合法，非法输出`New dish's attributes input illegal`，若均合法则判断菜品名称是否重复，若重复则输出`Name repeated`；成功则添加至列表并输出`Add dish success`。                                                        |
  | pm   |   |   |  |   | 打印当前菜单的所有菜品情况，具体的打印顺序参考Menu类的编写说明；注意，如果当前Menu没有菜品，打印`Empty Menu`。 |

说明一： 关于数字属性（例如菜品价格、总量）的格式错误和数据合法性，可以理解为
> 如果该属性错误输入为了其他类型例如字符串，则视为格式错误；如果该属性为数值类型（int或double等）但是数值大小不符合实际情况（比如菜品价格为负数）则认为数据不合法。
> 
> 如果没有特殊的说明，在oms中格式错误和非法认为是一种情况，按指示输出

说明二：`在参数（数量、格式）合法的前提下` - 这句话是指在已经完成了参数数量和格式的合法检查后的后继操作，因此优先级更高的是指令名称存在与否、参数数量是否合法、各参数格式是否合法（但凡存在一个参数格式错误，就视为参数格式错误）。

### 注意
今后若无具体说明，则oms对每一条指令的合法性判断顺序应为：**指令（选项/参数0）名称在环境中存在与否 > 参数数量是否正确 > 参数格式正确性以及合法性**；在本次的输出中，参数调用失败以及参数数量的错误输出如上表最后所示。
```
{我们建议的指令编写逻辑框架（非java代码，请勿复制使用，以 np 为例）}

[-] you get a string as an operational statement
|
↓
String[] / other vector<String> ops
|
| // {当然为了防止ops访问越界，我们建议你确保ops.len > 0后再进行下面的操作}
↓
// 指令（选项/参数0）名称在环境中存在与否
if (ops[0] === new String("nd")) {
  // 参数数量是否正确
  if (ops.len == 4) {
    // 参数合法性
    if (checkDID(ops[1])) {
      if (dishExisted(ops[1])) {
        // 当然这里不能这样写，因为输入的菜品价格和总量都有可能出现格式错误，你可以通过预检查或者异常处理来编写你的代码
        // 这里使用 && 是为了展示这些方法是短路判断的，存在一处不合法就认为不合法
        if (checkDishName(ops[2]) && 
            checkDishPrice(ops[3]->int) && 
            checkDishTotal(ops[4]->int)) {
          call: addDish(...);
          print("Add dish success");
        } else {
          print("New dish's attributes input illegal\n");
        }
      } else {
        print("Dish exists\n");
      }
    } else {
      print("Did input illegal");
    }
  } else {
    print("Params' count illegal\n");
  }
} else {
  print("Command not exists\n");
}

```

输入[-]输出[+]示例：

`[+]/[-]仅仅是提示作用，实际输出并不包括`

```
[-] pm
[+] Empty Menu
[-] nd H000000 FanQieChaoDan 5.0 30
[+] Add dish success
[-] pm
[+] 1. DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,TOTAL:30
[-] nd H000000 MaoXueWang 15.0 10
[+] Dish exists
[-] nd O000000 Cola 5.0 200
[+] Add dish success
[-] nd C000000 IceCream 4.0 100
[+] Add dish success
[-] pm
[+] 1. DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,TOTAL:30
    2. DID:C000000,DISH:IceCream,PRICE:4.0,TOTAL:100
    3. DID:O000000,DISH:Cola,PRICE:5.0,TOTAL:200
[-] gd -id H000001
[+] Dish does not exist
[-] gd -id H000000
[+] DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,TOTAL:30
[-] gd -key c
[+] 1. DID:H000000,DISH:FanQieChaoDan,PRICE:5.0,TOTAL:30
    2. DID:C000000,DISH:IceCream,PRICE:4.0,TOTAL:100
    3. DID:O000000,DISH:Cola,PRICE:5.0,TOTAL:200
[-] udd H000000 -n XiHongShiChaoJiDan
[+] Update dish's name success
[-] pm
[+] 1. DID:H000000,DISH:XiHongShiChaoJiDan,PRICE:5.0,TOTAL:30
    2. DID:C000000,DISH:IceCream,PRICE:4.0,TOTAL:100
    3. DID:O000000,DISH:Cola,PRICE:5.0,TOTAL:200
[-] dish
[+] Command not exists
[-] quit
[+] Command not exists
[-] QUIT
[+] ----- Good Bye! -----
```

## 提交格式注意

务必按照规定格式提交，特别是src文件夹以及其中的Test.java文件，这两者是不可或缺的，是测评时拉取的核心资源，如果按照非合法格式提交而导致的**最终测评**得分较低，自行负责。
