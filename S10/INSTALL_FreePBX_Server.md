
# Documentation Administrateur : Installation d'un serveur VoIP avec FreePBX

## 1. Présentation générale dans l'infra Ecotechsolutions
- **Nom de la machine** : SRV-BOR-FreePBX  
- **Rôle** : Serveur VoIP pour assurer la communication entre les collaborateurs sur les sites de Bordeaux, Paris, et Nantes.  
- **Réseau associé** : DMZ (10.15.6.0/24).  
- **Emplacement** : Situé dans la zone DMZ pour une exposition sécurisée.  

---

## 2. Prérequis
- **Configuration matérielle minimale** :
  - 1 CPU, 2 cœurs.
  - RAM : 2 Go.
  - Disque : 32 Go.  
- **ISO ou image système nécessaire** :
  - **Linux Debian 12** (téléchargeable depuis le site officiel Debian).  
- **Packages ou fichiers supplémentaires** :
  - FreePBX (interface de gestion pour Asterisk).  
  - Asterisk (logiciel de communication VoIP).
  - LAMP stack (Apache, MySQL/MariaDB, PHP) pour FreePBX.
  - **Autres dépendances** :
    - Network tools : `net-tools`, `curl`, `wget`.
    - Outils complémentaires pour VoIP : `ufw` (pare-feu) et `fail2ban` pour la sécurité.

---

## 3. Étapes d'installation

### 3.1. Création de la VM sous Proxmox
1. Se connecter à l'interface web de Proxmox.  
2. Créer une nouvelle VM en cliquant sur **Create VM**.  
3. Paramètres à configurer :
   - **General** : 
     - Node : (choisir le nœud Proxmox approprié).  
     - Name : `SRV-BOR-FreePBX`.
   - **OS** :
     - Utilisez l'ISO de Debian 12 préalablement uploadée.
   - **System** :
     - BIOS : **UEFI**.
     - Machine type : **q35**.
   - **Hard Disk** :
     - Size : 32 Go.
     - Format : qcow2.
   - **CPU** :
     - Sockets : 1.
     - Cores : 2.
   - **Memory** :
     - RAM : 2 Go.
   - **Network** :
     - Bridge : vmbr570 (interface réseau associée à la DMZ).  
4. Finaliser la création de la VM et démarrer la machine.  

### 3.2. Installation du système d’exploitation
1. Démarrer la VM avec l’ISO Debian 12.  
2. Suivre les étapes d’installation de Debian :  
   - Choisir la langue et clavier (France).  
   - Configurer un **hostname** : `SRV-BOR-FreePBX`.   
   - Installer les paquets de base (inclure SSH).  
3. Finaliser l’installation et redémarrer la machine.  

---

## 4. Configuration réseau
### 4.1. Modification de la configuration réseau :
   - Fichier à éditer : `/etc/network/interfaces`.
   - Configuration :
     ```bash
     auto ens18
     iface ens18 inet static
         address 10.15.6.20
         netmask 255.255.255.0
         gateway 10.15.6.254
         dns-nameservers 10.15.8.11
     ```
### 4.2. Redémarrage du réseau :  
   ```bash
   systemctl restart networking
   ```

---

## 5. Installation des services/applications

### 5.1. Installation de la pile LAMP
1. **Installation des packages** :
   ```bash
   apt update
   apt install apache2 mariadb-server php php-mysql -y
   ```
2. **Configurer MariaDB** :
   - Sécuriser l’installation avec `mysql_secure_installation`.
   - Créer une base de données pour FreePBX :
     ```sql
     CREATE DATABASE freepbx;
     GRANT ALL PRIVILEGES ON freepbx.* TO 'freepbxuser'@'localhost' IDENTIFIED BY 'password';
     FLUSH PRIVILEGES;
     ```

### 5.2. Installation de FreePBX et Asterisk
1. **Installation des dépendances** :
   ```bash
   apt install wget build-essential linux-headers-$(uname -r) -y
   ```
2. **Téléchargement et installation de Asterisk** :
   ```bash
   cd /usr/src
   wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-20-current.tar.gz
   tar -xvf asterisk-20-current.tar.gz
   cd asterisk-20.*
   ./configure
   make && make install
   make samples
   make config
   ```
3. **Installation de FreePBX** :
   ```bash
   cd /usr/src
   git clone https://github.com/FreePBX/freepbx.git
   cd freepbx
   ./start_asterisk start
   ./install -n
   ```

---

## 6. Tests et vérifications

### 6.1. Connectivité réseau
- **Vérification de la connectivité avec la passerelle DMZ** :
  ```bash
  ping 10.15.6.254
  ```
- **Vérification avec un autre serveur de l'infrastructure (ex. 10.15.8.1)** :
  ```bash
  ping 10.15.8.1
  ```

### 6.2. Vérification des services
- Accéder à l’interface web de FreePBX via :
  ```url
  http://10.15.6.20/admin
  ```
- Vérifier que les modules Asterisk et FreePBX fonctionnent correctement :
  ```bash
  asterisk -r
  ```

### 6.3. Test d’appel VoIP
- Configurer un téléphone SIP pour tester une communication entre deux extensions.

---

## 7. Difficultés rencontrées

### 7.1. Problème de connectivité avec la DMZ
- **Solution** : Vérifier les règles du pare-feu et la configuration réseau.

### 7.2. Erreur dans l’installation de FreePBX
- **Solution** : Consulter les logs dans `/var/log/asterisk` pour diagnostiquer les erreurs.
