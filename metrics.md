# Metrics
```
sudo sed -i 's/^SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
sudo sed -i 's/UMASK.*/UMASK 022/g' /etc/login.defs
sudo sed -i 's/umask.*/umask 022/g' /etc/bashrc
sudo sed -i 's/umask.*/umask 022/g' /etc/profile

# reboot!

sudo dnf -y update
sudo dnf install -y puppet wget git gcc openldap-devel net-tools
sudo dnf clean all
sudo rm -rf /var/lib/cloud/*

git clone -b docker https://github.com/hysds/puppet-metrics.git
cd puppet-metrics
sudo ./install.sh hysds docker develop

sudo su - ops
./install_hysds.sh develop develop

# logout and log back in for conda

sudo mkdir -p /export/home
sudo ln -s /home/ops /export/home/ops

# https://docs.docker.com/engine/install/linux-postinstall/#ip-forwarding-problems
# Add the IPForward=true to allow docker to connect to internet

sudo systemctl enable docker
sudo systemctl restart docker

sudo zip -q -d /home/ops/logstash-7.9.3/logstash-core/lib/jars/log4j-core-2.* org/apache/logging/log4j/core/lookup/JndiLookup.class

wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.9.3-x86_64.rpm  
sudo dnf localinstall -y --nogpgcheck elasticsearch-7.9.3-x86_64.rpm  
sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install repository-s3 --batch
sudo vi /etc/elasticsearch/elasticsearch.yml # set cluster.name: resource_cluster, node.name: master, network.host: 0.0.0.0, cluster.initial_master_nodes: ["master"]
sudo systemctl enable elasticsearch

sudo dnf install -y redis
# edit /etc/redis.conf to update `bind 0.0.0.0` and `protected no`
sudo systemctl restart redis
sudo systemctl enable redis

# Make sure ssh keys are same for all!
```
