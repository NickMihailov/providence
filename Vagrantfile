# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  config.vm.box = "generic/ubuntu1604"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 80, host: 8080

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  #
  config.vm.provider "libvirt" do |v|
  	v.memory = "1024"
        v.cpus = 2
  end

  config.vm.synced_folder "./", "/vagrant",
    id: "vagrant-root",
    owner: "vagrant",
    group: "www-data",
    mount_options: ["dmode=775,fmode=664"]

  # provision via shell script
  #
  config.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    setup_php="/vagrant/setup.php"

    apt-get update

    # uncomment the line below if you want to upgrade every time you provision
    # (which can take a while if there was a kernel update since you pulled the box)
    apt-get -q -y -o Dpkg::Options::=--force-confold upgrade

    if [[ -e /var/lock/vagrant-provision ]]; then
        exit;
    fi

    timezone=Europe/Sofia
    # In case you want accurate time
    echo "$timezone" > /etc/timezone
    ln -sf /usr/share/zoneinfo/$timezone /etc/localtime
    apt-get -q -y -o Dpkg::Options::=--force-confold install ntp

    echo "mysql-server mysql-server/root_password password root" | sudo debconf-set-selections
    echo "mysql-server mysql-server/root_password_again password root" | sudo debconf-set-selections
    apt-get -y install mysql-client mysql-server

    apt-get -q -y -o Dpkg::Options::=--force-confold install apache2
    apt-get -q -y -o Dpkg::Options::=--force-confold install php7.0 libapache2-mod-php7.0 php7.0-cli
    apt-get -q -y -o Dpkg::Options::=--force-confold install php7.0-curl php7.0-mysql php7.0-json php7.0-gd php7.0-imap php7.0-mcrypt php7.0-xml
    apt-get -q -y -o Dpkg::Options::=--force-confold install htop screen vim apachetop vnstat git
    apt-get -q -y -o Dpkg::Options::=--force-confold install ffmpeg graphicsmagick python-pdfminer
    apt-get -q -y -o Dpkg::Options::=--force-confold install ghostscript dcraw xpdf mediainfo exiftool phantomjs openctm-tools

    # slooooow setup with gmagick and libreoffice. if you want a shiny media processing setup, uncomment the following lines
    #
    # apt-get -q -y -o Dpkg::Options::=--force-confold install php7.0-dev php-pear libgraphicsmagick1-dev libreoffice abiword
	# pecl install gmagick-1.1.7RC3
	#
	# cat << EOF > /etc/php5/mods-available/gmagick.ini
    # extension=gmagick.so
    # EOF
	#
	# ln -s /etc/php5/mods-available/gmagick.ini /etc/php5/apache2/conf.d/20-gmagick.ini

    echo "CREATE DATABASE IF NOT EXISTS collectiveaccess" | mysql -u root --password=root

    cp -a /etc/php/7.0/apache2/php.ini /etc/php/7.0/apache2/php.ini.orig

    sed -i "s/memory\_limit\ \=\ 128M/memory\_limit\ \=\ 512M/g" /etc/php/7.0/apache2/php.ini
    sed -i "s/post\_max\_size\ \=\ 8M/post\_max\_size\ \=\ 64M/g" /etc/php/7.0/apache2/php.ini
    sed -i "s/upload\_max\_filesize \=\ 2M/upload\_max\_filesize\ \=\ 64M/g" /etc/php/7.0/apache2/php.ini

    if ! [ -L /var/www/html ]; then
      rm -rf /var/www/html
      ln -fs /vagrant /var/www/html
    fi

    if [[ ! -f /vagrant/setup.php ]]; then
      cp /vagrant/setup.php-dist /vagrant/setup.php
      sed -i "s/my_database_user/root/g" ${setup_php}
      sed -i "s/my_database_password/root/g" ${setup_php}
      sed -i "s/name_of_my_database/collectiveaccess/g" ${setup_php}
      #sed -i "s/INSTALLS\_\_\'\, false/INSTALLS\_\_\'\, true/g" ${setup_php}
    fi

    if ! [ -f /vagrant/app/conf/local/external_applications.conf ]; then
      cp /vagrant/app/conf/external_applications.conf /vagrant/app/conf/local/external_applications.conf
      #sed -i "s/pdf2txt\.py/pdf2txt/g" /vagrant/app/conf/local/external_applications.conf
      #sed -i "s/\/usr\/local\/bin\/phantomjs/\/usr\/bin\/phantomjs/g" /vagrant/app/conf/local/external_applications.conf
    fi


    service apache2 restart
    service mysql restart

    touch /var/lock/vagrant-provision

  SHELL
end
