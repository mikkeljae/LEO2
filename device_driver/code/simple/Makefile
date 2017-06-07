export ARCH:=arm
export CROSS_COMPILE:=arm-xilinx-linux-gnueabi-

CC=$(CROSS_COMPILE)gcc

obj-m += simple.o

KDIR  := ../../linux-xlnx-zynmp-dt-fixes-for-4.10
WD	  := $(shell pwd)

default:
	${MAKE} -C ${KDIR} M=${PWD} modules

clean:
	${MAKE} -C ${KDIR} M=${PWD} clean