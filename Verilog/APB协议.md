主要区别如下：

- APB2：APB总线的基础版本。

- APB3：

- - 增加PREADY信号：用于反压master（其在读和写两个场景中含义略有不同，过会讲解）。
  - 增加PSLVERR：用于代表传输是否发生错误。

- APB4：

- - 增加PPROT保护信号。
  - 增加PSTRB代表字节选通。
  - 

# 文章来源

https://zhuanlan.zhihu.com/p/623829190?utm_id=0



# APB2

### 简介

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240416143237792.png" alt="image-20240416143237792" style="zoom:67%;" />

### 写操作

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240416143459673.png" alt="image-20240416143459673" style="zoom:50%;" />

> PSEL 拉高第一个时钟周期，PENABLE应该为0。
>
> 至此一次传输结束。可以看到，APB对每一笔数据的传输，**需要花费两个时钟周期**。且APB的数据传输**不支持流水线操作（即不可以重叠）。因此APB是非常低效的。**
>
> **由于APB向下兼容，这一历史遗留问题一直延续至今。不过好在APB应用的场景本身就是配置寄存器等操作，因此多花一个时钟周期也没什么大不了的。**

### 读操作

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240416143721292.png" alt="image-20240416143721292" style="zoom:50%;" />

> 读操作也是需要两个周期，需要等PENABLE才能将读地址传输过去

### APB的状态机

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240416143904036.png" alt="image-20240416143904036" style="zoom: 67%;" />

> 从setup到enable不需要条件的进行跳转

### APB2的几个小问题

#### Q:是否需要penable？

A：有PCLK的情况是不需要的，**用两个状态的状态机就能解决**

```verilog
`timescale 1ns/1ps
`define DATAWIDTH 32
`define ADDRWIDTH 8
`define IDLE     2'b00
`define W_ENABLE  2'b01
`define R_ENABLE  2'b10
module APB_Slave
(
  input                         PCLK,
  input                         PRESETn,
  input        [`ADDRWIDTH-1:0] PADDR,
  input                         PWRITE,
  input                         PSEL,
  input        [`DATAWIDTH-1:0] PWDATA,
  output reg   [`DATAWIDTH-1:0] PRDATA,
);
reg [`DATAWIDTH-1:0] RAM [0:2**`ADDRWIDTH -1];
reg [1:0] State;
always @(negedge PRESETn or posedge PCLK) begin
  if (PRESETn == 0) begin
    State <= `IDLE;
    PRDATA <= 0;
    end
  else begin
    case (State)
      `IDLE : begin
        PRDATA <= 0;
        if (PSEL) begin
          if (PWRITE) begin
            State <= `W_ENABLE;
          end
          else begin
            State <= `R_ENABLE;
          end
        end
      end

      `W_ENABLE : begin
        if (PSEL && PWRITE) begin
          RAM[PADDR]  <= PWDATA;      
        end
          State <= `IDLE;
      end

      `R_ENABLE : begin
        if (PSEL && !PWRITE) begin
          PRDATA <= RAM[PADDR];
        end
        State <= `IDLE;
      end
      default: begin
        State <= `IDLE;
      end
    endcase
  end
end 
endmodule
//摘自原文章
```

> 如果APB slave是纯粹的组合逻辑，也就是没有PCLK的情况下。这个时候是需要penable信号的。



# APB3

### 简介

APB3在APB2的基础上增加了两个信号，**PREADY和PSLVERR**(slave error)，这两个信号都是由slave产生的。

对于写操作而言，PREADY信号用于标志slave设备是否已经准备好接收这一笔数据。而对于读操作而言，PREADY信号用于标志slave设备是否已经准备好了要返回给Master的数据。

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240416145614691.png" alt="image-20240416145614691" style="zoom:75%;" />

> “有了PREADY信号，从机就可以**反压**主机。因此PREADY这个信号可以说是非常的棒啊，它让主从之间的通信更加的可靠，也增加了从机的控制能力，**不至于出现主机写的数据从机压根没收到或者从机没有准备好读数据，进而主机读到错误的数据的情况。**
>
> 此外我再讲解一下为什么会出现从机没有准备好的情况。对于**读操作**而言，非常好理解，你要读的数据我还没准备好呢（可能正在运算），那我当然不能拉高PREADY，不然就给了你错误的数据了。**只有我数据准备好的时候，我拉高PREADY，这样才能确保读不出问题。**而写的话，这种情况往往出现在写**特定的地址**，这个时候外设本身要进行**判断是否可以写**，因此两个周期就完成不了，就暂时不能拉高PREADY，你的Master就必须要维持住。**当PREADY拉高代表这一拍能够写进去了。因此主机也就可以不再维持原有的状态了。**”

**PSLVERR，（**apb slave error**）**顾名思义。用于从slave向master返回传输错误，这个错误是slave自己定义的，比如写了不允许写的地址，即非法地址访问。或者是访问超时了，slave回应不了了。就可以拉高这个信号，从而避免总线锁死。

### 写操作

不需要等待的状态，即PREADY已经能很快准备好的状态：

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240416150811817.png" alt="image-20240416150811817" style="zoom: 67%;" />

需要等待的状态，即PREADY不能与PENABLE同一时间出现

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240416151157441.png" alt="image-20240416151157441" style="zoom:67%;" />

### 读操作

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240416151236544.png" alt="image-20240416151236544" style="zoom: 67%;" />

### Error

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240416152602831.png" alt="image-20240416152602831" style="zoom: 67%;" />

> 有点像那种感觉：就像写状态，先送地址，地址送到之后进行判断是否合法；不合法就标志传输失败，即DATA不允许被写入
>
> **也就是地址会先送过去判断**

# APB4

### 简介

APB4在APB3的基础上又增加了**PPROT（保护）和PSTRB（写选通）信号**。

>  **PPROT signals enable APB4 slaves to protect against illegal transf**<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240416153451888.png" alt="image-20240416153451888" style="zoom:67%;" /> 

- 对于CPU而言，可以工作在**用户模式**下也可以工作在**特权模式**下（比如RISC-V的USM三种模式）。
- 对于支持trustzone的CPU，可以工作在secure world下，也可以工作在normal mode下。
- 又由于现在的系统越来越复杂了，以前的外设是随意读写都可以，现在的一些外设或memory（比如一些CSR寄存器，Trusted RAM等）要求只能在secure下或者privileged模式下访问，因此就需要PPROT信号。

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240416160329696.png" alt="image-20240416160329696" style="zoom:67%;" />

### 兼容性问题

**从APB4到APB3**

把PSTRB信号固定为1，PPROT信号跟据APB3具体的使用场景固定成不同的值

至于APB3和APB2不建议一起用，因为APB2没有PREADY反压机制，因此实际使用起来完全不一样，强行一起用会有巨大的坑。



# RTL实现

### 自设计例子

仿照：https://github.com/ForrestBlue/cortexm0ds/tree/master/logical/cmsdk_apb4_eg_slave/verilog

首先是整体本身，它应该具有这些信号：

```verilog
module apb_slave_1(
	input 				PCLK,
    input 				PRESETn,
   
    input 				PSEL,
   	input 				PENABLE,
    input	[2:0]		PPROT,
    input 	[3:0]		PSTRB,
    
    input 				PWRITE,
    input 	[31:0]		PADDR,
    input	[31:0]		PWDATA,
    output 	[31:0]		PRDATA,
    
    output 				PREADY,
    output 				PSLVERR
);
endmodule
```



然后是要配置的寄存器那部分

```verilog
module apb_slave_reg_example(
	input 				clk,
    input 				rst_n,
    
    input	[31:0]		wdata,
    output 	[31:0]		rdata,
    input 				read_en,
    input 				write_en,
    input 	[31:0]		addr,
    
    input 	[3:0]		wr_strb,
    output 				ready,
    output 				error_resp
);
endmodule
```



最后是接口部分

```verilog
module apb_slave_if(
    //apb
	input 				pclk,
    input 				prst_n,
    
    input 				psel,
    input 				penable,
    output 				pready,
    output 				pslverr,
    
    input 	[31:0]		pwdata,
    output 	[31:0]		prdata,
    input 	[31:0]		paddr,
    input 				pwrite,
    
    input	[3:0]		pstrb,
    input 	[2:0]		pprot,
    
    //reg
    output 	[31:0]		wdata,
    output 	[31:0]		waddr,
    input 	[31:0]		rdata,
    output				read_en,
    output 				write_en,
    
    input 				ready,
    input 				error_resp
    
);
    
endmodule
```

