# gateway-ttn-configuration

Gateway LoRa configuration guide based on RHF0M301 on [TTN](https://www.thethingsnetwork.org/)

[Versão em português](#configuracoes-gateway-ttn)

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

![Gateway](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/gateway_montado.jpg)

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

[Gateway](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/console_2.png)

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

# configuracoes-gateway-ttn

Guia de configuração de Gateway LoRa na [TTN](https://www.thethingsnetwork.org/) utilizando RHF0M301

## Materiais utilizados

### Gateway:

* [Cartão SD](https://www.raspberrypi.org/documentation/installation/sd-cards.md)  
* Raspberry Pi 3 Model B V1.2 (**RPi**)
* Módulo Gateway LoRaWAN ([RHF0M301](https://www.robotshop.com/media/files/pdf/915mhz-lora-gateway-raspberry-pi-hat-datasheet1.pdf)) RISINGHF 
* Adaptador para módulo Gateway LoRaWAN
* Antena 915 MHz
* Fonte chaveada 5V 3A

![Materiais](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/gateway_componentes.jpg)

### Configuração do gateway:

* Monitor 
* Teclado
* Cabo HDMI

### Acesso a:
* [GitHub](https://github.com/)
* [The Things Network](https://www.thethingsnetwork.org/)

## Iniciando

Antes de tudo é necessário preparar o cartão SD. O passo a passo detalhado pode ser seguido a partir do [link](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) que basicamente consiste em:

* Download da imagem
* Escrita da imagem no cartão

## Montagem

* Insira o cartao SD na RPi. Encaixe o adaptador, o módulo gateway e a antena. Ao final você deve ter algo parecido com isso: 

![Gateway](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/gateway_montado.jpg)

Conecte a RPi e o adaptador a fonte e ao cabo Ethernet (não energise o módulo LoRa sem que a antena esteja conectada).

A conexão entre a RPi e o adaptador e entre o adaptador e o módulo devem ser perfeitas (todos os pinos machos conectados aos fêmeas) sem a necessidade da utilização de jumpers, como pode ser visto na imagem.

![Gateway finalizado](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/gateway_caixa.jpg)

> Aqui utilizamos uma caixa protetora para abrigar o gateway. Se fizer esta escolha tome cuidado para sempre deixar as entradas livres.

Caso você esteja utilizando outro módulo LoRa, ou até mesmo outro modo de alimentação para o módulo os pinos para a conexão entre ele e a RPi serão:

Descricao      | Pino físico na RPi 
:-------------:|:-----------------:
Alimentacao 5V | 2
GND            | 6
Reset          | 22
SPI CLK        | 23
MISO           | 21
MOSI           | 19
NSS            | 24

Agora está tudo pronto para a configuração do gateway. Caso você tenha acesso a uma rede LAN e não queira utilizar um monitor e um teclado para as configurações é necessário habilitar uma conexão SSH. Para isso basta criar um novo arquivo vazio chamado ssh (sem extensão) na partição de inicialização do cartão SD, qualquer dúvida é só dar uma olhada [aqui](https://www.raspberrypi.org/documentation/remote-access/ssh/) (tópico 3). Você pode também ligar o monitor e o teclado a RPi, e então liberar a conexão via SSH [aqui](https://www.raspberrypi.org/documentation/remote-access/ssh/) (tópico 2).

* Para o acesso via ssh, caso em uma rede LAN com o arquivo ssh na partição inicial:
```
$ ssh pi@raspberrypi.local
```

* Para o acesso via ssh, após a liberação feita pela RPi:
```
$ ssh pi@IP
```

A senha default para o usuário **pi** é **raspberry**.

## Configurações

Vale lembrar que o dispositivo deve estar conectado à Internet para seguir as proximas instruções. Essa conexão pode ser feita via cabo ou usando o [Wi-fi](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md).

### Configurações do dispositivo

Use o comando raspi-config para configurar local, timezone, habilitar o [SPI](https://pt.wikipedia.org/wiki/Serial_Peripheral_Interface) e [redimensionar a partição do cartão SD](https://jeffersonpalheta.wordpress.com/2017/09/25/redimensionar-particao-sd-card-raspberry-pi-raspbian-jessie/).
```
 $ sudo raspi-config
```
[4] Localization Options -> I1 Change Locale

[4] Localization Options -> I2 Change Timezone

[5] Interfacing options -> P4 SPI

[7] Advanced options -> A1 Expand filesystem

Ao sair, um pedido de reboot deve surgir, confirme (ou *$ sudo reboot* para fazer manualmente).

em seguida:

* Atualize o sistema e instale o git:
```
 $ sudo apt-get update
 $ sudo apt-get upgrade
 $ sudo apt-get install git
```

 A etapa a seguir é completamente opcional e de sua decisão!!

* Crie um novo usuário para TTN.
```
 $ sudo adduser ttn
 $ sudo adduser ttn sudo
```

* De um *reboot*, logue no sistema usando o usuário *ttn* e remova o usuário default *pi*
```
$ sudo userdel -rf pi
```

### Configurações do *gateway*

* Identificar EUI do dispositivo

Ao conectar ao terminal da RPi digite:
```
$ ifconfig
```

Uma tela parecida com a seguinte aparecerá:

![ifconfig - EUI](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/exemplo_ifconfig.png)

O número destacado em vermelho é o endereço MAC da RPi e será a base para o *Gateway* **EUI**. A este número devem ser adicionados 2 bytes **ff** no meio, portanto:

Se: 
>  b 8 : 2 7 : e b : f 9 : f f : 2 4

> *Gateway* EUI: b 8 : 2 7 : e b : **f** **f** : **f** **f** : f 9 : f f : 2 4


* Configurações remotas

Os gateways TTN podem ser ajustados para permitir configuração remota. Nesse caso, é verificado se há um novo arquivo de configuração em cada inicialização do dispositivo e, caso haja, o arquivo de configuração local é substituido.

Para utilizar esta opção é preciso criar um arquivo JSON com o nome da EUI no repositório [ttn-zh/gateway-remote-config](https://github.com/ttn-zh/gateway-remote-config).

Que consiste em:

- Criar um arquivo JSON com o **EUI** do gateway em letras maiúsculas, contendo as informações sobre o mesmo. 

- O arquivo deve ser adicionado usando *Create new file*:

![Create new file](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/create_new_file.png)

> Ex: Se o gateway EUI for B827EBFFFFF9FF24, o arquivo deverá ser chamado B827EBFFFFF9FF24.json

O conteúdo do arquivo deve ser:
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
    "ref_latitude": "LATITUDE",
    "ref_longitude": "LONGITUDE",
    "ref_altitude": "ALTITUDE",
    "contact_email": "SEU_EMAIL",
    "description": "Descrição do dispositivo"
  }
}
```

- Propor a adição do arquivo:

![Propose new file](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/propose_new_file.png)

![Create pull request1](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/create_pull_request1.png)

![Create pull request2](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/create_pull_request2.png)

Agora é só esperar ele ser inserido (não deve demorar muito, você deve receber um email com a confirmação).

* Dando sequencia as configurações:

- Clonar e executar os seguintes repositórios

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

* Remova o arquivo **global_config.json** (que está em: *$ cd lora_pkt_fwd*) 

```
$ sudo rm -rf /opt/packet_forwarder/lora_pkt_fwd/global_config.json
```

* Crie um novo (em /opt/packet_forwarder/lora_pkt_fwd/) com o conteúdo disponibilizado no arquivo **US-global_conf.json** que se encontra [neste](https://github.com/TheThingsNetwork/gateway-conf/) repositório

```
$ cd /opt/packet_forwarder/lora_pkt_fwd/
$ sudo curl -o global_conf.json https://raw.githubusercontent.com/TheThingsNetwork/gateway-conf/master/EU-global_conf.json
```

* Substitua o **gateway_ID** no arquivo **local_config.json** pelo EUI do *gateway*

```
$ sudo nano local_conf.json
```

> Fique a vontade para usar o editor de texto que preferir

Conteudo:

```
{
/* Put there parameters that are different for each gateway (eg. pointing one gateway to a test server while the others stay in production) */
/* Settings defined in global_conf will be overwritten by those in local_conf */
  "gateway_conf": {
    "gateway_ID": "XXXXXXXXXXXXXXXX" /* you must pick a unique 64b number for each gateway (represented by an hex string) */
  }
}
```

### Utilização do gateway em *background*

* Configure o *service* no *systemd* criando o arquivo *gateway.service*
```
$ nano /etc/systemd/system/gateway.service
```

* Inserir o conteúdo em *gateway.service*:
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

* Iniciar o *service*

Execute as seguintes linhas para que o script do gateway rode em *background* sempre que o RPi for inicializado:
```
$ sudo systemctl daemon-reload
$ sudo systemctl enable gateway
$ sudo systemctl start gateway
```

* Conferir se o serviço está rodando
```
sudo systemctl status gateway -l
```

### Registro na TTN

Agora você pode registrar o seu gateway na TTN!

* Partindo do princípio que você já possui uma conta na TTN e está logada nela vá para o **Console**

![Console](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/console_1.png)

* Clique em **Gateways**

![Gateway](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/console_2.png)

* *register gateway*

![registrar_gateway](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/console_3.png)

* Marque a opção *I'm using the legacy packet forwarder*

![box](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/box.png)

* Complete as informações restantes

- Gateway EUI : Identidade previamente identificada
- Description: Descrição simplificada para o seu gateway
- Frequency Plan: Frequência utilizada pelo gateway
- Router: O roteado mais proximo ao seu gateway
- Antenna Placement: Onde a antena está localizada

* e finalmente *Register Gateway*

![register](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/register_ok.png)

* Se tudo estiver ok, o status do gateway deve ser *conected*

![connected](https://github.com/mftutui/configuracoes-gateway-ttn/blob/master/images/gateway_ok.png)
