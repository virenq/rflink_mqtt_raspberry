# RFLink Gateway to MQTT

## Requirements

Only for raspbian (cf RFLinkGateway.py to modify)
pip install tornado pyserial paho-mqtt

## Purpose
Bridge between RFLink Gateway and MQTT broker.

## Input data
Forwarding messages received on TTY port from RFLink Gateway Arduino board
to MQTT broker in both directions.

Every message received from RFLinkGateway is split into single parameters
and published to different MQTT topics.
Example:
Message:
`20;83;Oregon Rain2;ID=2a19;RAIN=002a;RAINTOT=0054;BAT=OK;`

 is translated to following topics:

 `/data/RFLINK/Oregon Rain2/2a19/R/RAIN 002a`

 `/data/RFLINK/Oregon Rain2/2a19/R/RAINTOT 0054`

 `/data/RFLINK/Oregon Rain2/2a19/R/BAT OK`




Every message received on particular MQTT topic is translated to
RFLink Gateway and sent to 433 MHz.

## Output data
Application pushes informations to MQTT broker in following format:
[mqtt_prefix]/[device_type]/[device_id]/R/[parameter]

`/data/RFLINK/TriState/8556a8/W/1 OFF`

Every change should be published to topic:
[mqtt_prefix]/[device_type]/[device_id]/W/[switch_ID]

`/data/RFLINK/TriState/8556a8/W/1 ON`

## Configuration

Whole configuration is located in config.json file.

```json
{
  "mqtt_host": "your.mqtt.host",
  "mqtt_port": 1883,
  "mqtt_prefix": "/data/RFLINK",
  "mqtt_username": "xxxxx",
  "mqtt_password": "xxxxx",
  "rflink_tty_device": "/dev/ttyUSB0",
  "rflink_direct_output_params": ["BAT", "CMD", "SET_LEVEL", "SWITCH", "HUM", "CHIME", "PIR", "SMOKEALERT"]
}
```

config param | meaning
-------------|---------
| mqtt_host | MQTT broker host |
| mqtt_port | MQTT broker port|
| mqtt_prefix | prefix for publish and subscribe topic|
| mqtt_username | MQTT broker user|
| mqtt_password | MQTT broker password|
| rflink_tty_device | Arduino tty device |
| rflink_ignored_devices | Parameters transferred to MQTT without any processing|

## service

File creation:
`sudo nano /lib/systemd/system/rflinkmqtt.service`

File contents:
`[Unit]
Description=My Script Service
After=multi-user.target

[Service]
Type=idle
ExecStart=/usr/bin/python /home/pi/rflink_mqtt_raspberry/RFLinkGateway.py

[Install]
WantedBy=multi-user.target`

Rights management:
`sudo chmod 644 /lib/systemd/system/rflinkmqtt.service`


Validate and restart services:
`sudo systemctl daemon-reload`
`sudo systemctl enable`

Restart the computer
`sudo reboot`

Check
`sudo service rflinkmqtt.service status`


## References
- RFLink Gateway project http://www.nemcon.nl/blog2/
- RFLink Gateway protocol http://www.nemcon.nl/blog2/protref
- RFLingGateway Fork https://github.com/cephos/RFLinkGateway
- RFLingGateway https://github.com/Iture/RFLinkGateway
