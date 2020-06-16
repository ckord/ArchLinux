# Swapfile

## Swap size table

|RAM size|Swap Size (Without Hibernation)|Swap Size (With Hibernation)|
|:---:|:---:|:---:|
|4GB|2GB|6GB|
|8GB|3GB|11GB|
|16GB|4GB|20GB|
|24GB|5GB|29GB|
|32GB|6GB|38GB|
|64GB|8GB|72GB|

[Quelle](https://itsfoss.com/swap-size/)

## Konfiguration

Ich lege ein Swapfile mit ```3GB``` Größe an  
```sudo dd if=/dev/zero of=/swapfile bs=1M count=3072 status=progress```  

Rechte ändern  
```sudo chmod 600 /swapfile```  

Swap aktivieren  
```sudo mkswap /swapfile```  
```sudo swapon /swapfile```

Editieren der fstab  
```sudo nano /etc/fstab```  
```/swapfile none swap defaults 0 0```
