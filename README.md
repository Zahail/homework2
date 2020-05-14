# homework2
Для тестов и выполения ДЗ был взят Vagrantfile из репозитория https://github.com/erlong15/otus-linux

Были добавлены еще 4 диска. На виртуальной машине были созданы различные рейды с помощью утилиты `mdadm`

Собранные рейды были прописаны в конфигурационном файле `mdadm.conf`

С помощью следующих команд была протестирована поломка и восстановление рейдов в различных вариациях:
`mdadm /dev/md/raid1_example --fail /dev/sdd`

`mdadm /dev/md/raid1_example --remove /dev/sdd`

`mdadm /dev/md/raid1_example --add /dev/sdd`

Также были созданы разделы помощью утилиты `gdisk`

В результате проведенной работы был изменен Vagrantfile. При загрузке будет автоматически собираться **Raid0**,
будут созданы 6 GPT разделов, информация о рейде будет добавлена в конфигурационный файл `mdadm.conf`

Правки:
В MBR для создания 5 партиций сначала нужно создать расширенный раздел (extended partition), а на нем уже 5 логических разделов (партиии). Для этого можно вопользоваться утилитой `fdisk`. Командой `fdisk /dev/sdi` выбираем нужное устройство. Нажав `n` выбираем создание нового раздела, далее `е`(расширенный раздел), номер раздела, начало и размер раздела. Далее нажимаем `n` и выбираем `l` (логический раздел) и создаем необходимое кол-во. Нумерация таких разделов будет начинаться с 5 (sdh5, например). И необходимо выбрать `w` для записи разделов. Результат:

    Disk /dev/sdi: 314 MB, 314572800 bytes, 614400 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk label type: dos
    Disk identifier: 0x260a4d5f

     Device Boot      Start         End      Blocks   Id  System
     /dev/sdi1            2048       43007       20480   83  Linux
     /dev/sdi3          247808      411647       81920    5  Extended
     /dev/sdi5          249856      270335       10240   83  Linux
     /dev/sdi6          272384      313343       20480   83  Linux
     /dev/sdi7          315392      335871       10240   83  Linux
     /dev/sdi8          337920      358399       10240   83  Linux
     /dev/sdi9          360448      380927       10240   83  Linux

Raid:

    [root@otuslinux vagrant]# mdadm --detail /dev/md5
    /dev/md5:
           Version : 1.2
       Creation Time : Thu May 14 20:46:25 2020
        Raid Level : raid5
        Array Size : 915456 (894.00 MiB 937.43 MB)
     Used Dev Size : 305152 (298.00 MiB 312.48 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Thu May 14 20:46:32 2020
             State : clean 
    Active Devices : 4
    Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

    Consistency Policy : resync

              Name : otuslinux:5  (local to host otuslinux)
              UUID : 9d36efd9:d09116d3:f64acd33:43cbcafb
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       64        0      active sync   /dev/sde
       1       8       80        1      active sync   /dev/sdf
       2       8       96        2      active sync   /dev/sdg
       4       8      112        3      active sync   /dev/sdh

       
Ломаем:

    [root@otuslinux vagrant]# mdadm /dev/md5 --fail /dev/sdf
    mdadm: set /dev/sdf faulty in /dev/md5
    [root@otuslinux vagrant]# mdadm --detail /dev/md5
    /dev/md5:
           Version : 1.2
     Creation Time : Thu May 14 20:46:25 2020
        Raid Level : raid5
        Array Size : 915456 (894.00 MiB 937.43 MB)
     Used Dev Size : 305152 (298.00 MiB 312.48 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Thu May 14 20:49:50 2020
             State : clean, degraded 
    Active Devices : 3
     Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

    Consistency Policy : resync

              Name : otuslinux:5  (local to host otuslinux)
              UUID : 9d36efd9:d09116d3:f64acd33:43cbcafb
            Events : 20

    Number   Major   Minor   RaidDevice State
       0       8       64        0      active sync   /dev/sde
       -       0        0        1      removed
       2       8       96        2      active sync   /dev/sdg
       4       8      112        3      active sync   /dev/sdh

       1       8       80        -      faulty   /dev/sdf

Удаляем failty диск:
          
          [root@otuslinux vagrant]# mdadm /dev/md5 --remove /dev/sdf
          mdadm: hot removed /dev/sdf from /dev/md5
          
 Добавляем новый:
 
     [root@otuslinux vagrant]# mdadm /dev/md5 --add /dev/sdi
    mdadm: added /dev/sdi
 
 Проверяем рейд: State : clean
 
        [root@otuslinux vagrant]# mdadm --detail /dev/md5
        /dev/md5:
           Version : 1.2
     Creation Time : Thu May 14 20:46:25 2020
        Raid Level : raid5
        Array Size : 915456 (894.00 MiB 937.43 MB)
     Used Dev Size : 305152 (298.00 MiB 312.48 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Thu May 14 20:56:14 2020
             State : clean 
    Active Devices : 4
     Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

    Consistency Policy : resync

              Name : otuslinux:5  (local to host otuslinux)
              UUID : 9d36efd9:d09116d3:f64acd33:43cbcafb
            Events : 40

    Number   Major   Minor   RaidDevice State
       0       8       64        0      active sync   /dev/sde
       5       8      128        1      active sync   /dev/sdi
       2       8       96        2      active sync   /dev/sdg
       4       8      112        3      active sync   /dev/sdh
       
  root@otuslinux vagrant]# cat /proc/mdstat
  
           Personalities : [raid0] [raid6] [raid5] [raid4] 
           md5 : active raid5 sdi[5] sdh[4] sdg[2] sde[0]
          915456 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/4] [UUUU]

