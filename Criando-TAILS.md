#### Instalando pacotes extras
```
net-tools tor gnome-terminal iptables
```
Configurando PATH
```
export PATH=$PATH:/sbin/
```

#### Configurar o Tor
```
nano /etc/tor/torrc
```
```
# Configurar TransPort
TransPort 9040

# Configurar DNSPort
DNSPort 5353

# Configurar para resolver DNS através do Tor
AutomapHostsOnResolve 1
```

#### Configurando IPTABLES
```
nano /etc/iptables.rules
```

```
#!/bin/sh

# Limpar regras existentes
iptables -F
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X
iptables -X

# Configurar política padrão para bloquear tudo
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP

# Permitir tráfego local
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Permitir tráfego de e para o Tor
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -d 127.0.0.1/32 -j ACCEPT

# Redirecionar tráfego DNS e TCP para o Tor
iptables -t nat -A OUTPUT -p tcp --syn -j REDIRECT --to-ports 9040
iptables -t nat -A OUTPUT -p udp --dport 53 -j REDIRECT --to-ports 5353

# Permitir que o Tor se conecte à rede Tor
iptables -A OUTPUT -m owner --uid-owner debian-tor -j ACCEPT

# Bloquear todo o tráfego restante
iptables -A OUTPUT -j REJECT
```

#### Aplicar as Regras de IPTABLES

```
chmod +x /etc/iptables.rules
nano /etc/systemd/system/iptables.service
```
```
[Unit]
Description=Regras IPTABLES

[Service]
Type=oneshot
ExecStart=/etc/iptables.rules
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

```
systemctl enable iptables
systemctl start iptables
```

#### Reiniciar para testar configurações
```
reboot
```
Aqui precisa testar se as configurações foram aplicadas
```
iptables -L -v
iptables -t nat -L -v
```
