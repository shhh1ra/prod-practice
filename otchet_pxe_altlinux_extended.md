# PXE-сервер ALT Linux — расширенный отчёт

## Конфиги и каталоги

### dnsmasq
Файл: /etc/dnsmasq.d/pxe-server.conf

interface=enp0s8
bind-interfaces
dhcp-range=192.168.56.100,192.168.56.200,12h
dhcp-boot=pxelinux.0
enable-tftp
tftp-root=/srv/pxe
log-dhcp

### PXE каталог

/srv/pxe
├── pxelinux.0
├── menu.c32
├── pxelinux.cfg/default
└── images/
    ├── workstation/
    ├── kworkstation/
    └── server/

### Metadata

/var/www/html/metadata/
├── workstation-p11/
├── kworkstation-p11/
└── server-p11/

### NFS

/srv/public/netinst/
├── workstation/
├── kworkstation/
└── server/

В каталогах находится распакованное содержимое образов.

## Ключевые команды

dnsmasq --test
systemctl restart dnsmasq
journalctl -u dnsmasq -f

exportfs -ra
showmount -e localhost

cat /proc/cmdline
ls -la /tmp/metadata

## Важные находки

1. bootchain по умолчанию использует:
   /srv/public/netinst/current

2. Для выбора образа работает:
   automatic=method:nfs,server:192.168.56.10,directory:/srv/public/netinst/kworkstation

3. Metadata должна указываться каталогом:
   ai curl=http://192.168.56.10/metadata/kworkstation-p11/

4. Указание файла autoinstall.scm напрямую приводило к ошибкам.

## Результат

Реализована автоматическая установка ALT Workstation, ALT KWorkstation и ALT Server через единый PXE-сервер.
