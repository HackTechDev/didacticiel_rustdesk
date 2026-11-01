# Améliorations du tutoriel RustDesk / LXD

Pistes d'amélioration pour `item.md` (« Installer un serveur RustDesk dans un conteneur LXD »).

## À faire

### Front matter (Grav)

- [ ] **Compléter `og:image`** : l'URL s'arrête au dossier `02.admin-sys/`, sans nom d'image. L'aperçu sur les réseaux sociaux ne fonctionne pas.
- [ ] **Supprimer le bloc `content:`** (`items`, `limit`, `order`, `pagination`, `url_taxonomy_filters`) : il sert aux pages de liste de blog et n'a pas d'effet sur un article seul.
- [ ] **Vérifier la présence des images** dans le dossier de la page : `screenshot.png` (image d'en-tête) et `rustdesk01.png` à `rustdesk08.png`.
- [ ] **Ajouter des tags** : `LXD`, `Debian`, `Auto-hébergement`, `Bureau à distance`.

### Contenu à vérifier ou sourcer

- [ ] **Ports 21118 / 21119** : l'affirmation « exposés directement, ils permettent d'usurper l'IP source » doit être sourcée ou nuancée.
- [ ] **Priorité de la souris locale** (section 12) : « la fonction existe dans le code mais elle est désactivée » doit être sourcée (issue ou commit GitHub) ou reformulée.

### Sections à ajouter

- [ ] **Sauvegarde** :
  - snapshot du conteneur après l'installation : `lxc snapshot rustdesk apres-install` ;
  - copie de la paire de clés `/opt/rustdesk/id_ed25519` et `id_ed25519.pub` hors du conteneur (`lxc file pull`). Si la clé est perdue, il faut reconfigurer tous les clients.
- [ ] **Mise à jour** : donner les commandes complètes au lieu du simple lien vers le dépôt :
  ```bash
  wget https://raw.githubusercontent.com/techahold/rustdeskinstall/master/update.sh
  chmod +x update.sh
  ./update.sh
  ```
- [ ] **Choix de la méthode d'installation** : préciser que `techahold/rustdeskinstall` est un script tiers, et citer les alternatives officielles (paquets `.deb` de `rustdesk-server`, Docker).
- [ ] **Nom de domaine** : expliquer l'intérêt d'un enregistrement DNS (choix `2` du script) plutôt qu'une IP dans les clients. Ça facilite un changement d'IP ou de serveur.
- [ ] **IPv6** : préciser qu'il n'est pas traité, ou ajouter les proxy devices IPv6 correspondants.
- [ ] **Démarrage automatique** : mentionner `lxc config set rustdesk boot.autostart true` pour que le conteneur redémarre avec l'hôte.
- [ ] **Clients Windows / Android** : indiquer où trouver les paramètres réseau sur les autres plateformes, ou renvoyer à la documentation officielle.

## Fait (01-11-2026)

- [x] Tableau des réponses au script d'installation refait (lignes coupées, lignes vides).
- [x] Statut du client harmonisé sur « Prêt » (au lieu de « Succés »).
- [x] Règles ufw corrigées : `ufw route allow` vers le conteneur (le trafic DNAT passe par FORWARD).
- [x] Note sur l'IP publique attribuée par DHCP (`dynamic`).
- [x] Note sur le test UDP non vérifiable avec `nc`.
- [x] Chemin exact de la clé publique (`/opt/rustdesk/id_ed25519.pub`) avec la commande `cat`.
- [x] Images indentées dans la liste de la section 10, descriptions alternatives ajoutées.
- [x] Espaces en fin de ligne et doubles lignes vides supprimés, ponctuation des listes harmonisée.
