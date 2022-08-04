# Installing Ubuntu on the macbook

## USB installation disk
- Use a USB stick, at least 2Gb in size
- On the macbook, instert the USB stick and Erase it
- Select MS-DOS FAT as the file system
- Download open-source app balenaEtcher and install on the macbook
- Download an Ubuntu 20.04 LTS iso
- Use etcher to flash the USB drive with the iso image

## Ubuntu installation
- Insert the USB disk in the macbook
- Reboot the macbook, holding down the option key during boot
- Select the correct EFI instance to boot from the USB
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
