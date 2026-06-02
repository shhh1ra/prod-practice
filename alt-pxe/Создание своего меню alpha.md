# PXE-сервер ALT Linux — расширенный отчёт

## Первоначальная настройка
- Создание всех нужных каталогов:
```bash
cd /srv/
sudo mkdir -p iso pxe
cd pxe
sudo mkdir -p images && cd images
sudo mkdir -p kworkstation server workstation
cd ../ 
sudo mkdir pxelinux.cfg && cd pxelinux.cfg
sudo touch default
```
- В папку `/srv/iso` необходимо предварительно положить все необходимые образы систем, в моем случае kworkstation-p11, workstation-p11 и server-p11 
- Установка пакета для сетевой развертки альтератора
```bash
sudo apt-get update && sudo apt-get install alterator-netinst
```

После установки alterator-netinst заходим в браузер по адресу localhost:8080 и попадаем в веб версию альтератора. Здесь нужно зайти в меню альтератора используя данные рута (root:пароль) и перейти в меню Сервер сетевой установки (Network installation server), импортировать в него образ альт рабочей станции и после с каталога /var/lib/tftpboot командой cp забрать файлы pxelinux.0 и pxelinux.cfg/default и с каталога boot (cd syslinux/boot) забрать файлы initrd.img и vmlinuz, последние 2 файла уникальны для каждой системы. Повторить все шаги для кажого нужного образа.

## Конфиги и каталоги

### dnsmasq
- Файл: `/etc/dnsmasq.d/pxe-server.conf`
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
****
### PXE каталог
```
/srv/pxe
├── pxelinux.0
├── menu.c32
├── pxelinux.cfg/default
└── images/
    ├── workstation/
    ├── kworkstation/
    └── server/
```
- Получение всех нужных файлов:
 ```bash
 sudo cp /usr/lib/syslinux/menu.c32 /srv/pxe
 sudo cp /var/lib/tftboot/pxelinux.0 /srv/pxe
 ```
 - Получение image файлов (написано на примере импорта kworkstation в веб альтераторе):
```bash
sudo cp /var/lib/tftpboot/boot/initrd.img /srv/pxe/images/kworkstation
sudo cp /var/lib/tftpboot/boot/vmlinuz /srv/pxe/images/kworkstation
```
- Конфиг pxelinux.cfg/default:
```bash
default menu.c32
timeout 100
totaltimeout 100
prompt 0

menu title ALT PXE MultiBoot

label harddisk
	menu label Boot from local disk
	localboot -2
	
label kworkstation
	menu label ALT Kworkstation
	kernel images/kworkstation/vmlinuz
	append initrd=images/kworkstation/initrd.img fastboot root=bootchain bootchain=fg,altboot stagename=live systemd.unit=install2.target ramdisk_size=3584681 showopts vga=normal quiet splash lowmem automatic=method:nfs,server:192.168.56.10,directory:/srv/public/netinst/kworkstation ip=dhcp tz=Europe/Saratov lang=en_US ai curl=http://192.168.56.10/metadata/kworkstation-p11/
	
label workstation
	menu label ALT Workstation
	kernel images/workstation/vmlinuz
	append initrd=images/workstation/initrd.img fastboot root=bootchain bootchain=fg,altboot stagename=live systemd.unit=install2.target ramdisk_size=1685197 showopts vga=normal quiet splash lowmem automatic=method:nfs,server:192.168.56.10,directory:/srv/public/netinst/workstation ip=dhcp tz=Europe/Saratov lang=en_US ai curl=http://192.168.56.10/metadata/workstation-p11/
	
label server
	menu label ALT Server
	kernel images/server/vmlinuz
	append initrd=images/server/initrd.img fastboot root=bootchain bootchain=fg,altboot stagename=live systemd.unit=install2.target ramdisk_size=914433 showopts vga=normal mpath lowmem automatic=method:nfs,server:192.168.56.10,directory:/srv/public/netinst/server ip=dhcp tz=Europe/Saratov lang=en_US ai curl=http://192.168.56.10/metadata/server-p11/
```
****
### NFS
Каталоги, в которых находятся распакованные образы.
```
/srv/public/netinst/
├── workstation/
├── kworkstation/
└── server/
```
- Создание каталога с общим доступом:
```bash
sudo nano /etc/exports
```
- Конфиг exports:
```
/srv/public -ro,insecure_no_subtree_check, fsid=1 *
```
- Применение конфигов
```bash
sudo exportfs -ra
sudo systemctl restart nfs-server
```
- Получение содержимого образов для сетевой загрузки:
```bash
sudo mount -o loop /srv/images /mnt/kworkstation
```
****
### Metadata
- Каталог для написания nginx конфига:
`/etc/nginx/sites-enabled.d/metadata.conf`
- metadata.conf
```bash
server {
	listen 80;
	server_name _;
	
	root /var/www/html;
	
	location / {
		autoindex on;
	}
}
```
- Каталоги для автоответов: 
```
/var/www/html/metadata/
├── workstation-p11/
├── kworkstation-p11/
└── server-p11/
```
- Конфиг autoinstall.scm
```scheme
; Установка русского языка
("/sysconfig-base/language" action "write" lang ("ru_RU"))

; Настройка смены языка
("/sysconfig-base/kbd" language ("ru_RU" action "write" layout "alt_shift_toggle"))

; Настройка часового пояса
("datetime-installer" action "write" commit #t name "RU" zone "Europe/Saratov" utc #t)

; Настройка сети и получение адреса по dhcp
("/net-eth" action "write" reset #t)
("/net-eth" action "write" name "enp0s8" ipv "4" controlled "NetworkManager" configuration "dhcp" default "" search "" dns "" computer_name "newhost" ipv_enabled #t)
("/net-eth" action "write" commit #t)

; Автоматическая разбивка диска
("/evms/control" action "write" control open installer #t)
("/evms/control" action "write" control update)
("/evms/profiles/workstation" action apply commit #f clearall #t exclude ())
("/evms/control" action "write" control commit)
("/evms/control" action "write" control close)

; Установка базовых пакетов ОС
("pkg-install-init" action "write")

; Установка базовой системы
("/pkg-install" action "write" lists "" auto #t)
("/preinstall" action "write")

; Установка GRUB в EFI
;("/grub" action "write" device "efi" passwd #f passwd_1 "*" passwd_2 "*")

; Установка GRUB в BIOS
("/grub" action "write" device "/dev/sda" passwd #f passwd_1 "*" passwd_2 "*")

; Установка пароля рута '123'
("/root/change_password" passwd_2 "123" passwd_1 "123")

; Создание пользователей
("/users/create_account" new_name "user" real_name "Пользователь" allow_su #t auto #f passwd_1 "123" passwd_2 "123" autologin #f)
("/users/create_account" new_name "user2" real_name "Пользователь 2" allow_su #t auto #f passwd_1 "123" passwd_2 "123" autologin #f)

; Донастройка системы при первом запуске после установки
;("/postinstall/firsttime" sctipt "ftp://192.168.0.123/metadata/update.sh")
```
****
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
