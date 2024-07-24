
### CRIANDO PENDRIVE PARA COLDBOOT

Primeiro precisamos baixar o grub4dos bem como os ultilitarios necessarios para fazer o dump da ram, tudo isso ja esta nos repositorios do github do [baselsayeh](https://github.com/baselsayeh/coldboot-tools/), podemos baixar o pacote ja compilado para bootar via MBR
```
root@debian:/home/messias# wget https://github.com/baselsayeh/coldboot-tools/releases/download/2/bios_memimage64.zip
root@debian:/home/messias# unzip bios_memimage64.zip
root@debian:/home/messias# cd bios_memimage/
root@debian:/home/messias/bios_memimage# ls
 grldr       menu.lst            scraper32.bin            scraper64.bin
 grldr.mbr   menu_sec_part.lst   scraper32_haltonly.bin   scraper64_haltonly.bin
```
Primeiro temos que gravar o grldr.mbr na nossa unidade USB, para isso podemos usar o dd
```
root@debian:/home/messias/bios_memimage# dd if=grldr.mbr of=/dev/sde 
16+0 records in
16+0 records out
8192 bytes (8,2 kB, 8,0 KiB) copied, 0,0044466 s, 1,8 MB/s
```
Agora precisamos criar 2 partições, a primeira partição sera usada para armazenar os arquivos de inicialização bem como os arquivos para scraping da memoria, a segunra partição vai armazenar o dump da memoria, vou criar a primeira partição com 1Mb e a segunda ocupando todo restante do espaço do disco

```
root@debian:/home/messias/bios_memimage# fdisk /dev/sde

Welcome to fdisk (util-linux 2.38.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (1-4, default 1): +1M
Value out of range.
Partition number (1-4, default 1): 
First sector (2048-31260671, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-31260671, default 31260671): +1M

Created a new partition 1 of type 'Linux' and of size 1 MiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (2-4, default 2): 
First sector (4096-31260671, default 4096): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4096-31260671, default 31260671): 

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

root@debian:/home/messias/bios_memimage#
```
Assim ficara nossa unidade USB
```
root@debian:/home/messias/bios_memimage# fdisk /dev/sde -l
Disk /dev/sde: 14,91 GiB, 16005464064 bytes, 31260672 sectors
Disk model: Cruzer Blade    
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000

Device     Boot Start      End  Sectors  Size Id Type
/dev/sde1        2048     4095     2048    1M 83 Linux
/dev/sde2        4096 31260671 31256576 14,9G 83 Linux
root@debian:/home/messias/bios_memimage# 
```
Agora vamos formatar a primeira partição com fat e a segunda partição com ntfs

```
root@debian:/home/messias/bios_memimage# mkfs.fat /dev/sde1
mkfs.fat 4.2 (2021-01-31)
root@debian:/home/messias/bios_memimage# mkfs.ntfs /dev/sde2
Cluster size has been automatically set to 4096 bytes.
Initializing device with zeroes: 100% - Done.
Creating NTFS volume structures.
mkntfs completed successfully. Have a nice day.
root@debian:/home/messias/bios_memimage# 
```

Vamos montar a primeira partição e mover todos os arquivos .lst .bin e o grldr para la
```
root@debian:/home/messias/bios_memimage# ls
 grldr       menu.lst            scraper32.bin            scraper64.bin
 grldr.mbr   menu_sec_part.lst   scraper32_haltonly.bin   scraper64_haltonly.bin
root@debian:/home/messias/bios_memimage# mount /dev/sde1 /mnt
root@debian:/home/messias/bios_memimage# cp grldr *.lst *.bin /mnt
root@debian:/home/messias/bios_memimage# umount /mnt
root@debian:/home/messias/bios_memimage# 
```
A segunda partição sera usada para armazenar o dump da ram, porem para isso o binario do scraping espera substituir um arquivo chamado "ram.img" que devemos criar na partição, eu vou criar um arquivo vazio de 2gb (porem é recomendado criar uma imagem maior do que a ram que você ira dumpar)
```
root@debian:/home/messias/bios_memimage# mount /dev/sde2 /mnt
root@debian:/home/messias/bios_memimage# fallocate -l 2G ram.img
root@debian:/home/messias/bios_memimage# mv ram.img /mnt
root@debian:/home/messias/bios_memimage# umount /mnt
root@debian:/home/messias/bios_memimage# 

```
Agora a unidade USB esta pronta para ser bootada, uma vez inicializada o dump ira iniciar e sera armazenada no arquivo ram.img dentro da segunda partição da unidade




