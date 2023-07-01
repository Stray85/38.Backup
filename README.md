# Настраиваем бэкапы

Настроить стенд Vagrant с двумя виртуальными машинами: _server_ и _client_

Настроить удаленный бекап каталога `/etc` c сервера client при помощи borgbackup. 

На сервере настроено автоматическое архивирование по таймеру каждые 5 минут.

Заходим на серер и проверяем работу таймера:
```
[root@client ~]# systemctl list-timers --all
NEXT                         LEFT     LAST                         PASSED UNIT                         ACTIVATES
Fri 2023-06-30 23:12:30 UTC  51s left Fri 2023-06-30 23:11:30 UTC  8s ago borg-backup.timer            borg-back
Sat 2023-07-01 23:11:30 UTC  23h left Fri 2023-06-30 23:11:30 UTC  8s ago systemd-tmpfiles-clean.timer systemd-t
n/a                          n/a      n/a                          n/a    systemd-readahead-done.timer systemd-r

3 timers listed.
lines 1-6/6 (END)

```

Проверяем, что демон работает и бэкапы создаются:

```
[root@client ~]# journalctl -S today -f -u borg-backup.service
--
Jul 01 00:03:42 client borg[23017]: Archive name: etc-2023-07-01_00:03:39
Jul 01 00:03:42 client borg[23017]: Archive fingerprint: 860bc8030a16c2e1e28eafe1fc16143a4bb2bd5737e239bec99d74cad71d4fe2
Jul 01 00:03:42 client borg[23017]: Time (start): Sat, 2023-07-01 00:03:41
Jul 01 00:03:42 client borg[23017]: Time (end):   Sat, 2023-07-01 00:03:42
Jul 01 00:03:42 client borg[23017]: Duration: 0.67 seconds
Jul 01 00:03:42 client borg[23017]: Number of files: 1701
Jul 01 00:03:42 client borg[23017]: Utilization of max. archive size: 0%
Jul 01 00:03:42 client borg[23017]: ------------------------------------------------------------------------------
Jul 01 00:03:42 client borg[23017]: Original size      Compressed size    Deduplicated size
Jul 01 00:03:42 client borg[23017]: This archive:               28.44 MB             13.43 MB             24.14 kB
Jul 01 00:03:42 client borg[23017]: All archives:              142.18 MB             67.15 MB             11.82 MB
Jul 01 00:03:42 client borg[23017]: Unique chunks         Total chunks
Jul 01 00:03:42 client borg[23017]: Chunk index:                    1287                 8485
Jul 01 00:03:42 client borg[23017]: ------------------------------------------------------------------------------
Jul 01 00:03:42 client systemd[1]: Started Borg Backup.


```

Проверяем наличие бэкапов:

```
[root@client ~]# borg list borg@192.168.56.21:/var/backup/
"etc-2023-06-30_23:36:31"            Fri, 2023-06-30 23:36:32 [2378343d71536b10b7064e05166b9199f649d12bfbe6d89ce7182b1286b37431]
"etc-2023-06-30_23:42:31"            Fri, 2023-06-30 23:42:32 [c1746c1320fb7d50e8533035b6fc236f2b344871d5566f7812cad9cfd2b78c6a]
"etc-2023-06-30_23:48:31"            Fri, 2023-06-30 23:48:32 [ee8553d625f2e106b55aabfd85b013ca39916f325cbbccb450a4a1dd46a0b6a1]
"etc-2023-06-30_23:54:31"            Fri, 2023-06-30 23:54:32 [bf8fdbd19be64dac1c69ceb895b94a35fa528f921a2e87c1f2a42c43b9c2ecb1]
etc-2023-07-01_00:03:39              Sat, 2023-07-01 00:03:41 [860bc8030a16c2e1e28eafe1fc16143a4bb2bd5737e239bec99d74cad71d4fe2]


```
Пробуем восстановить один из них:

```
[root@client restore]# borg extract borg@192.168.56.21:/var/backup/::etc-2023-07-01_00:03:39
Warning: File system encoding is "ascii", extracting non-ascii filenames will not be supported.
Hint: You likely need to fix your locale setup. E.g. install locales and use: LANG=en_US.UTF-8
[root@client restore]# ls
etc
[root@client restore]#
```