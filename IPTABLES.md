iptables
```
sudo aptitude install iptables-persistent
```

Просмотр текущей конфигурации:```
iptables -L```
Ответ команды будет следующим:
```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination```
Эти правила разрешают все всем отовсюду.

Хранение правил IPTables в файле
Для начала создадим новый файл для правил:
```
nano /etc/iptables.rules```
В этот файл внесем несколько простых правил:
```
*filter

# Разрешить весь локальный (lo0) трафик и отбрасывать весь трафик на сеть 127/8, который не идет через lo0
-A INPUT -i lo -j ACCEPT
-A INPUT ! -i lo -d 127.0.0.0/8 -j REJECT

# Разрешить все установленные изнутри подключения
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Разрешить весь исходящий трафик
-A OUTPUT -j ACCEPT

# Разрешить HTTP и HTTPS подключения отовсюду на сервер
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT

# Разрешить подключения по SSH
# Номер порта --dport такой же как в /etc/ssh/sshd_config
-A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT

# Разрешить пинг
#  заметьте, что блокирование других типов icmp трафика нежелательно
#  удалите -m icmp --icmp-type 8 отсюда чтобы разрешить все виды icmp:
#  https://security.stackexchange.com/questions/22711
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT

# логировать неудачные попытки доступа (доступен через команду 'dmesg')
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

# Отбрасывать все входящие пакеты. по умолчанию все разрешается, что не хорошо:
-A INPUT -j REJECT
-A FORWARD -j REJECT

COMMIT```
Активировать эти правила:
```
iptables-restore < /etc/iptables.rules```
И посмотрите изменения:
```
iptables -L```
Теперь вывод команды говорит нам о том, что только те порты что разрешили открыты, остальные закрыты.

Теперь, когда все верно, можно сохранить эти правила для восстановления их при перезагрузке:
```
iptables-save > /etc/iptables.rules```
В файл:
```
nano /etc/network/if-pre-up.d/iptables```
Добавьте следующие строки:
```
#!/bin/sh
/sbin/iptables-restore < /etc/iptables.rules```
Файл должен быть исполняемым, поэтому выполняем команду:
```
chmod +x /etc/network/if-pre-up.d/iptables```


### Error
Bash script and /bin/bash^M: bad interpreter: No such file or directory
```
sed -i -e 's/\r$//' iptables.sh```