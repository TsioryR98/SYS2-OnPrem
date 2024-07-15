# SYS2-OnPrem
# Création d'un Sous-Réseau Privé sur Arch Linux

Ce guide détaille les étapes pour créer un sous-réseau privé sur Arch Linux en utilisant votre machine comme passerelle, permettant aux hôtes de se connecter à un autre réseau via votre machine, sans utiliser DHCP.

## Création d'un Sous-Réseau Privé avec iproute2 et iptables

### Étape 1: Installer les paquets nécessaires
Installez les paquets nécessaires :

```bash
sudo pacman -S iproute2 iptables hostapd
```
### Étape 2: Configurer hostapd pour le point d'accès
```bash
/etc/hostapd/hostapd.conf
```

```bash
interface=wlan0
driver=nl80211
ssid=MyAccessPoint
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=MySecurePassphrase
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

### Étape 3: Configurer l'interface réseau pour le sous-réseau privé
Attribuez une adresse IP statique à l'interface Wi-Fi :

```bash
sudo ip addr add 192.168.1.1/24 dev wlan0
sudo ip link set wlan0 up
```
### Étape 4: Activer le forwarding IP
Activez le forwarding IP pour permettre le routage entre les interfaces :

```bash
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
```

### Étape 5: Configurer iptables pour le NAT
Configurez iptables pour masquer les adresses IP du sous-réseau privé et permettre la communication avec l'extérieur :

```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

Enregistrez les règles iptables pour qu'elles persistent après redémarrage :

```bash
sudo iptables-save | sudo tee /etc/iptables/iptables.rules
sudo systemctl enable iptables
sudo systemctl start iptables
```

### Étape 6: Démarrer les services

```bash
sudo systemctl enable hostapd
sudo systemctl start hostapd
```

## Création d'un sous-réseau privé avec systemd-networkd

### Étape 1: Installer systemd-networkd

Installez ``` systemd-networkd ``` si ce n'est pas déjà fait :

```bash
sudo pacman -S systemd
```
### Étape 2: Configurer hostapd pour le point d'accès
Comme décrit ci-dessus, créez le fichier de configuration ``` /etc/hostapd/hostapd.conf ```. 

### Étape 3: Configurer les fichiers de réseau avec systemd-networkd
Créez un fichier de configuration pour l'interface Wi-Fi dans ``` /etc/systemd/network/ ```.

## /etc/systemd/network/10-wlan0.network

```bash
[Match]
Name=wlan0

[Network]
Address=192.168.1.1/24
```

### Étape 4: Configurer le forwarding IP
Créez un fichier de configuration pour activer le forwarding IP :

## /etc/sysctl.d/99-sysctl.conf

```bash
net.ipv4.ip_forward = 1
```
Rechargez les paramètres sysctl :

```bash
sudo sysctl --system
```

### Étape 5: Configurer iptables pour le NAT

```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

Enregistrez les règles iptables pour qu'elles persistent après redémarrage :

```bash
sudo iptables-save | sudo tee /etc/iptables/iptables.rules
sudo systemctl enable iptables
sudo systemctl start iptables
```
### Étape 6: Démarrer les services

Activez et démarrez les services ``` iptables ``` , ``` hostapd, et systemd-networkd ``` :

```bash
sudo systemctl enable iptables
sudo systemctl start iptables
sudo systemctl enable hostapd
sudo systemctl start hostapd
sudo systemctl enable systemd-networkd
sudo systemctl start systemd-networkd
```

## Connexion des hôtes avec des adresses IP statiques
Pour que vos hôtes se connectent au réseau sans DHCP, configurez manuellement les paramètres réseau sur chaque hôte avec les informations suivantes :

* # Adresse IP : 192.168.12.x (où x est un nombre unique entre 2 et 254)
* # Masque de sous-réseau : 255.255.255.0
* # Passerelle : 192.168.12.1
* # DNS Vous pouvez utiliser les DNS de votre fournisseur d'accès internet ou ceux de Google (8.8.8.8, 8.8.4.4)

## Connexion des hôtes avec des adresses IP statiques
Sur chaque hôte, éditez le fichier de configuration réseau ou utilisez les commandes appropriées pour attribuer une adresse IP statique. Par exemple, sous Linux :

```bash
sudo ip addr add 192.168.12.2/24 dev wlan0
sudo ip route add default via 192.168.12.1
```
Avec ces configurations, votre machine Arch Linux fonctionnera comme un point d'accès sans DHCP et une passerelle, permettant aux hôtes de se connecter à un autre réseau en passant par votre machine. Vous utiliserez une connexion Ethernet (eth0) pour accéder à Internet et une connexion Wi-Fi (wlan0) pour votre réseau privé.






