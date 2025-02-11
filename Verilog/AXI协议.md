x文章参考：https://zhuanlan.zhihu.com/p/641597910

作者：lawliet
链接：https://zhuanlan.zhihu.com/p/641978200
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

# AXI了解

## 1. AXI feature

### 独立的**读写通道**与分离的**数据地址通道**

在AHB的时候，不能同时读写; 它需要由HWRITE来控制是读操作还是写操作

![img](https://pic3.zhimg.com/v2-b3d7ebc12e03ddda1e98a28e46113656_b.jpg)

### 支持Outstanding

允许在接收**上一个事务的响应之前**，发出下一个事务的请求。

https://www.yisu.com/jc/524813.html

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240428223731545.png" alt="image-20240428223731545" style="zoom:67%;" />

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240428223934091.png" alt="image-20240428223934091" style="zoom: 67%;" />

## 2.握手机制

### 2.1 握手的要求

**VALID指示通道中的数据有效，READY表示准备接收通道的数据**

**VALID不得等待READY有效再有效，即生成VALID信号不包含READY；READY可以等待VALID，也可以先于VALID**

**写数据与写地址通道是同一方向的，而写响应通道与之相反，因而，写数据和写地址的VALID和READY，要先于写响应的VALID和READY**（先有写数据与写地址才能有写响应）

**读地址和读数据反向，因而读地址VALID和READY应该先于读数据VALID**（先有读地址才能有读数据与读响应）

![img](https://pic4.zhimg.com/80/v2-29456bf72c865f899120e5831be6df9f_720w.webp)

## 3. Transfer与Transaction

- A transaction is an entire **burst** of transfers, containing **an address transfer**, **one or more data transfers**, and, for write sequences, a response transfer.
- A transfer is a single exchange of information, with one VALID and READY handshake.

![img](https://pic2.zhimg.com/80/v2-443c135585151f894aa027e3a7a9c5c9_720w.webp)

## 4. AXI突发传输

**突发读**

AXI总线是基于**突发**传输的，并且**AXI的突发是只需要给一次地址信号即可**，这样就免去了地址计算的逻辑。

T2时刻，地址握手成功，传输地址信息；而在之后的每次RVALID与RREADY的握手成功， 就返回一个读数据，称之为一次transfer

**当返回最后一笔数据的时候，相应的RLAST需要拉高，从而代表整个transaction到此结束**

![img](https://pic2.zhimg.com/v2-a6e51551babb93b78c9d270b606488f9_b.jpg)

**支持Outstanding的情况**

即使第一个地址的数据没有返回，仍然可以继续给出下个地址

![img](https://pic2.zhimg.com/v2-19287a9f0b12581352755c6cfd2740b9_b.jpg)

**突发写操作**

给出地址，在AW的READY与VALID握手之后，再给出写数据。当第一个写数据握手成功，就给入第二个写数据，第二个写数据握手成功，就给出第三个数据。

当写入最后一个数据且WLAST拉高之后，认为写操作完成，于是**从下个周期**开始返回写响应并对写响应握手

![img](https://pic3.zhimg.com/v2-d0271bfdbc05fe218c3a7ab8c2388ed2_b.jpg)

### 突发传输信号

![img](https://pic2.zhimg.com/v2-91951c9c4cf3937335e18a8e6281e3c1_b.jpg)首先是突发长度，**AWLEN和ARLEN**两个信号指定了每一笔突发传输有多少笔，对于AXI3，支持1~16笔transfer，分别对应000~111，对于AXI4，支持1~256笔transfer，分别对应00000000~11111111。

值得注意的是，对于写而言，该信号很多时候没什么用，**因为WLAST才是真正的最后一笔写传输的标志**，有些设计AWLEN甚至就是默认值，主机那边控制好WLAST即可，这种方法也可以，**但不建议，WLAST按理说要和AWLEN对应好！**

然后是突发数据大小，**AWSIZE和ARSIZE**两个信号，该信号用来标志传输的数据位宽哪些bit是有效的，一般是从低位开始算，比如ARSIZE为'b001的时候，则代表transfer的size为2Bytes。对应ARDATA[15:0]是有效的。

**第一种类型为FIXED类型**，顾名思义，地址固定，这个时候可能是普通的传输，即Burst length为1。也有可能是重复访问相同的地址，比如加载或者清空FIFO的情况。

**第二种类型为INCR类型**，这种情况下地址是递增的，支持非对齐访问。增加的地址大小和AxSIZE相关。

**第三种类型为WRAP类型**，这种情况地址也是递增的，但是增加到某个地址以后会回过头去访问，也就是回环，前面我已经讲过了。这种情况实际上是用来写Cacheline的，用在critical word first的模式下，因此它实际上是必须要对齐访问的，因为cacheline典型情况是64或者32byte，但CPU最着急写或者读的是其中的某个word，这种情况下就需要用到WRAP类型，因此它的Length一般也严格限制在2、4、8、16。对应着cacheline不同的块。

作者：lawliet
链接：https://zhuanlan.zhihu.com/p/641978200
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

![img](https://pic3.zhimg.com/v2-a91382c8e1789f8de0f42682ca8e1662_b.jpg)

## 5. 其他的控制信号

### 5.1 保护功能支持

![img](https://pic2.zhimg.com/v2-7215cfce2550df711b8adec538cc2bf5_b.jpg)

**AXPROT[0]**用来区分是**普通访存**还是**特权访存**，如下图所示，比如操作系统这一块地址，普通的访存当然是不被允许的，一般是拥有较高的权限，才允许进行访存，体现在硬件上实际就是Normal access还是Privileged Access，这个时候就得借助该信号。

![img](https://pic2.zhimg.com/v2-be11e6e17b6448f80224cf4dc40c233d_b.jpg)

**AXPROT[1]**用来区分是**Secure**还是**Non-Secure**，如下图所示，**Secure Region**一般是内存中**特别机密**的信息，同理这也需要处理器的支持，只有Secure的程序才允许访问Secure Region。

![img](https://pic4.zhimg.com/v2-0c3e62810ddea27b4687adabd326972b_b.jpg)

**AXPROT[2]**用来区分是**Instruction**还是**Data**，这个用于区分CPU是取指令还是取数据，实际上用的很少，很多时候可能取得是指令，但是标明的是Data，ARM的官方手册也有这么一句话：This indication is provided as a hint and is not accurate in all cases. For example, where a transaction contains a mix of instruction and data items. It is recommended that, by default, an access is marked as a data access unless it is specifically known to be an instruction access。
因此对于该比特，大家选择性的使用即可，一般是用不到的。常用的就上面的[0]和[1]bit。

### 5.2 cache信号支持

![img](https://pic3.zhimg.com/v2-de400cd970f70fb4a313657ef32887ba_b.jpg)

**AXCACHE[0]**用来区分是**Bufferable**还是**Non-bufferable**，Bufferable即这笔Transaction是否可以**存在于Buffer**中，比如我要往某个地址写个数据，**我是否必须真正的写在了那个地址，然后收到了response才算传输结束，还是写到Buffer中，收到了Response就算结束。**（印象里是，**response是否来自目的地才算真的传输结束**）
对于写外设操作而言，一般是不允许Bufferable的，因为你没有写入外设的寄存器，外设没有按照你的预期产生工作状态的改变，你就认为传输结束了，是不符合预期的。只有写一些数据的时候，比如计算的中间结果，这个时候对你写在哪里不太敏感，是允许Bufferable的。**一般该信号只用在写上**，因为读的时候该Buffer可能已经被覆盖掉了，不是想要的值了，当然也不绝对，看你的设计和替换方式是怎样了。

**AXCACHE[1]**用来区分是**Cacheable**还是**Non-Cacheable**，在AXI4中更改为**Modifiable，**这个Modifiable实际上更加好理解**。**官方手册的说法是该比特用于决定实际的传输事务和你原本的传输事务必须是否相等，这么听起来可能有点抽象，以下是具体的实例。

以写为例子，比如你第一次是往地址0写，第二次是往地址1写，如果是Cacheable的，系统检查这两个地址的内存属性又是一样的，**那么它就可以把你的写Merge到一起。**

以读为例子，也就是可以进行指令预取（读一片指令），也可以进行多个transaction合并（比如连续几个连续的DDR空间），**该比特需要和AXCACHE的\[2\]\[3\]bit一起使用。**
值得注意的是，这个是否可以**Modifiable实际上也决定了是否允许写入Cache**，如果不准Modify，那么就意味着你要么**写入Buffer要么写入真正的目的地址**，这样自然不涉及Cache了，因此该比特如果为低，代表你的访存操作和Cache没有关系。

**AXCACHE[2]和[3]**用来分别表示读分配和写分配，所**谓的分配指的是我们什么情况下应该为数据分配cache line。cache分配策略分为读和写两种情况**。读分配的含义是，如果是一次读传输，发生了miss，那么应该在cache中给它分配相关的Cacheline，这个过程叫做Cacheline fill	

<img src="https://pic3.zhimg.com/v2-70deabf23cefe971a0e2d7247ea6e066_b.jpg" alt="img" style="zoom:50%;" />

**当写Cache Miss的时候，才会考虑写分配策略**。当我们不支持写分配的情况下，写指令只会更新主存数据，然后就结束了。当支持写分配的时候，我们首先从主存中加载数据到cache line中（相当于先做个读分配动作），然后会更新cache line中的数据，如下图所示，当出现写Cache Miss的时候，会先从主存加载数据到Cacheline，然后将要写的那一部分更新到Cacheline相应的位置（图中先Cacheline fill，然后写那个蓝色长方形的灰色格子）。

<img src="https://pic1.zhimg.com/v2-b22706876abf2d3f25db9db654e529c0_b.jpg" alt="img" style="zoom:50%;" />

4 bit 编码的组合

![img](https://pic3.zhimg.com/v2-27d30f0bad26054384dc1b35a58cb26e_b.jpg)

## 6. 原子访问机制

### 6.1原子访问机制

对于AXI3.0, 带有locked模式；而由于locked模式受支持少，且容易影响性能，因而在AXI4.0被舍弃

> 当为'b10的时候，即**Locked访问**的时候。如果Master1正在访问某个Slave，此时其它的Master也想访问该Slave的时候，那对不起，已经锁住了，不允许发起访问。相应的Interconnect也必须支持这一机制，仲裁器此时不允许别的Master访问相应的Slave。
>
> 当为'b01的时候，即**Exclusive访问**的时候，这种时候不会将整个访问路径都给阻塞住，而是用一个flag标志，比如当Master1访问该Slave的时候，会有个**Monitor**记录下这个地址，当其它的Master要访问这个地址的时候，就知道不允许访问，它可能会一直的loop，也有可能进入IDLE模式，当Master1访问结束以后，通过中断唤醒它，再去进行相应的访问，这种模式就不会将整个总线阻塞住。

接下来我们看一下**Exclusive access**整个流程是怎么样的：

- Master对某个地址发起exclusive read；
- 过了一会，Master尝试对同样的地址进行exclusive write来完成整个exclusive operation；
- 如果在读写之间没有别的Master往这个地址写数据，则上述的exclusive write成功，否则失败。



首先看第一个例子：

- Master0首先读0xA000这个地址，没有问题，Monitor会记录下这个地址以及对应的Master ID号。返回ExOKAY，读回来的值为0x1；
- Master1读0xB000这个地址，Monitor同样记录下这个地址以及对应的Master ID，返回ExOKAY，读回来的值为0x2；
- Master0去写0xA000这个地址，这个地址之前已经读过了返回ExOKAY，并且中间没有别的Master写过，所以没问题，写下0x3，返回ExOKAY；
- Master1去写0xB000这个地址，这个地址之前已经读过了返回ExOKAY，并且中间没有别的Master写过，所以没问题，写下0x4，返回ExOKAY；

![img](https://pic4.zhimg.com/v2-dbee69f22db6df610ad0bc92741c2a9f_b.jpg)

第二个例子

我们看一下第二个例子：

- Master0首先读0xA000这个地址，没有问题，Monitor会记录下这个地址以及对应的Master ID号。返回ExOKAY，读回来的值为0x1；
- Master1读0xA000这个地址，Monitor同样记录下这个地址以及对应的Master ID，返回ExOKAY，读回来的值为0x1；
- Master0去写0xA000这个地址，这个地址之前已经读过了返回ExOKAY，并且中间没有别的Master**写过**，所以没问题，写下0x3，返回ExOKAY；
- Master1去写0xA000这个地址，这个地址之前已经读过了返回ExOKAY，**但是中间有别的Master写过，所以Exclusive write失败，返回OKAY**；（对于Exclusive access操作而言，如果Slave返回的是OKAY而不是ExOKAY则代表Exclusive Failure）

![img](https://pic3.zhimg.com/v2-24623adac725dc81f647035ba0b45afa_b.jpg)



实现方法：

初始处于Open状态，需要的时候进入Exclusive状态，然后再将地址和ID存入表格中；Master完成一次Exclusives Operation，则清空对应的表项，同时回到Open状态。如果别的Master对同样的地址进行Store Exclusive的话，则会查表发现还占着，因此Fail，写不进去。

### 6.2 AXI的响应信号

AXI的响应有两种信号，分别是RRESP和BRESP。

其中的**RRESP**实际上是对应READ burst的每次transfer(beat)

而**BRESP**对应的是整个WRITE burst，即一个完整的transaction全部完成以后才会回**BRESP**。

> 个人猜测：由于RRESP与读数据是同一个通道，因而方便一起返回
>
> 而BRESP是独立的一个通道，只有全部写完才返回才有意义
>
> **这是因为整个读操作来说，读地址是从A到B，而读数据与读响应是从B到A，返回读数据时可以顺带返回读响应**
>
> **整个写操作来说，写数据与写地址是从A到B，这与写响应反向。因而只有全部写完之后再返回写响应是比较合理的**

响应总共有四种情况：

- **00代表OKAY**，普通的访问如果是OKAY的话则代表访问成功，如果是Exclusive访问的话，返回OKAY则代表失败；
- **01代表EXOKAY**，代表Exclusive访问成功；
- **10代表SLVERR**，即Slave ERROR，Slave指定的ERROR。代表访问Slave成功了，但是Slave要求返回Error，比如写了不该写的地方；
- **11代表DECERR**，即Decode ERROR，用来代表访问了禁止区域，比如跨4K了，或者不准读写的地方，或者访问到了Default Slave；

实际上10和11没有绝对的界限，比如同一种错误，访问禁止区域，你回复SLVERR或者DECERR实际上都可以的，不一定要完全符合ARM官方文档的描述。

## 7.AXI的Ordering model

一般ID的不同对应着不同的Master。对于同一ID的传输，必须保序。对于不同ID之间的传输，可以乱序。

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240429205306036.png" alt="image-20240429205306036" style="zoom:67%;" />

### 7.1 Transfer ID

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240429205440477.png" alt="image-20240429205440477" style="zoom:50%;" />

> 在AXI4.0中，WID被舍弃，因为这会导致设计的复杂度

以下是有关ID必须支持的一些点：

- **对于AXI transfer，都有着对应的ID号；**
- 对于一次transaction的所有的transfer，使用的都是**同一个ID号**；
- 对于同一个Master，可以使用**不同的ID**（同一个Master但是多线程）；
- Slave的ID应该是**可配置位宽**的；（需要一些额外的信息）

#### 7.1.1 write or read

![img](https://pic3.zhimg.com/v2-f4cb8c8dac4087d4e2352210155528ba_b.jpg)

如下图所示：

- 先发出了**地址为A，AWID为0**的写请求，然后又发出了**地址为B，AWID为1**的写请求，**这就是前面所讲的Outstanding；（其数量称为Write issuing capability）**
- 可以看到红色的写请求虽然在蓝色的写之后，但实际上先完成了，也就是说**对于不同的ID，整个transaction是可以乱序的；**
- 对于蓝色的写和红色的写，可以看到在蓝色的写中间穿插了红色的写，也就是说对于不同的ID，彼此之间是可以**Interleave**的（其数量称为Write interleave capability，在AXI4中已移除WID，因此也不支持Interleave，移除的原因是因为设计过于复杂，且需要消耗额外的大量面积）；
- 我们再看，无论是蓝色还是红色还是绿色，都是先写的0，再写的1，再写的2，最后再写的Last。也就是说**对于同一ID而言，内部必须是保序的；**

![img](https://pic2.zhimg.com/v2-6d6002440dab0b3e565da7ecc9e5f069_b.jpg)

> 有一点需要注意，就是读的话每一次transfer，Slave都会返回相应的RID，而写的话，只有最后一笔，才会返回相应的BID，对应整个Transaction。

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240429210145754.png" alt="image-20240429210145754" style="zoom: 50%;" />

#### 7.1.2 read and write

很多时候我们不是只读或者只写，而是先读一个地址，然后进行一系列的运算再写回去。**首先AXI协议它自己是不支持这一机制的，也就是不保证读和写之间的Ordering**，如果想实现该机制，应该自己去实现，以下是实现的一些建议：

- Master必须等**相应的BRESP返回以后**才允许对同一地址的读；

- Master必须**等相应的RRESP和RLAST返回以后**，才允许对同一地址发起相应的写；

  > 也就是说这种情况下不支持Outstanding

<img src="https://pic2.zhimg.com/v2-fc5098377880123ca12f95e3d5184ca5_b.jpg" alt="img" style="zoom:80%;" />



实际上CPU的一些Memory barrier机制，就是通过这种方式实现的，正是因为这种方式不允许Outstanding，不允许乱序，因此带宽利用实际上很差，所以特别的慢。如果跑过相关指令的朋友，应该是可以切身感受比一般的读写，要慢了起码一个数量级。正是如此，大家也应该可以感性的理解AXI的Outstanding对吞吐量的提高有多显著的帮助。

#### 7.1.3 AXI ID use

前面提到过，Slave的ID应该是**可配置位宽**的，因为需要**一些额外的信息**。比如根据ID，怎么返回给对应的Master？（**很多时候并不是一个Master就对应着唯一的一个ID，不同的Master使用同样的ID也是有可能的！**）

如下面的例子，通过在Slave侧的最低bit，用该bit区分这次transaction是来自Master0还是Master1。

![img](https://pic2.zhimg.com/v2-705e00bcf74245e1ef795c51a2031821_b.jpg)

比如Master1发出了一个访问，Interconnect相应的在最低比特添加了一个1，这样Slave返回的时候，Interconnect就知道这次Transaction是对应于Master1，则不会回复给错误的对象。**有些时候甚至是Interconnect连Interconnect，这个时候就需要给ID增加更多的bit了。**

此外通过id知道这次transaction是来自哪个Master，还可以做很多额外的事情，比如对于同一ID的保序操作，对于同一ID分配相应的outstanding id，不同ID之间的乱序等等。总而言之：**如果你的设计中Slave需要知道这次Transaction来自于哪个Master**，那就不妨使用该机制。

<img src="https://pic1.zhimg.com/v2-86af8886fba3c9690e06858299945634_b.jpg" alt="img" style="zoom:67%;" />

> 通过低位宽来区分**到底来自哪个master**，然后再通过高位宽来确定**同一master的不同的ID操作**

## 8.数据总线

**首先是Write Strobes信号**，Strobe信号用处非常广泛，只要用到AXI，基本都会实现该机制，即使是自定义总线基本也都会实现该信号。Strobe的英文翻译一般叫做“闸门”，听上去就很直观了，其类似于掩码信号，**当我们只想写特定的字节**，就需要使用该信号。Strobe为高相应的数据可以通过，进行传输，否则则会被堵住。

该信号实际上没有值得多讲的地方，看下面这个图就知道怎么使用了，对应的bit对应相应的Byte。注意WSTRB是可以不连续的，也可以全为0，实际上。

<img src="https://pic3.zhimg.com/v2-e9beb00f4b704cfe0a0c3add53c1a7b2_r.jpg" alt="img" style="zoom:67%;" />

**然后是大小端的问题**

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240430110008053.png" alt="image-20240430110008053" style="zoom:67%;" />

![img](https://pic4.zhimg.com/v2-8ae69f2f0090a8c679f721f3ba43984b_b.jpg)

## 9.非对齐访问

通常情况下，对于32bit的传输，一般是要满足4-Byte对齐，即访问地址的低两bit应该为00。AHB便是如此，不支持非对齐访问。而AXI非常贴心的加入了非对齐访问机制，可以满足用户相应的需求。

**对于AXI的非对齐访问，非常直观的判断标准就是AxADDR的值是否和AxSIZE相匹配**，如果AxSIZE对应着8Byte（8位为一个单位），那么AxADDR的低3bit应该就为0，否则就是非对齐访问。

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240430110414615.png" alt="image-20240430110414615" style="zoom:50%;" />

再看第二个例子，**其地址为0x03，对应的AXSIZE为16bit。**第一次transfer**相应的也只会去写3这个字节**。

然后第二次transfer会去写4、5字节，第三次transfer写6、7字节，第四次transfer写8、9字节，第五次transfer写A、B字节。

其实第二次transfer和第三次transfer，你如果是读的话，假如数据位宽是32bit，那么按理说读到的都是整个4、5、6、7这四个字节，但设计中一定要注意好了，该用哪两个字节，别因为重复导致弄错了。（**传输的时候，数据总量肯定是跟BUS宽度一致的，但是多少字节有效，是根据你的SIZE决定的**）

![img](https://pic1.zhimg.com/v2-5957de75703a1d17ac59ebec82bb9ad4_b.jpg)

很多IP设计的时候实际上不支持非对齐访问，是否需要非对齐访问取决于你的应用场景，加上非对齐访问的话，会让逻辑变得很复杂，因为要计算对齐地址，取相应的数以后还要进行字节选择，**会增加组合逻辑，导致频率受到影响**，如果不是硬性需求不用加上对非对齐访问的支持。

# AXI-Stream

本篇文章给大家带来AXI-Full的兄弟协议，AXI-stream。该协议在AMBA4中推出，**AMBA4中总共有以下三种跟AXI相关的协议：**

- **AXI-FULL：**或者直接简称AXI，我们之前的文章讲的都是这种协议；
- **AXI-Lite：**简化版本的AXI协议，少了很多特性，如果对之前的AXI文章都理解了话，该协议非常简单，不用特地去学，看一下接口信号就知道是怎么回事了；
- **AXI-Stream**：用于高速数据流传输，非存储映射接口；

**存储映射：**在这里我们首先解释一下**存储映射 （Memory Map）**这 一概念。如**果一个协议是存储映射的，那么主机所发出的会话（无论读或写）就会标明一个地址。**这个地址对应系统存储空间中的一个地址，标明是针对该存储空间的读写操作。这个非常好理解，前面我们讲解AXI协议的时候，都是**针对某个地址进行读写，那它就是存储映射的**。

<img src="https://pic1.zhimg.com/v2-ee755e89cbb2192a5c662319ef2f6ed4_b.jpg" alt="img" style="zoom: 67%;" />

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240430111036908.png" alt="image-20240430111036908" style="zoom:67%;" />

**握手机制**

其握手也可以分为三种情况：

- **TVALID在TREADY之前拉高**：这也是最常见的一种情况，后级因为种种原因暂时处理不了新的数据，**因此会反压住前级模块**，前级必须维持住VALID和INFORMATION不变，直到握手成功才允许更新，**当握手成功也就意味着下级模块拿到了前级传的数据。**

  实际上在很多情况下，是**不允许反压**的。比如DSP（数字信号处理）系统，要求**实时的源源不断的处理数据**，这种情况下一旦反压，新的数又来了，就会丢数。**一般这种系统出现这种情况是因为工作异常导致的，会直接报中断处理**。

<img src="https://pic1.zhimg.com/v2-c2541efc3d88c9a5e73508375662de68_b.jpg" alt="img" style="zoom:67%;" />

- **TREADY在TVALID之前拉高**：这种情况后级模块随时可以接收数据，一直在等着前级模块给数，比如前级模块还未开始工作，刚对其进行配置允许其采集数据，便是这样的情况。

<img src="https://pic1.zhimg.com/v2-a07bce73aed173843aa426b08868254c_b.jpg" alt="img" style="zoom:67%;" />

- **TVALID和TREADY同时拉高**：拉高当拍直接握手成功。

<img src="https://pic1.zhimg.com/v2-da47a1246a7f7eb584fcc7224a32b0e0_b.jpg" alt="img" style="zoom:67%;" />

同样的，VALID的逻辑生成，一定不能够依赖于READY，否则很有可能会造成死锁。但是反过来READY是可以依赖于VALID的，可以看到VALID拉高READY再拉高，**不过现在一般也不这么做了，基本都是完全解耦的。**

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240430111447409.png" alt="image-20240430111447409" style="zoom:67%;" />

**首先是TDATA**，这个没什么好解释的，就是一次transfer传输的数据，一般强制要求为8的整数倍，以和其他信号相对应。

**然后是TSTRB**，所谓的Strobe就是闸门，其每一个bit和DATA的每一个字节相对应，用来表示DATA的数据是否有效。

**然后是TLAST**，由于AXI-Stream在传输之前，也不知道你要传输多少笔数据，因此需要TLAST这个信号作为标志，它用来表示一组数据即Packet的最后一笔Transfer，当TLAST拉高，则代表整个的传输结束了。

**然后是TKEEP**，每一个bit与Data的字节对应，用来表示DATA的数据是否需要传输

要理解TKEEP 和 TSTRB区别，就在于理解**数据可能是无效的，但是为了凑齐，我还是将其传输**，因而二者配合就有了

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240430112423196.png" alt="image-20240430112423196" style="zoom:67%;" />

**Position Byte**：可以看到Position实际上是占据了这个位置的，相当于作为第二Byte来使用，虽然里面的数据是无效的。这种情况一般是传输队形要求固定，但每一组数据的某个Byte可以不看，就用以下的Position Byte。

<img src="https://pic2.zhimg.com/v2-daaf2eddd9b5932a65aa50bd85ea0b49_b.jpg" alt="img" style="zoom: 67%;" />

以下说明一下Position byte具体应用场景：

- Data Mark：比如在视频处理当中，可以使用Position Byte当做一帧的开头；
- Error Detection：每过多少比特插入一个Position Byte用于检测错误。如果不符合规律说明丢数之类的问题；
- Data Group：就比如上图中，**一次传4个Byte的数据，但是有一组或者两组数据是无效的，这样就可以用Position Byte**。让数据的Group是规律的，同时**Slave也可以感知到某个数据是无效的。**

我们看下面这样的一个例子，**该例子对应于上面的Data Mark**。我们有这样的一个图片，总共16个Pixel，一共是4行4列。我们希望发过去，Slave那边知道这是一次传输的开始，因为它那边可能也不知道这是无用数据还是真正要用的数据。

这种情况下我们就可以在**起始加入一个Position Byte。这样Slave一看，就知道一帧图开始了。**然后正常的收数据，当第五次Transfer的时候（W4），**后面3个Byte都是可以丢弃的，即设置为Null Byte。**就这样五次Transfer作为一次循环。**相比原始图像而言，可能浪费了一个周期，但是某些应用场景下Slave就是需要感知一帧图像的开始，使用这种方式就非常的合适。**

![img](https://pic2.zhimg.com/v2-a75f55e32422dbfeb0b6cc1cc05ac105_b.jpg)

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240430112749874.png" alt="image-20240430112749874" style="zoom:67%;" />

**TDEST**为数据流提供路由信息，比如下面的例子，**通过TDEST信号，就可以知道是发到哪个Slave。**实**际上该信号用的很少很少，因为AXI-stream基本上是点对点的，很少有Interconnect的说法。**

<img src="https://pic2.zhimg.com/v2-80b62de2ab1ff9b97ad25afa485bd06d_b.jpg" alt="img" style="zoom: 67%;" />

**TUSER**其实跟TID差不多，**实际上都是附带一些额外想传输的信息而已**，该信号更加的灵活，用户可以附带自己想要传输的额外信息**。**

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240430112930861.png" alt="image-20240430112930861" style="zoom:67%;" />

# AXI设计中的关键问题

## AXI如何提高性能

1. **Outstanding**对性能的影响最为显著

   尤其是主从之间的通讯需要很多拍才能完成的时候，它能够利用这段延时去发起别的传输，**即通过发起操作来弥补这部分的延时**

2. **Out of order乱序**

   **然后是乱序（Out of Order）**，这个提高性能主要体现在Slave无法及时响应，就可以让后发起的请求先完成。就比如你去超市买菜，你前面的人刚好手机没电了，你是等他充好电还是先让你付款？

3. **交织读写，纯垃圾**

   最后是Interleave，即交织读写，这个其实就是在空闲的Cycle中（气泡Cycle）插入读写事务，其实有点像乱序，可以理解成更加细粒度的乱序。当然粒度这么细，导致设计很复杂，所以在AXI4中写交织就被移除了

## AXI的memory feature

**最后是Memory Type的不同**，基于Memory类型的不同，可以决定**你的读写是否可以Bufferable或者Cacheable。**比如某些外设寄存器，**你写完是希望直接改变系统当前的运行状态的，希望即刻生效，这种当然既不可以Cacheable也不可以Bufferable。**而有些外设寄存器，比如写Flash Controller的寄存器，你写完以后，稍微延迟一些Cycle或者有乱序是可以接受的，这种情况就可以Bufferable。

<img src="https://pic1.zhimg.com/v2-9f5ed435f73df433a730d09ee8c72d30_b.jpg" alt="img" style="zoom:67%;" />

## AXI的Order考虑

首先我们看一下Write Channel和Read Channel之间的一个Order，**AXI协议并没有对它们之间的Order有一个约束**，如果想要实现该机制，就应该**在之前的transaction收到最后的response以后才允许发起新的transaction。**

你可以通过**配置某个寄存器**，来开启该功能。这个设计起来不是很复杂。如果你使用的是ARM或者X86或者主流的商用CPU，你可以使用memory barrier指令来确保这个顺序。



然后我们看一下对于Master而言，**对于来自Slave的Response，是可以乱序的**，通过ID来区分好就行。此外对于**Read response而言，是可以Interleave的**。读交织如下图所示。

而对于同一ID而言，**即Master发出的同一AWID/ARID而言，相应的数据必须是顺序的，如果不是顺序的，怎么知道哪个数据对应于哪一个**

![img](https://pic3.zhimg.com/v2-653adf5be05ed7c706f77614414286ce_b.jpg)

**我们看一下Slave的实现**，Slave想实现Order Model，比Master要复杂的多。

我们想象一下这样的一个例子，一个Master去访问一个Slave，而这个Slave有很多不同的Memory Type，比如既有**访问较快的寄存器**，也有相对慢一点的SRAM，**甚至还有DDR/FLASH**。比如你去写一个DDR Controller，你去写它的配置寄存器，很快，你去写DDR，其实也要走这个DDR Controller，这样一次访问就非常非常的慢。但是，**由于是同一个Master的访问，其ID是相同的，相同ID要保序，那难不成每次写DDR，都要等着数据回来？**显然非常的不合理，那么在其内部就可以做一个**转换表**，如下图所示，**来的时候是同样的ID，但Slave可以将其转换成不同的ID，这样就可以Outstanding了，也可以乱序了**，非常的Amazing啊。注意，**Slave内部是可以Out of Order返回的，但你返回给Master还是得顺序的。需要一块额外的存储空间来缓存这些需要返回的数据及相关信息**，当来了，你就返回给Master。

<img src="https://pic4.zhimg.com/v2-51dd13975c9b70f7d9261ae57a3de497_b.jpg" alt="img" style="zoom:67%;" />

我们再看一下Master的实现，当Master有多个Engine，比如DMA。**每个DMA发出的ID是不一样的，当发出多个Outstanding的请求**，Slave也相应的根据请求把数据拿回来了。**相应的准备若干个Queue，当ID为0的返回的时候送给Queue0**，当ID为0的最后一笔返回的时候即RLAST拉高的时候，做相应的处理返回给发出请求的Engine。**其实就是需要一定的缓冲机制，来避免接不到数据等情况。**

> **Master需要数据queue来支持out of order；** 

> **Master需要buffer来支持outstading；**

<img src="https://pic4.zhimg.com/v2-34a2ac771a3cb9d0e9de70512f61a16b_b.jpg" alt="img" style="zoom:67%;" />

## AXI的debug

大多数的时候，都认为各自是遵循总线协议的，但还是需要一定的能力，来做Debugging，毕竟某些IP会不遵守协议，会出错。

 

- 使用counter来监视发起了多少次传输事务（transaction），以Master那边为例，当发出一次outstading请求，相应的counter+1，当收到最后的response，counter-1。这样系统挂死以后，发现某个Master的monitor记录counter不为0，这样就可以快速定位错误；

- - axvalid&axready=1，cnt+1
  - 对于写，当bvalid&bready=1，cnt-1
  - 对于读，当rvalid&rready&rlast，cnt-1
  - 还可以记录transfer数量

- 还可以记录更多的信息，如axlen、axid、axsize等；

- 使用timeout机制；

> 发起一次请求——cnt+1；收到resp之后——cnt-1

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240501100439493.png" alt="image-20240501100439493" style="zoom:67%;" />

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240501100517308.png" alt="image-20240501100517308" style="zoom:67%;" />

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240501100531890.png" alt="image-20240501100531890" style="zoom:67%;" />

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240501100702859.png" alt="image-20240501100702859" style="zoom:50%;" />

> 我的理解是，有5个master，分为优先级的三组，最高的那组为M0, 其次的是M1、M2和M5的那组，最后的是M4和M3的那组。
>
> 三组之间首先存在优先级，然后在组内，再跟据谁没有被使用过进一步确定优先级
>
> **这种设计就挺好的，不过有点考验master的设计方式**

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240501100859338.png" alt="image-20240501100859338" style="zoom: 50%;" />

**时序的收敛问题**

AXI本身采用的握手机制非常灵活，使其的周期也很灵活，没有AHB和APB那样要插入延迟的要求，**因而很容易插入寄存器形成流水线，这也就是为什么AXI很容易插入流水线的原因**

一般是有三种方式，打断Valid，打断Ready或者都打断。在IC设计中，无论使用的是否是AXI总线，**只要用Valid和Ready握手机制来传递数据，都可以使用该方法让时序收敛**，不一定是局限于某个总线协议。大家完全可以将其用在模块内部和模块与模块间的流传输。

**上面这种机制用在总线上，一般称之为Register Slice**。先说说Slice的作用，在SoC中，**如果Master和Slave的距离比较远，那么它们之间的bus信号要满足timing就可能有点困难**，比如AXI中的 ARADDR、ARVALID这些信号从Master出来，要去很远的Slave，那么中间就要加很多的buffer，这引入的buffer delay就可能导致我们希望的timing不满足。这个时候就需要插入slice，给每个控制信号在中间加一级寄存器，把较长的走线缩短。当然插入的slice依然要保持bus的协议标准。简单来说，一个slice就是下面中间的那个模块。

我们看一下在Register插入的位置，可以如下图所示（其实有很多可以插入的位置）。一般AXI各个通道是分开的，**我们在哪个通道加入register slice，相应的就会晚了一拍，实际上晚了一拍完全不影响逻辑功能。**

<img src="https://pic4.zhimg.com/v2-9e1df44266a85c0657cfd34e83abdee7_r.jpg" alt="img" style="zoom: 33%;" />

如果一个组合逻辑需要50ns才能完成一个数据算数，那么假设时钟周期设置为50, 就会导致50ns才能完成一个数据运算

但是如果将50ns拆成5个10ns，也就是将一个组合逻辑分为5个部分，然后将时钟周期设置为10ns，这时候，除去最开始的50ns无法出数据，之后的每个周期都能出一个数据。

**AXI便于插入流水线，很大程度是因为插入寄存器的时候，不会影响其功能性**

<img src="https://pic3.zhimg.com/v2-6a2182c2bf1b9c490fe2cb81a2f981f6_b.jpg" alt="img" style="zoom:67%;" />

# 别吹AXI总线了，她到底好在哪？

## 1. 如何评估性能

关键指标：延迟

性能中一个关键的指标就是延迟，**什么是延迟（Latency）呢**？

比如有个人，叫做小明，在上大学需要用钱，因此给父亲寄信。父亲收到信以后将钱寄给小明同学。整个过程一共花费了2+3=5days。这就是整个寄信加寄钱所花费的延迟。

<img src="https://pic1.zhimg.com/v2-d8eb4ec72b985931b7432af1f2d3a0aa_720w.jpg?source=d16d100b" alt="img" style="zoom:67%;" />

我们假设每次寄钱只能够寄两万，由此我们又可以引出**带宽（Bandwidth）**的概念。所谓的带宽即单位时间可以传输的数据量多少。在上述的例子中BW=2W/5days = 0.4W/day。

如果按照这样的一个带宽速度，寄100万需要多久呢？我们用数据量除以带宽：100W/(0.4W/day)=250days!

如何让寄100万更快呢？一种很直接的方法，就是在收到钱之前，狂发信。父亲每收到一封就把钱寄回去。理论上这样只需要5天多一点，就可以寄100W！（Outstanding，但其实这里的理解也不是那么正确，要是严格来说，5天也不够，因为发出请求都需要消耗时钟周期，而发钱也需要时钟周期——但是这样绝对是比发一次请求再发一次钱要快的多）

**作者：lawliet**
**链接：https://zhuanlan.zhihu.com/p/643863702**
**来源：知乎**
**著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。**

> 很牛逼的理解，减不了一点

## 2.Outstanding对性能的影响

有了上面的例子，大家已经知道了什么是延迟和带宽了。由此我们接着往下讲。在之前的文章已经讲过，**AXI提高传输速度主要有三板斧**，**以下排名分先后顺序**：

- Outstanding
- Out-of-orde(ooo)
- Interleave

因此我们重点讲一下AXI的三板斧，**首先是Outstanding**。所谓的Outstanding就是在任务完成之前就可以下达新的任务，即“在路上”。Outstanding是上面三种方法中，提高总线性能最有效最简单的方法！

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240501152214399.png" alt="image-20240501152214399" style="zoom:67%;" />

> 我自己再想一个例子，假设Master向Slave发送读请求，需要5个周期送达——Slave花费5个周期准备数据——Master再过5个周期才能收到，这样以来，读一次就要花费15个周期。但是如果支持Outstanding的数量为16个，那么我就可以连续15个周期都发送请求，通过发送请求干活来处理这段啥都不干的冒泡了

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240502102528403.png" alt="image-20240502102528403" style="zoom:67%;" />

> 充分利用了准备时间（气泡时间来做准备与发送，从而形成类似于流水线的规模）

上面那个例子中，是过了5s再发出下一个Outstanding命令，实际上一般不会考虑这么多。当还没有发满的时候，就接着发。我们看下面这个例子：



<img src="https://pic2.zhimg.com/80/v2-8efccd91ffcbf24475bec5e2c634f6ed_720w.webp" alt="img" style="zoom: 67%;" />

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240502110931849.png" alt="image-20240502110931849" style="zoom:67%;" />

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240502110954853.png" alt="image-20240502110954853" style="zoom:50%;" />

<img src="https://pic3.zhimg.com/80/v2-0de0857e5cb03470693fbba3dede261e_720w.webp" alt="img" style="zoom:150%;" />

我们再来思考一个问题：Outstanding数量越多越好吗？

答案当然是否定的，更多的Outstanding意味着更多的硬件资源开销，比如就需要很多的Buffer去存储已经发送的CMD。但通过上面的例子我们可以看出，当Outstanding再增加，**其性能也已经饱和了**，不会再增加。因此选择一个合适的Outstanding数量，基于此进行设计很重要。

既然Outstanding不是越大越好，那我们怎么去估计Outstanding数量呢？

> 首先，频率为500MHz，这意味着每秒有500百万个周期，因此每个周期的时间为1/500百万，即2 ns（纳秒）。
>
> 其次，AXI cmd是基于burst-8传输的，这意味着对于一个命令（cmd），需要连续8个周期（cycle）传输相应的数据。
>
> 第三，第一笔数据返回的时候，需要200 ns的时间。这意味着从发出命令开始，到收到第一笔数据，需要等待200 ns。
>
> 因此，**我们可以计算出在等待第一笔数据返回的时候**（等待第一个200ns），AXI接口可以发出多少个命令。由于每个命令需要8个周期，因此在200 ns的时间内，可以发出200 ns / (2 ns/cycle * 8 cycle/cmd) = 12.5个命令。由于命令的数量通常是2的幂次方，因此实际中可以设置为8或者16。
>
> 这个例子只是一个简单的估算，实际中可能会更加复杂，需要考虑更多的因素。但是这个例子可以帮助理解AXI接口在数据传输中的基本思路。

我们再看一下Outstang的优缺点：

- 优点：提高带宽，减少平均延迟
- 缺点：可能会增加绝对延迟，一定会增加面积（需要Buffer存CMD，如果没有乱序，这个时候额外的面积其实还不太多，如果要支持乱序，那额外的面积开销就大了，因为还要存DATA）

此外，Outstanding是ooo(out of order)和Interleave（交织）的基础，没有Outstanding，后面二者是无从谈起的。

## 3. Out of order对性能的影响

Outstanding是相对于发送请求方而言的，而Out of order 则更多是相对于接收方返回数据和响应而言的

因为在上面的例子中，我们是假定各个模块准备数据的延迟是一样久的。但是在下面这个例子中，可以看到B需要更多的周期去准备相应的数据，又浪费了Cycle！假如B的数据是来自DDR的，此时还用这种方式且不支持乱序，那就非常Naive了，可能百分之九十多的周期都被浪费了。![img](https://pic2.zhimg.com/80/v2-2876a92cb7cab83c94aa1bc340fe3775_720w.webp)

这种情况下，我们就要使出第二板斧，乱序。可以看到下图这个例子，我们可以让快速的Slave先传，然后再传慢的Buffer。

![img](https://pic2.zhimg.com/80/v2-31ef19c0e9b46d3f8ed682e5e173d67d_720w.webp)

如果要做Out of Order。那么Master端就需要一个非常大的Buffer了。以上图例子中，就需要40Byte的Buffer了。因为最差的情况，数据ABCD是倒着回给你的，但是你还是得按照顺序用，那只能用额外的Buffer存储了。**因此如果不是某个Slave一定会乱序，且对性能影响很大，一般做Master是不建议支持Out of Order的。**

## 4.交织 Interleave

**交织其实就是更细颗粒度的乱序。普通的乱序基于的序列是多个transfer合成的transaction，也就是说序列乱的是transaction。**

**而交织则是进一步深入到transfer，在transaction1中插入transaction2的transfer**

假设一次CMD需要4次transfer，但是Slave数据准备很慢，每次只能准备一个CMD的两次transfer（假设可以同时准备其他的CMD的）

![img](https://pic2.zhimg.com/80/v2-88ea4c1a3a768f8270c3e62fcadbe635_720w.webp)

> 这样就会导致有一个周期被浪费

<img src="/home/zerohua/.config/Typora/typora-user-images/image-20240502112919451.png" alt="image-20240502112919451" style="zoom:67%;" />

## 5.Other

### 5.1 QoS

QoS实际上是计算机网络的概念，QoS（Quality of Service）即**服务质量**。 在有限的带宽资源下，QoS为各种业务分配带宽，为业务提供端到端的服务质量保证。 例如，语音、视频和重要的数据应用在网络设备中可以通过配置QoS优先得到服务。

ARM基于此概念，在AXI4新增QoS相关的信号，为AxQOS，共4bit，其定义了每次传输的优先级。我们一般认为0xF代表最高的优先级，0x0代表最低的优先级。 QoS一般有如下的作用：

- QoS可以用来解决访问冲突问题，当同时访问先仲裁得到优先级高的；
- Slave可以根据QoS来reorder或者优先考虑先回应哪笔传输；

下图是一个典型的复杂系统，这种系统就需要QoS来确定，到底哪个Master可以优先获得总线访问机制，因此就需要设计复杂的QoS控制系统，此系统一般是可以通过软件实时的去改配。

我们再思考一下，QoS主要是优化延迟还是带宽呢？

AXI的QoS主要是为了解决延迟问题。通过此值，可以更好的仲裁，以避免延迟大的阻塞住整个总线，而严重影响其它模块的访问延迟。当然它一定程度上也会影响到带宽。

## 5.2 Data and Transfer

**上面我们讲的这些AXI的优化策略，其实ARM的初衷，主要是为了DDR设计的**。因为大部分Slave和Master之间都是紧耦合的，根本就不需要Out of Order和Interleave。那么**如何针对DDR的读写去优化呢**？简单来讲就是你的CMD和DATA要更加符合DDR的胃口。

简单来讲就是你的CMD和DATA要更加符合DDR的胃口。

- 地址对齐；
- 增加Burst Length；Burst Combine；
- 避免Partial Write。即WSTRB有1有0。因为很多DDR是不支持此功能的，你要读回去再写，非常耗时；
- Read/Write Group。即不要写一个读一个，最好是写一组读一组；
- 避免Locked/Exclusive access；（Locked就不说了，Exclusive是一读一写，对DDR非常不友好）/









