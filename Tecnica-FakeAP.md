#### Tecnica EVIL TWIN

Primeiro é necessario instalar algumas ferramentas, hostapd que sera nosso servidor AP e dnsmasq sera nosso servidor DNS e nosso servidor DCHP, com o IPTABLES faremos o redirecionamento dos pacotes para nossa interface com acesso a internet
```
apt install hostapd dnsmasq iptables
```

Agora precisamos criar o arquivo de configuração do hostapd

```
nano /etc/hostapd/hostapd.conf
```
Vamos adicionar as seguintes configurações, claro que devemos adaptar ao nosso ambiente, como alterar o ssid para o nome especifico em questão, e a interface para sua placa de rede
```
interface=wlan0
driver=nl80211
ssid=MeuAP
hw_mode=g
channel=6
ieee80211n=1
wmm_enabled=1
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
ctrl_interface=/var/run/hostapd
ctrl_interface_group=0
```
Agora vamos configurar o hostapd para usar nosso arquivo de configuração
```
nano /etc/default/hostapd
```
Adicionar a seguinte linha
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
Agora vamos configurar o dnsmasq para usar trabalhar com DHCP
```
nano /etc/dnsmasq.conf
```
Adicionar as seguintes configurações, lembrando de adaptar a interface de rede para sua interface em questão
```
interface=wlan0
dhcp-range=192.168.10.10,192.168.10.50,12h
dhcp-option=3,192.168.10.1
dhcp-option=6,192.168.10.1
server=8.8.8.8
log-queries
log-dhcp
```
Agora vamos definir os endereços estaticos para a interface do nosso AP
```
ip addr add 192.168.10.1/24 dev wlan0
```
Agora precisamos habilitar o redirecionamento de IP
```
sysctl -w net.ipv4.ip_forward=1
```
Agora vamos usar o IPTABLES para criar uma regra de NAT para compartilhar internet com nossa interface de rede conectada a rede (no meu caso minha interface cabeada)
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
Agora apenas precisamos iniciar o nosso hostapd e o dnsmasqd
```
systemctl start hostapd
systemctl start dnsmasq
```
Uma vez iniciado os serviços, o AP deve estar on e com acesso a internet.<br><br>
A rede não póssui criptografia, então basta subir uma placa de rede em modo monitoramento para analizar o trafego de rede gerado pelos clientes com wireshark
<br><br>
Tambem é possivel analizar os clientes conectados com o comando "arp" para realizar um arpspoofing




