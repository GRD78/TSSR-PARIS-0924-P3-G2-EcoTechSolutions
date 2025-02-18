# Documentation Administrateur : Installation d'un serveur Web avec Apache2

## 1. Présentation générale dans l'infra Ecotechsolutions
- **Nom de la machine** : SRV-BOR-WEB  
- **Rôle** : Serveur Web pour héberger les sites internes accessibles depuis les sites de Bordeaux, Paris, et Nantes.  
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
  - Serveur web Apache2.
  - Modules nécessaires pour Apache2 : `mod_ssl` pour HTTPS.
  - Firewall : `ufw` pour sécuriser les accès.
  - **Autres dépendances** :
    - Network tools : `net-tools`, `curl`, `wget`.
    - Gestionnaire de bases de données si nécessaire (ex. : MariaDB pour des sites dynamiques).
    - PHP (dans le cas de sites utilisant des scripts dynamiques).

---

## 3. Étapes d'installation

### 3.1. Création de la VM sous Proxmox
1. Se connecter à l'interface web de Proxmox.  
2. Créer une nouvelle VM en cliquant sur **Create VM**.  
3. Paramètres à configurer :
   - **General** : 
     - Node : (choisir le nœud Proxmox approprié).  
     - Name : `SRV-BOR-WEB`.
   - **OS** :
     - Utiliser l'ISO de Debian 12 préalablement uploadée.
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
   - Configurer un **hostname** : `SRV-BOR-WEB`.   
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
         address 10.15.6.10
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
### 5.1. Installation du serveur Apache2

  - **Mise à jour des dépôts et installation des packages** :
    ```bash
    apt update
    apt install apache2 -y
    ```
    
  - **Activation des modules essentiels** :
    ```bash
    a2enmod ssl rewrite
    systemctl restart apache2
    ``` 

  - **Configuration des sites virtuels** :

    - Fichier à éditer : /etc/apache2/sites-available/000-default.conf.

    ```bash
     <VirtualHost *:80>
        ServerAdmin webmaster@ecotechsolutions.lan
        DocumentRoot /var/www/html
        ServerName www.ecotechsolutions.lan
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
     </VirtualHost>
    ``` 

### 5.2. Configuration HTTPS

  - **Génération d'un certificat auto-signé** :
    
    ```bash
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
    ```
    
  - **Configuration de Apache pour HTTPS** :

    - Fichier à éditer : /etc/apache2/sites-available/default-ssl.conf.

    ```bash
     <VirtualHost *:443>
        ServerAdmin webmaster@ecotechsolutions.lan
        DocumentRoot /var/www/html
        ServerName www.ecotechsolutions.lan
        SSLEngine on
        SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
        SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
     </VirtualHost>
     ``` 

  - **Activation du site HTTPS** :

    ```bash
    a2ensite default-ssl
    systemctl reload apache2
    ``` 

## 6. Tests et vérifications
### 6.1. Connectivité réseau

  - **Vérification de la connectivité avec la passerelle DMZ** :

ping 10.15.6.254

Vérification avec un autre serveur de l'infrastructure (ex. 10.15.8.1) :

    ```bash
    ping 10.15.8.1
    ```

### 6.2. Vérification du service web

  - **Accès au site web via IP** :

    ```bash
    http://10.15.6.10
    ```

- **Vérification du bon fonctionnement d’Apache** :

    ```bash
    systemctl status apache2
    ``` 

## 7. Difficultés rencontrées

- **Problème d’accès au site web** : 

    - Solution : Vérifier les permissions du dossier /var/www/html :

    ```bash
    chmod -R 755 /var/www/html
    ```

    - Vérifier les logs d’Apache pour diagnostiquer les erreurs :

    ```bash
    tail -f /var/log/apache2/error.log
    ```

- **Problème de configuration HTTPS** : 

    - Solution : Vérifier les chemins des fichiers de certificat dans /etc/apache2/sites-available/default-ssl.conf.
