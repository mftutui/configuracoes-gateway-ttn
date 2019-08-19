# gateway-ttn-configuration

Gateway LoRa configuration guide based using RHF0M301 on [TTN](https://www.thethingsnetwork.org/)

[Versão em português <span>&#x1f1e7;&#x1f1f7;</span>](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/README.md)

## Hardware

### Gateway:

* [SD card](https://www.raspberrypi.org/documentation/installation/sd-cards.md)
* Raspberry Pi 3 Model B V1.2 (**RPi**)
* LoRa Gateway ([RHF0M301](https://www.robotshop.com/media/files/pdf/915mhz-lora-gateway-raspberry-pi-hat-datasheet1.pdf))
* Gateway LoRaWAN supply adapter
* Antenna 900MHz 6dBi Omni
* Power supply 5V 3A

![Hardware](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/gateway_componentes.jpg)

### Setting up the gateway:

* Monitor 
* Keybord
* HDMI cable

### Access to:
* [GitHub](https://github.com/)
* [The Things Network](https://www.thethingsnetwork.org/)

## Starting

First of all it is necessary to prepare the SD card. You can follow the detailed step-by-step [here](https://www.raspberrypi.org/documentation/installation/installing-images/README.md).

* Download the image
* Upload the image into the SD card

## Setting up the hardware

* Insert the SD card on the RPi, connect the adapter, the gateway module and the antenna.

![Gateway](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/gateway_montado.jpg)

Connect the ethernet cable to the RPi and power up (do nor power up the LoRa module without the antenna).

The connection between the adapter and the RPi must be perfect (all the male pins connected to the females) without using jumpers, as it can seen in the image below.

![The gateway](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/gateway_caixa.jpg)

> Here we use a protection box to keep the gateway protected. If you make this choice, make sure you leave the entries free.

If you are using another LoRa module, or even another board to power up the module, the pins to make the connection between it and the RPi will be:

Description    | RPi physical pin
:-------------:|:-----------------:
Supply 5V      | 2
GND            | 6
Reset          | 22
SPI CLK        | 23
MISO           | 21
MOSI           | 19
NSS            | 24

Now you are ready to start the gateway configuration. 
If you have access to a LAN and don't want to use a monitor and a keyboard for the settings, you will need to create a new empty file named ssh (no extension) in the SD card boot partition. Or connect the monitor and keyboard to the RPi.

* To the ssh access:
```
local $ ssh pi@raspberrypi.local
```
The default password to **pi** user is **raspberry**.

## Configurations

### Device settings

Use the raspi-config command to enable the [SPI](https://pt.wikipedia.org/wiki/Serial_Peripheral_Interface) and [resize the SD card partition](https://jeffersonpalheta.wordpress.com/2017/09/25/redimensionar-particao-sd-card-raspberry-pi-raspbian-jessie/).
  
```
 $ sudo raspi-config
```
[5] Interfacing options -> P4 SPI

[7] Advanced options -> A1 Expand filesystem

A reboot request should come up (or * $ sudo reboot * to do manually).

* Configure location and time zone 
```
 $ sudo dpkg-reconfigure locales
 $ sudo dpkg-reconfigure tzdata
```

* Make sure you have an updated installation and install git:
```
 $ sudo apt-get update
 $ sudo apt-get upgrade
 $ sudo apt-get install git
```
 The two following steps are completely opcional and are up to you.

* Create new user for TTN and add it to *sudoers* file.
```
 $ sudo adduser ttn
 $ sudo adduser ttn sudo
```

* Reboot and login as ttn. You can now remove the default *pi* user.
```
$ sudo userdel -rf pi
```

### *Gateway* set up

* Identify the device EUI 

Connected to the RPi terminal:

```
$ ifconfig
```

A screen will appear, as following:

![ifconfig - EUI](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/exemplo_ifconfig.png)

The highlighted number is the RPi MAC address, and will be the base to our *Gateway* **EUI**. For this number we must add 2 bytes **ff** just in the midle.

So: 
> HWaddr: b 8 : 2 7 : e b : f 9 : f f : 2 4

> *Gateway* EUI: b 8 : 2 7 : e b : **f** **f** : **f** **f** : f 9 : f f : 2 4

Enable the checkbox *"I'm using the legacy packet forwarder* to set the gatewat on TTN.

* Remote configurations

TTN gateways using this setup can be set to allow remote configuration. In this case, the gateways checks if there's a new setting file and replaces the local configuration file.

To use this option you need to create an JSON file with the EUI name in [ttn-zh/gateway-remote-config](https://github.com/ttn-zh/gateway-remote-config) repository.

- Create a JSON file with the **EUI** of the gateway in uppercase.

e.g.: > If the gateway EUI is B827EBFFFFF9FF24, the file should be called B827EBFFFFF9FF24.json

The content of the file:
```json
{
  "gateway_conf": {
    "gateway_ID": "GATEWAY_EUI",
    "servers": [
      {
        "server_address": "router.us.thethings.network",
        "serv_port_up": 1700,
        "serv_port_down": 1700,
        "serv_enabled": true
      }
    ],
    "ref_latitude": "latitude",
    "ref_longitude": "longitude",
    "ref_altitude": "altitude",
    "contact_email": "email",
    "description": "descrption"
  }
}
```

- The file must be added (*New pull request*) to the repository. Now just wait for it to be inserted (it should not take too long, you should receive an email with the confirmation).

* Gateway settings:

- Clone and execute the following repositories

```
$ cd /opt
$ sudo git clone https://github.com/Lora-net/packet_forwarder
$ sudo git clone https://github.com/Lora-net/lora_gateway
```

```
$ cd /opt/lora_gateway
$ sudo make -j4
$ cd /opt/packet_forwarder
$ sudo make -j4
```

* Remove the file **global_config.json** (which is in: *$ cd lora_pkt_fwd*) 

```
$ sudo rm -rf /opt/packet_forwarder/lora_pkt_fwd/global_config.json
```

* Create a new one (on /opt/packet_forwarder) with the content available in the **US-global_conf.json**, file that is [here](https://github.com/TheThingsNetwork/gateway-conf/).

```
$ sudo curl -o global_conf.json https://raw.githubusercontent.com/TheThingsNetwork/gateway-conf/master/EU-global_conf.json
```

* Replace the **gateway_ID** in the **local_config.json** to the *gateway* EUI.

```
$ sudo nano local_config.json
```
Content:

```
{
/* Put there parameters that are different for each gateway (eg. pointing one gateway to a test server while the others stay in production) */
/* Settings defined in global_conf will be overwritten by those in local_conf */
  "gateway_conf": {
    "gateway_ID": "XXXXXXXXXXXXXXXX" /* you must pick a unique 64b number for each gateway (represented by an hex string) */
  }
}
```

### Using the gateway in *background*

* Configure the *service* in the *systemd*
```
$ nano /etc/systemd/system/gateway.service
```

* Insert the content:
```
[Unit]
Description=TTN Gateway Service
After=multi-user.target
[Service]
WorkingDirectory=/opt/packet_forwarder/lora_pkt_fwd
Type=simple
ExecStartPre=/opt/lora_gateway/reset_lgw.sh start
ExecStart=/opt/packet_forwarder/lora_pkt_fwd/lora_pkt_fwd
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
```

* Start the *service*

Execute the following commands to run the script that will keep the gateway up in *background* whenever the RPi is connected: 
```
$ sudo systemctl daemon-reload
$ sudo systemctl enable gateway
$ sudo systemctl start gateway
```

* Check if the service is running
```
sudo systemctl status gateway -l
```

### TTN registry

Now you can register your gateway!

* Assuming you already have a TTN account and you are logged in, then go to **Console**

![Console](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/console_1.png)

* Go to **Gateways**

![Gateway](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/console_2.png)

* *register gateway*

![registrar_gateway](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/console_3.png)

* Enable the checkbox *I'm using the legacy packet forwarder* 

![box](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/box.png)

* Complete the remaining information

- Gateway EUI : The EUI of the gateway
- Description: readable description of the gateway
- Frequency Plan: The frequency plan this gateway will use
- Router:  The closer router to the location of the gateway
- Antenna Placement: The placement of the gateway antenna

* and *Register Gateway*

![register](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/register_ok.png)

* The gateway status should be *conected*

![connected](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/gateway_ok.png)

----------------------------------------------------------------------------------------------------------
