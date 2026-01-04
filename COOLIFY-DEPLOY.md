# 🚀 Déploiement BillionMail sur Coolify

## Prérequis

### 1. Serveur dédié ou VPS
- **RAM minimum** : 4 GB (8 GB recommandé)
- **Stockage** : 20 GB minimum
- **OS** : Ubuntu 22.04+ ou Debian 12+

### 2. Ports réseau disponibles
Les ports suivants doivent être **ouverts** sur votre firewall :

| Port | Service | Obligatoire |
|------|---------|-------------|
| 25 | SMTP | ✅ Oui |
| 465 | SMTPS | ✅ Oui |
| 587 | Submission | ✅ Oui |
| 143 | IMAP | ✅ Oui |
| 993 | IMAPS | ✅ Oui |
| 110 | POP3 | ⚠️ Optionnel |
| 995 | POP3S | ⚠️ Optionnel |
| 80 | HTTP | ✅ Oui |
| 443 | HTTPS | ✅ Oui |

### 3. DNS Configuration
Avant le déploiement, configurez ces enregistrements DNS :

```
# Enregistrement A
mail.votredomaine.com    A       VOTRE_IP_SERVEUR

# Enregistrement MX
votredomaine.com         MX 10   mail.votredomaine.com

# SPF (TXT record)
votredomaine.com         TXT     "v=spf1 mx a:mail.votredomaine.com ~all"

# DMARC (TXT record)  
_dmarc.votredomaine.com  TXT     "v=DMARC1; p=quarantine; rua=mailto:postmaster@votredomaine.com"
```

> ⚠️ Le DKIM sera généré automatiquement après l'installation

---

## 📋 Étapes de déploiement sur Coolify

### Étape 1 : Créer un nouveau projet

1. Connectez-vous à votre dashboard Coolify
2. Cliquez sur **"+ New Resource"**
3. Sélectionnez **"Docker Compose"**

### Étape 2 : Configurer la source

**Option A - Depuis Git :**
- Repository URL : `https://github.com/aaPanel/BillionMail`
- Branch : `main`

**Option B - Depuis votre fork :**
- Utilisez l'URL de votre repository

### Étape 3 : Configurer les variables d'environnement

Dans Coolify, allez dans **"Environment Variables"** et ajoutez :

```env
# OBLIGATOIRE - Modifiez ces valeurs
BILLIONMAIL_HOSTNAME=mail.votredomaine.com
ADMIN_USERNAME=admin
ADMIN_PASSWORD=VotreMotDePasseFort123!
SafePath=votre-chemin-secret

# Base de données
DBNAME=billionmail
DBUSER=billionmail
DBPASS=MotDePasseDB32Caracteres!

# Redis
REDISPASS=MotDePasseRedis32Caracteres!

# Ports
SMTP_PORT=25
SMTPS_PORT=465
SUBMISSION_PORT=587
IMAP_PORT=143
IMAPS_PORT=993
POP_PORT=110
POPS_PORT=995
HTTP_PORT=80
HTTPS_PORT=443
REDIS_PORT=127.0.0.1:26379
SQL_PORT=127.0.0.1:25432

# Autres
TZ=Europe/Paris
IPV4_NETWORK=172.66.1
FAIL2BAN_INIT=y
IP_WHITELIST_ENABLE=false
RETENTION_DAYS=7
```

### Étape 4 : Configurer les volumes persistants

Dans Coolify, configurez les volumes suivants :

| Volume Local | Description |
|--------------|-------------|
| `./postgresql-data` | Données PostgreSQL |
| `./redis-data` | Données Redis |
| `./vmail-data` | Emails des utilisateurs |
| `./ssl` | Certificats SSL |
| `./ssl-self-signed` | Certificats auto-signés |
| `./logs` | Logs des services |
| `./conf` | Fichiers de configuration |
| `./core-data` | Données de l'application |

### Étape 5 : Configuration réseau Coolify

⚠️ **Important** : Dans les paramètres réseau de Coolify :

1. Désactivez le **proxy Coolify** pour ce projet (les ports mail ne passent pas par un reverse proxy HTTP)
2. Ou configurez Coolify pour exposer directement les ports sur l'hôte

### Étape 6 : Déployer

1. Cliquez sur **"Deploy"**
2. Attendez que tous les conteneurs soient en état "Running"
3. Vérifiez les logs pour les erreurs éventuelles

---

## 🔐 Premier accès

Une fois déployé, accédez à l'interface d'administration :

```
https://VOTRE_IP/SafePath
```

- **Username** : Valeur de `ADMIN_USERNAME`
- **Password** : Valeur de `ADMIN_PASSWORD`

---

## ✅ Vérification post-installation

### 1. Tester la connectivité SMTP

```bash
telnet mail.votredomaine.com 25
```

### 2. Ajouter un domaine d'envoi

1. Dans le dashboard BillionMail, allez dans **Domains**
2. Ajoutez votre domaine
3. Copiez les enregistrements DNS (DKIM) fournis
4. Ajoutez-les à votre zone DNS

### 3. Vérifier le score email

Utilisez [mail-tester.com](https://www.mail-tester.com) pour vérifier votre configuration.

---

## 🔧 Commandes utiles

Si vous avez accès SSH au serveur Coolify :

```bash
# Voir les logs
docker logs billionmail-core-billionmail-1

# Vérifier l'état des conteneurs
docker ps | grep billionmail

# Afficher les enregistrements DNS requis
docker exec billionmail-core-billionmail-1 bm show-record
```

---

## ⚠️ Notes importantes

1. **Ne pas utiliser derrière Cloudflare Proxy** pour les ports mail (25, 465, 587, 143, 993)
2. **Backup régulier** des volumes `postgresql-data` et `vmail-data`
3. **Port 25** : Certains hébergeurs cloud bloquent ce port - vérifiez avec votre provider

---

## 🆘 Dépannage

| Problème | Solution |
|----------|----------|
| Port 25 bloqué | Contactez votre hébergeur pour débloquer le port SMTP sortant |
| Certificat SSL invalide | Ajoutez votre domaine dans BillionMail et activez Let's Encrypt |
| Emails en spam | Vérifiez SPF, DKIM et DMARC dans votre zone DNS |
| Connexion IMAP échoue | Vérifiez que les ports 143/993 sont ouverts |
