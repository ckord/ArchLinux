# ArchLinux with Btrfs

## Partitionierung

Datenträger abfragen  
```lsblk```  
Im folgenden Beispiel ist der zu verwendende Datenträger ```sda```  
Ich installiere mit UEFI und möchte ```grub``` als Bootmanager verwenden. Wir benötigen zwei Partitionen: ```EFIBOOT``` und eine Partition mit dem gesamten restlichen freien Speicherplatz. Hier werden die Subvolumes für ```root``` und ```home``` erstellt. Ein Swapfile wird seit dem Linux Kernel 5.0 auf Btrfs unterstützt, allerdings nur mit Limitierungen.

Partitionierungstool mit ```cfdisk``` starten

### EFIBOOT

```EFIBOOT``` wird die erste Partition ```sda1``` am Anfang des Datenträgers ```sda```. Ich setze die Größe auf ```1GB```.

### Btrfs

```Btrfs``` wird die zweite Partition ```sda2```, direkt nach ```EFIBOOT```. Sie füllt den kompletten Rest des Datenträgers auf.  

## Dateisysteme

### EFIBOOT

```mkfs.fat -F 32 -n EFIBOOT /dev/sda1```

### root

```mkfs.btrfs -L p_arch /dev/sda2```

## Subvolumes und Mountpunkte

Das Default-Subvolume mounten  
```mount /dev/sda2 /mnt```  

In den Mountpunkt wechseln  
```cd /mnt```  

Subvolume für ```/``` erstellen  
```btrfs subvolume create @```  
Subvolume für ```home``` erstellen  
```btrfs subvolume create @home```  
Subvolume für ```snapshots``` erstellen  
```btrfs subvolume create @snapshots```

Subvolumes anzeigen lassen  
```btrfs subvolume list -p /mnt```  

Unmount des Default-Subvolumes 5  
```umount /mnt```  

Subvolume ```/``` mounten  
```mount -o rw,noatime,space_cache,compress=lzo,ssd,subvol=@ /dev/sda2 /mnt```  

Ordner für die Mountpunkte ```boot```, ```home``` und ```snapshots``` erstellen
```mkdir -p /mnt/boot```  
```mkdir -p /mnt/home```
```mkdir -p /mnt/.snapshots```

Subvolumes ```home``` und ```snapshots``` mounten  
```mount -o rw,noatime,space_cache,compress=lzo,ssd,subvol=@home /dev/sda2 /mnt/home```  
```mount -o rw,noatime,space_cache,compress=lzo,ssd,subvol=@snapshots /dev/sda2 /mnt/.snapshots```  

EFIBOOT mounten  
```mount /dev/sda1 /mnt/boot```

## Installation des Betriebssystems

### pacstrap

```nano /etc/pacman.d/mirrorlist```  
Die Zeilen löschen bis ein deutscher Spiegelserver ganz oben ist.  

Pacstrap durchführen  
```pacstrap /mnt base base-devel linux linux-firmware networkmanager intel-ucode zsh alacritty nano vim man tlp acpid dbus avahi grub efibootmgr openssh btrfs-progs```  

### fstab

Die Optionen werden automatisch vom mount der Subvolumes übernommen  
```genfstab -Lp /mnt > /etc/fstab```

Anschließend weiter mit der Installation unter [Systemkonfiguration](https://github.com/ckord/ArchLinux/blob/master/Installation%20Arch%20Linux.md).
