The following guide will help you get started with your freshly installed Linaro Linux.

***
# Getting Started
### Prerequisites
- Embedded platform with Linaro Linux 
- openssh-server installed on your host computer, [guide](https://help.ubuntu.com/lts/serverguide/openssh-server.html).

### Configuring IP addresses
The easiest way to persistently change to a static IP address of the target is by inserting the SD card in the host computer and changing the `/etc/network/interfaces` file to the following: 

```bash
auto eth0
iface eth0 inet static
address 192.168.10.1
netmask 255.255.255.0
gateway 192.168.10.2
```

Be sure to set your hosts IP address to `192.168.10.2`.

### Connection
Put the SD card back in your embedded platform, connect it to your host computer and determine the device by looking in the `/dev` folder.
Make a serial connection to it using socat: 

```bash
sudo socat - /dev/ttyUSB1,rawer,b115200,crnl
```

`rawer` makes terminal pass input and output almost unprocessed and turns off echo, `b115200` sets the baudrate to 115200 and `crnl` sets up correct line termination.

Now you should have a serial connection to your target device, where `bash` commands can be issued.

### Changing Shell 
As default the Linux installation uses `sh` shell. 
Many prefer the `bash` shell as it provides useful features such as tab completion and command history.
Changing to the `bash` shell is done and specifying it as login shell is done:

```bash
chsh -s /bin/bash
```

### Creating a new User

It is advised to run all commands as root, so we will create a new user and add it to the `sudo` group:

```bash
useradd -m -d /home/<user> <user>
passwd <user>
adduser <user> sudo
login <user>
```

### SSH

 `openssh-client` is installed on the device and the device can now be reached from your host computer by connecting the Ethernet cable (remember to have same IP address range) and running:

```bash
ssh <user>@192.168.10.1
```

Now you should have access to your platform.
Have fun :)