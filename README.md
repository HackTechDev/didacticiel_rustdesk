# Didacticiel RustDesk dans un conteneur LXD

Tutoriel en français pour héberger son propre serveur [RustDesk](https://rustdesk.com/) (hbbs + hbbr) dans un conteneur LXD, avec redirection des ports en mode `nat=true` pour conserver l'IP réelle des clients.

Il décrit une installation adaptée à un cas d'usage précis (hôte Ubuntu avec LXD, serveur OSS installé par script, clients Linux). D'autres méthodes existent pour le serveur (Docker, paquets `.deb`, version Pro) et le client : le tutoriel les présente dans la section « Un cas d'usage parmi d'autres ».

Il est publié sur le blog [labo.hacktech.dev](https://labo.hacktech.dev/), géré avec Grav.

## Contenu du dépôt

| Fichier | Rôle |
|---|---|
| [`item.md`](item.md) | Le tutoriel, au format page Grav (front matter YAML + Markdown) |
| `rustdesk01.png` à `rustdesk08.png` | Captures d'écran du client RustDesk utilisées dans le tutoriel |
| [`SECURITY.md`](SECURITY.md) | Pistes de sécurisation du serveur, classées par priorité |
| [`IMPROVEMENTS.md`](IMPROVEMENTS.md) | Améliorations prévues et déjà faites sur le tutoriel |
| [`presentation/`](presentation/) | Supports pour présenter le tutoriel en une heure : diaporama Marp ([`slides.md`](presentation/slides.md)), script de démonstration ([`demo.md`](presentation/demo.md)) et fiche questions-réponses ([`questions-reponses.md`](presentation/questions-reponses.md)) |

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
