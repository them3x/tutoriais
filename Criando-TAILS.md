#### Criando Disco virtual

criar um arquivo para ser o nosso disco virtual.
```
dd if=/dev/zero of=disco-virtual bs=1M count=8192
```

Criando um dispositivo virtual usando o arquivo disco-virtual.
```
losetup /dev/loop0 -P disco-virtual
```

Verificando:
```
fdisk -l /dev/loop0

Disk /dev/loop0: 3 GiB, 3221225472 bytes, 6291456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

##### Criando novo particionamento
criar um novo particionamento GPT.
```
fdisk /dev/loop0

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): g
Created a new GPT disklabel (GUID: 7B7598F2-48D3-2E4B-A1F7-FA69F933D733).
The old iso9660 signature will be removed by a write command.

Command (m for help): w

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Vendo como ficou:
```
fdisk -l /dev/loop0

Disk /dev/loop0: 3 GiB, 3221225472 bytes, 6291456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 471CE3D7-C134-3A49-A869-2729666C8626
```

Nossa tabela de partições terá 3 partições.

    (1M) Primeira para acomodar arquivos do GRUB para BIOS.
    (50M) Para acomodar o GRUB para UEFI.
    (~3G) Para os arquivos do Debian.

##### Criando partições
```
fdisk  /dev/loop0

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): n
Partition number (1-128, default 1):
First sector (2048-6291422, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-6291422, default 6291422): +1M

Created a new partition 1 of type 'Linux filesystem' and of size 1 MiB.

Command (m for help): t
Selected partition 1
Partition type (type L to list all types): 4
Changed type of partition 'Linux filesystem' to 'BIOS boot'.

Command (m for help): n
Partition number (2-128, default 2):
First sector (4096-6291422, default 4096):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4096-6291422, default 6291422): +50M

Created a new partition 2 of type 'Linux filesystem' and of size 50 MiB.

Command (m for help): t
Partition number (1,2, default 2):
Partition type (type L to list all types): 1

Changed type of partition 'Linux filesystem' to 'EFI System'.

Command (m for help): n
Partition number (3-128, default 3):
First sector (106496-6291422, default 106496):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (106496-6291422, default 6291422):

Created a new partition 3 of type 'Linux filesystem' and of size 3 GiB.

Command (m for help): x
Expert command (m for help): A
Partition number (1-3, default 3): 1

The LegacyBIOSBootable flag on partition 1 is enabled now.

Expert command (m for help): r

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
```

Vendo como ficou:
```
fdisk -l /dev/loop0

Disk /dev/loop0: 3 GiB, 3221225472 bytes, 6291456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 471CE3D7-C134-3A49-A869-2729666C8626

Device        Start     End Sectors Size Type
/dev/loop0p1   2048    4095    2048   1M BIOS boot
/dev/loop0p2   4096  106495  102400  50M EFI System
/dev/loop0p3 106496 6291422 6184927   3G Linux filesystem
```

Para instalar o Debian vamos usar uma ferramenta que automatiza o processo chamada grml-debootstrap.
```
apt install grml-debootstrap
```

Fazendo a instalação
```
grml-debootstrap --release buster --efi /dev/loop0p2 --target /dev/loop0p3 --grub /dev/loop0 --hostname debianilo --password senharoot

 * EFI support enabled now.
 * grml-debootstrap [0.90] - Please recheck configuration before execution:

   Target:          /dev/loop0p3
   Install grub:    /dev/loop0
   Install efi:     /dev/loop0p2
   Using release:   buster
   Using hostname:  debianilo
   Using mirror:    http://deb.debian.org/debian
   Using arch:      amd64
   Config files:    /etc/debootstrap

   Important! Continuing will delete all data from /dev/loop0p3!

 * Is this ok for you? [y/N]
```

#### Instalando pacotes extras
Terminada a instalação do Debian vamos "chrootar" o sistema novo instalado no disco virtual para instalar mais pacotes.

```
mount /dev/loop0p3 /mnt
mount /dev/loop0p2 /mnt/boot/efi

mount --bind /proc /mnt/proc
mount --bind /dev /mnt/dev
mount --bind /sys /mnt/sys

chroot /mnt

df -h

Filesystem      Size  Used Avail Use% Mounted on
/dev/loop0p3    2.9G  986M  1.8G  36% /
/dev/loop0p2     50M  130K   50M   1% /boot/efi
udev            1.9G     0  1.9G   0% /dev
```
Configurar locales
```
dpkg-reconfigure locales
```
Criar novo usuario
```
adduser usuario
```
Instalar alguns pacotes que serão necessarios
```
apt install network-manager shim-signed net-tools tor gnome-terminal iptables
```
Mapear teclado
```
apt install console-setup
```

Para dar boot UEFI
```
mkdir /boot/efi/EFI/BOOT
cp  /boot/efi/EFI/debian/grubx64.efi /boot/efi/EFI/BOOT/BOOTX64.EFI
```
O disco virtual já é bootável usando UEFI, agora vamos prepará-lo para bootar também usando BIOS.
```
grub-install --target=i386-pc --recheck /dev/loop0
Instalando para a plataforma i386-pc.
Installation finished. No error reported.
```
Recriando o grub.cfg

Vamos criar um grub.cfg que não coloque os sistemas que você porventura tenha na sua máquina.
```
chmod -x /etc/grub.d/30_os-prober
update-grub
```
Instalar ambiente desktop
```
apt install mate-desktop-environment lightdm
apt clean
```

#### Saindo do CHROOT e finalizando
```
exit ou ctrl-d
```
Configurando FSTAB
```
apt install arch-install-scripts
genfstab -U /mnt >/mnt/etc/fstab
```

Veja como ficou:
```
cat /mnt/etc/fstab

# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# /dev/loop0p3
UUID=43edb6ee-3109-44a5-843c-12f501a2fa76	/         	ext4      	rw,relatime	0 1

# /dev/loop0p2 LABEL=EFI\134x20System
UUID=414A-DE3E      	/boot/efi 	vfat      	rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro	0 2
```
Agora basta desmontar as partições
```
umount /mnt/proc
umount /mnt/dev
umount /mnt/sys
umount /mnt/boot/efi
umount /mnt/
```
Instalando ISO no pendrive
```
dd if=disco-virtual of=/dev/sdb bs=16M oflag=sync status=progress
```

#### APOS BOOTAR NOVO SISTEMA

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

sleep 10

# Redirecionar tráfego DNS e TCP para o Tor
iptables -t nat -A OUTPUT -p tcp --syn -j REDIRECT --to-ports 9040
iptables -t nat -A OUTPUT -p udp --dport 53 -j REDIRECT --to-ports 5353

```

#### Aplicar as Regras de IPTABLES

```
chmod +x /etc/iptables.rules
nano /etc/systemd/system/iptables.service
```
```
[Unit]
Description=Regras IPTABLES
After=network.target network-online.target nss-lookup.target multi-user.target
Wants=network-online.target

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
