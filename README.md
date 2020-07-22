# aprs-frontent
APRS Web GUI

![Screenshot](https://github.com/japalie/aprs-frontent/blob/master/readme/Screen1.png?raw=true)

![Screenshot](https://github.com/japalie/aprs-frontent/blob/master/readme/Screen2.png?raw=true)

![Screenshot](https://github.com/japalie/aprs-frontent/blob/master/readme/Screen4.png?raw=true)

**Functions**
* "Offline" Map Support (You need a Connection to the Webserver, localhost is engough)
* Login function
* Callsign Filtering (From / Via)
* Callsign Filtering (DB and View)
* Follow Objects
* Periodic Updates
* Display Tracks
* Display WX Data
* Display Speed / Course
* Raw Data View
* Message View
* API (JSON)
* Captcha Verification

# Idea


# Technical

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
  * [https://openmaptiles.com/server/](OpenMapTiles)

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
* MAP
  * [OpenStreetMap](https://www.openstreetmap.org/)
  * [OpenMapTiles](https://openmaptiles.org/)

## Installation (CentOS/RHEL)

**Apache and MariaDB**
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

**PHP7.4**
```
sudo yum install epel-release yum-utils -y
sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
sudo yum-config-manager --enable remi-safe
sudo yum install php74 php74-php php74-php-mysql php74-php-mcrypt php74-php-pecl-imagick
```

**Docker and Openmaptiles**
```
sudo curl -sSL https://get.docker.com/ | sh
sudo mkdir -p /opt/openmaptiles
sudo docker run -d --rm -t -v $/opt/openmaptiles:/data -p 8090:80 --name openmaptiles klokantech/openmaptiles-server
```
You can access openmaptiles through the Web: http://<ip>:8090 (Downloading Maps for personal use is free).

**Copy Data from Repositiry**
```
sudo rm -f /var/www/html/*
git clone https://github.com/japalie/aprs-frontent /var/www/html
```

**Start Services**
```
sudo systemctl start httpd.service
sudo systemctl enable httpd.service
sudo systemctl start mariadb.service
sudo systemctl enable mariadb.service
```

**Configure DB**
Load Schema from [aprs2mysql](japalie/aprs2mysql) and add a filter table:
```
CREATE TABLE `filter` (
  `callsign` varchar(32) NOT NULL
) ENGINE=MyISAM DEFAULT CHARSET=latin1;
ALTER TABLE `filter`
  ADD PRIMARY KEY (`callsign`);
COMMIT;
```

**Edit config.php**
Edit the file `/var/www/html/config.php` to your needs.
Example:
```
<?php
$_CONFIG = array();

//Datenbankkonfiguration
$_CONFIG["db"]["db"] = "aprs";	// Database
$_CONFIG["db"]["user"] = "root"; // DB-User
$_CONFIG["db"]["pass"] = "********"; //DB-Password
$_CONFIG["db"]["host"] = "10.0.0.141"; //DB-IP

$_CONFIG["db"]["packettable"] = "packets"; //Packet Table
$_CONFIG["db"]["filtertable"] = "filter"; //Filter Table

//APRS Spezifische Einstellungen
$_CONFIG["aprs"]["maxage"] = 60; // in minutes
$_CONFIG["aprs"]["maxrecords"] = 1000; // max shown objects
$_CONFIG["aprs"]["purgeafter"] = 48; // in hours
$_CONFIG["aprs"]["sessionkey"] = "********"; // For Session Encryption
?>
```
