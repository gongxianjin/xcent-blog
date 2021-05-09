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
  
  //point first blue button 
  fill(0,0,255);
  
  stroke(255);
  rect(rectX, rectY, rectSize, rectSize);
  
  
  //point second blue button 
  fill(255,0,0);
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



## 实验结果


点击不同颜色的区块可有发出不同的串口字节，下一次希望控制led灯


## 参考与致谢
 


> 文章作者：**xcent**  
> 原文地址：<https://xcent-blog.vercel.app/>  
> 版权声明：文章采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by/4.0/deed.zh) 协议，转载请注明出处。
