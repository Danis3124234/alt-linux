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
### Включение маршрутизации (Изменяем net.ipv4.ip_forward = 0 на net.ipv4.ip_forward = 1)
```
mcedit /etc/net/sysctl.conf
```
## 2. Установка nmtui
### Обновляем список пакетов и устанавливаем необходимые пакеты
```
apt-get update && apt-get install -y NetworkManager-{daemon,tui}
```
### Запускаем и добавляем в автозагрузку службу "NetworkManager"
```
systemctl enable --now NetworkManager
```
### Для того, чтобы интерфейсы стали видимы в nmtui или nmcli - необходимо поправить параметр в файле /etc/net/ifaces/<ИМЯ_ИНТЕРФЕЙСА/options
```
sed -i "s/NM_CONTROLLED=no/NM_CONTROLLED=yes/g" /etc/net/ifaces/ens33/options
```
### Перезапускаем службы "network" и "NetworkManager"
```
systemctl restart network
```
```
systemctl restart NetworkManager
```
### Проверяем
```
nmcli connection show
```
```
nmtui
```
## 3. Настройка тунеля
### Заходим в интерфейс
```
nmtui
```
![](https://github.com/Danis3124234/Demo2024/blob/main/1.png)
![](https://github.com/Danis3124234/Demo2024/blob/main/2.png)
```
Родительский 192.168.0.166
Локальный 192.168.0.162
Маска /30
```
![](https://github.com/Danis3124234/Demo2024/blob/main/3.png)
```
Родительский 192.168.0.162
Локальный 192.168.0.166
Маска /30
```
### Для BR-R
```
nmcli connection modify BR-R ip-tunnel.ttl 64
```
```
ip r add 192.168.0.0/25 dev gre1
```
### Для HQ-R
```
nmcli connection modify HQ-R ip-tunnel.ttl 64
```
```
ip r add 192.168.0.128/27 dev gre1
```
## 4. Настройка динамической (внутренней) маршрутизации средствами frr (HQ-R, BR-R)
### Установим пакет frr
```
apt-get update && apt-get install -y frr
```
### В конфигурационном файле "/etc/frr/daemons" необходимо активировать выбранный протокол для дальнейшей реализации его настройки
```
vim /etc/frr/daemons
```
### ![](https://github.com/Danis3124234/alt-linux/blob/main/1.JPG)
### Включаем и добавляем в автозагрузку службу frr
```
systemctl enable --now frr
```
### ![](https://github.com/Danis3124234/alt-linux/blob/main/2.JPG)
```
где:
- configure terminal - переход в режим глобальной конфигурации
- router ospf - переход в режим конфигурации OSPFv2
- passive-interface default - перевод всех интерфейсов в пассивный режим:
- далее туннельный интерфейс "tun1" будет сделать активным, для того чтобы устанавливать соседство с BR-R и обмениваться внутренними маршрутами
- network - объявляем локальную сеть офиса HQ и туннельную сеть
- после чего переводим интерфейс tun1 в активный режим
- сохраняем текущую конфигурацию
```
## 5. Настройка локальных учетных записей.
### Добавление пользователя
```
useradd admin -m -c "Admin" -U 
```
### Поменять пароль
```
passwd admin
```
### Просмотр пользователей
```
vim /etc/passwd
```
###
```

```
###
```

```
###
```

```
###
```

```
###
```

```
###
```

```
###
```

```
###
```

```
###
```

```
