# Home Assistant host and install

## Hardware

I've installed Home Assistant on an Intel NUC-like device, the [Gigabyte Brix GB-BLCE-4105](https://www.gigabyte.com/nl/Mini-PcBarebone/GB-BLCE-4105-rev-10#ov) with the following specs:

* *Processor* Intel Celeron J4105 (quad core)
* *Memory* Corsair ValueSelect 8GB DDR4 SO-DIMM RAM (2x4GB)
* *Storage* Samsung 60GB SLC SSD

I chose the Gigabyte box instead of an Intel NUC because of the quad core Celeron processor. The J4105 is far newer than the (then alternative) J3455 and faster than the dual core J4005. Both having about the same price.

For a slightly higher cost I put in 8GB of DDR4, which can be quite overkill as 4GB is sufficient for my complete system. 

Finally I settled on an old enterprise Samsung 60GB SLC SSD. Not the fastest drive in the world, but really robust because of the SLC memory that is used. I expect this drive to work for at least another 20 years.

## Software

For my use case I have a custom Home Assistant install based on top of Debian Linux. I chose Debian Linux, because it is lighter and more barebones than for instance Ubuntu Linux. Also I'm familiar with Debian Linux because of the Raspbian distribution that I used earlier.

### Debian install

The Debian installation is fairly straight forward with only a few steps involved. You will need an USB thumb drive and a working internet connection.

1. Download the latest stable version of Debian (server) from the [non-free firmware repo](http://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/current/amd64/iso-cd/). This version contains all the proprietary kernel modules needed for network connectivity
2. Put this ISO onto an USB thumb drive (ith the [balenaEtcher](https://www.balena.io/etcher/) utility)
3. Insert the thumb drive into the target device and boot the machine with the USB drive
4. Follow the network install of Debian
    * I've ignored the `root` user
    * I didn't set up an X environment
    * Additionally installed SSH server
    * Used the complete disk for provisioning

### Additional packages

After the install of Debian was complete. I opted to install some of my favorite CLI utilities:

```bash
sudo apt-get update
sudo apt-get install aptitude avahi-daemon bash curl dbus dnsutils git htop jq ncdu nmon socat sshpass vim
```

> _Some of these packages are also required by Home Assistant later_

### Additional settings

#### Swappiness

I lower the swappiness of the system, to prevent using swap.

```bash
# Set up
sudo sysctl -w vm.swappiness=5
sudo echo "vm.swappiness=5" >> /etc/sysctl.conf

# Check if correctly configured
sysctl vm.swappiness
cat /etc/sysctl.conf
```

#### Root user

We didn't set up a `root` user during install. This is fine, since we can use `sudo`, but prevents us from getting into safe mode when 💩 hits the fan. So we set up a password for the `root` user.

```bash
sudo su
passwd
```

### Docker install

It's quite easy to install Docker. We use the [official documentation](https://docs.docker.com/engine/install/debian/) as a reference for this step.

```bash
# First we change to root mode
sudo -i

apt update
apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common

curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
apt update

apt install docker-ce docker-ce-cli containerd.io

# This is optional, but I like to know for sure that the Docker unit file is enabled
systemctl enable docker

# Finally we exit root mode
exit
```

> _Before installing Docker, check if the steps above are still relevant!_

#### Docker group (optional)

We can also add ourselves to the Docker group. This removes the requirement of using `sudo` when we want to do anything with the Docker cli. This is an optional step.

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

#### CGroup policies (optional)

On my system, the CGroup policies were not set. I had to amend my Grub bootloader, by adding/replacing the following line in `/etc/default/grub`:

```
GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
```

After which you can reload grub with `sudo update-grub` and reboot your system to have it take affect.

#### Docker compose (optional)

I like to have Docker compose installed. Completely optional, but quite easy to do, according to the [official documentation](https://docs.docker.com/compose/install/):

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Home Assistant installation

We do a simple [supervised installation]() of Home Assistant:

```bash
sudo -i

curl -sL https://raw.githubusercontent.com/home-assistant/supervised-installer/master/installer.sh | bash -s

exit
```

## That's it!

This completes a complete Home Assistant install on your new, spanking machine. Check out Home Assistant on: `http://[LOCAL_IP_ADDRESS]:8123`.
