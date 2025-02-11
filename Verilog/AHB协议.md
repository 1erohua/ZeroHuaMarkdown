# AHB-lite

![img](https://pic1.zhimg.com/v2-8b9a6d54054df5948040d4c6fca3c3ac_b.jpg)

本文件将着重于AHB-lite协议

参考文章：作者：lawliet
链接：https://zhuanlan.zhihu.com/p/627821524
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

## 信号	

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240422110225413.png" alt="image-20240422110225413" style="zoom: 67%;" />

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240422110248502.png" alt="image-20240422110248502" style="zoom: 67%;" />

**Decoder的信号**

![img](https://pic1.zhimg.com/v2-4ede9d2876da1faf95fe39810926557c_b.jpg)

> Decoder实际上就产生一组信号，直接看文档。Decoder顾名思义，进行译码。对什么进行译码？实际上就是地址，当满足条件的时候，将多组HSELx的其中一个拉高，已选中需要选中的Slave。
>
> 此外这个信号不光要给Slave，通常还要给MUX，以方便Mux从多个Slave中选中一个，返回给Master。这个信号可能是HSELX本身，也可以是HSELX导出的信号。
>
> 作者：lawliet
> 链接：https://zhuanlan.zhihu.com/p/627821524
> 来源：知乎
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

**MUX信号**

![img](https://pic4.zhimg.com/v2-502a6632b757e4035465f60ad9e95777_b.jpg)

> MUX实际上就是从多个Slave的输入选择一个返回给Master即可。因此就有HRDATA和HRESEP。而这个HREADY和之前说的HREADYOUT不一样！这里的HREADY用于告诉Slave和Master，整个总线上是否有未完成的传输。而之前的HREADYOUT，用来代表正在传输的那个slave是否已经准备进行真正的数据传输，例如写操作时，slave此时是否可以将数据存下来，如果没拉高，说明那个slave正在反压Master。
>
> 我理解的反压就是从机通过ready来控制主机数据的阻通

## 读写

读数据

![img](https://pic1.zhimg.com/v2-5a5df68f83007ed79cbc82608c6e837c_b.jpg)

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240422111306578.png" alt="image-20240422111306578" style="zoom:50%;" />

> “主机不能改变它的信号，是指数据和下个周期的地址信号”，从整个的意义来说，也确实是不能改变它的信号
>
> **在ready没好之前，下个要传输的地址需要阻塞等待；并且等待的是当次数据的读出或写入。接下来看写数据（这里读数据是ready之后再读出的）**

<img src="https://pic1.zhimg.com/v2-d75dbfa78f0f01274d20d22f38237c58_b.jpg" alt="img" style="zoom: 80%;" />

> 写的话其实更简单，重点就是从机在反压主机，主机的信号不能变。(**Data（A）不能变**)

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240422113038970.png" alt="image-20240422113038970" style="zoom:67%;" />

> 当ready没有好的时候，**当前周期要读写的数据要保持，（流水线中）下个周期的地址与控制信号也要保持**



## 传输

此处省事，直接复制

作者：lawliet
链接：https://zhuanlan.zhihu.com/p/630647524
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



**HTRANS[1:0]**信号用于指示当前的**传输类型**，一共有**四种类型**：

- **IDLE**

- - 没有数据传输,其它的控制信号和地址信号因此也就不起作用。

- **BUSY**

- - 没有数据需要传输。
  - 这个信号可以在突发传输中（**什么是突发传输后面讲，因为官方文档也是这个顺序，如果这一部分看不懂的，先硬着头皮看完，然后等看完后面的突发传输再回过头来再看一遍**），用来插入空闲的CYCLE，表示主机在忙，也就是说传输还在继续，但是处于**暂停状态**。
  - **在不指定突发长度的情况下，在最后一拍用BUSY去传输，来表明这是Burst的最后一笔，这一鸡肋的机制被AXI的LAST信号完美替代。**
  - **实际上BUSY这个状态很少见**，在CORTEX-M系列基本上不会有这个BUSY状态，在大部分情况下都是使用NONSEQ和SEQ传输类型，因此可以暂时不去掌握该类型

- **NONSEQ**

- - 需要传输数据。
  - 可能是发一笔数据（Single传输），也可能是Burst传输的第一笔transfer（在这里transfer指的是突发传输的一次读或者一次写，或者就是Single传输）。
  - 地址和其它控制信息和之前的传输没有关系！只取决于这次transfer本身。

- **SEQ**

- - 需要传输数据。
  - 用在突发传输中，代表连续的传输。
  - 地址需要增加（除非上一笔transfer是BUSY transfer）。
  - 其它的控制信号保持不变。

![img](https://pic1.zhimg.com/v2-0b9ab72cd500fdf9036a514b32178ded_720w.jpg?source=d16d100b)

> 如果上一次传输是busy，那么seq传输将无需递增地址，而是完成上次传输的地址

由上图可以知道，这是一次突发读操作，实际上是4拍有效读：

- T0->T1：突发传输的**第一笔Transfer**，因此HTRANS应该为NONSEQ，HADDR为0x20代表起始地址.
- T1->T2：突发传输的第二笔Transfer（伪），因为HTRANS为BUSY，可能是因为主机忙碌没有空去读，所以插入了这样一拍，HADDR增加0x4，HRDATA返回了0x20地址的数据，这一拍没有发生数据传输。
- T2->T3：突发传输的第二笔Transfer（真），由于上一拍是BUSY，所以HADDR的地址不需要增加。HTRANS应该是SEQ。
- T3->T4：和上一拍差不多，第三笔Transfer。
- T4->T5：最后一拍读，由于Slave不能完成数据传输，因此将HREADY拉低，这一拍相当于wait state。
- T5->T6：最后一拍读，HREADY拉高，说明这拍主机可以将控制信号等信息发给从机。
- T6->T7：从机返回最后一拍读的数据。

​	

## 原子操作

印象里就是由HLOCK控制的，但从AXI4协议中，我们得知，这玩意用得少，还是exclusive 访问用得多而且还好

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240422113658519.png" alt="image-20240422113658519" style="zoom:67%;" />

> 还有一点需要注意，大部分Slave实际上不需要实现**HMASTERLOCK**信号，因为大部分Slave不会被多个Master所访问。只有能够被多个Master访问的Slave，才需要**HMASTERLOCK**信号，如多端口Memory Controller，就必须要实现**HMASTERLOCK**信号的机制。
>
> 作者：lawliet
> 链接：https://zhuanlan.zhihu.com/p/630647524
> 来源：知乎
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



## Burst 突发传输

突发(burst)将**多个传输（Transfer）**作为一个单元（有时候可以叫Transaction）进行执行，而不是独立地处理每次Transfer，核心思想是将多次Transfer看做一个整体。

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240422114240627.png" alt="image-20240422114240627" style="zoom:67%;" />

**INCR的例子**

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240422114354713.png" alt="image-20240422114354713" style="zoom: 50%;" />

<img src="https://picx.zhimg.com/v2-e464483cb0d9712c3cf86e0b5aea9b30_720w.jpg?source=d16d100b" alt="img" style="zoom: 75%;" />

**WRAP的例子**

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240422114705540.png" alt="image-20240422114705540" style="zoom:50%;" />

![img](https://picx.zhimg.com/v2-cb2635b4e28061be7bf076de71a92a07_720w.jpg?source=d16d100b)

最后我们回顾一下开始的思考题：**突发读或者突发写，和流水线形式的SINGLE TRANSFER是一样快的嘛？为什么？**

表面上来看，没什么区别。理论上都是N+1个Cycle。但是这是针对一拍可以回数的紧耦合SRAM而言的，如果访问的是DDR呢？

- 如果是突发传输，你只需要下发一次命令，DDR Controller可以帮你计算好，你总共需要读多少数据，比如每一拍是32Bit，突发长度是4，**DDR控制器只要对DDR发一次命令即可，一次性读回128Bit的数据**
- 如果是Single transfer，DDR Controller可不知道你下一笔传输的地址和这笔传输的地址只差了0x4，DDR Controller完全可能把你的第一次single transfer下发出去，然后又下发第二次，然后又下发第三次，**然后...由于DDR的读取时序没有那么简单，不是完全的流水式的，因此这中间就可以阻塞很多个周期。**
- 所以！突发传输和流水线的Single transfer是不一样的！**能用突发传输就用突发传输**，不要用多次Single Transfer。

作者：lawliet
链接：https://zhuanlan.zhihu.com/p/630647524
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



# AHB-lite Slave的响应和其他控制信号

## Slave的响应

在Master发起传输后，剩余的传输过程，**其实是由slave接手控制的，因而master不能中途取消本次传输**

AHB-lite 实际是通过HRESP与HREADY两个信号来控制传输响应的

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240422145641988.png" alt="image-20240422145641988" style="zoom:67%;" />

HRESP和HREADY这两个信号的组合来控制传输响应，因此理论上一共有四种组合，接下来分别介绍这几种情况：

- 当HRESP为0，而HREADY为1，代表本次传输成功。

- 当HRESP为0，而HREADY为0。则代表Slave正在反压主机，可能是从机此时不能够及时的响应，主机此时应该维持住控制信号。当HREADY拉高，并且HRESP仍然为0，则代表本次传输成功结束。（需要注意的是，对于正常的传输，**也就是不涉及错误情况的时候，HRESP应该始终为0**, 属于是1的未完成形态）

- - Note：通常而言，每个Slave应该提前规定好最大的Wait states周期数。此外建议Slave不要插入多于16个周期的等待时间，否则的话每次访问可能要花费大量的时间去等待。

- **当HRESP为1，则代表传输出现错误了。**大概率是写了不让写的地方导致的。对于之前的正常传输而言，只需要维持住一个周期的响应，但是ERROR response需要两个时钟周期。（**这是由于总线的流水线特性决定的，当slave发出ERROR响应时，下一个传输的地址已经被广播到总线上了。two-cycle响应给master提供了足够的时间取消下一次访问并将HTRANS [1：0]驱动到IDLE。**）

直接看产生错误回应的时序图

![ ](https://picx.zhimg.com/v2-0949a051de4e97f2ac58ec939f4b3771_720w.jpg?source=d16d100b)

> T1到T2, HREADY为0且HRESP为OKAY，说明正处于处理状态。（Slave发现突发写出现错误，需要返回EEROR，因而需要插入一个wait states准备后续的操作）（**应该是在T1的时候接收数据处理之后发现，突发写地址出现问题，需要wait states处理**）
>
> T2到T3, Slave返回的第一个ERROR，这是EEROR的第一个时钟周期，**这时候READY是0**
>
> T3到T4，Slave返回的第一个ERROR，这是ERROR的第二个时钟周期，**这时候READY是1，同时状态进入IDLE**
>
> > Master因此就有足够多的时间去决定下笔是**继续发这个transfer还是终止这次transfer**，这也就是为什么需要两个时钟周期去维持这个ERROR response，如果没有两个时钟周期的ERROR response，那么T2->T3的HREADY就应该为高，HTRANS应该迅速变成IDLE，来完成这次传输，**如果逻辑复杂的话，组合逻辑链路会过长，为了保证流水线特性，让Master有足够多的时间去感知本次传输出错了，则需要维持两拍的ERROR response**，此外这也是保证和其他的正常传输一致。回顾一下没有错误传输情况的突发传输，二者都具有流水线特性）。
>
> T4到T5，返回OKEY，表示本次突发传输结束

也可以在出错之后继续写入

<img src="https://pic1.zhimg.com/v2-1dc14e9f657d8062d245c84b00dad346_720w.jpg?source=d16d100b" alt="img"  />

> 容易看出，**地址判断可能是在接受了A之后就进行了判断，但是由于发现了问题，需要插入一个wait states（即返回的第二个OKAY），因而在获得地址后，需要维持一个周期的OKAY，再返回两个周期的ERROR**



# 保护控制信号协议 PROT

保护控制信号HPROT[3:0]，每一位代表不同的含义。该信号用于提供一些保护信息。

- HPORT[0]，代表是**取数据**还是**取指令**。

- HPORT[1]，代表是**特权**还是**非特权**模式访问。比如操作系统的地址区域，就应该是特权访问，并不是所有的写都有权限去写这段地址空间，不然就出大事了。

- HPROT[2]，代表**Bufferable**。这个Bufferable指的是**写的数据是可以放在Buffer里面**还是**直达最后的目标地址空间**。（实际上最终还是会写回最终目标地址空间的，只不过返回HRESP的时候，大概率是还没有写到最终的目标地址空间）**对于写具有严格的先后顺序之分的地址空间，一般是Non-cacheable和Non-bufferable，否则可能会乱序。**

- HPORT[3]，**代表Cacheable，表示是否可以存放在Cache中，是否更新最终的memory**。对于外设寄存器这种，当然是不可以Cacheable的。**我们本意就是希望写目标寄存器，达到某些功能。写到Cache里面，那就没法达到预期了。**（放在cache里面，就不能保证数据能够及时更新）

  <img src="/home/zerohua/.config/Typora/typora-user-images/image-20240422152454254.png" alt="image-20240422152454254" style="zoom:50%;" />
  
  <img src="/home/zerohua/.config/Typora/typora-user-images/image-20240425113301194.png" alt="image-20240425113301194" style="zoom:50%;" />



# AHB怎么设计、AHB2APB同步桥设计

## 设计要点

### 1.HREADYOUT 与 HREADY

> 其中**HREADYOUT信号是由Slave发出，送给MUX选择器**，用来表示Slave是否已经准备进行真正的数据传输（data phase阶段），例如写操作时，Slave是否可以将数据存下来。本质上是Slave对Master的反压信号。
>
> 而**HREADYIN，由MUX输出返回到所有的Slave，通知各个Slave，是否还有别的SLAVE有未完成的传输**。每个Slave在采样地址和控制信号的时候，都要看这个HREADYIN信号是否为1，如果为0的话则代表别的Slave还有未完成的传输，因此不能采样地址和控制信号。

首先我们思考一个问题，HREADYOUT默认的复位值应该是多少？

- 假设都为0，那么HREADY信号自然也为0。**这样返回给所有的Slave的HREADYIN信号就为0，则Slave会认为还存在别的transaction还没有结束。则会一直等待，那么整个系统就会不工作了**！这也是很多人设计AHB-Slave的时候会犯的错误。

复位状态要设置为**就绪状态**，避免死锁

当时看APB的MUX设置没明白为什么ready的默认值是1。如果默认值不是1,就会导致，各Slave接收到信息都是未完成的状态，那样各个Slave都会按兵不动

作者还举了具体有关于如果没有HREADYIN信号会发生什么的例子。其实解释起来很简单，比如两个slave，一个处于占用总线的状态，即还没完成传输（HREADYOUT为0），另一个已经是准备就绪（HREADYOUT为1）。无论Master看哪个行事，如果没有HREADYIN对各个Master进行反压，都会对Master的行为出现混乱

### 2.地址映射（Memory Mapping）

**1KB对齐、default Slave**

<img src="https://pic2.zhimg.com/v2-c59836b4259dbc85d0cd2ed40d4b3541_b.jpg" alt="img" style="zoom: 67%;" />

<img src="https://pic1.zhimg.com/v2-3b3445676f0730893f5529efc8f503f4_b.jpg" alt="img" style="zoom:50%;" />

Default Slave是用来干嘛的？我们做Memory Map的时候，自然不可能所有的地址都覆盖掉。**我们需要考虑有一个Default Slave，这样访问缺省地址的时候，会有一个符合你需求的Response。**如果没有相关逻辑的话，就会有问题，总线会卡住。

## 同步桥设计

![img](https://pic2.zhimg.com/v2-6f2d7aa5458897fa8d576f53a75e8e8d_b.jpg)

- 两边的时钟频率通常不一样，所以我们才需要两个子系统的桥接
- 有时候还会涉及到跨时钟域的问题（异步fifo、打拍传输（单信号）、握手信号）

本篇文章的转接桥是AHB高性能总线到APB总线桥接器，主要完成以下的功能：

- Bridge是APB总线中**唯一主机**；
- APB的时钟和AHB的时钟是**同步时钟**；APB的时钟和AHB时钟的**分频关系由PCLKEN信号决定**；
- 该模块支持**输入输出数据寄存或者不寄存，由模块参数控制**；
- 该同步桥**支持APB总线字节选通信号，保护控制信号**；
- 该同步桥支持**APB模块的使能信号**；
- **该同步桥只支持一个APB从设备，所以只有一个PSEL**；

> 1. 锁存AHB地址以及控制信号，并使之在整个APB传输期间有效（因为要完成跨时钟域的转换能）
> 2. 译码地址，并产生一个外设选择信号PSELx
> 3. 写传输，驱动数据到APB上；读传输，驱动数据到系统总线
> 4. 为传输产生一个时序选通信号

![img](https://pic4.zhimg.com/v2-d48973dfdba60aaa12f92fd795723d87_b.jpg)

1. AHB支持流水，而APB不支持流水，而因而需要通过HREADYOUT反压上级，让其只能通过APB的方式传输数据	



*在单slave下，hreadyout的值就是hready(hreadyin)的值*

*多slave下，hready（hreadyin）为各个slave的hreadyout的与门*

cstate

| IDLE                       | WAIT                                                 | TRNF                                                         | TRNF2                       | ENDOK          | ERROR1    | ERROR2    |
| -------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ | --------------------------- | -------------- | --------- | --------- |
| apb_select拉高             | abp_select拉低                                       |                                                              |                             | apb_select拉高 |           |           |
| readyout = 1               | readyout = 0                                         | readyout = 0                                                 | readyout可0可1              | readyout = 1   | 0         | 1         |
| 地址与控制信息拉高并且有效 | 地址与控制信息存入寄存器                             |                                                              |                             |                |           |           |
| sample_wdata_set为1        | sample_wdata_reg被存入1，sample_wdata_set为0，clr为1 | sample_wdata_reg为0, set为0,clr为0,至此，clr清零完成，且写数据写入完成 |                             |                |           |           |
|                            |                                                      | rwdata_reg被存入HWDATA                                       |                             |                |           |           |
|                            | PADDR、PWRITE、PSTRB                                 | PWDATA                                                       |                             |                |           |           |
|                            |                                                      | PSEL                                                         | PENABLE                     |                |           |           |
|                            |                                                      |                                                              | apb_tran_end拉高            |                |           |           |
|                            |                                                      |                                                              | PSLVERR拉起，并跳转到ERROR1 |                | HRESP拉起 | HRESP拉起 |
|                            |                                                      |                                                              |                             |                |           |           |
|                            |                                                      |                                                              |                             |                |           |           |
|                            |                                                      |                                                              |                             |                |           |           |
|                            |                                                      |                                                              |                             |                |           |           |
|                            |                                                      |                                                              |                             |                |           |           |

同步桥信号：

```verilog
input	HCLK，HRESETn, PCLKEN
input 	HSEL, HTRASNS,HSIZE,H
```

