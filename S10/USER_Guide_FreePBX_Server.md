
# User Guide : Utilisation d'un serveur VoIP avec FreePBX

## Sommaire

1. Présentation de la machine
2. Accès à la machine
    2.1. Accès local
    2.2. Accès distant
3. Utilisation principale
4. Résolution de problèmes courants

---

## 1. Présentation de la machine

- **Nom de la machine** : SRV-BOR-FreePBX  
- **Rôle** : Serveur VoIP pour assurer la communication entre les collaborateurs sur les sites de Bordeaux, Paris, et Nantes.  
- **Adresse IP** : `10.15.6.20`  
- **Réseau associé** : DMZ (10.15.6.0/24).  

---

## 2. Accès à la machine

### 2.1. Accès local

1. Accéder à la console via l'interface web de Proxmox.  
2. Se connecter avec les identifiants correspondants aux profils autorisés.  
3. Utiliser les commandes standard pour vérifier l'état des services VoIP :  
   ```bash
   systemctl status asterisk
   systemctl status apache2
   ```

### 2.2. Accès distant

1. Utiliser un client SSH pour se connecter à distance :  
   ```bash
   ssh <profil_autorisé>é@10.15.6.20
   ```  
2. Entrer le mot de passe du profil autorisé pour accéder à la machine.  
3. Si un problème d’accès survient, vérifier les règles de pare-feu sur la machine :  
   ```bash
   ufw status
   ```

---

## 3. Utilisation principale

- **Accéder à l’interface FreePBX** :  
  1. Ouvrir un navigateur web.  
  2. Entrer l’adresse suivante dans la barre d’URL : `http://10.15.6.20/admin`.  
  3. Se connecter avec les identifiants administratifs configurés pour accéder à l'administration du serveur FreePBX.  

- **Configurer des extensions SIP** :  
  1. Accéder à l’onglet **Applications > Extensions**.  
  2. Cliquer sur **Add New Extension** et suivre les étapes.  
  3. Attribuer un numéro et sauvegarder les modifications.

- **Redémarrer les services principaux en cas de besoin** :  
   ```bash
   systemctl restart asterisk
   systemctl restart apache2
   ```

---

## 4. Résolution de problèmes courants

### Problème : Impossible d'accéder à l'interface web
- **Solution** :  
  1. Vérifier si le service Apache est actif :  
     ```bash
     systemctl status apache2
     ```  
  2. Redémarrer le service si nécessaire :  
     ```bash
     systemctl restart apache2
     ```  
  3. S’assurer que le port 80 est ouvert dans le pare-feu :  
     ```bash
     ufw allow 80
     ufw reload
     ```

### Problème : Aucun son lors des appels VoIP
- **Solution** :  
  1. Vérifier la configuration des codecs dans FreePBX (onglet **Settings > Asterisk SIP Settings**).  
  2. S’assurer que les ports RTP sont ouverts dans le pare-feu :  
     ```bash
     ufw allow 10000:20000/udp
     ufw reload
     ```

### Problème : Service Asterisk arrêté
- **Solution** :  
  1. Vérifier le statut du service :  
     ```bash
     systemctl status asterisk
     ```  
  2. Redémarrer le service si nécessaire :  
     ```bash
     systemctl restart asterisk
     ```  
  3. Consulter les logs pour diagnostiquer :  
     ```bash
     tail -f /var/log/asterisk/full
     ```

---
