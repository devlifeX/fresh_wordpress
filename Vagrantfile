# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'json'
file = File.read('config.json')
option = JSON.parse(file)

hostname = option['vagrant']['vagrant']['hostname']
db_name = option['vagrant']['db']['name']
db_user = option['vagrant']['db']['user']
db_pass = option['vagrant']['db']['password']
synced_folder = option['vagrant']['vagrant']['synced_folder']

Vagrant.configure("2") do |config|

    config.vm.box = option['vagrant']['vagrant']['box']
    config.vm.network "private_network", ip: option['vagrant']['vagrant']['ip']
    config.vm.hostname = hostname
    config.vm.synced_folder ".", synced_folder, :mount_options => ["dmode=777", "fmode=666"]
    
    config.vm.provider "virtualbox" do |v|
        v.memory = 768
    end

    config.vm.provision "shell", inline: <<-SHELL
    sudo sed -i s,/var/www/public,#{synced_folder}/,g /etc/apache2/sites-available/000-default.conf
    sudo sed -i s,/var/www/public,#{synced_folder}/,g /etc/apache2/sites-available/scotchbox.local.conf
    sudo service apache2 restart

    mysql -u #{db_user} -p#{db_pass} -e "DROP DATABASE IF EXISTS #{db_name};"
    mysql -u #{db_user} -p#{db_pass} -e "CREATE DATABASE #{db_name};"
    mysql -u #{db_user} -p#{db_pass} -D #{db_name} < #{synced_folder}/#{db_name}.sql
    mysql -u #{db_user} -p#{db_pass} -D #{db_name} -e "UPDATE wp_options SET option_value = 'http://#{hostname}/' WHERE option_id = '1';"
    mysql -u #{db_user} -p#{db_pass} -D #{db_name} -e "UPDATE wp_options SET option_value = 'http://#{hostname}/' WHERE option_id = '2';"

    cp #{synced_folder}/wp-config-sample.php #{synced_folder}/wp-config.php

    sudo sed -i "s/'database_name_here'/'#{db_name}'/g" #{synced_folder}/wp-config.php
    sudo sed -i "s/'password_here'/'#{db_pass}'/g" #{synced_folder}/wp-config.php
    sudo sed -i "s/'username_here'/'#{db_user}'/g" #{synced_folder}/wp-config.php

    SHELL

end