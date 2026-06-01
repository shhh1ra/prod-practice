# PXE-сервер ALT Linux — расширенный отчёт

## Конфиги и каталоги

### dnsmasq
- Файл: /etc/dnsmasq.d/pxe-server.conf
```bash
interface=enp0s8
bind-interfaces

# DCHP
dhcp-range=192.168.56.100,192.168.56.200,12h
dhcp-boot=pxelinux.0

#TFTP
enable-tftp
tftp-root=/srv/pxe

# LOGS
log-dhcp
```
- Проверка конфигурации:
```bash
dnsmasq --test
systemctl restart dnsmasq
journalctl -u dnsmasq -f
```
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
- Каталоги для автоответов: 
/var/www/html/metadata/
├── workstation-p11/
├── kworkstation-p11/
└── server-p11/

### NFS
Каталоги, в которых находятся распакованные образы.
/srv/public/netinst/
├── workstation/
├── kworkstation/
└── server/



## Ключевые команды

exportfs -ra
showmount -e localhost

cat /proc/cmdline


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
