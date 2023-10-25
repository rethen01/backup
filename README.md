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
[root@client tmp]# borg list ssh://borg@192.168.56.160/var/backup</br>
etc-%Y-%m-%d_%H:%M:%S                Wed, 2023-10-25 17:49:15 </br>[104bb8880c68e7d143992db8c2ad9da2d02c23101b371936f9876137ef3eda2e]</br>
client-2023-10-25_18:25:46           Wed, 2023-10-25 18:25:49 </br>[f0c0f29326f4e344811adef317fba30ec01af88024443a3e661ea7769700836c]</br>
client-2023-10-25_18:26:23           Wed, 2023-10-25 18:26:27 </br>[199844381596f73f9724a971433bfb21e02419e4dfb2b070879d409fa669002e]</br>
client-2023-10-25_18:40:40           Wed, 2023-10-25 18:40:43 </br>[4f9d2ead5f8f8df1801d5a257a709a8b995c1b89f3df3e74a91c22990dd8b01c]</br>
client-2023-10-25_18:44:43           Wed, 2023-10-25 18:44:44 </br>[5f830561a408a416d99219f74d0b8b81747f78a4a953cd729e43f1a93ec7e83f]</br>
client-2023-10-25_18:46:37           Wed, 2023-10-25 18:46:37 </br>[279c7b865ab605d50b978888d1734744b6f83e8d9ba3895fdf259b82c0cd532f]</br>
client-2023-10-25_18:48:36           Wed, 2023-10-25 18:48:37 </br>[2cc168f64ac9307d84ab5f18e698d1f6495bad0c65fa6593dd7c12eb08fb8d46]</br>
client-2023-10-25_18:50:36           Wed, 2023-10-25 18:50:37 </br>[cd917587ef447daff17de42d7a7b3539cac191feef4a18d936c27f2581092a83]</br>


<H3>Проверка восстановления из бэкапа</H3>]</br>
[root@client tmp]# mkdir restore && cd restore]</br>
[root@client restore]# borg extract borg@192.168.56.160:/var/backup/::client-2023-10-25_18:54:36]</br>
Вывод dir в файле restore.txt]</br>




