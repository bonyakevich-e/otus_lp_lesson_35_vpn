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
   root@client:~# systemctl start openvpn@server
   root@client:~# systemctl enable openvpn@server
   ```
   Замеряем скорость в туннеле. На сервере запускаем:
   ```
   root@server:~# iperf3 -s
   ```
   На клиенте:
   ```
   root@client:~# iperf3 -c 10.10.10.1 -t 40 -i 5
   ...
   ...
   [  5]   0.00-25.00  sec  1.02 GBytes   350 Mbits/sec  755             sender
   [  5]   0.00-25.00  sec  0.00 Bytes  0.00 bits/sec                  receiver
   ```
   И в обратную сторону:
   ```
   root@client:~# iperf3 -c 10.10.10.1 -t 40 -i 5 -R
   ...
   ...
   [  5]   0.00-15.59  sec  0.00 Bytes  0.00 bits/sec                  sender
   [  5]   0.00-15.59  sec   549 MBytes   295 Mbits/sec                  receiver
   ```

   Меняем в конфигурационных файлах режим работы с tap на tun.  Замеряем скорость соединения:
   ```
   root@client:~# iperf3 -c 10.10.10.1 -t 40 -i 5
   ...
   ...
   [  5]   0.00-40.00  sec  1.54 GBytes   332 Mbits/sec  859             sender
   [  5]   0.00-40.06  sec  1.54 GBytes   331 Mbits/sec                  receiver
   ...
   ...
   ```
   И в обратную сторону:
   ```
   root@client:~# iperf3 -c 10.10.10.1 -t 40 -i 5 -R
   ...
   ...
   [  5]   0.00-40.04  sec  1.43 GBytes   308 Mbits/sec  234             sender
   [  5]   0.00-40.00  sec  1.43 GBytes   307 Mbits/sec                  receiver
   ...
   ...
   ```
   __Выводы:__ ...
