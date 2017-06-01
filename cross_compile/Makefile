export ARCH:=arm
export CROSS_COMPILE:=arm-xilinx-linux-gnueabi-

obj-m := dummy_memory.o

KDIR  := /home/age/Desktop/ZYBO_LINUX/linux-xlnx-zynmp-dt-fixes-for-4.10
PWD		:= $(shell pwd)

default:
	$(MAKE) -C $(KDIR) -I$(KDIR)/include M=$(PWD) modules

#	$(MAKE) -C $(KDIR) -I$(KDIR) SUBDIRS=$(PWD) modules

clean:
	rm -rf *.o *.mod.* *.symvers *.order *~ *.cmd .tmp*

