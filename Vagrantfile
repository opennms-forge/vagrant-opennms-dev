# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  # https://atlas.hashicorp.com/reiz/boxes/ubunut_java
  # Already contains Oracle JDK 8, so we do not have to download it (because it is very slow)
  config.vm.box = "antapos/ubuntu-trusty64-jdk8-maven"

  config.vm.network "forwarded_port", guest: 8980, host: 8980
  config.vm.network "forwarded_port", guest: 8001, host: 8001

#  config.vm.synced_folder "~/.m2","/home/vagrant/.m2", owner: "vagrant", group: "vagrant", type: "rsync", rsync__args: ["--verbose", "--rsync-path='rsync'", "--archive"]
#  config.vm.synced_folder "./", "/vagrant", owner: "vagrant", group: "vagrant", disabled: true

  config.vm.provider :virtualbox do |vb|
    vb.name = "Vagrant-OpenNMS-DEV"
    vb.customize ["modifyvm", :id, "--memory", "4096"]
    vb.customize ["modifyvm", :id, "--ioapic", "on"]
    vb.cpus = 4
    # vb.gui = true
  end

  config.vm.provision "shell", inline: <<-SHELL
    echo "deb http://debian.opennms.org stable main" > /etc/apt/sources.list.d/opennms.list
    echo "deb-src http://debian.opennms.org stable main" >> /etc/apt/sources.list.d/opennms.list

    # Add pgp key
    wget -O - http://debian.opennms.org/OPENNMS-GPG-KEY | apt-key add -

    # overall update
    apt-get update

    # install stuff
    apt-get install -y software-properties-common wget git-core
    apt-get install -y tree tcpdump iftop htop slurm screen
    apt-get install -y zsh
    apt-get install -y nsis

    # install and configure PostgreSQL
    apt-get install -y postgresql
    echo "local   all             postgres                                peer" > /etc/postgresql/9.3/main/pg_hba.conf
    echo "local   all             all                                     peer" >> /etc/postgresql/9.3/main/pg_hba.conf
    echo "host    all             all             127.0.0.1/32            trust" >> /etc/postgresql/9.3/main/pg_hba.conf
    echo "host    all             all             ::1/128                 trust" >> /etc/postgresql/9.3/main/pg_hba.conf

    # install OpenNMS basic dependencies
    apt-get install -y maven
    apt-get install -y jicmp jicmp6
    apt-get install -y jrrd

    # fix locales
    echo 'LC_ALL="en_US.utf8"' >> /etc/environment
    locale-gen
    chsh -s /bin/zsh vagrant

    # delete opennms artifacts from local m2 repository
    rm -rf /home/vagrant/.m2/org/opennms
    rm -rf /home/vagrant/.m2/com/opennms

    # now the code
    if [ ! -d /vagrant/opennms ]; then
      git clone https://github.com/OpenNMS/opennms.git /vagrant/opennms
    fi
  SHELL
end
