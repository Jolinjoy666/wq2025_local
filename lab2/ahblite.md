# AHB-Lite 总线介绍

Cortex-M0 处理器中所使用的是 AHB-Lite 总线，其结构示意图如下图所示，其包括一个主机（Master）、若干个从机（Slave），一个译码器（Decoder）用于选择对应的从机以及一个选通开关（MUX）用于选择对应的返回数据。这种总线结构极为简化，因只支持单一的主机而简化掉了总线仲裁等一系列复杂的机制，总线的传输机制也有较大的简化。因此被冠以 Lite 的后缀。

<center><img src="/img/lab2/53.png" alt="AHB-Lite总线结构" style="zoom:80%;" /></center><center style="color:#0";>AHB-Lite 总线示意图</center> 

## AHB-Lite 总线的细节

### 部分接口介绍

除了上图中的信号外，AHB-Lite 还有部分重要的接口信号。这些接口对于设计 SoC 的总线协议非常重要，下面我们简要介绍 Cortex-M0 处理器中的总线接口信号：

| 名称 | 来源 | 描述 |
| ---- | ---- | ---- |
| HADDR[31:0] | Master | 传输地址 |
| HBURST[2:0] | Master | Burst 类型 |
| HSIZE[2:0] | Master | 数据宽度<br>00：8bit Byte<br>01：16bit Halfword<br>10：32bit Word |
| HTRANS[1:0] | Master | 传输类型<br>00：IDLE，无操作<br>01：BUSY<br>10：NONSEQ，主要的传输方式<br>11：SEQ |
| HWDATA[31:0] | Master | 核发出的写数据 |
| HWRITE | Master | 读写选择（1：写，0：读） |
| HRDATA[31:0] | Slave | 外设返回的读数据 |
| HREADOUT | Slave | 何时传输完成（通常为 1） |
| HRESP | Slave | 传输是否成功（通常为 0） |

### Cortex-M0 支持的总线传输

通过分析 Cortex-M0 IP 核中 HBURST 和 HTRANS 的信号的赋值可以得出其支持的传输模式：

```verilog
assign HTRANS[0] = 1'b0;
assign HBURST[2] = 1'b0;
assign HBURST[1] = 1'b0;
assign HBURST[0] = 1'b0;
```

可以看到处理器核端的 HTRANS 最低位恒为 0，因此说明 M0 仅支持 NONSEQ 传输，并且 HBURST 也恒为 0，因此 M0 不支持 BURST 传输。在后面的外设总线接口设计中，只需要满足 NONSEQ 类型传输即可，为后续简化设计提供了条件。

### AHB-Lite 总线的传输时序

在《AMBA® 3 AHB-Lite Protocol》文档中详细介绍了 AHB-Lite 总线的信号描述、传输类型与时序。AHB-Lite 总线除了支持基本的读写操作外，还支持具有等待状态的读写操作、锁定读写操作和突发读写操作。由于 Cortex-M0 处理器核的特性，其 AHB-Lite 总线仅支持 NONSEQ 传输，即基本的读写操作和具有等待状态的读写操作。如果要了解更多关于 AHB-Lite 的知识，可以去详细阅读《AMBA® 3 AHB-Lite Protocol》文档。

#### 基本读操作

<center><img src="/img/lab2/48.png" alt="基本读操作时序" style="zoom:80%;" /></center><center style="color:#0";>基本读操作</center> 

当 Master 需要从外设读取数据时，总共需要经历两个阶段：Address phase & Data phase，因此一次读传输至少需要 2 cycle。在 Address phase 时，Master 会把读取地址输出在地址总线上，直到 HREADY 为 '1'。由于 HREADY 一直为 '1'，那么 Master 在 Address phase 放出地址后直接进入 Data phase；在 Data phase 时，Master 会在 HREADY 为 '1' 时读取数据总线 HRDATA 上的数据，至此传输完成。

#### 基本写操作

<center><img src="/img/lab2/49.png" alt="基本写操作时序" style="zoom:80%;" /></center><center style="color:#0";>基本写操作</center> 

类似基本读操作，写操作也会经历两个阶段：在 Address phase 时，Master 会把写地址输出在地址总线上，直到 HREADY 为 '1'。由于 HREADY 一直为 '1'，那么 Master 在 Address phase 放出地址后直接进入 Data phase；在 Data phase 时，Master 会将写数据放在数据总线 HWDATA 上，直到 HREADY 为 '1'，传输完成。

#### 具有等待状态的读写操作

<center><img src="/img/lab2/50.png" alt="具有等待状态的读操作" style="zoom:80%;" /></center><center style="color:#0";>具有等待状态的读操作</center> 

<center><img src="/img/lab2/51.png" alt="具有等待状态的写操作" style="zoom:80%;" /></center><center style="color:#0";>具有等待状态的写操作</center> 

HREADY 为当前正在进行传输的 Slave 返回的 HREADYOUT，Master 端会把 HREADY 既作为进入传输的判断条件（在 HREADY 为 '0' 时不会开始下一个传输），也会作为传输完成的条件（在 HREADY 为 '0' 时不会退出当前传输）。

总线是流水线结构，虽然对于一次传输至少需要两个 cycle，但是对于两次传输，例如把上图中的 A 与 B 看作两次传输，图中的 Address phase 为写传输 A 的地址阶段，图中的 Data phase 为 A 的数据阶段，但也同时作为读传输 B 的地址阶段。B 地址对应的外设在 Data phase 的第一个周期时，由于 HREADY 为 0，并不能进入传输，而在第二个周期时，对于 B 而言与图中 A 的 Address phase 无异，最终实现如下图所示的流水线传输。

<center><img src="/img/lab2/52.png" alt="总线流水操作" style="zoom:80%;" /></center><center style="color:#0";>总线流水操作</center> 

对于 Slave 而言，HREADY 只需要作为进入传输的判断条件，因为进入传输后，HREADY 就会被切换到自己的输出 HREADOUT 上，因此当 Slave 根据 HREADY 等信号进入传输状态后，自行控制传输结束的时间，并依此控制 HREADYOUT 输出。

在后面 SoC 具体设计中，所有的 Slave 接口都能在两个周期内完成读写（即判断出 Address phase 后能在下一个周期完成数据读写），因此所有外设的 HREADYOUT 信号都被置为常量 '1'，这也是简化设计的一个体现。

## 本实验需要完成的SoC中总线介绍

<center><img src="/img/lab2/54.png" alt="SoC系统结构" style="zoom:80%;" /></center><center style="color:#0";>本实验SoC系统结构</center> 

<!-- -->
> #### fread::我需要做什么 ?
> 在本实验中，你需要基于 AHB-Lite 总线协议完成 SoC 系统的设计。总线互连模块将连接 Cortex-M0 处理器核与各个外设，实现数据的读写传输。请结合上述 AHB-Lite 总线的介绍，完成相应的总线接口设计。

<!-- -->
> #### important::注意事项
> 在设计过程中，请特别注意：
> 1. Cortex-M0 仅支持 NONSEQ 传输模式，不支持 BURST 传输
> 2. 所有外设的 HREADYOUT 信号可以设置为常量 '1'
> 3. 地址译码要根据各外设的地址范围进行正确配置
> 4. 注意 AHB 信号的方向性，Master 与 Slave 的信号来源不同
