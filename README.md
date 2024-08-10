### OTUS Linux Professional Lesson #35 | Subject: Мосты, туннели и VPN

#### ЦЕЛЬ: Научится настраивать VPN-сервер в Linux-based системах

#### Описание домашнего задания:
1. Настроить VPN между двумя ВМ в tun/tap режимах, замерить скорость в туннелях, сделать вывод об отличающихся показателях
2. Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на ВМ
3. (*) Самостоятельно изучить и настроить ocserv, подключиться с хоста к ВМ

#### Инструкция по выполнению ДЗ:
1. Настроить VPN между двумя ВМ в tun/tap режимах
   Запускаем 2 виртуальные машины с помощью Vagrant - server и client
   ```
   # vagrant up
   ```
   Устанавливаем необходимые пакеты на двух серверах и отключаем Selinux:
   ```
   # apt update
   # apt install openvpn iperf3 selinux-utils
   # setenforce 0
   ```
   __Настраиваем сервер__
   
   Cоздаем файл-ключ:
   ```
   root@server:~# openvpn --genkey secret /etc/openvpn/static.key
   ```
   Cоздаем конфигурационный файл OpenVPN /etc/openvpn/server.conf со следующим содержимым:
   ```
   dev tap 
   ifconfig 10.10.10.1 255.255.255.0 
   topology subnet 
   secret /etc/openvpn/static.key 
   comp-lzo 
   status /var/log/openvpn-status.log 
   log /var/log/openvpn.log  
   verb 3 
   ```
   Создаем service unit для запуска OpenVPN /etc/systemd/system/openvpn@.service со следующим содержимым:
   ```
   [Unit] 
   Description=OpenVPN Tunneling Application On %I 
   After=network.target 
   [Service] 
   Type=notify 
   PrivateTmp=true 
   ExecStart=/usr/sbin/openvpn --cd /etc/openvpn/ --config %i.conf 
   [Install] 
   WantedBy=multi-user.target
   ```
   Запускаем сервис:
   ```
   root@server:~# systemctl start openvpn@server
   root@server:~# systemctl enable openvpn@server
   ```

   __Настраиваем клиент__

   Cоздаем конфигурационный файл OpenVPN /etc/openvpn/server.conf со следующим содержимым:
   ```
   dev tap 
   remote 192.168.56.10 
   ifconfig 10.10.10.2 255.255.255.0 
   topology subnet 
   route 192.168.56.0 255.255.255.0 
   secret /etc/openvpn/static.key
   comp-lzo
   status /var/log/openvpn-status.log 
   log /var/log/openvpn.log 
   verb 3 
   ```
   Копируем в директорию /etc/openvpn файл-ключ static.key, который был создан на сервере:
   ```
   root@server:~# cp /etc/openvpn/static.key /vagrant/
   root@client:~# cp /vagrant/static.key /etc/openvpn/
   ```
