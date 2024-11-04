### Étape 1 : Préparer les Adresses IP
- **Serveur Web IP** : `192.168.139.27`
- **Serveur DNS IP** : `192.168.139.4`

---

### Étape 2 : Configurer le Serveur DNS (Bind9)
Sur le serveur DNS, assurez-vous que Bind9 est installé. Si nécessaire, installez-le avec :

```bash
sudo apt update
sudo apt install bind9 -y
```

#### 2.1. Créer un Fichier de Zone pour le Domaine
1. Créez un dossier pour les zones si ce n’est pas déjà fait, puis créez un fichier de zone pour votre domaine `cat.doranco` :

   ```bash
   sudo mkdir -p /etc/bind/zones
   sudo nano /etc/bind/zones/db.cat.doranco
   ```

2. Ajoutez la configuration suivante dans le fichier `db.cat.doranco`. Cela configure le domaine `cat.doranco` et `www.cat.doranco` pour pointer vers l’adresse IP de votre serveur web (`192.168.139.27`).

   ```plaintext
   ; Zone file for cat.doranco
   $TTL    604800
   @       IN      SOA     cat.doranco. root.cat.doranco. (
                             2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
   ;
   @       IN      NS      cat.doranco.
   @       IN      A       192.168.139.27     ; IP du serveur web
   www     IN      A       192.168.139.27     ; IP du serveur web
   ```

   Assurez-vous que `192.168.139.27` est l’IP correcte de votre serveur web.

#### 2.2. Ajouter la Zone au Fichier de Configuration DNS
1. Ouvrez le fichier de configuration local de Bind9 pour ajouter la zone (`/etc/bind/named.conf.local`).

   ```bash
   sudo nano /etc/bind/named.conf.local
   ```

2. Ajoutez la configuration suivante pour inclure la zone `cat.doranco` :

   ```plaintext
   zone "cat.doranco" {
       type master;
       file "/etc/bind/zones/db.cat.doranco";
   };
   ```

---

### Étape 3 : Redémarrer le Serveur DNS
Pour appliquer les modifications, redémarrez le service Bind9 :

```bash
sudo systemctl restart bind9
```

---

### Étape 4 : Configurer la Machine Hôte pour Utiliser le Serveur DNS
1. Sur la machine hôte (votre ordinateur principal), vous devez configurer les paramètres DNS pour utiliser votre serveur DNS.
2. Modifiez le fichier `/etc/resolv.conf` pour ajouter l’adresse IP de votre serveur DNS :

   ```plaintext
   nameserver 192.168.139.4
   ```

   Remplacez `192.168.139.4` par l’adresse IP de votre serveur DNS.

   > **Remarque :** Cette configuration est temporaire et peut être écrasée au redémarrage. Pour une configuration DNS permanente, modifiez les paramètres réseau de votre connexion pour spécifier le serveur DNS `192.168.139.4`.

---

### Étape 5 : Tester la Configuration
Sur la machine hôte, ouvrez un navigateur ou utilisez la commande `ping` pour tester la résolution de nom de domaine. Par exemple :

```bash
ping cat.doranco
```

Ou dans un navigateur, tapez :

```plaintext
http://cat.doranco
```

