## Snapper install

Snapper works on a raspberry pi with a few more tweaks. The main one is a default subvolume needs to be set and the root subvol can be removed from fstab and cmdline files.

```
btrfs subvolume set-default @
```

To mount the root btrfs volume use:
```
sudo mount /dev/sda2 /mnt -o subvol=/
```

To install snapper:
```
sudo mkdir /.snapshots
sudo apt install snapper
sudo snapper -c root create-config /
sudo nano /etc/snapper/configs/root
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer
```

To backup the boot partition I added a boot folder to snapshots
```
sudo mkdir /.snapshots/boot
```

Here is a basic boot backup script
```
#!/bin/bash

BTPATH=/boot/firmware
current_date=$(date +"%Y-%m-%d_%H%M%S")
tar -cvpzf /.snapshots/boot/backup-$current_date.tar.gz --one-file-system $BTPATH
```