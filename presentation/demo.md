# Script de démonstration

Déroulé de la démonstration en direct, avec les commandes dans l'ordre, les points de reprise et les phrases de transition. Les valeurs sont celles du tutoriel (`203.0.113.10`, `10.10.10.50`) : les remplacer par celles de la machine de démonstration.

## Préparation (la veille)

### Machine de démonstration

- Utiliser un **hôte dédié à la démo** (VPS ou machine de test), pas le serveur de production.

> Si un serveur RustDesk de production tourne déjà sur le même hôte, il occupe les ports 21115-21117 de l'IP publique. Le conteneur de démo ne pourrait pas utiliser les mêmes.

- Télécharger l'image à l'avance, pour que `lxc launch` soit instantané :

```bash
lxc image copy images:debian/12 local: --alias debian12
```

### Conteneur de secours déjà installé

Préparer un conteneur `rustdesk` complètement installé, avec un snapshot à chaque étape. Il sert de point de reprise si quelque chose échoue en direct.

```bash
lxc launch debian12 rustdesk
lxc snapshot rustdesk etape2-cree

lxc config device override rustdesk eth0 ipv4.address=10.10.10.50
lxc restart rustdesk
lxc snapshot rustdesk etape3-ip-fixe

lxc config device add rustdesk rd-tcp proxy \
  listen=tcp:203.0.113.10:21115-21117 connect=tcp:10.10.10.50:21115-21117 nat=true
lxc config device add rustdesk rd-udp proxy \
  listen=udp:203.0.113.10:21116 connect=udp:10.10.10.50:21116 nat=true
lxc snapshot rustdesk etape5-ports

# installation complète dans le conteneur (install.sh), puis :
lxc snapshot rustdesk etape6-installe
```

Pour la démo, revenir au début :

```bash
lxc restore rustdesk etape2-cree
```

> Restaurer un snapshot restaure aussi la configuration du conteneur, y compris les proxy devices. Après `etape2-cree`, ils n'existent donc pas encore : ils seront créés en direct.

### Postes clients

- **Poste A** (le portable de présentation) : client RustDesk installé, **déjà configuré** avec le serveur de démo.
- **Poste B** (machine de démo à contrôler) : client installé et configuré, mot de passe connu.
- **Poste C** : un client **non configuré**, pour montrer l'erreur `rs-ny.rustdesk.com`.
- Prévoir le lien de téléchargement du client Android pour le volontaire.

### Matériel et réseau

- [ ] Tester la connexion **depuis la salle**, la veille si possible. Les réseaux de salle bloquent souvent l'UDP ou les ports non standard.
- [ ] Prévoir un **partage de connexion 4G** en secours.
- [ ] Enregistrer toute l'installation avec `asciinema rec demo.cast`, et filmer une connexion réussie : plan de secours si le réseau tombe.
- [ ] Générer le QR code du dépôt : `qrencode -o qr-depot.png https://github.com/HackTechDev/didacticiel_rustdesk`

### Terminal

- Police grande (au moins 20 pt), thème clair si le projecteur est faible.
- `tmux` coupé en deux : **hôte à gauche**, **conteneur à droite**.
- Un fichier `commandes.txt` ouvert à côté, pour copier-coller les commandes longues.
- Masquer les vraies valeurs : IP publique, clé, ID et mots de passe des postes.

---

## Étape 0 — Accroche (0:00-0:03)

Sur le poste A, se connecter au poste B à travers le serveur de démo (conteneur déjà installé, snapshot `etape6-installe`).

> « Je viens de prendre la main sur cette machine. La question, c'est : par où est passée la connexion ? Pas par les serveurs de RustDesk : par un serveur à moi, installé dans un conteneur. On va le construire ensemble. »

Puis :

```bash
lxc restore rustdesk etape2-cree
```

> « Je remets tout à zéro. »

## Étape 1 — Le problème (0:03-0:08)

Sur le **poste C** (non configuré), tenter une connexion vers l'ID du poste B. L'erreur `rs-ny.rustdesk.com:21116` apparaît.

> « Par défaut, le client passe par les serveurs publics de RustDesk. Ici, il ne trouve pas le poste, parce que le poste B est enregistré sur mon serveur, pas sur le leur. »

## Étape 2 — Création du conteneur (0:20)

```bash
lxc list
```

> « Le conteneur existe, avec une IP attribuée par DHCP. Problème : cette IP peut changer. »

## Étape 3 — IP fixe (0:22)

```bash
lxc config device override rustdesk eth0 ipv4.address=10.10.10.50
lxc restart rustdesk
lxc list rustdesk
```

> « IP fixe obligatoire : la règle DNAT doit savoir où envoyer les paquets. »

## Étape 4 — IP publique (0:24)

```bash
ip -4 addr show scope global
```

> « On cherche l'IP de l'interface externe. Pas `lxdbr0`, qui est le réseau privé des conteneurs. »

## Étape 5 — Proxy devices (0:25)

```bash
lxc config device add rustdesk rd-tcp proxy \
  listen=tcp:203.0.113.10:21115-21117 connect=tcp:10.10.10.50:21115-21117 nat=true
lxc config device add rustdesk rd-udp proxy \
  listen=udp:203.0.113.10:21116 connect=udp:10.10.10.50:21116 nat=true
```

**Moment clé** : montrer que rien n'écoute, mais que la règle existe.

```bash
sudo ss -tlnp | grep 2111          # rien sur l'hôte
sudo nft list ruleset | grep 2111  # la règle DNAT est là
```

> « Aucun programme n'écoute sur l'hôte. Pourtant, la redirection fonctionne : c'est le noyau qui réécrit la destination des paquets. »

> **Point de reprise** : `lxc restore rustdesk etape5-ports`

## Étape 6 — Installation (0:28-0:38)

Dans le panneau de droite :

```bash
lxc exec rustdesk -- bash
apt update && apt install -y wget sudo curl
wget https://raw.githubusercontent.com/techahold/rustdeskinstall/master/install.sh
chmod +x install.sh
./install.sh
```

Pendant que le script tourne, expliquer :

- le choix **IP ou nom de domaine** (question 1 du script) ;
- pourquoi refuser le **serveur HTTP** (question 2).

À la fin, **montrer la clé publique** (masquée si c'est une vraie clé).

> **Si le script traîne ou échoue** : `exit`, puis `lxc restore rustdesk etape6-installe`.
> « Comme dans une émission de cuisine, voici celui que j'ai préparé avant. » C'est aussi une démonstration de l'intérêt des snapshots.

## Étape 7 — Vérification (0:38-0:40)

```bash
systemctl list-units | grep -i rust
ss -tulpn | grep 2111
```

> « Dans le conteneur, cette fois, hbbs et hbbr écoutent bien. »

## Étape 8 — Configuration et volontaire (0:40-0:50)

1. Inviter un **volontaire** avec un téléphone Android ou un portable.
2. Lui faire installer le client, puis configurer **Serveur ID** et **Clé** (afficher la slide « Configuration du client »).
3. Attendre « Prêt ».
4. Depuis le poste A, se connecter à son appareil, ou l'inverse.

> Si l'installation sur son appareil est trop longue : basculer sur le poste B.

## Étape 9 — Les pièges en direct (0:50-0:53)

- **Clé fausse** : sur un client, remplacer la clé par une valeur fausse et valider. Observer le statut.

> Selon la version du serveur, le client reste « Pas prêt » ou la connexion échoue. Tester le comportement exact pendant la répétition, et adapter le commentaire.

- **Bouton Connexion** : cliquer sur le bouton **Connexion** au centre de l'onglet Accueil, pour montrer qu'il sert aux comptes de la version Pro.

## Étape 10 — Conclusion (0:55-0:60)

Afficher la slide « À retenir » avec le QR code du dépôt.

---

## Après la présentation

```bash
lxc restore rustdesk etape6-installe   # ou lxc delete --force rustdesk
```

- Retirer la configuration du serveur de démo sur l'appareil du volontaire, ou lui indiquer comment la retirer.
