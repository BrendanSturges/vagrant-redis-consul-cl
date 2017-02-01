# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT

echo "Installing common dependencies ..."
sudo apt-get update

echo "Installing consul dependencies ..."
sudo apt-get install -y unzip curl

echo "Installing redis dependencies ..."
sudo apt-get install build-essential tcl

echo "Fetching Consul version 0.7.3 ..."
cd /tmp/
curl -s https://releases.hashicorp.com/consul/0.7.3/consul_0.7.3_linux_amd64.zip -o consul.zip

echo "Fetching Redis stable ..."
curl -O http://download.redis.io/redis-stable.tar.gz

echo "Installing Consul version 0.7.3 ..."
unzip consul.zip
sudo chmod +x consul
sudo mv consul /usr/bin/consul

sudo mkdir /etc/consul.d
sudo chmod a+w /etc/consul.d

SCRIPT

# Specify a Consul version
CONSUL_VERSION = ENV['0.7.3'] || "0.7.3"

# Specify a custom Vagrant box for the demo
BOX_NAME = ENV['BOX_NAME'] || "debian/jessie64"

# Vagrantfile API/syntax version.
# NB: Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = BOX_NAME

  config.vm.provision "shell",
                          inline: $script,
                          env: {'CONSUL_VERSION' => CONSUL_VERSION}

  config.vm.define "s1" do |s1|
      s1.vm.hostname = "s1"
      s1.vm.network "private_network", ip: "172.20.20.10"
  end

  config.vm.define "s2" do |s2|
      s2.vm.hostname = "s2"
      s2.vm.network "private_network", ip: "172.20.20.11"
  end

  config.vm.define "s3" do |s3|
     s3.vm.hostname = "s3"
     s3.vm.network "private_network", ip: "172.20.20.12"
  end
end

echo "Installing Redis ..."
tar xzvf redis-stable.tar.gz
cd redis-stable
make
make install

echo "Configuring redis ..."
rm /etc/redis.conf
mkdir -p /etc/redis
mkdir /var/redis
chmod -R 777 /var/redis
useradd redis

cp -u /vagrant/redis.conf /etc/redis/6379.conf
cp -u /vagrant/redis.init.d /etc/init.d/redis_6379

update-rc.d redis_6378 defaults

chmod a+x /etc/init.d/redis_6379
/etc/init.d/redis_6379 start

echo "Joining consul to cluster ..."
vagrant ssh s2
consul join 172.20.20.10
exit

vagrant ssh s3
consul join 172.20.20.11
exit

