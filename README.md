# MH-Z19 Raspberry MQTT Client/Daemon

A simple Linux python script to query MH-Z19 sensor on Raspberry Pi and send the data to an **MQTT** broker,
e.g., the famous [Eclipse Mosquitto](https://projects.eclipse.org/projects/technology.mosquitto).
After data made the hop to the MQTT broker it can be used by home automation software, like [Home Assistant](https://www.home-assistant.io/).

The program can be executed in **daemon mode** to run continuously in the background, e.g., as a systemd service.
## Features

* Tested with MH-Z19 sensor
* Build on top of [mh-z19](https://github.com/UedaTakeyuki/mh-z19/)
* Highly configurable
* Data publication via MQTT
* Configurable topic and payload:
    * using the [HomeAssistant MQTT discovery format](https://home-assistant.io/docs/mqtt/discovery/)
* Announcement messages to support auto-discovery services
* MQTT authentication support
* No special/root privileges needed
* Daemon mode (default)
* Systemd service, sd\_notify messages generated
* Tested on Raspberry Pi 3

### Readings

The MH-Z19 sensor offers the following readings:

| Name            | Description |
|-----------------|-------------|
| `co2`           | CO2 value, in [ppm] |
| `temperature`   | Air temperature, in [°C] |
| `SS`            | Some kind of status byte |
| `UhUl`          | After booting the sensor, it starts out at 15000 exactly, then typically settles to about 10500. According to the geektimes.ru page, this value is related to the minimum CO2 uncorrected concentration measured in the past day. So guess this is a relevant parameter from the ABC-algorithm. |


## Prerequisites

An MQTT broker is needed as the counterpart for this daemon.
MQTT is huge help in connecting different parts of your smart home and setting up of a broker is quick and easy.

## Installation

On a modern Linux system just a few steps are needed to get the daemon working.
The following example shows the installation under Debian/Raspbian below the `/opt` directory:

```shell
sudo apt install git python3 python3-pip

git clone https://github.com/R4scal/mhz19-mqtt-daemon.git /opt/mhz19-mqtt-daemon

cd /opt/mhz19-mqtt-daemon
sudo pip3 install -r requirements.txt
```

## Configuration

To match personal needs, all operation details can be configured using the file [`config.ini`](config.ini.dist).
The file needs to be created first:

```shell
cp /opt/mhz19-mqtt-daemon/config.{ini.dist,ini}
vim /opt/mhz19-mqtt-daemon/config.ini
```

## Execution

A first test run is as easy as:

```shell
python3 /opt/mhz19-mqtt-daemon/mhz19-mqtt-daemon.py
```

Using the command line argument `--config_dir`, a directory where to read the config.ini file from can be specified, e.g.

```shell
python3 /opt/mhz19-mqtt-daemon/mhz19-mqtt-daemon.py --config_dir /opt/mhz19-config
```

The extensive output can be reduced to error messages:

```shell
python3 /opt/mhz19-mqtt-daemon/mhz19-mqtt-daemon.py > /dev/null
```

## Run as systemd service

1. Add the following file as `/etc/systemd/system/mhz19_mqtt.service`.
```
[Unit]
Description=Get co2 sensor values and post to mqtt broker
After=rc-local.service

[Service]
WorkingDirectory=/opt/mhz19-mqtt-daemon/
User=pi
ExecStart=/usr/bin/python3 /opt/mhz19-mqtt-daemon/mhz19-mqtt-daemon.py
Restart=always
RestartSec=30
Type=simple
PIDFile=/var/run/mh-z19.pid

[Install]
WantedBy=multi-user.target
```
2. Change `WorkingDirectory` to the path you cloned this repo to. Change the path in `ExecStart`.
3. Reload systemd service files:
```
sudo systemctl daemon-reload
```
4. Start the service
```
sudo systemctl start mhz19_mqtt.service
```
5. Enable startup on boot
```
sudo systemctl enable mhz19_mqtt.service
```
