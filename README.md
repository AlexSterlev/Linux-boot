# Выполнено Работа 5


 ### Подключение к консоли без ввода пароля. Способ первый.

 Общий момент всех трех способов - как зайти в редактор параметров загрузки.

 Для этого достаточно нажать - e - когда на экране выбор ядра системы.

 Далее, в данном случае, мы можем добавить вызов shell-интерпретатора в процессе загрузки: В конце строки начинающейся с linux16 указываем init=/bin/sh.

 После чего нажимаем ctrl-x, после чего оказываемся в консоли.

 Однако, рутовая файловая система находится в режиме Read-Only. Если мы ходим что-то писать в нее, то нужно ее перемонтировать

 ```bash
 $ mount -o remount,rw /
 ```

 Теперь файловая система доступна для записи.


 ### Подключение к консоли без ввода пароля. Способ второй.

 Аналогичным образом попадаем в редактов загрузки. Но на этот раз используем возможность прервать загрузку.

 Для этого в конце строки начинающейся с linux16 добавляем rd.break.

 После нажатия ctrl-x мы оказываемся в режиме Emergency mode.

 Опять же рутовая файловая система находится в режиме RO, плюс - мы еще и не в ней.

 Исправляем это при помощи следующих команд:

 ```bash
 $ mount -o remount,rw /sysroot
 $ chroot /sysroot
 ```

 Ок, теперь мы можем - сменить пароль рута.

 ```bash
 $ passwd root
 ```

 Прежде чем перезагрузиться, не забываем вызвать команду корректировки метки SELinux у файла с паролями (/etc/shadow)

 ```bash
 $ touch /.autorelabel
 ```

 ### Подключение к консоли без ввода пароля. Способ третий.

 С входом в редактор все так же.

 Но на этот раз мы сделаем так, чтобы входить с файловой системой в режиме записи.

 Для этого, в уже известной строчке, заменим ro на rw init=/sysroot/bin/sh.


 ### Переименовываем рутовый VG

 Если мы ходим изменить имя Volume Group, то воспользуемся командой:

 ```bash
 $ vgrename VolGroup00 OtusRoot
 ```

 Так как мы изменили имя VG, которая испрользуется при загрузке, то нам необходимо внести соответствующие правки в файлы /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. В них нам нужно заменить старое название VG на новое.

 Дополнительно к этому пересоздаем initrd image:

 ```bash
 mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
 ```

 Теперь можно попробовать перезагрузиться и обнаружить, что мы успешно вошли в систему, при чем с новым именем VG.

 ```bash
 [root@PL04 ~]# vgs
   VG       #PV #LV #SN Attr   VSize   VFree
   Test       1   2   0 wz--n- <31,00g 4,00m
 ```

 ### Добавляем модуль в initrd.

 Добавим в процесс загрузки модуль, который выводит псевдографикой прикольного пингвина.

 Создадим каталог для размещения скриптов модуля:

 ```bash
 $ mkdir /usr/lib/dracut/modules.d/01test
 ```

 Размещаем там два скрипта:

 module-setup.sh

 ```
 #!/bin/bash

 check() {
     return 0
 }

 depends() {
     return 0
 }

 install() {
     inst_hook cleanup 00 "${moddir}/test.sh"
 }
 ```

 и test.sh

 ```
 #!/bin/bash

 exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
 cat <<'msgend'
 Hello! You are in dracut module!
  ___________________
 < I'm dracut module >
  -------------------
    \
     \
         .--.
        |o_o |
        |:_/ |
       //   \ \
      (|     | )
     /'\_   _/`\
     \___)=(___/
 msgend
 sleep 10
 echo " continuing...."   
 ```

 Пересобирем образ initrd

 ```bash
 $ mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
 ```

 Чтобы при загрузке работал вывод, уберем некоторые параметры из описания загрузки.

 Отредактируем файл /boot/grub2/grub.cfg - уберем параметры rhgb и quiet.

 ```bash
 $ vi /boot/grub2/grub.cfg
 ``` 



