# -*- mode: ruby -*-
# vi: set ft=ruby :

# Jenkins 2.249.1 with insecureity
jenkinsport = "8080"
phpver = "70"
phpunitver = "6.5.14"

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.box_check_update = false
  config.ssh.insert_key = false
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end
  config.vm.provider "virtualbox" do |vb|
    #vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.memory = "2048"
    vb.check_guest_additions = false
  end
  
  $php70_install = <<-SCRIPT
      # Install php 70
      sudo yum install -y -q epel-release
      sudo yum install -y -q http://rpms.remirepo.net/enterprise/remi-release-7.rpm
      sudo yum install -y -q yum-utils
      sudo yum-config-manager --enable remi-php#{phpver}  > /dev/null 2>&1
      sudo yum install -y -q php-cli php-pear php-fileinfo php-pdo php-intl php-mbstring php-xml php-zip php-xdebug
      # update pear channel
      sudo pear channel-update pear.php.net > /dev/null 2>&1
      sudo pecl channel-update pecl.php.net
      # Install composer
      #php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
      #HASH="$(wget -q -O - https://composer.github.io/installer.sig)"
      #php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
      #sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
      curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
      composer global update
      echo "*** Current user: [`whoami`], Install phpunit, phpcs, phploc, pdepend, phpmd, phpcpd, phpdox ..."
      # PhpUnit 
      #wget https://phar.phpunit.de/phpunit.phar
      # phpunit 6.5.14
      #wget https://phar.phpunit.de/phpunit-6.phar
      # phpunit 7.5.8
      wget --no-verbose https://phar.phpunit.de/phpunit-7.phar
      chmod +x phpunit*.phar
      sudo mv phpunit*.phar /usr/bin/phpunit
      phpunit --version
      
      # Php AB
      wget --no-verbose https://github.com/theseer/Autoload/releases/download/1.25.3/phpab-1.25.3.phar
      chmod +x phpab*.phar
      sudo mv phpab*.phar /usr/bin/phpab
      
      # Php CodeSniffer
      sudo pear install PHP_CodeSniffer-2.3.4 > /dev/null 2>&1
      # PHPLOC
      wget --no-verbose https://phar.phpunit.de/phploc.phar
      chmod +x phploc.phar
      sudo mv phploc.phar /usr/bin/phploc
      # PDepend
      wget --no-verbose http://static.pdepend.org/php/latest/pdepend.phar
      chmod +x pdepend.phar
      sudo mv pdepend.phar /usr/bin/pdepend
      # PHPMD
      wget --no-verbose http://static.phpmd.org/php/latest/phpmd.phar
      chmod +x phpmd.phar
      sudo mv phpmd.phar /usr/bin/phpmd
      # PHPCPD
      wget --no-verbose https://phar.phpunit.de/phpcpd.phar
      chmod +x phpcpd.phar
      sudo mv phpcpd.phar /usr/bin/phpcpd
      # phpDox
      wget --no-verbose http://phpdox.de/releases/phpdox.phar
      chmod +x phpdox.phar
      sudo mv phpdox.phar /usr/bin/phpdox
      echo "***  [PHP] PHP-Tools install is done." 
SCRIPT

  $insecure_mode = <<-SCRIPT
    #!/bin/bash
    echo "     *** Current user: [`whoami`], enable insecure mode and install php-ci-plugin"
    sudo systemctl start jenkins
    # curl -sS -i -m 10 http://localhost:8080/ 
    echo "***  curl check Jenkins service(8080) abailable?"
    #!/bin/bash
    #CODE=$(curl -s -o /dev/null -w '%{http_code}' http://localhost:8080/ )
    #while [[ "$CODE" =~ ^0 ]]
    #do
    #  echo "     ERROR: server returned HTTP code $CODE, sleeping..."
    #  sleep 3
    #  CODE=$(curl -s -o /dev/null -w '%{http_code}' http://localhost:8080/ )
    #done
    printf "Checking Jenkins service is ready? "
    while [[ $(curl -s -w "%{http_code}" http://localhost:8080/login -o /dev/null) != "200" ]]; do
      printf "."
      sleep 2
    done
    printf "\n"
    
    echo "***  Check file(config.xml) is exists?"
    printf "Checking Jenkins config.xml is ready? "
    while [ ! -f /var/lib/jenkins/config.xml ]; do
      printf "."
      sleep 2
    done 
    printf "\n"
    echo "     CHECK config.xml exists. prepare modify config.xml for insecure mode.."
    #!/bin/bash
    grep -q "<useSecurity>true</useSecurity>" /var/lib/jenkins/config.xml
    if [[ $? -eq 0 ]]; then
      echo "Found config.xml contains the <useSecurity>, change it."
      sudo sed -i 's|<useSecurity>true<\/useSecurity>|<useSecurity>false</useSecurity>|' /var/lib/jenkins/config.xml
    fi 
    echo "***  check <useSecurity> in config.xml done; and replace security; Do NOT restart Jenkins"
    
    echo "***  check again."
    grep -q "<useSecurity>true</useSecurity>" /var/lib/jenkins/config.xml
    if [[ $? != 0 ]]; then
      echo "***  CHECK Jenkins is insecurity mode."
    fi 
    #systemctl restart jenkins
    echo "***  [Jenkins] insecure mod done."
SCRIPT

  $jenkins_insecure_restart = <<-SCRIPT
    #!/bin/bash
    echo "     *** [Jenkins] Get jenkins-cli.jar for php-ci-plugin."
cd /home/vagrant 
#wget http://localhost:#{jenkinsport}/jnlpJars/jenkins-cli.jar
printf "Checking Jenkins jenkins-cli.jar is ready? "
CODE=$(curl -s -o /dev/null -w '%{http_code}' http://localhost:8080/jnlpJars/jenkins-cli.jar )
while [[ "$CODE" != 200 ]]
do
  printf "."
  sleep 2
  CODE=$(curl -s -o /dev/null -w '%{http_code}' http://localhost:8080/jnlpJars/jenkins-cli.jar )
done
printf "\n"
cd /home/vagrant 
sudo wget --no-verbose http://localhost:8080/jnlpJars/jenkins-cli.jar

#!/bin/bash
echo "     Run php-ci-plugins install.(restart Jenkins and install right now.)"
sudo systemctl restart jenkins
while [[ $(curl -s -w "%{http_code}" http://localhost:8080/login -o /dev/null) != "200" ]]; do
  printf "."
  sleep 2
done
sudo java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin checkstyle cloverphp crap4j dry htmlpublisher jdepend plot pmd violations warnings xunit publish-over-ssh
if [ $? -eq 0 ]
then
  echo "Successfully install plugins by java "
else
  echo "Plugins install failed."
fi
#java -jar jenkins-cli.jar -s http://localhost:#{jenkinsport} install-plugin checkstyle cloverphp crap4j dry htmlpublisher jdepend plot pmd violations warnings xunit plot publish-over-ssh    
echo "***  [Jenkins] insecure and install-plugin done."
SCRIPT
  
  config.vm.provision "shell", inline: <<-SHELL
    sudo yum update -y > /dev/null 2>&1
    sudo yum install -y -q yum-utils sudo unzip wget > /dev/null 2>&1
    # yum-config-manager --disable \*
    #sudo yum repolist enabled # Just list all enable repos.
    #sudo yum-config-manager --enable local-base local-centosplus local-extras local-updates jenkins
    #sudo yum-config-manager --enable jenkins base updates extras
    sudo timedatectl set-timezone Asia/Taipei
    sudo yum install -y -q java-1.8.0-openjdk java-1.8.0-openjdk-devel ant
  SHELL
  
  config.vm.define "jenkins249" do|jenkins249|
    jenkins249.vm.provision :shell, inline: <<-SHELL
      echo "*** Current user: [`whoami`], Install Jenkins ..."
      # Jenkins repo
      curl --silent --show-error --location http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo | sudo tee /etc/yum.repos.d/jenkins.repo
      sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
      sudo yum-config-manager --enable jenkins  > /dev/null 2>&1
      sudo yum update -y -q > /dev/null 2>&1
      sudo yum install -y -q jenkins > /dev/null 2>&1
      sudo systemctl enable jenkins
      # enable insecure mode and install php-ci-plugin
#      sudo /bin/bash << 'SUSCRIPT'
#        echo "Current user: [`whoami`], change config.xml to insecure mode"
#        systemctl status jenkins
#SUSCRIPT
      # secure mode
      #echo "***  [Jenkins] installtation is done and status/init-password is: "
      #sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    SHELL
    jenkins249.vm.provision :shell, inline: $insecure_mode
    jenkins249.vm.provision :shell, inline: $jenkins_insecure_restart, privileged: false
    #jenkins249.vm.provision :shell, inline: $php70_install, privileged: true
  end

end
