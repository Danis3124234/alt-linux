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
### Изменение имени
```
hostnamectl set-hostname (имя);exec bash
```
### Создание каталога
```
mkdir /etc/net/ifaces/(интерфейс)
```
### Копирование настроек
```
cp /etc/net/ifaces/(интерфейс)/options /etc/net/ifaces/(интерфейс)/options
```
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
### Перезагрузка интерфейсов
```
service network restart
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
sed -i "s/NM_CONTROLLED=no/NM_CONTROLLED=yes/g" /etc/net/ifaces/ens160/options
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
### ![](https://github.com/Danis3124234/alt-linux/blob/main/brr%20frr.PNG)
### ![](https://github.com/Danis3124234/alt-linux/blob/main/hqr%20frr.PNG)
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
* проверка пинг с HQ-SRV на BR-SRV
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
## 6. NAT с помощью firewalld. (ISP, HQ-R, BR-R)
### Установка
```
apt-get -y install firewalld
```
### Автозагрузка
```
systemctl enable --now firewalld
```
### Правило к исходящим пакетам (тот интерфейс который смотрит во внеш. сеть например на исп 192)
```
firewall-cmd --permanent --zone=public --add-interface=ens__
```
### Правило к входящим пакетам (тот интерфейс который смотрит во внутрен. сеть например на исп 224 и 256)
```
firewall-cmd --permanent --zone=trusted --add-interface=ens__
```
### Включение NAT
```
firewall-cmd --permanent --zone=public --add-masquerade
```
### Cохранение правил
```
firewall-cmd --reload
```
## 7. DHCP
### Установка DHCP
```
apt-get install -y dhcp-server
```
### Вошёл в файл
```
mcedit /etc/sysconfig/dhcpd 
```
### Указал интерфейс который смотрит на SRV
```
DHCPDARGS=ens224
```
### Далее следует настройка раздачи адресов, а для этого захожу в файл
```
mcedit /etc/dhcp/dhcpd.conf
```
### Прописываю конфиг
```
# dhcp.conf

default-lease-time 6000;
max-lease-time 72000;

subnet 192.168.0.0 netmask 255.255.255.128 {
range 192.168.0.10 192.168.0.125;
option routers 192.168.0.1;
}
```
### Повторяю конфиг в файле /etc/dhcp/dhcpd.conf.example Запускаю и добавляю в автозагрузку слжубу
```
systemctl enable --now dhcpd
```
## 8. Измерение пропускной способности сети между двумя узлами
### Установка пакета
```
apt-get install -y iperf3
```
### Запуск службы
```
systemctl enable --now iperf3
```
### Запуск iperf3 в качестве клиента
```
iperf3 -c 192.168.0.162
```
## 9. Составление backup скриптов для сохранения конфигурации сетевых устройств
### Создание скрипта
```
nano backup-script.sh
```
### Содержимое файла
```
#!/bin/bash

echo "Start backup!"

backup_dir="/etc"
dest_dir="/opt/backup"

mkdir -p $dest_dir
tar -p $dest_dir/$(hostname -s)-$(date +"%d.%m.%y").tgz $backup_dir

echo "Done!"
```
### Назначение права на исполнение для данного файла
```
chmod +x backup-script.sh
```
### Запуск скрипта
```
./backup-script.sh
```
### Просмотр содержания архива
```
tar -tf /opt/backup/hq-r-00.00.00.tgz | less
```
## 10. Настройка подключения по SSH для удаленного конфигурирования устройства
### На HQ-SRV меняем порт с 22 на 2222
```
sed -i "s/#Port 22/Port 2222/g" /etc/openssh/sshd_config
```
### Перезагружаем службу sshd
```
systemctl restart sshd
```
### Проверка
```
ss -tlpn | grep sshd
```
### На HQ-R устанавливаем nftables
```
apt-get install -y nftables
```
### Включаем и добавляем в автозагрузку службу nftables
```
systemctl enable --now nftables
```
### Создаём правило
```
nft add table inet nat
```
### Добавляем цепочку в таблицу
```
nft add chain inet nat prerouting '{ type nat hook prerouting priority 0; }'
```
### Добавляем ещё одно правило
```
nft add rule inet nat prerouting ip daddr 192.168.0.161 tcp dport 22 dnat to 192.168.0.10:2222
```
### Сохраняем правила
```
nft list ruleset | tail -n 7 | tee -a /etc/nftables/nftables.nft
```
### Перезапускаем службу
```
systemctl restart nftables
```
### Проверка на BR-R
```
ssh admin@(адрес смотрящий в ISP)
password
hostname
```
## 11. Настройка контроль доступа до HQ-SRV по SSH
### Установка nftables
```
apt-get update && apt-get install -y nftables
```
### Включаю и добавляю в автозагрузку
```
systemctl enable --now nftables
```
### Добавляю правило
```
nft add rule inet filter input ip saddr 192.168.0.170 tcp dport 2222 counter drop
nft add rule inet filter input ip saddr 192.168.0.0/30 tcp dport 2222 counter drop
```
### Проверка
```
nft list ruleset
```
### В файле /etc/nftables/nftables.nft удаляю незакомментированные строчки Отправляю результат
```
nft list ruleset | tee -a /etc/nftables/nftables.nft
```
### Перезапускаю
```
systemctl restart nftables
```
### Проверка заключается в том, что подключиться не получится и будет выдано сообщение: "connect to host 192.168.0.170 port 2222: Connection timed out
```
