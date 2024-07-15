# SYS2-OnPrem
# Création d'un Sous-Réseau Privé sur Arch Linux

Ce guide détaille les étapes pour créer un sous-réseau privé sur Arch Linux en utilisant votre machine comme passerelle, permettant aux hôtes de se connecter à un autre réseau via votre machine, sans utiliser DHCP.

## Création d'un Sous-Réseau Privé avec iproute2 et iptables

### Étape 1: Installer les paquets nécessaires
Installez les paquets nécessaires :

```bash
sudo pacman -S iproute2 iptables hostapd

### Étape 2: Configurer hostapd pour le point d'accès
Créez un fichier de configuration pour hostapd :

``bash
/etc/hostapd/hostapd.conf
