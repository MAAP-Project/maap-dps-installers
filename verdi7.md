# Verdi (also Factotum) for Centos 7
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

sudo yum install -y dnf
sudo rpm -Uvh https://yum.puppet.com/puppet6-release-el-7.noarch.rpm
sudo rpm -Uvh https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/p/pbzip2-1.1.12-1.el7.x86_64.rpm

sudo dnf -y update
sudo dnf install -y puppet wget git gcc openldap-devel net-tools
sudo dnf clean all
sudo rm -rf /var/lib/cloud/*

# preemptively clone to fix come config files
cd /etc/puppetlabs/code/modules/
sudo git clone -b docker https://github.com/hysds/puppet-verdi verdi
sudo git clone -b develop https://github.com/hysds/puppet-hysds_base hysds_base

# Go into `/etc/puppetlabs/code/modules/hysds_base/manifests/init.pp`
# and remove the `--nobest` install option for docker as it's not a valid option in Centos 7
# remove `libnsl` completely, it's included in Centos 7 by default
# in `/etc/puppetlabs/code/modules/verdi/manifests/init.pp`
# remove entry for running `httpd-ssl-gencerts`, this only affects RHEL8
# Finally, go into `/etc/puppetlabs/code/modules/hysds_base/site.pp` and change docker url to "/7/"
# do the same in `/etc/puppetlabs/code/modules/verdi/site.pp` and change docker url to "/7/"

cd $HOME
git clone -b docker https://github.com/hysds/puppet-verdi.git
cd puppet-verdi
sudo su
./install.sh hysds docker develop
exit

sudo su - ops
./install_hysds.sh develop develop

# logout and log back in for conda

sudo mkdir -p /export/home
sudo ln -s /home/ops /export/home/ops

sudo dnf install -y docker-compose-plugin
sudo ln -s /usr/libexec/docker/cli-plugins/docker-compose /usr/local/bin/docker-compose

sudo systemctl start docker
sudo systemctl stop docker
sudo mkdir -p /data/var/lib/ # maybe obsolete with systemd scripts
sudo mv /var/lib/docker /data/var/lib/
sudo ln -s /data/var/lib/docker /var/lib/
sudo systemctl enable docker

# untar into /etc/systemd/system s3://maap-uat-registry/verdi-systemd.tar

sudo systemctl enable provision-verdi
sudo systemctl enable start-verdi
sudo systemctl enable harikiri
sudo systemctl enable spot_termination_detector

# Make sure ssh keys are same for all!
```
