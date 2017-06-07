Cross compiling kernel modules is somewhat different to cross compiling user space software. In addition to using the correct compiler and compiling for the correct architecture, the compiler must also be told for which kernel the module should be created. Throughout this guide a makefile is created which will enable the reader to correctly compile kernel modules for the Zynq-7000 series platform. A small example module is given in this guide for the reader to try and verify the technique.

### 1. Prerequisites
* Kernel is built as described [here](https://github.com/SDU-Embedded/linux_zynq/wiki/Installing-Linux-on-the-ZYBO). Only steps 3 and 6 are required, but it is not possible to test the module without a working installation of the kernel.


### 2. A Simple Device Driver
This simple device driver does nothing more than write a line to the kernel log when inserted and removed from the kernel. 

```C
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>
 
MODULE_LICENSE("GPL"); 
MODULE_AUTHOR("Mikkel and Thomas");
MODULE_DESCRIPTION("A very simple Linux device driver.");
MODULE_VERSION("0.1415926");
 
static int __init hello_init(void){
   printk(KERN_INFO "Hello World!, from your device.\n");
   return 0;
}

static void __exit hello_exit(void){
   printk(KERN_INFO "Goodbye World!, from your device.\n");
}

module_init(hello_init);
module_exit(hello_exit);
```


Each section of the will be explained here.

```C
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>
``` 
These headers are needed to define functions, types, macros etc. for the kernel.

```C
MODULE_LICENSE("GPL"); 
MODULE_AUTHOR("Mikkel and Thomas");
MODULE_DESCRIPTION("A very simple Linux device driver.");
MODULE_VERSION("0.1");
```
Information about the kernel module. Usually the license should be GPL-compatible. If it is not, the kernel will be marked as tainted. This will often happen due to using proprietary drivers or similar software not supported by the community. Some software in the kernel may disable features if the kernel is marked as tainted. 
```C
static int __init hello_init(void){
   printk(KERN_INFO "Hello World!, from your device.\n");
   return 0;
}

static void __exit hello_exit(void){
   printk(KERN_INFO "Goodbye World!, from your device.\n");
}
```
The kernel defines an API that any kernel module should adhere to. This includes the kernel module initialization and cleanup functions. In this simple kernel module, all they do is write to the kernel log.
```C
module_init(hello_init);
module_exit(hello_exit);
```
These two macros are used to identify the initialization and cleanup functions. The actual names of the functions does not matter. 

With a kernel module in hand, it is now time to get it built!

### 3. Setting up the Environment
This guide assumes that a working installation of Vivado 2016.4 or later is installed on the host machine. Xilinx provides a number of compilers with the Vivado installation. Which one to use depends on the platform to be compiled for. In the case of the Linux environment for the [MicroZed](https://github.com/SDU-Embedded/linux_zynq/wiki/Installing-Linux-on-the-MicroZed) or [Zybo](https://github.com/SDU-Embedded/linux_zynq/wiki/Installing-Linux-on-the-ZYBO) available here at SDU-Embedded, `arm-linux-gnueabihf-gcc` should be used. In order to access this, source the following in a terminal:

```	
source path/to/vivado/2016.4/settings64.sh
```

Running this command will add a number of Xilinx related paths to `$PATH`, among them the path to all of the provided compilers. Retyping this command becomes tedious reasonably fast and the recommended approach is to add the above line to the `.bashrc` file with:

```
echo "source path/to/vivado/2016.4/settings64.sh" >> ~/.bashrc
```

The remainder of this guide is spent presenting a makefile which can be used for building kernel modules. In order to actually put it to the test a (very) small kernel module is made [available](https://github.com/SDU-Embedded/linux_zynq/tree/master/examples/cross_compile/kernel_module) in this guide.

### 4. Creating a Makefile
This makefile is also available in the [examples](https://github.com/SDU-Embedded/linux_zynq/tree/master/examples/cross_compile/kernel_module). This file serves as a good starting point for using kbuild to build kernel modules. The functionality of the makefile is explained below.

```
export ARCH:=arm
export CROSS_COMPILE:=arm-xilinx-linux-gnueabi-

CC=$(CROSS_COMPILE)gcc

obj-m += simple.o

KDIR  := path/to/linux-xlnx-zynmp-dt-fixes-for-4.10
PWD		:= $(shell pwd)

default:
	${MAKE} -C ${KDIR} M=${PWD} modules

clean:
	${MAKE} -C ${KDIR} M=${PWD} clean
```

Initially, it is necessary to export a few variables. The target architecture is set with the variable ARCH to the ARM architechture. After this the CROSS_COMPILE variable is set to tell make which compiler to use. As mentioned previously, in the case of the Zynq platform, arm-xilinx-linux-gnueabi- should be used. Finally, the compiler is set to gcc.

```
obj-m += simple.o
```

This line is the list of kernel modules to be built. The += assignment ensures that the operand is appended to the list, rather than overwriting the current content.

```
KDIR := path/to/linux-xlnx-zynmp-dt-fixes-for-4.10
PWD := $(shell pwd)
```

In order for kbuild to correctly build the module, it needs to know the location of the kernel headers of the kernel that the module is being built for. This information is given in the KDIR variable. The location of the built files is given in the PWD variable. Here, the shell keyword is used to execute a command in the shell. In this case pwd is called to print the current directory. For this reason make should always be called from the folder where the makefile is present.

```
default:
	${MAKE} -C ${KDIR} M=${PWD} modules

clean:
	${MAKE} -C ${KDIR} M=${PWD} clean
```

The format of these final lines and a few other details on make and its syntax is explained in more detail [here](https://github.com/SDU-Embedded/linux_zynq/wiki/Cross-Compiling-for-the-Zynq-7000-Series-Platform#creating-a-makefile). The default rule calls make with the -C flag. This flag changes directory to, in this case, KDIR, the directory where the linux header files are present. M= lets kbuild know that an external module is being built and the given argument is the directory that the module is placed in. In this case the working directory. modules, while not strictly necessary, serves as a placeholder for the target.

Finally, the clean recipe will use kbuild to remove any files generated by the build process.

Using the Makefile can be done by simply calling `make` from the directory where `simple.c` is placed and calling `make clean` will empty the folder of all files but `Makefile` and `simple.c`.

#### 5. Transferring the Kernel Module
The kernel module needs to be transferred to the target. This can be done using `scp`, which a secure way of transferring files based on `ssh`:

```bash
scp simple.ko <user>@192.168.10.1:~/<path>
```

### 6. Testing the Device Driver
Before inserting any modules we want to take a look at the kernel log. This can be done by using `dmesg`. The option 'w' allows for following the log and `H` enables user-friendly features such as colors and relative time:
```bash
dmesg -wH
```

A module can be inserted to the kernel using the command  `insmod`. This works great for simple modules like this one, but if you are developing a more advanced module, `modprobe` [should be used](https://linux.die.net/man/8/modprobe).
Open a new terminal and write: 
```bash
sudo insmod simple.ko
```

The module should now be inserted and you should be able to see this in the 'dmesg' terminal.

The module is removed using `rmmod`.
```bash
sudo rm simple
```

`modinfo` can be used to show information about a kernel module.
```bash
modinfo simple.ko
```

You should now be ready to develop your own device drivers for the Zynq 7000 Series Platform. Have fun :)


***
Maintainers: 

Thomas Søndergaard Christensen <thomc12@student.sdu.dk> 

Mikkel Skaarup Jaedicke <mijae12@student.sdu.dk>