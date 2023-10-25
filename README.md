# Backup
Описание:
- Vagrantfile: для бэкап сервера добавлен диск 2 Гб + скрипт по настройке доступа по ключа с хостовой машины.
- Playbook: устанавливается epel-release, borg, добавляется пользователь borg, монтируется диск в /var/backup
Остальные настройки не получилось выполнить через ansible;( Выполняются вручную:
<H3>На клиенте настраивается доступ по ключу для пользователя borg до сервера</H3>
<p>ssh-keygen
<p>ssh-copy-id borg@192.168.56.160
<H3>Настройка borg на client</H3>
[root@client vagrant]# borg init --encryption=repokey borg@192.168.56.160:/var/backup/
<p>После выполнения команды по инициализации репозитория:</br>
[root@server backup]# ls -l</br>
total 68</br>
-rw-------. 1 borg borg    73 Oct 25 16:35 README</br>
-rw-------. 1 borg borg   700 Oct 25 16:36 config</br>
drwx------. 3 borg borg  4096 Oct 25 16:36 data</br>
-rw-------. 1 borg borg    52 Oct 25 16:36 hints.1</br>
-rw-------. 1 borg borg 41258 Oct 25 16:36 index.1</br>
-rw-------. 1 borg borg   190 Oct 25 16:36 integrity.1</br>
-rw-------. 1 borg borg    16 Oct 25 16:36 nonce</br>
<H3>Проверка создания бэкапа</H3>
[root@client vagrant]# borg create --stats --list borg@192.168.56.160:/var/backup/::"etc-{now:%Y-%m-%d_%H:%M:%S}" /etc
<p>Результат команды:
Archive name: etc-2023-10-25_16:39:46</br>
Archive fingerprint: 795e69a0067477958cfacf03d538c3e7a447605a90a0b46caaba39c0cd85ab17 </br> 
Time (start): Wed, 2023-10-25 16:39:54  </br>
Time (end):   Wed, 2023-10-25 16:39:58 </br> 
Duration: 3.92 seconds  </br>
Number of files: 1710  </br>
Utilization of max. archive size: 0%  </br>
------------------------------------------------------------------------------  </br>
                       Original size      Compressed size    Deduplicated size  </br>
This archive:               28.45 MB             13.50 MB             11.85 MB  </br>
All archives:               28.45 MB             13.50 MB             11.85 MB  </br>
  </br>
                       Unique chunks         Total chunks  </br>
Chunk index:                    1293                 1710  </br>
------------------------------------------------------------------------------  </br>
</br>
[root@client vagrant]# borg list borg@192.168.56.160:/var/backup/</br>
borg@192.168.56.160's password: </br>
Enter passphrase for key ssh://borg@192.168.56.160/var/backup: </br>
etc-2023-10-25_16:39:46              Wed, 2023-10-25 16:39:54 </br>[795e69a0067477958cfacf03d538c3e7a447605a90a0b46caaba39c0cd85ab17]</br>

<h3>Проверка списка файлов для бэкапа в файле list_files.txt</h3></br>
[root@client vagrant]# borg list borg@192.168.56.160:/var/backup/::etc-2023-10-25_16:39:46 > list_files.txt
<h3>Достаем файл /etc/hostname из бекапа</h3></br>
[root@client vagrant]# borg extract borg@192.168.56.160:/var/backup/::etc-2023-10-25_16:39:46 etc/hostname</br>
borg@192.168.56.160's password:</br> 
Enter passphrase for key ssh://borg@192.168.56.160/var/backup: </br>
[root@client vagrant]# ls -l</br>
total 248</br>
drwx------. 2 root root     22 окт 25 16:52 etc</br>
-rw-r--r--. 1 root root 251814 окт 25 16:46 list_files.txt</br>
[root@client vagrant]# cat /etc/hostname </br>
client</br>
<h3>Автоматизируем создание бэкапов с помощью systemd. Создаем сервис и таймер в каталоге /etc/systemd/system/
</h3></br>
borg-backup.sh - скрипт по созданию бэкапа</br>
borg-backup.service - сервис в котором вызывается скрипт borg-backup.sh</br>
borg-backup.timer - таймер в котором задано время выполнения службы borg-backup.service</br>
</br>
[root@client vagrant]# systemctl list-timers --all</br>
NEXT                          LEFT    LAST                          PASSED  UNIT                         ACTIVATES</br>
Ср 2023-10-25 18:17:36 UTC  14s ago Ср 2023-10-25 18:17:50 UTC  4ms ago borg-backup.timer            borg-backup.service</br>




