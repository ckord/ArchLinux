# Swapfile

## Swapfile unter Btrfs

Swapfiles werden unter Btrfs mit einem Linux Kernel > 5.0 unterstützt, jedoch mit Einschränkungen. Das Swapfile darf nicht auf einem Subvolume liegen, das gesnapshotted wird. Aus diesem Grund erstelle ich ein eigenes Subvolume für das Swapfile. Für Hibernation wird weitere [Konfiguration](https://wiki.archlinux.org/index.php/Power_management/Suspend_and_hibernate#Hibernation_into_swap_file_on_Btrfs) fällig. Da ich Hibernation jedoch nie benutze lasse ich diese aus.

## Swap size table

|RAM size|Swap Size (Without Hibernation)|
|:---:|:---:|
|4GB|2GB|
|8GB|3GB|
|16GB|4GB|
|24GB|5GB|
|32GB|6GB|
|64GB|8GB|

[Quelle](https://itsfoss.com/swap-size/)

## Subvolume und Mountpunkt

Das Subvolume für swap soll im Default-Subvolume erstellt werden. Dazu die Btrfs-Partition mounten und in den Mountpunkt wechseln  
```sudo mount /dev/sda2 /mnt```  
```cd /mnt/```  

Subvolume für ```swap``` erstellen  
```sudo btrfs subvolume create @swap```  

Mountpunkt verlassen und Partition unmounten
```cd ..```  
```sudo umount -R /mnt```  

Ordner für den Mountpunkt ```swap``` erstellen  
```sudo mkdir /.swap```

Subvolume ```swap``` mounten  
```sudo mount -o rw,noatime,space_cache,compress=lzo,ssd,subvol=@swap /dev/sda2 /.swap```

Subvolid abfragen und id notieren  
```sudo btrfs subvolume list -p /```  

In die fstab übernehmen  
```sudo nano /etc/fstab```  
```LABEL=p_arch /.swap btrfs rw,noatime,space_cache,subvolid=XXX,subvol=/@swap,subvol=@swap 0 0```  
(**ACHTUNG!** subvolid=XXX beachten!)

## Konfiguration

Bei Btrfs das Swapfile anlegen, das ```No_COW``` Attribut setzen und sicherstellen, dass Kompression abgeschaltet ist  

```sudo truncate -s 0 /.swap/swapfile```  
```sudo chattr +C /.swap/swapfile```  
```sudo btrfs property set /.swap/swapfile compression none```  

Ich lege ein Swapfile mit ```3GB``` Größe an  
```sudo dd if=/dev/zero of=/.swap/swapfile bs=1M count=3072 status=progress```  

Rechte ändern.  
```sudo chmod 600 /.swap/swapfile```  

Swap aktivieren.  
```sudo mkswap /.swap/swapfile```  
```sudo swapon /.swap/swapfile```

Editieren der fstab  
```sudo nano /etc/fstab```  
```/.swap/swapfile none swap defaults 0 0```
