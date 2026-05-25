Изучаемый стек:
- DHCP
- PXE
- TFTP
- NFS
- Network Loading

Теоретические материалы
- [Автоинстал дистра](https://docs.altlinux.org/ru-RU/alt-server/11.1/html/alt-server/install-distro--autoinstall--chapter.html)
- [Загрузка по сети](https://docs.altlinux.org/ru-RU/alt-server/11.1/html/alt-server/netinst.html)

```bash
sudo apt-get update && sudo apt-get install alterator-netinst
```

```bash
sudo nano /etc/xinetd.d/tftp
```

`disable = yes` --> `disable = no`

Топология сети:
Internet
    |
  [NAT]
    |
ALT Server
    |
Host-Only (192.168.56.0/24)
    |
Empty VM (PXE Client)

Настройки виртуалок:
- Alt Server:
2 ядра, 4 гига оперативы, 40 места на диске, 1. адапрет нат, 2. host-only (192.168.56.1;255.255.255.0)
- PXE Client VM:
2 ядра, 4 гига оперативы, 40 места на диске, 1. адаптер host-only (DHCP;DHCP)

загрузка клиента:
в виртуалбоксе нет варианта грузится с чего-то одного всегда, только выбор на F12, может быть я не нашел, пров�[...]

Тип адаптеров:
Intel PRO/1000 MT Desktop

Настройка ALT Server
смотрим интерфейсы
```bash
ip -c a
```

обычно:

| Интерфейс | Назначение |
| --- | --- |
| enp0s3 | NAT |
| enp0s8 | Host-Only |

Айпишники назначались через GUI Network Manager-а, nat интерфейс не трогался, на enp0s8 адрес был назначен вручную

Были установлен пакеты dnsmasq nfs-server syslinux
```bash
sudo apt-get update && sudo apt-get install dnsmasq nfs-server syslinux
```

Главная проблема обычной рабочей станции - live initrd, который не умеет в PXE/NFS netboot. Из-за этого была поймана ошибка initramfs: waiting for root

Подготовка ISO
Каталоги
```bash
sudo mkdir -p /root/iso
```
```bash
sudo mkdir -p /srv/alt/ws11
```
```bash
sudo mkdir -p /mnt/ws11
```

монтирование iso
```bash
sudo mount -o loop alt-netinstall.iso /mnt/ws11
```

копирование содержимого
```bash
sudo cp -r /mnt/ws11/* /srv/alt/ws11/
```

проверка
```bash
ls /srv/alt/ws11
```

Обязательно должны быть:
- boot
- live
- syslinux
- etc
или похожая installer-структура

настройка nfs
```bash
sudo nano /etc/exports
```

содержимое
```bash
/srv/alt/ws11 *(ro,sync,no_subtress_check,no_root_squash)
```

применение
```bash
sudo exportfs -r
```
```bash
sudo systemctl restart nfs-server && sudo systemctl enable nfs-server
```

проверка
```bash
showmount -e localhost
```
ожидаемый результат:
```bash
/srv/alt/ws11 *
```
Проверка версий NFS
```bash
rpcinfo -p
```
Должны быть: 
- NFSv3
- NFSv4

Настройка PXE/TFTP
Каталоги
```bash
sudo mkdir -p /srv/tftp/pxelinux.cfg
```
```bash
sudo mkdir -p /srv/tftp/alt
```
поиск pxelinux
```bash
sudo find / -name pxelinux.0 2>/dev/null
```
копирование
```bash
cp /user/lib/PXELINUX/pxelinux.0 /srv/tftp/
```

В ALT ldlinux.c32 может отсутствовать. Для минимального PXE достаточно pxelinux.0

копирование installer kernel/initrd

поиск
```bash
sudo find /srv/alt/ws11 | grep -Ei "vmlinuz|initrd"
```
копирование
```bash
sudo cp /srv/alt/ws11/mvlinuz /srv/tftp/alt/
```
```bash
sudo cp /srv/alt/ws11/initrd.img /srv/tftp/alt/
```

проверка
```bash
sudo ls /srv/tftp/alt
```
Должно быть:
- vmlinuz
- initrd.img
****
Настройка dnsmasq

```bash
sudo nano /etc/dnsmasq.conf
```
```ini
port=0

interface=enp0s8
bind-interface

dhcp-range=192.168.56.100,192.168.56.200,12h

dhcp-boot=pxelinux.0

enable-tftp
tftp-root=/srv/tftp
```

Проверка
```bash
sudo dnsmasq --test
```

перезапуск
```sudo
systemctl restart dnsmasq && sudo systemctl enable dnsmasq && sudo systemctl status dnsmasq
```

PXE Config
```bash
sudo nano /srv/tftp/pxelinux.cfg/default
```
```cfg
DEFAULT alt
LABEL alt
    KERNEL alt/vmlinuz
    APPEND initrd=alt/inired.uml
    ip=dhcp
    root@192.168.56.19:/srv/alt/ws11
    ramdisk_size=1500000
```

Права
```bash
sudo chmod -R 755 /srv/tftp
```
```bash
sudo chmod -R 755 /srv/alt


































