# Docker install

Make subvolumes for docker to keep root snapshots smaller
```
sudo mount /dev/sda2 /mnt -o subvol=/
cd /mnt/
sudo mkdir /var/lib/docker
sudo mkdir /data
sudo btrfs subvolume create @docker
sudo btrfs subvolume create @data
sudo nano /etc/fstab 
sudo mount -a
```

Now install ansible and docker. A daemon.json file is needed to tell docker to use the overlay driver not btrfs and to change network address pool.
```
sudo apt install python3-docker python3-pip
pip3 install ansible docker-compose

curl -fsSL https://get.docker.com -o get-docker.sh
sudo bash get-docker.sh
sudo usermod -aG docker $(whoami)

docker info
sudo nano /etc/docker/daemon.json
sudo systemctl restart docker
docker info

sudo reboot
```