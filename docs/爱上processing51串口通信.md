---
id: 爱上processing51串口通信
title: 爱上processing
sidebar_label: 爱上processing
---


—— processing 开发测试
 

## 背景

processing 是上位机开发的一个快速验证工具：

跨平台 （意味着可以在 Windows/MacOS/Linux 上使用）<br />
 
安装 官网下载对应版本


## 干货 直接上代码

```
import processing.serial.*;
 
Serial myPort;
float val; 
int whichKey = -1;  // Variable to hold keystoke values
int inByte = -1;    // Incoming serial data

int rectX, rectY;      // Position of square button
int circleX, circleY;  // Position of circle button
int rectSize = 90;     // Diameter of rect
int circleSize = 93;   // Diameter of circle


boolean rectOver = false;
boolean circleOver = false;

 void setup(){  
   size(400, 300);
  // List all the available serial ports:
  printArray(Serial.list());
  
  PFont myFont = createFont(PFont.list()[2], 14);
  textFont(myFont);
  
  // I know that the first port in the serial list on my mac
  // is always my  FTDI adaptor, so I open Serial.list()[0].
  // In Windows, this usually opens COM1.
  // Open whatever port is the one you're using.
  String portName = Serial.list()[1];
  myPort = new Serial(this, portName, 2400);
  
  circleX = width/2+circleSize/2+10;
  circleY = height/2;
  
  rectX = width/2-rectSize-10;
  rectY = height/2-rectSize/2;
  
  ellipseMode(CENTER);
 } 
  
 void draw(){
   update(mouseX, mouseY);
  //background(255);
  //if (mouseOverRect() == true) {  // If mouse is over square,
  //  fill(204);                    // change color and
  //  myPort.write('1');              // send an H to indicate mouse is over square
  //}  
  //rect(50, 50, 100, 100);         // Draw a square
  
  //background(0);
  //text("Last Received: " + inByte, 10, 130);
  //text("Last Sent: " + whichKey, 10, 100);
  
  //point first red button 
  
  fill(255,0,0);
  stroke(255);
  rect(rectX, rectY, rectSize, rectSize);
  
  
  //point second blue button 
  
  fill(255,255,255);
  stroke(0);
  ellipse(circleX, circleY, circleSize, circleSize);
  
  
 }
 
//boolean mouseOverRect() { // Test if mouse is over square
//  return ((mouseX >= 50) && (mouseX <= 150) && (mouseY >= 50) && (mouseY <= 150));
//}
 

void serialEvent(Serial myPort) {
  inByte = myPort.read();
}

void keyPressed() {
  // Send the keystroke out:
  myPort.write(key);
  whichKey = key;
}


void mouseClicked(){
  
  // if click in rect 
  if(rectOver){
      myPort.write('1');    
  }
  
  // if click in ellipse
   if(circleOver){
     myPort.write('2');    
  }
  

}


void update(int x, int y) {
  if ( overCircle(circleX, circleY, circleSize) ) {
    circleOver = true;
    rectOver = false;
  } else if ( overRect(rectX, rectY, rectSize, rectSize) ) {
    rectOver = true;
    circleOver = false;
  } else {
    circleOver = rectOver = false;
  }
}


boolean overRect(int x, int y, int width, int height)  {
  if (mouseX >= x && mouseX <= x+width && 
      mouseY >= y && mouseY <= y+height) {
    return true;
  } else {
    return false;
  }
}

boolean overCircle(int x, int y, int diameter) {
  float disX = x - mouseX;
  float disY = y - mouseY;
  if (sqrt(sq(disX) + sq(disY)) < diameter/2 ) {
    return true;
  } else {
    return false;
  }
}






```


51单片机

```

/***************************************************************************
标题：	串口调试程序，中断法接收数据
效果：	运行本例子，打开串口调试上位机软件，设置波特率2400，再进行发送数据，数码管显示接收内容
工作于：51开发板 
网站：	
******************************************************************************/

//头文件:
#include "reg51.h"

//变量定义:
unsigned char table[]=
			{0xc0,0xf9,0xa4,0xb0,0x99,0x92,0x82,0xf8,0x80,0x90,0x88,0x83,0xc6,0xa1,0x86,0x8e};
			//表：共阳数码管 0-9 a-f,十六进制显示	
unsigned char ldata=0;	//作为接收及发送的数据变量
unsigned char oldkey=0xff;	//保按键接口状态

//引脚定义:	
sbit SMG_q = P1^0;	//定义数码管阳级控制脚（千位）
sbit SMG_b = P1^1;	//定义数码管阳级控制脚（百位）
sbit SMG_s = P1^2;	//定义数码管阳级控制脚（十位）
sbit SMG_g = P1^3;	//定义数码管阳级控制脚（个位）

sbit SPK  = P1^5;	//定义蜂鸣器
sbit key1 = P3^7;	//定义按键1
sbit key2 = P3^6;	//定义按键2
sbit key3 = P3^5;	//定义按键3
sbit key4 = P3^4;	//定义按键4
sbit key5 = P3^3;	//定义按键5
sbit key6 = P3^2;	//定义按键6

sbit led = P2^0;

//函数声明:
unsigned char ReadKey(void);	//读键值返回
void display(unsigned char da);//定义显示函数，参数为显示内容，只显示两位十六进制	
void delay(unsigned int timer);


//主函数，C语言的入口函数:	
void main(void)  
{
	unsigned char lold=1;	//作为按键放开的凭证，1为放开，0为按下

	EA=1;					//开启总中断	
	TMOD=0X20;				//设置定时器1为模式2，做为波特率发生器
	TL1=0xF3;				//2400波特率的初值,板子使用12M晶振，初值=256-12000000/32/12/2400
	TH1=0xF3;				//
	TR1=1;					//启动波特率发生
	SCON=0x50;				//串口方式2
	ES=1;					//开始串口中断

	while(1){					//循环扫描按键及显示
		switch(ReadKey()){		//读取键值做以下相应处理
		case 1:ldata++;			//1键显示内容加1
			break;
		case 2:ldata--;			//2键显示内容减1
			break;
		case 3:ldata+=16;		//3键显示内容加10
			break;
		case 4:ldata-=16;		//4键显示内容减10
			break;
		case 5:ldata=0;			//5键显示内容清0
			break;
		case 6:
			SBUF=ldata;	//这里直接将数据发送出去
			while(!TI){}
			TI=0;   	//等待发送完毕返回继续检测
			break;
		}	
		if(ldata==0x31){
			led=0;
		}	
		if(ldata==0x32){
			led=1;
		}
		display(ldata);		//显示相应的数值，这里用函数的形式调用显示
	}
}

//串口中断入口函数
void com_isr(void) interrupt 4
{	
	if(RI){					//是接收中断处理
		RI=0;				//将寄存器复位等待接收下一个
		ldata=SBUF;			//将接收的数据存于变量中
	}
}

//扫描键盘
unsigned char ReadKey(void)	
{
	unsigned char lkey=0;
	if((P3&0xfc)!=0xfc){	//检测所有按键是否有按键按下

		SPK=0;				//按键按下则蜂鸣器发声

		delay(10);			//有按键按下则延时一段时间，以消除按键拌动
		if(oldkey!=(P3&0xfc)){//检测按键未松开而且不是之前按下的按健就进入读键值，否则不作处理
			oldkey=P3&0xfc;		//保存当前状态，为下一次扫描做判断，请看上一行		
			if(key1==0)
				lkey=1;		//第一个键按下复值1
			else if(key2==0)
				lkey=2;		//第二个键按下复值2
			else if(key3==0)
				lkey=3;		//第三个键按下复值3
			else if(key4==0)
				lkey=4;		//第四个键按下复值4
			else if(key5==0)
				lkey=5;		//第五个键按下复值5
			else if(key6==0)
				lkey=6;		//第六个键按下复值6
		}
	}
	else{
		oldkey=0xfc;		//如果都没有按键按下，将复值初始状态

		SPK=1;				//无按键按下则退出蜂鸣
	}
	return lkey;
}

void display(unsigned char da)
{
	SMG_b=0;			//选择百位数码管
	P0=table[da/16];	//查表输出定义好的数码管段值与P0口，显示相应的0-f
	delay(1);			//延时
	P0=0XFF;			//清除数码管显示，因是共阳型，所以不是0
	SMG_b=1;			//关闭百位数码管

	SMG_s=0;			//选择十位数码管
	P0=table[da%16];	//查表输出定义好的数码管段值与P0口，显示相应的0-f
	delay(1);			//延时
	P0=0XFF;			//清除数码管显示，因是共阳型，所以不是0
	SMG_s=1;			//关闭十位数码管

	SMG_g=0;			//选择个位数码管，
	P0=0x89;			//显示一个"H",表示十六进制显示
	delay(1);			//延时
	P0=0XFF;			//清除数码管显示，因是共阳型，所以不是0
	SMG_g=1;			//关闭个位数码管
}

//延时子函数
void delay(unsigned int timer){		
	while(timer--);
}


```


## 实验结果


点击不同颜色的区块可有发出不同的串口字节，下一次希望控制led灯


## 参考与致谢
 


> 文章作者：**xcent**  
> 原文地址：<https://xcent-blog.vercel.app/>  
> 版权声明：文章采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by/4.0/deed.zh) 协议，转载请注明出处。
