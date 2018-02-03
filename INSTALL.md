# WIP (work in progress)

Start installing Gentoo acording to the manual (x86 in my case [where pfSense dropped the ball on my ancient hardware {VIA C7-D Processor 1500MHz with 4 ethernet ports and some built in crypt muscle}])

At the time when you get to the kernel install "Default: Manual configuration" we change the step `root # emerge --ask sys-apps/pciutils` to:
- `emerge app-portage/eix` - not really necessary, but very helpful for searching packages
- `eix-update` - not applicable if you skip the step above
- `emerge app-portage/cpuid2cpuflags`
- `cpuinfo2cpuflags-x86 >> /etc/portage/make.conf` - I am not a big fan of guessing CPU Flags
- `emerge sys-apps/pciutils sys-apps/usbutils app-portage/gentoolkit`
- let's continue with the regular install... that's where we pick up at `root #cd /usr/src/linux` and `root #cd /usr/src/linux`

Now, before you run `make menuconfig`, run `lspci` and `lsusb` to grab the essentials for your configuration.
For example:
02:04.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL-8100/8101L/8139 PCI Fast Ethernet Adapter (rev 10)  
02:05.0 Ethernet controller: VIA Technologies, Inc. VT6120/VT6121/VT6122 Gigabit Ethernet Adapter (rev 11)  
02:06.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL-8100/8101L/8139 PCI Fast Ethernet Adapter (rev 10)  
02:07.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL-8100/8101L/8139 PCI Fast Ethernet Adapter (rev 10)  

As in my case, don't forget to disable the 64-bit support in `make menuconfig`, otherwise you end up with `error: CPU you selected does not support x86-64 instruction set`. Everybody else, make sure you have the 64-bit support enabled... don't want to hear about wasting resources.

Follow all the instructions of the manual until after `make && make modules install`.
This might take a while (one slow core in my use case)... grab a couple beers or a pizza, watch some episodes of your favorite TV show/DVD collection/DVR or go back to your smoker and check on them ribs/brisket... or go to bed and check the result in the morning... or whatever you like to do.

A couple steps later you wil read `Optional: Installing firmware`. Just install it.  
There is always a piece of hardware that might need it (NIC/SCSI/CPU/GPU/Audio/USB, you name it..)

When the time comes to install the logger as stated as ``, use `emerge app-admin/rsyslog` instead.

Now, after you managed to get your new setup booted and you are logged in, we install a webserver and php.
Let's get started with typing `ufed` and hit enter.
Within the 'Gentoo USE flags editor' type `php` to jump to the php use flag and hit the space bar to select it.
We will need to do the same to all these use flags: `cli fastcgi fpm`, 
...no worries, if some other USE-flags are missing, just fire up `ufed` again, add the flag and try to compile the package again.

Now would be a good time to get our system up-to-date, based on the changes USE-flags.
Type `emerge -uDNav --with-bdeps=y @world` on the command ine and hit enter.

...and again, based on your hardware, that might take a while!!!

