# Remote-Control-Car
Making a remote control car using Arduino Uno, HC-05 Bluetooth module, L298N and so on ...

> **关键词：** 遥控小车；Arduino；直流电机；L298N电机驱动板；串口蓝牙模块

# 一、简介

本项目使用Arduino实现了一个最小功能的蓝牙遥控小车

![整车效果图](https://upload-images.jianshu.io/upload_images/9308403-fb82ccaf662a0a64.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)

![电路原理图](https://upload-images.jianshu.io/upload_images/9308403-0924c388a7429fbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



制作流程主要分为四个部分：1.组装小车；2.烧写代码；3.调试蓝牙；4.整车测试
其中，前三个部分的耦合性很低，每个部分可以独立操作和调试

# 二、实现过程

## 1. 组装小车
> **所需环境：** 十字螺丝刀×1
![车身零件全家福](http://upload-images.jianshu.io/upload_images/9308403-6416efef87aa2693.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)




#### 1.1 电机绕线及固定
###### 直流电机

本项目使用直流电机（即常见的玩具四驱车的马达）作为小车的驱动装置，直流电机的特点是：只要电机的两极有电势差，电机就可以运转，反接则逆转，两极电势差为零时停止运转。

###### 电极绕线

这款直流电机没有预先引出导线，所以需要我们手工连接导线：
首先，把杜邦线裸露出铜线的一段塞入电机铜电极的小孔中；
然后把探出的那部分铜线弯折一下，用指尖压住弯折处两段的铜线，旋转几圈，让两段铜线缠绕在一起；
这样保证了铜线和电极的充分接触，省去了焊接的麻烦。
![图片发自简书App](http://upload-images.jianshu.io/upload_images/9308403-d51c578927ce774d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)



注：没有要求电极和不同颜色导线的对应关系，可以随意连接。

###### 测试电机
把两个电机和4根杜邦线连接完毕之后就可以对电机进行简单的测试：只要把杜邦线的两头接触18650锂电池的两极即可，可以观察到电机开始快速转动，如果调换电池的电极，会发现电机发生反转。

###### 固定电机
把两个固定电机用的插销插到车底盘上对应的空槽中，然后将长螺丝穿过，用手轻轻旋上螺母
![固定电机](http://upload-images.jianshu.io/upload_images/9308403-864fb28aae32ddbb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)




#### 1.2 固定驱动板和万向轮

###### L298N工作原理

由上文的内容可知，直流电机只要一极接高电平，一极接低电平即可转动，大家也知道Arduino的数字输出引脚可以直接输出高电平或低电平，但我们一般不直接将Arduino连接到直流电机上，因为Arduino板的电流负载是有限的，直接连接电极容易引发电流过载，导致Arduino板被烧坏，所以我们选用**L298N**这块转接驱动板，作为**Arduino和电机之间的桥梁**

针脚对应关系如下图，其中**in1 ~ in4**对应**OUT1~OUT4**，我们将Arduino的数字输出针脚接到**in1 ~ in4**上，即可将对应的高低电平信号映射到**OUT1~OUT4**的接线柱上，从而控制电机
L298N除了有转换信号的功能外，内部还有稳压模块，可以接受7 ~ 12V的输入，然后转换出一个5V的输出，分别对应这下部的3个接线柱，我们之后会将18650电池组的正极接到 `7 ~ 12V输入`的接线柱上，负极`接地`，然后用L298N提供的`5V输出`和`接地`作为正负极来为Arduino板供电

> ![L298N针脚对应关系图](https://upload-images.jianshu.io/upload_images/9308403-661a1805d8cb3328.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###### 固定驱动板和接线

固定驱动板的方法很简单，我们只用两组螺丝螺母，固定到下图所示的位置，注意螺丝不要扭太紧

![固定驱动板](http://upload-images.jianshu.io/upload_images/9308403-77f40f6e1c7c1dfb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

###### 把杜邦线固定到接线柱上
使用螺母固定好L298N之后，需要将电极引出的杜邦线的针脚固定到L298N的接线柱上，方法是**先用螺丝刀松开接线柱内的螺丝，然后塞入杜邦线的针脚，最后再用螺丝上紧**，如下图

![将杜邦线固定到接线柱上](http://upload-images.jianshu.io/upload_images/9308403-67681c9f7224291c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

这部分完成


#### 1.3 装配18650电池盒和电源开关
###### 18650锂电池
18650型锂电是电子产品中比较常用的可充电锂电池，单节电压一般为3.7V，常在充电宝或笔记本电脑的电池中作为电芯使用。其型号的定义法则为：如18650型，即指电池的直径为18mm，长度为65mm，0代表是圆柱体型的电池。*

###### 选用原因

由于Arduino UNO的标准输入电压为5 ~ 9V，L298N驱动板的输入电压为7 ~ 12V，所以本项目选用了两节18650锂电池串联（串联后总电压为7.4V）的方式作为小车的电源，同时给Arduino和电机驱动板供电

###### 安装电池盒，连接开关

![安装电池盒和开关](http://upload-images.jianshu.io/upload_images/9308403-f25244f3ad3a8e84.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

给船型开关的两个引脚缠线，这部分需要耐心些，注意两个引脚的导线不要接触到一起，否则开关就会失效

![这部分需要耐心些](http://upload-images.jianshu.io/upload_images/9308403-a4f71d8370a42400.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

最后用一个螺丝螺母的组合穿过底板（*注意穿过的孔的位置*）固定电池盒和用于支撑Arduino的铜柱（*用螺母固定*），还要在L298N的接线柱上增添两条用于给Arduino供电的杜邦线，安装的时候注意牢固

![注意孔的位置](http://upload-images.jianshu.io/upload_images/9308403-cee0da0c15e6181e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

#### 1.4 固定Arduino，连接剩余杜邦线
###### 固定Arduino，根据原理图连线

然后可以用一个螺丝固定Arduino到车身上，并根据电路原理图连接从L298N给Arduino供电的杜邦线

![连接供电线路](http://upload-images.jianshu.io/upload_images/9308403-3b9acef6987b0466.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

![连接给L298N的信号线](http://upload-images.jianshu.io/upload_images/9308403-a60075565b38ea6a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

#### 1.5 通电调试
###### 无程序测试电机方法

可以用Arduino板上固定的3.3V和GND输出测试L298N

![用Arduino固定的3.3V和GND输出测试L298N](http://upload-images.jianshu.io/upload_images/9308403-8249cc72bc637aa6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

![按照原理图连线](http://upload-images.jianshu.io/upload_images/9308403-d7ff19b042d0caf1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)


## 2. 烧写代码
#### 2.1 使用Arduino IDE向开发板上传代码

连接Arduino和PC机，使用Arduino IDE打开本项目的代码文件，在【工具】菜单中选择所需要上传的端口，然后点击上传

#### 2.2 使用Arduino IDE串口工具进行调试

底部状态栏显示上传成功后，保持Arduino和PC机连接的状态，点击右上角的串口监视器，出现一个小程序框

![上传成功后，打开串口监视器](https://upload-images.jianshu.io/upload_images/9308403-cf2cc6f9d6fee7d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

串口监视器程序框的顶部是一个输入栏，在这里我们可可以对上传到板上的程序进行测试，在输入栏输入“w”后，点击【发送】按钮，会发现文本框里可以从Arduino得到相应的反馈（这是在源代码中设置的），然后还可以依次测试发送“a”、“d”、“s”、“x”这几种消息，如均正常，则程序无误。
> 使用了键盘的W、A、S、D、X键位的布局来与前进、左转、右转、停止和后退这几个概念做对应

![在串口监视器中测试](https://upload-images.jianshu.io/upload_images/9308403-62727b2395cb0b51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.3 连接车身调试
如果小车车身安装好了，这时可以把Arduino和小车车上的引脚按照原理图连接好，然后还通过上一个步骤的方法，在PC机上用串口监视器给Arduino发送消息，观察轮胎的运转情况，看是否按照程序的描述运转，如果不能，可能是在电机的引脚或者连线的时候出现偏差，解决方法有三种：
* 修改源代码中控制轮的变量与数字引脚的对应关系
* 修改Arduino到L298N信号线的连线
* 修改L298N和电机之间的连线

每种方法都可以解决小车不能按照规定接受消息的方式运转的问题，请同学们自由选择

![烧录代码和测试](http://upload-images.jianshu.io/upload_images/9308403-1ad61ec19e1216dd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)


校正轮胎转动方向

## 3. 调试蓝牙
###### 关于HC-05和调试
HC-05是一个串口蓝牙通信模块，内部的芯片上封装了蓝牙通信协议以及用于调试的AT指令集，功能是：可以通过蓝牙接受数据，再从串口通信协议从针脚发送出去，也可以从串口接受消息，再经过芯片用蓝牙发送出去，相当于通信无线到有线通信的一个桥梁。

AT指令是应用于终端设备与PC应用之间的连接与通信的指令。AT即Attention。每个AT命令行中只能包含一条AT指令；对于AT指令的发送，除AT两个字符外，最多可以接收1056个字符的长度（包括最后的空字符）

我们将蓝牙模块（HC-05）通过转换器（TTL转USB）连接到电脑上， 在电脑上使用串口调试软件（CoolTerm）向蓝牙模块发送特定的AT指令来对蓝牙模块的一些参数，比如设备名称、配对码、主从角色等等。

#### 3.1 将 串口蓝牙模块（HC-05） 与PC相连接
使用 USB转TTL模块 连接 HC-05，连线如下图

![连线：GND--GND；5V--5V；RX--TX；TX--RX](http://upload-images.jianshu.io/upload_images/9308403-5e3e1ccd2e177ef9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

> ![要按住RST键连接USB，才能进入AT调试模式](https://upload-images.jianshu.io/upload_images/9308403-4511fc2eec695a6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* 按住蓝牙模块上的RST键，插入PC机的USB口，松开RST键，进入AT指令调试模式，标志是蓝牙模块上的红色指示灯**慢速闪烁**
* 如果不按住RST键直接连接USB口的话，红色指示灯会**快速闪烁**，标志进入蓝牙连接模式



#### 3.2 打开CoolTerm，建立串口通信连接
将蓝牙模块连接到PC上后，查看设备管理器，打开CoolTerm软件，点击【Options】图标进入选项设置，【Port】选项选择设备管理器中CH340对应的端口（根据电脑的不同，不一定是COM5），【Baudrate】修改为 38400，然后点击底部的【OK】确定

![进入设置，修改端口和波特率](https://upload-images.jianshu.io/upload_images/9308403-6fa48b931e76543d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

设置完成后，点击软件上方的【Connect】图标

#### 3.3 使用AT指令调整蓝牙模块的参数
【Connect】成功后，依次点选菜单栏上的【Connection】→【Send String】，会出现一个消息发送框，通过这个可以向蓝牙模块发送AT指令，如下图：

![输入“AT+回车”](https://upload-images.jianshu.io/upload_images/9308403-651d62b4b6370083.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意，使用AT指令的时候一定要回车到第二行再点击【Send】发送

###### AT指令集
分别输入如下指令，查看设备的当前状况
“AT”：回复“OK”，说明设备正常。
“AT+UART”：查看设备的串口通信参数，本实验模块蓝牙模式的波特率为9600
“AT+ADDR”：查看设备的蓝牙地址
“AT+ROLE”：查看设备的主从转台，“0”为从模式，“1”为主模式
“AT+PSWD”：查看设备当前的配对码，默认是1234
“AT+NAME”：查看设备当前的名称，默认是 HC-05
“AT+RESET”：重启设备

###### 用AT指令配置设备
发送完上述AT指令查看完设备信息后，需要根据你的情况修改一些设备信息，方法是，在相应的AT指令后加“=”号，输入要修改的信息即可，本项目主要修改信息如下例
* “AT+ROLE=0”：设置蓝牙设备为从模式
* “AT+PSWD=123456”：修改配对码为123456，用户自定义
* “AT+NAME=Carduino”：修改名称为Carduino，用户自定义
* “AT+UART=9600,0,0”：修改蓝牙工作状态波特率为9600，无停止位和校验位

![注意！每次都要回车到第二行再点击“Send”](https://upload-images.jianshu.io/upload_images/9308403-8a323975de09fa80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于硬件存在一定的缺陷，HC-05蓝牙模块在设置【NAME】参数时会有些问题，可能需要多设置几次
如果改名不成功，请使用“AT+ADDR”查询**蓝牙设备地址**，在手机上根据搜索到的**设备地址**来连接蓝牙模块


#### 3.4 下载安装BlueSPP软件，设置按键消息
BlueSPP是一个手机端的蓝牙串口通讯通信工具，可以连接蓝牙设备，通过串行通讯协议发送消息
* 打开APP
![进入BlueSPP](http://upload-images.jianshu.io/upload_images/9308403-342a135cc25c459b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

* 连接设备
![点右上角图标搜索设备](http://upload-images.jianshu.io/upload_images/9308403-75458a4a746d85d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

* 连接
![输入事先配置的配对码](http://upload-images.jianshu.io/upload_images/9308403-347e1a3224dcbeca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

* APP首页可以在聊天窗口中向设备发送消息
![可以发送消息](http://upload-images.jianshu.io/upload_images/9308403-2e510328d73c143d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

* 右滑，进入自定义键盘页面，每个按钮的“名称”可定义，按钮对应的“按下”，“松开”事件都可配置成发送特定消息
![配置按钮“前进”](http://upload-images.jianshu.io/upload_images/9308403-0596e91946fb6229.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)
![配置按钮“左转”](http://upload-images.jianshu.io/upload_images/9308403-a84e7f55f2e72061.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)
![配置按钮“后退”](http://upload-images.jianshu.io/upload_images/9308403-b08c9b687a05bdb7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)
![配置结束](http://upload-images.jianshu.io/upload_images/9308403-34c3616f6c287a41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

| 按键名称 | 按下发送 | 松开发送 |
| ------ | :-----: | :-----: | 
| 前进 |  w  |    | 
| 后退 |  x  |  s  | 
| 左转 |  a  |  w  |
| 右转 |  d  |  w  |
| 停止 |  s  |    |
上表为笔者的配置，大家可以根据自己的控制习惯进行设置


## 4. 整车测试
> 终于到激动人心的最终环节了
#### 4.1 给Arduino连接蓝牙模块
![连接蓝牙](http://upload-images.jianshu.io/upload_images/9308403-55b9ed0145db2c40.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)
#### 4.2 使用独立电源测试
打开开关，L298N、Arduino、和HC-05蓝牙模块上的能正常闪烁，就可以在手机端用BlueSPP连接蓝牙，让小车下地开始真正的遥控测试了