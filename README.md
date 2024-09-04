Цель домашнего задания
Научиться создавать пользователей и добавлять им ограничения
Описание домашнего задания
1. Запретить всем пользователям кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников

* дать конкретному пользователю права работать с докером и возможность перезапускать докер сервис


1. Подключаемся к нашей созданной ВМ: vagrant ssh
2. Переходим в root-пользователя: sudo -i
3. Создаём пользователя otusadm и otus: sudo useradd otusadm && sudo useradd otus
4. Создаём пользователям пароли: echo "Pass2024!" | sudo passwd --stdin otusadm && echo "Pass2024!" | sudo passwd --stdin otus
Для примера мы указываем одинаковые пароли для пользователя otus и otusadm

5. Создаём группу admin: sudo groupadd -f admin
6. Добавляем пользователей vagrant,root и otusadm в группу admin:
usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin
7. Проверим, что пользователи root, vagrant и otusadm есть в группе admin:
[root@pam ~]# cat /etc/group | grep admin
admin:x:1003:otusadm,root,vagrant
[root@pam ~]# 

8. Создадим файл-скрипт /usr/local/bin/login.sh

vim /usr/local/bin/login.sh

#!/bin/bash
#Первое условие: если день недели суббота или воскресенье
if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun" ]; then
 #Второе условие: входит ли пользователь в группу admin
 if getent group admin | grep -qw "$PAM_USER"; then
        #Если пользователь входит в группу admin, то он может подключиться
        exit 0
      else
        #Иначе ошибка (не сможет подключиться)
        exit 1
    fi
  #Если день не выходной, то подключиться может любой пользователь
  else
    exit 0
fi
9. Добавим права на исполнение файла: chmod +x /usr/local/bin/login.sh
10. Укажем в файле /etc/pam.d/sshd модуль pam_exec и наш скрипт:

vim /etc/pam.d/sshd 


#%PAM-1.0
auth       substack     password-auth
auth       include      postlogin
auth required pam_exec.so debug /usr/local/bin/login.sh
account    required     dad
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    optional     pam_motd.so
session    include      password-auth
session    include      postlogin


Настройка завершена.


