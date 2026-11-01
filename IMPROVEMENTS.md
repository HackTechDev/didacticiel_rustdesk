# Améliorations du tutoriel RustDesk / LXD

Pistes d'amélioration pour `item.md` (« Installer un serveur RustDesk dans un conteneur LXD »).

## À faire

### Priorité haute

- [ ] **Capture d'une connexion réussie** (section 10) : ajouter une capture de la fenêtre de session ouverte sur le poste distant.

### Priorité moyenne

- [ ] **Debian 13 au lieu de Debian 12** : Debian 13 (Trixie) est sortie en août 2025, Debian 12 n'est plus la version stable. Vérifier d'abord que le script `techahold/rustdeskinstall` fonctionne sur Debian 13.
- [ ] **Chaîne de configuration** (section 6) : le tutoriel dit qu'on peut l'importer, sans dire comment. Expliquer l'import (icône presse-papier du formulaire, visible sur `rustdesk07.png`) ou retirer la phrase.
- [ ] **Démarrage automatique** : mentionner `lxc config set rustdesk boot.autostart true` pour que le conteneur redémarre avec l'hôte.

### Priorité basse

- [ ] **IPv6** : préciser qu'il n'est pas traité, ou ajouter les proxy devices IPv6 correspondants.
- [ ] **Clients Windows / Android** : la section 9 indique quel fichier télécharger, mais pas où trouver les paramètres réseau sur ces plateformes. Ajouter des captures ou renvoyer à la documentation officielle.
- [ ] **Priorité de la souris locale** (section 16, FAQ) : « la fonction existe dans le code mais elle est désactivée » doit être sourcée (issue ou commit GitHub) ou reformulée.

### Front matter (Grav)

- [ ] **Compléter `og:image`** : l'URL s'arrête au dossier `02.admin-sys/`, sans nom d'image. L'aperçu sur les réseaux sociaux ne fonctionne pas.
- [ ] **Supprimer le bloc `content:`** (`items`, `limit`, `order`, `pagination`, `url_taxonomy_filters`) : il sert aux pages de liste de blog et n'a pas d'effet sur un article seul.
- [ ] **Ajouter l'image d'en-tête `screenshot.png`** : elle est déclarée dans `hero_image` mais absente du dossier. Les captures `rustdesk01.png` à `rustdesk08.png` sont bien présentes.

## Fait (01-11-2026)

- [x] Tableau des réponses au script d'installation refait (lignes coupées, lignes vides).
- [x] Statut du client clarifié : message « Succès » après Valider, puis « Prêt » sur l'onglet Accueil.
- [x] Règles ufw corrigées : `ufw route allow` vers le conteneur (le trafic DNAT passe par FORWARD).
- [x] Note sur l'IP publique attribuée par DHCP (`dynamic`).
- [x] Note sur le test UDP non vérifiable avec `nc`.
- [x] Chemin exact de la clé publique (`/opt/rustdesk/id_ed25519.pub`) avec la commande `cat`.
- [x] Images indentées dans la liste de la section 10, descriptions alternatives ajoutées.
- [x] Espaces en fin de ligne et doubles lignes vides supprimés, ponctuation des listes harmonisée.
- [x] Tags ajoutés dans le front matter : `LXD`, `Debian`, `Auto-hébergement`, `Bureau à distance`.
- [x] Section « Sauvegarde » ajoutée : snapshots (manuels et programmés), copie externe des clés et de la base, export du conteneur, restauration.
- [x] Section « Maintenance » : procédure de mise à jour complète (snapshot, `update.sh`, vérification, retour arrière).
- [x] Section 6 : sous-partie « Utiliser un nom de domaine » (enregistrement A, vérification `dig`, choix `2` du script, configuration des clients, passage d'une IP à un nom).
- [x] Informations sur les ports 21118/21119 retirées (inutiles : ports non redirigés, pas de client web).
- [x] Section 6 : nom de domaine déplacé avant l'installation, note sur la clé publique remontée juste après la sortie du script.
- [x] hbbs / hbbr définis en introduction, petit lexique (proxy device, DNAT, hole punching, INPUT/FORWARD) dans « Principe ».
- [x] Ton harmonisé à l'infinitif (suppression du tutoiement).
- [x] Section 9 : note sur les clients Windows, macOS, Debian/Ubuntu et Android.
- [x] Sections réordonnées : Sauvegarde (12) avant Maintenance (13), « Remarques » sur la souris déplacées en FAQ (14).
- [x] Section « Pourquoi LXD ? » : définition, avantages et inconvénients, mention d'Incus.
- [x] Schéma du chemin réseau dans « Principe » (client → IP publique → règle DNAT → conteneur), avec la note sur l'absence de proxy en mode `nat=true`.
- [x] Lexique : définition de « proxy device » corrigée, SNAT et DNAT comparés à une box internet.
- [x] Section 10 : sous-partie « Se connecter à un poste distant » (ID, mot de passe à usage unique, bouton Se connecter). Dépannage : bouton « Connexion » (compte) distingué de « Se connecter ».
- [x] Section 9 : explication du message « Pas prêt » avant la configuration.
- [x] Prérequis : installation de LXD (snap, groupe `lxd`, `lxd init`).
- [x] README.md ajouté au dépôt.
- [x] Section 14 « Sécurisation » : test de la clé obligatoire, SSH par clé, renvoi vers SECURITY.md.
- [x] Section 15 « Désinstallation » : suppression du conteneur, des règles ufw, du DNS, réinitialisation des clients.
- [x] Section « Un cas d'usage parmi d'autres » : contexte du tutoriel, autres méthodes d'installation du serveur (Docker, `.deb`, binaires, script, Pro) et formats du client par système.
