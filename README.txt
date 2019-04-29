一、简介
    本文档主要介绍如何在Nvidia tx2上使用tensorrt加速yolov3-tiny。
二、步骤
    1、环境配置
        1.1 所需环境：  jetpack4.2:
                        ubuntu 18.04
                        tensorrt 5.0.6.3
                        cuda 10.0
                        cudnn 7.3.1
        1.2 tx2安装环境较麻烦，建议使用jetpack刷机，将自动安装最新驱动、CUDA Toolkit、cuDNN、tensorRT等所需软件，一步到位。
            下面以jetpack4.2为例，说明tx2刷机步骤。
                (1)所需设备：NVIDIA tx2及其配件  x1;
                            显示器(连接tx2)     x1;
                            键盘(连接tx2)       x1;
                            鼠标(连接tx2)       x1;
                            host机器            x1;
                            网线                x2;
                      说明：1)host主机需要x86_64结构的ubuntu16操作系统，用来运行sdkmanager-[version].[build#].deb文件（实测可以使用虚拟机）;
                            2)烧写期间需要将tx2与host连接到同一局域网内;
                            3)烧写jetpack4.2需要在tx2端操作，需提前准备鼠标、键盘以及显示器;
                            4)烧写时会从互联网下载各种包，网络原因可能导致无法连接至安装源;
                            5)烧写耗时较长;
                (2)准备 sdkmanager-[version].[build#].deb (见压缩包根目录) 
                (3)在host机器上运行.deb文件，进入jetpack4.2可视界面;
                (4)根据提示，选择设备型号等;
                (5)进入下载页面，待install进程到40%时，会提示连接tx2，选择manual step;
                (6)之后将设备置于usb恢复模式，所以
                   1)断开tx2电源，使其处于断电关机状态;
                   2)用网线将tx2连入host所在的局域网中;
                   3)用Micro USB线(黑色)将tx2连接到host机上;
                   4)接通tx2电源，按power键开机;
                   5)长按recovery键不松开，同时点按一下reset键，等待3s后松开recovery键，此时tx2处于强制恢复模式。
                (7)检查tx2与host是否正确连接，打开终端输入lsusb，列表中有Nvidia Corp说明连接成功;
                (8)检查无误后点击flash;
                (9)当Install到50%时tx2会开机，根据指示设置完成ubuntu18系统，在host端输入账号密码，开始安装软件;
                (10)当install到100%时，刷机完成。
		
             刷机参考： 
                https://blog.csdn.net/weixin_43842032/article/details/88753724
        2、使用说明
        2.1 依赖安装及脚本使用
            (1)安装所需软件包
                解压 yolov3-tiny_trt.rar
                进入yolov3-tiny_trt文件夹
                使用安装requirement.txt中所列的包：
                # 使用pip安装，需先安装pip：
                sudo apt-get install python-pip
		
                # 安装pycuda等：
                pip install pycuda
                pip install Pillow    # 安装Pillow时可能会提示缺少依赖
	    
            (2)脚本使用：
                # 打开inference.py，在105行至107行处分别指定 输入尺寸，输入图像列表文件，保存路径：
                INPUT_SIZE = 608    # [416, 480, 544, 608]
                INPUT_LIST_FILE = './ped_list.txt'
                SAVE_PATH = './images_results/'
				
                # 之后运行脚本
                python inference.py
		
                可得到检测结果，检测时间，并保存图像至指定目录。
		
                Performance：engine_416:  9.486ms
                             engine_480: 11.174ms
                             engine_544: 17.271ms
                             engine_608: 20.249ms					 
         2.2 使用说明
                在tx2上运行脚本时，建议开始最大功耗模式，并开启风扇，以实现最优加速效果：
                # 查询当前工作模式
                sudo nvpmodel -q verbose
		
                # 开启最大工作模式 MAXN 0
                sudo nvpmodel -m 0
		
                # 打开风扇
                sudo /usr/bin/jetson_clocks
		
        3、工程结构与调用说明
        3.1 文件列表：
            --yolov3-tiny_trt/
                --engines/
                --images/
                --images_results/
                --README.txt
                --requirements.txt
                --common.py
                --data_processing.py
                --inference.py
                --ped_list.txt
        3.2 (1)如3.1，engines中保存有416, 480, 544, 608, 4种不同输入尺寸的engine文件，用于推理;
            (2)在images中存数张有测试图像;
            (3)requirements.txt中为所需包的列表;
            (4)common.py文件中，定义allocate_buffers()函数以及do_inference()函数，用于分配空间及推理;
            (5)data_processing.py中，分别定义函数，用于图像的加载和预处理，以及模型输出的后处理，如nms等;
            (6)inference.py中，定义检测框绘制函数和engine加载函数，以及检测main函数，加载指定engine文件后，对输入进行推理，后对输出结果进行处理，绘制检测结果并保存。
					
 					
		
