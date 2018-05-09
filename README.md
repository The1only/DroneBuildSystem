# DroneBuildSystem
Solo GameChanger:

g_mulit.ko for Solo, and Controller…

It might already exist, and be old news, but for me it took quite some days to get working, so I’ll save you the time.
I am able to compile the software on the Solo and Controller, and thus also modify it, and make new sd card images etc. 
And I am able to compile kernel drivers that will actually load on the latest solo linux.
I do this on my Linux computer, after some struggle, I do not use the vagrant method, I needed more flexibility.
All this from the official code on the 3DR Solo and OpenSolo repository.

Why:
The Solo’s AUX connector got USB, but on the software side, is quite limited. 
You get one virtual serial port and a file system (a disk, when connected to a Linux PC).
However what I want is g_mulit witch add an rndis (ethernet over usb). 
This will allow me to connect a Raspberry Pi Zero or a PC eq. to the bay connector and then get a full network connection.
I verified that I get 10MBytes/s eq. to  80Mbit/s sustainable transfer between my PC and the Solo I.mx6 board over the AXU bay.
The routing on the AUX board and maybe also the Solo is not optimised for 480Mbit/s burst speeds, however I have not seen any issues.

The rndis part generate a /dev/usb0 ethernet connection, this will act as any other ethernets, and you can do tcp/ip. 
or udp or even forward net to the ground station and to the internet.
The goal is to send data from the Raspberry PI to the cloud using the Solo radio link, 
as using the Raspberry Pi w-lan will totally mess up the radio system and range… 

I will enable USB also on the controller and add a USB stick LTE cell phone modem directly to it. 

Just as an example


How:

1) Download the drivers from:
	git clone https://github.com/The1only/DroneBuildSystem.git
	This is a fully unmodified version, all I did was to compile it, all the code can be found in the 3DR Solo and OpenSolo repository 

2) Copy the g_mulit.ko to this catalog on the Solo or the Controller:
       /lib/modules/3.10.17-rt12-1.0.0_ga+g4176d57/kernel/drivers/usb/gadget

3) run:  depmod -a -v
	on the Solo or Controller

4) run:  nano /etc/modules
        Change the work g_acm_ms.ko.  to g_multi.ko

5) reboot

6) You now got g_multi, from here you have to set up the usb0 network.
   	I just did this manually for now with: ifconfig usb0 192.168.1.10
	There are several ways to do this, maybe the best is to use the g_mulit parameters.


I also got the pl2303 driver should any one need on, this will enable some Arduino boards and other utilities etc. that uses the Prolific USB solution.

Driver list:

./usb/serial/zte_ev.ko
./usb/serial/usb_wwan.ko
./usb/serial/ftdi_sio.ko
./usb/serial/pl2303.ko
./usb/serial/option.ko
./usb/serial/usbserial.ko
./usb/gadget/g_ether.ko
./usb/gadget/g_webcam.ko
./usb/gadget/g_acm_ms.ko
./usb/gadget/usb_f_acm.ko
./usb/gadget/libcomposite.ko
./usb/gadget/g_multi.ko
./usb/gadget/u_serial.ko
./usb/class/cdc-acm.ko
./usb/class/cdc-wdm.ko
./mxc/mlb/mxc_mlb150.ko
./i2c/algos/i2c-algo-pcf.ko
./i2c/algos/i2c-algo-pca.ko
./media/v4l2-core/videobuf2-memops.ko
./media/v4l2-core/videobuf2-vmalloc.ko
./media/v4l2-core/videobuf2-core.ko
./media/platform/mxc/capture/ipu_bg_overlay_sdc.ko
./media/platform/mxc/capture/fsl_csi.ko
./media/platform/mxc/capture/ipu_still.ko
./media/platform/mxc/capture/adv7610_video.ko
./media/platform/mxc/capture/csi_v4l2_capture.ko
./media/platform/mxc/capture/ipu_prp_enc.ko
./media/platform/mxc/capture/ipu_csi_enc.ko
./media/platform/mxc/capture/mxc_v4l2_capture.ko
./media/platform/mxc/capture/ipu_fg_overlay_sdc.ko


To be on the safe side, this is how I do it:


On an empty Ubuntu 14.4 in the home directory of the user vagrant:
mkdir solo-build-alt
git clone https://github.com/OpenSolo/solo-builder
sudo ln -s /home/vagrant/solo-builder/ /vagrant
sudo ln -s /home/vagrant/solo-build-alt/ /solo-build-alt
cd ./solo-builder
sudo apt-get install gawk chrpath
sudo apt-get install texinfo
./builder.sh 
DONE !

Build Solo Single code:
cd solo-build-alt/
export MACHINE=imx6solo-3dr-1080p
EULA=1 source ./setup-environment build
bitbake -c devshell virtual/kernel

... You now got a buld shell ...

cd ./tmp-eglibc/work/imx6solo_3dr_1080p-oe-linux-gnueabi/linux-imx/3.10.17-r0/git
make menuconfig 
    change whant you need
make modules

PS: If you change anything outside the modules you must rebuld the entire kernel.
That will set you back days as then you need to use the bitbake/yocto methods to get is all right.
Rebuld the enfire file system and reflash the SD card using dd or similar...

To make custom make modules:

make M=/home/user/my_module modules

Given that you got the Makefile and soruce code right:

in the Makefile:  
obj-m += my_module.o

in the my_module.c:
#include <linux/module.h>
#include <linux/init.h>
#include <linux/kernel.h>

static int __init my_init(void)
{
    printk(KERN_INFO "hello, my module\n");
    return 0;
}

static void __exit my_exit(void)
{
    printk(KERN_INFO "good bye, my module\n" );
}

module_init(my_init);
module_exit(my_exit);

MODULE_LICENSE("Dual BSD/GPL");
MODULE_AUTHOR("T.N.");
MODULE_DESCRIPTION("My First Driver");


