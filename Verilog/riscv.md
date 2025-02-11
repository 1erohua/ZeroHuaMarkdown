# RISC-V CPU EX模块文档

## 模块概述

EX模块是RISC-V CPU的执行阶段模块，它接收来自ID阶段的指令和操作数，执行指令，并将结果输出到下一阶段的模块。EX模块还负责处理CSR寄存器的读写操作，以及与除法器和乘法器的交互。

这段Verilog代码定义了一个名为`EX`的模块，该模块主要负责处理指令的执行阶段（EX阶段）。以下是对代码功能的简要概括以及各个部分的用途分析：

### 功能概括
`EX`模块接收来自ID/EX阶段的各种输入信号，包括指令、操作数、地址等，并根据指令类型执行相应的操作，如算术运算、逻辑运算、内存访问、分支跳转等。执行结果通过输出信号传递给后续阶段（如EX/MEM阶段）。

### 主要组成部分及用途

1. **输入信号**：
   - `inst_i`, `inst_addr_i`, `op1_i`, `op2_i`, `rd_addr_i`, `rd_wen_i`, `base_addr_i`, `addr_offset_i`, `csr_we_i`, `csr_rdata_i`, `csr_waddr_i`, `id_axi_araddr_i`, `read_ram_i`, `write_ram_i`, `int_assert_i`, `int_addr_i`, `div_finish_i`, `div_rem_data_i`, `div_busy_i`, `mult_finish_i`, `mult_product_val`, `mult_busy_i`：这些信号用于传递指令、操作数、地址、控制信号等信息。

2. **输出信号**：
   - `rd_addr_o`, `rd_data_o`, `rd_wen_o`, `id_axi_araddr_o`, `read_ram_o`, `write_ram_o`, `inst_o`, `inst_addr_o`, `op2_o`, `csr_we_o`, `csr_wdata_o`, `csr_waddr_o`, `jump_addr_o`, `jump_en_o`, `hold_flag_o`, `div_ready_o`, `div_dividend_o`, `div_divisor_o`, `div_op_o`, `mult_ready_o`, `mult_op1_o`, `mult_op2_o`, `mult_op_o`：这些信号用于传递执行结果、控制信号等信息。

3. **内部信号**：
   - `opcode`, `rd`, `funct3`, `rs1`, `rs2`, `funct7`, `imm`, `shamt`, `uimm`：这些信号用于解析指令字段。
   - `op1_i_equal_op2_i`, `op1_i_less_op2_i_signed`, `op1_i_less_op2_i_unsigned`：这些信号用于比较操作数。
   - `op1_i_add_op2_i`, `op1_i_sub_op2_i`, `op1_i_and_op2_i`, `op1_i_xor_op2_i`, `op1_i_or_op2_i`, `op1_i_shift_left_op2_i`, `op1_i_shift_right_op2_i`, `base_addr_add_addr_offset`：这些信号用于执行算术和逻辑运算。
   - `SRA_mask`, `load_index`, `store_index`：这些信号用于处理移位和内存访问。
   - `op1_mul`, `op2_mul`, `mul_temp`, `mul_temp_inv`, `op1_div_op2_res`, `op1_rem_op2_res`, `op1_div_op2_res_w`, `op1_rem_op2_res_w`, `op1_i_inv`, `op2_i_inv`：这些信号用于处理乘法和除法运算。
   - `sli_shift`, `op1_lower32bit_mask`, `op1_lower32bit_rlshift`, `op1_lower32bit_rashift`, `op1_lower32bit_srawmask`, `op1_lower32bit_srlwshift`, `op1_lower32bit_srawshift`, `sllw_temp`：这些信号用于处理移位运算。
   - `div_reg_waddr`, `mult_reg_waddr`, `reg_wdata`, `reg_we`, `reg_waddr`, `hold_flag`, `jump_flag`, `jump_addr`, `div_wdata`, `div_we`, `div_waddr`, `div_hold_flag`, `div_jump_addr`, `div_jump_flag`, `mult_wdata`, `mult_we`, `mult_waddr`, `mult_hold_flag`, `mult_jump_addr`, `mult_jump_flag`：这些信号用于处理寄存器写入和控制信号。

4. **逻辑块**：
   - `assign`语句：用于解析指令字段和执行简单的逻辑运算。
   - `always @ (*)`块：用于处理复杂的逻辑运算和控制信号，如乘法、除法、分支跳转等。

### 组成方式
代码通过组合逻辑和时序逻辑相结合的方式实现指令的执行。组合逻辑部分主要用于解析指令字段和执行简单的逻辑运算，而时序逻辑部分则用于处理复杂的逻辑运算和控制信号。通过这种方式，`EX`模块能够高效地处理各种指令，并将执行结果传递给后续阶段。

## 变量列表

### 输入端口

- `inst_i`: 32位指令输入。
- `inst_addr_i`: 64位指令地址输入。
- `op1_i`: 64位操作数1输入。
- `op2_i`: 64位操作数2输入。
- `rd_addr_i`: 5位目的寄存器地址输入。
- `rd_wen_i`: 1位目的寄存器写使能输入。
- `base_addr_i`: 64位基地址输入。
- `addr_offset_i`: 64位地址偏移输入。
- `csr_we_i`: 1位CSR写使能输入。
- `csr_rdata_i`: 64位CSR读数据输入。
- `csr_waddr_i`: 64位CSR写地址输入。
- `id_axi_araddr_i`: 32位AXI读地址输入。
- `read_ram_i`: 1位内存读使能输入。
- `write_ram_i`: 1位内存写使能输入。
- `int_assert_i`: 1位中断断言输入。
- `int_addr_i`: 64位中断地址输入。
- `div_finish_i`: 1位除法完成标志输入。
- `div_rem_data_i`: 64位除法余数数据输入。
- `div_busy_i`: 1位除法器忙标志输入。
- `mult_finish_i`: 1位乘法完成标志输入。
- `mult_product_val`: 64位乘法积输入。
- `mult_busy_i`: 1位乘法器忙标志输入。

### 输出端口

- `rd_addr_o`: 5位目的寄存器地址输出。
- `rd_data_o`: 64位目的寄存器数据输出。
- `rd_wen_o`: 1位目的寄存器写使能输出。
- `id_axi_araddr_o`: 32位AXI读地址输出。
- `read_ram_o`: 1位内存读使能输出。
- `write_ram_o`: 1位内存写使能输出。
- `inst_o`: 32位指令输出。
- `inst_addr_o`: 64位指令地址输出。
- `op2_o`: 64位操作数2输出。
- `csr_we_o`: 1位CSR写使能输出。
- `csr_wdata_o`: 64位CSR写数据输出。
- `csr_waddr_o`: 64位CSR写地址输出。
- `jump_addr_o`: 64位跳转地址输出。
- `jump_en_o`: 1位跳转使能输出。
- `hold_flag_o`: 1位暂停标志输出。
- `div_ready_o`: 1位除法器就绪输出。
- `div_dividend_o`: 64位除法器被除数输出。
- `div_divisor_o`: 64位除法器除数输出。
- `div_op_o`: 10位除法器操作码输出。
- `mult_ready_o`: 1位乘法器就绪输出。
- `mult_op1_o`: 64位乘法器操作数1输出。
- `mult_op2_o`: 64位乘法器操作数2输出。
- `mult_op_o`: 10位乘法器操作码输出。

### 寄存器列表

- `reg_wdata`: 64位目的寄存器写数据寄存器。
- `reg_we`: 1位目的寄存器写使能寄存器。
- `reg_waddr`: 5位目的寄存器写地址寄存器。
- `hold_flag`: 1位暂停标志寄存器。
- `jump_flag`: 1位跳转标志寄存器。
- `jump_addr`: 64位跳转地址寄存器。
- `div_wdata`: 64位除法器写数据寄存器。
- `div_we`: 1位除法器写使能寄存器。
- `div_waddr`: 5位除法器写地址寄存器。
- `div_hold_flag`: 1位除法器暂停标志寄存器。
- `div_jump_flag`: 1位除法器跳转标志寄存器。
- `div_jump_addr`: 64位除法器跳转地址寄存器。
- `mult_wdata`: 64位乘法器写数据寄存器。
- `mult_we`: 1位乘法器写使能寄存器。
- `mult_waddr`: 5位乘法器写地址寄存器。
- `mult_hold_flag`: 1位乘法器暂停标志寄存器。
- `mult_jump_flag`: 1位乘法器跳转标志寄存器。
- `mult_jump_addr`: 64位乘法器跳转地址寄存器。

### 线网类型

在Verilog代码中，每个非端口的wire变量都被用来存储中间结果或控制信号，这些变量在整个模块中被使用。以下是每个非端口wire变量的作用：

1. `opcode`: 存储指令的操作码，用于确定指令的类型。

2. `rd`, `funct3`, `rs1`, `rs2`, `funct7`, `imm`, `shamt`, `uimm`: 这些变量分别存储指令的各个字段，用于解析指令的操作和参数。

3. `op1_i_equal_op2_i`, `op1_i_less_op2_i_signed`, `op1_i_less_op2_i_unsigned`: 这些变量用于比较操作数`op1_i`和`op2_i`，分别表示它们是否相等、是否小于（有符号和无符号）。

4. `op1_i_add_op2_i`, `op1_i_sub_op2_i`, `op1_i_and_op2_i`, `op1_i_xor_op2_i`, `op1_i_or_op2_i`, `op1_i_shift_left_op2_i`, `op1_i_shift_right_op2_i`, `base_addr_add_addr_offset`: 这些变量分别存储指令的各种操作结果，如加法、减法、与、异或、或、左移、右移和基地址加上偏移量。

5. `SRA_mask`: 存储算术右移的掩码，用于实现算术右移。

6. `load_index`, `store_index`: 这些变量用于确定内存访问的字节或字的位置。

7. `op1_mul`, `op2_mul`, `mul_temp`, `mul_temp_inv`: 这些变量用于乘法操作，分别存储操作数、乘积和中间结果。

8. `op1_i_inv`, `op2_i_inv`: 这些变量存储操作数的取反结果，用于乘法操作中的补码加法。

9. `sli_shift`, `op1_lower32bit_mask`, `op1_lower32bit_rlshift`, `op1_lower32bit_rashift`, `op1_lower32bit_srawmask`, `op1_lower32bit_srlwshift`, `op1_lower32bit_srawshift`, `sllw_temp`: 这些变量用于不同类型的移位操作，如逻辑左移、算术右移和逻辑右移。

10. `div_reg_waddr`, `mult_reg_waddr`: 这些变量存储除法器和乘法器的写入地址，用于将结果写回通用寄存器。

11. `div_wdata`, `div_waddr`, `div_hold_flag`, `div_jump_addr`, `div_jump_flag`, `mult_wdata`, `mult_waddr`, `mult_hold_flag`, `mult_jump_addr`, `mult_jump_flag`: 这些变量用于控制除法器和乘法器的操作，如写入数据、地址、保持标志、跳转地址和跳转标志。

12. `reg_wdata`, `reg_we`, `reg_waddr`, `hold_flag`, `jump_flag`, `jump_addr`: 这些变量用于控制通用寄存器的写入操作，如写入数据、写使能、地址、保持标志、跳转标志和跳转地址。

13. `csr_wdata_o`: 存储写入CSR寄存器的数据。

14. `jump_addr_o`, `jump_en_o`, `hold_flag_o`: 这些变量用于控制程序的跳转和保持操作。

这些变量在模块的不同部分被使用，以实现指令的解码和执行。



## always 语句块

在这段Verilog代码中，有多个`always`语句块，每个语句块都有其特定的用途和含义。以下是对每个`always`语句块的简要分析：

1. **`always @ (*) begin`（第一个）**
   - **用途**：处理除法操作。
   - **含义**：根据指令类型和功能码，决定是否启动除法操作，并设置相关的控制信号（如`div_ready_o`、`div_jump_flag`、`div_hold_flag`等）。如果除法操作正在进行中或已完成，更新相应的寄存器和控制信号。

2. **`always @ (*) begin`（第二个）**
   - **用途**：处理乘法操作。
   - **含义**：根据指令类型和功能码，决定是否启动乘法操作，并设置相关的控制信号（如`mult_ready_o`、`mult_jump_flag`、`mult_hold_flag`等）。如果乘法操作正在进行中或已完成，更新相应的寄存器和控制信号。

3. **`always @(*) begin`（第三个）**
   - **用途**：根据指令类型和功能码，执行不同的操作。
   - **含义**：根据不同的指令类型（如`INST_TYPE_I`、`INST_TYPE_B`、`INST_TYPE_64IW`等），执行相应的操作，如算术运算、逻辑运算、移位操作、分支跳转等，并更新相应的寄存器和控制信号。

这些`always`语句块的主要目的是根据输入的指令和操作数，执行相应的操作，并更新输出信号和寄存器状态。每个`always`块都负责处理特定类型的指令或操作，确保处理器能够正确执行各种指令。



## assign 语句块

好的，我将代码中的 `assign` 语句分块，并解释每一块的功能：

### 块1：指令译码
```verilog
assign opcode = inst_i[6 :0 ];
assign rd     = inst_i[11:7 ];
assign funct3 = inst_i[14:12];
assign rs1    = inst_i[19:15];
assign rs2    = inst_i[24:20];
assign funct7 = inst_i[31:25];
assign imm    = inst_i[31:20];
assign shamt  = inst_i[25:20];
assign uimm   = inst_i[19:15];
```
这一块的 `assign` 语句用于从输入的指令 `inst_i` 中提取不同的字段，如操作码 (`opcode`)、目标寄存器 (`rd`)、功能码 (`funct3`)、源寄存器 (`rs1`, `rs2`)、立即数 (`imm`)、移位量 (`shamt`) 和无符号立即数 (`uimm`)。

### 块2：输出信号直接传递
```verilog
assign read_ram_o      = read_ram_i     ;
assign write_ram_o     = write_ram_i    ;
assign inst_o          = inst_i         ;
assign inst_addr_o     = inst_addr_i    ;
assign id_axi_araddr_o = id_axi_araddr_i;
assign op2_o           = op2_i          ;
```
这一块的 `assign` 语句将输入信号直接传递到输出端口，用于读内存 (`read_ram_o`)、写内存 (`write_ram_o`)、指令 (`inst_o`)、指令地址 (`inst_addr_o`)、AXI 读地址 (`id_axi_araddr_o`) 和操作数2 (`op2_o`)。

### 块3：比较操作
```verilog
assign op1_i_equal_op2_i         = (op1_i == op2_i)?1'b1:1'b0                 ;
assign op1_i_less_op2_i_signed   = ($signed(op1_i) < $signed(op2_i))?1'b1:1'b0;
assign op1_i_less_op2_i_unsigned = (op1_i < op2_i)?1'b1:1'b0                  ;
```
这一块的 `assign` 语句用于比较操作数 `op1_i` 和 `op2_i`，生成相等 (`op1_i_equal_op2_i`)、有符号小于 (`op1_i_less_op2_i_signed`) 和无符号小于 (`op1_i_less_op2_i_unsigned`) 的结果。

### 块4：算术和逻辑操作
```verilog
assign op1_i_add_op2_i           = op1_i + op2_i              ;	// 加法器
assign op1_i_sub_op2_i           = op1_i - op2_i              ;	// 减法器
assign op1_i_and_op2_i           = op1_i & op2_i              ;	// 与
assign op1_i_xor_op2_i           = op1_i ^ op2_i              ;	// 异或
assign op1_i_or_op2_i 			 = op1_i | op2_i              ;	// 或
assign op1_i_shift_left_op2_i 	 = op1_i << op2_i             ;	// 左移
assign op1_i_shift_right_op2_i 	 = op1_i >> op2_i             ;	// 右移
assign base_addr_add_addr_offset = base_addr_i + addr_offset_i; // 计算地址单元
```
这一块的 `assign` 语句用于执行各种算术和逻辑操作，包括加法 (`op1_i_add_op2_i`)、减法 (`op1_i_sub_op2_i`)、与 (`op1_i_and_op2_i`)、异或 (`op1_i_xor_op2_i`)、或 (`op1_i_or_op2_i`)、左移 (`op1_i_shift_left_op2_i`)、右移 (`op1_i_shift_right_op2_i`) 和地址计算 (`base_addr_add_addr_offset`)。

### 块5：掩码生成
```verilog
assign SRA_mask = 64'hffff_ffff_ffff_ffff >> op2_i[5:0];
```
这一块的 `assign` 语句用于生成一个右移掩码 (`SRA_mask`)，用于算术右移操作。

### 块6：乘法和除法操作
```verilog
assign mul_temp     = op1_i * op2_i;
assign mul_temp_inv = ~mul_temp + 1;
```
这一块的 `assign` 语句用于执行乘法操作，生成乘积 (`mul_temp`) 和乘积的补码 (`mul_temp_inv`)。

### 块7：移位操作
```verilog
assign sli_shift                = op1_i << {58'b0,op2_i[5:0]}                                                               ;
assign op1_lower32bit_mask      = ~(32'hffff_ffff >> {59'b0,op2_i[4:0]})                                                    ;
assign op1_lower32bit_rlshift   = op1_i[31:0] >> {59'b0,op2_i[4:0]}                                                         ;
assign op1_lower32bit_rashift   = (op1_i[31] == 1)?op1_lower32bit_mask|op1_lower32bit_rlshift:op1_lower32bit_rlshift        ;
assign op1_lower32bit_srawmask  = ~(32'hffff_ffff >> op2_i[4:0])                                                            ;
assign op1_lower32bit_srlwshift = op1_i[31:0] >> op2_i[4:0]                                                                 ;
assign op1_lower32bit_srawshift = (op1_i[31] == 1)?op1_lower32bit_srawmask|op1_lower32bit_srlwshift:op1_lower32bit_srlwshift;
assign sllw_temp                = op1_i[31:0] << op2_i[4:0]                                                                 ;
```
这一块的 `assign` 语句用于执行各种移位操作，包括逻辑左移 (`sli_shift`)、逻辑右移 (`op1_lower32bit_rlshift`)、算术右移 (`op1_lower32bit_rashift`) 和算术右移掩码 (`op1_lower32bit_srawmask`)。

### 块8：CSR写使能控制
```verilog
assign csr_we_o    = (int_assert_i == 1'b1) ? 1'b0 : csr_we_i;
assign csr_waddr_o = csr_waddr_i                             ;
```
这一块的 `assign` 语句用于控制 CSR 寄存器的写使能 (`csr_we_o`) 和写地址 (`csr_waddr_o`)，在响应中断时不写 CSR 寄存器。

这些 `assign` 语句块分别负责指令译码、信号传递、比较操作、算术和逻辑操作、掩码生成、乘法和除法操作、移位操作以及 CSR 写使能控制。
