```
1. Попасть в систему без пароля несколькими способами.
2. Установить систему с LVM, после чего переименовать VG.
3. Добавить модуль в initrd.
```

1. С помощью Вагрант развернул Centos в VirtualBox, в меню выбора ядра нажал E для редактирования.

```
Способ загрузки 1.
В конце строки, начинающейся со слова linux, добавил init=/bin/sh
сtrl-x - загрузка ОС
Система сообщает о загрузке в аварийном режиме и предлагает поработать в консоли.
Монтирую систему в режим записи
mount -o remount,rw /
далее меняю пароль
chroot /sysroot
passwd root
touch /.autorelabel
```

```
Способ загрузки 2.
В конце строки, начинающейся со слова linux, добавил rd.break
сtrl-x - загрузка ОС
Система сообщает о загрузке в аварийном режиме и предлагает поработать в консоли.
Монтирую систему в режим записи
mount -o remount,rw /sysroot
далее меняю пароль
chroot /sysroot
passwd root
touch /.autorelabel
После перезапуска, произойдёт переконфигурирование системы, ещё одна автоматическая перезагрузка, после чего можно загружаться с помощью нового пароля
```

```
Способ загрузки 3.
В конце строки, начинающейся со слова linux, заменяю rо на rw init=/sysroot/bin/sh
сtrl-x - загрузка ОС
Система сообщает о загрузке в аварийном режиме и предлагает поработать в консоли.
Систему уже смонтирована в режим записи.
Повторяем процедуру из предыдущих способов для смены пароля с помощью passwd
```

2. С помощью Вагрант развернул Centos с LVM в VirtualBox.
Переименовываю Volume Group
```
[root@lvm ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0 

[root@lvm ~]# vgrename VolGroup00 OtusRoot
  Volume group "VolGroup00" successfully renamed to "OtusRoot"

```

```
Чтобы корректно загрузить ОС меняю название Volume Group в файлах /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg через VI.
Дальше пересоздаётся initrd image
[root@lvm ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
Executing: /sbin/dracut -f -v /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64
.....
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
```
Проверяю
```
[root@lvm ~]# vgs
  VG       #PV #LV #SN Attr   VSize   VFree
  OtusRoot   1   2   0 wz--n- <38.97g    0 
```

3. В каталоге /usr/lib/dracut/modules.d/ создал папку 01test. Поместил два скрипта

```
[root@lvm ~]# mkdir /usr/lib/dracut/modules.d/01test
[root@lvm ~]# cd /usr/lib/dracut/modules.d/01test
[root@lvm 01test]# touch module-setup.sh
[root@lvm 01test]# cat >> module-setup.sh << EOF
> #!/bin/bash
> 
> check() {
>     return 0
> }
> 
> depends() {
>     return 0
> }
> 
> install() {
>     inst_hook cleanup 00 "${moddir}/test.sh"
> }
> EOF
```
```
[root@lvm 01test]# touch test.sh
[root@lvm 01test]# cat >> test.sh << EOF
> #!/bin/bash
> 
> exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
> cat <<'msgend'
> Hello! You are in dracut module!
>  ___________________
> < I'm dracut module >
>  -------------------
>    \
>     \
>         .--.
>        |o_o |
>        |:_/ |
>       //   \ \
>      (|     | )
>     /'\_   _/`\
>     \___)=(___/
> msgend
> sleep 10
> echo " continuing...."
> EOF
```
Пересобираю образ initrd
```
[root@lvm 01test]# dracut -f -v
[root@lvm 01test]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test
```
При перезагрузке, убрав опции, загрузилось изображение пингвина.