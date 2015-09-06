# Surface Pro 3
This is a work in progress and will be obsolete at some point when all changes make it upstream.

I have not disabled TPM or SecureBoot.  I have both enabled and have them enabled during install, reboot, etc.  No need any longer to disable them, if using Ubuntu 15.04 which has support for secure boot and key management.

!!!Disclaimer!!!
I do not dual boot.  I used to about 15 years ago back in 2000ish but found I always booted into one OS or the other and rarely if ever booted into both on the same day.  So, I no longer dual boot.  I am currently running Ubuntu Gnome 15.04 nightly CD and erased everything on the disk and performed a fresh installation.  With VMware, VirtualBox, KVM, etc, I find no reason to dual boot any longer, just run your Windows/OSX apps in a VM.  Thus, I cannot help with dual boot issues.

Most of this work is not my own, rather it is a collection of patches and instructions to simplify running Linux on Surface Pro 3.

# Ubuntu (15.10) Kernel
I have just installed Ubuntu Gnome 15.10 Beta and have mixed results.

The kernel is 4.2.0-7 Ubuntu.  It seems to work well with only the following issues:
* Touchpad didn't work (resolved by adding the device section at the end of the evdev.conf file)
* Buttons don't work (no hotkeys or sound buttons)
* Touchpad two-finger anything does not work but otherwise it's functional after change above
* It's Beta so it's bound to be broken but damn it is working better than I thought it would

Here is the good news:
* Wifi is rock solid
* Power management works, what?  Yes, suspend works.

# Ubuntu (15.04) Kernel
A [Google Group](https://groups.google.com/forum/?hl=en#!forum/linux-surface) has also been created by another enthusiast to use as a discussion board for issues/successes when using Linux on SP3.  This group, specifically the first post, is where I received much of this information, along with kernel bug [84651](https://bugzilla.kernel.org/show_bug.cgi?id=84651)

- battery.patch - patch to allow the battery to be enumerated and displays accurate capacity
- cam.patch - camera patch
- buttons1.patch - adds support for power, home, volume buttons
- buttons2.patch - helps with sleep fix

## What doesn't work
* suspend - it's flaky and resumes quickly
* trackpad - it's registered as a mouse but is usable IMHO

## Binary pre-built debs
As I build new kernels, I'll place them in [Google Drive](https://drive.google.com/folderview?id=0BzNI3Zdy9Y6kfklBazc5Y3VQXzd6MU1oaUFMS0NxWEI4dmpFRmFITWZFZWpfM0U1dUJJaTQ&usp=sharing)

## Get the kernel
From [Ubuntu Kernel Git Guide](https://wiki.ubuntu.com/Kernel/Dev/KernelGitGuide?action=show&redirect=KernelTeam%2FKernelGitGuide)

```
sudo apt-get install git
mkdir ~/source && cd ~/source
git clone git://kernel.ubuntu.com/ubuntu/ubuntu-vivid.git
```

## Apply the patches and Build
Copy or download the above patches and apply each of them with:
```
patch -p1 --ignore-whitespace -i {patch}
```

Before building the kernel, I update the version number to avoid software update from constantly overwriting (or attempting to) my kernel.  Simply edit ubuntu-vivid/debian.master/changelog and increment the version number by 1.
```
example: 3.19.0-13.13 -> 3.19.0-13.14
```

Build your new kernel with:
```
fakeroot debian/rules clean
DEB_BUILD_OPTIONS=parallel=4 AUTOBUILD=1 NOEXTRAS=1 fakeroot debian/rules binary-generic
```

## Install the kernel
**I have found that sometimes after touching grub, the first boot may take a few minutes.  DO NOT STOP IT, let the system boot.  Afterwards, should be fast again.**
Also, always remember to run grub-install after anything touches/updates grub.

Install using:
```
cd .. && dpkg -i linux-headers* linux-image*
sudo grub-install
```

## Fix the Type Cover Trackpad

Add the following at the end of /usr/share/X11/xorg.conf.d/10-evdev.conf
```
Section "InputClass"
    Identifier "Surface Pro 3 cover"
    MatchIsPointer "on"
    MatchDevicePath "/dev/input/event*"
    Driver "evdev"
    Option "vendor" "045e"
    Option "product" "07dc"
    Option "IgnoreAbsoluteAxes" "True"
EndSection
```

## Recover from grub prompt or UEFI loop
If you find yourself in a loop where the SP3 continues to enter UEFI, do not fret, simply insert your ubuntu 15.04 install USB.

If your system will not boot beyond grub, no worries, do this at the grub prompt:
```
"press 'c' to enter command line"
grub> set root=(hd1,2)
grub> linux /vmlinuz root=/dev/sda2
grub> initrd /initrd.img
grub> boot
```

Enjoy!

My current kernel:
```
Linux reeps-sp3 3.19.0-14-generic #15 SMP Wed Apr 15 09:41:49 PDT 2015 x86_64 x86_64 x86_64 GNU/Linux
```
