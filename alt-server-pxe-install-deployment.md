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
cp -r /mnt/ws11/* /srv/alt/ws11/














































