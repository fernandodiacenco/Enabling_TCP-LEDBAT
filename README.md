# Enabling_LEDBAT-TCP
A how-to on how to enable TCP-LEDBAT on your Linux and Windows servers, a delay-based alternative congestion control that uses all available bandwidth while keeping latency down.

---

<b>INTRODUCTION</b>

This guide focuses on how to compile and load the congestion control algorithm LEDBAT as a kernel module.

And how to enable LEDBAT on a Standard Windows 2019 Server.


---

<b>CONFIGURATION</b>

On Linux you need a kernel version of 5.4 and later

On Windows you need at least Server 2019 for the official support, but it can be enabled on Server 2016 with a registry hack.


---

<b> WINDOWS SERVER 2019</b>

Open powershell console as administrator

Clean previous Transport Filters with:

    Remove-NetTransportFilter -SettingName *

Check if it removed previous filters with:

    Get-NetTransportFilter

You should see only the "Automatic" transport filter

Apply LEDBAT to all templates with:

    Set-NetTCPSetting -SettingName * -CongestionProvider LEDBAT

Check if its applied with:

    Get-NetTCPSetting

You should see CongestionProvider: LEDBAT in all templates (Internet, Datacenter, etc). That's it.


---

<b>DEBIAN 10</b>

You need at least Kenrel version 5.4 and later, check the running version with:

    sudo uname -a

Install Linux headers, build essential and git

    sudo apt install linux-headers-`uname -r` build-essential git

Get LEDBAT from github

    cd /usr/src && sudo git clone https://github.com/silviov/TCP-LEDBAT && cd TCP-LEDBAT

Compile the module

    sudo make -C "/usr/src/linux-headers-`uname -r`" M=/usr/src/TCP-LEDBAT/src modules

Load module into the Kernel to test

    sudo insmod /usr/src/TCP-LEDBAT/src/tcp_ledbat.ko

Check if module is loaded correctly

    sudo lsmod | grep ledbat

If the module was loaded correctly, make the changes persistent

    sudo nano /etc/sysctl.conf

Add this line at the end of the file, save and exit

    net.ipv4.tcp_congestion_control=ledbat

Apply the new configuration

    sudo sysctl -p

Copy module file into system

    sudo cp /usr/src/TCP-LEDBAT/src/tcp_ledbat.ko /lib/modules/`uname -r`

Add LEDBAT to startup modules list

    sudo nano /etc/modules

    Add tcp_ledbat to the end of the file, save and exit
    tcp_ledbat

Run Depmod

    sudo depmod

Restart the system

    sudo shutdown -r now

And check if the module was loaded correctly on startup with

    sudo cat /proc/sys/net/ipv4/tcp_congestion_control
    
If you see LEDBAT listed, that's it.

---

<b>CONCLUSION</b>

When comparing with the default Linux congestion control, CUBIC is optimized for throughput LEDBAT is optimized for low latency

Latency is a key metric when it comes to quality of experience for the end user

LEDBAT is designed to yield quickly to "common" tcp traffic when challenged as to focus on keeping latency down

Also LEDBAT, according to Wikipedia, is estimated to carry 13% to 20% of all intenet traffic.

---

<b>REFERENCES</b>

Article on LEDBAT - https://4sysops.com/archives/new-in-windows-server-2019-networking-http-2-cubic-ledbat-dpdk-and-kubernetes</br>
LEDBAT Github - https://github.com/silviov/TCP-LEDBAT</br>
LEDBAT article on Wikipedia - https://en.wikipedia.org/wiki/LEDBAT</br>
Other congestion control algorithms - https://en.wikipedia.org/wiki/TCP_congestion_control</br>
