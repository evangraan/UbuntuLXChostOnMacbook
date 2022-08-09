# Installing Ubuntu on the macbook

## USB installation disk
- Use a USB stick, at least 4Gb in size
- On the macbook, instert the USB stick and Erase it using disk utility
- Select MS-DOS FAT as the file system
- Download open-source app balenaEtcher and install on the macbook
- Download an Ubuntu 20.04 LTS iso
- Use etcher to flash the USB drive with the iso image
- Note: the USB drive will be unreadable by OSX. Do not initialize it when OSX asks you to. If you need to flash it again, use Etcher on OSX. It will work.

## Ubuntu installation
- Insert the USB disk in the macbook
- Reboot the macbook, holding down the option key during boot
- Select the correct EFI instance to boot from the USB
- Ensure that the full disk is utilized
- Install Ubuntu
- Reboot the macbook

## Access
```
ssh-keygen
vi .ssh/authorized_keys
```

Paste a client machine's public key into authorized_keys

```
chmod 0700 .ssh/authorized_keys
```

Test login without password.

## Install LXC

```
cd /usr/local/src/
sudo apt-get install lxc aptitude wget unzip zip build-essential

```

## Configuring LXC

```
echo "$(id -un) veth lxcbr0 10" | sudo tee -a /etc/lxc/lxc-usernet
mkdir ~/.config/lxc 
cp /etc/lxc/default.conf ~/.config/lxc/default.conf
MS_UID="$(grep "$(id -un)" /etc/subuid  | cut -d : -f 2)"
ME_UID="$(grep "$(id -un)" /etc/subuid  | cut -d : -f 3)"
MS_GID="$(grep "$(id -un)" /etc/subgid  | cut -d : -f 2)"
ME_GID="$(grep "$(id -un)" /etc/subgid  | cut -d : -f 3)"
echo "lxc.idmap = u 0 $MS_UID $ME_UID" >> ~/.config/lxc/default.conf
echo "lxc.idmap = g 0 $MS_GID $ME_GID" >> ~/.config/lxc/default.conf
echo 'export DOWNLOAD_KEYSERVER="hkp://keyserver.ubuntu.com"' >> ~/.bashrc
```

If you run into permissions errors on ```~/.local``` starting a container, do:
```
cd ~/
setfacl -m u:100000:x . .local
mkdir .local
sudo chmod -R ugo+x .local
```

## Dynamic DNS
Install noip:

```
sudo apt install build-essential
cd /usr/local/src/
sudo wget http://www.noip.com/client/linux/noip-duc-linux.tar.gz
sudo tar xf noip-duc-linux.tar.gz
sudo rm noip-duc-linux.tar.gz
cd noip-2.1.9-1/
sudo make install
```

Enter the NoIP credentials

## Port forwarding

If you'd like to access services in the containers via the host's public IP, add port forwarding as per https://www.digitalocean.com/community/tutorials/how-to-forward-ports-through-a-linux-gateway-with-iptables

## Shared storage with the host

You can share a directory on the host with a container by editing the container's configuration. First, create the storage directory to be sheared. The mount point will be mounted with user nobody. To ensure write access to the directory:

```
mkdir -p /path/to/storage
chmod -R o+w /path/to/storage
```

Stop the container and edit its configuration:

```
./stop_container.sh NAME
vi .local/share/lxc/NAME/config
```

Add a mount point:

```
lxc.mount.entry = /path/to/storage storage none bind,create=dir 0.0
```

Then start the container

```
./start_container.sh NAME
```

## Install synergy (if Ubuntu desktop was installed)

- Download the correct synergy client from https://symless.com/synergy/download
```
sudo apt install aptitude
sudo aptitude install libqt5core5a libqt5dbus5 libqt5gui5 libqt5gui5-gles libqt5network5 libqt5widgets5 
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb
sudo dpkg -i ./libssl1.1_1.1.0g-2ubuntu4_amd64.deb
rm -i libssl1.1_1.1.0g-2ubuntu4_amd64.deb
sudo dpkg -i synergy_1.14.5-stable.a975f61a_ubuntu21_amd64.deb
```

Synergy currently does not support Wayland and only Xorg, so switch to Xorg:
```
sudo vi /etc/gdm3/custom.conf
```

Set ```WaylandEnable=false```

```
sudo systemctl restart gdm3
```

Run ```synergy``` from a graphical terminal and configure.

## OAM&P scripts
Create the following scripts:

### connect_to_container.sh 
```
#!/bin/bash

if [ "$1" == "" ]; then
  echo "Please specify a container name"
  exit 1
fi

systemd-run --unit=my-unit --user --scope -p "Delegate=yes" -- lxc-attach -n $1
if [ "$?" == "1" ]; then
  systemd-run --user --scope -p "Delegate=yes" -- lxc-attach -n $1
fi
```

### create_container.sh
```
#!/bin/bash

if [ "$1" == "" ]; then
  echo "Please specify a container name"
  exit 1
fi

systemd-run --unit=my-unit --user --scope -p "Delegate=yes" -- lxc-create -t download -n $1
if [ "$?" == "1" ]; then
  systemd-run --user --scope -p "Delegate=yes" -- lxc-create -t download -n $1
fi
```

### list_containers.sh
```
#!/bin/bash
systemd-run --unit=my-unit --user --scope -p "Delegate=yes" -- lxc-ls
if [ "$?" == "1" ]; then
  systemd-run --user --scope -p "Delegate=yes" -- lxc-ls --fancy
fi
```

### remove_containers.sh
```
#!/bin/bash

if [ "$1" == "" ]; then
  echo "Please specify a container name"
  exit 1
fi

systemd-run --unit=my-unit --user --scope -p "Delegate=yes" -- lxc-destroy -n $1
if [ "$?" == "1" ]; then
  systemd-run --user --scope -p "Delegate=yes" -- lxc-destroy -n $1
fi
```

### start_container.sh
```
#!/bin/bash

if [ "$1" == "" ]; then
  echo "Please specify a container name"
  exit 1
fi

systemd-run --unit=my-unit --user --scope -p "Delegate=yes" -- lxc-start $1
if [ "$?" == "1" ]; then
  systemd-run --user --scope -p "Delegate=yes" -- lxc-start $1
fi
```

### stop_container.sh
```
#!/bin/bash

if [ "$1" == "" ]; then
  echo "Please specify a container name"
  exit 1
fi

systemd-run --unit=my-unit --user --scope -p "Delegate=yes" -- lxc-stop $1
if [ "$?" == "1" ]; then
  systemd-run --user --scope -p "Delegate=yes" -- lxc-stop $1
fi
```
