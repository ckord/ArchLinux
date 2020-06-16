# Installationsanleitung

## Inhaltsverzeichnis

* [Vorbereitungen](#Vorbereitungen)
* [Partitionierung](#Partitionierung)  
    * [EFIBOOT](#EFIBOOT)  
    * [root](#root)  
* [Dateisysteme](#Dateisysteme)  
    * [EFIBOOT](#EFIBOOT)  
    * [root](#root)
* [Installation des Betriebssystems](#Installation-des-Betriebssystems)  
    * [pacstrap](#pacstrap)  
    * [fstab](#fstab)  
* [Systemkonfiguration](#Systemkonfiguration)
* [Bootloader](#Bootloader)
* [Benutzer](#Benutzer)  
    * [Benutzer](#Benutzer)  
    * [sudo](#Benutzer-zur-Gruppe-Sudo-hinzufügen)  

## Vorbereitungen

LAN Kabel einstecken und Start des Installationsmediums im UEFI Modus.
Mit ```passwd``` das Passwort für root setzen und mit ```systemctl start sshd``` den SSH-Server starten. Mit ```ip a``` die IP-Adresse abfragen, sollte keine IP-Adresse vorhanden sein mit ```dhcpcd``` eine Adresse vom DHCP-Server beziehen. Anschließend die Verbindung mit ```ssh root@IP-Adresse``` die herstellen. Deutsches Tastaturlayout mit ```loadkeys de``` laden.  

## Partitionierung

Datenträger abfragen  
```lsblk```  
Im folgenden Beispiel ist der zu verwendende Datenträger ```sda```  
Ich installiere mit UEFI und möchte ```grub``` als Bootmanager verwenden. Wir benötigen zwei Partitionen: ```EFIBOOT``` und ```root```. Anstelle einer Swap-Partition benutzen wir ein [Swapfile](https://github.com/ckord/ArchLinux/blob/master/swapfile.md).

Partitionierungstool mit ```cfdisk``` starten

### EFIBOOT

```EFIBOOT``` wird die erste Partition ```sda1``` am Anfang des Datenträgers ```sda```. Ich setze die Größe auf ```1GB```.

### root

```root``` wird die zweite Partition ```sda2```, direkt nach ```EFIBOOT```. Sie füllt den kompletten Rest des Datenträgers auf.  

## Dateisysteme

### EFIBOOT

```mkfs.fat -F 32 -n EFIBOOT /dev/sda1```

### root

```mkfs.ext4 -L p_arch /dev/sda2```

### Partitionen einhängen, boot Ordner erstellen

```mount -L p_arch /mnt```  
```mkdir -p /mnt/boot```  
```mount -L EFIBOOT /mnt/boot```  

## Installation des Betriebssystems

### pacstrap

```nano /etc/pacman.d/mirrorlist```  
Die Zeilen löschen bis ein deutscher Spiegelserver ganz oben ist  

Pacstrap durchführen  
```pacstrap /mnt base base-devel linux linux-firmware networkmanager intel-ucode zsh alacritty nano vim man tlp acpid dbus avahi openssh```  

### fstab

```genfstab -Lp /mnt > /mnt/etc/fstab```  
Datei öffnen um auf SSD Konfig zu wechseln  

```nano /mnt/etc/fstab```  
```LABEL=p_arch / ext4 rw,defaults,noatime,discard 0 1```  

## Systemkonfiguration

chroot in die Betriebssystemumgebung  
```arch-chroot /mnt```  

DHCP aktivieren & SSH aktivieren  
```systemctl enable NetworkManager```  
```systemctl enable sshd```  

Rechnername festlegen  
```echo ArchLinux > /etc/hostname```  

Hosts Datei anpassen  
```nano /etc/hosts```  
```127.0.0.1 localhost```  
```::1 localhost```  
```127.0.1.1 ArchLinux.home ArchLinux```  

Spracheinstellungen auf Deutsch festlegen  
```nano /etc/locale.gen```  

Kommentarzeichen ```#``` entfernen  
```de_DE.UTF-8 UTF-8```  
```de_DE ISO-8859-1```  
```de_DE@euro ISO-8859-15```  
```en_US.UFT-8 UTF-8```  

Locale generieren  
```locale-gen```  

Die Tastaturbelegung und Schriftart festlegen  
```echo KEYMAP=de-latin1 > /etc/vconsole.conf```  
```echo FONT=lat9w-16 >> /etc/vconsole.conf```  

Die Zeitzone festlegen  
```ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime```  

Automatische Zeiteinstellung aktivieren  
```systemctl enable systemd-timesyncd.service```  

Initramfs erzeugen  
```mkinitcpio -p linux```  

Root Passwort festlegen  
```passwd```  

## Bootloader

Grub installieren  
```pacman -S grub efibootmgr```  
EFI-Booteintrag installieren und einrichten  
```grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ArchLinux```  
Konfiguration erzeugen  
```grub-mkconfig -o /boot/grub/grub.cfg```  

## Benutzer

### Benutzer anlegen

Benutzer anlegen, Passwort erstellen und zu Gruppen hinzufügen  
```useradd -m -g users -s /bin/zsh christoph```  
```passwd christoph```  
```usermod -aG audio,video,power,wheel christoph```  

### Benutzer zur Gruppe Sudo hinzufügen

Sudoers Datei editieren  
```nano /etc/sudoers```  
Kommentarzeichen ```#``` vor der Zeile  
```%wheel ALL=(ALL) ALL``` entfernen

## Neustart

```chroot``` mit ```exit``` verlassen.  
```umount -R /mnt```  
```reboot```

Abschließend mit der Installation der gewünschenten DE/WM [i3](https://github.com/ckord/ArchLinux/blob/master/i3.md) fortfahren.  
*Optional*
Mit der Konfiguration eines [Swapfiles unter Btrfs](https://github.com/ckord/ArchLinux/blob/master/swapfile_Btrfs.md) fortfahren.
