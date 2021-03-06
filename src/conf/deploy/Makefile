RESOURCES:=$(CURDIR)/resources

DOCKER_TAG=smarthome-openhab

MACHINE=$(shell uname -m)

OS_PLATFORM=amd64

HOST_IP=`hostname -I | awk '{print $$1}'`

ifeq ($(MACHINE),armv7l)
	OS_PLATFORM=armhf
endif

ifeq ($(OS_PLATFORM),)
	@echo Platform not supported
	exit 1
endif

openhab-core:
  # sudo useradd openhab
	# sudo usermod -u 9001 openhab
	# sudo groupmod -g 9001 openhab
	@sudo apt-get install mariadb-client mariadb-server mosquitto mosquitto-clients -y
	@sudo apt-get install python3 python3-dev python3-pip python3-venv gcc libffi-dev -y
	@sudo apt-get install i2c-tools python3-smbus virtualenv logrotate vim motion git -y
	#@sudo apt-get install mplayer alsa-utils usbutils -y
	@sudo apt-get install usbutils -y

	# sudo modprobe i2c_dev
	# dtoverlay=w1-gpio
	# dtparam=i2c_arm=on /boot/config.txt
	# sudo modprobe i2c_dev
	@sudo pip3 install setuptools
	@sudo pip3 install wheel
	#@sudo pip3 install broadlink paho-mqtt rpi-rf RPi.GPIO Adafruit_DHT Adafruit_BME280
	@sudo pip3 install paho-mqtt rpi-rf RPi.GPIO Adafruit_DHT Adafruit_BME280 py-sony-bravia-remote
#	sudo mysql -u root -e `cat resources/sql/database.sql`
# /etc/mysql/my.cnf
#[mysqld]
#bind-address = 0.0.0.0

#  /etc/default/motion start_motion_daemon=yes

#Edit .kodi/userdata/guisettings.xml and search for section and look for <webserverport>8080</webserverport>


openhab-docker-build: openhab-docker-clean
	sudo cp $(RESOURCES)/dockerfile/Dockerfile.template $(CURDIR)/Dockerfile
	@sed -i "s/OS_PLATFORM/$(OS_PLATFORM)/g" $(CURDIR)/Dockerfile
	docker build -t $(DOCKER_TAG) .

openhab-resources-workspace:
	@mkdir -p $(RESOURCES)

openhab-docker-stop:
	-@docker rm -f $(DOCKER_TAG)

openhab-docker-clean:
	@docker image prune -f

openhab-service-install:
	@sudo cp $(RESOURCES)/service/openhab.service /etc/systemd/system/openhab.service
	@sed -i "s/DOCKER_TAG/$(DOCKER_TAG)/g" /etc/systemd/system/openhab.service
	@sed -i "s/HOST_IP/$(HOST_IP)/g" /etc/systemd/system/openhab.service
	@sudo systemctl daemon-reload
	@sudo systemctl enable openhab

openhab-servo-service-install:
	@sudo cp $(RESOURCES)/service/openhab-servo.service /etc/systemd/system/openhab-servo.service
	@sudo systemctl daemon-reload
	@sudo systemctl enable openhab-servo
	@sudo systemctl restart openhab-servo

openhab-server-service-install:
	@sudo cp $(RESOURCES)/service/openhab-server.service /etc/systemd/system/openhab-server.service
	@sudo systemctl daemon-reload
	@sudo systemctl enable openhab-server
	@sudo systemctl restart openhab-server

openhab-powermeter-service-install:
	@sudo cp $(RESOURCES)/service/openhab-powermeter.service /etc/systemd/system/openhab-powermeter.service
	@sudo systemctl daemon-reload
	@sudo systemctl enable openhab-powermeter
	@sudo systemctl restart openhab-powermeter

openhab-camera-service-install:
	@sudo cp $(RESOURCES)/service/openhab-camera.service /etc/systemd/system/openhab-camera.service
	@sudo systemctl daemon-reload
	@sudo systemctl enable openhab-camera
	@sudo systemctl restart openhab-camera

openhab-service-restart:
	@sudo systemctl restart openhab
	@tail -f /opt/openhab/userdata/logs/openhab.log

openhab-info:
	@echo MACHINE=$(MACHINE)
	@echo OS_PLATFORM=$(OS_PLATFORM)
	@echo HOST_IP=$(HOST_IP)

openhab-develop: openhab-info openhab-docker-stop openhab-docker-build

openhab-all: openhab-info openhab-docker-stop openhab-docker-build \
	openhab-service-install openhab-service-restart

# bmp280
# http://faradaysclub.com/?p=1325
# VIN 	Pin 1 (3.3V)
# Ground 	Pin 6
# SCK 	Pin 5 (SCL1)
# SDI 	Pin 3 (SDA1)
