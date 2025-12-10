# 微处理器系统结构与嵌入式系统设计 课程实验 2025秋

## 实验前阅读

> #### important::必读通知
>
> - 2025/10/26: 本指导书正式发布.
> 
> - 2025/10/26: 更新 <font color="red">LAB0</font>, 其余实验将在后续发布.
> 
> - 2025/11/01: 更新 <font color="red">LAB1</font>.
> 
> - 2025/11/05: 本周进行 LAB1 实验, 请大家仔细阅读本页面, 获取 LAB1 的配套代码以及实验报告模板. 实验报告截止日期为<font color="red"> 11 月 19 日中午 12 点</font>, 请大家在这个日期之前提交实验报告到对应助教的邮箱中, 并完成实验报告上要求填写的内容, 过期提交或未交将会扣除相应的实验平时分. 实验报告和邮件主题都请按照以下方法命名后提交: <font color="red">学号-姓名-第几次实验</font>.
> 
> - 2025/11/05: 请同学们将 LAB1.1 中汇编程序的运行过程录屏, 时长限制在 30s 内, 然后在<font color="red"> 11 月 11 日 23:59 分前</font>将录屏提交到对应助教的邮箱中, 命名格式为: <font color="red">学号-姓名-LAB1.1</font>.
> 
> - 2025/11/07: 更新<font color="red">代码仓库备用链接</font>, 之前链接无法进入的同学可以点击[此处](https://gitee.com/fmq03/CortexM0_SoC/tree/main/code25)获取代码和实验报告模板.
> 
> - 2025/11/15: 更新 <font color="red">LAB2</font>.
>
> - 2025/11/19: 本周进行 LAB2 实验, 请大家及时更新自己的[代码仓库](https://gitee.com/fmq03/CortexM0_SoC/tree/main/code25), 实验报告截止日期为<font color="red"> 11 月 26 日中午 12 点</font>.
>
> - 2025/11/26: 更新 <font color="red">LAB3</font>. 本周进行 LAB3 实验, 实验报告截止日期为<font color="red"> 12 月 10 日中午 12 点</font>.
> 
> - 2025/12/07: 更新 <font color="red">LAB4</font>.

<!-- -->

> #### fread::关于本指导书的配套代码
> 点击[此处](https://gitee.com/fmq03/CortexM0_SoC/tree/main/code25)或[此处](https://github.com/UESTC-ICSDLab/CortexM0_SoC/tree/main/code25)获取本指导书的配套代码, 文件夹名称为 CortexM0_SoC/code25, <font color="red">请务必将该文件夹放在英文路径下</font>.
> 本指导书中实验与代码文件的对应关系:

> | 实验名称 | 对应代码文件夹 |
> | ---- | ---- |
> | LAB0 - 工欲善其事 必先利其器 | Task0 |
> | LAB1 - "施法"让 CPU 动起来 | Task1 |
> | LAB2 - "点石成金" - 实现你的首个 SoC | Task2 |
> | LAB3 - "灯, 等灯, 等灯"-流水灯的几种点法 | Task3 |
> | LAB4 - 如何召唤"沉睡的软件" | Task4 |


<!-- -->
> #### question::如何提交实验报告
>
> 请大家按每一次实验的要求, 在规定时间内, 将符合规范的实验报告提交至自己所在班级助教的邮箱.
>
> | 授课教师 | 助教 | 实验报告提交邮箱 |
> |  :-:  | :-:  | :-:  |
> | 黄乐天  | 陈晨 | 1647242532@qq.com |
> | 杨成韬  | 陈飞扬 | 457063678@qq.com |
> | 万里冰  | 陈雨阳 | 212913904@qq.com |
> | 廖永波  | 曹欣雨 | 3143216985@qq.com |
> | 张驰  | 郝禹 | 3627923058@qq.com |

<!-- -->
> #### hint::如果你遇到问题
>
> - 请先阅读[常见问题](faq/introduction.md).
> - 阅读官方文档.
> - 尝试使用搜索引擎(只推荐 [Google](https://google.com))寻找解决方案.
> - 如果你做了以上尝试, 却依然无法解决问题, 请向助教或老师提问. 在此之前请确保你了解[提问的智慧](https://github.com/ryanhanwu/How-To-Ask-Questions-The-Smart-Way/blob/main/README-zh_CN.md).

<!-- -->
> #### fread::外部资源
> 
> - [B 站录播](https://www.bilibili.com/video/BV1Wf4y1W7gd?spm_id_from=333.999.0.0)
> - [知乎专栏](https://www.zhihu.com/column/conquest-on-chip)(如果不熟悉 Verilog 和 FPGA)
>

