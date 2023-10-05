*前言：本人在写下此笔记时，模电基础接近0。*

*网课为[勤于回看课堂未懂时视频专辑-勤于回看课堂未懂时视频合集-哔哩哔哩视频 (bilibili.com)](https://space.bilibili.com/1293957708/channel/series)*

*再次感谢这位老师*

*强烈推荐清华大学王红老师的课程*

[门电路_其他功能&传输们&三态门_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1hS4y1k7jV?p=13&vd_source=f613fe1150e201362cd3c231b5ba57b8)

*同时强烈推荐*<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115001.png" alt="image-20230930110742234" style="zoom:50%;" />此书，第六版

# 一和二、数字电路基础

## 1.1 数字电路基础

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115002.png" alt="69476516947400332" style="zoom:30%;" />



抗干扰强，高低电平是有范围的，数字电路有一定的抗干扰能力

## 1.2 编码

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115003.png" alt="105870416947403252" style="zoom:30%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115004.png" alt="77053816947400862" style="zoom:33%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115005.png" alt="116891916947404642" style="zoom:33%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115006.png" alt="129453016947408072" style="zoom:33%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115007.png" alt="26985716947415302(1)" style="zoom:25%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115008.png" alt="46794916947423532" style="zoom:25%;" />



8421码，也被称为BCD码，是十进制代码中最常用的一种12。在这种编码方式中，每一位二值代码的“1”都代表一个固定数值12。这个固定数值就是这一位的权12。

8421代表的是权值分配3。代码中从左至右看每一位“1”分别代表数字“8”“4”“2”“1”，故得名8421码12。例如，二进制数1001在8421码中表示的就是十进制数9（8+0+0+1=9）4。

因为每位的权都是固定不变的，所以8421码是恒权码12。这意味着无论在何种情况下，每一位的权都不会改变。例如，在8421码中，最左边的位总是代表8，无论其他位是什么

> BCD编码以及它的权的解释
>
> 5421码是一种有权BCD码，它的权值从高位到低位分别为5、4、2、1。这意味着在5421码中，每一位二进制数的“1”代表的十进制数分别为5、4、2、1。
>
> 例如，如果我们有一个5421码为1100，那么这个码代表的十进制数就是9（5*1 + 4*1 + 2*0 + 1*0 = 9）。
>
> 需要注意的是，5421码和2421码的编码方案都不是唯一的。例如，在5421 BCD码中，数码5既可以用1000表示，也可以用0101表示。这是因为在这两种情况下，"1"所在的位置的权值之和都等于5（5*1 + 4*0 + 2*0 + 1*0 = 5 或者 5*0 + 4*1 + 2*0 + 1*1 = 5）。
>
> Source: Conversation with Bing, 9/15/2023
> (1) BCD码-8421码、5421码、2421码、余3码 - CSDN博客. https://blog.csdn.net/qq_73505285/article/details/126797353.
> (2) 二进制编码(8421BCD、余三码、格雷码) - 知乎 - 知乎专栏. https://zhuanlan.zhihu.com/p/578175252.
> (3) 2.1.3 什么是BCD码？（有权码和无权码、格雷码、8421码、余3码、余3循环码、2421码等）_BitHachi的博客-CSDN博客. https://blog.csdn.net/weixin_43914604/article/details/104181040.
> (4) 二进制编码(8421BCD、余三码、格雷码) - 知乎 - 知乎专栏. https://bing.com/search?q=5421+BCD%e7%bc%96%e7%a0%81%e7%9a%84%e6%9d%83%e7%9a%84%e5%90%ab%e4%b9%89.
> (5) 【无标题】8421码，5421码，2421码，余三码之间的区别及对数的表示规则_5421bcd码_m0_73485205的博客-CSDN博客. https://blog.csdn.net/m0_73485205/article/details/126810444.
> (6) 8421码，5421码，2421码，余3码的区别 - CSDN博客. https://blog.csdn.net/m0_73648269/article/details/126807136.

## 1.3 数制

### 二进制的补码运算

用**同位数**的**最大数**减去**减数**得到**反码**（反相器、取反），**反码**+1变为**补码**

再用**被减数**加上**补码**，并**舍去新增加的高位**，就是结果

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115009.png" alt="image-20230916105329252" style="zoom:50%;" />

> 常用于没有减法的装置

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115010.png" alt="image-20230916105609036" style="zoom:50%;" />

整数的原码、补码和反码速通

>例题：x = +1011，y = -1001，并且默认为7位数
>
>**对于原码**：符号位用状态0表示+，用状态1表示
>
>X原 = +0001011 = 00001011；Y原 = -0001001 = 10001001。
>
>**对于反码**：对于正数，数值与真值的数值部分相同；对于负数，数值部分为真值的数值部分按位取反
>
>X反 = +0001011 = 00001011；Y反 = -001001 = 11111111-0001001 = 11110110。
>
>**对于补码**：对于正数，数值部分与真值的数值部分相同；对于负数，数值部分为真值的数值部分按位取反且末位加1
>
>X补 = +0001011 = 00001011；Y补 = -0001001+11111111+1=11110111

小数的原码、补码和反码速通

> 例题：x = +0.1011，y = -0.1001
>
> 原码：X原 = +0.1011 = 0.1011000，Y原 = -0.1001 = 1.1001000（在整数最低位补1表示负数，并且需要将其低位填0补齐n位）
>
> 反码：X反 = +0.1011 = 0.1011000，Y反 = -0.1001 = 1.1111111 - 0.1001000 = 1.0110111
>
> 补码：X补 = +0.1011 = 0.1011000，Y补 = -0.1001 = 1.1111111 - 0.1001000 + 0.0000001 = 1.0111000



### 十六进制

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115011.png" alt="image-20230916110437563" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115012.png" alt="image-20230916110549526" style="zoom:33%;" />

### 各进制表的对比

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115013.png" alt="image-20230916110622917" style="zoom:33%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115014.png" alt="image-20230916110727489" style="zoom:33%;" />

## 1.4 逻辑代数基础

### 同或与异或

[同或和异或是两种逻辑运算符，它们都是在二进制位上进行操作的](about:blank#)[1](https://www.zhihu.com/question/592701911)。

- [**异或**（xor）的运算法则为：如果a、b两个值不相同，则异或结果为1。如果a、b两个值相同，异或结果为0](about:blank#)[2](https://zhidao.baidu.com/question/1954567007904002228.html)[。也就是说，异或运算是在两个二进制位上进行“相同为0，相异为1”的运算](about:blank#)[3](https://baijiahao.baidu.com/s?id=1726966984466035760)。
- [**同或**运算的结果与异或运算正好相反。同或运算的结果为：如果a、b两个值相同，则同或结果为1。如果a、b两个值不相同，则同或结果为0](about:blank#)[3](https://baijiahao.baidu.com/s?id=1726966984466035760)[。也就是说，同或运算是在两个二进制位上进行“相同为1，相异为0”的运算](about:blank#)[3](https://baijiahao.baidu.com/s?id=1726966984466035760)。

[所以，我们可以说同或和异或互为非运算](about:blank#)[2](https://zhidao.baidu.com/question/1954567007904002228.html)[。也就是说，对一个异或运算的结果进行非运算（取反），就得到了同或运算的结果，反之亦然](about:blank#)[2](https://zhidao.baidu.com/question/1954567007904002228.html)。

[此外，同或和异或都满足结合律，即 (A ⊙ B) ⊙ C = A ⊙ (B ⊙ C)和 (A ⊕ B) ⊕ C = A ⊕ (B ⊕ C)。这意味着可以通过改变同或和异或的括号方式，得到相同的结果](about:blank#)[4](https://wenku.baidu.com/view/c7abec8d4a649b6648d7c1c708a1284ac85005da.html)。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115015.png" alt="image-20230916205009081" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115016.png" alt="image-20230916205027059" style="zoom:50%;" />



<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115017.png" alt="image-20230916112214244" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115018.png" alt="image-20230916112407054" style="zoom:50%;" />

比如第一条式子，需要**与门**、**或门**和**反相器**三个集成电路

但第二条只需要**与非门**一个集成电路（反相器是可以用与非门搭出来的，只要两端接口接入相同的信号，就能输出非门）

### 逻辑代数的化简（三个多余！很重要）

**超实用！**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115019.png" alt="image-20230916193658352" style="zoom:50%;" />

- 含项多余（）
- 非因子多余
- 第3项多余（化简很有用）（当n个或式，一个拥有A正，另一个拥有A反时，那么巴拉巴拉如图，B是A的另一半，C是A反的另一半，那么BC将多余）

例子：<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115020.png" alt="image-20230916194140839" style="zoom:50%;" /><img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115021.png" alt="image-20230916194212745" style="zoom: 50%;" />

### 逻辑代数的基本规则

**带入规则**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115022.png" alt="image-20230916194554988" style="zoom:50%;" />

**反演规则**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115023.png" alt="image-20230916194651779" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115024.png" alt="image-20230916194726477" style="zoom:50%;" />

如果是在一个大非号下有多个变量

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115025.png" alt="image-20230916195610902" style="zoom:50%;" />

**对偶规则**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115026.png" alt="image-20230916195640878" style="zoom:50%;" />

> [在逻辑代数中，对偶式的概念是非常重要的。对偶式的定义是：如果将逻辑函数表达式F中所有的“·”变成“+”，“+”变成“·”，“0”变成“1”，“1”变成“0”，并保持原函数中的运算顺序不变，则所得到的新的逻辑表达式称为函数F的对偶式](https://www.baike.com/wiki/对偶式)[1](https://www.baike.com/wiki/对偶式)[2](https://baike.baidu.com/item/对偶规则/6155756)[3](https://wapbaike.baidu.com/item/逻辑代数/1933102)。
>
> 对偶式在逻辑代数和数字电路设计中有以下应用：
>
> 1. [**简化电路设计**：在数字电路设计中，我们可以利用对偶原理来简化电路。例如，如果我们有一个已经设计好的电路，但是需要设计一个新的电路，新电路的逻辑函数是原电路逻辑函数的对偶式，那么我们可以直接将原电路中的所有AND门替换为OR门，OR门替换为AND门，同时将所有的0替换为1，1替换为0，就得到了新的电路](https://zhuanlan.zhihu.com/p/153320116)[4](https://zhuanlan.zhihu.com/p/153320116)[5](https://zhuanlan.zhihu.com/p/80693923)。
> 2. [**证明定理**：在逻辑代数中，许多定理和它们的对偶式都是同时成立的。也就是说，如果一个定理是正确的，那么它的对偶式也是正确的。这大大简化了逻辑代数定理的证明过程](https://zhuanlan.zhihu.com/p/153320116)[4](https://zhuanlan.zhihu.com/p/153320116)[5](https://zhuanlan.zhihu.com/p/80693923)。
> 3. [**优化布尔表达式**：在优化布尔表达式时，我们可以利用对偶原理来简化布尔表达式。例如，如果我们有一个布尔表达式，但是这个表达式过于复杂，不易于理解或实现，那么我们可以尝试找到这个布尔表达式的对偶式，看看对偶式是否更简单](https://zhuanlan.zhihu.com/p/153320116)[4](https://zhuanlan.zhihu.com/p/153320116)[5](https://zhuanlan.zhihu.com/p/80693923)。
>
> [总之，在逻辑代数和数字电路设计中，对偶原理是一个非常强大的工具](https://zhuanlan.zhihu.com/p/153320116)[4](https://zhuanlan.zhihu.com/p/153320116)[5](https://zhuanlan.zhihu.com/p/80693923)。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115027.png" alt="image-20230916200256386" style="zoom:50%;" />

## 1.5 逻辑函数的建立与表示方法

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115028.png" alt="image-20230916202620892" style="zoom:50%;" />

真值表

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115029.png" alt="image-20230916202737706" style="zoom:50%;" />

逻辑函数<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115030.png" alt="image-20230916202839988" style="zoom:50%;" />

不做化简要用到与门或门和反相器

因此要做出化简

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115031.png" alt="image-20230916203032859" style="zoom:50%;" />

注意这里的同或门，它不是将两个信号输入或门

**例题**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115032.png" alt="image-20230917192910583" style="zoom:50%;" />

双反，将**或**消除，同时保留**与**的那部分不变（绝妙的思路！在第二步到第三步中，对于原来的与，不去反号，这样就能保留与；对于第三步的或，则直接反演）

**例题**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115033.png" alt="image-20230917195520260" style="zoom:50%;" />

**用VN图来解释，A＋A还是A，所以可将+ABC变为+ABC+ABC+ABC**（牛逼！）

**M变为异或门和同或门的组合**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115034.png" alt="image-20230917195702429" style="zoom:50%;" />

## 1.6 逻辑函数的公式法化简与交换

### 1.6.1 逻辑函数的最简形式

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115035.png" alt="image-20230917201538001" style="zoom:35%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115036.png" alt="image-20230917203844978" style="zoom:50%;" />

**与或式**（本次课程以这个为主）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115037.png" alt="image-20230917204021541" style="zoom:50%;" />

**或于式**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115038.png" alt="image-20230917204041543" style="zoom:50%;" />

**1.并项法**（提取公因式）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115039.png" alt="image-20230917204421118" style="zoom:50%;" />

**2.吸收法**（这里是用含项多余，原项目可以提取1出来）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115040.png" alt="image-20230917204843816" style="zoom:50%;" />

**3.消去法**（非因子多余）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115041.png" alt="image-20230917204946041" style="zoom:50%;" />

对非B来说，AB的B就是非因子，因此多余，去掉；

同理的，对A来说，非A且B且C，则非A是非因子，去掉；同理对于非B来说，B是多因子，去掉，只剩下C

**4.配项法**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115042.png" alt="image-20230917205503424" style="zoom:50%;" />

​	<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115043.png" alt="image-20230917205723775" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115044.png" alt="image-20230917205746349" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115045.png" alt="image-20230917210048752" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115046.png" alt="image-20230917211051852" style="zoom:50%;" />

**常见式**（常用与非）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115047.png" alt="image-20230917211511764" style="zoom:50%;" />

常用的集成电路，都是集成一种

![image-20230917211627414](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115048.png)

### 1.6.2 卡诺图化简（直观且一般都能化简到最简）

#### 1. 逻辑函数的最小项及其性质

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115049.png" alt="image-20230917213412005" style="zoom:50%;" />

 <img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115050.png" alt="image-20230917213829355" style="zoom:50%;" /><img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115051.png" alt="image-20230917213850134" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115053.png" alt="image-20230917213915434" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115054.png" alt="image-20230917214021418" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115055.png" alt="image-20230917214311645" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115056.png" alt="image-20230917224644487" style="zoom:50%;" />

 某两个相加为1，那么代表着这两个互为相反关系

**最小项还原**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115057.png" alt="image-20230917225031298" style="zoom:50%;" />

#### 2. 卡诺图

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115058.png" alt="image-20230917225654737" style="zoom:50%;" />



**三变量的情况要注意中心线对称**<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115059.png" alt="image-20230918092912332" style="zoom:50%;" />

 **四变量的情况注意上下和左右对称**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115060.png" alt="image-20230918093829804" style="zoom:50%;" />

#### 3. 使用卡诺图解题

**先使用最小项填图**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115061.png" alt="image-20230918093940395" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115062.png" alt="image-20230918094730583" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115063.png" alt="image-20230918095016960" style="zoom:50%;" />

使用卡诺图上还原接近最小值，找出真值表

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115064.png" alt="image-20230918095133229" style="zoom:50%;" />

**K图化简（最重要）**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115065.png" alt="image-20230918100130285" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115066.png" alt="image-20230918100354656" style="zoom:50%;" />

这里的相邻包含几种情况（卡诺图上处在相邻、相对、相重位置的小方格所代表的最小项为相邻最小项。）

（要看成世界地图与实际的关系）

1. 相邻（周围8个格子，如果周围不足8个，则根据下面的找）
2. 相对（关于中心线对称）
3. 相重（两张表重叠的位置）

既有0又有1的要消去

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115067.png" alt="image-20230918102954480" style="zoom:50%;" />

**卡诺图化简要点**

有几个圈就有几个相乘的式子

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115068.png" alt="image-20230918104043337" style="zoom:50%;" />

**每一个圈都要尽可能大，最好只剩下一个变量！**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115069.png" alt="image-20230918104201921" style="zoom:50%;" />

**零少的时候也可以圈0！用作反函数**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115070.png" alt="image-20230918104339405" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115071.png" alt="image-20230918104509079" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115072.png" alt="image-20230918105412918" style="zoom:50%;" />



## 1.7 具有无关项逻辑函数的化简

![image-20230918105557305](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115073.png)

### 1.7.1 约束项

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115074.png" alt="image-20230918105744130" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115075.png" alt="image-20230918105932179" style="zoom:50%;" />（这里我解释一下，当水位降到B时，那么这时候水位也一定降到了C之下，因为B在C下面；同理，A在B下面，因此放水位降低到了A，那么水位肯定在B、C之下；所以A为1的时候，B和C都一定是1；B为1的时候，C也一定是1。那么010、100、101、110都是不可能会出现的情况）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115076.png" alt="image-20230918110036362" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115077.png" alt="image-20230918110343019" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115078.png" alt="image-20230918110705774" style="zoom:50%;" />

### 1.7.2 任意项

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115079.png" alt="image-20230918111913871" style="zoom:50%;" />

### 1.7.3 多输出逻辑函数的公用项设计

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115080.png" alt="image-20230918113816543" style="zoom:50%;" />

如果各自为营，那么化简结果如下图所示

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115081.png" alt="image-20230918113837402" style="zoom:50%;" />

如果不选择各自为营，而是选择共用项设计

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115082.png" alt="image-20230918114014058" style="zoom:50%;" />

保留公用项

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115083.png" alt="image-20230918114043473" style="zoom:50%;" />

---

---

# 二、集成逻辑门电路

## 2.1 基本逻辑门电路DTL（以前被用过，但现在被淘汰了）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115084.png" alt="image-20230918202724306" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115085.png" alt="image-20230918203148893" style="zoom:33%;" />

**预备知识：二极管的开关特性**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115086.png" alt="image-20230918204411966" style="zoom:33%;" />

**预备知识：三极管的特性**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115087.png" alt="image-20230918204600541" style="zoom:33%;" />

**预备知识：MOS管的特性**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115088.png" alt="image-20230918211045134" style="zoom: 33%;" />

### 2.1.1 二极管与门 及 或门 电路 

**与门**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115089.png" alt="image-20230918211622051" style="zoom:33%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115090.png" alt="image-20230918212350506" style="zoom:33%;" />

**或门**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115091.png" alt="image-20230918212257606" style="zoom:33%;" />

**非门**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115092.png" alt="image-20230918212719406" style="zoom:33%;" />

**DTL与非门**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115093.png" alt="image-20230918212843521" style="zoom:33%;" />

**DTL或非门**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115094.png" alt="image-20230918212912924" style="zoom:33%;" />

只用二极管组成，缺点：

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115095.png" alt="image-20230918213210483" style="zoom:33%;" />

---

## 2.2 CMOS逻辑门

### 2.2.1 NMOS管和PMOS管（插曲）

[NMOS和PMOS是两种类型的金属氧化物半导体场效应晶体管(MOSFET)，它们的工作原理主要基于控制源极和漏极之间的电流](https://blog.csdn.net/tabactivity/article/details/103639877)[1](https://blog.csdn.net/tabactivity/article/details/103639877)[2](https://zhuanlan.zhihu.com/p/519594505)。

[**NMOS管**的工作原理](https://blog.csdn.net/tabactivity/article/details/103639877)[3](https://www.elecfans.com/analog/202302212014587.html)[4](https://zhuanlan.zhihu.com/p/368263926)：

- [NMOS全称为N型金属-氧化物-半导体，我们称拥有这种结构的晶体管为NMOS晶体管](https://www.elecfans.com/analog/202302212014587.html)[3](https://www.elecfans.com/analog/202302212014587.html)。
- [当栅极电压高于一定值时，NMOS管会导通](https://blog.csdn.net/tabactivity/article/details/103639877)[3](https://www.elecfans.com/analog/202302212014587.html)[。这使得NMOS管适合用于源极接地的情况（低端驱动）](http://www.kiaic.com/article/detail/1100.html)[5](http://www.kiaic.com/article/detail/1100.html)。
- [使用NMOS当下管，S极直接接地，只需将G极电压固定值6V即可导通](https://blog.csdn.net/tabactivity/article/details/103639877)[3](https://www.elecfans.com/analog/202302212014587.html)。

[**PMOS管**的工作原理](https://blog.csdn.net/tabactivity/article/details/103639877)[5](http://www.kiaic.com/article/detail/1100.html)[6](https://www.elecfans.com/analog/202302212014611.html)：

- [PMOS是指n型衬底、p沟道，靠空穴的流动运送电流的MOS管](https://blog.csdn.net/tabactivity/article/details/103639877)[5](http://www.kiaic.com/article/detail/1100.html)。
- [当栅极电压小于一定值时，PMOS管会导通](https://blog.csdn.net/tabactivity/article/details/103639877)[5](http://www.kiaic.com/article/detail/1100.html)[。这使得PMOS管适合用于源极接VCC时的情况（高端驱动）](https://blog.csdn.net/tabactivity/article/details/103639877)[5](http://www.kiaic.com/article/detail/1100.html)。
- [使用PMOS当上管，S极直接接电源VCC，只需将G极电压比S极低6V即可导通](https://blog.csdn.net/tabactivity/article/details/103639877)[6](https://www.elecfans.com/analog/202302212014611.html)。

G极(gate)—栅极，不用说比较好认
**S极(source)—源极，不论是P沟道还是N沟道，两根线相交的就是**
**D极(drain)—漏极，不论是P沟道还是N沟道，是单独引线的那边**

//下列文章搬运自知乎

[一文彻底弄清MOS管 （NMOS为例） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/368263926?utm_id=0)

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115096.png" alt="image-20230927151405817" style="zoom:50%;" />

> 1. 硅中参**杂电子多**的话，会在那里写个**N**，参杂**空穴多**的话，会在那里写个**P**。
> 2.  电流的方向指的是**正电荷（空穴**）流动的方向，是电子流动方向的反方向。（初中物理）
> 3. 电源的**正电压端**会**吸引电子**聚集过来，**负电压端**会吸引**空穴**聚集过来。（初中物理：异性相吸）
> 4. **mos管是个开关，栅极（G）是控制端**，它控制着源极（S）和漏极（D）之间是否能导通，即是否可以通过电流。
> 5. 源极和漏极导通电流的意思是，给他们两加一个电压源，可以形成电流，从漏流到源也行，从源流到漏也行，这个电流是空穴移动产生的也行，是电子移动产生的也行。
> 6. 二极管正向（**从P到N接正电压）导通**，反向（从N到P接正电压）不导通。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115097.png" alt="image-20230927152619130" style="zoom:50%;" />

拿到这副mos管的结构图，首先看到s极和d极连着的是**N参杂区域**，代表这两个区域**电子多**，衬底是P参杂区域，代表**空穴多**。

此时，想象在S极和D极两端加一个电压源(方向无所谓)，可惜中间衬底部分都是P型啊，【由已知第6条】**电子想移动形成电流可是从N到P是二极管反向走不通啊**。那我们的目标就来了，在这两个N参杂区域之间架起一条可让电子通过的“桥梁”。（有点像三极管，左边放一个二极管，右边放一个二极管）

（**太牛逼了，简直通俗易懂！**一下子就明白了！）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115098.png" alt="image-20230927154434195" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115099.png" alt="image-20230927154750347" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115100.png" alt="image-20230927155000165" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115101.png" alt="image-20230927155143655" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115102.png" alt="image-20230927155323059" style="zoom:50%;" />

神他么一下子就懂了。

### 2.2.1 CMOS逻辑门（1）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115103.png" alt="image-20230926090051470" style="zoom:50%;" />

数字电路中，经常工作在，要么是截止区，要么是可变电阻区，一般不工作在饱和区

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115104.png" alt="image-20230926090241105" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115105.png" alt="image-20230926090258375" style="zoom:33%;" />最基本的非门/反相器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115106.png" alt="image-20230926090358690" style="zoom:50%;" />这两个管子是轮流导通的

在CMOS电路中，逻辑0代表低电平电压 **0V**，逻辑1代表高电平电压 **VDD** （实际上大于电源电压的70%就已经接受了逻辑1，小于电源电压30%也认可是逻辑0）（它是个范围）

((即u0测试的是两个漏极)	)

![image-20230927155845777](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115107.png)

![image-20230927160705832](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115108.png)

#### 1.**CMOS门电路中的与非门**（我时常感叹前人的智慧）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115109.png" alt="image-20230927161304277" style="zoom:50%;" />

由于PMOS连接着电源VDD，NMOS连接着接地GND。

并且PMOS并联意味着任意一个低电平就能导通；NMOS串联意味着都是高电平才能导通。

**因此，PMOS收到任意一个低电平就会导通，且导通输出为VDD；NMOS收到所有高电平才能导通，导通输出为接地**

#### 2. **CMOS门电路中的或非门**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115110.png" alt="image-20230927161553785" style="zoom:50%;" />

**VDD且P串—>皆为低电平才能导通—>导通后是高电平；**

**GND且N并—>只要有高电平就能导通—>导通后是低电平**

#### 3. CMOS门电路中的异或门（同或门）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115111.png" alt="image-20230927162904181" style="zoom:50%;" />

#### 4. 例题尝试

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115112.png" alt="image-20230927170231282" style="zoom:50%;" />

本题重点：CMOS非门是基础，优先找到CMOS非门，并圈起来；

A、B、C先是各自通过反相器变成A非、B非、C非；之后，它们三人又碰到一个反相器，不过这个反相器是，PMOS上面三个并联的，以及NMOS下面三个串联的，**即A非、B非和C非又反相一次，并且是串联反相**，所以是串或

> 有人可能会不理解为什么这里的串联是“或”运算。视A非、B非和C非为X、Y和Z。当XYZ三者有一个输入低电平（0），M处就会得到1；相反，只有三者都输入高电平的时候，M才会出现低电平。（与非）
>
> 因此在M处，是M = (XYZ)非；换成或运算，变成：M = X(非)+ Y(非)+ Z(非)

### 2.2.3 CMOS传输门和双向模拟开关

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115113.png" alt="image-20230928195450598" style="zoom:50%;" />

**先说明一下，虽然栅极电压高，能够吸引衬底的电子，但是，如果源极或者漏极的电压过高，则会影响到电子桥梁的浓度，即下图**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115114.png" alt="image-20230928201846173" style="zoom:50%;" />

因此，虽然PMOS/NMOS管的导通与否由栅极决定，但也会受到源极和漏极的影响

#### 传输截止

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115115.png" alt="image-20230928174643325" style="zoom:50%;" />

这里实际上看的是电压差，即对于PMOS来说，如果 **V**Pgs （**PMOS管的栅极G与源极S的电压差**）的值（负数）小于一定的阈值（PMOS的阈值电压），那么将会导通。然而这里栅极却**输入了高电平**。这也意味着，如果这时候，输入信号如果为大于0的电压值，那么将无法导通。

同样的，对于NMOS来说，如果 **V**Ngd （**NMOS管的栅极G与漏极D的电压差**）的值（正数）也必须大于一定的值。然而它却给了g一个低电平。因此，只要输入信号时低电平，那么将无法导通。

#### 传输导通

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115116.png" alt="image-20230928202713810" style="zoom:50%;" />

注：图上的**UTN的绝对值**等于**UTP的绝对值**（UTN、UTP都是阈值电压）

当C=1，C’=0的时候，这时候大家都有机会导通了。

这时候，C‘=0，则是代表接地；C=1，则是接入VDD。

此时假设接入的信号是，最低是0V，最高是VDD的正向电压。

如果要使得 PMOS导通，那么 **V**Pgs要低于阈值电压。又因为g极接地，电位为0，所以**该电压是一个负值（相反数是输入信号u1）**，而且阈值电压也是一个负值，最后变成了 u1要大于阈值电压的绝对值（就是UTN），也就是

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115117.png" alt="image-20230928203657146" style="zoom:50%;" />

那么对于NMOS而言也是同样的道理，**V**Ngd（即VDD-u1）要大于阈值电压

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115118.png" alt="image-20230928203750027" style="zoom:50%;" />

最终，在红色区域内，NMOS导通；蓝色区域内，PMOS导通。交叉区域内，都导通。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115119.png" alt="image-20230928204031305" style="zoom:50%;" />

最终得到这个传输门的简略图，要注意，上下必须输入取反的信号，即必须一个高电平一个低电平

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115120.png" alt="image-20230928204211088" style="zoom:50%;" />

####  双向模拟开关

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115121.png" alt="image-20230928205144524" style="zoom:50%;" />

能够输入两个信号，一个反相，一个正相

#### CMOS异或门

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115122.png" alt="image-20230928205349369" style="zoom:50%;" />

### 2.2.4 CMOS漏极开路门以及三态门

#### 1.CMOS漏极开路门

> 在漏极开路门中，"漏极开路"这个术语的含义是指在某些状态下，输出端（漏极）是开路的。这并不意味着漏极物理上是断开的，**而是指在电路的工作状态下，漏极可以处于高阻态，也就是说，它可以看作是电路中的一个开关。**
> 当我们说漏极开路时，我们实际上是在描述一个特定的工作状态。在这种状态下，内部的N沟道场效应管关闭，**上拉电阻R会把输出拉到高电平**，此时场效应管的漏电流将非常小。当内部N沟道场效应管导通的时候，它会把输出引脚拉到接近GND。
> 所以，尽管在物理上漏极是通过上拉电阻与VDD连接在一起的，但在逻辑功能上，它可以根据输入信号的状态进行连接或断开连接。这就是为什么我们说在漏极开路门中，漏极是开路的。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115123.png" alt="image-20230929120621022" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115124.png" alt="image-20230929120626481" style="zoom:50%;" />

要注意它前面还有一个非门，老师这里讲得很傻逼。

实际上它这个漏极开路输出前面会接入一个非门（因为在通过这个漏极开路门之后，相当于反相一次，而我们的目的并不是反相，而是实现 **线与**）

**RL电阻不能太大，否则会导致RL上压降太大从而使得漏极处电压太小，无法被识别成高电平逻辑1**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115125.png" alt="image-20230929201411424" style="zoom:50%;" />

**RL电阻也不能太小，否则会造成损毁短路，这里用作保护作用**（一个管子能够允许的最大电流来限定的，至少要一个管子导通）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115126.png" alt="image-20230929201908285" style="zoom:50%;" />

#### 2.CMOS三态门反相器

*养成习惯：信号+反相器，并且在输入端——一般代表低电平有效*

三态门的三态：高电平、低电平和高阻态

这里的输出缓冲器又和FPGA对上了

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115127.png" alt="image-20230930085453992" style="zoom:50%;" />

我们看到这里的EN'是用<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115128.png" alt="image-20230930085732000" style="zoom:50%;" />这个，区别于A的<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115129.png" alt="image-20230930085750791" style="zoom:50%;" />

这里就是我们所说的低电平有效，**并且这里也是反相器**。（这里的低电平有效是指 **当EN'输入低电平的时候，整个三态门工作，输出结果为高低电平其中一个**）

**当EN'=0时**

- 若A=0，则G4输出为0，T1导通；G5输出也是0，T2截止——结果高电平
- 若A=1，则G4输出为1，T1截止；G5输出也是1，T2导通——结果低电平

> 可以看到，实际上这是一个反相器

**当EN‘=1时**

- **G4一定是1**，因为G1是0，“与”结果一定是0，则“与非”结果一定是1，**T1一定截止**
- **G5一定是0**，因为G3是1，”或结果“一定是1，则“或非”结果一定是0，**T2一定截止**

**T1、T2同时截止，呈现高阻态**

![image-20230930091205467](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115131.png)

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115132.png" alt="image-20230930091229460" style="zoom:50%;" />

这玩意到底有什么用？老师举了一个很好的例子，就是USB插口，你可能会插很多设备到电脑里面，但真正未必要用到，需要用到的时候，则使它有效工作。*通过高阻态使得电气连接不存在*

> 物理连接和电气连接是两个不同的概念。物理连接通常指设备之间通过某种介质（如电线、网线、无线电等）在物理上存在的实际连接。这种连接可以看得见摸得着，比如你可以看到一根网线连接了两台电脑。
>
> 而电气连接则更专指在电气系统中，设备或元件之间通过导体（如电线）进行的连接。这种连接主要是为了实现信号或数据的传输。例如，一个开关和一个灯泡通过电线连接，当开关打开时，电流可以通过电线流向灯泡，使灯泡亮起。
>
> 所以，当说“物理连接存在但电器连接不存在”时，可能是指虽然设备之间有实际的物理连接（如通过一根电线连接），但是在电气上并没有实现有效的连接，无法实现信号或数据的传输。例如，两台电脑之间虽然有一根网线相连（物理连接存在），但如果网线的一头没有插入网口或者网络设置不正确，那么这两台电脑就无法进行数据传输（电器连接不存在）。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115133.png" alt="image-20230930093225848" style="zoom:50%;" />

## 2.3 TTL门电路

### 2.3.1 双极型三极管

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115134.png" alt="image-20230930093815679" style="zoom:50%;" />

#### 输入特性曲线

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115135.png" alt="image-20230930094045119" style="zoom:50%;" />

#### 输出特性曲线

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115136.png" alt="image-20230930094236696" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115137.png" alt="image-20230930104408722" style="zoom:50%;" />

#### 双极型三极管基本开关电路——三极管反相器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115138.png" alt="image-20230930105132590" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115139.png" alt="image-20230930104952405" style="zoom:75%;" />



**截止状态下**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115140.png" alt="image-20230930105218354" style="zoom:50%;" />

**Von**其实就是be的导通电压降，当大于这个导通电压降时，be处就会开始有电流；小于该电压降时截止。

**放大状态下**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115141.png" alt="image-20230930105932748" style="zoom:50%;" />

**饱和状态下**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115142.png" alt="image-20230930110424675" style="zoom:50%;" />

**使用图解法进行分析**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115143.png" alt="image-20230930110649499" style="zoom:67%;" />

在这里，我们只想研究饱和区和截止区，即当成一个基本开关来使用。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115144.png" alt="image-20230930111305761" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115145.png" alt="image-20230930111537774" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115146.png" alt="image-20230930111818097" style="zoom:50%;" />

#### 三极管的动态开关特性

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115147.png" alt="image-20230930112235539" style="zoom:50%;" />

---

### 2.3.2 TTL反相器的电路结构和工作原理

#### 电路结构

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115148.png" alt="image-20230930113401032" style="zoom:80%;" />

**前置条件**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115149.png" alt="image-20230930113448608" style="zoom:50%;" />

**电路分析**

*当输入一个低电压*

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115150.png" alt="image-20230930114136634" style="zoom:70%;" />

这段话全是重点，全是分析，而且都很好懂，我稍微再补充一下

首先，T2发射结不导通，是因为T1的基极电位是0.9V（0.2V加上二极管的钳制0.7V）。而要使得T2发射结导通，至少需要大于两倍的Von（因为从T1到T2有两个二极管）

其次，此时T1处的集电极所在线路就是 T2的 **基极**，这时候请回顾到 **双极型三极管** 的输入与输出特性曲线。

再者，它这里所说的T1的集电极回路电阻是R2和T2的b-c结，反向电阻之和。这里看得回路是，从Vcc一直到A端，经过R2电阻的那段。由于这段会经过一个反向结（即b-c反向结，反向电阻非常大），所以此处电流很小。

最后，当基极电流乘以放大倍数远大于集电极电流时，三级管会进入深度饱和状态。



*当输入一个高电压*

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115151.png" alt="image-20230930210607912" style="zoom:70%;" />

这里就不做补充了。很简单的结果，在能够导通的情况下，先导通，后钳制住电压值进行改变。

> 为什么？今天的雨不下得，大一点？我应该听些悲情的歌，但我不能停止，我一旦停止了，什么都没了。我必须没有感情的前进，必须不把自己的情绪当作一回事，才能强大。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115152.png" alt="image-20230930211859306" style="zoom:50%;" />

推挽式电路，又叫推拉式电路，是由一对互补的晶体管组成的。这样的电路结构也被称为图腾柱（Totem-pole）输出电路。

#### 电压传输特性

还是要结合这个图看

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115153.png" alt="image-20230930212705870" style="zoom:70%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115154.png" alt="image-20230930213030385" style="zoom:50%;" />

### 2.3.3 TTL反相器的静态输入特性和输出特性

#### 1.输入特性

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115155.png" alt="image-20231001093920844" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115156.png" alt="image-20231001093930565" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115157.png" alt="image-20231001094342989" style="zoom:50%;" /><img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115158.png" alt="image-20231001094415585" style="zoom:50%;" />

#### 2.输出特性

**高电平输出特性**（Voh）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115159.png" alt="image-20231001095336217" style="zoom:67%;" />

然而实际会存在功耗限制，导致负载电流在5ma以下

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115160.png" alt="image-20231001095507478" style="zoom:50%;" />

**低电平输出特性**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115161.png" alt="image-20231001100343651" style="zoom:50%;" />

**Vol**相应就是输出低电压

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115162.png" alt="image-20231001100418592" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115163.png" alt="image-20231001100727652" style="zoom:50%;" />

#### 3.输入端负载特性

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115164.png" alt="image-20231001101039502" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115165.png" alt="image-20231001101141347" style="zoom:50%;" />插入电阻，通过电流影响电阻电位，最后影响到V1处的电位（就会影响到三极管的导通问题）

### 2.3.4 TTL反相器的动态特性

#### 1. 传输延迟时间

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115166.png" alt="image-20231001102111961" style="zoom:50%;" />

#### 2. 交流噪声容限

> 噪声容限（英语：Noise Margin）是指在前一级输出为最坏的情况下，为保证后一级正常工作，所允许的最大噪声幅度。在数字电路中，一般常以“1”态下（上）限噪声容限和“0”态上（下）限噪声容限中的最小值来表示电路（或元件）的噪声容限。
> **例如，一条数字电路中的电压也许被设计在0.0和1.2v之间变化，任何在0.5v以下的电压被认为是逻辑‘0’，而任何在0.7v之上的电压被认为是逻辑1。然后0的噪声容限是电压值在0.5v以下的信号，并且‘1’的噪声容限是电压值在0.7v以上的信号。**
> 简单来说，噪声容限就是整个电路所允许的噪声极限。噪声容限越大说明容许的噪声越大，电路的抗干扰性越好。这就意味着，如果一个信号受到了噪声的影响，只要这个噪声没有超过噪声容限，那么这个信号就能被正确地识别出来。因此，我们可以说噪声容限是衡量一个电路抗干扰能力的一个重要参数。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115167.png" alt="image-20231001102932320" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115168.png" alt="image-20231001103015296" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115169.png" alt="image-20231001103043973" style="zoom:50%;" />

#### 3. 电源的动态尖峰电流

**在输出低电平的情况下**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115170.png" alt="image-20231001104100447" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115171.png" alt="image-20231001104146821" style="zoom:50%;" />

**在输出高电平的情况下**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115172.png" alt="image-20231001104209221" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115173.png" alt="image-20231001104237442" style="zoom:50%;" />

**输出发生切换时，会出现特殊情况**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115174.png" alt="image-20231001104629385" style="zoom:50%;" />

> 在TTL反相器中，T4比T5先导通的原因主要是由于三极管的工作特性决定的。
>
> 当输入信号由低电平变为高电平时，T2和T5需要从饱和状态转变为截止状态，这个过程被称为“退饱和”。然而，**退饱和的过程相对较慢，这是因为在饱和状态下，三极管的基极-集电极间存在大量的多余载流子，当转变为截止状态时，这些多余的载流子需要被清除掉，这个过程需要一定的时间。**
>
> 在此期间，由于T2和T5还未完全截止，它们的集电极电压仍然较低，这使得T4的基极-发射极间形成正向偏置，从而使T4导通。等到T2和T5完全截止后，它们的集电极电压上升，此时T4的基极-发射极间不再形成正向偏置，于是T4也就截止了。
>
> **因此，在输入信号由低电平变为高电平的过程中，由于退饱和过程的存在，使得T4会在T5截止之前先导通。这就是为什么在TTL反相器中，“退饱和会来得慢一些”会导致T4比T5先导通的原因。**

**电源尖峰电流带来的影响**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115175.png" alt="image-20231001104751417" style="zoom:50%;" />

### 2.3.5 其他类型的TTL门电路

#### 1.与非门

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115176.png" alt="image-20231001111019170" style="zoom:50%;" />

#### 2.或非门

​	<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115177.png" alt="image-20231001111220664" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115178.png" alt="image-20231001111453174" style="zoom:50%;" />

#### 3.与或非门

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115179.png" alt="image-20231001111614411" style="zoom:70%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115180.png" alt="image-20231001111637041" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115181.png" alt="image-20231001111658022" style="zoom:50%;" />

#### 4.异或门

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115182.png" alt="image-20231001112007539" style="zoom:67%;" />

![image-20231001112854060](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115183.png)

可以看出T4、T5和T6、T7管子连接起来的绝妙

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115184.png" alt="image-20231001113425573" style="zoom:50%;" />

#### 集电极开路输出的门电路（OC门）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115185.png" alt="image-20231001113830003" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115186.png" alt="image-20231001114407249" style="zoom:50%;" />

**开路输出的与非门**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115187.png" alt="image-20231001114310289" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115188.png" alt="image-20231001114335955" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115189.png" alt="image-20231001114450546" style="zoom:50%;" />

#### OC门外接电阻的计算方法

**与门**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115190.png" alt="image-20231001114716711" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115191.png" alt="image-20231001114737096" style="zoom:70%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115192.png" alt="image-20231001114757893" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115193.png" alt="image-20231001114807208" style="zoom:50%;" />

这里所说的图3.4.27:<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115194.png" alt="image-20231001114917817" style="zoom:50%;" />

**或门**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115195.png" alt="image-20231001115153293" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115196.png" alt="image-20231001115200062" style="zoom:70%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115177.png" alt="image-20231001111220664" style="zoom:50%;" />

#### 三态输出门电路

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115197.png" alt="image-20231001160943578" style="zoom:50%;" />

**高电平有效**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115198.png" alt="image-20231001161002667" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115199.png" alt="image-20231001161548109" style="zoom:50%;" />

二极管D截止，则变回原来类似TTL与非门电路一样。

然而二极管D一旦导通，就会导致T4和T5的基极受到影响，从而使得这两个地方的电位都被钳制住

**低电平有效**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115200.png" alt="image-20231001161014253" style="zoom:50%;" />

**原TTL与非门电路**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115201.png" alt="image-20231001161305726" style="zoom:50%;" />

---

# 四、组合逻辑电路

## 4.1 什么是组合逻辑电路

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115202.png" alt="image-20231001162534745" style="zoom:50%;" />

**逻辑功能的描述**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115203.png" alt="image-20231001162709022" style="zoom:50%;" />

## 4.2 组合逻辑电路的分析方法

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115204.png" alt="image-20231001162835204" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115205.png" alt="image-20231001164509628" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115206.png" alt="image-20231001164603045" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115207.png" alt="image-20231001164655268" style="zoom:50%;" />

公式是永远不如例子直观的

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115208.png" alt="image-20231001164720880" style="zoom:70%;" />

可以观察到每5次，就轮到一个信号输出高电平。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115210.png" alt="image-20231001165003869" style="zoom:50%;" />

## 4.3 组合逻辑电路的基本设计方法

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115211.png" alt="image-20231001190841186" style="zoom:50%;" />

#### 一、进行逻辑抽象

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115212.png" alt="image-20231001191646215" style="zoom:50%;" />

#### 二、写出逻辑函数表达式

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115213.png" alt="image-20231001191715256" style="zoom:50%;" />

#### 三、选定器件类型

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115214.png" alt="image-20231001192432986" style="zoom:50%;" />

#### 四、将逻辑函数化简活转换成适当的描述形式

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115215.png" alt="image-20231001192759545" style="zoom:50%;" />

#### 五、

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115216.png" alt="image-20231001203646940" style="zoom:50%;" />

#### 六、

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115217.png" alt="image-20231001203708345" style="zoom:50%;" />

#### 七、

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115218.png" alt="image-20231001203742263" style="zoom:50%;" />

## 4.4 若干常用的组合逻辑电路模块

### 4.4.1 编码器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115219.png" alt="image-20231001204435586" style="zoom:50%;" />

#### 一、普通编码器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115220.png" alt="image-20231001214201646" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115221.png" alt="image-20231001214321317" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115222.png" alt="image-20231001214648136" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115223.png" alt="image-20231001214732953" style="zoom:50%;" />

#### 二、优先编码器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115224.png" alt="image-20231001225629658" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115225.png" alt="image-20231001225322733" style="zoom:50%;" />





<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115226.png" alt="image-20231001225343760" style="zoom:50%;" />

![image-20231002092348213](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115227.png)

*关于此电路的解析前提*

1. 在等下展示的逻辑式中，**I‘**代表原输入，并且表示的是低电平有效。
2. 相应的，**I**则代表经过反相器的原输入，即 **对原输入取反**
3. 同理，**S'**代表原输入，**S**代表对 **该原输入取反** 。

> 采用低电平作为有效输入，是因为一般来说低电平比较稳定，不容易受到干扰

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115228.png" alt="image-20231002160850713" style="zoom:50%;" />

我们观察Y’2得出，我们的电路解析前提是正确的。

**关于选通输入端是如何控制电路的**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115229.png" alt="image-20231002161044403" style="zoom:60%;" />

**选通输入端经过所有的与门和与非门。**所以，一旦 **S‘=0**，就会有 **S=1**，此时与门 和 与非门 的状态则不由 **S'** 锁定。相反的，如果 **S‘=1**，就会有 **S=0**，此时所有的与门结果都为0，所有的与非门结果都是1，输出均被锁死。



**关于选通输出端对电路的影响**

*选通输出端的作用往往是为了和多个器件一起执行所用的，它很重要，因为它能直接告诉与它外部相连的逻辑电路它的状态。*

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115230.png" alt="image-20231002161511250" style="zoom:67%;" />

对于 **Y's**，7个互与原输入再与上 **S’**的反相，再作反相。因此，对于Y‘s为低电平，就需要内部8个互与结果为高电平，即每个都需要是高电平。因此需要 **S'** 是低电平（**S**是高电平）。得出结果为，**电路工作，但无编码输入**

对于 **Y'ex**，它是 **Y's** 与 S 的与非结果。从化简结果看，**I0 到 I7 都是 8个输入端取反的结果**。如果要使得**Y'ex**输出低电平，那么，S 和 I0到I7的和，二者必须都是1。即 **S’=0，并且I‘0到I’7之中有人取0**。得出结果是，**电路工作，而且有编码输入**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115231.png" alt="image-20231002163425093" style="zoom:70%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115232.png" alt="image-20231002163458414" style="zoom:60%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115233.png" alt="image-20231002163632403" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115234.png" alt="image-20231002164018225" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115235.png" alt="image-20231002164036079" style="zoom:50%;" />

（因此，在框图外部 **外加小圆圈**，不是取反的意思，而是代表低电平有效。）

**二-十进制编码器**

> 一开始我还不明白那四个输出能代表什么，直到看到真值表，看到它所代表的位数

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115236.png" alt="image-20231002165346362" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115237.png" alt="image-20231002165406403" style="zoom:50%;" />

### 4.4.2 译码器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115238.png" alt="image-20231002194409996" style="zoom:50%;" />

#### 一、二进制译码器

##### 二进制与门阵列

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115239.png" alt="image-20231002194534146" style="zoom:50%;" />

> 有没有疑惑，为什么3会对应8个呢？
>
> 很简单，2的3次方就是8。换句话说，我给出三个值，求对应的真值表，大概是这个意思

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115240.png" alt="image-20231002194658002" style="zoom:70%;" />

看到这个东西不要慌，我们这样整一下

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115241.png" alt="image-20231002194722821" style="zoom:67%;" />

看懂没？要是还没看懂，就变成 二极管的与门电路

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115242.png" alt="image-20231002194742498" style="zoom:67%;" />

以Y0为例，D1、D2、D3就是那三个二极管。这三个二极管对应连上的极就是A0'、A1'、A2'

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115243.png" alt="image-20231002200531066" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115244.png" alt="image-20231002200547036" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115245.png" alt="image-20231002200600430" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115246.png" alt="image-20231002200842641" style="zoom:50%;" />

一些大规模集成电路才采用。中规模多数采用三极管集成

##### CMOS门电路译码器

不要着急，首先让我们看着这个

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115247.png" alt="image-20231002201522808" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115248.png" alt="image-20231002201212749" style="zoom:70%;" />



<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115249.png" alt="image-20231002201543072" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115250.png" alt="image-20231002201024526" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041140128.png" alt="image-20231004114017075" style="zoom:67%;" />

我必须强调，逻辑框图真的很重要。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115251.png" alt="image-20231002201910936" style="zoom:60%;" />

> Gs输出高电平就不会有影响到与门/与非门。但如果输出是低电平，那么所有的与非门结果都是高电平
>
> 但其实Gs是与门，与门想要输出低电平，只要有一个低电平输入进来就好。那么这个低电平，可以是S1，也可以是S‘2，还可以是S’3

> 至于控制端之外的三个输入端（A0、A1和A2），它们实际上的思想原理是和二极管与门阵列构成的译码器是一样的。比如Y‘0，你会发现，它是与A0非、A1非、A2非的与非连在一起的。所以这个结构其实也是很好理解的（比上面那个二极管与门阵列好理解多了）

**下面是它的功能表**/真值表

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115252.png" alt="image-20231002202447035" style="zoom:70%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115253.png" alt="image-20231002202638368" style="zoom:70%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115254.png" alt="image-20231002202759808" style="zoom:50%;" />

这个很重要，这也是译码器实际的原理——将译码结果化为 **最小项**（最小项是在离散数学中出现的）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115255.png" alt="image-20231002204158460" style="zoom:50%;" />

我觉得可以这么理解：
S1输入进来一个1，它需要将1输出到某个特定的端口，但其他端口又不输出这个1

那么特定端口的地址就由 **A2A1A0来决定** ，并且输出的结果为1的反码0.

**这样就是 根据 A2A1A0 提供的地址将 1 输出到 特定的端口，并且输出的数据为 1的反码0**



#### 二、二-十进制译码器

> **这里的 二-十 的意思是，将 0到9 这个10个数字与 二进制互转**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115256.png" alt="image-20231002204813688" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041139457.png" alt="image-20231004113907407" style="zoom: 67%;" />

**我必须强调一下，逻辑框图真的很重要**

我们前面说过，**Y’n**代表这是在低电平有效，高电平无效。

即，当输出结果是0的时候，则为有效值，**确切地说，是我们想要的结果**

因此这是下面的真值表

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115257.png" alt="image-20231002205107935" style="zoom:67%;" />

从这个真值表我们不难看出，如果我们想要十进制的 **0**，那么就需要 **Y‘0** 是一个低电平（因为低电平有效）。

这时候就需要让逻辑电路连接成——当我 4个输入位变成二进制位 时都是0的时候——**Y’0**必须是低电平。

同理，其他的 **Y‘n** 也是同样的道理。我们要学会从结果去分析电路本身

![image-20231002205506916](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115258.png)

这里的伪码实际上就是，9之后的结果。什么意思呢？我4个输入实际上能够表示16个真值输出结果（2的4次方），但我只需要0到9，那么剩下的，我就舍弃了，就是伪码。

这是它的逻辑表达式

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115259.png" alt="image-20231002205423656" style="zoom:70%;" />

#### 三、显示译码器

##### 1.七段字符显示器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115260.png" alt="image-20231003085816453" style="zoom:70%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115261.png" alt="image-20231003085910617" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115262.png" alt="image-20231003085951260" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115263.png" alt="image-20231003090015618" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115264.png" alt="image-20231003090159496" style="zoom:70%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115265.png" alt="image-20231003090310621" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115267.png" alt="image-20231003090418533" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115268.png" alt="image-20231003090458560" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115269.png" alt="image-20231003090542595" style="zoom:50%;" />

##### 2.BCD - 七段显示译码器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115270.png" alt="image-20231003090824634" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115271.png" alt="image-20231003090838775" style="zoom:67%;" />

从真值表直接看出BCD译码器的原理。我想要显示数字5，那么就要找到那5条杠的二极管，并且在0101时使这5条杠发光。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115272.png" alt="image-20231003091224474" style="zoom:50%;" />（我感觉这东西，都是按需求来的，需求万岁）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115273.png" alt="image-20231003091425730" style="zoom:50%;" />

**BCD译码器内部原理图**

![image-20231003091523665](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115274.png)

经过分析，G2是双重反相器（即输入之后反相并在输出之后再反相一次）。 

- **灯测试*LT'*:**	 LT' 通过G2与 G5、G6、G7还有G3相连接。由于G2是双重反相器，所以G2的输出结果与LT'输入结果相同。因此当LT'= 0，则由于与非门特性，**G5、G6、G7和G3（乃至G4）结果都是1**。那么如果这时候LT' = 1，G5、G6、G7都由A0、A1和A2来决定。**如果要使得G5、G6、G7结果都是1，那么就需要A0、A1和A2都是0，结果就是8（二极管全亮）。换句话说，由于只需要一个LT' = 0就可以实现G5、G6、G7结果都是1，即实现二极管全亮，因此将LT' 称为灯测试。它用来检测数码管各段是否能正常发光**。 平时应该将LT'=1(置于高电平)。

- **灭零输入*RBI'*:**    灭零操作是为了让不希望显示的0熄灭。如果要达成此目的，必须使得**所有输出都是低电平**（已知高电平输出才会点亮二极管）。然而真值表中却没有这一项，因此就需要额外的控制电路。 要熄灭零，那么就**代表它本来就是0**，所以**A3、A2、A0和A1的输入必须都是0**。在此基础上对电路进行改造

  RBI'的反相器符号是低电平有效，因此我们令 **RBI'=0** ，且A3、A2、A1和A0也都是0。此时 **G5、G6、G7还有G8结果都是1，G1输出也是1。**而G3是对G1、G5、G6、G7、G8还有稳定在1的G2 构成的 与非门。因此，此时 **G3、G4** 输出是0。这将直接导致G9~G12输出都是高电平1。**由于G13~G19。每个与或非门都有一组输入全为高电平,所以Ya~Yg全为低电平**，所以输出结果是全灭。

- **灭灯输入/灭零输出 BI‘ /RBO’**：
  - 当用作输入端时，**它相当于是一个不受G2和G5~G8输出结果控制的G3**，这样一说就清晰了吧。所以它能够不用照顾A0~A3的输入。
  - 当用作输出端时，**它实际上就是G3的输出端。**我们都知道G3输出结果是0时，灯会全灭。即RBO' 结果是0时，灯会全灭。这个因果关系是，只有G3输出结果是0，灯才会全灭，RBO'输出才是0。即，**它是用来检测灭零输出是否奏效的，当RBO'=0，才能说明奏效**

> RBI’与 RI'的区别在于，前者要求**A3、A2和A1以及A0的输入必须都是0**才灭灯，而后者无论它们输入是什么都会灭灯。
>
> RBI' —— 灭零输入，低电平有效——在全输入为0的情况下使得0灭灯
>
> RBO' ——灭零输出，低电平有效——检验RBI'是否执行，通过输出结果为0检验
>
> RI' ——灭灯输入，低电平有效——无论输入情况，直接灭灯

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041137003.png" alt="image-20231004113758954" style="zoom:67%;" />

**逻辑框图真的很重要，不要忽视了**



**7488驱动共阴极的半导体数码管**

这是一位数字显示的时候

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115275.png" alt="image-20231003104439064" style="zoom:67%;" />

当需要多位数字显示的时候，希望其他的高位整数0和低位小数0都灭灯不显示

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115276.png" alt="image-20231003104559972" style="zoom:67%;" />

相当巧妙的设计！

如果我最高位的**RBI是0输入**，并且**A0~A3都是0**，那么灭灯，并且输出 **RBO=0**。此高位接收 **RBI=0**，这时候其实有两种情况：

1. 如果此高位也是全输入为0，那么ok，和最高位一样，灯全灭
2. 如果此高位并非全输入为0，此时 **RBI=0** 失效，不产生作用，该怎么亮就怎么亮。

所以说这真的是很绝妙的设计。



### 4.4.3 数据选择器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115277.png" alt="image-20231003161711586" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115278.png" alt="image-20231003161725113" style="zoom:50%;" />

逻辑实现图

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115279.png" alt="image-20231003185103227" style="zoom:50%;" />

有种，感觉从SEL中解码确定是选A还是选B的感觉



**4选1数据选择器**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115280.png" alt="image-20231003185202234" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115281.png" alt="image-20231003185326401" style="zoom:60%;" />

**牛逼，这里是通过地址代码，利用CMOS传输门来确定谁能输入，谁能输出，谁应该是什么状态**

比如TG1和TG2，很明显的，如果TG1能过去，那么TG2就过不去了。即便TG3能过去，等TG3输出到了TG6，又过不去了。原理是四筛二然后二筛一。

**逻辑表达式**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115282.png" alt="image-20231003185953950" style="zoom:70%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041136906.png" style="zoom:67%;" />

逻辑框图真的很重要，千万别忽视了！

### 4.4.4 加法器

二进制数之间的加减乘除都是通过加法器进行的。因此加法器是构成算术运算器的基本单元。

#### 一、1位加法器

##### 1. 半加器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115283.png" alt="image-20231003190537822" style="zoom:67%;" />

##### 2.全加器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115284.png" alt="image-20231003191724840" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115285.png" alt="image-20231003191805712" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115286.png" alt="image-20231003192443219" style="zoom:70%;" />（依据卡诺图化简结果构建的逻辑电路，其实将逻辑表达式化简下来构建，反而是用不到异或门什么的。化简之后清晰很多了）

#### 二、多位加法器

##### 串行进位加法器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115287.png" alt="image-20231003192404859" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115288.png" alt="image-20231003192729853" style="zoom:50%;" />

##### 超前进位加法器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115289.png" alt="image-20231003193441172" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115290.png" alt="image-20231003193650502" style="zoom:67%;" />

由于CI是来自上一位（低位）的进一，即，它是上一位的CO，本质是CO(i-1)，因此一次对所有CI展开

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115291.png" alt="image-20231003194405031" style="zoom:50%;" />

前面我们说了，当前位的结果是，两个相加位 与 进位 ，三者互相异或的结果。因此，在得到上一位的进1（即CO(i)）之后，我们就可以根据这个来求解了。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310032115292.png" alt="image-20231003194558191" style="zoom:50%;" />

![image-20231004095935191](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310040959287.png)

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041000052.png" alt="image-20231004100005000" style="zoom:50%;" />

### 4.4.5 数值比较器

#### 一、1位数值比较器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041001391.png" alt="image-20231004100141316" style="zoom:50%;" />

#### 二、多位数值比较器

![image-20231004100319904](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041003971.png)

在这里暂停并思考一下，应该是用或！最高位的或结果，如果结果是1，那么其他位的比较就没有意义，结果就已经出来了；相反，如果最高位或结果是0，也不会妨碍其他位数比较。

同时了为了限制其他位数的比较，要给它们都 与上 前面每个位数 同或结果。只有前面确实都是相同（同或结果都为1）那么确实是当前位数比较才有效。具体理解看下面这段。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041009168.png" alt="image-20231004100946095" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041010014.png" alt="image-20231004101000967" style="zoom:50%;" />

这段话也挺经典了，其实只需要弄出任意两个逻辑比较结果电路，第三个在它们基础上进行再一次集成就好了

![image-20231004101109041](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041011105.png)

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041012015.png" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041134634.png" alt="image-20231004113411584" style="zoom:67%;" />

说真的，逻辑框图真的很重要。

实际上不用真的每次都把电路图推一遍。。。有空的时候可以看看。

---

## 4.5 层次化和模块化的设计方法

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041052304.png" alt="image-20231004103308053" style="zoom:50%;" />

### 例1

![image-20231004103536016](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041035071.png)

> 这里都是低电平有效，因此视低电平输入为有效输入。
>
> S' 是选通输入端，当S‘ = 1时，所有输出封锁在高电平；S’=0，电路正常工作。
>
> Y‘s 与 Y’ex是选通输出端。当S‘=0时，如果Y’s为低电平，那么代表电路正常工作，但无编码输入（所有编码输入皆是高电平）；Y‘ex与其相反，如果Y'ex是低电平，那么代表电路正常工作，且至少有一个编码输入。（至少有一个编码输入是0）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041047215.png" alt="image-20231004104737145" style="zoom:67%;" />

天才的想法！当前面输入均无信号时，选通输出端Y's输出是低电平，此时再让下一个电路正常工作（连接选通输入端）。毕竟Y’s和Y‘ex是反着来的，它们就是用来告诉外设，电路到底有没有编码输入！

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041052018.png" alt="image-20231004105224954" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041056087.png" alt="image-20231004105647024" style="zoom:67%;" />

> 这里应该不是逻辑或，是与非才对。
>
> 这么想，当我想要输出二进制的15时，Z3是最高位，结果肯定是0的取反1；
>
> 这时候Y’s =1 ，有编码输入，那么 右侧的 T2‘、Y1’、Y0‘的输出全是 1 （因为低电平有效，所以无效输出高电平，实际上看的是反码。）*注意Z3、Z2、Z1和Z0都是高电平有效，它们都是原码*
>
> 因此与非门的输出结果此时取决于左侧，而不是或门的你是1那就是1。
>
> 对于15，它是一个差1进位的数，并且对于左侧来说，它实际上是 “7”。对于7来说，它的三位低电平有效输出结果就是 111的反码000。因此分别放入与非门，得出结果是 1111，所以这里确实是应该为与非门而不是或门



### 例2

![image-20231004111205316](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041112377.png)

![image-20231004111534660](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041115712.png)

> 一旦需要扩展，就离不开我们的控制端口了。
>
> 回顾一下，当S1=1且S2’=S3‘=0的时候，电路才能正常工作。
>
> 并且该译码器也是输出结果为低电平有效，即，我输入1111，只有Z’15应该输出低电平（有效），其他Z‘n应该输出高电平（无效）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041119624.png" alt="image-20231004111938561" style="zoom:67%;" />

![image-20231004112359092](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041123166.png)

> 控制第一片S1为高电平，使得第一片的使用与否 只与S2、S3有关。第二片同理，不再赘述。

### 例3

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041125832.png" alt="image-20231004112531782" style="zoom:67%;" />

![image-20231004113303663](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041133753.png)

**I是与对应的Y相连的，不是互联的，要与上面的编码器和译码器区分开**



### 和数据选择器有关的例题

![image-20231004151037921](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041510060.png)

> 这段话前面想讲的是，n位二进制译码器能够获得不大于n的组合逻辑函数；而根据数据选择器的逻辑表达式，则可以同样获得一定的组合逻辑函数。即使用数据选择器进行对n位二进制译码器的替代。

**下面这道例题将解释上面那段话所说的，“同时令D0~D3为第三个输入变量的适当状态（包括原变量、反变量、0或1）”**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041515259.png" alt="image-20231004151556176" style="zoom:50%;" />

就是右边D0~D3那里，它是这个意思，通过这样来控制电路的结果。因为有一些最小项我们未必需要，所以可以用 **1** 或 **0** 作为输入变量删去一些最小项。

**再来一道复杂的例题**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041521385.png" alt="image-20231004152120306" style="zoom: 67%;" />

我们要拿Y的逻辑表达式与Z的逻辑表达式进行对比，很明显的嘛，我们可以直接得出代数结果：

> Z = A'B'C'+AB'C+ABC+A'BC
>
> D0=1，D1=0，D2=0，D3=1，D4=0，D5=1，D6=0，D7=0。
>
> 你可能会疑惑，这里为什么没有令Dn等于输入信号的。因为，这里不需要，实际上这里的需求是三变量的，而我们不需要四变量

据此，结局决定

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041532200.png" alt="image-20231004153229148" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041533467.png" alt="image-20231004153304400" style="zoom:67%;" />

> 永远不要觉得这种电路图很复杂，实际上，只要你明白它设计的原理以及设计的目的，它是什么组成，很多东西就迎刃而解了。

---

## 4.6 可编程逻辑器件

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041535647.png" alt="image-20231004153516581" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041535498.png" alt="image-20231004153502431" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041542959.png" alt="image-20231004154216886" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041542945.png" alt="image-20231004154245887" style="zoom:67%;" />

那个交叉的地方实际上就是可编程点。

---

## 4.9 组合逻辑电路中的冒险与竞争

### 4.9.1 竞争-冒险现象及其成因

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041602002.png" alt="image-20231004160210910" style="zoom: 67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041602033.png" alt="image-20231004160257963" style="zoom:50%;" />

​	<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041607665.png" alt="image-20231004160737584" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041609517.png" alt="image-20231004160919443" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041610196.png" alt="image-20231004161030139" style="zoom:67%;" />

### 4.9.2 检查竞争-冒险现象的方法

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041613556.png" alt="image-20231004161322491" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041614244.png" alt="image-20231004161402182" style="zoom: 67%;" />

**例题**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041615061.png" alt="image-20231004161518006" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041615170.png" alt="image-20231004161529124" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041617341.png" alt="image-20231004161705291" style="zoom:50%;" />

**结论**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041617713.png" alt="image-20231004161741615" style="zoom:50%;" />

### 4.9.3 消除竞争-冒险的方法

#### 一、接入滤波电容

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041620762.png" alt="image-20231004162058695" style="zoom:67%;" />

#### 二、引入选通脉冲

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041623633.png" alt="image-20231004162315566" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041623181.png" alt="image-20231004162334118" style="zoom:67%;" />

> 简单来说，就是，在产生脉冲的那个瞬间，我们不选择切换状态输出给Y3，而是等到一小会，等到P信号来了再输出出去。换句话说，我主动延时暂停，将主动权掌握在手上。

#### 三、 修改逻辑设计

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041628824.png" alt="image-20231004162802734" style="zoom:67%;" />

#### 四、四种方法的总结：

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041629965.png" alt="image-20231004162906880" style="zoom:67%;" />

> 本人觉得还是引入选通脉冲的方法好一些。尽管可能麻烦些，但还是希望能够将电路掌控在手中，不出意外。

---

# 五、半导体存储电路

## 5.1 概述

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041916757.png" alt="image-20231004191610642" style="zoom:67%;" />

半导体存储电路中使用的存储单元可以分为静态存储单元和动态存储单元两大类。

静态存储单元**由门电路连接而成**，其中包括各种电路结构形式的**锁存器和触发器**。只要不切断供电电
源,静态存储单元的状态会一直保持下去。

动态存储单元则是利用**电容的电荷存储效应**来存储数据的。由于电容的充放电需要一定的时间,因而它的工作速度低于静态存储单元。而且,**电容上存储的电荷会随着时间的推移而逐渐泄漏**,必须定期进行“刷新”(即将原来的数据重新写人),才能保证数据不会丢失。虽然如此,由于**动态存储单元的电路结构十分简单,**所以仍然被广泛用于大容量的存储器当中。

寄存器由**一组触发器**组成,每个触发器的输人和输出都有引出端,可以直接和周围电路连接,快速地进行数据交换。由**n个触发器组成的寄存器**可以存储**一组n位的二值数据**。

存储器的种类虽然很多,但它们的基本结构形式都是由**存储矩阵**和**读/写控制电路**两部分组成的。

首先,根据工作方式的不同,可以将存储器分为 **随机存储器(RandomAccessMemory,简称 RAM )** 和 **只读存储器( Read-OnlyMemory , 简称 ROM )**两大类。

随机存储器的工作特点是**可以随时从其中快速地读出或写人数据**。随机存储器又分成**静态随机存储器( StaticRandomAccessMemory ,简称 SRAM )**和**动态随机存储器( DynamieRandomAccessMemory ,简称 DRAM)** ，静态随机存储器中采用的是**静态存储单元**,而动态存储器中采用的则是**动态存储单元**。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041931825.png" alt="image-20231004193143704" style="zoom:50%;" />

## 5.2 SR存储器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041936101.png" alt="image-20231004193622005" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041936276.png" alt="image-20231004193633205" style="zoom:67%;" />

![image-20231004194046393](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041940470.png)

> Vo2一开始是低电平。
>
> 若V11是低电平，Vo1是高电平——Vo2还是低电平
>
> 若V11是高电平，Vo1是低电平——Vo2是高电平（或非门，在V11通的时候，Vo2通什么其实不影响）。当V11断开以后（相当于是低电平），Vo2由于是高电平，仍然会使得电路状态会保持在原状态，整个链路的数据仍然保持。



<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310041952927.png" alt="image-20231004195248870" style="zoom:67%;" />

<img src="C:\Users\32620\AppData\Roaming\Typora\typora-user-images\image-20231004195604224.png" alt="image-20231004195604224" style="zoom:67%;" />

由图可知，只有单个输入时，或非门相当于是反相器。因此Rd输入0，Q得出为1；Sd输入1，Q'得出是0。

此时断掉Sd输出，使之为低电平。那么，由于Q'=0，则输出Q仍然是1。状态仍然不变。**此时电路是1状态**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042013364.png" alt="image-20231004201340312" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042014945.png" alt="image-20231004201433885" style="zoom:67%;" />

![image-20231004201641632](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042016688.png)

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042018384.png" alt="image-20231004201832329" style="zoom:67%;" />

> 注意，这里的Q不是那个锁存器的输出！是电路的 **1**态还是 **0**态！那个Q星号表示的是电路新的状态，不是Q输出的反相数值！

<img src="C:\Users\32620\AppData\Roaming\Typora\typora-user-images\image-20231004203319788.png" alt="image-20231004203319788" style="zoom: 50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042038606.png" alt="image-20231004203812545" style="zoom: 67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042039500.png" alt="image-20231004203952445" style="zoom:50%;" />

<img src="C:\Users\32620\AppData\Roaming\Typora\typora-user-images\image-20231004204003921.png" alt="image-20231004204003921" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042042459.png" alt="image-20231004204251379" style="zoom:67%;" />

## 5.3 触发器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042050992.png" alt="image-20231004205012932" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042050821.png" alt="image-20231004205027766" style="zoom:50%;" />

### 5.3.1 电平触发的触发器

#### 一、电路结构和工作原理

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042053096.png" alt="image-20231004205330041" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042053651.png" alt="image-20231004205349598" style="zoom: 80%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042055139.png" alt="image-20231004205538083" style="zoom:67%;" />

> 这里还是很巧妙的运用了与非门的特点。**与门/与非门** 和 **或门/或非门** 都有一个特点，就是可以被某个输入信号锁死（控制段必备）

这里也很恰巧，只用了一个与非门。因为以与非门构建的SR存储器，确实也是以低电平为有效输入。

<img src="C:\Users\32620\AppData\Roaming\Typora\typora-user-images\image-20231004214646749.png" alt="image-20231004214646749" style="zoom: 50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042147837.png" alt="image-20231004214700772" style="zoom: 50%;" />

**异步电路（FPGA不需要这个）**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042148824.png" alt="image-20231004214800768" style="zoom: 50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042153921.png" alt="image-20231004215326839" style="zoom:50%;" />

#### 二、电平触发方式的动作特点

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042206602.png" alt="image-20231004220644526" style="zoom:50%;" />

> 这里我差点把与非门的SR与或非门的SR搞混了

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310042209281.png" alt="image-20231004220917228" style="zoom: 50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310050956890.png" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310050956054.png" alt="image-20231005095635998" style="zoom:67%;" />

##### **电平触发D锁存器**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051001308.png" alt="image-20231005100148248" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051002648.png" alt="image-20231005100203596" style="zoom:67%;" />

> 这样一来就能保证 S 与 R的输入是绝对反相的了，但就不会出现同时造成维持态（即SR与非门存储器两段都输入1）。这个维持态由CLK来实现。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051008781.png" alt="image-20231005100813723" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051008024.png" alt="image-20231005100838974" style="zoom:67%;" />

<img src="C:\Users\32620\AppData\Roaming\Typora\typora-user-images\image-20231005100900781.png" alt="image-20231005100900781" style="zoom:67%;" />

##### **CMOS传输门构建的D触发器**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051013534.png" alt="image-20231005101350432" style="zoom: 50%;" />

> 你不得不感叹前人的智慧。
>
> 仅通过一个SR触发器，一步步推到了D触发器，这实在是一种很伟大的进步。
>
> SR触发器的锁存功能确实是开天辟地的。我们甚至能看到，这个D锁存器仍然有着SR触发的那种思想。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051022452.png" alt="image-20231005102202377" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051022646.png" alt="image-20231005102213567" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051022752.png" alt="image-20231005102226676" style="zoom:50%;" />

### 5.3.2 边沿触发的触发器

#### 一、电路结构和工作原理

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051029426.png" alt="image-20231005102940333" style="zoom: 50%;" />

![image-20231005103619816](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051036895.png)

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051042790.png" alt="image-20231005104227707" style="zoom:67%;" />

> 由于CLK1=1，此时FF1的输出Q等于D，并且此时CLK2=0，此时Q2保持原来的状态不变

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051044977.png" alt="image-20231005104401901" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051044325.png" alt="image-20231005104456259" style="zoom: 67%;" />

> CLK1=0时，FF1的输出Q此时不再受D输入控制，而是保持原状态。CLK2=1，此时FF2输出Q2 由FF1的Q1控制。为什么是这样，请查看上文电平D触发器的特性表/真值表

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051045137.png" alt="image-20231005104525066" style="zoom:67%;" />

> D型号锁存器（也称为FF）
>
> **在CLK=1时，D型锁存器的次态只与D输入有关，与原态无关。**这是因为D型锁存器是一种数据锁存器，它会在时钟脉冲的上升沿或下降沿将D输入的值传递到Q输出。所以，只要CLK=1，D输入的任何变化都会立即反映在Q输出上，与原态无关。这使得D型锁存器在数字电路中非常有用，特别是在需要数据存储和传输的应用中。
>
> 有的时候要仅看逻辑图就要知道它的功能！知道内部原理确实很重要，但逻辑图也很重要！

关于这整一个的原理解释：

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051118432.png" alt="image-20231005111820321" style="zoom:67%;" />

低电平时，新的数据在 **FF1处** 输出，**FF2** 输出上一阶段的旧的数据。

高电平时，**FF1处** 输出上次的数据， **FF2** 输出 **此时FF1传过来的数据**

举个例子。

低电平时，我往 **FF1** 持续输入一个正弦信号，而 **FF1** 也是正常是将这个正弦信号输出给 **FF2** 的输入端口。但 **FF2** 不接收这个正弦信号，**FF2**输出的是上一个阶段，它得到的信号（总之不是这个时候的正弦）。

高电平时，**FF1** 不接收信号，阻隔输入端口，此时它将持续输出上一个阶段的 **正弦信号**。而此时的 **FF2** 放开，它接受该正弦信号，并且再将它输出到输出段。

切换回低电平。**FF2** 将持续对外输出该正弦信号，但自身将不再接受来自 **FF1** 的信号。相应的，**FF1** 又开始追求 **FF2** 了（舔狗不得house）

 ![image-20231005113551549](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051135618.png)

别觉得这种东西不重要，大家都是看集成的。所以要学会看这些b符号。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051136396.png" alt="image-20231005113649336" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051138286.png" alt="image-20231005113812208" style="zoom:67%;" />

> 在数字电路中，异步置位和复位功能的实现是非常重要的。主要原因如下：
>
> - 确定的初始状态：复位信号在数字电路中的重要性仅次于时钟信号。对一个芯片来说，**复位的主要目的是使芯片电路进入一个已知的，确定的状态。**这主要是为了让触发器进入确定的状态。
> - 错误状态恢复：如果电路发生了异常，比如状态不正常，中断异常，firmware程序跑飞，这个时候就可以对电路进行复位，让它**从错误的状态回到一个正常的状态。**
> - 不依赖时钟：**异步复位不依赖于时钟**。所以如果时钟是外部输入的，而且时钟有可能丢失，例如处于省电模式时，只能使用异步复位。
> - 设计更快的物理实现：相对于同步复位，**异步复位有更宽松的时序约束**。从而布局布线工具使用更少的时间便可达到约束条件。
> - 资源优化：一般来说，同步系统都使用异步复位。这是因为同步复位的电路实现，比异步复位的电路实现，要浪费更多电路资源。
>   总的来说，异步置位和复位功能在数字电路设计中起着至关重要的作用，它们确保了电路能够在任何情况下都能恢复到一个已知和可控制的状态。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202310051200434.png" alt="image-20231005120050356" style="zoom:67%;" />

<img src="C:\Users\32620\AppData\Roaming\Typora\typora-user-images\image-20231005120158317.png" alt="image-20231005120158317" style="zoom:67%;" />
