# ALT-linux
## Таблица
| Имя устройства | Интерфейс | IPv4/IPv6 | Маска/Префикс | Шлюз |
| ------------- | ------------- | ------------- | ------------- | ------------- | 
| ISP | ens 192 | 10.12.12.2 | /24 | 10.12.12.254|
| | ens 224 | 192.168.0.165 | /30 | |
| | ens 256 | 192.168.0.161 | /30 | |
| BR-R | ens 192 | 192.168.0.162 | /30 | 192.168.0.161 |
| | ens 224 | 192.168.0.129 | /27 | |
| HQ-R | ens 192 | 192.168.0.166 | /30 | 192.168.0.165 |
| | ens 224 | 192.168.0.1 | /25 | |
| HQ-SRV | ens 192 | 192.168.0.2 | /25 | 192.168.0.1 |
| BR-SRV | ens 192 | 192.168.0.130 | /27 |192.168.0.129 |
## 1. Настройка интерфейсов
### Изменение настроек интерфейса (замена dhcp на static)
```
mcedit /etc/net/ifaces/(интерфейс)/options
```
### Настройка ip адреса
```
mcedit /etc/net/ifaces/(интерфейс)/ipv4address
```
### Настройка шлюза 
```
mcedit /etc/net/ifaces/(интерфейс)/ipv4route
```
### Создание каталога
```
mkdir /etc/net/ifaces/(интерфейс)
```
### Копирование настроек
```
cp /etc/net/ifaces/(интерфейс)/options /etc/net/ifaces/(интерфейс)/options
```
### Перезагрузка интерфейсов
```
service network restart
```
### Изменение имени
```
hostnamectl set-hostname (имя);exec bash
```
