# -*- mode: ruby -*-
# vi: set ft=ruby :

$hostname = "tower-3-2-3"
$private_ip = "192.168.33.40"
$tower_bundle_download_url = "https://releases.ansible.com/ansible-tower/setup-bundle/ansible-tower-setup-bundle-3.2.3-1.el7.tar.gz"
$tower_bundle_install_file = "ansible-tower-setup-bundle-3.2.3-1.el7.tar.gz"

$script = <<-SCRIPT
tower_bundle_download_url=$1
tower_bundle_install_file=$2

echo "****************************************"
echo "- pre-install"
timedatectl set-timezone Asia/Tokyo
yum -y install curl tar
cd /vagrant
if [[ -e "./${tower_bundle_install_file}" ]]; then
  echo "- found ansible-tower-setup-bundle, skip download."
else
  curl -O ${tower_bundle_download_url}
fi

echo "- extract bundle setup file"
if [[ -d "$(basename ${tower_bundle_install_file} .tar.gz)" ]]; then
  echo "  - found ansible-tower-setup-bundle, skip extract."
  cd $(basename ${tower_bundle_install_file} .tar.gz)
else
  tar xf ${tower_bundle_install_file}
  cd $(basename ${tower_bundle_install_file} .tar.gz)
  echo "  - change password for tower"
  sed -e "s/_password=''/_password='password'/g" -i ./inventory
fi

echo "- run setup.sh"
./setup.sh bundle_install

echo "- create shortcut of tower projects"
if [[ ! -d "/vagrant/projects" ]]; then
  su -c "mkdir -m 777 /vagrant/projects" vagrant
fi
rm -rf /var/lib/awx/projects
ln -s /vagrant/projects /var/lib/awx/projects
chown -h awx:awx /var/lib/awx/projects

echo "- copy new license"
if [[ -a /vagrant/license.txt ]]; then
    cp -r /vagrant/license.txt /etc/tower/license
    chown awx:awx /etc/tower/license
fi

# echo "restart tower service"
# ansible-tower-service restart
echo "****************************************"
SCRIPT

$msg = <<MSG
------------------------------------------------------
Ansible Tower(#{$hostname}) is available at:
 - ssh vagrant@#{$private_ip}
 - https://#{$private_ip}
------------------------------------------------------
MSG

# Overwrite host locale in ssh session
ENV["LC_ALL"] = "en_US.UTF-8"

Vagrant.configure("2") do |config|

  config.vm.provider "virtualbox"

  # This vagrant box is downloaded from https://vagrantcloud.com/centos/7
  # Other variants https://app.vagrantup.com/boxes/search
  config.vm.box = "centos/7"

  config.vm.hostname = $hostname
  config.vm.network "private_network", ip: $private_ip
  # config.ssh.forward_agent = true
  config.vm.synced_folder ".", "/vagrant", type: "virtualbox", mount_options: ['dmode=777','fmode=755']
  config.vm.provider :virtualbox do |vb, override|
    vb.customize [
      "modifyvm", :id,
      "--name", "tower3.2.3",
      "--memory", "2048",
      "--cpus", 2
    ]
  end
  config.vm.post_up_message = $msg

  # Enable provisioning ansible-tower with a shell script.
  config.vm.provision "shell" do |s|
    s.inline = $script
    s.args   = [$tower_bundle_download_url, $tower_bundle_install_file]
  end
end
