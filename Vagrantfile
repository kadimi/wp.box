# -*- mode: ruby -*-
# vi: set ft=ruby :

required_plugins = %w(vagrant-hostsupdater vagrant-reload)

plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
if not plugins_to_install.empty?
    puts "Installing plugins: #{plugins_to_install.join(' ')}"
    if system "vagrant plugin install #{plugins_to_install.join(' ')}"
        exec "vagrant #{ARGV.join(' ')}"
    else
        abort "Installation of one or more plugins has failed. Aborting."
    end
end

ip = (ENV['ip'] if ENV['ip'] != nil) || (File.read('ip').strip if File.file?('ip')) || "192.168.100.#{rand(100..200)}"
File.write "ip", "#{ip}\n"

hostname = (ENV['hostname'] if ENV['hostname'] != nil) || (File.read('hostname').strip if File.file?('hostname')) || "wp.dev"
File.write "hostname", "#{hostname}\n"

Vagrant.configure("2") do |config|

    config.vm.box = "scotch/box"
    config.vm.graceful_halt_timeout = 10
    config.vm.network "private_network", ip: File.read('ip').strip
    config.vm.synced_folder ".", "/var/www", :mount_options => ["dmode=777", "fmode=666"]
    config.vm.hostname = File.read('hostname').strip

    config.vm.post_up_message = "
                                                                          
        ##################################################################
                                                                          
        Thank you for developing with wp.dev!                             
                                                                          
                                                                          
                                                _|                        
        _|      _|      _|  _|_|_|          _|_|_|    _|_|    _|      _|  
        _|      _|      _|  _|    _|      _|    _|  _|_|_|_|  _|      _|  
          _|  _|  _|  _|    _|    _|      _|    _|  _|          _|  _|    
            _|      _|      _|_|_|    _|    _|_|_|    _|_|_|      _|      
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
        apt-get install -y php7.0-common php7.0-xml php7.0-xdebug subversion > /dev/null 2>&1

        # Enable Zend OPcache.
        echo 'Enabling Zend OPcache'
        sed -i s/"opcache.enable = 0"/"opcache.enable = 1"/g /etc/php/7.0/apache2/conf.d/user.ini > /dev/null 2>&1

        # Update Composer.
        echo 'Updating Composer'
        composer self-update > /dev/null 2>&1

        # Add Composer to path for user `vagrant`.
        echo 'Adding Composer to PATH for user `vagrant`'
        echo 'PATH="$HOME/.composer/vendor/bin:$PATH"' >> /home/vagrant/.profile

        # Install PHPUnit for user `vagrant`.
        echo 'Installing PHPUnit for user `vagrant`'
        sudo su -c "composer global require phpunit/phpunit" vagrant > /dev/null 2>&1

        # Create the directory.
        mkdir -p /var/www/public/
        cd /var/www/public/

        # Download adminer.
        echo 'Downloading adminer'
        curl -sL https://github.com/vrana/adminer/releases/download/v4.3.1/adminer-4.3.1-mysql-en.php > adminer.php

        # Create the database.
        echo 'Creating the database'
        mysql -u root -proot -e 'CREATE DATABASE IF NOT EXISTS `#{config.vm.hostname}`'

        # Update WP-CLI.
        echo 'Updating WP-CLI'
        curl -sL https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar > /usr/local/bin/wp

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

        # Done.
        echo 'Done!'

        # Delete motd.tail which contains the original Scotch box splash screen.
        rm /etc/motd.tail > /dev/null 2>&1

        # Update splash message.
        echo '#{config.vm.post_up_message}' >> /etc/motd.tail

    SHELL

    # Reboot for SSH splash message to take effect.
    config.vm.provision :reload

end
