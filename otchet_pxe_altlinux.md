# Отчёт по выполненной работе

Развёртывание сервера централизованной сетевой установки ALT Linux с использованием PXE.

## Краткий итог

- Настроен dnsmasq (DHCP, PXE, TFTP)
- Организован каталог /srv/pxe с ядрами и initrd
- Реализовано PXE-меню для ALT Workstation, ALT KWorkstation и ALT Server
- Настроен nginx для публикации metadata и autoinstall.scm
- Настроен NFS для передачи второго этапа установки
- Исследован bootchain и механизм выбора каталога установки
- Реализован выбор различных дистрибутивов через параметр directory в automatic=method:nfs
- Устранены ошибки 'Second stage file not found: live' и проблемы с autoinstall
- Получен рабочий сервер централизованного развёртывания ALT Linux
