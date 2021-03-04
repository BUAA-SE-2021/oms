# OMS01

> 点餐管理系统 01

## 题目背景

点餐系统需要有基本用户，每位用户在系统内有个人信息，包括用户名、性别、手机号等。其中手机号需要满足一定的格式，因此需要对手机号进行校验。

根据我国手机号码的规定，长度为 11 位，由 0-9 阿拉伯数字组成，各段有不同的含义和编码规则：

第 1-3 位：网络识别号
第 4-7 位：地区编码
第 8-11 位：用户号码

**手机号码的格式：**

<div style="display: flex; flex-direction: row; text-align: center; font-family: 'Fira Code'">
    <div style="border: firebrick solid 1px; width: 30px; height: 25px; background: #ffe9e9">1</div>
    <div style="border: firebrick solid 1px; width: 30px; height: 25px; background: #ffe9e9">2</div>
    <div style="border: firebrick solid 1px; width: 30px; height: 25px; background: #ffe9e9">3</div>
    <div style="border: firebrick solid 1px; width: 30px; height: 25px; background: #cdf6e7">4</div>
    <div style="border: firebrick solid 1px; width: 30px; height: 25px; background: #cdf6e7">5</div>
    <div style="border: firebrick solid 1px; width: 30px; height: 25px; background: #cdf6e7">6</div>
    <div style="border: firebrick solid 1px; width: 30px; height: 25px; background: #cdf6e7">7</div>
    <div style="border: firebrick solid 1px; width: 30px; height: 25px; background: #dddff8">8</div>
    <div style="border: firebrick solid 1px; width: 30px; height: 25px; background: #dddff8">9</div>
    <div style="border: firebrick solid 1px; width: 30px; height: 25px; background: #dddff8">10</div>
    <div style="border: firebrick solid 1px; width: 30px; height: 25px; background: #dddff8">11</div>
</div>

为了方便餐厅点餐系统的编写，对实际规定进行简化，即**合法的手机号码严格按照下面的规定给出**，如果不符合下面的规定，即认为该手机号码不合法。

**规则如下:**

1. 前三位“网络识别号”的范围为: $[130, 187]$
2. 中间四位“地区编码”的范围为: $[0000, 9999]$
3. 最后四位“用户号码”的组成较为特殊，用户的手机号码的最后一位可表示手机号码持有人的性别，规定0为男性、1为女性，因此最后四位的范围为: $[031, 071] + 0/1$


**合法的手机号示例：**

- 13020210310
- 18501100451

**不合法的手机号码示例：**

- 10000000310
- 18900000311
- 17701210000
- 15600010452

## 题目描述

编写一个 Person 类：

- 属性至少包括用户名（仅可能由 26 个字母大小写和数字构成，不含空格）、性别（字符 M/F，分别表示男性/女性）、手机号（储存合法的格式，详见题目背景）

- 以上属性应为私有属性。需提供相应的 getter 和 setter 方法（通常 IDE 有支持自动生成的功能和快捷键，可自行探索）

- 实现静态方法 static boolean checkSex(char sex)，判断传入的性别是否为合法，合法为 true

- 实现静态方法 static boolean checkNum(String phoneNum)，判断传入的手机号是否合法，合法为 true

- 实现方法 String toString()产生格式化的信息，具体要求如下：

  - 冒号为英文字符':'
  - 保证所有信息合法
  - 不包含多余的空格
  - 所有字符为全角字符

  以 ZhangSan，手机号 13766660310 为例：

  ```shell
  Name:ZhangSan
  Sex:M
  Phone:13766660310
  ```

- 实现静态方法 addPerson(String name, char sex, String phoneNum)，要求：
  - 输入参数为表示用户名的字符串name，表示性别的字符 sex，和表示手机号的字符串 phoneNum
  - 检查参数是否合法。若非法，返回 null
  - 若合法，返回一个 Person 对象

在上次的 Test 类基础上，增加以下功能：

- 在命令行终端中输入以下命令，执行相应的操作：

基本格式为：

```shell
选项 [参数1] [参数2] [参数3]
```

| 选项 | [参数 1] | [参数 2] | [参数 3] | 功能描述                                                                                                      |
| ---- | -------- | -------- | -------- | ------------------------------------------------------------------------------------------------------------- |
| np   | 用户名   | 性别     | 手机号   | 调用上文所述的 addPerson()方法。对于非法输入，终端输出相应的信息。对于合法输入，调用该对象的 toString()方法。 |

具体要求如下：

- 在输入 np 命令时，并不保证参数数量、类型严格对应，这也是一种非法情况，需要在终端输出：
  
  ```shell
  Arguments illegal
  ```

- 如果参数合法，按以下顺序依次进行信息检查

  - 用户名由 26 个字母和数字构成，保证测试点中一定合法
  - 性别必须为 F 或 M，其他情况请输出

  ```shell
  Sex illegal
  ```

  - 手机号需满足指定格式，非法请输出

  ```shell
  Phone number illegal
  ```

- 如果存在多种非法情况，按上述顺序只输出最先发生的非法信息。
- 如果输入合法，命令行按 toString()输出格式打印。如：

  ```shell
  np ZhangSan M
  Arguments illegal
  np ZhangSan M 1376666123
  Phone number illegal
  np ZhangSan NB 1376666123
  Sex illegal
  np ZhangSan M 13766660310
  Name:ZhangSan
  Sex:M
  Phone:13766660310
  ```
