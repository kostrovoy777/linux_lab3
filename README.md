# linux_lab3
# 1.a Создать нескольких пользователей, задать им пароли, домашние директории и шеллы;
Для создания новых пользователей используем команду sudo useradd ```-d /home/Dirname -s /path/to/shell Username```. И задаем пароль с помощью ```sudo passwd Username```. Результат:
```bash
sudo -d /home/Nikita -s /bin/bash Nikita
sudo -d /home/Evgen -s /bin/bash Evgen
sudo -d /home/Ed -s /bin/bash Ed
sudo passwd Nikita
Введите новый пароль UNIX:
Повторите ввод нового пароля UNIX:
passwd: пароль успешно обновлен
sudo passwd Evgen
Введите новый пароль UNIX:
Повторите ввод нового пароля UNIX:
passwd: пароль успешно обновлен
sudo passwd Ed
Введите новый пароль UNIX:
Повторите ввод нового пароля UNIX:
passwd: пароль успешно обновлен
```
