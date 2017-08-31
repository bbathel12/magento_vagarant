# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

vagrantConfig = YAML.load_file './Vagrantfile.config.yml'

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.


Vagrant.configure("2") do |config|
 config.vm.box = "ubuntu/xenial64"
 config.vm.network "private_network", ip: vagrantConfig['ip']
  config.vm.provision "shell", inline:"sudo apt-get update && sudo apt-get upgrade -y "
  config.vm.provision "shell", inline:"sudo apt-get install build-essential linux-headers-$(uname -r) -y"
  config.vm.provision "shell", inline:"sudo apt-get install virtualbox-guest-x11 -y"
  #config.vm.provision "shell", inline:"sudo mount -t vboxsf -o dmode=775,fmode=664 #{vagrantConfig['synced_folder']['host_path']} #{vagrantConfig['synced_folder']['guest_path']}" 

config.vm.synced_folder vagrantConfig['synced_folder']['host_path'],  vagrantConfig['synced_folder']['guest_path'], owner:"ubuntu", group: "www-data", mount_options:["dmode=775,fmode=664"]

 config.vm.provider "virtualbox" do |vb|
	 vb.gui = false
	 vb.memory = "2048"
	 vb.cpus = 2
 end


  config.vm.provision "file", source: "~/.gitconfig", destination: ".gitconfig"
  config.vm.provision "shell", inline: "sudo apt-get update"

  config.vm.provision "shell", inline: "sudo apt-get -y install php php-dev php-zip php-curl php-imagick php-gd php-mcrypt  libapache2-mod-php7.0 php-mysql php-xdebug php-intl php-xsl php-mbstring"

  config.vm.provision "shell", inline: "echo \"xdebug.max_nesting_level=200\" >> /etc/php/7.0/apache2/php.ini"

  config.vm.provision "shell", inline: "sudo apt-get -y install phpunit"


  config.vm.provision "shell", inline: "sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password #{vagrantConfig['mysql']['password']}'"
  config.vm.provision "shell", inline: "sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password #{vagrantConfig['mysql']['password']}'"
  config.vm.provision "shell", inline: "sudo apt-get -y install mysql-server"
  config.vm.provision "shell", inline: "sudo service mysql stop"
  config.vm.provision "shell", inline: "sudo service mysql start"
  config.vm.provision "shell", inline: "sudo update-rc.d mysql defaults"
  config.vm.provision "shell", inline: "echo Made it to after mysql provision"


  config.vm.provision "shell", inline: "sudo apt-get -y install apache2"
  config.vm.provision "shell", inline: "sudo update-rc.d apache2 defaults"
  config.vm.provision "shell", inline: "sudo service apache2 start"
  config.vm.provision "shell", inline: "sudo a2enmod rewrite"
  config.vm.provision "shell", inline: "sudo touch /tmp/tmp.apache2.conf"
  config.vm.provision "shell", inline: "sudo chmod 777 /tmp/tmp.apache2.conf"
  config.vm.provision "shell", inline: "sudo awk '/<Directory \\/>/,/AllowOverride None/{sub(\"None\", \"All\",$0)}{print}' /etc/apache2/apache2.conf > /tmp/tmp.apache2.conf"
  config.vm.provision "shell", inline: "sudo mv /tmp/tmp.apache2.conf /etc/apache2/apache2.conf"
  config.vm.provision "shell", inline: "sudo awk '/<Directory \\/var\\/www\\/>/,/AllowOverride None/{sub(\"None\", \"All\",$0)}{print}' /etc/apache2/apache2.conf > /tmp/tmp.apache2.conf"
  config.vm.provision "shell", inline: "echo Made it to before magento provision"

  config.vm.provision "shell", inline: "sudo rm -Rf /var/www/html"
  config.vm.provision "shell", inline: "sudo ln -s #{vagrantConfig['synced_folder']['guest_path']} /var/www/html"

  config.vm.provision "shell", inline: "curl -sS https://getcomposer.org/installer | php"
  config.vm.provision "shell", inline: "mv composer.phar /usr/local/bin/composer"
  config.vm.provision "shell", inline: "composer clearcache"


  config.vm.provision "shell", inline: "if [ ! -d \"/root/.composer/\" ]; then sudo mkdir /root/.composer/ ;fi"
  config.vm.provision "shell", inline: "echo '{\"http-basic\": {\"repo.magento.com\": {\"username\": \"#{vagrantConfig['http_basic']['repo_magento_com']['username']}\",\"password\": \"#{vagrantConfig['http_basic']['repo_magento_com']['password']} \"}}, \"github-oauth\": {\"github.com\": \"#{vagrantConfig['github_oauth']['github_com']}\"}}' > /root/.composer/auth.json"
  config.vm.provision "shell", inline: "composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition /var/www/html/"

  config.vm.provision "shell", inline: "sudo mysql --user=#{vagrantConfig['mysql']['username']} --password=#{vagrantConfig['mysql']['password']} -e \"CREATE DATABASE #{vagrantConfig['magento']['db_name']};\""

  config.vm.provision "shell", inline: "echo \"\n\n\n\n\n\n\n*******************************************\""
  config.vm.provision "shell", inline: "echo Made it to after magento database"
  config.vm.provision "shell", inline: "echo \"\n\n\n\n\n\n\n*******************************************\""

  config.vm.provision "shell", inline: "sudo php /var/www/html/bin/magento setup:install --base-url=\"#{vagrantConfig['magento']['base_url']}\" --db-host=\"#{vagrantConfig['mysql']['host']}\" --db-user=\"#{vagrantConfig['mysql']['username']}\" --db-password=\"#{vagrantConfig['mysql']['password']}\" --db-name=\"#{vagrantConfig['magento']['db_name']}\" --admin-firstname=\"#{vagrantConfig['magento']['admin_firstname']}\" --admin-lastname=\"#{vagrantConfig['magento']['admin_lastname']}\" --admin-email=\"#{vagrantConfig['magento']['admin_email']}\" --admin-user=\"#{vagrantConfig['magento']['admin_user']}\" --admin-password=\"#{vagrantConfig['magento']['admin_password']}\" --backend-frontname=\"#{vagrantConfig['magento']['backend_frontname']}\" --language=\"#{vagrantConfig['magento']['language']}\" --currency=\"#{vagrantConfig['magento']['currency']}\" --timezone=\"#{vagrantConfig['magento']['timezone']}\""
  config.vm.provision "shell", inline: "sudo php /var/www/html/bin/magento deploy:mode:set developer"
  config.vm.provision "shell", inline: "sudo php /var/www/html/bin/magento cache:disable"
  config.vm.provision "shell", inline: "sudo php /var/www/html/bin/magento cache:flush"
  config.vm.provision "shell", inline: "sudo php /var/www/html/bin/magento setup:performance:generate-fixtures /var/www/html/setup/performance-toolkit/profiles/ce/small.xml"




end

