# Mozart

```
sudo sed -i 's/^SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
sudo sed -i 's/UMASK.*/UMASK 022/g' /etc/login.defs
sudo sed -i 's/umask.*/umask 022/g' /etc/bashrc
sudo sed -i 's/umask.*/umask 022/g' /etc/profile

# reboot!

sudo dnf -y update
# sudo dnf install -y epel-release puppet nscd wget curl subversion git vim screen gcc openldap-devel
sudo dnf install -y puppet wget git gcc openldap-devel net-tools
sudo dnf clean all
sudo rm -rf /var/lib/cloud/*

git clone -b docker https://github.com/hysds/puppet-mozart.git
cd puppet-mozart
sudo ./install.sh hysds docker develop

#git clone -b docker https://github.com/hysds/puppet-verdi.git
#cd puppet-verdi
#sudo ./install.sh hysds docker develop

sudo su - ops
#./install_hysds.sh develop develop
wget https://hysds.jfrog.io/artifactory/generic-local/installation/ops/v4.1.0-beta.3/hysds-conda_env-v4.1.0-beta.3.tar.gz
wget https://hysds.jfrog.io/artifactory/generic-local/installation/ops/v4.1.0-beta.3/hysds-mozart_venv-v4.1.0-beta.3.tar.gz
export PATH=/export/home/ops/conda/bin:$PATH
conda-unpack


# logout and back in



# logout and log back in for conda

sudo mkdir -p /export/home
sudo ln -s /home/ops /export/home/ops

# https://docs.docker.com/engine/install/linux-postinstall/#ip-forwarding-problems
# Add the IPForward=true to allow docker to connect to internet

sudo zip -q -d /home/ops/logstash-7.9.3/logstash-core/lib/jars/log4j-core-2.* org/apache/logging/log4j/core/lookup/JndiLookup.class

# sudo dnf install -y autoconf make libtool
# git clone https://git.code.sf.net/p/tsunami-udp/code_git tsunami-udp-code_git
# cd tsunami-udp-code_git/tsunami-udp
# ./recompile.sh
# sudo make install
# 
# # installing dbxml, maybe already installed?
# wget https://github.com/hysds/puppet-hysds_base/raw/develop/files/dbxml-6.1.4-1.x86_64.rpm
# sudo dnf localinstall -y --nogpgcheck dbxml-6.1.4-1.x86_64.rpm

wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.9.3-x86_64.rpm  
sudo dnf localinstall -y --nogpgcheck elasticsearch-7.9.3-x86_64.rpm  
sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install repository-s3 --batch
sudo vi /etc/elasticsearch/elasticsearch.yml # set cluster.name: resource_cluster, node.name: master, network.host: 0.0.0.0, cluster.initial_master_nodes: ["master"]
sudo systemctl enable elasticsearch

sudo dnf install -y redis
# edit /etc/redis.conf to update `bind 0.0.0.0` and `protected no`
sudo systemctl restart redis
sudo systemctl enable redis

wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.9.5/rabbitmq-server-3.9.5-1.el8.noarch.rpm
sudo dnf localinstall -y --nogpgcheck rabbitmq-server-3.9.5-1.el8.noarch.rpm
wget https://github.com/rabbitmq/erlang-rpm/releases/download/v24.0.6/erlang-24.0.6-1.el8.x86_64.rpm # restore old version of erlang
sudo dnf localinstall -y --nogpgcheck erlang-24.0.6-1.el8.x86_64.rpm

sudo systemctl enable rabbitmq-server
sudo /usr/sbin/rabbitmq-plugins enable rabbitmq_management
sudo systemctl restart rabbitmq-server

sudo systemctl enable httpd

sudo htpasswd -c /etc/httpd/conf/passwd ops

# Add entries in /etc/hosts for grq and metrics machines, i.e.
# 10.46.8.18 grq
# 10.46.9.193 metrics

# Make sure ssh keys are same for all!
ssh-keygen -t rsa -b 2048
cp .ssh/id_rsa.pub .ssh/authorized_keys
```
