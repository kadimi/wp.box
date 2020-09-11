# -*- mode: ruby -*-
# vi: set ft=ruby :

required_plugins = %w(vagrant-hostsupdater)

plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
if not plugins_to_install.empty?
    puts "Installing plugins: #{plugins_to_install.join(' ')}"
    if system "vagrant plugin install #{plugins_to_install.join(' ')}"
        exec "vagrant #{ARGV.join(' ')}"
    else
        abort "Installation of one or more plugins has failed. Aborting."
    end
end

ip = (ENV['ip'] if ENV['ip'] != nil) || (File.read('ip').strip if File.file?('ip')) || "192.168.100.#{rand(100..199)}"
File.write "ip", "#{ip}\n"

hostname = (ENV['hostname'] if ENV['hostname'] != nil) || (File.read('hostname').strip if File.file?('hostname')) || "wp.box"
File.write "hostname", "#{hostname}\n"

port = (ENV['port'] if ENV['port'] != nil) || (File.read('port').strip if File.file?('port')) || rand(8800..8899)
File.write "port", "#{port}\n"

Vagrant.configure("2") do |config|

    config.vm.box = "scotch/box"
    config.vm.box_url = "https://vagrantcloud.com/scotch/box"
    config.vm.graceful_halt_timeout = 10
    config.vm.network "private_network", ip: File.read('ip').strip
    config.vm.synced_folder ".", "/var/www", nfs: true, nfs_version: 4, nfs_udp: false
    config.vm.hostname = File.read('hostname').strip
    config.vm.network "forwarded_port", guest: 80, host: File.read('port').strip

    config.vm.post_up_message = "
                                                                          
        ##################################################################
                                                                          
        Thank you for developing with wp.box!                             
                                                                          
                                                                          
                                          _|                              
        _|      _|      _|  _|_|_|        _|_|_|      _|_|    _|    _|    
        _|      _|      _|  _|    _|      _|    _|  _|    _|    _|_|      
          _|  _|  _|  _|    _|    _|      _|    _|  _|    _|  _|    _|    
            _|      _|      _|_|_|    _|  _|_|_|      _|_|    _|    _|    
                            _|                                            
                            _|                                            
                                                                          
                                                                          
                                                                          
        Your WordPress website details:                                   
        - URL      : http://#{config.vm.hostname}                         
        - Username : admin                                                
        - Password : password                                             
                                                                          
        ##################################################################
    ".gsub(/^\s{8}/, '')

    config.vm.provision "shell", inline: <<-SHELL

        # Remove faulty repository.
        rm /etc/apt/sources.list.d/ondrej-php5-5_6-trusty.list > /dev/null 2>&1

        # Update package list.
        echo 'Updating APT package list'
        apt-get update > /dev/null 2>&1

        # Install php-common, php-xml, php-xdebug and subversion.
        echo 'Installing php-common, php-xml, php-xdebug and subversion'
        apt-get install -y php-common php-xml php-xdebug subversion > /dev/null 2>&1

        # Increase allowed upload size to 100M.
        echo 'Increasing  allowed upload size to 100M'
        sed -i s/"post_max_size = 8M"/"post_max_size = 2000M"/g /etc/php/7.0/apache2/php.ini > /dev/null 2>&1
        sed -i s/"post_max_size = 8M"/"post_max_size = 2000M"/g /etc/php/7.0/cli/php.ini > /dev/null 2>&1
        sed -i s/"upload_max_filesize = 2M"/"upload_max_filesize = 2000M"/g /etc/php/7.0/apache2/php.ini > /dev/null 2>&1
        sed -i s/"upload_max_filesize = 2M"/"upload_max_filesize = 2000M"/g /etc/php/7.0/cli/php.ini > /dev/null 2>&1

        # Enable Zend OPcache.
        echo 'Enabling Zend OPcache'
        sed -i s/"opcache.enable = 0"/"opcache.enable = 1"/g /etc/php/7.0/apache2/conf.d/user.ini > /dev/null 2>&1

        # Update Composer.
        echo 'Updating Composer'
        composer self-update > /dev/null 2>&1

        # Add Composer to path for user `vagrant`.
        echo 'Adding Composer to PATH for user `vagrant`'
        echo 'PATH="$HOME/.composer/vendor/bin:$PATH"' >> /home/vagrant/.profile

        # Install hirak/prestissimo and PHPUnit for user `vagrant`.
        echo 'Installing PHPUnit and hirak/prestissimo for user `vagrant`'
        sudo su -c "composer global require hirak/prestissimo" vagrant > /dev/null 2>&1
        sudo su -c "composer global require phpunit/phpunit" vagrant > /dev/null 2>&1

        # Install and configure PHPCompatibility/PHPCompatibility and PHPwp-coding-standards/wpcs for user `vagrant`.
        echo 'Installing and configuring PHPCompatibility/PHPCompatibility and wp-coding-standards/wpcs for user `vagrant`'
        sudo su -c "composer global require PHPCompatibility/PHPCompatibility" vagrant > /dev/null 2>&1
        sudo su -c "composer global require wp-coding-standards/wpcs" vagrant > /dev/null 2>&1
        sudo su -c "phpcs --config-set installed_paths /home/vagrant/.composer/vendor/PHPCompatibility/PHPCompatibility/,/home/vagrant/.composer/vendor/wp-coding-standards/wpcs/" vagrant > /dev/null 2>&1

        # Create empty `public` directory.
        rm -fr /var/www/public/ > /dev/null 2>&1
        mkdir -p /var/www/public/
        cd /var/www/public/

        # Install Adminer.
        echo 'Installing Adminer'
        mkdir -p /usr/share/adminer
        curl -sL http://www.adminer.org/latest-mysql-en.php > /usr/share/adminer/adminer.php
        echo "Alias /adminer.php /usr/share/adminer/adminer.php" > /etc/apache2/conf-available/adminer.conf
        a2enconf adminer.conf

        # Create the database.
        echo 'Creating the database'
        MYSQL_PWD=root mysql -u root -e 'DROP DATABASE IF EXISTS `#{config.vm.hostname}`'
        MYSQL_PWD=root mysql -u root -e 'CREATE DATABASE `#{config.vm.hostname}`'

        # Update WP-CLI.
        echo 'Updating WP-CLI'
        curl -sL https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli-nightly.phar > /usr/local/bin/wp

        # Install WP-CLI completions
        echo 'Installing WP-CLI completion'
        mkdir -p '/usr/local/share/wp-cli'
        curl -sL https://raw.githubusercontent.com/wp-cli/wp-cli/master/utils/wp-completion.bash > /usr/local/share/wp-cli/wp-completion.bash
        echo 'source /usr/local/share/wp-cli/wp-completion.bash' >> /home/vagrant/.profile

        # Download WordPress.
        echo 'Downloading WordPress'
        wp core download --allow-root

        # Create WordPress configuration file.
        echo 'Creating WordPress configuration file'
        wp core config --allow-root               \
            --dbname=#{config.vm.hostname}        \
            --dbuser=root                         \
            --dbpass=root                         \
            --dbhost=localhost                    \
            --dbprefix=wp_

        # Run the installation.
        echo 'Running the installation'
        wp core install --skip-email --allow-root \
            --url="http://#{config.vm.hostname}"  \
            --title="Box"                         \
            --admin_user="admin"                  \
            --admin_password="password"           \
            --admin_email="admin@#{config.vm.hostname}"

        # Uninstall Hello and Akismet.
        echo 'Uninstalling Hello and Akismet'
        wp plugin uninstall --allow-root hello akismet > /dev/null 2>&1

        # Flush permalinks.
        echo 'Flushing permalinks'
        wp rewrite flush --allow-root > /dev/null 2>&1

        # Done.
        echo 'Done!'

        # Delete motd.tail which contains the original Scotch box splash screen.
        rm /etc/motd > /dev/null 2>&1

        # Update splash message.
        echo '#{config.vm.post_up_message}' >> /etc/motd

        # Rertart Apache.
        apache2ctl restart

    SHELL

    config.trigger.after :all do |trigger|
        if not (File.file?('Vagrantfile') && File.file?('wp'))
            File.delete("ip")
            File.delete("hostname")
            File.delete("port")
        end
    end
end
