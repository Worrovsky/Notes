# Linux Foundation Certified System Administrator (LFCS)

## Lesson 8 Сети

#### Команда ip (runtime конфигурация)

* `ip link show`
* `ip address show`
* установка адреса `ip address add dev <имя интерфейса напр. ens33> 10.0.0.0/24`
* следует исользовать `ip`, а не  `ifconfig`
* `ip route show`

#### Biosdevname

* Сетевым интерфейсам даются имена на основании физ. расположения:
    - em123 (Ethernet Motherboard portnum)
    - p<port> p<slot> (PCI)
    - eno123 (Ethernet onboard)
    - если драйвер устройства не предоставляет инфо, eth0, eth1, ...

#### Настройка сети CentOS (постоянная)

* графическая утилита `nwtui`
* консольная `nmcli`
* `man nmcli-examples`
* для удобства использовать дополнение командной строки (`rmp -qa | grep bash-completion`). По частям набираем через tab x2
* конфигурации хранятся в `/etc/sysconfig/network-scripts/`
* при изменении выполнить `nmcli .. .. up` или перезагрузиться

#### Hostname

* содержится в `/etc/hostname`
* просмотр `uname -a`
* ядро работает с `/proc/sys/kernel/hostname`, поэтому или перезагрузка после редактирования `/etc/hostname` или использование команд типа `hostnamectl`

#### Разрешение доменных имен

* `/etc/resolv.conf`
    - содержит настройки серверов dns
    - заполняется менеджерами настроек типа `nmcli`
    - можно вручную редактировать, но при перезагрузке не сохраняется
* `/etc/hosts` - ручные разрешения
* `/etc/nsswitch.conf`
    - указан порядок поиска: файл hosts, потом сервера dns 

#### Вспомогательные утилиты

* `ping`
* `dig` - проверка работы dns, показывает кем, как было разрешено доменное имя
* `nmap` -  сканер портов (требуется доп. установка)
    - аккуратно применять вне своей сети (м.б. расценено как атака)
