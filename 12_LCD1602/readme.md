# 12 LCD1602液晶屏

 [toc]

注：笔记主要参考B站江科大自化协教学视频“[51单片机入门教程-2020版 程序全程纯手打 从零开始入门](https://www.bilibili.com/video/BV1Mb411e7re?spm_id_from=333.1007.top_right_bar_window_custom_collection.content.click)”。
***

## 12.1 LCD1602介绍
LCD1602（Liquid Crystal Display）液晶显示屏是一种字符型液晶显示模块，可以显示ASCII码的标准字符和其它的一些内置特殊字符，还可以有8个自定义字符。
> 显示容量：16×2个字符，每个字符为5*7点阵

<div align=center>
<img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/%E6%B6%B2%E6%99%B6%E5%B1%8F%E5%AE%9E%E7%89%A9%E5%9B%BE.png" width=65%>
</div><div align=center>
图12-1 各种各样的液晶屏
</div>

<div align=center>
<img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/LCD1602%E5%8E%9F%E7%90%86%E5%9B%BE1.png" width=65%>
</div><div align=center>
图12-2 LCD1602原理图
</div>

LCD1602控制的关键，在于中间这几个加粗的引脚。优先级最高的是 **使能引脚EN**，DB0~DB7位数据，通过RS和RW的配合就可以完成对于液晶屏数据/指令的读写。目前主要关心**写指令**（怎么显示，在哪里显示）、**写数据**（写什么）。


<div align=center>
<img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/LCD1602%E5%86%85%E9%83%A8%E7%BB%93%E6%9E%84%E6%A1%86%E5%9B%BE.png" width=70%>
</div><div align=center>
图12-3 LCD1602内部结构框图
</div>

> - 屏幕(16*2)。
> - CGRAM+CGROM(字模库)：“CG”表示“Character Generator”，即字模生成。用于存放需要显示的字符与液晶屏上像素点的对应关系，相当于数码管显示中的“段码表”。
> <img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/LCD1602-%E5%AD%97%E6%A8%A1%E5%BA%93.png" width=60%>
> 上面左侧的CGRAM所示的8个寄存器可以自定义，下面的8个和上面8个地址相同，相当于重复。后面的字模都放在CGROM中。
> - DDRAM(40*2)：用于存放屏幕上需要显示的数据。注意其一般只显示前16列元素到液晶屏上，剩余的区域不会显示。但是也可以利用这些区域做滚动显示。
> <img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/LCD1602-DDRAM%E5%9C%B0%E5%9D%80.png" width=60%>
> - 控制器：尤其注意AC(光标位置)：“AC”意为Address Counter。即对于DDRAM进行操作时要格外关心所操作的地址。


**对LCD显示进行操作的基本流程为：**
> **1. LCD初始化：**
	发送指令0x38    //八位数据接口，两行显示，5*7点阵
	发送指令0x0C	//显示开，光标关，闪烁关
	发送指令0x06	//数据读写操作后，光标自动加一，画面不动
	发送指令0x01	//清屏
> **2. 显示字符：**
	发送指令0x80|AC	//设置光标位置
	发送数据		//发送要显示的字符数据
	发送数据		//发送要显示的字符数据
	……

<div align=center>
<img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/LCD1602%E6%8C%87%E4%BB%A4%E9%9B%86.png" width=80%>
</div><div align=center>
图12-4 LCD1602指令集
</div>

下面给出读写操作的时序图：
> - 写操作时序图及时间要求：
> <img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/LCD1602-%E5%86%99%E6%93%8D%E4%BD%9C.png" width=60%>
> <img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/LCD-%E5%86%99%E6%93%8D%E4%BD%9C%E6%97%B6%E9%97%B4%E8%A6%81%E6%B1%82.png" width=60%>
> - 读操作时序图及时间要求：
> <img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/LCD-%E8%AF%BB%E6%93%8D%E4%BD%9C.png" width=60%>
> <img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/LCD-%E8%AF%BB%E6%93%8D%E4%BD%9C%E6%97%B6%E9%97%B4%E8%A6%81%E6%B1%82.png" width=60%>
> - **注意，最重要的点在于使能信号 ```EN``` 不仅拉高时有延时要求，拉低后同样也有时间要求。** 另外经实测，时间要求也都写的不对，**建议使能信号 ```EN``` 拉高和拉低的持续时间都为1ms。**


下面介绍一些有关C语言语法的知识：
> - 字符：根据一定规则建立的数字到字符的映射 **（ASCII码表）**
	例如：0x21=’!’，0x41=’A’，0x00=’\0’
	定义方法：```char x=‘A’;```（等效于```char x=0x41;```）
> - 字符数组：存储字符变量的一个数组
	定义方法：```char y[]={’A’, ’B’, ’C’};```（等效于```char y[]={0x41,0x42,0x43};```）
> - 字符串：在字符数组后加一个字符串结束标志，本质上是字符数组
	定义方法：```char z[]=”ABC”;```（等效于```char z[]={’A’, ’B’, ’C’, ’\0’};```）


## 12.2 实验：LCD1602功能代码
需求：在液晶屏上依次显示：
> 1. 一个字符 ‘A’
> 2. 一个字符串 "Hello"
> 3. 一个无符号数字 88
> 4. 一个有符号数字 -00
> 5. 一个十六进制数字 0xFE
> 6. 一个二进制数字 01011100
> 7. 显示特殊符号℃上面的“句号”

<div align=center>
<img src="https://raw.githubusercontent.com/jjejdhhd/Git_img2023/main/8051chip/%E4%BB%A3%E7%A0%81%E8%B0%83%E7%94%A8-LCD1602%E6%98%BE%E7%A4%BA.png" width=20%>
</div><div align=center>
图12-5 “LCD1602”代码调用关系
</div>

代码展示：
**- main.c**
```c
#include <REGX52.H>
#include "LCD1602.h"

void Delay500ms(){//@11.0592MHz
	unsigned char i, j, k;
	i = 4;
	j = 129;
	k = 119;
	do{
		do{
			while (--k);
		}while (--j);
	}while (--i);
}

void main(){
  //LCD初始化
  LCD1602_Init();
  // LCD显示
  LCD1602_DispChar(1,1,'A');//显示单个字符
  LCD1602_DispString(1,3,"$_$");//显示字符串
  LCD1602_DispUnInt(1,7,99,3);//显示无符号数字
  LCD1602_DispInt(1,11,-32768,6);//显示有符号数字
  LCD1602_DispUnInt_Hex(2,1,255,2);//显示无符号整型的16进制表示
  LCD1602_DispUnInt_Bin(2,4,3,3);//显示无符号整型的2进制表示
  LCD1602_DispFloat(2,8,-20.99,7,3);//显示单精度浮点数-20.99
  LCD1602_DispChar(2,15,0xdf);//显示特殊符号（主要就是查看字模表）
  //LCD移屏
  LCD1602_DispString(1,20,"Fries on the pier!");
  while(1){
    LCD1602_WriteCommd(0x18);
    Delay500ms();
    Delay500ms();
  }
}
```

**- LCD1602.h**
```c
#ifndef __LCD1602_H__
#define __LCD1602_H__

// LCD1602写指令
void LCD1602_WriteCommd(unsigned char wByte);
// LCD1602写数据
void LCD1602_WriteData(unsigned char wByte);
// 初始化函数
void LCD1602_Init(void);
//显示单个字符
void LCD1602_DispChar(unsigned char Row,unsigned char Column,unsigned char wChar);
//显示字符串
void LCD1602_DispString(unsigned char Row,unsigned char Column,char wString[]);
//显示无符号数字
void LCD1602_DispUnInt(unsigned char Row, unsigned char Column, unsigned int wNum, unsigned char Length);
//显示有符号数字
void LCD1602_DispInt(unsigned char Row, unsigned char Column,int wNum, unsigned char Length);
//显示无符号整型的16进制表示
void LCD1602_DispUnInt_Hex(unsigned char Row, unsigned char Column, unsigned int wNum, unsigned char Length);
//显示无符号整型的2进制表示
void LCD1602_DispUnInt_Bin(unsigned char Row, unsigned char Column, unsigned int wNum, unsigned char Length);
//显示单精度浮点数（有符号）
void LCD1602_DispFloat(unsigned char Row, unsigned char Column, float wNum, unsigned char Total_Len, unsigned char Frac_len);

#endif
```

**- LCD1602.c**
```c
#include <REGX52.H>

//下面这些有可能需要更改
/***************************************************************/
// 重新命名端口
sbit LCD1602_RS = P2^6;
sbit LCD1602_RW = P2^5;
sbit LCD1602_EN = P2^7;
#define  LCD1602_DB P0


// LCD1602专用延时函数，晶振11.0592MHz，延时1ms
void LCD1602_Delay1ms(void){//@11.0592MHz
	unsigned char i, j;
	i = 2;
	j = 199;
	do{
		while (--j);
	}while (--i);
}
/***************************************************************/

/***************************************************************/
/**
  * @brief :LCD1602写指令。
  * @param :wByte指令。
  * @retval :无。
 */
void LCD1602_WriteCommd(unsigned char wByte){// LCD1602写指令
  LCD1602_EN = 0;
  LCD1602_RS = 0;
  LCD1602_RW = 0;
  LCD1602_DB = wByte;
  LCD1602_EN = 1;
  LCD1602_Delay1ms();//延时1ms
  LCD1602_EN = 0;
  LCD1602_Delay1ms();//延时1ms
}

/**
  * @brief :LCD1602写数据。
  * @param :wByte数据。
  * @retval :无。
 */
void LCD1602_WriteData(unsigned char wByte){// LCD1602写数据
  LCD1602_EN = 0;
  LCD1602_RS = 1;
  LCD1602_RW = 0;
  LCD1602_DB = wByte;
  LCD1602_EN = 1;
  LCD1602_Delay1ms();//延时1ms
  LCD1602_EN = 0;
  LCD1602_Delay1ms();//延时1ms
}

/**
  * @brief :设置光标位置。
  * @param :Row行，范围1~2。
  * @param :Column列，范围1~40。
  * @retval :无。
 */
void LCD1602_SetCursor(unsigned char Row,unsigned char Column){
    unsigned char LCD_AC=0x00;
    // 更新坐标位置
    if(Row==1){LCD_AC += (Column-1);}
    else      {LCD_AC += (0x40+Column-1);}
    // 设置光标位置
    LCD1602_WriteCommd(0x80|LCD_AC);
}

/**
  * @brief :无符号数的指数运算。
  * @param :x无符号整数基底。
  * @param :y无符号整数幂次。
  * @retval :result无符号整数运算结果。
 */
int LCD1602_Pow(int x, int y){//无符号数的指数运算
  unsigned int i;
  unsigned int result = 1;
  for(i=0;i<y;i++){
    result *= x;
  }
  return result;
}

/**
  * @brief :初始化函数。
  * @param :无。
  * @retval :无。
 */
void LCD1602_Init(void){// 初始化函数
  LCD1602_WriteCommd(0x38);//八位数据接口，两行显示，5*7点阵
  LCD1602_WriteCommd(0x0c);//显示开，光标关，闪烁关
  LCD1602_WriteCommd(0x06);//数据读写操作后，光标自动加一，画面不动
  LCD1602_WriteCommd(0x01);//清屏
}

/**
  * @brief :显示字符
  * @param :Row行，范围1~2。
  * @param :Column列，范围1~40。
  * @param :wChar待显示的字符。
  * @retval :无。
 */
void LCD1602_DispChar(unsigned char Row,unsigned char Column,unsigned char wChar){//显示单个字符
  // 首先保证行列的位置有效，才运行函数
  if((Row==1 || Row==2) && Column>=1 && Column<=40){
    // 更新光标位置
    LCD1602_SetCursor(Row,Column);
    // 发送要显示的数据
    LCD1602_WriteData(wChar);
  }
}

/**
  * @brief :显示字符串
  * @param :Row行，范围1~2。
  * @param :Column列，范围1~40。
  * @param :wString[]待显示的字符串。
  * @retval :无。
 */
void LCD1602_DispString(unsigned char Row,unsigned char Column,char wString[]){//显示字符串
  unsigned char i=0;
  // 首先保证行列的位置有效，才运行函数
  if((Row==1 || Row==2) && Column>=1 && Column<=40){
    // 更新光标位置
    LCD1602_SetCursor(Row,Column);
    // 发送要显示的数据
    while(wString[i]!='\0'){
      LCD1602_WriteData(wString[i]);
      i++;
    }    
  }
}
  
/**
  * @brief :显示无符号数字
  * @param :Row行，范围1~2。
  * @param :Column列，范围1~40。
  * @param :wNum待显示的无符号数字，范围0~65535。
  * @param :Length在屏幕上显示的长度，范围1~16。
  * @retval :无。
 */
void LCD1602_DispUnInt(unsigned char Row, unsigned char Column,
                       unsigned int wNum, unsigned char Length){//显示无符号数字
  unsigned char i=0;
  unsigned char temp=0;//存储单个位上的数据
  // 首先保证行、列、显示长度有效，才运行函数
  if((Row==1 || Row==2) && Column>=1 && Column<=40 && Length>=1){
    // 1. 更新光标位置
    LCD1602_SetCursor(Row,Column);
    // 2.发送要显示的数据
    // 2.1 超过5位的长度都显示0
    if(Length>16){Length=16;}
    while(Length>5){
      LCD1602_WriteData(0x30);//直接发送'0'的ASCII码
      Length--;
    }
    // 2.2 5位以内的真实数值
    for(i=0;i<Length;i++){
      temp = (wNum/LCD1602_Pow(10,Length-1-i))%10;
      LCD1602_WriteData(0x30|temp);//直接计算ASCII码
    } 
  }
}

/**
  * @brief :显示有符号数字
  * @param :Row行，范围1~2。
  * @param :Column列，范围1~40。
  * @param :wNum待显示的有符号数字，范围-32768~32767。
  * @param :Length在屏幕上显示的数字长度（包括正负号），范围1~16。
  * @retval :无。
 */
void LCD1602_DispInt(unsigned char Row, unsigned char Column,
                     int wNum, unsigned char Length){//显示有符号数字
  unsigned char i=0;
  unsigned char temp=0;//存储单个位上的数据
  unsigned int un_wNum=0;
  // 首先保证行、列、显示长度有效，才运行函数
  if((Row==1 || Row==2) && Column>=1 && Column<=40 && Length>=2){
    // 1. 更新光标位置
    LCD1602_SetCursor(Row,Column);
    // 2. 发送要显示的数据
    // 2.1 显示正负号，并将原来的数字转换成正数
    if(Length>16){Length=16;}
    if(wNum>=0){LCD1602_WriteData('+'); un_wNum=wNum;}
    else       {LCD1602_WriteData('-'); un_wNum=-wNum;}
    Length--;
    // 2.2 超过5位的长度都显示0
    while(Length>5){
      LCD1602_WriteData(0x30);//直接发送'0'的ASCII码
      Length--;
    }
    // 2.3 5位以内的真实数值
    for(i=0;i<Length;i++){
      temp = (un_wNum/LCD1602_Pow(10,Length-1-i))%10;
      LCD1602_WriteData(0x30|temp);//直接计算ASCII码
    } 
  }
}

/**
  * @brief :显示无符号整型的16进制表示。
  * @param :Row行，范围1~2。
  * @param :Column列，范围1~40。
  * @param :wNum待显示的无符号数字，范围0~65535。
  * @param :Length在屏幕上显示的16进制长度，范围1~4。
  * @retval :无。
 */
void LCD1602_DispUnInt_Hex(unsigned char Row, unsigned char Column,
                           unsigned int wNum, unsigned char Length){//显示无符号整型的16进制表示
  unsigned char i=0;
  unsigned char temp=0;//存储单个位上的数据
  // 首先保证行、列、显示长度有效，才运行函数
  if((Row==1 || Row==2) && Column>=1 && Column<=40&& Length>=1){
    // 1. 更新光标位置
    LCD1602_SetCursor(Row,Column);
    // 2. 发送要显示的数据
    if(Length>4){Length=4;}
    else        {wNum = (wNum<<(4*(4-Length)))>>(4*(4-Length));}//根据要显示的长度截取有用的位
    for(i=0;i<Length;i++){
      temp = (wNum>>((Length-1-i)*4))%16;
      if(temp>=0 && temp<=9){LCD1602_WriteData('0' + temp);}
      else                  {LCD1602_WriteData('A' + temp - 10);}
    } 
  }
}

/**
  * @brief :显示无符号整型的2进制表示。
  * @param :Row行，范围1~2。
  * @param :Column列，范围1~40。
  * @param :wNum待显示的无符号数字，范围0~65535。
  * @param :Length在屏幕上显示的2进制长度，范围1~16。
  * @retval :无。
 */
void LCD1602_DispUnInt_Bin(unsigned char Row, unsigned char Column,
                           unsigned int wNum, unsigned char Length){//显示无符号整型的2进制表示
  unsigned char i=0;
  // 首先保证行、列、显示长度有效，才运行函数
  if((Row==1 || Row==2) && Column>=1 && Column<=40&& Length>=1){
    // 1. 更新光标位置
    LCD1602_SetCursor(Row,Column);
    // 2. 发送要显示的数据
    if(Length>16){Length=16;}
    for(i=0;i<Length;i++){
      if(wNum&(0x0001<<(Length-1-i))) {LCD1602_WriteData('1');}
      else                            {LCD1602_WriteData('0');}
    } 
  }
}

/**
  * @brief :显示单精度浮点数（有符号）。
  * @param :Row行，范围1~2。
  * @param :Column列，范围1~40。
  * @param :wNum待显示的小数，范围-3.4E38～3.4E38。
  * @param :Total_Len在屏幕上显示的总长度长度（包括正负号），范围3~16。
  * @param :Frac_len在屏幕上显示的小数部分的长度，范围1~16。
  * @retval :无。
  * 说明：优先显示小数部分。
 */
void LCD1602_DispFloat(unsigned char Row, unsigned char Column,
                       float wNum, unsigned char Total_Len,
                       unsigned char Frac_len){//显示单精度浮点数（有符号）
  unsigned char Int_len = Total_Len-2-Frac_len; //整数段长度
  unsigned int Int_part=0, Fra_part=0;
  // 首先保证行、列、显示长度有效，才运行函数
  if((Row==1 || Row==2) && Column>=1 && Column<=40 && Total_Len>=3){
    //1. 显示正负号，并将原来的数字转化成绝对值
    if(wNum>=0){LCD1602_DispChar(Row,Column,'+');}
    else       {LCD1602_DispChar(Row,Column,'-'); wNum=-wNum;}
    //2. 获取整数部分、小数部分
    Int_part=(unsigned int)wNum;
    Fra_part = ((unsigned int)(wNum*LCD1602_Pow(10,Frac_len)))%LCD1602_Pow(10,Frac_len);
    //3. 显示整数部分
    LCD1602_DispUnInt(Row,Column+1,Int_part,Int_len);
    //4. 显示小数点
    LCD1602_DispChar(Row,Column+1+Int_len,'.');
    //5. 显示小数部分
    LCD1602_DispUnInt(Row,Column+2+Int_len,Fra_part,Frac_len);

  }
}
```

编程感想：
> 1. 关于 ```sbit``` 和 ```sfr```：```sbit```可以直接在等式右边使用端口名，而```sfr```在等式右边只能写地址。所以一般使用宏定义```#define```来完成对于一个寄存器的重命名。
> 2. 函数定义时声明的简写：51单片机用的C89没有简写，只要没写就默认为```int```型。
> 3. 传递字符串：直接将字符串给函数，那么形参定义是应使用 ```char 形参名[]``` 或者 ```char *形参名``` 的形式。另外，在主函数中传递数组形式的变量进函数时，都是传递的数组首元素地址。在函数中，数组的调用格式 ```形参名[i]``` 与解引用格式 ```*(形参名+i)``` 相同。
> 4. 天坑：LCD读写数据的时序中，使能信号 ```EN``` 持续的时间，不能按照器件手册中的us级别，而是要延时大约1ms。
> 5. C语言没有自带的指数运算符，```^```表示的按位异或。而头文件<Math.h>中的```pow(x,y)```函数，两个操作数都是```float```类型。
> 6. 关于移屏：有两种可能，一是移为速度过快；二是VLCD驱动电压过高。但目前暂未找到解决方法。
> 
