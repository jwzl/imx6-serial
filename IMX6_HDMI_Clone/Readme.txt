该方案提供 【相同分辨率的】LVDS（单通道/双通道） + HDMI 显示相同内容， 
1. 打完patch 后，重新编译kernel 即可获得该方案的work.
2. 该方案中LVDS 在设备树中的IPU ID & DI 均不能被修改；
3. 该方案可以通过 uboot 环境变量来enable/disable， 具体请参考下面的参数:
在使能该方案时，uboot 命令行参数如下：
setenv mmcargs "setenv bootargs console=${console},${baudrate} ${smp}  root=${mmcroot} ${bootargs} advboot_version=2015.04-advantech_rsb4411a1_V6.150_svn1991+g6cf684a uboot_version=2016.03-imx_v2016.03_4.1.15_2.0.0_ga+gbe08920 video=mxcfb0:dev=ldb video=mxcfb2:dev=hdmi,1920x1080M@60,if=RGB24,clone"

添加clone 关键字会enable 这个方案， 去掉clone 会 disable 掉这个方案;


