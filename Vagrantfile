# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  # Not using the debian/jessie64 since it doesn't have vboxfs for
  # synced_folder.
  config.vm.box = "bento/debian-8.2"

  # Maps guest:9091 -> localhost:9091
  config.vm.network "forwarded_port",
    guest: 9091,
    host: 9091,  # Change this to whatever port you want on the host.
    host_ip: "localhost"

  # Map using synced folder.
  config.vm.synced_folder "downloads", "/vagrant_data"

  # Virtualbox specific stuff.
  config.vm.provider "virtualbox" do |vb|
    # Give the VM a meaningful name
    vb.name = "transmission"
    # Customize the amount of memory on the VM:
    vb.memory = "128"
  end

  # Inline script for provisioning.
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update -qq
    sudo apt-get install -yqq transmission-daemon
    sudo service transmission-daemon stop
    sudo /etc/init.d/transmission-daemon stop
    sleep 2
    sudo sed -i.orig \
      -e 's/"rpc-whitelist-enabled": true/"rpc-whitelist-enabled": false/' \
      -e 's/"rpc-authentication-required": true/"rpc-authentication-required": false/' \
      -e 's/"incomplete-dir-enabled": false/"incomplete-dir-enabled": true/' \
      -e 's/"download-dir": "\\/var\\/lib\\/transmission-daemon\\/downloads"/"download-dir": "\\/vagrant_data"/' \
      -e 's/"incomplete-dir": "\\/var\\/lib\\/transmission-daemon\\/Downloads"/"incomplete-dir": "\\/vagrant_data\\/incomplete"/' \
      /etc/transmission-daemon/settings.json
    sudo service transmission-daemon start
  SHELL

  # Restart after mount, to make sure the mountpoint is correct before the
  # daemon starts.
  config.vm.provision "shell", run: "always", inline: <<-SHELL
    sudo service transmission-daemon restart
  SHELL
end
