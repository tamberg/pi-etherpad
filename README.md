# Pi Etherpad

Based on https://titipi.org/wiki/index.php/TITiPI%27s_local_server by [TITiPI](https://titipi.org/) licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

> Note: We assume your computer runs MacOS.

## Run Etherpad
- Plug the power cable into the Pi and a socket
- Wait for the Pi to start up (can take a few minutes)

## Use Etherpad
On your computer

- Visit http://raspberrypi.local:9001

> Note: Your computer has to be in the same Wi-Fi network.

# Maker your own
## Get the hardware
- [Raspberry Pi 3 B+](https://www.pi-shop.ch/raspberry-pi-3-model-b) (CHF 34)
- [Kingston SD card 16 GB](https://www.pi-shop.ch/kingston-microsdhc-karte-industrial-uhs-i-16-gb) (CHF 23)
- [Micro USB power adapter](https://www.pi-shop.ch/raspberry-pi-12-5w-micro-usb-power-supply-2255) (CHF 12)

## Set up the SD Card
On your computer

- Install _Pi Imager_ from https://www.raspberrypi.com/software/
- Insert the SD card
- Select device _Pi 3_
- Select _Pi OS (other)_ > _Pi OS Lite (64-bit)_
- Select SD card storage (compare size)
- Next > Edit
    - Wi-Fi credentials, e.g. MY_SSID, MY_PASSWORD
    - Computer name _raspberrypi_
    - User _pi_
    - Password, e.g. PI_PASSWORD
    - Enable SSH
    - ...
- Wait for the SD card to be written
- Insert the SD card into the Pi
- Try to [log into the Pi](#log-into-the-pi)

On the Pi

- Insert the SD card into the Pi

## Log into the Pi
On your computer

- Open the Terminal app
- Type the following command
    ```bash
	$ ssh pi@raspberrypi.local
    ```

(You're now on the Pi, via SSH)

## Set up the Pi
On the Pi

- Expand the file system
    ```bash
    $ sudo raspi-config
    Advanced Options > Expand Filesystem > Ok
    ```
- Update and upgrade
    ```bash
    $ sudo apt-get update
    $ sudo apt-get upgrade
    ```

## Install Prerequisites
On the Pi

- Install Apache
    ```bash
    $ sudo apt-get install apache2
    ```
- Install Node.js
    ```bash
    $ sudo apt install nodejs
    $ node -v
    v18.19.0
    (or higher)
    ```
- Install git
    ```bash
    $ sudo apt-get install git
    ```
- Install npm
    ```bash
    $ sudo apt-get install npm
    ```
- Install pnpm
    ```bash
    $ sudo npm install -g pnpm
    ```
- Install MariaDB (MySQL) Server
    ```bash
    $ sudo apt-get install mariadb-server
    ```

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

- [Use Etherpad](#use-etherpad)

- Stop and remove the service (optional)

	$ sudo systemctl stop etherpad.service
	$ sudo rm /etc/systemd/system/multi-user.target.wants/etherpad.service
	$ sudo rm /etc/systemd/system/etherpad.service

## Errors
- On `zsh: command not found: $` try to remove the leading `$` character when copying the above commands.
- On `[ERROR] ueberDB - MySQL error: Error: Access denied for user` check the user and password in settings.json.
- On `[ERROR] ueberDB - Fatal MySQL error: Error: connect ECONNREFUSED` check the database name and port in settings.json.
