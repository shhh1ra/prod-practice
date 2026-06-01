## Изучаемый стек:
- **PXE**
- **TFTP**
- **HTTP**
- **DHCP**
- **NFS**
- **Деплой сервисов**
- **Создание своего autoinstall.scm**
****
## Полезные теоретические материалы:
- [Вики альта по работе с bootchain/altboot](https://www.altlinux.org/Installer/common/altboot#bootchain+altboot)
- [Вики альта по работе с autoinstall.scm (11 ветка)](https://www.altlinux.org/Autoinstall#%D0%9F%D1%80%D0%B8%D0%BC%D0%B5%D1%80_Autoinstall.scm_%D0%B4%D0%BB%D1%8F_%D0%B4%D0%B8%D1%81%D1%82%D1%80%D0%B8%D0%B1%D1%83%D1%82%D0%B8%D0%B2%D0%BE%D0%B2_11%D0%BE%D0%B9_%D0%B2%D0%B5%D1%82%D0%BA%D0%B8)
****
## Используемый стек:
- Alterator-web
>	Автоматическая подготовка требуемых initrd.img и vmlinuz
- Dnsmasq
>	Автоматическая выдача IP-адресов при загрузке через PXE
- TFTP-сервер