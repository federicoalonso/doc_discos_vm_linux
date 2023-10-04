# Ampliar espacio de disco VM

## PRIMERO

Hacer un BK del server

Avisar

### Procedimiento

Para verificar el espacio en disco, debemos ingresar al server, y como root ejecutar:

```bash
[root@infra-zabbix ~]# df -h
Filesystem                           Size  Used Avail Use% Mounted on
devtmpfs                             3.8G     0  3.8G   0% /dev
tmpfs                                3.8G     0  3.8G   0% /dev/shm
tmpfs                                1.6G  8.9M  1.5G   1% /run
/dev/mapper/almalinux-root            14G  3.9G   11G  28% /
/dev/mapper/almalinux-home           2.0G  317M  1.7G  16% /home
/dev/mapper/almalinux-var             18G  9.1G  0.9G  91% /var
/dev/mapper/almalinux-var_log        6.0G  1.1G  5.0G  18% /var/log
/dev/mapper/almalinux-var_tmp       1014M   40M  975M   4% /var/tmp
/dev/mapper/almalinux-tmp           1014M  132M  883M  13% /tmp
/dev/mapper/almalinux-var_log_audit  3.0G   89M  3.0G   3% /var/log/audit
/dev/sda1                           1014M  301M  714M  30% /boot
tmpfs                                769M     0  769M   0% /run/user/1140814649
```

Podemos ver que la partición /var tiene un 91% ocupado, siendo ésta la que deseamos ampliar.

Debemos verificar el tipo de file system y si tiene volúmenes lógicos o sólo físicos.

```bash
[root@infra-zabbix ~]# lsblk -f
NAME                        FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                                                          
├─sda1                      xfs                        ab71300e-3fc3-40b0-8393-679caa908835    713.5M    30% /boot
└─sda2                      LVM2_member LVM2 001       P5bjH5-SUug-ojtq-HjoL-IujV-h3dn-2ycwnl                
  ├─almalinux-root          xfs                        95313bba-c4f4-4e5d-8426-bd87d0906076     10.2G    27% /
  ├─almalinux-swap          swap        1              cb41c136-76b7-4494-b8da-4a1bca615608                  [SWAP]
  ├─almalinux-home          xfs                        d1500470-0c9c-44e6-8c4c-f5e52dc4bae4      1.7G    16% /home
  ├─almalinux-var_tmp       xfs                        0f478bc5-cd17-478f-b98d-56478a7996af    974.6M     4% /var/tmp
  ├─almalinux-var           xfs                        4daccbaf-896e-4dee-8af4-fcead93241b5      8.9G    91% /var
  ├─almalinux-var_log       xfs                        8197a3c1-a02c-4576-866c-8de47c97c11f      4.9G    19% /var/log
  ├─almalinux-var_log_audit xfs                        ea02f476-7aff-4125-97f9-17b4dffc66eb      2.9G     3% /var/log/audit
  └─almalinux-tmp           xfs                        401a09c2-ba8b-4c78-883d-7a027c4c6ab9    880.9M    13% /tmp

```

Acá vemos que el tipo de filesystem es xfs y debemos ampliar la partición sda2.

```bash
[root@infra-zabbix ~]# vgdisplay 
  --- Volume group ---
  VG Name               almalinux
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  9
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                8
  Open LV               8
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <39.00 GiB
  PE Size               4.00 MiB
  Total PE              9983
  Alloc PE / Size       9983 / <39.00 GiB
  Free  PE / Size       0 / 0   
  VG UUID               Dh19LC-507t-4780-2YNe-ysZK-fmyb-brqxok
```

Acá vemos que contiene un volumen lógico llamado almalinux.

### Teoría

La teoría para entender lo realizado en este instructivo se encuentra en el siguiente enlace:

[https://bookstack.genexusconsulting.com/public/books/trabajos-varios/page/teoria-file-system](https://bookstack.genexusconsulting.com/public/books/trabajos-varios/page/teoria-file-system "https://bookstack.genexusconsulting.com/public/books/trabajos-varios/page/teoria-file-system")

### Pasos para incrementar el espacio

#### Asignarle más espacio de disco

Primero debemos agregarle más espacio en el vcenter.

Con el comando cfdisk podemos chequear que tenemos espacio libre en los discos. En caso de que no suceda, debemos reiniciar el server.

[![image.png](https://bookstack.genexusconsulting.com/public/uploads/images/gallery/2022-12/scaled-1680-/2fUimage.png)](https://bookstack.genexusconsulting.com/public/uploads/images/gallery/2022-12/2fUimage.png)

#### Cambiar la tabla de particiones

Lo primero que debemos hacer es modificar nuestra tabla de particiones para que sda2 termine al final del disco: ¡no se preocupe, no perderá sus datos existentes! Sin embargo, esta tarea requerirá un reinicio para escribir los cambios que vamos a realizar y también para volver a leer la tabla de particiones actualizada.

Comencemos ejecutando el siguiente comando:

```bash
[root@infra-zabbix ~]# fdisk /dev/sda

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

This disk is currently in use - repartitioning is probably a bad idea.
Its recommended to umount all file systems, and swapoff all swap
partitions on this disk.


Command (m for help): p

Disk /dev/sda: 48 GiB, 51539607552 bytes, 100663296 sectors
Disk model: Virtual disk    
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x65b76dbc

Device     Boot   Start      End  Sectors Size Id Type
/dev/sda1  *       2048  2099199  2097152   1G 83 Linux
/dev/sda2       2099200 83886079 81786880  39G 8e Linux LVM
```

Esto hará que la terminal entre en modo fdisk: una vez allí, escriba p para imprimir la tabla de particiones actual: es muy importante tomar nota de los valores numéricos de las columnas START y END para la partición /dev/sda2, ya que estamos los necesitare pronto. Si quiere asegurarse de no perderlos o escribirlos mal, simplemente imprímalos en pantalla o en papel.

Una vez hecho esto, escriba d para acceder al modo de eliminación y luego el número del registro de partición que desea eliminar (que sería 2 en el escenario anterior). Nuevamente, NO SE PREOCUPE: no está eliminando los datos de la partición, solo sus direcciones de asignación en la tabla de particiones.

Inmediatamente después, escriba n para crear una segunda partición completamente nueva: elija el mismo modo de partición que la anterior (que sería Primaria en el escenario anterior), luego ingrese el valor numérico INICIO que registró anteriormente, que debería ser el valor predeterminado sugerido; también, asegúrese de que el final esté al final del disco, que también debería ser el valor predeterminado.

Por último, pero no menos importante, necesitamos cambiar el tipo de partición de Linux a Linux LVM: para hacerlo, escriba t para acceder al modo de cambio de tipo de partición, luego 2, luego 8e y listo.

Cuando haya terminado, escriba p para revisar su nuevo diseño de partición. Asegúrese de verificar tres veces que el inicio de la nueva segunda partición esté exactamente donde estaba la antigua segunda partición: ¡esto es muy importante! En caso de que lo haya hecho mal, escriba d para eliminar la partición equivocada y empezar de nuevo.

Si todo parece correcto, emita w para escribir la tabla de particiones en el disco.

#### Reiniciar

Inmediatamente después de escribir la nueva tabla de particiones en el disco, recibirá inmediatamente un mensaje de error del sistema debido a que no se pudo acceder a la tabla de particiones para leerla porque el disco está en uso. Es por eso que necesitamos reiniciar nuestro sistema.

#### Problemas

Me pasó que no tomó el tipo de partición Linux LVM y me dejaba en emergencia dracut, ahí después de muchos rezos encontré la guía definitiva para solucionar el error. Quedó un error en la escritura de la partición 2 en los primeros sectores, se puede corregir a mano, la guía está acá: [http://technicalprose.blogspot.com/2020/01/dracut-emergency-lvm-label-missing.html](http://technicalprose.blogspot.com/2020/01/dracut-emergency-lvm-label-missing.html) por las dudas la copio al final del procedimiento.

#### Expandir la partición LVM

Correr el siguiente comando:

```
[root@infra-zabbix ~]# pvresize /dev/sda2
  Physical volume "/dev/sda2" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized
```

Justo después de eso, ejecute cfdisk nuevamente ahora. Si todo salió bien, debería ver lo siguiente:

```
[root@infra-zabbix ~]# cfdisk

                                                                                               Disk: /dev/sda
                                                                           Size: 48 GiB, 51539607552 bytes, 100663296 sectors
                                                                                     Label: dos, identifier: 0x65b76dbc

    Device                      Boot                                   Start                       End                  Sectors                 Size                 Id Type
>>  /dev/sda1                   *                                       2048                   2099199                  2097152                   1G                 83 Linux                               
    /dev/sda2                                                        2099200                 100663295                 98564096                  47G                 8e Linux LVM

 ┌────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
 │ Partition type: Linux (83)                                                                                                                                                                             │
 │     Attributes: 80                                                                                                                                                                                     │
 │Filesystem UUID: ab71300e-3fc3-40b0-8393-679caa908835                                                                                                                    							      │
 │     Filesystem: xfs                                                                                                                                                                                    │
 │     Mountpoint: /boot (mounted)                                                                                                                                                                        │
 └────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
                                                       [Bootable]  [ Delete ]  [ Resize ]  [  Quit  ]  [  Type  ]  [  Help  ]  [  Write ]  [  Dump  ]

                                                                     Device is currently in use, repartitioning is probably a bad idea.
                                                                                    Quit program without writing changes
```

Podemos ver que la partición tomó todo el tamaño.

#### Extender el volumen Lógico

Lo siguiente que debemos hacer ahora es extender nuestro volumen lógico a ese espacio. Para hacer eso, necesitamos recuperar su ruta, lo que se puede hacer emitiendo el siguiente comando:

```bash
[root@infra-zabbix ~]# lvdisplay -v
.
.
.
  --- Logical volume ---
  LV Path                /dev/almalinux/var
  LV Name                var
  VG Name                almalinux
  LV UUID                XDGcl1-UOjh-auR3-Alf6-ixPv-4Axw-02MeQi
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2022-07-19 10:12:49 -0300
  LV Status              available
  # open                 1
  LV Size                18.00 GiB
  Current LE             4608
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:4
.
.
.
```

De aquí tomamos el nombre del volumen lógico que queremos ampliar, la ruta es /dev/almalinux/var.

Una vez recuperado, podemos extender ese volumen lógico con el siguiente comando:

```bash
[root@infra-zabbix ~]# lvextend -l +100%FREE /dev/almalinux/var
  Size of logical volume almalinux/var changed from 10.00 GiB (2560 extents) to 18.00 GiB (4608 extents).
  Logical volume almalinux/var successfully resized.
```

#### Expandir el File System

Ahora que el volumen lógico se ha extendido con éxito para usar todo el espacio no asignado, todavía tenemos que realizar un último paso: aumentar el sistema de archivos para que coincida con el tamaño del volumen lógico. Hacer eso es tan fácil como escribir uno de los siguientes comandos, dependiendo de si estamos usando EXT4 (que debería ser el predeterminado hoy en día) o XFS. Nuevamente, tenemos que usar la ruta del volumen lógico aquí, la misma que usamos con el comando lvextend hace un momento.

Para EXT4, escriba lo siguiente:

```bash
resize2fs /dev/vg/lv_raíz
```

Para XFS, que es nuestro caso, escriba lo siguiente:

```bash
[root@infra-zabbix ~]# xfs_growfs /dev/almalinux/var
meta-data=/dev/mapper/almalinux-var isize=512    agcount=4, agsize=655360 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1
data     =                       bsize=4096   blocks=2621440, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2621440 to 4718592
```

Esto activará el cambio de tamaño en línea del sistema de archivos, que se ampliará para utilizar todo el espacio de volumen lógico disponible.

#### Verificar

Para verificar, volvemos con el primer comando y debemos ver el nuevo espacio asignado.

```bash
[root@infra-zabbix ~]# df -h
Filesystem                           Size  Used Avail Use% Mounted on
devtmpfs                             3.8G     0  3.8G   0% /dev
tmpfs                                3.8G     0  3.8G   0% /dev/shm
tmpfs                                1.6G  8.9M  1.5G   1% /run
/dev/mapper/almalinux-root            14G  3.9G   11G  28% /
/dev/mapper/almalinux-home           2.0G  317M  1.7G  16% /home
/dev/mapper/almalinux-var             18G  9.1G  9.0G  51% /var
/dev/mapper/almalinux-var_log        6.0G  1.1G  5.0G  18% /var/log
/dev/mapper/almalinux-var_tmp       1014M   40M  975M   4% /var/tmp
/dev/mapper/almalinux-tmp           1014M  132M  883M  13% /tmp
/dev/mapper/almalinux-var_log_audit  3.0G   89M  3.0G   3% /var/log/audit
/dev/sda1                           1014M  301M  714M  30% /boot
tmpfs                                769M     0  769M   0% /run/user/1140814649
```

### Guía solución de problema de no booteo

# [Dracut emergency! LVM label missing after fdisk catastrophe](http://technicalprose.blogspot.com/2020/01/dracut-emergency-lvm-label-missing.html)

<div class="article-content entry-content" id="bkmrk-one-sunny-and-crisp-" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">One sunny and crisp Saturday morning in January, when one of my kids was out at football practise, and the other children were quietly amusing themselves with their homework, I thought I spotted the first opportunity I'd had in a long time to do a quick upgrade of my development environment. I was going to upgrade my Fedora VM in VirtualBox to the latest version - it was already many years out of date and it really was high time it were refreshed - but I was about to be rudely reminded that there is no such thing as a *quick* upgrade because, and after first discovering that upgrading from a version many years out of date to the latest available version was not a single operation but one with many steps which lasted through Saturday and into Sunday and after successfully hopping 6 versions (ok it really was *many* years behind, what can I say I've been a busy parent!), the next hop wouldn't be possible as my root filesystem only had about 1.5GB free space which wasn't enough to perform the required upgrade!  
  
The VDI (the virtual disk on which the root filesystem for my VM resided) was hosted on Windows 10 and my C: drive still had plenty of available space - well it did after I cleaned out my Downloads folder, you can probably imagine how many GB I could recover by deleting numerous ISOs from Linux distributions I had downloaded over the years - and so, I concluded, all I would need to do was make my virtual disk bigger and grow my root filesystem. How hard can that be?!  
  
I found the [instructions](http://derekmolloy.ie/resize-a-virtualbox-disk/) I needed for resizing the VDI, and as I had snapshots I had to include the all important step of resizing each of the snapshots associated with the VDI as well, otherwise the new disk size would not be reported up to the Linux guest. I then proceeded to follow [these instructions](https://superuser.com/questions/335038/how-do-you-add-more-space-to-a-fedora-lvm-partition) to extend the root filesystem, specifically the steps provided in the *accepted answer*, beginning with the steps in `fdisk` required to recreate the partition with a larger number of blocks. I remember following a similar procedure in my former life as a Solaris Sysadmin (using `format` instead of `fdisk` of course which, I might point out, had a sensible option for *modifying* a partition rather than having to delete and re-add it as you do on Linux), so I felt pretty confident that the whole thing would be done in a few minutes, which was handy, because this whole upgrade m'larkey had already taken much longer than I had planned and I was under increasing time pressure.  
  
It started out well enough:</div></div>```
[root@galileo mark]# fdisk /dev/sda

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): d
Partition number (1,2, default 2):

Partition 2 has been deleted.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (2-4, default 2):
First sector (1026048-41943039, default 1026048):
Last sector, +sectors or +size{K,M,G,T,P} (1026048-41943039, default 41943039):

Created a new partition 2 of type 'Linux' and of size 19.5 GiB.
Partition #2 contains a LVM2_member signature.

Do you want to remove the signature? [Y]es/[N]o:
```

<div class="article-content entry-content" id="bkmrk-do-i-want-to-remove-" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">Do I want to remove the signature? Hmm, do I want to remove the signature ... ? This question didn't appear in the steps I was dutifully following (it later transpired that this question was added in a later version of `fdisk`). Well, unfortunately for me I a) didn't know much about LVM at the time I was asked the question (this was about to change!), b) the line that reads 'Partition #2 contains a LVM2_member signature' was dark red on a dark grey background (default Fedora colours) which made it very hard to read (really, would you paint a road sign in red on a black background?? I think not!) and c) as I may have already mentioned ... I was in a bit of a hurry (it's always bad to be in a hurry when invoking `fdisk`!). So I said 'Y' I wanted to remove the signature :-) Confident smile. Of course I presumed this meant that yes I was fine to create the new partition. Are you cringing yet? Sorry I should have warned the squeamish to "look away now". I didn't really know what I was doing at this point. Crazy fool!  
  
Ok, then onto the reboot and my heart sank to my feet. The VM wouldn't boot, I got this instead:</div></div>```
[  OK  ] Started Show Plymouth Boot Screen.
[  OK  ] Reached target Paths.
[  OK  ] Started Forward Password Requests to Plymouth Directory Watch.
[  OK  ] Reached target Basic System.

dracut-initqueue[426]: Warning: dracut-initqueue timeout - starting timeout scripts
... (repeated many times) ...

Entering emergency mode. Exit the shell to continue.
Type "journalctl" to view system logs.
You might want to save "/run/initramf/rdsosreport.txt" to a USB stick or /boot
after mounting them and attach it to a bug report.

dracut:/#
```

<div class="article-content entry-content" id="bkmrk-dracut-emergency-she" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">Dracut emergency shell?! This was uncharted territory, I mean I knew someone once who came from Plymouth, but Dracut sounded like an altogether worse place to be. A bit of googling around it seems there wasn't much I could do in this shell besides a few simple commands, including this one:</div></div>```
dracut:/# blkid
/dev/sda1: UUID="..." TYPE="ext4" PARTUUID="..."
/dev/sda2: PARTUUID="..."
```

<div class="article-content entry-content" id="bkmrk-...-but-even-my-untr" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">... but even my untrained eye (i.e. never having run that command before) could see a slightly troubling and inexplicable difference between `/dev/sda1` and `/dev/sda2`.  
  
So I booted into the Fedora Live image to do some basic analysis. Well ok, that sounds as if I knew at the time that that was the next logical step, it turns out that the Fedora installation ISO is called Fedora *Live* and will allow you to try out Fedora from a DVD (which would probably be painfully slow) or as a virtual DVD in VirtualBox (which actually performs acceptably well) without actually installing it. Once in Fedora Live, and having already frantically read about a few more LVM commands while I was waiting for it to boot, I could now run:</div></div>```
[root@localhost-live liveuser]# pvs
[root@localhost-live liveuser]# ls /dev/mapper
control  live-base  live-rw
```

<div class="article-content entry-content" id="bkmrk-not-being-an-lvm-exp" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">Not being an LVM expert, this didn't look good. LVM seemed to be unaware of any of my physical volumes.</div></div>```
[root@localhost-live liveuser]# fdisk -l /dev/sda
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xcf55af00

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sda1  *       2048  1026047  1024000  500M 83 Linux
/dev/sda2       1026048 41943039 40916992 19.5G 8e Linux LVM
```

<div class="article-content entry-content" id="bkmrk-well-i-could-see-tha" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">Well I could see that `/dev/sda2` was there, that was a relief, and the size was correct, but I couldn't find any way of mounting `/dev/sda2`.  
  
After a day and a night of worrying and googling (woogling?), I decided I should compare the first few blocks of the broken disk with the first few blocks of the disk as it was in a previous snapshot. Yes, as this is VirtualBox, I had previous snapshots, albeit a few months old, so I didn't want to just restore from the last snapshot and be done with it, because I didn't fancy losing a few months of work. I wanted to see what the difference was between a working disk and a broken disk.  
  
I made a clone from a previous snapshot in VirtualBox and booted that up to compare:</div></div>```
[root@galileo mark]# pvs
  PV         VG             Fmt  Attr PSize   PFree
  /dev/sda2  fedora_galileo lvm2 a--  <19.51g    0
[root@galileo mark]# ls /dev/mapper
control  fedora_galileo-root  fedora_galileo-swap
```

<div class="article-content entry-content" id="bkmrk-right%2C-so-this-is-wh" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">Right, so this is what it is supposed to look like!</div></div>```
[root@galileo mark]# fdisk -l /dev/sda
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xcf55af00

Device     Boot   Start      End  Sectors  Size Id Type
/dev/sda1  *       2048  1026047  1024000  500M 83 Linux
/dev/sda2       1026048 41943039 40916992 19.5G 8e Linux LVM
```

<div class="article-content entry-content" id="bkmrk-using%C2%A0dd%C2%A0to-read-the" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">Using `dd` to read the first few disk blocks on the broken disk, I found this label 512 bytes into it:</div></div>```
[root@localhost-live liveuser]# dd if=/dev/sda2 count=1 skip=8 bs=64 2>/dev/null | od -Ax -tx1z -v
000000 4c 41 42 45 4c 4f 4e 45 01 00 00 00 00 00 00 00  >LABELONE........<
000010 35 da 50 fb 20 00 00 00 00 00 00 00 00 00 00 00  >5.P. ...........<
000020 56 6e 47 6d 50 54 30 4e 30 55 77 68 49 6e 35 46  >VnGmPT0N0UwhIn5F<
000030 70 62 4c 33 64 57 53 36 76 57 4f 45 53 55 6d 57  >pbL3dWS6vWOESUmW<
000040
```

<div class="article-content entry-content" id="bkmrk-and-compared-with-th" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">And compared with the disk from the snapshot:</div></div>```
[root@galileo mark]# dd if=/dev/sda2 count=1 skip=8 bs=64 2>/dev/null | od -Ax -tx1z -v
000000 4c 41 42 45 4c 4f 4e 45 01 00 00 00 00 00 00 00  >LABELONE........<
000010 35 da 50 fb 20 00 00 00 4c 56 4d 32 20 30 30 31  >5.P. ...LVM2 001<
000020 56 6e 47 6d 50 54 30 4e 30 55 77 68 49 6e 35 46  >VnGmPT0N0UwhIn5F<
000030 70 62 4c 33 64 57 53 36 76 57 4f 45 53 55 6d 57  >pbL3dWS6vWOESUmW<
000040
```

<div class="article-content entry-content" id="bkmrk-%28alas%C2%A0xxd%C2%A0was-not-in" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">(Alas `xxd` was not in the Fedora Live image, so I had to resort to the old `od` school method, conveniently documented with the correct incantation for a hex dump in the examples at the bottom of `man od`).  
  
Ok, so everything was the same, except in the broken disk the 8 bytes that read "LVM2 001" were missing. According to [LVM Internals](http://talk.manageiq.org/t/lvm-internals-structures-disk-layout/1328) the "LVM2 001" part seems to be the `lvm_type`.  
  
What if I recreate that label? I couldn't find a tool to repair this with a quick google around, but these are the bytes I needed to change:</div></div>```
[root@localhost-live liveuser]# dd if=/dev/sda2 count=1 skip=67 bs=8 2>/dev/null | od -Ax -tx1z -v
000000 00 00 00 00 00 00 00 00                          >........<
000008
```

<div class="article-content entry-content" id="bkmrk-and-this-is-how-it-l" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">And this is how it looked in the snapshot:</div></div>```
[root@galileo mark]# dd if=/dev/sda2 count=1 skip=67 bs=8 2>/dev/null | od -Ax -tx1z -v
000000 4c 56 4d 32 20 30 30 31                          >LVM2 001<
000008
```

<div class="article-content entry-content" id="bkmrk-so-i-decided-to-make" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">So I decided to make the change using `vi` in binary mode, why not! (A colleague later quipped, in jest, that perhaps I should have just run `vi -b /dev/sda2`, but I wouldn't recommend that to anyone). First I used `dd` to read the first 1KB of data from the disk to a file I could edit, before launching `vi`:</div></div>```
[root@localhost-live liveuser]# dd if=/dev/sda2 of=tmpblock count=1 bs=1024
[root@localhost-live liveuser]# vi -b tmpblock
```

<div class="article-content entry-content" id="bkmrk-i-wanted-to-change-t" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">I wanted to change this line displayed in `vi` as:</div></div>```
^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@LABELONE^A^@^@^@^@^@^@^@5
<da>P<fb> ^@^@^@^@^@^@^@^@^@^@^@VnGmPT0N0UwhIn5Fp
```

<div class="article-content entry-content" id="bkmrk-to%3A" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">to:</div></div>```
^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@LABELONE^A^@^@^@^@^@^@^@5
<da>P<fb> ^@^@^@LVM2 001VnGmPT0N0UwhIn5Fp
```

<div class="article-content entry-content" id="bkmrk-if-you%27re-not-famili" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">If you're not familiar with `vi`, each `^@` that appears when you edit a binary file represents a single null byte (a value of zero). The following `vi` commands will do the trick:  
  
<table><tbody><tr><td>`/LABELONE`</td><td>*(search for text "LABELONE")*</td></tr><tr><td>`24l`</td><td>*(move 24 characters - bytes - to the right, N.B. that's a lowercase L)*</td></tr><tr><td>`8s`</td><td>*(change the next 8 characters)*</td></tr><tr><td>`LVM2 001`</td><td>*(the new string I want)*</td></tr><tr><td>`<Esc>`</td><td>*(press Escape to return to command mode)*</td></tr><tr><td>`:wq`</td><td>*(save and quit)*</td></tr></tbody></table>

  
Checking the output of `od` again to confirm the change is correct:</div></div>```
[root@localhost-live liveuser]# od -Ax -tx1z -v tmpblock
000200 4c 41 42 45 4c 4f 4e 45 01 00 00 00 00 00 00 00  >LABELONE........<
000210 35 da 50 fb 20 00 00 00 4c 56 4d 32 20 30 30 31  >5.P. ...LVM2 001<
000220 56 6e 47 6d 50 54 30 4e 30 55 77 68 49 6e 35 46  >VnGmPT0N0UwhIn5F<
000230 70 62 4c 33 64 57 53 36 76 57 4f 45 53 55 6d 57  >pbL3dWS6vWOESUmW<
```

<div class="article-content entry-content" id="bkmrk-then-i-wrote-it-back" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">Then I wrote it back to the disk:</div></div>```
[root@localhost-live liveuser]# dd if=tmpblock of=/dev/sda2
2+0 records in
2+0 records out
1024 bytes (1.0 kB, 1.0 KiB) copied, 0.111038 s, 9.2 kB/s
```

<div class="article-content entry-content" id="bkmrk-and-confirmed-the-da" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">And confirmed the data is now correct on the disk:</div></div>```
[root@localhost-live liveuser]# dd if=tmpblock of=/dev/sda2
2+0 records in
2+0 records out
1024 bytes (1.0 kB, 1.0 KiB) copied, 0.111038 s, 9.2 kB/s
[root@localhost-live liveuser]# dd if=/dev/sda2 count=1 skip=8 bs=64 2>/dev/null | od -Ax -tx1z -v
000000 4c 41 42 45 4c 4f 4e 45 01 00 00 00 00 00 00 00  >LABELONE........<
000010 35 da 50 fb 20 00 00 00 4c 56 4d 32 20 30 30 31  >5.P. ...LVM2 001<
000020 56 6e 47 6d 50 54 30 4e 30 55 77 68 49 6e 35 46  >VnGmPT0N0UwhIn5F<
000030 70 62 4c 33 64 57 53 36 76 57 4f 45 53 55 6d 57  >pbL3dWS6vWOESUmW<
000040
```

<div class="article-content entry-content" id="bkmrk-pvs%C2%A0and%C2%A0%2Fdev%2Fmapper%C2%A0" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">`pvs` and `/dev/mapper` should now be correct:</div></div>```
[root@localhost-live liveuser]# pvs
  PV         VG             Fmt  Attr PSize   PFree
  /dev/sda2  fedora_galileo lvm2 a--  <19.51g    0
[root@localhost-live liveuser]# ls /dev/mapper
control  fedora_galileo-root  fedora_galileo-swap  live-base  live-rw
```

<div class="article-content entry-content" id="bkmrk-eureka%21%C2%A0-my-volumes-" itemprop="articleBody"><div class="article-content entry-content" itemprop="articleBody">Eureka! My volumes are visible again! Can I mount them?</div></div>```
[root@localhost-live liveuser]# mount /dev/mapper/fedora_galileo-root /mnt
[root@localhost-live liveuser]# df -h /mnt
Filesystem                       Size  Used Avail Use% Mounted on
/dev/mapper/fedora_galileo-root   18G   12G  5.6G  68% /mnt
```

<div class="article-content entry-content" id="bkmrk-perfect%2C-now-my-syst" itemprop="articleBody">Perfect, now my system will boot again!  
  
Lessons learned:  
1. Always make a backup or snapshot before fooling around with your disks.
2. Don't enter `fdisk` if you're in a hurry - slow down and take your time.
3. Always read prompts properly when you get them. If they are in red and hard to read as a result, that doesn't mean to ignore them, that normally means they're quite important.
4. If you have accidentally asked `fdisk` to "remove the signature" then the above steps may help you.
5. Woogling<sup>TM</sup> is the road to glory.
6. `vi -b /dev/sda2` would probably not be a good thing to do.

</div>