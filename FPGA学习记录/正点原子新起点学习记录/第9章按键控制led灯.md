# 没绷住，找了半天bug，竟然是引脚没分配好

### 事情是这样的，当我根据正点原子的思路写好代码之后，发现右边有一个灯处于常亮且微亮的状态。于是一直查代码问题

### 最后发现是脚本文件分配的引脚和我这里的不符，并且之前其实已经修改回来了，但是没执行，导致引脚分配的还是错的

### 每次改完引脚之后记得要用tcl scrip重新执行

## 正文：

**先从一个简单的按键灯开始吧**

```verilog
module key_leds_main(sys_clk,sys_rst,keys,leds);
	
	input sys_clk,sys_rst;
	input[3:0] keys;
	
	output reg [3:0] leds;

	always @(posedge sys_clk or negedge sys_rst)
		begin
		if(!sys_rst)
			leds<=4'b0000;
		else
			begin
			case(keys)
			4'b1110:leds<=4'b0001;
			4'b1101:leds<=4'b0010;
			4'b1011:leds<=4'b0100;
			4'b0111:leds<=4'b1000;
			default:leds<=leds;
			endcase
			end
		end
endmodule
```

看看这次的需求

![image-20231119101222349](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311191012427.png)

![image-20231119101241176](https://gitee.com/zero_hua_no_sb/blog-pic/raw/master/202311191012217.png)

流水效果就是状态切换，这里就需要有一个 **状态**，这个状态是用来切换的，但是也不是记录当前状态的

**准确来说，它是对流水灯或者闪烁这种东西单一作用的。换句话说，它是实现流水灯的另一种形式**

**实现流水灯的另一种形式：流水灯其实就4个状态，并且这4个状态是轮流的，那我们就再用另一个计数器（2位计数器，4个状态分别为00,01,10,11），轮流计4个状态**

直接看代码

```verilog
module key_leds_main(sys_clk,sys_rst,keys,leds);
	
	input sys_clk,sys_rst;
	input[3:0] keys;
	
	output reg [3:0] leds;


		reg [23:0]counter;//0.2秒计数器
		reg [1:0] state;//让流水灯流动的状态，相当于是0.2，0.4，0.6，0.8，总之是对流水灯进行操作的另一个方法
	
	//众所周知的原因，需要一个计数器
	always @(posedge sys_clk or negedge sys_rst)
		begin
			if(!sys_rst)
			begin
				counter<=24'd999_9999;
			end
			else if(counter<24'd999_9999)
				counter<=counter+1;
			else counter<=0;
		end
	
	//只用在流水灯上面的四状态转换
	always @(posedge sys_clk or negedge sys_rst)
		begin
			if(!sys_rst)
				state <= 2'b00;
			else if(counter==24'd999_9999)
				state <= state+1;
			else 
				state <=state;
		end
		
	//按键实现状态切换
	always @(posedge sys_clk or negedge sys_rst)
		begin
			if(!sys_rst)
				leds<=4'b0000;
			else
				begin
					case (keys)
					//左流水灯
					4'b1110:
						begin
							case (state)
							2'b00:leds<=4'b0001;
							2'b01:leds<=4'b0010;
							2'b10:leds<=4'b0100;
							2'b11:leds<=4'b1000;
							default:leds<=4'b0000;
							endcase
						end
					//右流水灯
					4'b1101:
						begin
							case (state)
							2'b00:leds<=4'b1000;
							2'b01:leds<=4'b0100;
							2'b10:leds<=4'b0010;
							2'b11:leds<=4'b0001;
							default:leds<=4'b0000;
							endcase
						end
					//全亮
					4'b1011:
						begin
							leds<=4'b1111;
						end
					//全部闪烁
					4'b0111:
						begin
							case (state)
							2'b00:leds<=4'b1111;
							2'b01:leds<=4'b0000;
							2'b10:leds<=4'b1111;
							2'b11:leds<=4'b0000;
							default:leds<=4'b0000;
							endcase
						end
					default:leds<=4'b0000;
					endcase
				end
		end
endmodule
```

**我后面遇到的还有一个问题就是全部闪烁怎么实现，这里其实也就是两个状态的转换，就是亮与不亮的问题。**