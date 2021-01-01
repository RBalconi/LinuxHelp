# Enable Hibernate Linux Mint
### Tested in 31/12/2020

1. Introduction and preparations

Hibernation, also called "Suspend to Disk", is a variant to the regular suspend feature where the computer ends up completely powered off (unlike regular suspend which only keeps the system in a state of low power consumption), but starts up as if you had resumed it from a regular suspend. To achieve this, memory contents are written to disk and restored when you turn the system back on (which takes a bit longer than regular suspend).

Not all systems are able to successfully resume from hibernation, usually due to driver or BIOS issues. Therefore, before you continue, use the pre-installed Timeshift tool to create a system snapshot so the system can easily be restored to its previous state. Also ideally have a live USB/DVD at hand for easy restoring of said snapshot in case the system fails to boot.

2. Required swap file size

By default, the kernel writes a compressed hibernation image of a size up to 2/5 the size of your RAM. If your RAM is filled more than that, the surplus gets swapped before hibernation can occur. If there is not enough room in the swap file for the contents of your RAM plus whatever else you may have swapped out already, hibernation will fail.

There is no "safe" size, it's always a trade-off and depends on how much you usually use your RAM and swap space. As a rule of thumb, simply set up your swap file to at least the size of your RAM, or even double your RAM on systems with very low total RAM (since you are more likely to swap).

Check both your total RAM size as well as your current swap usage by running this command in a terminal window:

`free -h`

Important: All commands below assume that your swapfile is located at /swapfile, which is the installation default in LM19. They will not work if you set up a custom swap file location.

First check if your existing swap file meets the requirements laid out above by opening a terminal window and running this command:

`swapon`

If it is big enough and at the right location, you can skip straight to section 4 below.

3. Setting up the swap file

Otherwise, if the existing swapfile is too small or you do not have a swap file yet, we need to create a large enough one.

First we disable all swap space:

`sudo swapoff -a`

Then we set up the desired size of the swap file in full Gigibytes (GiB). For example, to set up a 4 GB swap file, you run this:

`SIZE=4`

Adjust the number according to your needs.

Now check that you've got enough free space available by running this command, the value under Avail should be considerably bigger than the size you configured above:

`df / -h`

If you do not have enough free space available, you cannot continue.

Otherwise run this (copy & paste as a whole into the same terminal window):

``sudo dd if=/dev/zero of=/swapfile bs=1M count=$(($SIZE * 1024))
sudo chmod 0600 /swapfile
sudo mkswap /swapfile
sudo sed -i '/swap/{s/^/#/}' /etc/fstab
sudo tee -a /etc/fstab<<<"/swapfile  none  swap  sw 0  0"``

This creates the swap file and configures the system to use it. It also disables any existing swap space you may have had set up already because that can lead to conflicts (it's possible to use multiple swap spaces but that's outside the scope of this guide).

4. Setting up the kernel parameters

Run this (copy & paste as a whole into a terminal window):

`RESUME_PARAMS="resume=UUID=$(findmnt / -o UUID -n) resume_offset=$(sudo filefrag -v /swapfile|awk 'NR==4{gsub(/\./,"");print $4;}') "`

followed by:

`if grep resume /etc/default/grub>/dev/null; then echo -e "\nERROR: Hibernation already configured. Remove the existing configuration from /etc/default/grub and add these parameters instead:\n$RESUME_PARAMS";else sudo sed -i "s/GRUB_CMDLINE_LINUX_DEFAULT=\"/GRUB_CMDLINE_LINUX_DEFAULT=\"$RESUME_PARAMS/" /etc/default/grub;fi`

If there was an error message, do what it tells you, otherwise run:

`sudo update-grub`

5. Adding Hibernate to the shutdown dialog

Run this (copy & paste as a whole into a terminal window):

``sudo tee /etc/polkit-1/localauthority/50-local.d/com.ubuntu.enable-hibernate.pkla <<'EOB'
[Enable hibernate]
Identity=unix-user:*
Action=org.freedesktop.login1.hibernate;org.freedesktop.login1.handle-hibernate-key;org.freedesktop.login1;org.freedesktop.login1.hibernate-multiple-sessions
ResultActive=yes
EOB``

6. Trying it out

Reboot the system, run this command in a terminal window to verify whether hibernation is generally possible:

`busctl call org.freedesktop.login1 /org/freedesktop/login1 org.freedesktop.login1.Manager CanHibernate`

The output should be "yes", otherwise you'll need to fix that before you can continue (see the Troubleshooting section below).

Now leaving the terminal window open, try to Hibernate from the shutdown menu, wait for the system to shut down and then turn it back on. If all worked correctly, your system will resume to the point that you hibernated from - the terminal window you opened earlier along with the command you ran should still be there.

You can also hibernate directly from a command line using this command:

`systemctl hibernate`

7. Troubleshooting

If your system says it does not support hibernation, it's likely due to your BIOS/UEFI. Boot to your BIOS/UEFI and check the settings, it needs to support ACPI sleep state S4. An UEFI's secure boot and fast boot features can also prevent hibernation in some cases, so disable them.

If you are using a non-default kernel it's also possible that it was compiled without hibernation support. To check:

`grep CONFIG_HIBERNATION /boot/config-$(uname -r)`

If your system does allow for hibernation, it can still fail either at the hibernation or more commonly at the resume stage. This can again be due to issues with the BIOS/UEFI, some of them have a faulty ACPI implementation, or due to issues with the hardware, including kernel drivers and firmware.

So update your BIOS, remove/disable as many devices as possible and try again, maybe that's already enough. Try out different kernels as well.

### For NVIDIA users:

In case of freezing after startup, make sure you have the updated video card drives.
