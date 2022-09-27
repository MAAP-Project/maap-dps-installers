# Verdi (also Factotum)
```
sudo sed -i 's/^SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
sudo sed -i 's/UMASK.*/UMASK 022/g' /etc/login.defs
sudo sed -i 's/umask.*/umask 022/g' /etc/bashrc
sudo sed -i 's/umask.*/umask 022/g' /etc/profile

# lsblk, see that there's a `nvme1n1`
sudo mkfs -t xfs /dev/nvme1n1
sudo mkdir /data
sudo vi /etc/fstab # add `/dev/nvme1n1 /data xfs defaults 0 0`

# reboot!

sudo dnf -y update
sudo dnf install -y puppet wget git gcc openldap-devel net-tools
sudo dnf clean all
sudo rm -rf /var/lib/cloud/*

git clone -b docker https://github.com/hysds/puppet-verdi.git
cd puppet-verdi
sudo ./install.sh hysds docker develop

sudo su - ops
./install_hysds.sh develop develop

# logout and log back in for conda

sudo mkdir -p /export/home
sudo ln -s /home/ops /export/home/ops

# https://docs.docker.com/engine/install/linux-postinstall/#ip-forwarding-problems
# Add the IPForward=true to allow docker to connect to internet

sudo dnf install -y docker-compose-plugin
sudo ln -s /usr/libexec/docker/cli-plugins/docker-compose /usr/local/bin/docker-compose

sudo systemctl start docker
sudo systemctl stop docker
sudo mkdir -p /data/var/lib/ # maybe obsolete with systemd scripts
sudo mv /var/lib/docker /data/var/lib/
sudo ln -s /data/var/lib/docker /var/lib/
sudo systemctl enable docker

# untar into systemd https://maap-uat-registry.s3.us-west-2.amazonaws.com/verdi-systemd.tar

sudo systemctl enable provision-verdi
sudo systemctl enable start-verdi
sudo systemctl enable harikiri
sudo systemctl enable spot_termination_detector

# Make sure ssh keys are same for all!
```
