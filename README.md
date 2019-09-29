## ������� � ������� ��� ������ ����������� ���������
��� ���� ����� ��������������� GRUB � ������ �������� �������������� GUI VirtualBox. ��� ��������� ���� �������� ����� � ����� ��������������.
� ������ ������������ �� linux16 ������� ������� ����������� console=tty0, console=ttyS0,115200n8  � ������� � ����� ������ int=/bin/sh
�������� � ���������� ��������.  ����� �������� ������� ������� ���������� shell � �������� �������� ������� �������������� � read only
��� ��������� ������ �������� ��������� ������� 

	mount -o remount,rw /  -- �������������� �������� ������� � ������ read-write
	/usr/sbin/load_policy -i  -- ��� ��� � ������� ����������� SELinux �������� �������� ��� ���������� ������� ���������
	passwd root          -- ������� ������ root
	mount -o remount,ro /   -- �������������� �������� ������� � ����� read only
	exec /usr/sbin/reboot -f  -- �������� ������� � ������������

� ���������� ����� ������������ ������� ����������� � ����� �������� ������� root

������ ������ �� ������ ���������� �������, � ����� ������ ������ �������  rd.break. � ���������� ������� �������� ������ � ����� emergency mode 
� ���� ���������� shell � �������� ������� /sysroot. ��� ��������� ������ �������� ��������� �������
      
	mount -o remount,rw /sysroot  -- �������������� �������� ������� � ������ read-write
	chroot /sysroot   -- ��������� �������� ����������
	passwd root   -- ����� ����� ������ root
	touch /.autorelabel  -- ��� ��� � ������� ����������� SELinux, ��� ������������ ����� ������������� �������� �������� ������� ��� SELinux


������ ������ �������� �����������. ���, � ���� �������� � ������ linux16 �������� ���������� ro (read only) �� rw init=/sbin/sysroot/sh
��� �������� � emergency mode ������� �������� ������� ��� �������������� � ������ read-write. ��������� ������� �� ��.


## �������������� volume group
�������� ������������ � �������:
	[root@lvm vagrant]# lsblk
	NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
	sda                     8:0    0   40G  0 disk 
	|-sda1                  8:1    0    1M  0 part 
	|-sda2                  8:2    0    1G  0 part /boot
	`-sda3                  8:3    0   39G  0 part 
  	 |-VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
	  `-VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]

�������� �������������� ������

	vgrename VolGroup00 newroot

���� ��������� � ����� /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. ������� ������ �� ��������.  ���������� ����� initrd, ����� ����
�������� ��� ����� ������

	mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

�������� ������������ �������. � ���������� ������� ����� ������������:

	[root@lvm vagrant]# lsblk
	NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
	sda                     8:0    0   40G  0 disk 
	 |-sda1                  8:1    0    1M  0 part 
	 |-sda2                  8:2    0    1G  0 part /boot
	 `-sda3                  8:3    0   39G  0 part 
  	   |-newgroup-LogVol00 253:0    0 37.5G  0 lvm  /
  	   `-newgroup-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]


## �������� ������ � initrd

��� ���������� ������ ������ ���������� 01pingvin � /usr/lib/dracut/modules.d. �������� � ��� ������� module-setup.sh � pingvin.sh
���������� ����� initrd, ����� ����� ������ ��� ������� � �����

	dracut -f -v 

�������� ��� ������ �������. 

	[root@lvm vagrant]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep pingvin
	pingvin

���������� ������� � �� 10 ������ � ��������� �������� ������� ��������
