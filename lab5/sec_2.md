# 4x4 矩阵键盘接口设计

#### 学习目标
- 了解矩阵键盘的扫描原理与消抖机制。
- 熟悉 AHB 从设备包装器与核心模块的分工。
- 完成键盘硬件逻辑中“留白”部分的练习，培养阅读与补全 RTL 代码的能力。

<!-- -->
> #### flag::矩阵键盘扫描
> + 在 lab4 中, 我们用 4 个按键分别对应 4 个中断. 但如果需要更多按键呢? 16 个按键就需要 16 个中断? 那 CPU 的中断资源很快就不够用了!
> + 本实验我们设计一个矩阵键盘控制器, 16 个按键只占用 1 个中断, CPU 通过总线读取按键值.

## 矩阵键盘原理

### 为什么叫"矩阵"键盘?

看下面这张图就明白了:

```
        COL[3]  COL[2]  COL[1]  COL[0]
           │       │       │       │
ROW[3] ────┼───────┼───────┼───────┼────
           │       │       │       │
         [F]     [E]     [D]     [C]
           │       │       │       │
ROW[2] ────┼───────┼───────┼───────┼────
           │       │       │       │
         [B]     [A]     [9]     [8]
           │       │       │       │
ROW[1] ────┼───────┼───────┼───────┼────
           │       │       │       │
         [7]     [6]     [5]     [4]
           │       │       │       │
ROW[0] ────┼───────┼───────┼───────┼────
           │       │       │       │
         [3]     [2]     [1]     [0]
```

16 个按键排成 4 行 4 列, 只需要 8 根线 (4 行 + 4 列) 就能检测所有按键!

### 扫描原理

怎么知道哪个键被按下了呢? 我们用"逐行扫描"的方法:

1. 让 ROW[0] = 0, 其他行 = 1
2. 读取 COL[3:0], 如果某列为 0, 说明第 0 行该列的键被按下
3. 让 ROW[1] = 0, 其他行 = 1
4. 读取 COL[3:0], 如果某列为 0, 说明第 1 行该列的键被按下
5. 以此类推...

### 按键消抖

<!-- -->
> #### note::为什么需要消抖?
> 机械按键在按下和释放的瞬间会产生抖动, 就像弹簧一样来回弹跳几次才稳定. 这个抖动时间大约 5-20ms. 如果不处理, 按一次键可能会被误认为按了好几次!

消抖的方法很简单: 连续检测到相同状态 N 次后, 才认为状态真的改变了.


## 硬件模块设计

<!-- -->
> #### note::矩阵键盘硬件代码组成
> `AHBlite_Keyboard.v`
> - **功能**: 键盘模块的 AHB 接口包装器，负责与总线交互并提供寄存器映射。
>
> `Keyboard.v`
> - **功能**: 键盘控制核心模块。
> - **子模块**:
>   - `keyboard_scan.v`: 扫描 4x4 矩阵键盘，产生行扫描信号并读取列信号。
>   - `keyboard_filter.v`: 对按键信号进行消抖处理。
>   - `keyboard_reg.v`: 锁存按键状态，处理清除信号。

<!-- -->
> #### important::必读通知
> 在矩阵键盘设计部分，会沿用前四个实验中掌握的基础能力，希望同学们可以继续动手补全以下三个 Verilog 文件：
>
> 1. **`Keyboard.v`**
>    - **任务**: 实例化子模块。
>    - **说明**: 需要在模块内部正确实例化 `keyboard_scan` 和 `keyboard_filter` 模块，并连接相应的端口（如时钟、复位、行列信号及中间连线）。
>
> 2. **`keyboard_scan.v`**
>    - **任务**: 实现扫描时钟生成逻辑。
>    - **说明**: 需要定义计数器和时钟寄存器，编写分频逻辑以产生用于键盘扫描的低频时钟（例如 10kHz）。
>
> 3. **`keyboard_filter.v`**
>    - **任务**: 实现按键消抖脉冲生成逻辑。
>    - **说明**: 需要编写逻辑，当消抖计数器从非满状态变为满状态时，产生一个单周期的脉冲信号 `key_pulse`，用于确认按键按下。

## 软件部分

### 修改启动代码

我们需要修改 `startup_CMSDK_CM0.s` 启动文件，添加按键中断的支持。在本实验中，矩阵键盘模块通过产生中断信号来通知 CPU 读取按键值，因此必须在中断向量表中注册键盘相关的 IRQ 入口。

<!-- -->
> #### note::中断向量表的作用
> 中断向量表位于内存起始地址 (0x00000000)，存放了各个中断/异常的处理函数入口地址。当中断发生时，CPU 会根据中断号查找向量表，跳转到对应的处理函数执行。

#### 第一步：修改中断向量表

打开 `/keil/startup_CMSDK_CM0.s`，找到 `__Vectors` 段，将原来的内容:

```armasm
; filepath: startup_CMSDK_CM0.s (修改前)
__Vectors       DCD     __initial_sp              ; Top of Stack
                DCD     Reset_Handler             ; Reset Handler
                DCD     0                         ; NMI Handler
                DCD     0                         ; Hard Fault Handler
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; SVCall Handler
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; PendSV Handler
                DCD     0                         ; SysTick Handler
                DCD     0                         ; IRQ0 Handler
                DCD     0                         ; IRQ1 Handler
                DCD     0                         ; IRQ2 Handler
                DCD     0                         ; IRQ3 Handler
```

修改为:

```armasm
; filepath: startup_CMSDK_CM0.s (修改后)
__Vectors       DCD     __initial_sp              ; Top of Stack
                DCD     Reset_Handler             ; Reset Handler
                DCD     0                         ; NMI Handler
                DCD     0                         ; Hard Fault Handler
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; SVCall Handler
                DCD     0                         ; Reserved
                DCD     0                         ; Reserved
                DCD     0                         ; PendSV Handler
                DCD     0                         ; SysTick Handler
                DCD     KEY0_Handler              ; IRQ0 Handler
                DCD     KEY1_Handler              ; IRQ1 Handler
                DCD     KEY2_Handler              ; IRQ2 Handler
                DCD     KEY3_Handler              ; IRQ3 Handler
```

<!-- -->
> #### hint::修改说明
> 我们将 IRQ0-IRQ3 的处理函数从 `0` (空处理) 改为 `KEY0_Handler` 到 `KEY3_Handler`。这样当矩阵键盘触发中断时，CPU 会跳转到对应的按键处理函数。

#### 第二步：添加中断服务函数入口

在启动文件中找到注释 `; add IRQ Handler function here`，添加以下中断服务函数的汇编入口:

```armasm
; filepath: startup_CMSDK_CM0.s
; add IRQ Handler function here

KEY0_Handler    PROC
                  EXPORT KEY0_Handler            [WEAK]
                BX      LR
                  ENDP

KEY1_Handler    PROC
                  EXPORT KEY1_Handler            [WEAK]
                BX      LR
                  ENDP

KEY2_Handler    PROC
                  EXPORT KEY2_Handler            [WEAK]
                BX      LR
                  ENDP

KEY3_Handler    PROC
                  EXPORT KEY3_Handler            [WEAK]
                BX      LR
                  ENDP
```
<!-- -->
> #### note::为什么使用 [WEAK] 标记？
> 使用 `[WEAK]` 的目的是提供一个**默认的空实现**。如果用户在 C 代码中定义了同名函数（如 `void KEY0_Handler(void)`），链接器会使用 C 代码中的强定义，覆盖这里的弱定义。这样的设计使得：
> - 即使用户没有实现中断处理函数，程序也不会链接失败
> - 用户可以方便地在 C 代码中重写中断处理逻辑

<!-- -->
> #### hint::当前实现的局限性
> 上述代码中，每个 Handler 只有一条 `BX LR` 指令，表示**直接返回，不做任何处理**。如果需要在中断中调用 C 函数，需要修改为如下形式：
> ```armasm
> KEY0_Handler    PROC
>                 EXPORT KEY0_Handler            [WEAK]
>                 IMPORT KEY0                    ; 导入 C 函数
>                 PUSH    {R0,R1,R2,LR}          ; 保存寄存器
>                 BL      KEY0                   ; 调用 C 函数
>                 POP     {R0,R1,R2,PC}          ; 恢复寄存器并返回
>                 ENDP
> ```
> 这样可以在 C 代码中实现具体的中断处理逻辑。

<!-- -->
> #### hint::思考题
> 1. 如果两个键同时按下会怎样? 这叫"按键冲突", 如何检测和处理?
> 2. 消抖时间设多少合适? 太短会怎样? 太长又会怎样?
> 3. 如何实现"长按"检测功能?
