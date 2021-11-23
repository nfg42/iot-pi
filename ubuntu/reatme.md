# Ubuntu server 64bit on Pi with USB SSD


## SSD Tweaks
https://forums.raspberrypi.com/viewtopic.php?f=131&t=278791

So Here are the steps:

1) Download the Ubuntu image for raspberry pi 4 form the Ubuntu official website.

2) Flash the image to a USB drive (USB 3.0 SSD or UBS flash drive).

3) Download the updated firmware files from the raspberry pi github site (https://github.com/raspberrypi/firmware ... aster/boot). Copy all *.dat and *.elf files to the Ubuntu boot partition. (Overwrite the files that were previously there).
NOTE: You must have MSD Booting EEPROM flashed onto your RPI4, or else this will not work!!! See: https://www.raspberrypi.org/documentati ... torageboot

Note: As of August 2020, you may not need to perform step 3. It has been reported that the current Ubuntu image contains the correct firmware files. I will leave this step here in case it does help someone. Be aware that it may not be necessary, but does not hurt to do anyway.

4) Decompress vmlinuz on the boot partition
```
zcat vmlinuz > vmlinux
```

5) Update the config.txt as follows for the [pi4] section:
```
[pi4]
max_framebuffers=2
dtoverlay=vc4-fkms-v3d
boot_delay
kernel=vmlinux
initramfs initrd.img followkernel
```
6) Add a new script to the boot partition called auto_decompress_kernel with the following:
```
#!/bin/bash -e

#Set Variables
BTPATH=/boot/firmware
CKPATH=$BTPATH/vmlinuz
DKPATH=$BTPATH/vmlinux

#Check if compression needs to be done.
if [ -e $BTPATH/check.md5 ]; then
	if md5sum --status --ignore-missing -c $BTPATH/check.md5; then
	echo -e "\e[32mFiles have not changed, Decompression not needed\e[0m"
	exit 0
	else echo -e "\e[31mHash failed, kernel will be compressed\e[0m"
	fi
fi

#Backup the old decompressed kernel
mv $DKPATH $DKPATH.bak

if [ ! $? == 0 ]; then
	echo -e "\e[31mDECOMPRESSED KERNEL BACKUP FAILED!\e[0m"
	exit 1
else 	echo -e "\e[32mDecompressed kernel backup was successful\e[0m"
fi

#Decompress the new kernel
echo "Decompressing kernel: "$CKPATH".............."

zcat $CKPATH > $DKPATH

if [ ! $? == 0 ]; then
	echo -e "\e[31mKERNEL FAILED TO DECOMPRESS!\e[0m"
	exit 1
else
	echo -e "\e[32mKernel Decompressed Succesfully\e[0m"
fi

#Hash the new kernel for checking
md5sum $CKPATH $DKPATH > $BTPATH/check.md5

if [ ! $? == 0 ]; then
	echo -e "\e[31mMD5 GENERATION FAILED!\e[0m"
	else echo -e "\e[32mMD5 generated Succesfully\e[0m"
fi

#Exit
exit 0
```

7) Make the script executable -- This is not actually needed as the boot partition is a fat32 partition. There is no executable bit to set.
```
sudo chmod +x auto_decompress_kernel
```
8) Create a script in the /ect/apt/apt.conf.d/ directory and call it 999_decompress_rpi_kernel. The script should contain the following:
```
DPkg::Post-Invoke {"/bin/bash /boot/firmware/auto_decompress_kernel"; };
```
9) Make the script executable
```
sudo chmod +x 999_decompress_rpi_kernel
```
Steps 8 and 9 can be done before first boot if you can mount the root file system on your computer, if you cannot, you can do them after you boot up the pi and log on to ubuntu for the first time. I recommend that you do it before first boot, but if you cannot, once you have the files in the correct place, please run the script auto_decompress_kernel from the /boot/firmware directory. If you do not, and you cannot boot after a restart, you will need to manually decompress your kernel again before this script will actually work (step 3).

10) Enjoy Ubuntu on the RPI4 without hassle :D

## First boot

### Cleanup
```
sudo snap remove lxd
sudo snap remove core18
sudo snap remove core20
sudo systemctl stop snapd
sudo systemctl stop snapd.socket 
sudo apt remove --purge snapd
rm -rf ~/snap/
sudo rm -rf /var/cache/snapd/ 
sudo apt autoremove
```

### Backup stuff
```
sudo apt install snapper
sudo snapper -c root create-config /
sudo nano /etc/snapper/configs/root 

sudo mkdir /.snapshots
sudo nano /etc/fstab 
#LABEL=writable	/.snapshots	 btrfs defaults,ssd,noatime,nodiratime,compress=zstd,subvol=@snapshot 0 0

sudo mount -a

sudo sudo chmod a+rx /.snapshots
sudo chmod 750 /.snapshots/
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer

sudo snapper create --description "test"

sudo mkdir /.snapshots/boot
nano backup-boot.sh 
sudo nano /boot/firmware/auto_decompress_kernel 
```
