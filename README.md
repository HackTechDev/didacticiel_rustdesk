# Didacticiel RustDesk dans un conteneur LXD

Tutoriel en français pour héberger son propre serveur [RustDesk](https://rustdesk.com/) (hbbs + hbbr) dans un conteneur LXD, avec redirection des ports en mode `nat=true` pour conserver l'IP réelle des clients.

Il est publié sur le blog [labo.hacktech.dev](https://labo.hacktech.dev/), géré avec Grav.

## Contenu du dépôt

| Fichier | Rôle |
|---|---|
| [`item.md`](item.md) | Le tutoriel, au format page Grav (front matter YAML + Markdown) |
| `rustdesk01.png` à `rustdesk08.png` | Captures d'écran du client RustDesk utilisées dans le tutoriel |
| [`SECURITY.md`](SECURITY.md) | Pistes de sécurisation du serveur, classées par priorité |
| [`IMPROVEMENTS.md`](IMPROVEMENTS.md) | Améliorations prévues et déjà faites sur le tutoriel |

## Ce que couvre le tutoriel

- Création d'un conteneur Debian et attribution d'une IP fixe
- Redirection des ports 21115-21117 (TCP) et 21116 (UDP) avec des proxy devices LXD
- Installation du serveur avec le script [techahold/rustdeskinstall](https://github.com/techahold/rustdeskinstall)
- Configuration des clients et connexion entre deux postes
- Dépannage, sauvegarde, mise à jour, sécurisation et désinstallation

## Environnement de référence

- Hôte : Ubuntu 24.04 avec LXD
- Conteneur : Debian 12
- Client : RustDesk 1.5.0

> Les adresses, noms et clés du tutoriel sont des exemples (plages réservées à la documentation, comme `203.0.113.0/24`).
