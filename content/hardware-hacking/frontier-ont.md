+++
title = 'Frontier ONT FOG421'
date = 2024-07-14T09:58:51-07:00
draft = true
+++

The target for this project is a Frontier Optical Network Terminal, Model FOG421. The initial goal was to dump the firmware and obtain a root shell, but I ended up going much farther. 

### 1. Hardware analysis

![](https://raw.githubusercontent.com/redawl/firmware-dumps/main/frontier-FOG421/images/inside.jpg)

The most interesting I see when I looked inside was some kind of header with 5 pins in the bottom right. This is unusual, since I am used to seeing 4 pin headers on modems in the past. 

With a little dinking around, I was able to determine that the middle pin was ground, so I hooked up my Logic analyzer to the pins, making sure to connect the ground pins correctly. I then powered on the ONT. 

[[ Image here ]]

I see lots of activity on pin 1, which means it's is probably the TX pin, if we are dealing with UART. I added an Async Serial analyzer to pin1, setting the baud rate to 115200, which in my experience it the most common rate, so it's a good first guess. Leaving everything else as default, I applied the analyzer, and we see U-Boot!

### 2. Exploring the shell

Now that we have found GND and TX, we need to figure out which of the remaining 3 pins is RX. For this, I used my Bus Pirate. After some trial and error (moving the TX cable on the bus pirate to each pin and testing for response to keypresses), I was able to identify that the pin on the other side of GND was in fact RX. I then found myself at a prompt I had never seen before:

```shell
ONT> 
```

None of the usual help commands appeared to do anything:

```shell
ONT>help
ONT>?
ONT>ls
ONT>show help
```

Time to search the internet. After a little searching I found an [article](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst_pon/software/command_reference/b-gpon-olt-cr/ont_device_configuration.html) which set me on the right path: 

```shell
ONT>enable
#ONT>help
  Description: CLI Root
    +traffic             Service CLI menu
    +system              System CLI menu
#ONT>
```

After reading some help output and trying different commands, I found:

```shell
#ONT>system
#ONT/system>shell
#ONT/system/shell>ls
bin      etc      linuxrc  root     tmp      var
bootimg  home     mnt      sbin     uImage   web
dev      lib      proc     sys      usr
#ONT/system/shell>
```

Progress! A linux commandline, with access to the filesystem.

### Dumping bootloader and firmware

I struggled to find an easy way to exfiltrate data from the limited environment on the device, but I finally was able to exfiltrate data by setting up networking over and ethernet cable and manually assigning ips to both ends, and then using ftp:

On my PC:

```bash
# ip addr add 192.168.100.2/24 dev eth0
```

On the ONT:

```ash
#ONT/system/shell> ip addr add 192.168.100.3/24 dev eth0
```

Setting up ftp server on my PC:

```bash
$ python3 -m pyftpdlib -w
```

Extracting bootloader and firmware from ONT:

```ash
#ONT/system/shell>ftpput -P 2121 192.168.100.2 uImage /uImage
#ONT/system/shell>cat /proc/partitions
major minor  #blocks  name

  31        0        768 mtdblock0
  31        1       2048 mtdblock1
  31        2      28672 mtdblock2
  31        3      28672 mtdblock3
  31        4       3072 mtdblock4
  31        5        768 mtdblock5
  31        6       2048 mtdblock6
  31        7      14336 mtdblock7
  31        8      14336 mtdblock8
  31        9       1024 mtdblock9
  31       10       1024 mtdblock10
  31       11       3072 mtdblock11
  31       12      22528 mtdblock12
  31       13       8192 mtdblock13
#ONT/system/shell>for i in $(seq 0 13); do ftpput -P 2121 192.168.100.3 mtdblock${i} /dev/mtdblock${i}; done
```
