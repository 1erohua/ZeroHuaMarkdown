# 第2章 Verilog硬件描述语言（语法部分，会不断完善）

## 2.2 Verilog HDL 基本结构

```verilog
//首先是模块声明
//模块的声明要包括这个模块的名字，以及它的端口
module module_name (port_name1,port_name2,...);
    
//然后就是端口的定义，将端口说明为 输入端口 还是 输出端口
    input [width-1:0] 端口名 ;
    output [width-1:0] 端口名 ;
    inout [width-1:0] 端口名 ;
    
//信号类型声明——用于定义内部信号的数据类型（不会影响到外部）
    wire a,b,c;
    reg e,d,f;
    parameter s=10;//参数声明

//逻辑功能描述，类似于assign、always等语句
    always @(xxxx);
    begin
    end
    
    assign xxx= ;

//模块结束
endmodule
```

> 对于端口的数据类型，没有定义时默认是 **线网型数据类型**，且 **输入端口只能是线网型**

> **组合逻辑**的输出端口只能定义成 **线网型**。
>
> **时序逻辑**的输出端口应该定义成  **寄存器型。**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062008767.png" alt="image-20231106200808724" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062008956.png" alt="image-20231106200833914" style="zoom:67%;" />

​	<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062009983.png" alt="image-20231106200959931" style="zoom: 67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062010703.png" style="zoom:50%;" />



## 2.3 模块与声明

### 2.3.1 模块与声明



#### 模块命名的规则

将模块单词的每个 **首字母**组合起来，形成一个 3到5 个字符的缩写。

如果模块的英文名只有一个单词，**那么可以取这个单词的前三个字母。**



#### 模块端口的链接规则

模块的端口 是用来 连接 **另一个端口的外部** 与  **自身的内部**。

**有两种连接方法**，一种是，就像是C语言中的函数一样，按顺序依次放入括号内。

比如我有一个全加器模块

```verilog
module full_Adder(in1,in2,cin,cout,sum);
//描述
endmodule
```

如果使用 **信号传输命名法（信号顺序对应）**，就是相当于C语言那样一一对应

```verilog
full_Adder add1(x,y,z,s,t);
//其中x对应的in1，y对应其中的in2.。。。
```

> 简单，但在大型系统中容易出错。

但是，如果是 **命名端口连接**，就是类似于，我指定该模块内的某个端口与我想要的端口连接，**相当于是我直接指名道姓**，不需要一一指定。并且没有指定到的端口直接进行悬空。

```verilog
full_Adder add1(.in1(x),.in2(y),.cin(z),.cout(s),.sum(t));
```

就是类似于，**.被调用端口（调用模块需要的连接）**

![image-20231105195334001](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311051953138.png)

 

#### 模块划分

主模块内仅包含 **模块I/O端口**，**内部连线** 和 **子模块**。

子模块间 **不应该寄生逻辑**，禁止在主模块内设置逻辑功能。

```verilog
// 主模块
module TopModule (
    input wire clk,
    input wire reset,
    input wire [7:0] data_in,
    output wire [7:0] data_out
);
    // 内部连线
    wire [7:0] data_to_submodule1;
    wire [7:0] data_from_submodule2;

    // 子模块1
    SubModule1 u_submodule1 (
        .clk(clk),
        .reset(reset),
        .data_in(data_in),
        .data_out(data_to_submodule1)
    );

    // 子模块2
    SubModule2 u_submodule2 (
        .clk(clk),
        .reset(reset),
        .data_in(data_to_submodule1),
        .data_out(data_from_submodule2)
    );

    // 连接子模块2的输出到主模块的输出
    assign data_out = data_from_submodule2;
endmodule

// 子模块1
module SubModule1 (
    input wire clk,
    input wire reset,
    input wire [7:0] data_in,
    output wire [7:0] data_out
);
    // 子模块1的逻辑功能
    // ...
endmodule

// 子模块2
module SubModule2 (
    input wire clk,
    input wire reset,
    input wire [7:0] data_in,
    output wire [7:0] data_out
);
    // 子模块2的逻辑功能
    // ...
endmodule
```

### 2.3.2 信号的命名

全局系统信号以**字符串sys**开头

复位信号以 **rst** 和 **reset** 为标识符

置位信号以 **set** 标识

时钟信号一般以**clk**来表示，并在后面添加相应的的频率值进行区分信息，例如`clk_freq_200m`

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311052027034.png" alt="image-20231105202715917" style="zoom:50%;" />

### 2.3.3 端口的声明

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311052028958.png" alt="image-20231105202808899" style="zoom: 67%;" />



### 2.3.4 变量的声明

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311052032438.png" alt="image-20231105203240353" style="zoom:50%;" />

> aes_output就是变量，就是使用的时候才声明的。
>
> assign是 用于创建连续赋值语句，因为赋值一般是在逻辑块内进行，没办法在模块内进行，后续会讲的。

----



## 2.4 数据类型

### 2.4.1 数字声明

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311052058351.png" alt="image-20231105205837299" style="zoom:67%;" />

![image-20231105205949806](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311052059863.png)

> 一定要注意，这里的 **位宽**，指的是，将该数根据进制转换成二进制的 **位数**

![image-20231105210225660](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311052102728.png)

![image-20231105210246930](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311052102984.png)

> 总结：
>
> B/b（Binary）——二进制
>
> O/o（Octal）——八进制
>
> D/d（Decimal）——十进制
>
> H/h（Hexadecimal）——十六进制

### 2.4.2 数值逻辑

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311052151219.png" alt="image-20231105215133146" style="zoom: 67%;" />

### 2.4.3 常量数据类型

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311052205901.png" alt="image-20231105220525832" style="zoom:67%;" />

```verilog
integer i;
for (i = 0; i < 10; i = i + 1) begin
    // 循环体
end
```

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311060857642.png" alt="image-20231106085744546" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311061945867.png" alt="image-20231106194539806" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311061948273.png" alt="image-20231106194857220" style="zoom:50%;" />

### 2.4.4 数据类型

#### 1.线网型

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311060900675.png" alt="image-20231106090041573" style="zoom:67%;" />

> 这里线的位宽就当成是一维数组。后面会有实例来解释这到底是怎么回事。

> 还是要提醒一下
>
> input只能是线网型
>
> output端口可以是寄存器型或线网型
>
> inout只能是线网型

> 常用的网络数据类型包括wire型和tri型。 wire型变量通常是用来表示**单个门驱动**或**连续赋值语句驱动的网络型数据**，tri型变量则用来表示**多驱动器驱动**的网络型数据。
>
> **如果wire型或tri型变量没有定义逻辑强度(logic strength)，在多驱动源的情况下， 逻辑值会发生冲突从而产生不确定值。**(这里的逻辑强度没听懂)
>
> 在Verilog中，逻辑强度是一种用于描述电路行为的概念，特别是在开关级仿真中。当一个net（网络）由多个驱动器驱动且其值互相矛盾时，可以使用强度的概念来描述这种逻辑行为。
>
> 逻辑强度分为驱动强度和充电强度。驱动强度有多种级别，包括**supply（最强）、strong、pull、weak，以及highz（最弱）。这些强度级别可以用于定义net的驱动强度。**
>
> 例如，当需要确定net的实际逻辑值和强度时，或者当net由多个驱动器驱动而且驱动相互间出现冲突时，出现冲突的两个强度值在强弱顺序表中的相对位置就会对该net的真实逻辑值起作用。
>
> 需要注意的是，**强度是不可综合的**，也就是说，它主要用于仿真，而不是用于硬件描述语言（HDL）的综合。此外，你可以在$display和$monitor等中用特定的格式控制符%V显示其强度值。
>
> > 看不懂后面再说

#### 2.寄存器型

![image-20231106092630119](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311060926240.png)

> 因为寄存器本身是抽象级的存在，所以应该放入 同样抽象层次较高的 行为级描述
>
> reg类型数据的缺省初始值为不定值，x。  

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062038342.png" alt="image-20231106203834294" style="zoom: 67%;" />

#### 3.参数

![image-20231106193703718](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311061937846.png)

> 大概看一眼，有需要再回来看看

![image-20231106200525470](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062005523.png)

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062014378.png" alt="image-20231106201458340" style="zoom:50%;" />

```verilog
//看得我大受震撼
module Test_1
    wire wire1;
    Test_2 test2();
endmodule

module Test_2
    wire wire2;
    parameter p=0;
endmodule

module Tes_3
    defparam
    Test_1.test2.p=1;//这里就像是？调用了一个类，然后再调用这个类里面的实例元素，最后改变这个实例元素里面的参数。
endmodule
    
```

#### 4. 存储器（reg型数组）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062041321.png" alt="image-20231106204138264" style="zoom: 50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062047493.png" alt="image-20231106204742437" style="zoom: 50%;" />

## 2.5 运算符及其表达式

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062049739.png" alt="image-20231106204954691" style="zoom: 50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062050552.png" alt="image-20231106205035506" style="zoom: 50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062053147.png" alt="image-20231106205304085" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062053266.png" alt="image-20231106205358197" style="zoom:50%;" />



# 第3章节 运算符、赋值语句和结构说明语句

## 3.1 逻辑运算符

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062057372.png" alt="image-20231106205723324" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062059841.png" alt="image-20231106205944794" style="zoom:67%;" />

## 3.2 关系运算符

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062101266.png" alt="image-20231106210109225" style="zoom: 67%;" />

## 3.3 等式运算符

![image-20231106210339104](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062103171.png)

> === 与 ==，以及!= 与 !==的差异

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062105567.png" alt="image-20231106210510523" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062105972.png" alt="	" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311062105175.png" alt="image-20231106210547139" style="zoom:50%;" />

## 3.4 移位运算符

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311070916435.png" alt="image-20231107091638346" style="zoom:67%;" />

> 注意，移位之后，数的位数将发生变化

> 移位运算符又分为算术移位与逻辑移位，上面个说的是逻辑移位
>
> [不过，`<<<` 和 `>>>` 是算术移位运算符，它们在移位时会考虑符号位](https://blog.csdn.net/Reborn_Lee/article/details/89813616)[1](https://blog.csdn.net/Reborn_Lee/article/details/89813616)[2](https://blog.csdn.net/weixin_43649647/article/details/120399079)[。例如，对于算术右移，如果符号位为1，就在左边补1；否则，就补0](https://blog.csdn.net/Reborn_Lee/article/details/89813616)[1](https://blog.csdn.net/Reborn_Lee/article/details/89813616)[。这使得算术右移可以进行有符号位的除法，右移 n 位就等于除以 2 的 n 次方](https://blog.csdn.net/Reborn_Lee/article/details/89813616)[1](https://blog.csdn.net/Reborn_Lee/article/details/89813616)。

## 3.5 位拼接运算符

![image-20231107092412425](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311070924480.png) 

![image-20231107092451228](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311070924275.png)

```verilog
module fulladder(a, b, cin, sum, cout);
  input a, b, cin;
  output sum, cout;
 
  assign {cout, sum} = a + b + cin;    // 进位输出与和拼接在一起
 
endmodule
//因为如果a+b+cin产生了溢出位，正好被参与拼接中的cout所接收，就起到了简化表达式的作用
```

> 比如3位拼接4位，得出结果是7位
>
> 即[2:0]拼接[3:0]，得出[6:0]；

## 3.6 缩减运算符

![image-20231107181354147](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311071813269.png)

> 顾名思义就是缩减，从低位往高一位位缩减，缩减的方式就是通过 **与或非**（非是指的，与非或非） 运算的结果

## 3.7 优先级别 

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311071816863.png" alt="image-20231107181632809" style="zoom: 67%;" />

## 3.9 赋值语句与块语句

### 3.9.1 赋值语句

#### 1.非阻塞赋值

在语句块中，上面语句所赋的变量值**不能立即**就为下面的语句所使用。

```verilog
b<=a;
c<=b;
//c无法立即获得a的值，它必须等下一个时间才能获得a的值的，而此时b的值已经不是a了
//类似移位寄存器
```

急死了，那么什么时候才能得到这个值呢？

只有**块结束之后**才能完成这次赋值操作，而所赋的变量值是上一次赋值得到的。

> 非阻塞赋值只能用在寄存器，因此只能在always和initial语句

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311071947614.png" alt="image-20231107194759505" style="zoom: 67%;" />

#### 2. 阻塞赋值

​	![image-20231107195054802](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311071950868.png)

#### 3. 两者的差异

![image-20231107195144639](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311071951741.png)

![image-20231107195410129](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311071954275.png)

> 这个例子很经典了。如果是阻塞赋值，a的值就会立刻给到c
>
> 并且，无论是顺序块还是并行块，非阻塞赋值都是并行进行。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311072002101.png" alt="image-20231107200230001" style="zoom: 50%;" />

### 3.9.2 块语句

#### 1. 顺序块

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311072008105.png" alt="image-20231107200835929" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311072012108.png" alt="image-20231107201256060" style="zoom:50%;" />

#### 2. 并行块

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311072015516.png" alt="image-20231107201542455" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311072016107.png" alt="image-20231107201616063" style="zoom:50%;" />

>  19:44:39
> 顺序块中的语句是一条条执行的。当然，非阻塞赋值除外。
>
>  19:44:46
> 并行块中的语句是并行执行的，即便是阻塞形式的赋值。
>
> 非阻塞赋值逼格就是高

#### 3. 块名

![image-20231107204327857](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311072043933.png)

> 当一个块被命名的时候，它的局部变量将可以被**层次化引用**，但这种层次化引用不可综合。
>
> 层次化引用就是像类的成员函数、成员字段那样，模块.命名块.局部变量

#### 4. 块的起始时间和结束时间

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311072100546.png" alt="image-20231107210009483" style="zoom: 67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311072100579.png" alt="image-20231107210037533" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311072100226.png" alt="image-20231107210057168" style="zoom:50%;" />

---



# 第五章 条件语句、循环语句、块语句与生成语句

## 5.1 条件语句

> 条件语句 必须在 **过程块** 中使用，所谓过程块语句就是initial语句与always语句的begin end块中可以编写条件语句之外，模块的其他地方都不能编写

就是简单的 if，else if，else

应当注意if与else的配对关系,else总是与它上面的最近的if配对。

**在综合过程中，条件语句会被综合成多个数据选择器**

一般当信号有明显优先级时首先考虑if else结构，但是if 嵌套过多会导致速度很慢，路径延时很大，因此一般条件较少时适用，**if else结构下最终综合的电路速度较慢，面积小**

![img](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311072114296.webp)

这里的就是被综合之后的结果

它的代码应该是这样：

```verilog
always @(*)
    begin
        if(x>y)
            d=a;
        else if(x<y)
            d=b;
        else
            d=c;
    end
```

 **而且还要注意一点，无论是现在的if还是后面的case，都不能忽略最终的 else和default，必须把所有情况包含在内。**

**因为一旦出现不包含的情况，就会产生锁存器，将原有数据保存起来。就是说出现了预期之外并且不满足每个if和else if的情况，而且你还没添加else，这时输出只能保持原态，就会产生锁存器，十分浪费资源。**

## 5.2 case 语句

直接给出综合后的电路图

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311081519752.png" alt="image-20231108151956691" style="zoom:50%;" />

case语句适用于无明显优先级的逻辑判断，这些逻辑条件都处于同一个优先级且互斥**，比如实现对速度要求较高的编解码；case结构电路速度较快，但占用面积较大。**综合为 n选1 mux电路。

> if-else：组合逻辑和时序逻辑中的always语句块中实现是不同的。
>
> 组合逻辑中：if缺少else 时，会有latch；
>
> **时序逻辑中：尽管缺少else，依旧是D触发器，不存在latch**。
>
> ```verilog
> always @( *) begin
>     if (sel=a) begin
>         out=q1;
>     end
> end
> ```
>
> **case语句：case列举不全并且还没写default语句，则会综合出锁存器。所以一定写default，无论是组合还是时序逻辑。**
>
> ``` verilog
> case (sel)
>     2'b00: 
>     2'b01:
>     2'b10:
>     //2'b11:
>     //default: 
> endcase
> ```
>
> 避免方法简单：review代码保证if-else对应齐全；case必写default

![image-20231108172517268](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311081725358.png)

> 第7条应该是相当重要的，即任何时候都应该将位宽写清楚

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311081806678.png" alt="image-20231108180606599" style="zoom: 67%;" />

> 就是说，比如我使用了casez，那么2'b01和2'b0z就是相同的，但是2'b01与2'b0x是不相同的
>
> 但如果是casex，无论是2'b0z还是2'b01还是2'b0x都是相同的。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311082120198.png" alt="image-20231108212025111" style="zoom: 67%;" />

> 可以将横轴看为 case括号里面的，然后将 竖轴 看为 case的冒号前面

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311082124152.png" alt="image-20231108212410091" style="zoom:60%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311082124706.png" alt="image-20231108212451655" style="zoom:67%;" />

## 5.5 循环语句

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311082126158.png" alt="image-20231108212623092" style="zoom: 50%;" />

> 这四个中，只有第三个才能被综合，其他几个只能用在仿真。

### 5.5.1 forever 语句

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311092019768.png" alt="image-20231109201950622" style="zoom:60%;" />

> 这个后面有机会讲，就不多讲了

### 5.5.2 repeat 语句

![image-20231109225403706](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311092254795.png)

> 表达式用于表示重复的次数
>
> 但真正牛逼的是下面的乘法器实现
>
> 充分阐释了二进制的乘法是如何做到的

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311092301948.png" alt="image-20231109230123896" style="zoom:67%;" />

> 首先光是这里低位赋值给高位已经让我大受震撼了，然而下面对于乘法移位的理解超出我的想象

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311092306927.png" alt="image-20231109230607874" style="zoom: 67%;" />

> 它是怎么实现乘法的？没错，就是加法，就是像我们手写多位数乘法的思维一样，写两个数，一个在上面，另一个在下面，在下面那个依次乘以上面那个的每个数
>
> 但是由于二进制只有0和1，因此，只要相乘的那个位是1，咱们就把上面的值加一遍
>
> 那么怎么表示“下面的每一个数乘以上面的所有呢？”
>
> 就是通过移位，上面的数每次都左移，就像乘了一个10；下面的数每次都右移，末位不断更新。

### 5.5.3 while 语句

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311100915424.png" alt="image-20231110091510363" style="zoom:67%;" />

> 用法与C语言中的相近，因此不做过多的赘述，表达式为真时会不断执行，直到表达式为假

### 5.5.4 for语句

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311100916036.png" alt="image-20231110091646992" style="zoom:67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311100917944.png" alt="image-20231110091711880" style="zoom:50%;" />

> 其实就和C语言的类似

## 5.6 顺序块与并行块



### 5.6.1 块语句类型

#### 1. 顺序块

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311100920099.png" style="zoom: 50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311100920800.png" alt="image-20231110092050754" style="zoom:67%;" />

#### 2. 并行块

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311100922444.png" alt="image-20231110092206381" style="zoom: 50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311100923642.png" alt="image-20231110092343588" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311100924229.png" alt="image-20231110092437155" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311100925837.png" alt="image-20231110092559789" style="zoom: 67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311100926336.png" alt="image-20231110092609287" style="zoom:50%;" />

### 5.6.2 块语句的特点

#### 1.嵌套

块可以嵌套使用

![image-20231110113520362](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101135407.png)

#### 2.命名

块可以被命名，被命名的块可以层级调用。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101137475.png" alt="image-20231110113749424" style="zoom: 50%;" />

#### 3. 命名块的禁用

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101139129.png" alt="image-20231110113929077" style="zoom:50%;" />

> 这个我还没看过，可以好好了解一下。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101141107.png" alt="image-20231110114143050" style="zoom: 50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101141082.png" alt="image-20231110114151019" style="zoom:50%;" />

> 牛的，正好是符合了那句：使用disable可以禁用设计中任意一个命名块



## 5.7 生成块（可综合）

> 说实话，我一开始都没明白这个生成块讲得什么够吧意思，后面才发现，这里是用来连续创建实例的

下面可能会看的有点蒙圈，没关系，先看着，等把5.7全部看完再回头看就明白了	

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101849016.png" alt="image-20231110184949895" style="zoom:50%;" />

> 我举一个简单的例子，比如说我要对2个四位输入进行按位与运算
>
> 那么是不是要实例化4个与门对吧
>
> 但用生成语句只需要实例化1个与门，然后使用循环执行4遍，实际上也是生成了4个与门

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101853185.png" alt="image-20231110185305104" style="zoom:50%;" />

> 接着上面的那个例子，这4个与门都有自己唯一的标识名。
>
> 比如说我在生成语句中实例化的与门名为and4in
>
> 那么第一个与门是 and4in[0]

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101853222.png" alt="image-20231110185336161" style="zoom:50%;" />

> 这个我不了解，不细说

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101859331.png" alt="image-20231110185913270" style="zoom:50%;" />

> 我首先剧透一下吧
>
> 因为硬件语言中，你使用的与或非门这种器件，它不是像函数一样调用的，它是实例化出来的。
>
> 你需要用的时候就实例化一个，是直接画在电路图上的。
>
> 如果你只创建一个与门，然后进行4次按位与（纯循环无生成块），那么结果肯定是第4次的结果。（它就一个普普通通的2输入1输出的与门，你让他输出4位？）
>
> 但是用生成块的循环生成，它是展开成4个与门，每个与门的名字分别为（假设生成块中与门实例名为and1）：and1[0],and[1],and[2],and[3]
>
> 而条件生成则是只是在各个条件中根据常数表达式生成符合条件的电路。但if是生成各个条件的电路，然后根据表达式进行选择。
>
> 简单来讲，用某个参数的值来判断是生成加法还是减法器，if是先都生成，然后再根据数据选择器的结果判断；generate if则是先确定是哪个条件，再生成对应的那个条件的电路

### 5.7.1 循环生成语句

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101918983.png" alt="image-20231110191841917" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101922483.png" alt="image-20231110192230425" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101924349.png" alt="image-20231110192409288" style="zoom:50%;" />

> 你可以将generate拿掉，结果就是轮流对着g1一个门进行输出，那请问g1就一个门，面对4轮2输入1输出，它应该选择哪一个呢？它选择了其中一个，其他的该怎么办？
> generate的作用就是让你少写3个与门

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101937378.png" alt="image-20231110193737289" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101939808.png" alt="image-20231110193924732" style="zoom:50%;" />

### 5.7.2 条件生成语句

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101942442.png" alt="	" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101942052.png" alt="image-20231110194242997" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101951319.png" alt="image-20231110195153240" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311101954175.png" alt="image-20231110195450112" style="zoom:50%;" />

> 没错，用C语言的话来说，generate if就是在编译的时候完成条件的选择的
>
> 而普通的if选择则是在运行时进行选择
>
> 由于使用parameter进行的变量的定义，因此我们可以从外部进行改变，从而改变条件的选择

借用别人博客的图·，这是使用generate if的结果

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311102004124.png" alt="image-20231110200447050" style="zoom:50%;" />

> 电路的综合结果是二选一

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311102012165.png" alt="image-20231110201256083" style="zoom:50%;" />

> 但这个却是纯if综合的结果

### 5.7.3 case语句

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311102028295.png" alt="image-20231110202815224" style="zoom:50%;" />

> 其实结合if和case的差异就不难得出generate case是怎么样的

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311102029979.png" alt="image-20231110202904919" style="zoom:50%;" />

> **它这里是 “#” 不是用来改变时序的，而是用来改变内部的参数的**
>
> 具体用法大概如下：

### 横插一脚：关于#的改变参数的用法

在Verilog中，`#`符号可以用于改变模块内的参数值。具体来说，有两种主要的方法：

1. [**实例化时传递参数**](https://blog.csdn.net/sweet_jr/article/details/60149954)[1](https://blog.csdn.net/sweet_jr/article/details/60149954)[2](https://www.cnblogs.com/PhiloSky/p/3390559.html)[3](https://blog.csdn.net/CLL_caicai/article/details/104440915)：在这种方法中，实例化模块时将参数传递进去。例如：

```verilog
module top (.....)
    input....;
    output....;
    M1 ＃(10) U1 (..........);
endmodule
```

在上述例子中，`#(10)`修改了参数`para1`的值。当有多个参数时，可以用逗号隔开，如`＃(10, 5)`。

> 这里的传递参数同时也适用于 前面说到的两种 **信号传输命名** 和 **命名端口连接**

2. [**使用`defparam`重定义参数**](https://blog.csdn.net/sweet_jr/article/details/60149954)[1](https://blog.csdn.net/sweet_jr/article/details/60149954)：`defparam`可以用于重新定义低层模块的参数值。例如：

```verilog
module top (.....)
    input....;
    output....;
    defparam U1.Para1 = 10;
    M1 U1 (..........);
endmodule
```

在上述例子中，`defparam U1.Para1 = 10;`将参数`Para1`的值改为10。

[这两种方法都可以在顶层模块调用子模块时进行参数修改](https://blog.csdn.net/sweet_jr/article/details/60149954)[3](https://blog.csdn.net/CLL_caicai/article/details/104440915)。这样，你就可以在Verilog中使用`#`来改变参数了。



然后我发现，它用parameter定义的参数居然也可以写在模块名的后面！如下！

> [在Verilog中，实例化时传递参数的顺序和模块定义中参数的顺序是一致的。例如，如果你有一个模块定义如下](https://www.runoob.com/w3cnote/verilog-defparam.html)[1](https://www.runoob.com/w3cnote/verilog-defparam.html)[2](https://blog.csdn.net/xs1326962515/article/details/78223061)[3](https://www.cnblogs.com/agllero/p/5976600.html)[4](https://blog.csdn.net/weiweiliulu/article/details/24106955)：
>
> ```verilog
> module M #(parameter para1 = 1, parameter para2 = 2);
> ```
>
> 那么在实例化时，你可以这样传递参数：
>
> ```verilog
> M #(3, 4) instance_name (/* port connections */);
> ```
>
> 在这个例子中，`para1`被赋值为3，`para2`被赋值为4。
>
> [另外，你也可以通过参数名来指定参数值，这样就不需要关心参数的顺序](https://www.runoob.com/w3cnote/verilog-defparam.html)[1](https://www.runoob.com/w3cnote/verilog-defparam.html)[2](https://blog.csdn.net/xs1326962515/article/details/78223061)[3](https://www.cnblogs.com/agllero/p/5976600.html)[4](https://blog.csdn.net/weiweiliulu/article/details/24106955)。例如：
>
> ```verilog
> M #(.para2(4), .para1(3)) instance_name (/* port connections */);
> ```
>
> 在这个例子中，即使`.para2`在`.para1`之前，`para1`仍然被赋值为3，`para2`被赋值为4。
>
> 总的来说，你可以通过参数的顺序或者参数的名字来确定你想要传递的实参。希望这个答案对你有所帮助！

---



# 第6章 结构语句、系统任务和函数语句以及显示系统任务

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311102044408.png" alt="image-20231110204404335" style="zoom:50%;" />

> 一定要以电路思维去思考Verilog
>
> 

> **关于例6.15 常量函数的解释**
>
> 
>

> 



> $display 和 $strobe的区别
>
> 在Verilog中，`$display`和`$strobe`都是用于打印调试信息的系统任务，但它们的工作方式有所不同。
>
> - [`$display`：这个任务的使用方法和C语言中的`printf`函数非常类似，可以直接打印字符串，也可以在字符串中指定变量的格式对相关变量进行打印](https://www.runoob.com/w3cnote/verilog2-display.html)[1](https://www.runoob.com/w3cnote/verilog2-display.html)[。当许多语句与`$display`任务在同一时间内执行时，这些语句和`$display`的执行顺序是不确定的，一般按照程序的顺序结构执行](https://www.runoob.com/w3cnote/verilog2-display.html)[1](https://www.runoob.com/w3cnote/verilog2-display.html)[2](https://www.cnblogs.com/Alfred-HOO/articles/17455599.html)[3](https://blog.csdn.net/wuzhikaidetb/article/details/125340502)。
> - [`$strobe`：这个任务的使用方法与`$display`一致，但打印信息的时间和`$display`有所差异](https://www.runoob.com/w3cnote/verilog2-display.html)[1](https://www.runoob.com/w3cnote/verilog2-display.html)[2](https://www.cnblogs.com/Alfred-HOO/articles/17455599.html)[3](https://blog.csdn.net/wuzhikaidetb/article/details/125340502)[。`$strobe`是在其他语句执行完毕之后，才执行显示任务](https://www.runoob.com/w3cnote/verilog2-display.html)[1](https://www.runoob.com/w3cnote/verilog2-display.html)[2](https://www.cnblogs.com/Alfred-HOO/articles/17455599.html)[3](https://blog.csdn.net/wuzhikaidetb/article/details/125340502)。这就意味着，如果在同一时间点有多个语句和`$strobe`任务一起执行，`$strobe`会等到所有其他语句都执行完毕后才执行。
>
> [因此，`$display`和`$strobe`的主要区别在于它们执行的时间点：`$display`会立即执行，而`$strobe`会等待当前时间单位结束后执行](https://www.runoob.com/w3cnote/verilog2-display.html)[1](https://www.runoob.com/w3cnote/verilog2-display.html)[2](https://www.cnblogs.com/Alfred-HOO/articles/17455599.html)[3](https://blog.csdn.net/wuzhikaidetb/article/details/125340502)[。这种差异可能会影响打印出的调试信息，特别是在处理并行操作或者非阻塞赋值时](https://www.runoob.com/w3cnote/verilog2-display.html)[1](https://www.runoob.com/w3cnote/verilog2-display.html)。所以，在选择使用`$display`还是`$strobe`时，需要根据具体的调试需求和仿真环境来决定。



## 6.1 结构说明语句

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311111707727.png" alt="image-20231111170713606" style="zoom:50%;" />

### 6.1.1 initial 语句

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311111708397.png" alt="image-20231111170803333" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311111708593.png" alt="image-20231111170823534" style="zoom:50%;" />

> 多个initial块并行运行。

### 6.1.2 always 语句

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311111710251.png" alt="image-20231111171031192" style="zoom:50%;" />

> "always" 总是——总是执行，但有条件，可以放入组合逻辑和时序逻辑

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311111711794.png" alt="image-20231111171115738" style="zoom:50%;" />

> 总不能让它总是死锁，所以需要条件

**两种触发方式**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311111713282.png" alt="image-20231111171300210" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311111715115.png" alt="image-20231111171535040" style="zoom:50%;" />

> 省流：
>
> 1. 沿触发的always常用来表示时序逻辑
> 2. 电平触发的always常用来表示组合逻辑
> 3. 同一个模块中，always并行执行。

[然而，需要注意的是，如果多个`always`块使用相同的敏感信号（sensitivity list），那么它们的执行顺序是未定义的（undefined order），这意味着编译器可以按任意顺序执行这些`always`块](https://zhuanlan.zhihu.com/p/35442938)[（网址链接）](https://bing.com/search?q=verilog中，多个always块是并行的吗？为什么？多个initial块呢？)[。因此，应该避免在同一个模块中使用多个`always`块来驱动同一个敏感信号](https://zhuanlan.zhihu.com/p/35442938)[（网址链接）](https://bing.com/search?q=verilog中，多个always块是并行的吗？为什么？多个initial块呢？)。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311111718101.png" alt="image-20231111171849038" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311111718762.png" alt="image-20231111171855708" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311111719499.png" alt="image-20231111171907442" style="zoom:50%;" />

> **@*的使用细节与限制**
>
> 在Verilog中，`@*`是一种隐式事件控制语句，它会自动感知当前语句块中所有的右值（即被读取的变量）并在这些变量发生变化时触发。这种语法可以使代码更简洁，但也有一些限制和注意事项：
>
> 1. `@*`只能在`always`或`initial`块中使用。
> 2. `@*`会自动感知当前语句块中所有的右值，包括条件判断语句中的变量。因此，如果语句块中的某个变量值的改变并不需要触发整个语句块，那么使用`@*`可能会导致过度触发。
> 3. `@*`不能感知到在函数或任务中使用的变量，因为函数和任务是独立的语句块。
> 4. 在使用`@*`时，需要注意避免产生竞争条件。例如，如果两个`always`块都使用`@*`并且读写同一个变量，那么可能会产生竞争条件。
>
> [总的来说，`@*`是一种强大的工具，可以使Verilog代码更简洁，但使用时也需要注意以上的限制和潜在问题。](https://zhuanlan.zhihu.com/p/129243480)[1](https://zhuanlan.zhihu.com/p/129243480)[2](https://www.zhihu.com/question/309627605)[3](https://zhuanlan.zhihu.com/p/614048441)[4](https://zhuanlan.zhihu.com/p/482762500)。如果你有更具体的问题或者需要关于某个特定情况的建议，欢迎随时向我提问！

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311111720057.png" alt="image-20231111172040983" style="zoom:50%;" />

> 在Verilog中，`wait`语句用于暂停当前线程的执行，直到某个条件满足为止。`wait`语句的一般形式是 `wait (expression);`，其中 `expression` 是一个布尔表达式。当 `expression` 的结果为真时（即非零），`wait` 语句结束，程序继续执行。如果 `expression` 的结果为假（即零），则程序在 `wait` 语句处暂停，直到 `expression` 的值改变并且结果为真为止。
>
> [以下是一些关于 `wait` 语句的重要注意事项](https://blog.csdn.net/kebu12345678/article/details/86560175)[网页链接](https://blog.csdn.net/kebu12345678/article/details/86560175)：
>
> 1. `wait` 语句只能在 `initial` 和 `always` 块中使用。
> 2. `wait` 语句会阻塞当前线程的执行，直到等待的条件满足。这可能会导致模拟时间的停滞，因此在使用 `wait` 语句时需要特别小心。
> 3. `wait` 语句的条件表达式中应该包含至少一个发生变化的变量，否则 `wait` 语句可能会导致死锁。
> 4. 在 `wait` 语句中使用阻塞赋值可能会导致未定义的行为，因此通常建议在 `wait` 语句中使用非阻塞赋值。



## 6.2 task和function说明语句

### 6.2.1 task和function语句的区别

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311111748412.png" alt="image-20231111174829315" style="zoom:50%;" />

> 1. 任务常用来可以实现时序逻辑，但函数只能实现组合逻辑
> 2. 任务不返回值，但它可以改变参数的值，类似于C语言传递地址给函数，然后函数根据地址改变地址所对应的值；函数则是必须要返回一个值。
> 3. 函数必须要有一个输入变量，而任务不需要。因为函数是时刻反映的。



### 6.2.2 task说明语句

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121056845.png" alt="image-20231112105657763" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121057726.png" alt="image-20231112105712671" style="zoom:50%;" />

**通过任务实现一个红绿灯转换**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121058789.png" alt="image-20231112105822726" style="zoom:50%;" />

> 定义数据并初始化

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121108717.png" alt="image-20231112110845658" style="zoom:50%;" />

> 产生时钟信号

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121109737.png" alt="image-20231112110926679" style="zoom:50%;" />

> 注意一下，它这里没有always，因此不是什么时钟上升沿触发的意思。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121115250.png" alt="image-20231112111522184" style="zoom:50%;" />

> 任务的调用。

### 横插一脚：@符号是什么意思

[在Verilog中，`@`符号表示在某一时刻。它可以用于仿真时刻，以产生某一事件](https://zhidao.baidu.com/question/154181157.html)[1](https://zhidao.baidu.com/question/154181157.html)[2](https://blog.csdn.net/qq_34159047/article/details/111873301)。例如：

```verilog
@(posedge clk) input=1;
@(negedge clk) input=0;
```

[上述代码表示在一个`clk`的上升沿，输入为1，在接下来的一个下降沿，输入为0](https://zhidao.baidu.com/question/154181157.html)[1](https://zhidao.baidu.com/question/154181157.html)。

[此外，`@`符号也可以用在`always`块后面，表示敏感信号列表。当这些信号发生改变时，`always`块里的语句顺序执行一遍](https://zhidao.baidu.com/question/154181157.html)[1](https://zhidao.baidu.com/question/154181157.html)。例如：

```verilog
always @(sl or a or b) // 表示只要 sl 或 a 或 b 其中若有一个变化时就执行下面的语句。
```

> 总的来说，@就是在括号内的信号符合时，产生某一事件。当信号没来的时候，就不会执行。



### 6.2.3 function说明语句

**函数的目的是返回一个表达式的值**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121118892.png" style="zoom:50%;" />

> 内置与函数名同名的寄存器

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121118959.png" alt="image-20231112111845883" style="zoom:50%;" />

> 这里有个说法，后面会讲到，函数的递归调用是有条件的。

**例子：左右移位寄存器**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121122883.png" alt="	" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121122987.png" alt="image-20231112112221928" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121122573.png" alt="image-20231112112243513" style="zoom:50%;" />

> **关于宏定义'define的注意事项：**
>
> [在Verilog中，`define是一个编译器指令，用于执行全局宏替换，并在宏定义后读取/编译的所有文件中保持活动状态。它将一直可用，直到另一个宏定义更改了值，或者直到使用`undef编译器指令取消定义宏](https://verificationguide.com/systemverilog/systemverilog-parameters-and-define/)[1](https://verificationguide.com/systemverilog/systemverilog-parameters-and-define/)[2](https://www.design-reuse.com/articles/45979/system-verilog-macro-a-powerful-feature-for-design-verification-projects.html)。
>
> 例如，你可以这样定义一个宏：
>
> ```verilog
> `define WIDTH 8
> ```
>
> 然后在你的代码中，你可以通过(`)字符后跟宏名称来调用宏：
>
> ```verilog
> input [`WIDTH-1:0] addr;
> ```
>
> 在这个例子中，`WIDTH将被替换为8。
>
> [你还可以使用`ifdef，`ifndef，`else等指令来检查宏是否已经定义，以避免重复定义](https://verificationguide.com/systemverilog/systemverilog-parameters-and-define/)[1](https://verificationguide.com/systemverilog/systemverilog-parameters-and-define/)。例如：
>
> ```verilog
> `ifdef WIDTH
> // 如果WIDTH已经定义，那么就不做任何事情
> `else
> `define WIDTH 8
> `endif
> ```
>
> 在这个例子中，如果WIDTH已经定义，那么就不会再次定义它。否则，它将被定义为8。



### 6.2.5 自动函数与静态函数

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121124162.png" alt="image-20231112112406083" style="zoom:50%;" />

![image-20231112112419010](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121124082.png)

### 6.2.6 常量函数

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121127315.png" alt="image-20231112112755251" style="zoom:50%;" />

> 可以看得出来，返回值是integer的函数，就是常量函数。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121129717.png" alt="image-20231112112914668" style="zoom:50%;" />

> **另外关于上面的那个例子的整型移位**
>
> 是的，你的理解是正确的。在计算机中，所有的数都是以二进制的形式存储的。当我们对一个数进行右移操作时，实际上就是将这个数的二进制表示向右移动一位。
>
> 例如，假设我们有一个十进制数8，它的二进制表示是1000。如果我们将它右移一位，就会变成100，这个二进制数对应的十进制数是4。所以，右移操作就相当于将原数除以2。
>
> 同样，左移操作就相当于将原数乘以2。因为在二进制表示中，向左移动一位就相当于将原数乘以2。
>
> 这就是为什么在这段代码中，`depth = depth >> 1 ;`这行代码可以用来计算`depth`的以2为底的对数。每次右移一位，`depth`就相当于被2除一次，同时`clog2`的值就加1，直到`depth`变为0为止。这样，`clog2`的值就是原来`depth`的以2为底的对数，向上取整。这是一个非常巧妙的计算对数的方法。
>
> ```verilog
> begin
> for ( clog2 = 0 ; depth > 0 ; clog2 = clog2 + 1 )
> depth = depth >> 1 ;
> end
> endfunction
> ```



## 6.3 关于使用任务和函数的总结

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121130340.png" alt="image-20231112113026271" style="zoom:50%;" />



## 6.4 常用的系统任务

### 6.4.1 $display 和 $write 任务

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121138953.png" alt="image-20231112113806881" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121142909.png" alt="image-20231112114208844" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121142847.png" alt="image-20231112114226789" style="zoom:50%;" />

> 像极了C语言的printf

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121720747.png" alt="image-20231112172003632" style="zoom:50%;" />

> 什么意思？
>
> 就是说，如果是表达式的结果是 12'b1000_1000_1000，以十六进制输出的时候，它会先思考该表达式的位的最大值为 12'b1111_1111_1111，而在这十六进制中，则是 FFF,十进制中则是4095，因此在十六进制会给它输出3位，十进制则是4位

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121733627.png" alt="image-20231112173351568" style="zoom:50%;" />

### 6.4.2 文件的输出

#### 1.打开文件

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121735082.png" alt="image-20231112173513007" style="zoom:50%;" />

> **多通道描述符**
>
> MCD：32bit，最高位保留不用，剩下的每bit代表一个打开的文件（置1），因此同时最多只能打开31个文件。LSB被“标准输出”文件占用。
>
> 　　　　这种方法的优势是可以将多个MCD用按位或的方式组合在一起，从而**同时向多个文件写入内容**。这种打开方式相当于是FD的"w"方式，不支持读取文件内容。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121737242.png" alt="image-20231112173711190" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121737366.png" alt="image-20231112173725309" style="zoom:50%;" />

#### 2. 写入文件

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121740867.png" alt="image-20231112174008793" style="zoom:50%;" />

> 这里已经很明显了，所谓的“文件描述符”，其实就是文件的地址，或者说是，文件的唯一标识，是用来识别，我是要对“这个文件进行操作的”

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121743900.png" alt="image-20231112174343836" style="zoom:50%;" />

> 这里面的运用非常的高深。1恰好就是0001，也就是所谓的标准输出，根据按位取或之后，它肯定要化成二进制，最后的形式就是二进制了

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121746395.png" alt="image-20231112174653339" style="zoom:50%;" />

> file = $fopen ("filename"); 最多打开30个文件，但是这样打开的句柄可以做或运算，实现多文件同时输出：

#### 3.关闭文件

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121748243.png" alt="image-20231112174856186" style="zoom:50%;" />

> 可以关闭指定通道符的文件



### 6.4.3 显示层次

> 类似于C#的反射？

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121751251.png" alt="image-20231112175142185" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121756264.png" alt="image-20231112175657203" style="zoom: 67%;" />

### 6.4.4 选通显示

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121757176.png" alt="image-20231112175743117" style="zoom: 67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121759285.png" alt="image-20231112175920211" style="zoom:50%;" />

> 就是说，如果你这里使用display，它可能中途没等赋值完就直接显示了
>
> 但如果是strobe，那就是等到这些赋值完成之后再显示



----

# 第七章 调试用系统任务和常用预处理语句

## 7.1 系统任务$monitor

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121829330.png" alt="image-20231112182949265" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121830484.png" alt="image-20231112183012410" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121832547.png" alt="image-20231112183203495" style="zoom:50%;" />

1. 输出参数列表的用法 与 $display 的相同
2. 只有当参数列表的值发生变化时才会打印；多个变化只打印一次
3. 参数可以是$time

> 当我们在Verilog代码中使用`$ monitor;`命令时，它会**自动监视并打印当前模块中所有变量的值，每当这些变量的值发生变化时。**
>
> 所以，`$ monitor;`命令可以帮助我们跟踪和理解代码的行为，特别是在调试复杂的电子设计时。

但是，任何时刻下，只能有一个$monitor起作用，就需要有人关闭和开启。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121834569.png" alt="image-20231112183416482" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311121834854.png" alt="image-20231112183425792" style="zoom:50%;" />

> 这个仅当了解就好，因为它这里都只是简单说一下，都没给出例子。



## 7.2 时间度量系统函数 $time 与 $realtime

#### 1.系统函数$time

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122002142.png" alt="image-20231112200227080" style="zoom:50%;" />

>  
>
> 是的，timescale 指令会影响其后面所有模块中的时延值，直至遇到另一个 timescale 指令或 resetall 指令。
>
> 如果一个设计中的多个模块带有自身的 timescale 编译指令，模拟器将定位在所有模块的最小时延精度上，并且所有时延都相应地换算为最小时延精度。
>
> 在 Verilog 中，如果一个模块没有指定 timescale，那么它可能会错误地继承了前面编译模块的 timescale 参数。因此，建议在每个模块的前面指定 timescale，并在最后加一个 resetall 来确保 timescale 的局部有效。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122006715.png" alt="image-20231112200638660" style="zoom:60%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122007921.png" alt="image-20231112200714855" style="zoom:50%;" />

在 Verilog 代码中，``timescale` 是一个指令，用来设置模拟的时间单位和时间精度。在您的代码中，`timescale 10 ns/1 ns` 就是这个设置。

- `10 ns` 是时间单位，可以理解为一种“尺度”或者“标尺”，用来衡量时间的长短。就像我们平时用“小时”、“分钟”来衡量时间一样，这里用的是“纳秒”（ns）。
- `1 ns` 是时间精度，可以理解为时间的最小刻度。就像我们的手表可以精确到秒一样，这里的时间可以精确到1纳秒。

> [`timescale` 不是宏定义，而是 Verilog HDL 中的一种预编译指令](https://blog.csdn.net/qq_43546203/article/details/124136837)[（扩展资料一）](https://blog.csdn.net/qq_43546203/article/details/124136837)[。预编译指令在 Verilog 编译系统的一个组成部分中，称为“编译预处理”](https://blog.csdn.net/qq_43546203/article/details/124136837)[1](https://blog.csdn.net/qq_43546203/article/details/124136837)[。这些预处理命令以符号“`”开头，这些预处理命令的有效作用范围为定义命令之后到本文件结束或到其它命令定义替代该命令之处](https://blog.csdn.net/qq_43546203/article/details/124136837)[（扩展资料一）](https://blog.csdn.net/qq_43546203/article/details/124136837)。
>
> [`timescale` 用来定义模块的仿真时的时间单位和时间精度](https://blog.csdn.net/qq_43546203/article/details/124136837)[（扩展资料一）](https://blog.csdn.net/qq_43546203/article/details/124136837)[（扩展资料二）](https://blog.csdn.net/super_haifeng/article/details/50110371)。格式如下：
>
> ```verilog
> `timescale 仿真时间单位/时间精度
> ```
>
> [这里的数字只能是1、10、100，不能为其它的数字。而且，时间精度不能比时间单位还要大](https://blog.csdn.net/super_haifeng/article/details/50110371)[（扩展资料二）](https://blog.csdn.net/super_haifeng/article/details/50110371)。
>
> [例如，`timescale 1ns/1ps` 和 `timescale 100ns/100ns` 都是正确的定义，但 `timescale 1ps/1ns` 就是错误的定义](https://blog.csdn.net/super_haifeng/article/details/50110371)[（扩展资料二）](https://blog.csdn.net/super_haifeng/article/details/50110371)。
>
> [在编译过程中，`timescale` 指令影响这一编译器指令后面所有模块中的时延值，直至遇到另一个 `timescale` 指令](https://blog.csdn.net/super_haifeng/article/details/50110371)[（扩展资料二）](https://blog.csdn.net/super_haifeng/article/details/50110371)。

另外里面还有一句话很重要，**$time输出的是时间尺度的倍数**



#### 2.$realtime 系统函数

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122015944.png" alt="image-20231112201519895" style="zoom: 67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122015410.png" alt="image-20231112201541345" style="zoom: 67%;" />

## 7.3. 系统任务 $finish

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122023180.png" alt="image-20231112202345113" style="zoom:67%;" />

## 7.4 系统任务$stop

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122036049.png" alt="image-20231112203651987" style="zoom: 50%;" />

## 7.5 系统任务 $readmemb 和 $readmemh

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122109579.png" alt="image-20231112210914480" style="zoom:50%;" />

> 也就是说，它们末尾的bh之分，只是数字是二进制还是十进制的区别

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122110794.png" alt="image-20231112211033720" style="zoom:50%;" />

> 看了下面的例子就能明白了

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122110123.png" alt="image-20231112211041063" style="zoom:50%;" />

> 代码的结果如下

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122112784.png" alt="image-20231112211222723" style="zoom:50%;" />

**而关于两个任务的不同参数的解释如下**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122113140.png" alt="image-20231112211311061" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122113422.png" alt="image-20231112211347361" style="zoom:50%;" />

## 7.6 系统任务$random

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122118568.png" alt="image-20231112211852488" style="zoom:50%;" />

> 当我们在 {} 中放入一个有符号的数时，Verilog 会将其视为无符号的数。**这是因为 {} 运算符在处理有符号数时，会忽略其符号位，只看其数值位。**因此，**{$random} 会将 $random 生成的有符号随机数转换为无符号数。**

**随机脉冲序列的意思**

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122121803.png" alt="image-20231112212136747" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122121807.png" alt="image-20231112212147755" style="zoom:50%;" />

## 7.7 编译预处理

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122124483.png" alt="image-20231112212445416" style="zoom:50%;" />

> 所谓预处理，其实就是预热打铁的意思，就像炒菜之前热热锅，先热锅，再下油

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122126569.png" alt="image-20231112212600496" style="zoom:50%;" />

> 注意的命令的的 **有效范围**

### 7.7.1 宏定义`define

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122128369.png" alt="image-20231112212816297" style="zoom: 50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311122136161.png" alt="image-20231112213657087" style="zoom: 67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311130930701.png" alt="image-20231113093038593" style="zoom: 67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311130932313.png" alt="image-20231113093205242" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311130934432.png" alt="image-20231113093438369" style="zoom:50%;" />

### 7.7.2 ”文件包含“ 处理 `include

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131154467.png" alt="image-20231113115150838" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131152142.png" alt="image-20231113115208080" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131154332.png" alt="image-20231113115427268" style="zoom:50%;" />

> 原来这就是文件的复制粘贴呀

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131155963.png" alt="image-20231113115529881" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131156035.png" alt="image-20231113115617979" style="zoom: 67%;" />

> 反正都是复制粘贴，终究会把引用拉到来的



### 7.7.3 时间尺度`timescale

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131929025.png" alt="image-20231113192955951" style="zoom:50%;" />

> 在Verilog中，timescale指令用于定义模拟仿真时的时间刻度。timescale的格式为timescale time_unit/precision，**其中time_unit表示时间单位，precision表示时间精度。**
>
> 例如，timescale 1ns/10ps表示时间单位为1纳秒，时间精度为10皮秒。这意味着，如果你在代码中写#100，它实际上表示100纳秒的延迟。然而，由于时间精度设置为10皮秒，所以你可以精确到10皮秒的延迟。例如，#100.1111会被认为是#100.111ns，因为它的精度高于timescale的时间精度，而被四舍五入。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131931062.png" alt="image-20231113193127999" style="zoom:50%;" />

> 根据上面的例子很容易知道，所谓时间单位，就是一个 #1 代表的延迟时间（举例）
>
> 比如 `timescale 10ns/1ns
>
> 则 #1.6 代表 16ns，#1.66是17ns（取整以四舍五入）

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131933499.png" alt="image-20231113193350441" style="zoom:50%;" />

![image-20231113193525829](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131935885.png)

> 我是没想到还有%t这种格式



### 7.7.4 条件编译命令 

一般情况下,Verilog HDL源程序中所有的行都将参加编译。但是有时希望对其中的一部分内容**只有在满足条件时才进行编译**，也就是对一部分内容**指定编译的条件**，这就是“条件编译”。有时，希望当满足条件时，对一组语句进行编译,而当条件不满足时则编译另一部分。

> 怎么还能这么玩

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131956364.png" alt="image-20231113195656281" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131957806.png" alt="image-20231113195725732" style="zoom:50%;" />

> 在Verilog中，ifdef和ifndef都是条件编译指令，它们可以根据指定的条件来生成对应的电路，这可以减少电路面积并提高代码的复用性。
>
> ●   `ifdef`：全称 "if defined"，当文件编译到这一行时，如果这个文件已经被编译过（不是首次编译），就会执行后续的命令。例如，如果使用 `define定义了称为`FLAG`的宏，那么关键字`ifdef会告诉编译器包含这段代码，直到下一个`else或`endif。
>
> ●   `ifndef`：全称 "if not defined"，顾名思义，即当文件编译到这一行时，如果这个文件没有被编译过（首次编译），就会执行后续的命令。关键字`ifndef只是告诉编译器，如果给定的名为FLAG的宏没有使用 `define指令定义，则将这段代码包含在下一个`else或`endif之前。
>
> 这两个指令通常和预编译指令`define配套使用。它们可以出现在设计中的任何地方，并且可以相互嵌套。这对于程序的移植和调试是很有用的。

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131958839.png" alt="image-20231113195824787" style="zoom:50%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131958930.png" alt="image-20231113195849874" style="zoom: 67%;" />

<img src="https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311131959084.png" alt="image-20231113195919012" style="zoom:50%;" />

