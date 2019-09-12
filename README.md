# Magento

Try Magento with docker image.


## Prerequisite

1. A Magento account with access keys

    https://marketplace.magento.com/customer/accessKeys/

2. Docker


## Setup

Use the following container image to create LAMP envirionment.
https://github.com/mattrayner/docker-lamp#mysql-databases

```sh
## On local

docker run -it -d -p 80:80 mattrayner/lamp:latest-1604
docker ps
docker exec -t d9f19da3f6b5 bash
```

Mostly follow https://devdocs.magento.com/guides/v2.3/install-gde/composer.html.

```sh
## In the docker container

# install php72 (default it 73 which is out of Magento requirement)
add-apt-repository ppa:ondrej/php
apt-get install -y php7.2 php7.2-common php7.2-fpm php7.2-mysql php7.2-mbstring php7.2-zip php7.2-opcache php7.2-cli php7.2-gd php7.2-curl php7.2-xml php7.2-bcmath php7.2-intl php7.2-soap
update-alternatives --config php  #select php7.2

# Magento meta-package
MAGENTO_DIR=magento2ee
cd /var/www/html
rm -rf $MAGENTO_DIR
composer create-project --repository=https://repo.magento.com/ magento/project-community-edition $MAGENTO_DIR
#Username: pub-key (https://marketplace.magento.com/customer/accessKeys/)
#Password: prv-key (https://marketplace.magento.com/customer/accessKeys/)

# Configure permission
cd /var/www/html/$MAGENTO_DIR
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
chown -R :www-data . # Ubuntu
chmod u+x bin/magento

# Magento
bin/magento setup:install \
--base-url=http://localhost/magento2ee \
--db-host=localhost \
--db-name=magento \
--db-user=root \
--db-password= \
--admin-firstname=admin \
--admin-lastname=admin \
--admin-email=admin@admin.com \
--admin-user=admin \
--admin-password=admin123 \
--language=en_US \
--currency=USD \
--timezone=America/Chicago \
--use-rewrites=1
#-> result
# ...
# [Progress: 864 / 865]
# Write installation date...
# [Progress: 865 / 865]
# [SUCCESS]: Magento installation complete.
# [SUCCESS]: Magento Admin URI: /admin_1gek3j
# Nothing to import.

# Post process
chmod -R 777 /app/magento2ee
php -i | grep "Loaded Configuration File"
cp /etc/php/7.2/cli/php.ini /etc/php/7.2/cli/php.ini.org
vim /etc/php/7.2/cli/php.ini #extension=intl, date.timezone = Asia/China
service apache2 restart

# Install sample data
bin/magento sampledata:deploy
#Username: pub-key (https://marketplace.magento.com/customer/accessKeys/)
#Password: prv-key (https://marketplace.magento.com/customer/accessKeys/)
bin/magento setup:upgrade
bin/magento setup:di:compile
bin/magento setup:static-content:deploy -f

# If needed
#tail -f /var/log/apache2/access.log
#tail -f /var/log/apache2/error.log
#php bin/magento info:adminuri
```

After all, access to http://localhost/magento2ee.
(or admin page http://localhost/magento2ee/admin_1gek3j with user:admin, password:admin123)

Congrat!

