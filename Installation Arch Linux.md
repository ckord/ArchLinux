# Installationsanleitung

## 1. Vorbereitungen

LAN Kabel einstecken und Start des Installationsmediums im UEFI Modus.
Mit ```passwd``` das Passwort für root setzen und mit ```systemctl start sshd``` den SSH-Server starten. Mit ```ip a``` die IP-Adresse abfragen, sollte keine IP-Adresse vorhanden sein mit ```dhcpcd``` eine Adresse vom DHCP-Server beziehen. Anschließend die Verbindung mit ```ssh root@IP-Adresse``` die herstellen.

## 2. Partitionierung des Datenträgers und Formatierung der Partitionen

Deutsches Tastaturlayout laden  
```loadkeys de```  
Datenträger abfragen  
```lsblk```  
Im folgenden Beispiel ist der zu verwendende Datenträger ```sda```  
Ich installiere mit UEFI und möchte ```grub``` als Bootmanager verwenden. Wir benötigen zwei Partitionen: ```EFIBOOT``` und ```root```.  
Partitionierungstool mit ```cfdisk``` starten

### 2.1 EFIBOOT

```EFIBOOT``` wird die erste Partition ```sda1``` am Anfang des Datenträgers ```sda```. Ich setze die Größe auf ```1GB```.

#### 2.2 root

```root``` wird die zweite Partition ```sda2```, direkt nach ```EFIBOOT```. Sie füllt den kompletten Rest des Datenträgers auf.  
Anstelle einer Swap-Partition benutzen wir ein Swapfile (Konfiguration unter 7.3).  

## 3. Anlegen der Dateisysteme, Vergabe der Label und mounten der Partitionen am jeweiligen Aufhängpunkt

### 3.1 EFIBOOT

```mkfs.fat -F 32 -n EFIBOOT /dev/sda1```

### 3.2 root

```mkfs.ext4 -L p_arch /dev/sda2```

### 1.4 Partitionen einhängen, boot Ordner erstellen

```mount -L p_arch /mnt```  
```mkdir -p /mnt/boot```  
```mount -L EFIBOOT /mnt/boot```  

## 4. Betriebssystem installieren

### 4.1 Das Basissystem installieren

```nano /etc/pacman.d/mirrorlist```  
Die Zeilen löschen bis ein deutscher Spiegelserver ganz oben ist.  

Pacstrap durchführen  
```pacstrap /mnt base base-devel linux linux-firmware dhcpcd intel-ucode zsh xterm nano vim man tlp acpid dbus avahi grub efibootmgr```  

Bei Laptops mit WLAN  
```pacstrap /mnt base base-devel linux linux-firmware dhcpcd intel-ucode zsh xterm nano vim man tlp acpid dbus avahi grub efibootmgr wpa_supplicant dialog```  

### 4.2 fstab erzeugen

```genfstab -Lp /mnt > /mnt/etc/fstab```  
Datei öffnen um auf SSD Konfig zu wechseln  
```nano /mnt/etc/fstab```  
```LABEL=p_arch / ext4 rw,defaults,noatime,discard 0 1```  

## 5. Systemkonfiguration

chroot in die Betriebssystemumgebung  
```arch-chroot /mnt/```  
DHCP aktivieren  
```systemctl enable dhcpcd```  
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
Initramfs erzeugen  
```mkinitcpio -p linux```  
Root Passwort festlegen  
```passwd```

## 6. Bootloader installieren

Installation von grub  
```pacman -S grub efibootmgr```  
EFI-Booteintrag installieren und einrichten  
```grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ArchLinux```  
Konfiguration erzeugen  
```grub-mkconfig -o /boot/grub/grub.cfg```  

## 7. Konfiguration & Benutzeranlage

### 7.1 Benutzer anlegen

Benutzer anlegen, Passwort erstellen und zu Gruppen hinzufügen  
```useradd -m -g users -s /bin/zsh christoph```  
```passwd christoph```  
```usermod -aG christoph audio,video,power,wheel```  

### 7.2 Benutzer zur Gruppe Sudo hinzufügen

Sudoers Datbei editieren  
```nano /etc/sudoers```  
Kommentarzeichen ```#``` vor der Zeile  
```%wheel ALL=(ALL) ALL``` entfernen

### 7.3 Swap-File anlegen

Ich lege ein Swapfile mit ```1GB``` Größe an.  
```fallocate -l 1G /swapfile```  
File mit Nullen überschreiben.  
```dd if=/dev/zero of=/swapfile bs=1024 count=1048576```  
Rechte ändern.  
```chmod 600 /swapfile```  
Swap aktivieren.  
```mkswap /swapfile```  
```swapon /swapfile```  

### 7.4 Sonstiges

Automatische Zeiteinstellung aktivieren  
```systemctl enable --now systemd-timesyncd.service```  
TLP aktivieren  
```tlp start```  
Dienst TLP automatisch starten lassen  
```systemctl enable tlp```  

Abschließend mit der Installation der gewünschenten DE/WM fortfahren.
