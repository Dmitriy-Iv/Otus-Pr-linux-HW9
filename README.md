# **Введение**

В данном домашнем задании нам необходимо получить практический опыт работы с загрузкой системы linux.

---

# **Попасть в систему без пароля несколькими способами** 

### **Способ 1. init=/bin/sh**
1. При загрузке виртуальной машине и выборе ядра нажимаем `e`, меняем параметры загрузки, добавляя в конец строки начинающейся с linux16 - `init=/bin/sh`, a также можно сразу перевести рутовую файловую систему в режим Read-Write - изменим букву `o` на `r` в этой же строке, как на скрине ниже.
![alt text](/screenshots/hw9-1.PNG?raw=true "Screenshot1")
2. Далее нажинаме `ctrl+x`, и проверяем монтирование файловой системы.
![alt text](/screenshots/hw9-2.PNG?raw=true "Screenshot2")

### **Способ 2. rd.break**
1. Аналогично пункту 1 из способа 1 попадаем в параметры загрузки системы и добавляем в конец строки начинающейся с linux16 - `rb.break`.
![alt text](/screenshots/hw9-3.PNG?raw=true "Screenshot3")
2. Меняем пароль у root.
![alt text](/screenshots/hw9-4.PNG?raw=true "Screenshot4")
3. Проверяем вход под новым паролем.
![alt text](/screenshots/hw9-5.PNG?raw=true "Screenshot5")

### **Способ 3. rw init=/sysroot/bin/sh**
1. В строке начинающейся с linux16 заменяем `ro` на `rw init=/sysroot/bin/sh`. 
![alt text](/screenshots/hw9-6.PNG?raw=true "Screenshot6")
2. Нажимаем `сtrl-x` и загружаемся.
![alt text](/screenshots/hw9-7.PNG?raw=true "Screenshot7")

---

# **Переименование VG** 
1. Для выполнения данного задания возьмём Vagrantfile из ДЗ по LVM. Запускаем VM, смотрим `vgs`
```
[root@lvm ~]# vgdisplay
  --- Volume group ---
  VG Name               VolGroup00
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <38.97 GiB
  PE Size               32.00 MiB
  Total PE              1247
  Alloc PE / Size       1247 / <38.97 GiB
  Free  PE / Size       0 / 0
  VG UUID               SA8LTU-F2yz-FEV1-RdgT-hw0Z-iRxh-yHFKuU

[root@lvm ~]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk
sdc                       8:32   0    2G  0 disk
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk
```
2. Переименовываем `Volume Group` и меняем старое название `VolGroup00` на `OtusRoot`с следующих файлах - `/etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg`.
Пересоздаём initrd image и перезагружаемся.
```
[root@lvm ~]# vgrename VolGroup00 OtusRoot
  Volume group "VolGroup00" successfully renamed to "OtusRoot"

[root@lvm ~]# sed -i 's/VolGroup00/OtusRoot/g' /etc/fstab /etc/default/grub /boot/grub2/grub.cfg

[root@lvm ~]# mkinitrd -f -v /boot/initramfs-(uname -r).img (uname -r)
-bash: syntax error near unexpected token `('
[root@lvm ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
Executing: /sbin/dracut -f -v /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***

[root@lvm ~]# reboot

```
3. Проверяем название Volume Group.
```
[vagrant@lvm ~]$ sudo -i
[root@lvm ~]# vgs
  VG       #PV #LV #SN Attr   VSize   VFree
  OtusRoot   1   2   0 wz--n- <38.97g    0
```

---

# **Добавить модуль в initrd** 
1. Создаём папку `/usr/lib/dracut/modules.d/01test`, создаём там два скрипта - `module-setup.sh` и `test.sh`.
```
[root@lvm ~]# mkdir /usr/lib/dracut/modules.d/01test

[root@lvm ~]# cd /usr/lib/dracut/modules.d/01test/

[root@lvm 01test]# touch module-setup.sh
[root@lvm 01test]# chmod 755 module-setup.sh

[root@lvm 01test]# touch test.sh
[root@lvm 01test]# chmod 755 test.sh

[root@lvm 01test]# cat module-setup.sh
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


[root@lvm 01test]# cat test.sh
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
2. Пересобираем образ initrd, проверяем что модуль `test` попал в наш образ initrd.
```
[root@lvm 01test]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
Executing: /sbin/dracut -f -v /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: test ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***

[root@lvm 01test]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test
```
3. Чтобы увидеть при загрузке желаемый результат, правим `/etc/default/grub`, убирая от туда `rhgb quiet`, обновляем `grub.cfg`.
```
[root@lvm 01test]# sed -i 's/ rhgb quiet//g' /etc/default/grub

[root@lvm 01test]# cat /etc/default/grub
GRUB_TIMEOUT=1
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=OtusRoot/LogVol00 rd.lvm.lv=OtusRoot/LogVol01 "
GRUB_DISABLE_RECOVERY="true"

[root@lvm 01test]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done
```
4. Перезагружаем и смотрим загрузку.
![alt text](/screenshots/hw9-8.PNG?raw=true "Screenshot8")