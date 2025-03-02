# Pi Etherpad

Based on https://titipi.org/wiki/index.php/TITiPI%27s_local_server licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

## Set up the SD Card

On your computer

- Install Pi Imager
- Insert SD Card
- Select Pi 3 (or Pi 4)
- Select Other > Pi OS 64-bit Lite
- User: pi, Password: PI_PASSWORD
- Wi-Fi: MY_SSID, MY_PASSWORD
- SSH enable
- Wait for the SD card to be ready

On the Pi

- Insert the SD card into the Pi

## Log into the Pi

On your computer

- Open the Terminal app (or PuTTY on Windows)
- Type the following command (without the '$')

	$ ssh pi@raspberrypi.local

(You're now on the Pi, via SSH)

## Set up the Pi

On the Pi

- Expand the file system
	
	$ sudo raspi-config
	Advanced Options > Expand Filesystem > Ok

- Update and upgrade

	$ sudo apt-get update
	$ sudo apt-get upgrade

## Install Prerequisites

On the Pi

- Install Apache

	$ sudo apt-get install apache2

- Install Node.js

	$ sudo apt install nodejs
	$ node -v
	v18.19.0

	(or higher)

- Install git

	$ sudo apt-get install git

- Install npm

	$ sudo apt-get install npm

- Install pnpm

	$ sudo npm install -g pnpm

- Install MariaDB (MySQL) Server

	$ sudo apt-get install mariadb-server

## Install Etherpad Lite

On the Pi

Based on https://github.com/ether/etherpad-lite licensed under [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0.txt).

- Download the source from GitHub

	$ cd ~
	$ git clone -b master https://github.com/ether/etherpad-lite.git

- Build Etherpad with pnpm

	$ cd etherpad-lite
	$ pnpm i
	$ pnpm run build:etherpad

- Run Etherpad with pnpm

	$ pnpm run prod

- Stop Etherpad

	CTRL-C

	(works for most programs)

## Set up a database for Etherpad Lite

- Create a database

	$ sudo mysql
	create database etherpad_db;
	grant all on etherpad.* to 'DB_USER'@'localhost' identified by 'DB_PASSWORD';

- Test the database

	$ mysql -u DB_USER -pDB_PASSWORD

	> Note: not putting a space after -p is important

- Connect Etherpad to the database

	$ nano settings.json
	{
	  "dbType" : "mysql",
	  "dbSettings" : {
	    "user":     "DB_USER",
	    "host":     "127.0.0.1",
	    "port":     3306,
	    "password": "DB_PASSWORD",
	    "database": "etherpad_db",
	    "charset":  "utf8mb4"
	  },
	}

- Run Etherpad once, via bin/run.sh

	$ bin/run.sh

## Set up a service to run Etherpad

- Create a file etherpad.service

	$ sudo nano /etc/systemd/system/etherpad.service

	[Unit]
	Description=Etherpad
	After=syslog.target network.target

	[Service]
	Type=simple
	User=pi
	Group=pi
	WorkingDirectory=/home/pi/etherpad-lite
	ExecStart=/usr/local/bin/pnpm run prod
	Restart=always

	[Install]
	WantedBy=multi-user.target

- Reload, enable and start the service

	$ sudo systemctl daemon-reload
	$ sudo systemctl enable etherpad.service
	$ sudo systemctl start etherpad.service

- Stop and remove the service (optional)

	$ sudo systemctl stop etherpad.service
	$ sudo rm /etc/systemd/system/multi-user.target.wants/etherpad.service
	$ sudo rm /etc/systemd/system/etherpad.service

## Use Etherpad

On your computer

- Visit http://raspberrypi.local:9001

> Note: your computer has to be in the same Wi-Fi network (or LAN)

## Errors

- On [ERROR] ueberDB - Fatal MySQL error: Error: connect ECONNREFUSED ::1:3306

	Check the database name in settings.json

- On [ERROR] ueberDB - MySQL error: Error: Access denied for user

	Check the user and password in settings.json
