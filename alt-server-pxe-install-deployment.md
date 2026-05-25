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
в виртуалбоксе нет варианта грузится с чего-то одного всегда, только выбор на F12, может быть я не нашел, проверю

Тип адаптеров:
Intel PRO/1000 MT Desktop

Настройка ALT Server
смотрим интерфейсы
```bash
ip -c a
```

обычно:
| --------- | ---------- |
| Интерфейс | Назначение |
