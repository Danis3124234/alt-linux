# ALT-linux
## 1. Настройка интерфейсов
```
mcedit /etc/net/ifaces/(интерфейс)/options
```
```
mcedit /etc/net/ifaces/(интерфейс)/ipv4address
```
```
mcedit /etc/net/ifaces/(интерфейс)/ipv4route
```
```
mkdir /etc/net/ifaces/(интерфейс)
```
```
cp /etc/net/ifaces/(интерфейс)/options /etc/net/ifaces/(интерфейс)/options
```
```
service network restart
```
```
hostnamectl set-hostname (имя);exec bash
```
