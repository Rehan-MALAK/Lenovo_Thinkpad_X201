***All of these below, worked for me, but may damage irremediably your computer. Use it at your own risk !***

### Repare Wifi (nothing in ''lspci'' and wifi disappearing randomly)

https://bugzilla.kernel.org/show_bug.cgi?id=91171

```
Dec 02 16:46:29 K kernel: iwlwifi 0000:03:00.0: Error sending REPLY_ADD_STA: time out after 2000ms.
Dec 02 16:46:29 K kernel: iwlwifi 0000:03:00.0: Current CMD queue read_ptr 143 write_ptr 144
Dec 02 16:46:29 K kernel: ------------[ cut here ]------------
Dec 02 16:46:29 K kernel: WARNING: CPU: 3 PID: 14198 at drivers/net/wireless/iwlwifi/pcie/trans.c:1552  iwl_trans_pcie_grab_nic_access+0xf2/0x100 [iwlwifi]()
Dec 02 16:46:29 K kernel: Timeout waiting for hardware access (CSR_GP_CNTRL 0xffffffff)
```

Open the laptop, clean it, remove Intel Centrino Wireless-N 1000 pci card and put it back.

I know this sounds crazy, but if you have the exact same kernel messages and if you have time to lose, just give it a try and you will see in Linux bugzilla that I'm not the only one !

### Upgrade BIOS with a already working Grub2 Linux

(only if you have a BIOS older than the last upgrade from Lenovo : 1.40-1.15 (2013))

Download the BIOS Update Bootable CD for Windows 8 (32-bit, 64-bit), 7 (32-bit, 64-bit), Vista (32-bit, 64-bit), XP - ThinkPad
Version : 1.40-1.15

https://download.lenovo.com/ibmdl/pub/pc/pccbbs/mobiles/6quj19us.iso

The documentation

https://download.lenovo.com/ibmdl/pub/pc/pccbbs/mobiles/6quj19uc.txt

shows that it's the last ever version

```
  Package        BIOS (BIOS ID)  ECP       (ECP ID)             Rev.  Issue Date
  -------------- --------------- -----------------------------  ----  ----------
  1.40-1.15/1.15 1.40 (6QET70WW) 1.15/1.15 (6QHT34WW/6SHT34WW)  01    2013/06/21
                                           (8VHT34WW)
```
Two methods that did not work or too complicated :

  - the "geteltorino (genisoimage Debian package) + F12 (boot on USB key)" method  => "Missing operating system" !
  - the "backup + new partition at the beginning of the hard disk method  with memdisk + bios upgrade" method => not needed !

Instead we will just tell grub2 where to find memdisk + bios upgrade program on your current Linux root partition. (Don't mixed up with the geteltorino-ed file otherwise you will have a "MEMDISK: image too big to load" or something close)

Just do :

```bash
sudo cp 6quj19us.iso /boot/images/bios.iso
sudo apt install syslinux-common
sudo cp /usr/lib/syslinux/memdisk /boot/images/
```

Edit ``/etc/grub.d/40_custom`` and put the correct root partition below (in Grub2 nomenclature, ie : ``'hd0,msdos3'`` for the linux ``/dev/sda3`` third partition of the first primary hard disk)

```bash
#!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.
menuentry "BIOS UPDATE" {
	insmod part_msdos
	insmod ext2
	set root='hd0,msdos3'
	linux16 /boot/images/memdisk.iso iso raw
	initrd16 /boot/images/bios.iso
}
```

``sudo update-grub2``

and reboot on the "BIOS UPDATE" choice in Grub2 Menu

(the Bios Upgrade Menu :

	2.Update system program

works

But the

	3. Update model number

didn't and we don't care !

### Repare Ethernet Intel e1000e(nothing in ''ip link'')

If you have nothing in ''ip link'' and your ''dmesg'' has some

```
e1000e: probe of 0000:00:19.0 failed with error -3
```

because of a failure of a hardware reset in the e1000e driver

or

```
e1000e 0000:00:19.0: The NVM Checksum Is Not Valid
e1000e: probe of 0000:00:19.0 failed with error -5
```

you probably have your NIC in a shitty state.

BIOS update, kernel module arguments, disable BIOS options ... nothing worked

https://askubuntu.com/questions/1158452/e1000e-error-3-and-unclaimed-network-interface
https://bbs.archlinux.org/viewtopic.php?id=185765
https://www.whtop.com/blog/e1000e-probe-failed-with-error-2/

I took a radical solution...
Rebuild e1000e driver and removing (commenting the code) everything that was not working : hw physical reset and NVM checksum...

and of course don't forget to update the initramfs

```bash
sudo cp e1000e.ko /lib/modules/5.4.0-4-amd64/kernel/drivers/net/ethernet/intel/e1000e/
sudo update-initramfs -k $(uname -r) -u -v
```

https://github.com/Rehan-MALAK/e1000e-3.6.0

probably a terrible idea ... but now my NIC works and I'm using it to push this comment on github...

On another level, Intel should probably try to make their Bootutil tool buildable on Debian (and compatible with linux-headers-$(uname -r)-common and linux-headers-$(uname -r)-amd64 and linux-kbuild-5.4)

https://downloadcenter.intel.com/download/29137?v=t

I will try this later.

### The XXI century laptop : coreboot

Some day...
