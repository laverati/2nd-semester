# Лабораторная работа №7
#### 
## Задачи:

* Настроить consul
* Поставить postgressql на pg1 и pg2 
* Создать диски sdb и sdc, проинициализировать и примонтировать их    
* На серверах настроить пакет репликации Patroni
* Обеспечить настройку VIP адреса на мастер сервере


## Дано:
* Vagrantfile, который состоит из:
  + pg1, pg2 - диски
  + dcs1, dcs2, dcs3
* Готовая роль consul.play
* inventory
* ansible.cfg
* postgresql.yml

## Шаги:

1. Настроиваием consul.play
```` 
ansible-playbook consul.play
````

2. Создаем диски sdb, sdc:
````
vgcreate pgdata /dev/"dcs"
````

3. На дисках необходимо выполнить следующее:
````
lvcreate -l+100%FREE -n pgdata pgdata 
````
````
lvcreate -l+100%FREE -n pgwal pgwal 
````
Командой __lsblk__ проверяем

<a href="https://imgbb.com/"><img src="https://i.ibb.co/0B85r1p/2022-04-12-13-44-30.png" alt="2022-04-12-13-44-30" border="0"></a>

4. Устанавливаем __postgressql__ на __pg1__ и __pg2__
````
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
````
5. Создаем папки __pg_data__ и __pg_wal__ и присваиваем им роль __postgres.postgres__
````
chown -R postgres.postgres /pgsql/
````
6. Переходим к настройке оркестратора репликации __Patroni__
````
yum install patroni-2.0.2-1.rhel7.x86_64.rpm
````
7. Создаем __postgres.yml__, присваиваем ему роль __postgres__ и переносим туда все данные из файла __postgresql.yml__

8. Создаем папки __/var/log/patroni__ и __/var/log/postgres__, присваиваем им роль __postgres__
9. Запускаем и потом включаем в автозагрузку:
````
systemctl start patroni-watchdog.service 
systemctl enable patroni-watchdog.service 
````
Точно так же запускаем __patroni__ и не забываем включать его в автозагрузку

10. Проверяем всю нашу работу командой:
 ````
 patronictl -c /opt/app/patroni/etc/postgresql.yml list
 ````
 11. Устанавливаем утилиту vip-manager
 ````
 yum install https://github.com/cybertec-postgresql/vip-manager/releases/download/v1.0.2/vip-manager-1.0.2-1.x86_64.rpm
 ````
 12. Делаем небольшие изменения в файлике __vip-manager.yml__ 

 13. Запускаем, включаем в автозагрузку __vip-manager__
 ````
 systemctl start vip-manager
 systemctl enable vip-manager
 ````
 14. Проверяем интерфейс на __pg2__


<a href="https://ibb.co/2gnd300"><img src="https://i.ibb.co/18vTGYY/2022-04-12-14-07-01.png" alt="2022-04-12-14-07-01" border="0"></a>

Пишем в консоль команду __patronictl switchover__, конкретно в моем случае команда немного изменена
<a href="https://ibb.co/8KdJSdd"><img src="https://i.ibb.co/ZLY3kYY/2022-04-12-14-06-31.png" alt="2022-04-12-14-06-31" border="0"></a>

И еще раз проверяем интерфейс

<a href="https://ibb.co/1MshVSC"><img src="https://i.ibb.co/Jqx0LhY/2022-04-12-14-07-35.png" alt="2022-04-12-14-07-35" border="0"></a>

Он успешно переехал! :)



