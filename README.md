# wmt_eai80_study
My EAI80 Study

## Ref  
* http://forum.banana-pi.org/t/bpi-eai80-aiot-board-eaiseries-sdk-usergride/11573  
* http://wiki.banana-pi.org/BPI-EAI80_AIoT_board  
* https://github.com/SoCXin/EAI80  
* https://www.zhihu.com/zvideo/1302283780456452096  
* https://blog.csdn.net/sinovoip/article/details/106082774  

## 烧录笔记    
我把bpi-eai80开发板的hello world跑通了。官方的烧写方法是通过bootloader和虚拟u盘来做，我试过不行，改用j-link ob去烧录。步骤如下（仅供参考，会冲走原有的bootloader，导致波特率变成19200，原版是115200）：  
0) 解压BPI-EAI80-bsp-1.0.zip，安装Keil.EAISeries_DFP.1.4.1.pack  
1) 用MDK5打开BPI-EAI80-bsp-1.0\ugelis\kelis_example\app\helloworld\mdk\ugelis_demo.uvprojx  
2) 在Options对话框中User标签中加入After Build:  
fromelf.exe --bin -o "@L.bin" "#L"  
并且勾选  
3) 编译MDK5工程  
4) 打开BPI-EAI80-bsp-1.0\ugelis\tools\uGelisFlash_f6721b\uGelisFlash.exe  
右键browser选择boot1==images/boot1_1.bin  
右键browser选择system==ugelis_demo.bin（前面fromelf转换的bin文件）  
取消勾选其他分区如body_weights，只保留前面的info、boot1和system这三个分区  
5) 烧写：连接j-link ob，Flash Size选择32M，模式选择USB(JLink)，然后按右下角的Start按钮  
6) 串口查看：连接usb-c口，putty波特率19200，循环输出SDRAM BringUp:Hello World!  
7) 不需要拔掉usb线和jlinkob线，以后编译bin后只要烧写最后的分区system分区即可（usb-c的串口会提示进入JLinkFlashLoader），然后按reset按钮重新启动系统  
 
