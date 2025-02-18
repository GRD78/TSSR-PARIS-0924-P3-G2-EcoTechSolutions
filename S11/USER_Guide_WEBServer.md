# User Guide : Utilisation d'un serveur Web Apache2

## Sommaire

1. Présentation de la machine
2. Accès à la machine
    2.1. Accès local
    2.2. Accès distant
3. Utilisation principale
4. Résolution de problèmes courants

---

## 1. Présentation de la machine

- **Nom de la machine** : SRV-BOR-WEB  
- **Rôle** : Serveur Web pour héberger les sites internes accessibles depuis les sites de Bordeaux, Paris, et Nantes.  
- **Adresse IP** : `10.15.6.10`  
- **Réseau associé** : DMZ (10.15.6.0/24).  

---

## 2. Accès à la machine

### 2.1. Accès local

1. Accéder à la console via l'interface web de Proxmox.  
2. Se connecter avec les identifiants correspondants aux profils autorisés.  
3. Utiliser les commandes standard pour vérifier l'état du service Apache2 :  
   ```bash
   systemctl status apache2
   ```

### 2.2. Accès distant

- Utiliser un client SSH pour se connecter à distance :

   ```bash
   ssh <profil_autorisé>@10.15.6.10
   ```

- Entrer le mot de passe du profil autorisé pour accéder à la machine.
- Si un problème d’accès survient, vérifier les règles de pare-feu sur la machine :

  ```bash
    ufw status
  ```

## 3. Utilisation principale

- Accéder à l’interface web du serveur :
  1. Ouvrir un navigateur web.
  2. Entrer l’adresse suivante dans la barre d’URL : http://10.15.6.10.

- Vérifier que le service Apache2 fonctionne correctement :
  1. Accéder à la machine via SSH.
  2. Vérifier le statut d'Apache2 avec la commande suivante :

   ```bash
    systemctl status apache2
   ```

  3. Redémarrer le service Apache2 si nécessaire :

   ```bash
    systemctl restart apache2
   ```
   

## 4. Résolution de problèmes courants

- **Impossible d'accéder à l'interface web** 
- Solution : Vérifier si le service Apache2 est actif :

   ```bash
   systemctl status apache2
   ```

- Redémarrer le service si nécessaire :
   ```bash
   systemctl restart apache2
   ```

- S’assurer que le port 80 (et 443 pour HTTPS) est ouvert dans le pare-feu :

   ```bash
   ufw allow 80
   ufw allow 443
   ufw reload
   ``` 

- **Problème : Page web non chargée ou erreur 500**
- Solution : Vérifier les logs d'Apache pour identifier l'erreur :
   ```bash
   tail -f /var/log/apache2/error.log
   ```

- Vérifier les permissions du répertoire /var/www/html :

   ```bash
   chmod -R 755 /var/www/html
   ``` 

- **Problème : Problème de configuration HTTPS**
- Solution : Vérifier les chemins des fichiers de certificat dans la configuration d'Apache : /etc/apache2/sites-available/default-ssl.conf
- Redémarrer Apache après modification des fichiers de configuration :

   ```bash
   systemctl restart apache2
   ``` 

- **Problème : Le site est lent**
- Solution : Vérifier l'utilisation des ressources du serveur (CPU, RAM) avec la commande top ou htop ; Vérifier si d'autres services sur le serveur utilisent des ressources excessives.
