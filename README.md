## Попасть в систему без пароля несколькими способами
Для того чтобы отредактировать GRUB в момент загрузки воспользовался GUI VirtualBox. При появлении меню загрузки зашел в режим редактирования.
В строке начинающейся на linux16 поменял местами очередность console=tty0, console=ttyS0,115200n8  и дописал в конце строки int=/bin/sh
Отправил в дальнейшую загрузку.  После загрузки модулей получил управление shell в корневой файловой системе смонтированной в read only
Для изменения пароля выполнил следующие команды 

	mount -o remount,rw /  -- перемонтировал файловую систему в режиме read-write
	/usr/sbin/load_policy -i  -- так как в системе активирован SELinux загрузил политику для сохранения текущих изменений
	passwd root          -- задание пароля root
	mount -o remount,ro /   -- перемонтировал файловую систему в режим read only
	exec /usr/sbin/reboot -f  -- отправил систему в перезагрузку

В результате после перезагрузки успешно залогинился с вновь заданным паролем root

Второй способ во многом аналогичен первому, в конце строки только дописал  rd.break. В результате процесс загрузки входим в режим emergency mode 
и дает управление shell в файловой системе /sysroot. Для изменения пароля выполнил следующие команды
      
	mount -o remount,rw /sysroot  -- перемонтировал файловую систему в режиме read-write
	chroot /sysroot   -- установил корневую директорию
	passwd root   -- задал новый пароль root
	touch /.autorelabel  -- так как в системе активирован SELinux, при перезагрузке будет автоматически заменена файловая система для SELinux


Третий способ вариация предыдущего. Так, в меню загрузки в строке linux16 заменяем инструкцию ro (read only) на rw init=/sbin/sysroot/sh
При загрузке в emergency mode получил файловую систему уже смонтированную в режиме read-write. Остальные команды те же.


## Переименование volume group
Исходная конфигурация в системе:
	[root@lvm vagrant]# lsblk
	NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
	sda                     8:0    0   40G  0 disk 
	|-sda1                  8:1    0    1M  0 part 
	|-sda2                  8:2    0    1G  0 part /boot
	`-sda3                  8:3    0   39G  0 part 
  	 |-VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
	  `-VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]

Выполнил переименование группы

	vgrename VolGroup00 newroot

Внес изменения в файлы /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. Образцы файлов во вложении.  Пересоздал образ initrd, чтобы было
включено имя новой группы

	mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

Выполнил перезагрузку системы. В результате получил такую конфигурацию:

	[root@lvm vagrant]# lsblk
	NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
	sda                     8:0    0   40G  0 disk 
	 |-sda1                  8:1    0    1M  0 part 
	 |-sda2                  8:2    0    1G  0 part /boot
	 `-sda3                  8:3    0   39G  0 part 
  	   |-newgroup-LogVol00 253:0    0 37.5G  0 lvm  /
  	   `-newgroup-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]


## Добавить модуль в initrd

Для добавления модуля создал директорию 01pingvin в /usr/lib/dracut/modules.d. Поместил в нее скрипты module-setup.sh и pingvin.sh
Пересобрал образ initrd, чтобы новый модуль был включен в образ

	dracut -f -v 

Проверил что модуль включен. 

	[root@lvm vagrant]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep pingvin
	pingvin

Перегрузил систему и на 10 секунд в терминале появился рисунок пингвина
