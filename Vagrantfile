# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  # We use Ubuntu instead of a pre-build image which contains java, as it may
  # has an outdated java version installed
  config.vm.box = "ubuntu/xenial64"

  # Network Configuration
  config.vm.network "forwarded_port", guest: 8980, host: 58980 # OpenNMS http port
  config.vm.network "forwarded_port", guest: 8001, host: 58001 # OpenNMS karaf ssh port
  config.vm.network "forwarded_port", guest: 8101, host: 58101 # OpenNMS remote debug port (if started with -t (debug) option)

  # Shared "opennms repository" folder
  # This only works, if Virtualbox Guest Additions are installed
  config.vm.synced_folder "~/dev/opennms", "/home/ubuntu/dev/opennms"

  # Virtualbox Config
  config.vm.provider :virtualbox do |vb|
    vb.name = "Vagrant-OpenNMS-DEV"
    vb.customize ["modifyvm", :id, "--memory", "4096"]

    # we set it to 1 CPU, as sometimes multiple cpus cause problems
    vb.cpus = 1
    # vb.gui = true
  end

  # Dev Box Provisioning
  config.vm.provision "shell", inline: <<-SHELL
    # Add OpenNMS repository to allow installation of some dependencies, e.g. jicmp
    echo "deb http://debian.opennms.org stable main" > /etc/apt/sources.list.d/opennms.list
    echo "deb-src http://debian.opennms.org stable main" >> /etc/apt/sources.list.d/opennms.list

    # Add pgp key
    wget -O - http://debian.opennms.org/OPENNMS-GPG-KEY | apt-key add -

    # initialize package manager (otherwise apt-get install may not work)
    apt-get update

    # install useful common utils
    apt-get install -y wget curl
    apt-get install -y tree tcpdump iftop htop slurm screen

    # install oh-my-zsh (optional, please uncomment if not desired)
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

    # install Oracle JDK 8
    apt-get install -y software-properties-common
    add-apt-repository ppa:webupd8team/java
    apt-get update
    echo debconf shared/accepted-oracle-license-v1-1 select true | debconf-set-selections
    echo debconf shared/accepted-oracle-license-v1-1 seen true | /usr/bin/debconf-set-selections
    apt-get install -y oracle-java8-installer

    # install and configure PostgreSQL
    apt-get install -y postgresql
    echo "local   all             postgres                                peer" > /etc/postgresql/9.5/main/pg_hba.conf
    echo "local   all             all                                     peer" >> /etc/postgresql/9.5/main/pg_hba.conf
    echo "host    all             all             127.0.0.1/32            trust" >> /etc/postgresql/9.5/main/pg_hba.conf
    echo "host    all             all             ::1/128                 trust" >> /etc/postgresql/9.5/main/pg_hba.conf

    # install OpenNMS basic dependencies
    apt-get install -y maven
    apt-get install -y jicmp jicmp6
    apt-get install -y jrrd

    # install NSIS (is required by the remote-poller windows installer)
    apt-get install -y nsis

    # fix locales
    echo 'LC_ALL="en_US.utf8"' >> /etc/environment
    locale-gen
  SHELL
end
