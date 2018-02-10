# WIP (work in progress)

*Start installing Gentoo acording to the manual (x86 in my case [where pfSense dropped the ball on my ancient hardware {VIA C7-D Processor 1500MHz with 4 ethernet ports and some built in crypt muscle}])*

At the time when you get to the kernel install "Default: Manual configuration" we change the step

```bash
root # emerge --ask sys-apps/pciutils
```
to:

Not really necessary, but very helpful for searching packages
```bash
root # emerge app-portage/eix
```
Not applicable if you skip the step above
```bash
root # eix-update
```
Nice tool to figure out your CPU flags
```bash
root # emerge app-portage/cpuid2cpuflags
```
And since I am not a big fan of guessing CPU Flags
```bash
root # cpuinfo2cpuflags-x86 >> /etc/portage/make.conf
```
Almost back to normal
```bash
root # emerge sys-apps/pciutils sys-apps/usbutils app-portage/gentoolkit app-portage/ufed
```
Let's continue with the regular install... that's where we pick up at
```bash
root #cd /usr/src/linux
```
Now, before you run
```bash
root # make menuconfig
```
run
```bash
root # lspci
```
and
```bash
root # lsusb
```
to grab the essentials for your configuration.

## For example:
>00:00.0 Host bridge: VIA Technologies, Inc. CX700/VX700 Host Bridge (rev 03)
>00:00.1 Host bridge: VIA Technologies, Inc. CX700/VX700 Host Bridge
>00:00.2 Host bridge: VIA Technologies, Inc. CX700/VX700 Host Bridge
>00:00.3 Host bridge: VIA Technologies, Inc. CX700/VX700 Host Bridge
>00:00.4 Host bridge: VIA Technologies, Inc. CX700/VX700 Host Bridge
>00:00.7 Host bridge: VIA Technologies, Inc. CX700/VX700 Host Bridge
>00:01.0 PCI bridge: VIA Technologies, Inc. VT8237/VX700 PCI Bridge
>00:0f.0 IDE interface: VIA Technologies, Inc. VX800 Serial ATA and EIDE Controller
>00:10.0 USB controller: VIA Technologies, Inc. VT82xx/62xx UHCI USB 1.1 Controller (rev 90)
>00:10.1 USB controller: VIA Technologies, Inc. VT82xx/62xx UHCI USB 1.1 Controller (rev 90)
>00:10.2 USB controller: VIA Technologies, Inc. VT82xx/62xx UHCI USB 1.1 Controller (rev 90)
>00:10.4 USB controller: VIA Technologies, Inc. USB 2.0 (rev 90)
>00:11.0 ISA bridge: VIA Technologies, Inc. CX700/VX700 PCI to ISA Bridge
>00:11.7 Host bridge: VIA Technologies, Inc. CX700/VX700 Internal Module Bus
>00:13.0 Host bridge: VIA Technologies, Inc. CX700/VX700 Host Bridge
>00:13.1 PCI bridge: VIA Technologies, Inc. CX700/VX700 PCI to PCI Bridge
>01:00.0 VGA compatible controller: VIA Technologies, Inc. CX700/VX700 [S3 UniChrome Pro] (rev 03)
>02:04.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL-8100/8101L/8139 PCI Fast Ethernet Adapter (rev 10)
>02:05.0 Ethernet controller: VIA Technologies, Inc. VT6120/VT6121/VT6122 Gigabit Ethernet Adapter (rev 11)
>02:06.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL-8100/8101L/8139 PCI Fast Ethernet Adapter (rev 10)
>02:07.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL-8100/8101L/8139 PCI Fast Ethernet Adapter (rev 10)
>02:08.0 FireWire (IEEE 1394): VIA Technologies, Inc. VT6306/7/8 [Fire II(M)] IEEE 1394 OHCI Controller (rev 80)
>80:01.0 Audio device: VIA Technologies, Inc. VT8237A/VT8251 HDA Controller (rev 10)

With the above info (from your system) you should be able to track down the hardware you need to compile into your kernel.

As in my case, don't forget to disable the 64-bit support in `make menuconfig`, otherwise you end up with `error: CPU you selected does not support x86-64 instruction set`.
Everybody else, make sure you have the 64-bit support enabled... don't want to hear about wasting resources.
Yes, I know, ancient hardware, but I always enjoy a good challenge.

Follow all the instructions of the manual until after `make && make modules install`.
This might take a while (one slow core in my use case)... grab a couple beers or a pizza, watch some episodes of your favorite TV show/DVD collection/DVR or go back to your smoker and check on them ribs/brisket... or go to bed and check the result in the morning... or whatever you like to do.

A couple steps later you wil read `Optional: Installing firmware`. Just install it. There is always a piece of hardware that might need it (NIC/SCSI/CPU/GPU/Audio/USB, you name it..)

When the time comes to install the logger, we `don't` do
```bash
root # emerge --ask app-admin/sysklogd
```
Instead we install rsyslog, in case we want to send our logs to a log server
```bash
root # emerge app-admin/rsyslog
```

Now, after you managed to get your new setup booted and you are logged in, we install a webserver and php.
Let's get started
```bash
root # ufed
```

Within the 'Gentoo USE flags editor' type `php` to jump to the php use flag and hit the space bar to select it.
We will need to do the same to all these use flags: `cli fastcgi fpm`.
...no worries, if some other USE-flags are missing, just fire up `ufed` again, add the flag and try to compile the package again.

Now would be a good time to get our system up-to-date, based on the changes USE-flags.
```bash
root # emerge -uDNav --with-bdeps=y @world
```
on the command line and hit enter.

...and again, based on your hardware, that might take a while!!!

Finally we can install some php
```bash
root # emerge dev-lang/php
```

I know, I know... what about the other USE-flags we need?
We get there, while we build the system!
This document is still work in progress and will change over time.

Let's take a look
```bash
root # php -v
PHP 7.1.13 (cli) (built: Feb  3 2018 11:17:29) ( NTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2017 Zend Technologies
    with Zend OPcache v7.1.13, Copyright (c) 1999-2017, by Zend Technologies
```
...good, now to the web server.

Let's get Nginx installed (small footprint and fast).
```bash
root # emerge www-servers/nginx
```

Once Nginx is installed, we need to modify `/etc/nginx/nginx.conf`
Find the section that looks similar to this:
```bash
server {
                listen 127.0.0.1;
                server_name localhost;
}
```

and change the IP address to `0.0.0.0`, otherwise we will get no connection.

Let's find out if we can start Nginx and if it responds.
```bash
root # /etc/init.d/nginx start
```

Open a web browser (on a different machine/phone) and browse to
[http://<myfirewall_or_ipaddress>](http://localhost) <- change `myfirewall` to whatever your DNS understands or the IP address you used.
As long as we see something that looks like a regular web server response with a 404 from nginx, we are good.

Let's find out if we can get something else on the browser
```bash
root # cd /var/www/localhost
root # mkdir htdocs
root # cd htdocs
root # echo "Hello, world!" > index.html
```
Now back to your browser and refresh. You should see `Hello, world!`.
Perfect!

The next check will fail, but we try anyway...
```bash
root # printf "<?php\n    phpinfo();\n" > index.php
```
Now we try [http://<myfirewall_or_ipaddress>/index.php](http://localhost/index.php) in the browser.
Correct - epic fail! You probably just dowloaded the index.php file...

Let's get that fixed...

Open `/etc/nginx/nginx.conf` in your editor of choice and change the server section.
```
server {
                listen 0.0.0.0;
                server_name localhost;

                access_log /var/log/nginx/localhost.access_log main;
                error_log /var/log/nginx/localhost.error_log info;

                root /var/www/localhost/htdocs;

                location ~ .php$ {
                        fastcgi_pass 127.0.0.1:9000;
                        include fastcgi.conf;
                }
        }
```
Now that that is taken care of, start and restart some more services.
```
root # /etc/init.d/nginx restart
root # /etc/init.d/php-fpm start
```
Back to the browser and refresh.

There you have it

In order to start Nginx and php during startup
```bash
root # rc-update add nginx
root # rc-update add php-fpm
```

Now I will get some basic code written and come back to you when I have something to show.

In case you get bored...
```bash
root # eix-sync
root # emerge -uDNav --with-bdeps=y @world
```

A new week, a couple more hours I could spend on this little project!
I forgot about the importance of `sudo`. A lot of settings and command executions are more or less impossible without the help of `sudo`, unless everything runs under `root` - bad idea.

It is safe to say that we can install `sudo` without messing with the USE-Flags, for now.
To do so:
```bash
root # emerge app-admin/sudo
```
With instaling `sudo`, we introduced a new file to `/etc` called `sudoers`.
Within this file we can get all creative. However, we try to be as conservative as we can.
Therefore, let's switch gears real quick.
We know that all the commands we need to run are executed via PHP...
Good, let's change the PHP-FPM user from `nobody` to something more meaningful, let's say `tapog` !?!?

What do we need to do to achive that?

Oh, before I forget, here is another good companion to install:
```bash
root # emerge app-misc/mc
```
This is not neccessrly for production, but helps during the `dev` stage.

Ok, back to `sudo` and name changes...
Assuming you settled on `tapog`:
```bash
root # groupadd tapog
root # useradd tapog -g tapog
```
That should fix the first problem.  
So, how do we get php to run under the new user `tapog`?  
Open `/etc/php/fpm-php7.1/fpm.d/www.conf` in your editor of choice and change
```bash
; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;   will be used.
user = nobody
group = nobody

; The address on which to accept FastCGI requests.
```
to
```bash
; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;   will be used.
user = tapog
group = tapog

; The address on which to accept FastCGI requests.
```
To make sure it worked
```bash
root # /etc/init.d/php-fpm restart
```
```bash
root # ps aux | grep pgp
tapog    30353  0.0  0.9 157160  4688 ?        S    22:08   0:00 php-fpm: pool www
tapog    30354  0.0  0.9 157160  4688 ?        S    22:08   0:00 php-fpm: pool www
```
Sounds better than `nobody`, don't you think?
