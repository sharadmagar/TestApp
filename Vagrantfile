$master_script = <<SCRIPT
#!/bin/bash

echo "\n----- Installing Java JDK ------\n"
sudo apt-get update
sudo apt-get install default-jdk
java -version

echo "\n----- Installing Apache ------\n"
sudo apt-get update
sudo apt-get install apache2

echo "\n----- Clonning Git Repo ------\n"
sudo cd /opt/
sudo git clone https://github.com/sharadmagar/TestApp.git
sudo cd ..

echo "\n----- Creating Sample Web Page ------\n"
sudo mkdir -p /var/www/example.com/public_html
sudo chown -R $USER:$USER /var/www/example.com/public_html
sudo chmod -R 755 /var/www
sudo mv /opt/TestApp/index_Example.html /var/www/example.com/public_html/

echo "\n----- Reconfigure Apache to Https ------\n"
sudo DocumentRoot /var/www/example.com/public_html
sudo mv /opt/TestApp/example.com.conf /etc/apache2/sites-available/example.com.conf
sudo a2ensite example.com.conf
sudo service apache2 restart
sudo mv /opt/TestApp/hosts /etc/hosts

echo "\n----- Installing Memcached ------\n"
sudo apt-get update
sudo apt-get install -y python-memcache
sudo apt-get install -y rrdtool

echo "\n----- Monitorin MemCached using python ------\n"
chmod +x memcached_stats_rrd.py

echo "\n----- Creating Cron Job for Memcached ------\n"
*/1 * * * * /home/perfserver/memcached_stats_rrd.py

SCRIPT

$hosts_script = <<SCRIPT
cat > /etc/hosts <<EOF
127.0.0.1       localhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

EOF
SCRIPT

Vagrant.configure("2") do |config|

  # Define base image
  config.vm.box = "precise64"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"

  # Manage /etc/hosts on host and VMs
  config.hostmanager.enabled = false
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true
  config.hostmanager.ignore_private_ip = false

  config.vm.define :master do |master|
    master.vm.provider :virtualbox do |v|
      v.name = "vm-cluster-node1"
      v.customize ["modifyvm", :id, "--memory", "2048"]
    end
    master.vm.network :forwarded_port, guest: 80, host: 8080
    master.vm.hostname = "vm-cluster-node1"
    master.vm.provision :shell, :inline => $hosts_script
    master.vm.provision :hostmanager
    master.vm.provision :shell, :inline => $master_script
  end

end