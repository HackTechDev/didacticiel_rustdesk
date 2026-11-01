# Sécurisation du serveur RustDesk dans LXD

Pistes de sécurisation complémentaires au tutoriel `item.md`, classées par priorité.

## Déjà couvert par le tutoriel

- Seuls les ports utiles sont redirigés : 21115-21117 en TCP et 21116 en UDP.
- Le port 21114 (API Pro) et les ports 21118/21119 (client web) ne sont pas exposés.
- Les règles ufw `route` limitent le trafic redirigé au conteneur `rustdesk`.
- Le serveur HTTP proposé par le script d'installation n'est pas installé.
- La paire de clés et la base sont sauvegardées (section 14).

## Priorité haute

### 1. Vérifier que la clé est obligatoire

Si hbbs/hbbr acceptent des clients sans clé, n'importe qui peut utiliser le serveur comme relais et consommer sa bande passante. Les versions récentes de rustdesk-server exigent la clé par défaut, mais c'est à vérifier pour la version installée.

- **Test** : configurer un client avec le serveur et une clé vide ou fausse. S'il passe à « Prêt » et arrive à se connecter, la clé n'est pas imposée.
- **Contrôle** : regarder les options de lancement (paramètre `-k`) dans le service :

```bash
root@rustdesk:~# grep ExecStart /etc/systemd/system/rustdesksignal.service /etc/systemd/system/rustdeskrelay.service
```

### 2. Sécuriser le SSH de l'hôte

L'étape 1 du tutoriel montre une connexion par mot de passe.

- Passer à une authentification par clé :

```bash
utilisateur@poste:~$ ssh-keygen -t ed25519
utilisateur@poste:~$ ssh-copy-id utilisateur@serveur.example.com
```

- Une fois la connexion par clé vérifiée, désactiver les mots de passe dans `/etc/ssh/sshd_config` :

```
PasswordAuthentication no
PermitRootLogin no
```

```bash
utilisateur@serveur:~$ sudo systemctl restart ssh
```

> Garder une session SSH ouverte pendant le changement, pour ne pas perdre l'accès en cas d'erreur.

- Option : installer fail2ban (`sudo apt install fail2ban`), qui bloque les IP après plusieurs échecs de connexion.

### 3. Protéger l'accès aux postes clients

Le serveur ne fait que mettre les postes en relation. C'est le mot de passe du poste qui protège réellement l'accès. Dans **Paramètres > Sécurité** du client :

- préférer le mot de passe à usage unique, ou un mot de passe permanent long ;
- activer la double authentification (2FA), si la version du client la propose ;
- sur les postes sensibles, n'accepter les connexions qu'après un clic de confirmation.

## Priorité moyenne

### 4. Pare-feu de l'hôte en refus par défaut

N'autoriser que SSH et le trafic RustDesk redirigé :

```bash
utilisateur@serveur:~$ sudo ufw default deny incoming
utilisateur@serveur:~$ sudo ufw allow OpenSSH
utilisateur@serveur:~$ sudo ufw route allow proto tcp to 10.10.10.50 port 21115:21117
utilisateur@serveur:~$ sudo ufw route allow proto udp to 10.10.10.50 port 21116
utilisateur@serveur:~$ sudo ufw enable
```

> Autoriser SSH **avant** `ufw enable`, sinon la connexion en cours est coupée. Selon la configuration, il peut aussi falloir `sudo ufw allow in on lxdbr0` pour que le DHCP et le DNS de LXD fonctionnent dans le conteneur.

### 5. Mises à jour automatiques de sécurité

Sur l'hôte (Ubuntu) **et** dans le conteneur (Debian) :

```bash
apt install -y unattended-upgrades
dpkg-reconfigure -plow unattended-upgrades
```

### 6. Vérifier que le conteneur n'est pas privilégié

```bash
utilisateur@serveur:~$ lxc config get rustdesk security.privileged
```

La valeur doit être vide ou `false`. C'est le cas par défaut. Un conteneur privilégié donnerait les droits root de l'hôte en cas d'évasion.

## Priorité basse

### 7. Lire les scripts avant de les lancer

`install.sh` et `update.sh` viennent d'un dépôt tiers et s'exécutent en root. Les parcourir avant de les lancer est une bonne habitude :

```bash
root@rustdesk:~# less install.sh
```

### 8. Surveiller les journaux

Consulter de temps en temps les journaux, pour repérer un usage anormal (beaucoup de connexions inconnues) :

```bash
root@rustdesk:~# journalctl -u rustdesksignal -u rustdeskrelay --since today
```
