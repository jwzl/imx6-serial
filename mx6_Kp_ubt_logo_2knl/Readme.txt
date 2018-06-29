This Patch is mainly for keeping logo from uboot to kernel.

The version 1 has been done by fsl community, and logo from which pannel was selected
by some numerous MACRO. and once finished the build, you can't select the logo from which
pannel. After you used this patch, the system just support one display device, and it's so 
terrible!

 
To solve these above issue, advantech will provide the version 2. It has some difference with version 1,
So we show this document as user guide.

1. Which new feature in the version 2 patch?
	1) uboot can enable/disable "keeping logo" feature by uboot environment variable (logo=xxx).
	   /* In version 1, this is controled by MACRO */		
	2) uboot can select which display device to show logo (hdmi,ldb,lcd) by uboot environment variable (logo=xxx).
		/* In version 1, this is controled by MACRO */
	3) kernel can enable/disable "keeping logo" feature by uboot boot command line.
	   /* In version 1, this is controled by MACRO */
	4) kernel can select which display device to keep logo (hdmi,ldb,lcd) by uboot boot command line (according to
		the uboot's logo display device).
		/* In version 1, this is controled by device tree and MACRO*/ 
	5) Kernel support multi-dsiplay in this function.
		/* In version 1, this is not support */ 
	  




2. How to use this patch?
   The patch is based on advantech yocto2.1 bsp. 	
   In the uboot directory:
		001_uboot-imx_keep_logo_from_uboot_2_kernel.uboot.patch is the common source code for
		support keep uboot logo to kernel. so you can apply this patch directly.

		002_uboot-imx_keep_logo_from_uboot_2_kernel.adv.patch is mainly platfrom source code for
		support keep uboot logo to kernel. you can modeify the releative code by this patch for
		other platform. 
					
	In the kernel directory:
		001_keep_uboot_logo_to_kernel.kernel.patch is core code for support keep uboot logo to kernel,
		so you can apply this patch directly on advantech yocto2.1 bsp.
		002_keep_uboot_logo_to_kernel.dts.patch is device tree patch. and the default setting in device tree 
		should not be modified.
		003_keep_uboot_logo_to_kernel.fix.patch is for fix some issue for supporting keep uboot logo to kernel
		in advantech yocto2.1 bsp.

	Finished applying these patch, the bsp would support the funtion of keeping logo from uboot to kernel.






3. How to use the function of keeping logo from uboot to kernel?

3.0 flash the logo image to emmc flash.

	Before show the logo, you must flash the logo image into specified address of emmc flash.
	The resolution of this logo image must match the display device's.

	The following, we give you the flash command:
	# dd if=/dev/zero of=/dev/block/mmcblk0 bs=1 count=5242880 seek=1048576
	# dd if=./${your_logo}.bmp of=/dev/block/mmcblk0 bs=1 seek=1048576 skip=54

	For debug , to use the following command to check the image flashed okay.
	#hexdump -C /dev/sdb -s 1048576 -n 64  



3.1 uboot logo

	All is done by uboot environment variable.
	1) Enable/disable uboot logo:
	  When 	uboot environment variable logo  is not set or empty , the uboot logo feature is disabled.
	  when 	uboot environment variable logo 's vlaue contain  hdmi, ldb or lcd, the uboot logo feature is enabled.
	  etc. # setenv logo "hdmi:1920x1080M@60"   ## enable hdmi logo
		   # setenv logo ""		# disbale uboot logo		
		

	2) Set uboot display resolution:
		In general, the resolution is specified with enable/disable uboot logo.
		etc. # setenv logo "hdmi:1920x1080M@60"   ## enable hdmi logo , and resolution is "1920x1080M@60"
		The Supportted display resolution is defined in ${uboot_src}/common/disp_logo.c. 
		1920x1080M@60, 800x480M@60, 1024x768M@60 and so on.
	

	3) Store this setting:
	   To make uboot show logo, you should store this setting, then reset the system . It will not work bofore the system reset.
		# saveenv
		# reset



3.2  Enable/disable kernel keep logo:

		All is done by uboot environment variable!

		Kernel logo must keep same as the uboot logo, that is if uboot show logo on 1920x1080M@60 HDMI device, 
		kernel must show logo on 1920x1080M@60 HDMI device. This is important!

		Add the "logo" into video option to enable kernel keep logo. and remove 
		the "logo" from video option to disable kernel keep logo.
	
		The "bypass-reset" should also be added into video option when keep logo enabled. It means that IPU will be not reset.
		0 as IPU1 for LVDS, 1 as IPU2 for HDMI, and this is fixed. 

		1) Enable/disable HDMI keep logo:
		 a) Enter the uboot command line,  enable HDMI logo:
		# setenv bootargs console=ttymxc0,115200 init=/init video=mxcfb0:dev=hdmi,1920x1080M@60,if=RGB24,bpp=32,logo,bypass-reset=1  video=mxcfb2:dev=ldb,800x480M@60,if=RGB24,bpp=32  video=mxcfb3:off vmalloc=384M androidboot.console=ttymxc0 consoleblank=0  androidboot.hardware=freescale cma=384M androidboot.selinux=disabled androidboot.dm_verity=disabled	
		# saveenv 

		 b) Disable HDMI logo:
		When uboot enable the logo, then boot command line setting is: 
		# setenv bootargs console=ttymxc0,115200 init=/init video=mxcfb0:dev=hdmi,1920x1080M@60,if=RGB24,bpp=32,bypass-reset=1  video=mxcfb2:dev=ldb,800x480M@60,if=RGB24,bpp=32  video=mxcfb3:off vmalloc=384M androidboot.console=ttymxc0 consoleblank=0  androidboot.hardware=freescale cma=384M androidboot.selinux=disabled androidboot.dm_verity=disabled
		# saveenv 
		
		When uboot logo is disabled , then boot command line setting is: 
		# setenv bootargs console=ttymxc0,115200 init=/init video=mxcfb0:dev=hdmi,1920x1080M@60,if=RGB24,bpp=32  video=mxcfb2:dev=ldb,800x480M@60,if=RGB24,bpp=32  vmalloc=384M androidboot.console=ttymxc0 consoleblank=0 androidboot.hardware=freescale cma=384M androidboot.selinux=disabled androidboot.dm_verity=disabled	
		# saveenv 


		2) Enable/disable LVDS keep logo:	
		 a) Enter the uboot command line,  enable LVDS logo:
		# setenv bootargs console=ttymxc0,115200 init=/init video=mxcfb0:dev=ldb,800x480M@60,if=RGB24,bpp=32,logo,bypass-reset=0 video=mxcfb2:dev=hdmi,1920x1080M@60,if=RGB24,bpp=32 video=mxcfb3:off vmalloc=384M androidboot.console=ttymxc0 consoleblank=0 androidboot.hardware=freescale cma=384M androidboot.selinux=disabled androidboot.dm_verity=disabled
		# saveenv

		
		b) Disable LVDS logo:
		# setenv bootargs console=ttymxc0,115200 init=/init video=mxcfb0:dev=ldb,800x480M@60,if=RGB24,bpp=32  video=mxcfb2:dev=hdmi,1920x1080M@60,if=RGB24,bpp=32 video=mxcfb3:off vmalloc=384M androidboot.console=ttymxc0 consoleblank=0 androidboot.hardware=freescale cma=384M androidboot.selinux=disabled androidboot.dm_verity=disabled
		# saveenv	

		3) Enable/disable LCD keep logo:
		 a) Enter the uboot command line,  enable Lcd logo:
		# setenv bootargs console=ttymxc0,115200 init=/init video=mxcfb0:dev=lcd,1024x768M@60,if=RGB24,bpp=32,logo,bypass-reset=0 video=mxcfb2:dev=hdmi,1920x1080M@60,if=RGB24,bpp=32 video=mxcfb3:off vmalloc=384M androidboot.console=ttymxc0 consoleblank=0 androidboot.hardware=freescale cma=384M androidboot.selinux=disabled androidboot.dm_verity=disabled
		# saveenv

		
		b) Disable LCD logo:
		# setenv bootargs console=ttymxc0,115200 init=/init video=mxcfb0:dev=lcd,1024x768M@60,if=RGB24,bpp=32  video=mxcfb2:dev=hdmi,1920x1080M@60,if=RGB24,bpp=32 video=mxcfb3:off vmalloc=384M androidboot.console=ttymxc0 consoleblank=0 androidboot.hardware=freescale cma=384M androidboot.selinux=disabled androidboot.dm_verity=disabled
		# saveenv		


																Author: Qing
																Date: 2018/06/29		

