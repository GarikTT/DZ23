Занятие 23. Пользователи и группы. Авторизация и аутентификация.
Цели занятия -
	рассмотреть механизмы авторизации и аутентификации;
	объяснить какие бывают права у пользовталей;
	управлять правами с помощью sudo, umask. sgid, suid и более сложными инструментами как PAM и ACL, PolicyKit.
Домашнее задание -
	PAM
	Запретить всем пользователям, кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников.
Начинаем выполнять задание по методичке, пересматривая запись занятия, исправляя некоторые ошибки -

1. vagrant up
2. vagrant ssh
3. sudo -i
4. sudo useradd otusadm && sudo useradd otus
5. echo "Otus2022!" | sudo passwd --stdin otusadm && echo "Otus2022!" | sudo passwd --stdin otus
6. sudo groupadd -f admin
7. usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin
8. ssh otus@192.168.56.10
9. Otus2022!
10. exit
11. ssh otusadm@192.168.56.10
12. Otus2022!
13. exit
14. cat /etc/group | grep admin
15. Создадим файл-скрипт /usr/local/bin/login.sh -
	cat > /usr/local/bin/login.sh
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
16. chmod +x /usr/local/bin/login.sh
17. Сохраняем содержимое файла /etc/pam.d/sshd
	cat /etc/pam.d/sshd
#%PAM-1.0                                                                                                                                                                    
auth       substack     password-auth
auth       include      postlogin
account    required     pam_sepermit.so
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

18. Изменяем содержимое файла /etc/pam.d/sshd
	cat > /etc/pam.d/sshd
#%PAM-1.0
auth       required		pam_sepermit.so
auth       substack     password-auth
auth       include      postlogin
#account    required     dad
#auth required pam-script.so /usr/local/bin/login.sh
auth       optional		pam_reauthorize.so prepare
account    required     pam_exec.so /usr/local/bin/login.sh
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
session    include      password-auth
session    include      postlogin
session    optional		pam_reauthorize.so prepare

19. Устанавливаем дату выходного дня - date 082712302022.00
20. Пытаемся делать входы -
	ssh otus@192.168.56.10
	Otus2022!
	Нет входа !
	ssh otusadm@192.168.56.10
	Otus2022!
	Вход успешно выполнен !
	exit
21. Устанавливаем дату рабочего дня - date 082512302022.00
22. Пытаемся делать входы -
	ssh otus@192.168.56.10
	Otus2022!
	Работает!
	exit
	ssh otusadm@192.168.56.10
	Otus2022!
	Вход успешно выполнен !
	exit
23. Все! Пошаговый результат работы -
	scriptreplay --timing=time_loading_log loading.log -d 30
24. Занятие было понятно. На выполнение ДЗ ушло около 4-х часов.