# AppArmor und Firejail für Arch Linux

## Installation

Installation von AppArmor und Firejail  

`sudo pacman -S apparmor firejail`

## Aktivierung AppArmor

Dienst automatisch starten  

`sudo systemctl enable apparmor.service`

Um AppArmor als default security module bei jedem boot zu laden GRUB konfigurieren  

`sudo nano /etc/default/grub`  

Die folgende Zeile einfügen  

`GRUB_CMDLINE_LINUX_DEFAULT="apparmor=1 lsm=lockdown,yama,apparmor"`  

Anschließend die Konfiguration neu erstellen  

`sudo grub-mkconfig -o /boot/grub/grub.cfg`

Das System neu starten. Mit `sudo aa-enabled` kann nach dem Neustart geprüft werden, ob AppArmor korrekt gestartet wurde.

## Konfiguration Firejail

Firejail Integration in AppArmor  

```
su -
apparmor_parser -r /etc/apparmor.d/firejail-default
exit
```

Um jedes Programm, das ein Firejail Profil hat, automatisch mit Firejail starten  

`sudo firecfg`

Prüfung, ob ein Programm Firejail nutzt  

`sudo firejail --list`

## Ausnahmen

Bei der automatischen Benutzung Firejails wird ein symbolischer Link für jedes Programm, das ein Firejail-Profil hat, in `/usr/local/bin` erzeugt. Wird dieser Link gelöscht, dann wird das Programm nicht standardmäßig mit Firejail gestartet. 

`sudo rm /usr/local/bin/code`