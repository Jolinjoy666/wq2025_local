# GCD 硬件加速器设计

<!-- -->
> #### flag::GCD 硬件加速器
> + 在这次实验中, 我们将设计一个 GCD（最大公约数）硬件加速器, 通过硬件实现辗转相减算法, 让 CPU 只需要写入两个数, 等待计算完成后读取结果即可.

## 为什么需要硬件加速器?

<!-- -->
> #### note::软件 vs 硬件
> 假设我们要计算 GCD(48, 18), 用软件实现需要 CPU 执行多次循环, 每次循环包含比较、跳转、减法等指令. 而硬件加速器可以在每个时钟周期完成一次比较和减法, 大大提升计算效率. 更重要的是, 在硬件计算期间, CPU 可以去做其他事情!

## GCD 算法回顾

辗转相减法的思路很简单:
- 两个数比大小
- 大的减去小的
- 重复上述过程, 直到两数相等
- 此时的值就是最大公约数

举个例子, 计算 GCD(48, 18):

| 步骤 | A | B | 操作 |
|------|-----|-----|------|
| 1 | 48 | 18 | A > B, A = 48 - 18 = 30 |
| 2 | 30 | 18 | A > B, A = 30 - 18 = 12 |
| 3 | 12 | 18 | A < B, B = 18 - 12 = 6 |
| 4 | 12 | 6 | A > B, A = 12 - 6 = 6 |
| 5 | 6 | 6 | A == B, 搞定! GCD = 6 |

## 硬件模块设计

### 寄存器地址分配

我们给 GCD 加速器分配基地址 `0x40000000`, 内部有 4 个寄存器:

| 地址 | 寄存器名 | 读/写 | 说明 |
|------|----------|-------|------|
| 0x40000000 | DATA_A | R/W | 输入数据 A |
| 0x40000004 | DATA_B | R/W | 输入数据 B（写入后自动开始计算） |
| 0x40000008 | RESULT | R | 计算结果 |
| 0x4000000C | STATUS | R | 状态寄存器, bit0=1 表示计算完成 |

<!-- -->
> #### note::GCD 硬件代码组成
> `AHBlite_GCD.v`
> - **功能**: 最大公约数 (GCD) 硬件加速器的 AHB 接口包装器。
> - **寄存器映射**:
>  - `0x00`: 输入 A
>  - `0x04`: 输入 B
>  - `0x08`: 控制寄存器 (Start)
>  - `0x0C`: 状态与结果寄存器 (Done, Result)
>
> `GCD.v`
> - **功能**: 实现欧几里得算法计算两个数的最大公约数。
> - **状态机**: `IDLE` -> `CALC` -> `DONE`。

### GCD 核心模块

GCD 模块的代码位于 `/rtl/GCD.v`, 你需要在其基础上补全关键逻辑.

<!-- -->
> #### note::代码补全
> 根据辗转相减法的算法原理, 完成状态机的设计.

```verilog
case(state)
            IDLE: begin
                done <= 1'b0;
                if(start) begin
                        /********** 需要补充的代码 **********/
                        // 提示: 加载输入数据到 a_reg 和 b_reg, 切换到 CALC 状态
                        
                        /************************************/
                end
            end
            
            CALC: begin
                if(reg_b == 32'd0) begin
                    result <= reg_a;
                    done <= 1'b1;
                    state <= DONE;
                end else if(reg_a > reg_b) begin
                        /********** 需要补充的代码 **********/
                end else begin
                        /********** 需要补充的代码 **********/
                end
            end
```

完成补全后, 你的代码应该如下所示:

```verilog
case(state)
            IDLE: begin
                done <= 1'b0;
                if(start) begin
                    reg_a <= a;
                    reg_b <= b;
                    state <= CALC;
                end
            end
            
            CALC: begin
                if(reg_b == 32'd0) begin
                    result <= reg_a;
                    done <= 1'b1;
                    state <= DONE;
                end else if(reg_a > reg_b) begin
                    reg_a <= reg_a - reg_b;
                end else begin
                    reg_b <= reg_b - reg_a;
                end
            end
```


### AHB-Lite 总线接口

接下来我们需要给 GCD 核心加上 AHB-Lite 总线接口, 让 CPU 能够访问它。

<!-- -->
> #### note::AHB-Lite 接口的作用
> AHB-Lite 接口是 GCD 加速器与 CPU 之间的桥梁。CPU 通过总线向 GCD 模块写入操作数, 并读取计算结果和状态。接口需要实现地址译码、读写控制等功能。

#### 读操作的地址译码

在 AHB-Lite 总线中, CPU 读取外设数据时, 需要根据地址选择返回对应寄存器的值。在 AHBlite_GCD.v 中，以下代码实现了读操作的地址译码逻辑:

```verilog
// Read operations - use latched address
reg [31:0] hrdata_reg;
always@(*) begin
    case(addr_reg)
                        /********** 需要补充的代码 **********/
    endcase
end
```

<!-- -->
> #### hint::补全提示
> `addr_reg` 是锁存的地址低位 (通常取 `HADDR[3:2]`), 用于区分 4 个寄存器:
> - `2'b00` (偏移 0x00): DATA_A 寄存器
> - `2'b01` (偏移 0x04): DATA_B 寄存器  
> - `2'b10` (偏移 0x08): 控制/启动寄存器
> - `2'b11` (偏移 0x0C): 状态和结果寄存器

补全数据读取阶段的代码, 完成补全后, 你的代码应该如下所示:

```verilog
// Read operations - use latched address
reg [31:0] hrdata_reg;
always@(*) begin
    case(addr_reg)
        2'b00: hrdata_reg = reg_a;
        2'b01: hrdata_reg = reg_b;
        2'b10: hrdata_reg = {31'b0, start};
        2'b11: hrdata_reg = {gcd_done, gcd_result[30:0]};  // Done in bit[31], result in bit[30:0]
        default: hrdata_reg = 32'h00000000;
    endcase
end
```

<!-- -->
> #### fread::代码功能说明
> | 地址 | 表达式 | 功能说明 |
> |------|--------|----------|
> | `2'b00` | `hrdata_reg = reg_a` | 返回操作数 A 的值, CPU 可以回读之前写入的数据 |
> | `2'b01` | `hrdata_reg = reg_b` | 返回操作数 B 的值, CPU 可以回读之前写入的数据 |
> | `2'b10` | `hrdata_reg = {31'b0, start}` | 返回启动信号状态, bit[0] 表示是否正在启动计算 |
> | `2'b11` | `hrdata_reg = {gcd_done, gcd_result[30:0]}` | 关键寄存器: bit[31] 为完成标志 `gcd_done`, bit[30:0] 为计算结果 |
> | `default` | `hrdata_reg = 32'h00000000` | 非法地址返回 0, 保证总线不会读到未知值 |

<!-- -->
> #### note::状态寄存器的设计巧思
> 注意 `2'b11` 地址的设计: 将 `gcd_done` 放在最高位 (bit[31]), 结果放在低 31 位。这样 CPU 可以通过一次读操作同时获取完成状态和结果:
> ```c
> uint32_t status = GCD->STATUS;
> if (status & 0x80000000) {          // 检查 bit[31] 是否为 1
>     uint32_t result = status & 0x7FFFFFFF;  // 取低 31 位作为结果
> }
> ```

<!-- -->
> #### hint::思考题
> 1. 如果输入的两个数中有一个是 0, 会发生什么? 如何改进?
> 2. 辗转相减法效率不高 (想想 GCD(1000000, 1)), 如果改用辗转相除法, 硬件该怎么设计?
> 3. 能否让 GCD 加速器支持中断, 计算完成后通知 CPU?
