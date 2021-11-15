удаленный доступ к файлам по локальной сети

```
sudo apt-get update
sudo apt-get install samba samba-common-bin
```

Настройка доступа к новой папке

```
sudo mkdir -m 1777 /share
sudo nano /etc/samba/smb.conf
```

```
[share]
Comment = Pi shared folder
Path = /share
Browseable = yes
Writeable = yes
only guest = no
create mask = 0777
directory mask = 0777
Public = yes
Guest ok = yes
```
```
sudo /etc/init.d/samba restart
```


Настройка доступа к существующей папке

```
[share]
Comment = Pi shared folder
Path = /home/pi
Browseable = yes
Writeable = yes
only guest = no
create mask = 0777
directory mask = 0777
Public = yes
Guest ok = yes
```


Создание пароля Samba

В случаях, когда нужно ограничить доступ к ресурсам Raspberry Pi 3, необходимо во-первых, запретить доступ гостям — для этого из файла конфигурации нужно удалить строку Guest ok = yes. Во-вторых, нужно задать пароль Samba:

```
sudo smbpasswd -a pi
 
sudo /etc/init.d/samba restart

```