# aprs-frontent
APRS Web GUI




## Tested on

* Centos / RHEL 7 with Apache and PHP 7.4
* Windows 10 with Apache (Xampp) and PHP 7.3


## Dependencies

* Apache or Nginx Webserver (Xampp on Windows works as well)
  * Apache Module mod_proxy_html
* PHP 7.0 or higher
  * Mcrypt (php-mcrypt)
  * MySQLi (php-mysql)
  * GD (php-gd) OR PHP Imagick (php-pecl-imagick)
* MySQL or MariaDB
* For "Offline" Maps
  * Docker
  * [https://openmaptiles.org/](OpenMapTiles)

## Sources

* [Importer](japalie/aprs2mysql)
* APRS Symbols
  * [hessu/aprs-symbols](https://github.com/hessu/aprs-symbols)
  * [APRS Direct](https://www.aprsdirect.com/)
* JS Librarys
  * [Leaflet](https://github.com/Leaflet/Leaflet)
  * [Mapbox-GL for "Offline" Maps](https://github.com/mapbox/mapbox-gl-js)
  * [Map Marker Spiderfier](https://github.com/jawj/OverlappingMarkerSpiderfier-Leaflet)
  * [Map Center Coordinates](https://github.com/xguaita/Leaflet.MapCenterCoord)
  * [JQuery](https://jquery.com/)
* PHP Librarys
  * [APRS Passcode Generator](https://github.com/magicbug/PHP-APRS-Passcode)
  * [GeoIP](https://www.ip2location.com/development-libraries/ip2location/php)


## Installation (CentOS/RHEL)

** Apache and MariaDB **
```
sudo yum install httpd mod_proxy_html mariadb-server -y

#VHOST
echo "<VirtualHost *:80>
        ServerName APRS
        ProxyPreserveHost On
        DocumentRoot "/var/www/html"

        ProxyPass /data/ http://127.0.0.1:8090/data/
        ProxyPassReverse /data/ http://127.0.0.1:8090/data/

        ProxyPass /fonts/ http://127.0.0.1:8090/fonts/
        ProxyPassReverse /fonts/ http://127.0.0.1:8090/fonts/

        ProxyPass /styles/ http://127.0.0.1:8090/styles/
        ProxyPassReverse /styles/ http://127.0.0.1:8090/styles/

        <location />
                Require all granted
        </location>
</VirtualHost>
" > /etc/httpd/conf.d/vhost.conf

```



Add the following line under the Topic `[mysqld]` in /etc/my.cnf
```
skip-name-resolve
```

** PHP7.4 **
```
sudo yum install epel-release yum-utils -y
sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
sudo yum-config-manager --enable remi-safe
sudo yum install php74 php74-php php74-php-mysql php74-php-mcrypt php74-php-pecl-imagick
```

** Docker and Openmaptiles **
```
sudo curl -sSL https://get.docker.com/ | sh
sudo mkdir -p /opt/openmaptiles
sudo docker run -d --rm -t -v $/opt/openmaptiles:/data -p 8090:80 --name openmaptiles klokantech/openmaptiles-server
```
You can access openmaptiles through the Web: http://<ip>:8090 (Downloading Maps for personal use is free).

** Copy Data from Repositiry **
```
sudo rm -f /var/www/html/*
git clone https://github.com/japalie/aprs-frontent /var/www/html
```

** Start Services **
```
sudo systemctl start httpd.service
sudo systemctl enable httpd.service
sudo systemctl start mariadb.service
sudo systemctl enable mariadb.service
```

** Configure DB **
Load Schema from [aprs2mysql](japalie/aprs2mysql) and add a filter table:
```
CREATE TABLE `filter` (
  `callsign` varchar(32) NOT NULL
) ENGINE=MyISAM DEFAULT CHARSET=latin1;
ALTER TABLE `filter`
  ADD PRIMARY KEY (`callsign`);
COMMIT;
```
