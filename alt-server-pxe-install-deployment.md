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
