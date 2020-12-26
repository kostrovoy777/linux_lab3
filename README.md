# linux_lab3
## 1.a Создать нескольких пользователей, задать им пароли, домашние директории и шеллы;
Для создания новых пользователей используем команду sudo useradd `-d /home/Dirname -s /path/to/shell Username`. И задаем пароль с помощью `sudo passwd Username`. Результат:
```bash
$ sudo -d /home/Nikita -s /bin/bash Nikita
$ sudo -d /home/Evgen -s /bin/bash Evgen
$ sudo -d /home/Ed -s /bin/bash Ed
$ sudo passwd Nikita
Введите новый пароль UNIX:
Повторите ввод нового пароля UNIX:
passwd: пароль успешно обновлен
$ sudo passwd Evgen
Введите новый пароль UNIX:
Повторите ввод нового пароля UNIX:
passwd: пароль успешно обновлен
$ sudo passwd Ed
Введите новый пароль UNIX:
Повторите ввод нового пароля UNIX:
passwd: пароль успешно обновлен
```
## 1.б Создать группу admin;
Для создания группы используем простую команду `sudo groupadd admin`

## 1.в. Включить нескольких из ранее созданных пользователей, а также пользователя root, в группу admin;
Добавим пользователей в группу с помощью команды `sudo usermod -aG admin Username`, и проверим добавилось ли с помощью команды `id`
```bash
$ sudo usermod -aG admin Nikita
$ sudo usermod -aG admin Evgen
$ sudo usermod -aG admin root
$ id Nikita
uid=1001(Nikita) gid=1001(Nikita) группы=1001(Nikita),1005(admin)
```
## 1.г. Запретить всем пользователям, кроме группы admin, логин в систему по SSH в выходные дни (суббота и воскресенье, без учета праздников).
Для начала установим PAM `sudo apt install libpam-script`. Далее создадим файл `/usr/share/libpam-script/pam_script_acct`, с таким кодом:
```bash
#!/bin/bash
script="$1"
shift

if groups $PAM_USER | grep admin > /dev/null
then
        exit 0
else
        if [[ $(date +%u) -lt 6 ]]
        then
                exit 0
        else
                exit 1
        fi
fi

if [ ! -e "$script" ]
then
        exit 0
fi
```
Потом сделаем этот файл исполняемым с помощью команды ```sudo chmod +x /usr/share/libpam-script/pam_script_acct```. 
И затем добавляем запись в файл ```/etc/pam.d/sshd```
```bash
account    required     pam_script.so
```

Попробуем зайти в выходной день под пользователем добавленным в группу
```
andry@andry-VirtualBox:~$ ssh Nikita@localhost
Nikita@localhost's password: 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


Могут быть обновлены 35 пакетов.
0 обновлений касаются безопасности системы.

New release '18.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

*** Требуется перезагрузка системы ***

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Sat Dec 26 18:09:26 2020 from 127.0.0.1
Could not chdir to home directory /home/Nikita: No such file or directory
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

Nikita@andry-VirtualBox:/$ 
```
А теперь попробуем под недобавленным
```
andry@andry-VirtualBox:~$ ssh Ed@localhost
Ed@localhost's password: 
Connection to localhost closed by remote host.
Connection to localhost closed.
andry@andry-VirtualBox:~$ 
```

## 2. Установить docker; дать конкретному пользователю права работать с docker
Докер у меня на системе был установлен до этого, так что мне не пришлось его заново. При попытке использования докера пользователем не добавленным в его группу будет отказ. Добавить пользователя в группу докера можно так же как и в группу admin: `sudo usermod -aG docker Nikita`. Проверим работоспособность
```
Nikita@andry-VirtualBox:/$ docker ps -a
CONTAINER ID   IMAGE                                   COMMAND                  CREATED        STATUS                      PORTS     NAMES
4423aafe1932   prom/haproxy-exporter                   "/bin/haproxy_export…"   47 hours ago   Exited (137) 47 hours ago             highload-hw-6_haproxy-exporter_1
1b39672b68ca   haproxy:2.3                             "/docker-entrypoint.…"   47 hours ago   Exited (137) 47 hours ago             highload-hw-6_haproxy_1
914a257ab07d   prom/prometheus                         "/bin/prometheus --c…"   47 hours ago   Exited (137) 47 hours ago             highload-hw-6_prometheus_1
e3d106cc0563   grafana/grafana                         "/run.sh"                47 hours ago   Exited (137) 47 hours ago             highload-hw-6_grafana_1
03cede79fd60   assisken/service-weather                "/entrypoint.sh /sta…"   47 hours ago   Exited (137) 47 hours ago             highload-hw-6_app_1
```
