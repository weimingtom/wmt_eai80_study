# wmt_eai80_study
My EAI80 Study

## SDK  
* https://github.com/BPI-SINOVOIP/BPI-EAI80-bsp  

## Ref  
* http://forum.banana-pi.org/t/bpi-eai80-aiot-board-eaiseries-sdk-usergride/11573  
* http://wiki.banana-pi.org/BPI-EAI80_AIoT_board  
* https://github.com/SoCXin/EAI80  
* https://www.zhihu.com/zvideo/1302283780456452096  
* https://blog.csdn.net/sinovoip/article/details/106082774  

## 烧录笔记    
我把bpi-eai80开发板的hello world跑通了。官方的烧写方法是通过bootloader和虚拟u盘来做，我试过不行，改用j-link ob去烧录。步骤如下（仅供参考，会冲走原有的bootloader，导致波特率变成19200，原版是115200）：  
0.) 解压BPI-EAI80-bsp-1.0.zip，安装Keil.EAISeries_DFP.1.4.1.pack  
1.) 用MDK5打开BPI-EAI80-bsp-1.0\ugelis\kelis_example\app\helloworld\mdk\ugelis_demo.uvprojx  
2.) 在Options对话框中User标签中加入After Build:  
fromelf.exe --bin -o "@L.bin" "#L"  
并且勾选  
3.) 编译MDK5工程  
4.) 打开BPI-EAI80-bsp-1.0\ugelis\tools\uGelisFlash_f6721b\uGelisFlash.exe  
右键browser选择boot1==images/boot1_1.bin  
右键browser选择system==ugelis_demo.bin（前面fromelf转换的bin文件）  
取消勾选其他分区如body_weights，只保留前面的info、boot1和system这三个分区  
5.) 烧写：连接j-link ob，Flash Size选择32M，模式选择USB(JLink)，然后按右下角的Start按钮  
6.) 串口查看：连接usb-c口，putty波特率19200，循环输出SDRAM BringUp:Hello World!  
7.) 不需要拔掉usb线和jlinkob线，以后编译bin后只要烧写最后的分区system分区即可（usb-c的串口会提示进入JLinkFlashLoader），然后按reset按钮重新启动系统  

## 感想  
bpi-eai80似乎是基于格力的F6721B系列芯片（可能是内部的名称），芯片尺寸比较小，  
比stm32f103略大一点，比stm32h750小，架构是Cortex-M4F，可能是外挂的Flash，  
所以可以用于AI边缘计算（可以把模型单独放在一个只读内存分区来烧写，避免重复写入）。  
如果按照网上的说法，这个开发板不适合初学者，后续缺乏相应支持，生态不够完善，  
个人开发较难。但我用了一下，觉得应该是与普通的stm32f4相似，  
就是扩展了SDRAM和Flash，应该可以用stm32f446代替，或者用扩展SDRAM和Flash的  
stm32h750来做相似的事情，例如语音识别、计算机视觉、物体检测、vSLAM、边缘计算等  

## SDRAM烧录（掉电丢失）  
我跑通了香蕉派EAI80如何直接在Keil 5（MDK5）里面烧录程序了（之前只是跑通了uGelisFlash_f6721b的烧录）。  
实际上准确来说，《4.EAISeries_SDK_UserGuide》里面提到的Keil烧录方法是SDRAM烧录，掉电会丢失，  
不同于uGelisFlash，uGelisFlash那个烧录是SPI Flash，掉电后不会丢失的。如果理解这一点，  
那么《4.EAISeries_SDK_UserGuide》里面所说的烧录方法就很容易做到了，就是不要勾选Reset and Run，  
也不要勾选Debug标签里面的Reset after Connect，这样CPU就不会重新加载SPI Flash里面的程序（防止冲走SDRAM的内容），  
然后点击Start/Stop Debug Session(F5)按钮，执行Dbg_Init.ini，把PC寄存器指向SDRAM的程序。然后再点击一下全速执行按钮即可。  
这种临时烧录方式有点类似F1C100S的内存执行方式，掉电就会丢失程序。有些人确实喜欢这种方式，  
通过J-Flash的命令来手工执行程序，就像这里的所说的Dbg_Init.ini  

## 驱动正点原子4.3寸屏（480x272分辨率）  
昨天晚上我终于把EAI80的UI工程（实际上是LCD显示示例）跑通了，如图，驱动4.3寸正点原子屏（40pin，480x272分辨率），  
用了两个屏线转接板和大量杜邦线，没接触摸脚，只接了RGB线，剩下的悬空，屏幕的BL脚接在高电平或者EAI80的LCD_PWREN。  
我没有用官方的7寸屏，因为不知道什么原因失败了。另外这个板接屏幕时不可以通过USB-C线供电（原本是用于串口调试输出），  
会导致屏幕输出发生干扰现象和变色，原因不明。原理图左图开发板右图正点原子屏，都是40针，  
多出来的41脚和42脚（A和B）不是在屏线上，而是在接口底座上，不用管的  
* 针序（接线方法）：  
https://github.com/weimingtom/wmt_eai80_study/blob/master/alientek_4.3inch_rgb_lcd.txt  
* EAI80的屏线接口  
https://github.com/weimingtom/wmt_eai80_study/blob/master/02_eai80_screen_interface.jpg  
* 4.3寸屏幕的屏线接口  
https://github.com/weimingtom/wmt_eai80_study/blob/master/03_lcd43.jpg  

## 驱动BPI的7寸mipi屏（800 x 480分辨率）
有三种屏线接口：mipi 40pin, mipi 20pin, rgb 40pin。
这里使用rgb 40pin的屏线接口  
```
香蕉派 Banana PI 七寸LCD触摸屏 ,支持 BPI 全志系列主板  
支持 ： BPI M1/M1+/M3/M2 Ultra/M2 Berry/M64/M2M (A33&R16)  
PS：M1/M1+ 使用 RGB 接口 ，   
M3/M2 Ultra/M64/M2 Berry  使用40PIN MIPI 接口 ，   
M2M (A33&R16) 使用 20PIN MIPI 接口  
（注：这是淘宝商店的说法，可能不准确）  
```
* 针序（接线方法）：  
https://github.com/weimingtom/wmt_eai80_study/blob/master/bpi_7inch_lcd_rgb.txt  
* EAI80的屏线接口  
注：当时测试这个屏时，测试的次序是：先接VCC, GND, VOUT_DE~VOUT_CLK测试背光问题；然后再接VOUT_R7~VOUT_R3测试红色。最后再试其他脚  
屏线反接不会烧，不知道为什么（可能电压不高的缘故）  
https://github.com/weimingtom/wmt_eai80_study/blob/master/04_eai80.jpg  
* 7寸屏幕的屏线接口  
https://github.com/weimingtom/wmt_eai80_study/blob/master/05_bpi7inch_rgb.jpg  

## 全志 BPI-M64驱动mipi屏（480x1280分辨率）  
https://blog.csdn.net/babyshan1/article/details/86980629  
