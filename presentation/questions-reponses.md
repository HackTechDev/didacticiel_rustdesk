# Fiche questions-réponses

Questions probables après la présentation, avec des réponses courtes. Les points marqués **à vérifier** dépendent des versions : les contrôler avant la présentation.

## RustDesk

### Quelle différence entre la version OSS et la version Pro ?

- **OSS** (gratuite, open source) : serveur ID et relais. On se connecte avec l'ID et le mot de passe du poste distant.
- **Pro** (payante) : ajoute une console web d'administration, des comptes utilisateurs, un carnet d'adresses partagé, l'authentification OIDC/LDAP, le générateur de clients personnalisés et l'API (port 21114).

Pour un usage personnel ou une petite équipe, la version OSS suffit.

### Les sessions sont-elles chiffrées ?

Oui. Les sessions entre deux postes sont chiffrées. La clé publique du serveur permet en plus au client de vérifier qu'il parle bien à **ce** serveur, et pas à un imposteur.

### Combien de bande passante consomme le serveur ?

Très peu tant que les postes se connectent **directement** : le serveur ne fait que les mettre en relation. La bande passante n'est consommée que lorsque le **relais** est utilisé, par exemple si les deux postes sont derrière des pare-feu stricts. Elle dépend alors de la résolution et de l'activité à l'écran.

### Est-ce que ça marche sous Wayland ?

En partie. RustDesk fonctionne mieux sous X11. Sous Wayland, certaines fonctions sont limitées, notamment l'accès sans surveillance. **À vérifier** avec la version du client utilisée.

### Peut-on utiliser RustDesk depuis un navigateur ?

Le client web utilise les ports WebSocket 21118/21119. Il faut alors les placer derrière un reverse proxy en HTTPS. La configuration documentée par RustDesk concerne surtout la version Pro : **à tester** avant de compter dessus avec la version OSS. Le tutoriel ne l'utilise pas.

### Combien ça coûte ?

Le serveur OSS est gratuit. Il faut seulement une machine avec une IP publique : un petit VPS suffit, puisque le serveur consomme peu.

### Les clients doivent-ils tous être configurés ?

Oui, les deux : celui qui contrôle **et** celui qui est contrôlé. Si un seul est configuré, ils ne sont pas sur le même serveur et ne se trouvent pas.

### Et si on perd la clé du serveur ?

Le serveur en génère une nouvelle, et il faut reconfigurer **tous** les clients. D'où l'importance de sauvegarder `/opt/rustdesk/id_ed25519` (section Sauvegarde du tutoriel).

### L'ID d'un poste change-t-il ?

Il peut changer lors du passage au serveur auto-hébergé. Toujours utiliser l'ID affiché **après** la configuration.

## LXD et réseau

### Pourquoi LXD plutôt que Docker ?

- Un conteneur LXD est une Debian complète : le script d'installation fonctionne tel quel, et on l'administre comme un serveur classique (systemd, apt).
- Docker est très bien aussi : RustDesk fournit d'ailleurs une image officielle. C'est surtout une question d'habitude et d'outillage existant.

### Pourquoi pas une machine virtuelle ?

Une VM isole mieux, puisqu'elle a son propre noyau. Mais elle consomme plus de mémoire et démarre plus lentement. Pour un service aussi léger, un conteneur suffit.

### C'est quoi la différence entre LXD et Incus ?

Incus est un fork communautaire de LXD, créé en 2023 après le changement de gouvernance de LXD par Canonical. Il s'utilise de la même façon : les commandes `lxc` deviennent `incus`.

### Pourquoi `nat=true` ?

Sans `nat=true`, un processus de LXD (`forkproxy`) relaie les connexions : le conteneur voit alors l'IP de l'hôte au lieu de celle du client. hbbs ne peut plus établir de connexions directes, et tout passe par le relais.

### Pourquoi une IP fixe pour le conteneur ?

Le mode `nat=true` crée une règle DNAT vers une adresse précise. Si l'IP du conteneur changeait, la règle pointerait dans le vide.

### Pourquoi `ufw route allow` et pas `ufw allow` ?

Le trafic redirigé traverse l'hôte pour aller vers le conteneur : il passe par la chaîne *FORWARD*. `ufw allow` ne règle que la chaîne *INPUT*, qui concerne le trafic destiné à l'hôte lui-même.

### Et IPv6 ?

Le tutoriel ne traite que l'IPv4. Il faudrait ajouter des proxy devices pour l'IPv6 de l'hôte.

### Que se passe-t-il si l'IP publique de l'hôte change ?

La règle DNAT ne correspond plus. Il faut mettre à jour l'adresse `listen` des proxy devices et, si les clients utilisent un nom de domaine, l'enregistrement DNS. Avec une IP dans les clients, il faut aussi les reconfigurer.

## Sécurité

### N'importe qui peut-il utiliser mon serveur ?

Pas si la clé est obligatoire : un client sans la bonne clé est refusé. Le tester avec une clé fausse (section Sécurisation du tutoriel).

### Le script d'installation est-il sûr ?

C'est un script tiers (`techahold/rustdeskinstall`) exécuté en root. Bonne habitude : le lire avant de le lancer. Alternatives officielles : les paquets `.deb` de `rustdesk-server` ou l'image Docker.

### Comment éviter qu'on prenne la main sur mon poste ?

C'est le mot de passe du **poste** qui protège l'accès, pas le serveur. Préférer le mot de passe à usage unique, activer la double authentification si elle est disponible, et n'accepter les connexions qu'après un clic de confirmation sur les postes sensibles.

## Usage en entreprise

### Peut-on l'utiliser en entreprise ?

Oui, la version OSS est sous licence AGPL. Pour gérer de nombreux postes et utilisateurs (comptes, carnet d'adresses, déploiement de clients préconfigurés), la version Pro devient plus pratique.

### Peut-on déployer des clients déjà configurés ?

Avec la version OSS, il est possible d'importer la chaîne de configuration générée par le script dans chaque client. Le générateur de clients personnalisés est réservé à la version Pro.
