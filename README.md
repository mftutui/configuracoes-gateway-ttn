# configuracoes-gateway-ttn

Guia de configuração de Gateway LoRa na [TTN](https://www.thethingsnetwork.org/) utilizando RHF0M301.

## Materiais utilizados

Gateway:

* Cartao SD 
* Raspberry Pi 3 Model B V1.2 (RPi)
* Módulo Gateway LoRaWAN (RHF0M301) RISINGHF
* Adaptador para módulo Gateway LoRaWAN
* Antena 915 MHz
* Fonte chaveada 5V 3A

![Materiais](https://github.com/mftutui/configuracoes-gateway-ttn/gateway_componentes.jpg)

Configuração do gateway:
* Monitor 
* Teclado
* Cabo HDMI

## Installing

Antes de tudo é necessário preparar o cartão SD. O passo a passo detalhado pode ser seguido a partir do [link](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) que basicamente consiste em:

* Download da imagem
* Escrita da imagem no cartão

## Montagem

* Insira o cartao SD na RPi
* Conecte do adaptador para o modulo gateway
* Conecte o módulo gateway
* Conecte a antena 

ao final você deve ter algo parecido com isso: 

![Gateway montado](https://github.com/mftutui/configuracoes-gateway-ttn/gateway_montado.jpg)

Conecte a RPi e o adaptador a fonte e ao cabo ethernet.
(nao esqueça, nunca energise o módulo LoRa sem que a antena esteja conectada).

A conexão entre a RPI e o adaptador e entre o adaptador e o módulo devem ser perfeitas (todos os pinos machos conectados aos fêmeas) sem a necessidade da utilização de jumpers.

![Gateway montado](https://github.com/mftutui/configuracoes-gateway-ttn/gateway_caixa.jpg)

Caso você esteja utilizando outro módulo LoRa, ou até mesmo uma outra placa para alimentação do módulo os pinos para a conexão entre ele e a RPi serão:

Descricao      | Pino físico na RPi 
:-------------:|:------------------
Alimentacao 5V | 2
GND            | 6
Reset          | 22
SPI CLK        | 23
MISO           | 21
MOSI           | 19
NSS            | 24

Agora está tudo pronto para a configuração do gateway. Caso você tenha acesso a uma rede LAN e não queira utilizar um monitor e um teclado para as configurações é necessário criar um novo arquivo vazio chamado ssh (sem extensão) na partição de inicialização do cartão SD. Isso vai habilitar o ssh na inicialização da RPI

ou você pode ligar o monitor e o teclado a RPI.

Para o acesso via ssh:

```
local $ ssh pi@raspberrypi.local
```
A senha default para o usuário pi é raspberry.

## Configurações

### Configurações do dispositivo

Use o comando raspi-config para habilitar o [SPI](https://pt.wikipedia.org/wiki/Serial_Peripheral_Interface) e [redimensionar a partição do cartão SD](https://jeffersonpalheta.wordpress.com/2017/09/25/redimensionar-particao-sd-card-raspberry-pi-raspbian-jessie/).
  
```
 $ sudo raspi-config
```
[5] Interfacing options -> P4 SPI
[7] Advanced options -> A1 Expand filesystem

Um pedido de reboot deve surgir, confirme (ou *$ sudo reboot* para fazer manualmente).

em seguida:

* Configure o local e time zone 
```
 $ sudo dpkg-reconfigure locales
 $ sudo dpkg-reconfigure tzdata
```

* Atualize o sistema e instale o git:
```
 $ sudo apt-get update
 $ sudo apt-get upgrade
 $ sudo apt-get install git
```
 As etapas a seguir são completamente opcionais e de sua decisão!!

* Crie um novo usuário para TTN e adicione ao arquivo sudoers.
```
 $ sudo adduser ttn
 $ sudo adduser ttn sudo
```

Para evitar que o sistema solicite a senha de root regularmente adicione o usuário TTN no arquivo sudoers.

```
$ sudo visudo
```

> Adicione a linha:  ttn ALL=(ALL) NOPASSWD: ALL

Cuidado, isso permite que um console conectado com o usuário ttn emita quaisquer comandos no sistema, sem qualquer controle de senha.

* De um *reboot* e logue no sistema usando o usuário ttn e remova o usuário default pi 
```
$ sudo userdel -rf pi
```

### Configurações do *gateway*

* Identificar EUI do dispositivo

A identificação é bem simples, ao conectar ao terminal da RPi digite:
```
$ ifconfig
```

Uma tela parecida com a seguinte aparecerá:

![ifconfig - EUI](https://github.com/mftutui/configuracoes-gateway-ttn/exemplo_ifconfig.jpg)

O número destacado em vermelho é o endereço MAC da RPi e será a base para o *Gateway* **EUI**. A este número devem ser adicionados 2 bytes **ff** no meio, portanto:

> HWaddr: b8:27:eb:f9:ff:24 => *Gateway* EUI: b8:27:eb:**ff**:**ff**:f9:ff:24

O box *I'm using the legacy packet forwarder* deve ser marcado ao fazer a adição do *gateway* na TTN. 


* Configurações remotas

Os gateways TTN podem ser ajustados para permitir configuração remota. Nesse caso, é verificado se há um novo arquivo de configuração em cada inicialização do dispositivo e, caso haja, o arquivo de configuração local é substituido.

Para utilizar esta opção é preciso criar um arquivo JSON com o nome da EUI no repositório [ttn-zh/gateway-remote-config](https://github.com/ttn-zh/gateway-remote-config).


Que basicamente consiste em:
Criar um arquivo JSON com o **EUI** do gateway em letras maiúsculas. 

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

O arquivo deve ser adicionado (*New pull request*) ao repositório, agora é só esperar ele ser inserido (não deve demorar muito, você deve receber um email com a confirmação).


Dando sequencia as configurações:

Clonar e executar os seguintes repositórios

```
$ cd /opt
$ sudo git clone https://github.com/Lora-net/packet_forwarder
$ sudo git clone https://github.com/Lora-net/lora_gateway
```

```
cd /opt/lora_gateway
sudo make -j4
cd /opt/packet_forwarder
sudo make -j4
```

Remova o arquivo **global_config.json** (que está em: *$ cd lora_pkt_fwd*) e crie um novo com o conteúdo disponibilizado no arquivo **US-global_conf.json** que se encontra [neste](https://github.com/TheThingsNetwork/gateway-conf/) repositório.

Substitua o **gateway_ID** no arquivo **local_config.json** pelo EUI do *gateway*.

´´´
$ sudo nano local_config.json
´´´
Conteúdo:
´´´json
{
/* Put there parameters that are different for each gateway (eg. pointing one gateway to a test server while the others stay in production) */
/* Settings defined in global_conf will be overwritten by those in local_conf */
  "gateway_conf": {
    "gateway_ID": "XXXXXXXXXXXXXXXX" /* you must pick a unique 64b number for each gateway (represented by an hex string) */
  }
}
´´´

* Utilização do gateway em *background*

Configure o *service* no *systemd*

```
$ nano /etc/systemd/system/gateway.service
```

Inserir o conteúdo:

```json
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

Execute as seguintes linhas para que o script do gateway rode em *background* sempre que o RPi estiver ligado:

```
$ sudo systemctl daemon-reload
$ sudo systemctl enable gateway

```
* Conferir se o serviço está rodando

```
sudo systemctl status gateway -l
```





