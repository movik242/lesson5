# lesson5
При выполнении задания использовал образ centos/7 - v. 2004.01

**Домашнее задание**

-vagrant up должен поднимать 2 виртуалки: сервер и клиент;
 
-на сервер должна быть расшарена директория;
 
-на клиента она должна автоматически монтироваться при старте (fstab или autofs);
 
-в шаре должна быть папка upload с правами на запись;
 
-требования для NFS: NFSv3 по UDP, включенный firewall.
 

1) *Настраиваю сервер*
 
	    yum install nfs-utils -y
	    systemctl enable firewalld
	    systemctl status firewalld #проверяю работу сервиса
	    firewall-cmd --add-service="nfs3" \
	    --add-service="rpc-bind" \
	    --add-service="mountd" \
	    --permanent 
	    firewall-cmd --reload
   
Запускаю сервер NFS

    systemctl enable nfs --now
    

![image](https://github.com/movik242/lesson5/assets/143793993/394bc99b-d680-410c-abc9-4c0d740a510a)

Настраиваю директорию upload

		mkdir -p /srv/share/upload 
		chown -R nfsnobody:nfsnobody /srv/share 
		chmod 0777 /srv/share/upload 

Создаю структуру в /etc/exports

		cat << EOF > /etc/exports 
		/srv/share 10.0.2.24/32(rw,sync,root_squash)
		EOF

Экспортирую и проверяю

![image](https://github.com/movik242/lesson5/assets/143793993/6c54a920-58c7-43bd-9f5d-52ee933d5a7c)


2) *Настраиваю клиент*

	Выполняю теже действия, что и при настройке сервера

		yum install nfs-utils -y
	    systemctl enable firewalld
	    systemctl status firewalld #проверяю работу сервиса
	    firewall-cmd --add-service="nfs3" \
	    --add-service="rpc-bind" \
	    --add-service="mountd" \
	    --permanent 
	    firewall-cmd --reload


	 ![image](https://github.com/movik242/lesson5/assets/143793993/e1fc2027-6f7d-4838-bb86-0b2ef3d92062)


Вношу изменения в fstab  для автомонтирования 

		echo "10.0.2.23:/srv/share/ /mnt nfs vers=3,proto=udp,noauto,x-systemd.automount 0 0" >> /etc/fstab
		systemctl daemon-reload 
		systemctl restart remote-fs.target



![image](https://github.com/movik242/lesson5/assets/143793993/4eab1564-d053-4ed0-a1e8-f46aa9f021d4)


Далее провожу ряд тестов для проверки работы.

Создаю файл на стороне сервера

			touch /srv/share/upload/check_file

Проверяю  на стороне клиента

		ls - l /mnt/upload

 И наоборот 
 		touch /mnt/upload/client_file

		ls -l /srv/share/upload

Перезагружаю клиента и проверяю, что папка примонтирована

В приложенных файлах vagrantfile, который поднимает две ВМ, скрипты выполнения ДЗ как на стороне сервера, так и клиента.









