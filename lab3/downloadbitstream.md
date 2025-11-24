# 下载比特流到 FPGA

上一实验中, 我们通过仿真验证了 SoC 的功能, 接下来我们将系统实现在 FPGA 上, 进行上板调试.

打开 TD 软件(推荐使用 5.6.2 版本), 点击 Project--New Project, 创建一个新的工程, 给工程命名为 "CortexM0_SoC", 工程建立在 "/Task3/Count/TD/" 文件夹下. 记得检查一下下方显示的工程路径是否合理, <font color="red">注意不能出现中文字体</font>. 选择 FPGA 芯片型号, 在 Device Family 栏处选择 EG4, 然后在下方列表中选择 EG4S20BG256.

<center><img src="/img/lab2/pics/1.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>创建文件与型号选择</center> 

<center><img src="/img/lab3/16.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>创建文件与型号选择</center> 

<center><img src="/img/lab3/17.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>创建文件与型号选择</center> 

进入源文件添加界面, 因为我们的设计源文件都在 "/Task3/Count/rtl" 下, 所以这里右键点击 Hierarchy, 选择 ADD Sources 直接将 "/Task3/Count/rtl" 目录添加进来. 点击 OK 即可添加成功.

<center><img src="/img/lab2/pics/4.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>添加源文件</center> 

<center><img src="/img/lab3/18.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>添加源文件</center> 

右键 constraint_1（active）, 选择 Add ADC File 进入约束文件添加界面, 我们已经在 "/Task3/Count/TD/" 中为你准备好了约束文件 "pin.adc", 将这个文件添加进来即可.

<center><img src="/img/lab2/pics/6.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>添加约束文件</center> 

<center><img src="/img/lab3/19.png" alt="20" style="zoom:60%;" /></center><center style="color:#0";>添加约束文件</center> 

<!-- -->

> #### question::什么是约束文件?
>
> 简单来说, 约束文件是逻辑设计与物理芯片之间的"桥梁". 在 FPGA 设计中, 约束文件主要分为两大类:
>
> 1. 物理约束 (Physical Constraints): 主要用于指定逻辑端口对应的物理引脚位置 (Pin Assignment) 及电气标准. 主要告诉软件"代码里的信号连到芯片的哪根管脚上". 在本实验使用的 TD 软件中, 这类约束保存在 .adc 文件中.
> 2. 时序约束 (Timing Constraints): 用于设定时钟频率, 建立/保持时间等时序要求, 主要告诉软件"芯片需要跑多快", 指导布局布线以满足性能指标. 业界通用的格式通常为 .sdc 文件.
>
> 在本实验中, 我们主要关注物理约束 (.adc), 即将设计的外部端口连接到 FPGA 具体的物理引脚上.


<!-- -->
> #### hint::本实验中的约束文件
>
> 打开 "/Task3/TD/pin.adc":
> ```
> set_pin_assignment	{ RSTn }	{ LOCATION = E3; IOSTANDARD = LVCMOS25; PULLTYPE = PULLUP; }
> set_pin_assignment	{ SWCLK }	{ LOCATION = K3; IOSTANDARD = LVCMOS25; PULLTYPE = PULLUP; }
> set_pin_assignment	{ SWDIO }	{ LOCATION = K6; IOSTANDARD = LVCMOS25; DRIVESTRENGTH = 8; PULLTYPE = PULLUP; }
> set_pin_assignment	{ clk }	{ LOCATION = R7; IOSTANDARD = LVCMOS25; PULLTYPE = PULLUP; }
> ```
> 上述代码中, clk 管脚约束到 FPGA 开发板上的 50MHZ 晶振的引脚; RSTn 管脚约束到 FPGA 上的复位按钮.



完成 TD 工程的创建. 在进入 TD 主界面后, 可以在 Source 窗口的 Hierarchy 标签页中看到我们的设计文件, 仿真文件和约束文件. 若要对某个文件临时进行编辑, 以约束文件为例, 我们可以双击 "pin.adc", 该文件的内容会被显示到右侧的编辑窗口中.



<!-- -->
> #### fread::Hierarchy
> + Hierarchy 标签页是对设计的层次化展示, 便于我们直观理解多个模块之间的调用关系. TD 会自动选择设计的顶层文件, 并标上<img src="/img/lab2/pics/8.png" alt="30" style="zoom:100%;" />图标. 你也可以通过右键文件选择 Set as Top 将任意文件设为顶层文件. <font color="red">注意, RTL 分析, 综合和实现的对象都是顶层文件.</font>

<center><img src="/img/lab2/pics/9.png" alt="25" style="zoom:100%;" /></center><center style="color:#0";>TD 工程主界面</center>

然后点击上侧导航栏的 Run, 这时 TD 将会依次执行 RTL 分析 (RTL Analysis), 综合 (Synthesis), 实现 (Implementation) 和 生成比特流文件 (Generate Bitstream) 四个步骤. 最终生成的比特流文件用于 FPGA 的配置.

等待 TD 运行一段时间后, 可以看到弹出的提示窗口, 表明比特流文件生成成功, 关闭该窗口. 

<!-- -->
> #### hint::出错了？
> 初次综合生成可能会有提示：
> `"ERROR:Cannot find license file .../Anlogic/TD5.6.2/license/Anlogic.lic"`
> 这是因为 TD 软件没有找到 license 文件, 相当于软件没有激活, 把提供的 Anlogic.lic 放到它提示的文件夹(你的 TD 安装目录下的 license 文件夹里面即可), 之后再次综合生成应该就正常了. Anlogic.lic 位于 Task3 文件夹中.

<center><img src="/img/lab2/pics/10.png" alt="31" style="zoom:73%;" /></center><center style="color:#0";>比特流生成成功提示窗口</center>

点击上侧导航栏中 Tools 下的 Schematic Viewer--Read Design Schematic 选项, 可以看到综合和布局布线得到的原理图. 

<center><img src="/img/lab2/pics/11.png" alt="32" style="zoom:73%;" /></center><center style="color:#0";>进入 Schematic 界面</center>

<center><img src="/img/lab2/pics/12.png" alt="32" style="zoom:73%;" /></center><center style="color:#0";>进入 Schematic 界面</center>

点击 Chip Viewer 标签, 可以看到设计对 FPGA 内部资源的占用情况.

<center><img src="/img/lab2/pics/13.png" alt="27" style="zoom:100%;" /></center><center style="color:#0";>Device 界面</center>

接下来进行板卡的连接. 使用两条数据线将 FPGA 板卡上的两个 Micro-USB 接口与 PC 上的两个 USB 接口连接起来.

<center><img src="/img/lab2/pics/14.png" alt="28" style="zoom:63%;" /></center><center style="color:#0";>板卡连接</center>

<!-- -->

> #### important::为什么用两个接口?
+ 标有 JTAG 的接口是 FPGA 的下载配置接口, 用于比特流文件或其他配置文件的下载.
+ 标有 DAP-Link 的接口是 CMSIS 调试接口, 用于 FPGA 中实现的 CortexM0 的调试. 该接口与板卡上的 CMSIS-DAP 调试器连接, 该调试器的另一端连接在 FPGA 的引脚上, 这些引脚通过约束文件与 CortexM0 的 SWDIO 和 SWCLK 这两个调试信号连接. 

确保连线无误后, 双击左侧栏中的 Download, 打开下载比特流的 Download 窗口, 这里软件一般会默认指定刚生成的比特流文件路径. 但这里仍要格外注意, 因为当你在多个工程间来回穿梭时, TD 可能会突然抽风, 给你默认指定一个其他工程中的比特流文件, 如果你没有注意而将它下载了进去, 那么接下来的调试工作可能会让你心态爆炸. 如果 Download 窗口中没有比特流文件, 选择左侧导航栏中的 ADD 按钮, 选择你刚刚创建的 TD 工程文件夹里面的 ***_RUNS 文件, 选择 phy_1 文件夹, 选择刚刚生成的 bit 文件. 

<center><img src="/img/lab2/pics/15.png" alt="29" style="zoom:70%;" /></center><center style="color:#0";>双击打开 Download 窗口</center>

<center><img src="/img/lab2/pics/16.png" alt="29" style="zoom:70%;" /></center><center style="color:#0";>Bit文件选择</center>

<center><img src="/img/lab3/20.png" alt="29" style="zoom:70%;" /></center><center style="color:#0";>Bit文件选择</center>

<center><img src="/img/lab3/21.png" alt="36" style="zoom:70%;" /></center><center style="color:#0";>每次下载前记得检查比特流文件路径是否有误</center>

<!-- -->

选择好比特流文件的路径之后, 点击 Program 开始下载比特流, 等待进度条加载完毕, 便可开始下一步的调试工作了.

> #### important::没有识别到开发板?
+ 如果在 Bit 文件选择的时候, 发现 Download 窗口上方的下拉栏中是空的, 没有识别到 FPGA 开发板, 请参考[JTAG驱动安装](#jtag驱动安装).

<!-- -->
> #### hint::TD 的工程目录结构
> 为了更好地帮助你找到工程中的文件(如日志文件,比特流文件等), 这里有必要简单介绍一下 TD 默认的工程目录结构.
>
> + ***_Runs: 存放综合实现的中间文件和结果文件
> + + .logs:日志文件
> + + syn_1: 与整个设计有关的综合 (Synthesis) 文件
> + + phy_1: 与整个设计有关的实现 (Implementation) 文件, 包括比特流文件和 bin 文件等


<!-- -->

> #### question::思考
> 我们知道, 下载比特流文件是将硬件电路配置到 FPGA 中, 那么我们的软件程序是如何被下载到 FPGA 中的呢? 
>
> 还记得我们使用 keil 生成的 "code.hex" 文件对 RAM 进行了初始化. 所以在生成比特流文件时, RAM 的初始数据, 即程序的指令代码也就成为了比特流文件的一部分, 被下载到了 FPGA 中.

<!-- -->

### JTAG驱动安装

初次下载比特流到 FPGA 上的时候, 有可能会发现没有找到 FPGA 开发板, 原因是没有安装驱动. 在 windows 的设备管理器 (win8 以上系统对着 WIN 徽标右键即可找到并打开) 的串行总线设备中, 你可能会发现有有一个设备是这样 (如果通用串行总线设备中没有就找找其它的列表里) ：

<center><img src="/img/lab2/pics/21.png" alt="29" style="zoom:70%;" /></center><center style="color:#0";>未安装驱动的设备名称</center>

这个时候, 你需要右键：更新驱动程序-浏览我的电脑以查找驱动程序.

选择 Anlogic TD 的安装目录下的 driver 文件夹`".../Anlogic/TD5.6.2/driver/al-link"`, 勾选包括子文件夹, 选择下一步, 之后它会自动搜索并驱动. 

<center><img src="/img/lab3/27.png" alt="29" style="zoom:70%;" /></center><center style="color:#0";>手动浏览驱动</center>

<center><img src="/img/lab3/28.png" alt="29" style="zoom:55%;" /></center><center style="color:#0";>选择TD安装文件夹下的driver文件夹</center>


安装完成后, 然后你会发现设备名称改变了: 

<center><img src="/img/lab2/pics/23.png" alt="29" style="zoom:70%;" /></center><center style="color:#0";>安装完驱动的设备名称</center>

这个时候, 在 TD 里面重新打开 Download 窗口, 应该能找到设备了. 

> #### question::驱动并没有正确安装 ?
>
> 在很多情况下, 电脑装好驱动后设备上会有一个黄色叹号, 打开设备属性会有下面的情况(代码48或者代码52). 
>
> <center><img src="/img/lab3/30.png" alt="29" style="zoom:70%;" /></center>
> <center><img src="/img/lab3/29.png" alt="29" style="zoom:70%;" /></center><center style="color:#0";>报错现象</center>
> 
> 请你先**卸载原有驱动**(设备管理器对设备右键 - 卸载设备 - 勾选尝试删除此设备的驱动程序, 重新插拔设备完成驱动卸载), 尝试按照下面的方法逐一排查问题:
>
> 1. 请在设备管理器中安装驱动的时候, 关闭**杀毒/安全软件**, 目前已有因为某想电脑管家的导致驱动安装不正常的案例, 如果你是**联想/拯救者笔记本**, 请务必注意这一点. 关掉杀毒软件后, 重新安装驱动试一试. 
> 2. 在安装驱动,手动选择文件夹的时候,选择到 al-link - win10 - x86/x64 这一级的文件夹(不知道是x86还是x64? 电脑内存大于4GB就一定是x64), 再重新安装驱动试一试. 如果是win7系统, 则选择win7.
> 3. 对于代码52, 应该是需要关闭强制数字签名. 可以临时关闭或永久关闭(参考:[该博客](https://blog.iyatt.com/?p=18677)). 对于代码48, 此操作有效性未知, 请优先排查1和2的操作是否可行, 再尝试本操作.