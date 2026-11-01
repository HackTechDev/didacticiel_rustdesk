# Améliorations du tutoriel RustDesk / LXD

Pistes d'amélioration pour `item.md` (« Installer un serveur RustDesk dans un conteneur LXD »).

## À faire

### Front matter (Grav)

- [ ] **Compléter `og:image`** : l'URL s'arrête au dossier `02.admin-sys/`, sans nom d'image. L'aperçu sur les réseaux sociaux ne fonctionne pas.
- [ ] **Supprimer le bloc `content:`** (`items`, `limit`, `order`, `pagination`, `url_taxonomy_filters`) : il sert aux pages de liste de blog et n'a pas d'effet sur un article seul.
- [ ] **Vérifier la présence des images** dans le dossier de la page : `screenshot.png` (image d'en-tête) et `rustdesk01.png` à `rustdesk08.png`.

### Contenu à vérifier ou sourcer

- [ ] **Priorité de la souris locale** (section 14, FAQ) : « la fonction existe dans le code mais elle est désactivée » doit être sourcée (issue ou commit GitHub) ou reformulée.

### Sections à ajouter

- [ ] **Choix de la méthode d'installation** : préciser que `techahold/rustdeskinstall` est un script tiers, et citer les alternatives officielles (paquets `.deb` de `rustdesk-server`, Docker).
- [ ] **IPv6** : préciser qu'il n'est pas traité, ou ajouter les proxy devices IPv6 correspondants.
- [ ] **Démarrage automatique** : mentionner `lxc config set rustdesk boot.autostart true` pour que le conteneur redémarre avec l'hôte.
- [ ] **Clients Windows / Android** : indiquer où trouver les paramètres réseau sur les autres plateformes, ou renvoyer à la documentation officielle.

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
- [x] Section 14 « Sauvegarde » ajoutée : snapshots (manuels et programmés), copie externe des clés et de la base, export du conteneur, restauration.
- [x] Section 13 « Maintenance » : procédure de mise à jour complète (snapshot, `update.sh`, vérification, retour arrière).
- [x] Section 6 : sous-partie « Utiliser un nom de domaine » (enregistrement A, vérification `dig`, choix `2` du script, configuration des clients, passage d'une IP à un nom).
- [x] Informations sur les ports 21118/21119 retirées (inutiles : ports non redirigés, pas de client web).
- [x] Section 6 : nom de domaine déplacé avant l'installation, note sur la clé publique remontée juste après la sortie du script.
- [x] hbbs / hbbr définis en introduction, petit lexique (proxy device, DNAT, hole punching, INPUT/FORWARD) dans « Principe ».
- [x] Ton harmonisé à l'infinitif (suppression du tutoiement).
- [x] Section 9 : note sur les clients Windows, macOS, Debian/Ubuntu et Android.
- [x] Sections réordonnées : Sauvegarde (12) avant Maintenance (13), « Remarques » sur la souris déplacées en FAQ (14).
