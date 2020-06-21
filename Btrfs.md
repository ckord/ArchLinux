# ArchLinux with Btrfs

## Partitionierung

Datenträger abfragen  
```lsblk```  
Im folgenden Beispiel ist der zu verwendende Datenträger ```sda```  
Ich installiere mit UEFI und möchte ```grub``` als Bootmanager verwenden. Wir benötigen zwei Partitionen: ```EFIBOOT``` und eine Partition mit dem gesamten restlichen freien Speicherplatz. Hier werden die Subvolumes für ```root``` und ```home``` erstellt. Ein Swapfile wird seit dem Linux Kernel 5.0 auf Btrfs unterstützt, allerdings nur mit Limitierungen.

Partitionierungstool mit ```cfdisk``` starten

### EFIBOOT Parition

```EFIBOOT``` wird die erste Partition ```sda1``` am Anfang des Datenträgers ```sda```. Ich setze die Größe auf ```1GB```.

### Btrfs Partition

```Btrfs``` wird die zweite Partition ```sda2```, direkt nach ```EFIBOOT```. Sie füllt den kompletten Rest des Datenträgers auf.  

## Dateisysteme

### EFIBOOT Dateisystem

```mkfs.fat -F 32 -n EFIBOOT /dev/sda1```

### Btrfs Dateisystem

```mkfs.btrfs -L Btrfs /dev/sda2```

## Subvolumes und Mountpunkte

Das Default-Subvolume mounten  
```mount /dev/sda2 /mnt```  

In den Mountpunkt wechseln  
```cd /mnt```  

Subvolumes für ```/```, ```home```, ```snapshots``` und ```pkg``` erstellen  
```btrfs subvolume create @```  
```btrfs subvolume create @home```  
```btrfs subvolume create @snapshots```  
```btrfs subvolume create @pkg```  
*Optional:*  
*Subvolume für ```swap``` erstellen*  
*```btrfs subvolume create @swap```*  

Subvolumes anzeigen lassen  
```btrfs subvolume list -p /mnt```  

Unmount des Default-Subvolumes 5  
```umount /mnt```  

Subvolume ```/``` mounten  
```mount -o rw,noatime,space_cache,compress=lzo,ssd,subvol=@ /dev/sda2 /mnt```  

Ordner für die Mountpunkte ```boot```, ```home```, ```snapshots``` und ```pkg``` erstellen  
```mkdir -p /mnt/boot```  
```mkdir -p /mnt/home```  
```mkdir -p /mnt/.snapshots```  
```mkdir -p /mnt/var/cache/pacman/pkg```  

*Optional:*  
```mkdir -p /mnt/.swap```

Subvolumes ```home```, ```snapshots``` und ```pkg``` mounten  
```mount -o rw,noatime,space_cache,compress=lzo,ssd,subvol=@home /dev/sda2 /mnt/home```  
```mount -o rw,noatime,space_cache,compress=lzo,ssd,subvol=@snapshots /dev/sda2 /mnt/.snapshots```  
```mount -o rw,noatime,space_cache,compress=lzo,ssd,subvol=@pkg /dev/sda2 /mnt/var/cache/pacman/pkg```  
*Optional:*  
```mount -o rw,noatime,space_cache,compress=lzo,ssd,subvol=@swap /dev/sda2 /.swap```  

EFIBOOT mounten  
```mount /dev/sda1 /mnt/boot```

## Installation des Betriebssystems

### pacstrap

```nano /etc/pacman.d/mirrorlist```  
Die Zeilen löschen bis ein deutscher Spiegelserver ganz oben ist  

Pacstrap durchführen (Intel CPU)  
```pacstrap /mnt base base-devel linux linux-firmware networkmanager intel-ucode zsh alacritty nano vim man tlp acpid dbus avahi grub efibootmgr openssh btrfs-progs```  

Pacstrap durchführen (AMD CPU)  
```pacstrap /mnt base base-devel linux linux-firmware networkmanager amd-ucode zsh alacritty nano vim man tlp acpid dbus avahi grub efibootmgr openssh btrfs-progs```  

### fstab

Die Optionen werden automatisch vom mount der Subvolumes übernommen  
```genfstab -Lp /mnt > /mnt/etc/fstab```

Anschließend weiter mit der Installation unter [Systemkonfiguration](https://github.com/ckord/ArchLinux/blob/master/Installation%20Arch%20Linux.md).
