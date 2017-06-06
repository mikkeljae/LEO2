### A Simple Device Driver
This simple device driver does nothing more than writing to the kernel log, when inserted or removed from the kernel. 
#### The Code
```C
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>
 
MODULE_LICENSE("GPL"); 
MODULE_AUTHOR("Mikkel and Thomas");
MODULE_DESCRIPTION("A very simple Linux device driver.");
MODULE_VERSION("0.1");
 
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


Sections of the code will be explained here.

```C
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>
``` 
These headers are needed to define mark up functions, types, macros etc. for the kernel.

```C
MODULE_LICENSE("GPL"); 
MODULE_AUTHOR("Mikkel and Thomas");
MODULE_DESCRIPTION("A very simple Linux device driver.");
MODULE_VERSION("0.1");
```
Information about the kernel module. License type affects runtime behavior as any module loaded with a non-GPL-compatible license will mark the kernel as tainted.
```C
static int __init hello_init(void){
   printk(KERN_INFO "Hello World!, from your device.\n");
   return 0;
}

static void __exit hello_exit(void){
   printk(KERN_INFO "Goodbye World!, from your device.\n");
}
```
The kernel module initialization and cleanup functions writes to the kernel log.
```C
module_init(hello_init);
module_exit(hello_exit);
```
The macros are used to identify the initialization and cleanup functions. The actual names of the functions does not matter. 

#### Testing the Device Driver
Before inserting any modules we want to take a look at the kernel log. This can be done by using `dmesg`. The option 'w' allows for following the log and `H` enables user-friendly features such as colors and relative time:
```bash
dmesg -wH
```

A module can be inserted to the kernel using the command  `insmod`. This works great for simple modules like this one, but if you are developing a more advanced module, `modprobe` [should be used](https://linux.die.net/man/8/modprobe).
Open a new terminal and write: 
```bash
sudo insmod hello.ko
```

The module should now be inserted and you should be able to see this in the 'dmesg' terminal.

The module is removed using `rmmod`.
```bash
sudo rm hello
```

`modinfo` can be used to show information about a kernel module.
```bash
modinfo hello.ko
```

#### Some kind of ending