# 6 串口通信

 [toc]

注：笔记主要参考B站江科大自化协教学视频“[51单片机入门教程-2020版 程序全程纯手打 从零开始入门](https://www.bilibili.com/video/BV1Mb411e7re?spm_id_from=333.1007.top_right_bar_window_custom_collection.content.click)”。
注：工程及代码文件放在了本人的[Github仓库](https://github.com/jjejdhhd/Learn_STC89C52)。
***

## 6.1 串口通信原理
串口是一种应用十分广泛的通讯接口，串口成本低、容易使用、通信线路简单，可实现两个设备的互相通信。单片机的串口可以使单片机与单片机、单片机与电脑、单片机与各式各样的模块互相通信，极大的扩展了单片机的应用范围，增强了单片机系统的硬件实力。**51单片机内部自带UART**（Universal Asynchronous Receiver Transmitter，通用异步收发器），可实现单片机的串口通信。

 <div align=center>
<img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/%E4%B8%B2%E5%8F%A3%E9%80%9A%E4%BF%A1%E6%8E%A5%E5%8F%A3%E7%A4%BA%E6%84%8F%E5%9B%BE.png" width=60%>
</div><div align=center>
图6-1 串口通信接口示意图
</div>

上图所示的简单双向串口通信，有两根交叉连接的通信线（发送端TXD和接收端RXD）。当只需单向的数据传输时，可以只接一根通信线。当**电平标准**不一致时，需要加电平转换芯片。电平标准是数据1和数据0的表达方式，是传输线缆中人为规定的电压与数据的对应关系，串口常用的电平标准有如下三种：
> 1. TTL电平：+5V表示1，0V表示0。
> 2. RS232电平：(-3V,-15V)表示1，(+3V,+15V)表示0。
> 3. RS485电平：两线压差(+2V,+6V)表示1，(-2V,-6V)表示0（差分信号）。

 <div align=center>
<img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/DB9%E6%8E%A5%E5%8F%A3%E7%A4%BA%E6%84%8F%E5%9B%BE1.png" width=60%>
</div><div align=center>
图6-2 DB9接口示意图
</div>

 <div align=center>
<img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/%E7%8E%B0%E4%BB%A3%E4%B8%B2%E5%8F%A3%E5%BA%94%E7%94%A8%E6%A1%88%E4%BE%8B.png" width=80%>
</div><div align=center>
图6-3 现代串口应用案例
</div>

图6-2是串口通信最经典的DB9接口（单片机不支持流控信号），使用RS232电平标准，但是由于其体积大、速率慢等原因，逐渐的被笔记本电脑领域所淘汰。而为了完成与电脑的串口通信，现代通常使用USB转串口的模块（如上图6-3左）。同时也可以看出，由于51单片机最常用的还是串口通信，所以为了配合51单片机，有许许多多的模块开始与串口通信结合起来。

<div align=center>
表6-1 常见通信接口比较
</div>
<table class="tg">
<thead>
  <tr>
    <th class="tg-7btt"><span style="font-weight:bold">名称</span></th>
    <th class="tg-7btt"><span style="font-weight:bold">引脚定义</span></th>
    <th class="tg-7btt"><span style="font-weight:bold">通信方式</span></th>
    <th class="tg-7btt"><span style="font-weight:bold">特点</span></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-9wq8">UART</td>
    <td class="tg-lboi">TXD：发送端<br>RXD：接收端<br>GND：公共(参考)地</td>
    <td class="tg-9wq8">全双工、异步</td>
    <td class="tg-9wq8">点对点通信</td>
  </tr>
  <tr>
    <td class="tg-9wq8">I²C</td>
    <td class="tg-lboi">SCL：同步时钟<br>SDA：数据输入/输出端</td>
    <td class="tg-9wq8">半双工、同步</td>
    <td class="tg-9wq8">可挂载多个设备</td>
  </tr>
  <tr>
    <td class="tg-9wq8">SPI</td>
    <td class="tg-lboi">SCLK：同步时钟<br>MISO：主机输入，从机输出<br>MOSI：主机输出，从机输入<br>CS：从设备使能信号，由主设备控制。</td>
    <td class="tg-9wq8">全双工、同步</td>
    <td class="tg-9wq8">可挂载多个设备</td>
  </tr>
  <tr>
    <td class="tg-9wq8">1-Wire</td>
    <td class="tg-lboi">DQ：发送/接收端</td>
    <td class="tg-9wq8">半双工、异步</td>
    <td class="tg-9wq8">可挂载多个设备</td>
  </tr>
  <tr>
    <td class="tg-0pky" colspan="4">此外还有CAN（差分传输）、USB等</td>
  </tr>
</tbody>
</table>

> 对于上表的注释：
> - 全双工：通信双方可以在同一时刻互相传输数据
> 半双工：通信双方可以互相传输数据，但必须分时复用一根数据线
> 单工：通信只能有一方发送到另一方，不能反向传输
> - 异步：通信双方各自约定通信速率
> 同步：通信双方靠一根时钟线来约定通信速率
> - 总线：连接各个设备的数据传输线路（类似于一条马路，把路边各住户连接起来，使住户可以相互交流）
> - TXD全称：transmit external data。

在开发板上，I²C用于与EEPROM（AIM23A10）通信，SPI改编后与RTC芯片（DS1302通信），1-Wire用于与温度传感器（DS18B20）通信。CAN总线常用语汽车控制领域，USB协议则在日常生活中最常见。STC89C52只有1个UART（如下图P3.0/P3.1），这个UART有四种工作模式：
> 模式0：同步移位寄存器
> 模式1：8位UART，波特率可变（常用）
> 模式2：9位UART，波特率固定
> 模式3：9位UART，波特率可变

<div align=center>
<img src="https://raw.githubusercontent.com/jjejdhhd/Git-Img/main/STC89C516-90C%E7%89%88%E6%9C%AC%E5%BC%95%E8%84%9A%E5%9B%BE.png" width="30%">

图6-4 STC89C52上的串口示意图
</div>

下面图6-5给出串口通信在单个通信线上的时序图，上面发送和接收各数据位的间隔时间就是 **波特率**，表明了串口通信的速率。可以看到主要包含 **4个部分**：
> 1. 起始位：拉低信号，表明开始传输8bit的数据。
> 2. 数据位：每次传输8bit的数据，注意先传输低位LSB。
> 2. 检验位（选填）：只出现在9位UART的最后一位，用于数据验证。
> 3. 停止位：拉高信号，用于数据帧间隔。
>
> 注意波特率只能为：4800、9600、115200等约定好的值。

<div align=center>
<img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/%E4%B8%B2%E5%8F%A3%E6%97%B6%E5%BA%8F%E5%9B%BE.png" width="70%">

图6-5 串口时序图
</div>

由于器件手册中给出的串口模式图过于复杂，包含了许多不利于学习理解的信号。所以下图给出了单片机内部的串口简化模式图。可以看到，最关键就是```SBUF```(存储发送和接收的数据)、```RI```（接收完毕中断）、```TI```（发送完毕的中断）这三个信号，甚至于```RXD```、```TXD```都不用我们手动配置，只需要关心那两个中断请求就好。

<div align=center>
<img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/%E4%B8%B2%E5%8F%A3%E6%A8%A1%E5%BC%8F%E5%9B%BE.png" width="70%">

图6-6 串口模式图
</div>

> - **SBUF**：串口数据缓存寄存器，物理上是两个独立的寄存器，但占用相同的地址。对于MCU来说，发送缓冲器只能写入，接收缓冲器只能读出，所以 **可以同时操作**。
> - 根据上面的图可以发现，串口发送/接收所需要的 **波特率都只能由定时器T1**来提供！！且T1工作在模式3（8位自动重装）。
> - T1溢出率意思为定时器T1的溢出速率，可以SBUF的1bit数据保持时间与波特率一致。

<div align=center>
<img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/%E4%B8%B2%E5%8F%A3%E7%9B%B8%E5%85%B3%E5%AF%84%E5%AD%98%E5%99%A8.png" width="70%">

图6-7 串口相关寄存器
</div>

> - **串行控制寄存器SCON**（可位寻址）：选择串行通信的工作方式和某些控制功能。
> SM0/FE：**PCON**中的SMOD0选择其为SM0时，与下面的SM1共同选择串口工作模式（[SM0,SM1]=[0,1]模式1）；**PCON**中的SMOD0选择其为FE时，检测到帧错误则拉高，且必须由软件归零。
> SM2：允许方式2/3多机通信。方式1中，SM2=1则接收到有效的停止位RI才拉高。
> REN：允许/禁止串行接收控制位（1/0）。默认禁止。
> TB8：方式2/3中发送的第9位数据，软件控制。
> RB8：方式2/3要接收的第9位数据。方式
> TI：表明发送完毕的中断请求，停止位开始的发送时拉高，必须软件复位。
> RI：表明接收完毕的中断请求。只有模式0需要软件置位。
> - **波特率选择特殊功能寄存器PCON**（不可位寻址）：
> SMOD：波特率加倍控制位，软件置位。SMOD=1，工作模式1、2、3波特率加倍；SMOD=0，波特率不加倍。
> SMOD0：帧错误检测有效控制位。决定了**SCON** 中的SM0/FE被选为FE（帧错误检测）还是SM0（0/1）。
> - **IE**、**IPH**、**IP**：
> 由于单片机串口通信用到了RI、TI两个中断，所以还需要配置 **串口通信中断源** 中断寄存器。可以参考上一节。


## 6.2 实验：串口向电脑发送数据
需求：单片机不断每秒向电脑发送一个8位递增数字（也可以直接发送单个字符，如'A'）。

<div align=center>
<img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/%E4%BB%A3%E7%A0%81%E8%B0%83%E7%94%A8-%E4%B8%B2%E5%8F%A3%E5%8F%91%E9%80%81%E8%BF%9E%E7%BB%AD%E6%95%B0%E5%AD%97.png" width="20%">

图6-8 “串口发送连续数字”代码调用关系
</div>

代码展示：
**- main.c**
```c
#include <REGX52.H>
#include "Delay.h"
#include "UART.h"

void main(){
  unsigned char num = 0x00; // 要发送的变量
  UART_Init(); // 串口初始化
  while(1){
    UART_SendByte(num);
    Delay(1000); // 软件延时，防止打乱定时器的操作
    num += 1;
  }
}
```

**- UART.h**
```c
#ifndef __UART_H__
#define __UART_H__

void UART_Init(); // 串口初始化
void UART_SendByte(unsigned char Byte); // 串口发送1Byte数据

#endif
```

**- UART.c**
```c
#include <REGX52.H>

/**
  * @brief :串口通信初始化，4800bps@11.0592MHz
  * @param :无
  * @retval :无
 */
void UART_Init(){
  SCON = 0x40; // 串口控制寄存器，暂时接收不使能
  PCON &= 0x7f; // 波特率选择特殊功能寄存器
  
  // 发送数据不需要开启中断，接收数据才需要
  
  // 下面配置定时器，用于波特率设置
  // 注：串口只能用定时器T1（8位自动重装模式）
  TMOD&=0x0F;TMOD|=0x20; //设置定时器模式
	TL1 = 0x66;  //设置定时初始值
	TH1 = 0xFA;  //设置定时初始值
	ET1 = 0;		 //禁止定时器中断，只要有溢出就可以生成波特率
	TR1 = 1;		 //定时器1开始计时
}

/**
  * @brief :通过串口发送8位数据
  * @param :需要发送的8位数据
  * @retval :无。但函数会直到该字节发送完成才返回。
 */
void UART_SendByte(unsigned char Byte){
  SBUF = Byte;
  while(TI==0); // 等待发送完毕
  TI = 0; // 软件复位
}
```

**- Delay.h**
```c
#ifndef __DEALY_H_
#define __DEALY_H_

// 延时cycles ms，晶振@11.0592MHz
void Delay(unsigned int cycles){
  unsigned char i, j;
  do{
    i = 2;
    j = 199;
    do{
      while (--j);
    }while (--i);
  }while(--cycles);
}

#endif
```

## 6.3 实验：电脑通过串口控制LED
需求：电脑向单片机发送一个8位数字，用于控制8个LED的点亮情况，同时单片机将接收到的数据返回给电脑。

本节实验与上一个实验最大的区别就是，串口初始化的过程中<u>添加了允许串口中断</u>。下面是代码展示：
**- main.c**
```c
#include <REGX52.H>
#include "UART.h"

void main(){
  UART_Init(); // 串口初始化
  while(1){}
}

// 串口接收中断子函数
// 注意发送和接收两种中断都会进入到这个子函数中来
void UART_Routine() interrupt 4{
  if(RI==1){
    P2 = ~SBUF; // 存储接收到的字节
    RI = 0; // 软件清空接收完成标志
    UART_SendByte(SBUF);
  }
}
```

**- UART.h**
```c
#ifndef __UART_H__
#define __UART_H__

void UART_Init(); // 串口初始化
void UART_SendByte(unsigned char Byte); // 串口发送1Byte数据

#endif
```

**- UART.c**
```c
#include <REGX52.H>

/**
  * @brief :串口通信初始化，4800bps@11.0592MHz
  * @param :无
  * @retval :无
 */
void UART_Init(){
  SCON = 0x50; // 串口控制寄存器，接收使能
  PCON &= 0x7f; // 波特率选择特殊功能寄存器
  
  // 接收数据需要串口中断
  EA = 1; ES = 1; // 允许全局中断、串口中断
  
  // 下面配置定时器，用于波特率设置
  // 注：串口只能用定时器T1（8位自动重装模式）
  TMOD&=0x0F;TMOD|=0x20; //设置定时器模式
  TL1 = 0xFA;  //设置定时初始值
  TH1 = 0xFA;  //设置定时初始值
  ET1 = 0;		 //禁止定时器中断，只要有溢出就可以生成波特率
  TR1 = 1;		 //定时器1开始计时
}

/**
  * @brief :通过串口发送8位数据
  * @param :需要发送的8位数据
  * @retval :无。但函数会直到该字节发送完成才返回。
 */
void UART_SendByte(unsigned char Byte){
  SBUF = Byte;
  while(TI==0); // 等待发送完毕
  TI = 0; // 软件复位
}

/*串口中断函数模板
// 串口接收中断子函数
// 注意发送和接收两种中断都会进入到这个子函数中来
void UART_Routine() interrupt 4{
  if(RI==1){ // 若接收到1Byte数据
    
  }
}
*/
```

编程感想：
> 其实好几次现象不对，都是因为对于串口助手的使用不熟练。注意点的是“发送数据”，而不是“发送回车”；另外发送和接收的模式“文本模式”/“HEX模式”要匹配上。
