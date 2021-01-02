# Disabled USB ports 

- ## Method 01

In order to see what is enable to wake up your system from suspend, take a look at ``/proc/acpi/wakeup``, like this:

``$ cat /proc/acpi/wakeup``

The result will look like:

```
Device  S-state   Status   Sysfs node
EHC1      S3    *disabled  pci:0000:00:1d.0
EHC2      S3    *disabled  pci:0000:00:1a.0
GLAN      S4    *enabled   pci:0000:08:00.0
USB7      S3    *disabled
WLAN      S3    *disabled  pci:0000:03:00.0
XHCI      S3    *disabled  pci:0000:07:00.0
```

In ``/etc/rc.local``
If file doesn`t exist make: 

```
printf '%s\n' '#!/bin/bash' 'exit 0' | sudo tee -a /etc/rc.local
sudo chmod +x /etc/rc.local
```

The file should look something like:
```
#!/bin/bash
echo EHC1 > /proc/acpi/wakeup
echo EHC2 > /proc/acpi/wakeup
echo XHCI > /proc/acpi/wakeup
exit 0
```

Reboot.



- ## Method 02

In order to see what is enable to wake up your system from suspend, take a look at ``/proc/acpi/wakeup``, like this:

``$ cat /proc/acpi/wakeup``

The result will look like:

```
Device   S-state     Status   Sysfs node
P0P1     S4   *disabled  pci:0000:00:01.0
P0P2     S4   *disabled 
P0P3     S4   *disabled 
P0P4     S4   *disabled 
GBE      S4   *enabled   pci:0000:00:19.0
BR20     S3   *disabled 
EUSB     S3   *enabled   pci:0000:00:1d.0
USBE     S3   *enabled   pci:0000:00:1a.0
PEX0     S4   *disabled  pci:0000:00:1c.0
BR21     S4   *disabled  pci:0000:02:00.0
PEX1     S4   *disabled 
PEX2     S4   *disabled 
PEX3     S4   *disabled  pci:0000:00:1c.3
PEX4     S4   *disabled 
PEX5     S4   *disabled 
PEX6     S4   *disabled 
PEX7     S4   *disabled 
PWRB     S3   *enabled
```

If you echo one of those "Device" abbreviation strings to /proc/acpi/wakeup, it toggles the state between enable/disabled.

So in my case I did (as root):

``$ printf "EUSB\nUSBE\n" > /proc/acpi/wakeup``

hat change however is not permanent.
You could write a small script though, call it "acpi_wakeup" and put it in "/etc/init.d":
```
#!/bin/sh
printf "EUSB\nUSBE\n" > /proc/acpi/wakeup
```
Make it executable:

``$ chmod 755 /etc/init.d/acpi_wakeup``

Then use 'update-rc.d' to make the required symbolic links automatically in other directories:

``$ update-rc.d acpi_wakeup defaults``
