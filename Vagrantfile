# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.network "forwarded_port", guest: 8980, host: 8980
  config.vm.network "forwarded_port", guest: 8001, host: 8001

  config.vm.provider :virtualbox do |vb|
    vb.name = "Vagrant-OpenNMS"
    vb.customize ["modifyvm", :id, "--memory", "4096"]
    vb.customize ["modifyvm", :id, "--ioapic", "on"]
    vb.cpus = 4
    # vb.gui = true

    config.vm.synced_folder "~/.m2","/home/vagrant/.m2", type: "rsync", rsync__args: ["--verbose", "--rsync-path='rsync'", "--archive", "--chown=vagrant:vagrant" ]
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
    apt-get install -y tree zsh tcpdump iftop htop slurm

    # install and configure PostgreSQL
    apt-get install -y postgresql
    echo "host all  all    0.0.0.0/0  trust" >> /etc/postgresql/9.3/main/pg_hba.conf

    # install Oracle Java
    add-apt-repository ppa:webupd8team/java
    apt-get update
    echo debconf shared/accepted-oracle-license-v1-1 select true | debconf-set-selections
    echo debconf shared/accepted-oracle-license-v1-1 seen true | /usr/bin/debconf-set-selections
    apt-get install -y oracle-java8-installer

    # install OpenNMS basic dependencies
    apt-get install -y maven
    apt-get install -y jicmp jicmp6
    apt-get install -y jrrd

    # now the code
    cd /vagrant
    if [ ! -d opennms ]; then
      git clone https://github.com/OpenNMS/opennms.git
    fi

    #cd opennms
    #git checkout develop
    #git pull
    #./clean.pl
    #./compile.pl
  SHELL
end
