---
marp: true
theme: default
paginate: true
title: Son propre serveur RustDesk dans LXD
---

<!--
Support de présentation au format Marp (https://marp.app/).
Aperçu : extension « Marp for VS Code », ou en ligne de commande :
  npx @marp-team/marp-cli slides.md --html      (export HTML)
  npx @marp-team/marp-cli slides.md --pdf       (export PDF)
Les commentaires HTML comme celui-ci sont les notes de l'orateur.
-->

# Son propre serveur RustDesk
## dans un conteneur LXD

Prendre la main à distance… sans passer par les serveurs des autres

<!--
0:00 — Ne pas commencer par cette slide trop longtemps.
Enchaîner tout de suite sur la démonstration d'accroche (voir demo.md, étape 0).
-->

---

# Démonstration

**Prise en main d'un poste à distance**
à travers mon propre serveur

<!--
0:00-0:03 — Accroche : se connecter en direct à la machine de démo.
Puis demander à la salle :
« Qui utilise TeamViewer, AnyDesk ou RustDesk ? Par où passent vos sessions ? »
-->

---

# Par défaut, où passe le trafic ?

```
WebRTC failed: Failed to connect: the target device is offline or does not exist.;
fallback failed: Failed to connect to rs-ny.rustdesk.com:21116
```

- Un client RustDesk non configuré utilise les **serveurs publics** de RustDesk
- Serveurs partagés, parfois saturés, hors de notre contrôle
- **Solution** : héberger son propre serveur, gratuit et open source

<!--
0:03-0:08 — Montrer l'erreur en direct si possible (client non configuré).
Message clé : on ne maîtrise ni la disponibilité ni le point de passage.
-->

---

# Un serveur RustDesk, c'est deux programmes

| Programme | Rôle | Ports |
|---|---|---|
| **hbbs** | Serveur ID : enregistre les postes et les met en relation | 21115, 21116 (TCP + UDP) |
| **hbbr** | Relais : fait transiter le trafic si la connexion directe échoue | 21117 |

> La plupart du temps, les postes se connectent **directement** (*hole punching*) : le serveur ne fait que les présenter.

<!--
Insister : le serveur consomme peu tant que les connexions directes réussissent.
-->

---

# Pourquoi LXD ?

**Conteneur système** : une petite Debian complète, qui partage le noyau de l'hôte

| ✅ Avantages | ⚠️ Inconvénients |
|---|---|
| Isolé du reste du serveur | Isolation moins forte qu'une VM |
| Léger, démarre en secondes | Réseau privé : redirection nécessaire |
| S'administre comme un serveur Debian | Hôte **et** conteneur à mettre à jour |
| Snapshots et export en une commande | Moins documenté que Docker |

<!--
0:08-0:10 — Question probable : « Pourquoi pas Docker ? » → voir questions-reponses.md.
-->

---

# Le problème du réseau

```
Internet ──► Hôte 203.0.113.10 ──► ??? ──► Conteneur 10.10.10.50
```

- Le conteneur est sur un **réseau privé** (`lxdbr0`)
- Il faut rediriger les ports de l'hôte vers le conteneur
- **Contrainte** : hbbs doit voir l'**IP réelle** des clients, sinon pas de connexion directe

<!--
0:10 — Poser la question à la salle : « Comment feriez-vous chez vous avec une box ? »
Réponse attendue : une redirection de ports. Parfait, c'est exactement ça.
-->

---

# La solution : DNAT

| | Change… | Exemple |
|---|---|---|
| **SNAT** | l'adresse **source** | La box quand un PC sort sur Internet |
| **DNAT** | l'adresse **de destination** | La « redirection de ports » de la box |

```
Avant : client 198.51.100.7 → 203.0.113.10:21116
Après : client 198.51.100.7 → 10.10.10.50:21116    (source inchangée ✅)
```

<!--
Montrer que seule la destination change : hbbs voit toujours 198.51.100.7.
-->

---

# Proxy devices LXD en mode `nat=true`

```bash
lxc config device add rustdesk rd-tcp proxy \
  listen=tcp:203.0.113.10:21115-21117 \
  connect=tcp:10.10.10.50:21115-21117 nat=true
```

- On **déclare** la redirection dans la configuration du conteneur
- LXD **crée la règle DNAT** dans le noyau de l'hôte
- **Aucun programme** ne relaie le trafic
- Règles créées et supprimées avec le conteneur, incluses dans les snapshots

<!--
0:15 — Démo : `sudo nft list ruleset | grep 2111` pour montrer la règle « invisible ».
-->

---

# Le chemin complet

```
Client RustDesk
      │
      ▼
IP publique de l'hôte : 203.0.113.10
ports 21115-21117 (TCP) et 21116 (UDP)
      │
      ▼
Règle DNAT nftables, créée par LXD (nat=true)
      │   destination réécrite, IP du client conservée
      ▼
Conteneur rustdesk : 10.10.10.50
hbbs (21115, 21116) – hbbr (21117)
```

---

# Piège : le pare-feu

```bash
sudo ufw allow 21116            # ❌ ne suffit pas
sudo ufw route allow proto tcp to 10.10.10.50 port 21115:21117   # ✅
sudo ufw route allow proto udp to 10.10.10.50 port 21116         # ✅
```

- Le trafic redirigé **traverse** l'hôte : chaîne *FORWARD*
- `ufw allow` ne règle que *INPUT* (trafic destiné à l'hôte lui-même)

<!--
Bon moment pour une question à la salle : « Pourquoi ça ne marcherait pas ? »
-->

---

# Installation en direct

1. Créer le conteneur et lui donner une **IP fixe**
2. Déclarer les **proxy devices**
3. Lancer le script `install.sh` dans le conteneur
4. **Noter la clé publique**

<!--
0:20-0:40 — Passer au terminal (demo.md, étapes 2 à 6).
Pendant que le script tourne : expliquer le choix IP / nom de domaine.
-->

---

# Configuration du client

| Champ | Valeur |
|---|---|
| Serveur ID | `203.0.113.10` ou `rustdesk.example.com` |
| Clé | `<CLE_PUBLIQUE>` |
| Relais, API | vides |

**Sur chaque poste** : celui qui contrôle **et** celui qui est contrôlé

<!--
0:40-0:50 — Faire venir un volontaire : il configure son client, puis connexion en direct.
-->

---

# Les pièges classiques

| Symptôme | Cause |
|---|---|
| `rs-ny.rustdesk.com:21116` | Un des deux postes n'est pas configuré |
| `Connexion refusée` port 21114 | Bouton **Connexion** (compte) : réservé à la version Pro |
| « Pas prêt » | Serveur injoignable, clé ou IP erronée |

<!--
0:50-0:53 — Provoquer les erreurs en direct (demo.md, étape 8).
-->

---

# Sécurité et sauvegarde

- 🔑 **La clé doit être obligatoire** : tester avec une clé fausse
- 🔐 **SSH par clé**, mots de passe désactivés
- 💾 **Sauvegarder `id_ed25519`** : la perdre oblige à reconfigurer tous les clients
- 📸 **Snapshots LXD** avant chaque mise à jour

<!--
0:53-0:55 — Insister sur la clé privée : c'est l'élément irremplaçable.
-->

---

# À retenir

1. **DNAT** (`nat=true`) : la redirection qui conserve l'IP des clients
2. **La clé** : obligatoire, notée, sauvegardée
3. **Les snapshots** : le filet de sécurité de LXD

**Tutoriel complet** : github.com/HackTechDev/didacticiel_rustdesk

<!--
0:55-0:60 — Afficher le QR code du dépôt (à générer : voir demo.md, préparation).
-->

---

# Questions ?

---

<!-- Slides de réserve, à afficher seulement si la question vient. -->

# Réserve : proxy avec ou sans `nat=true`

| | `nat=true` | Sans `nat` |
|---|---|---|
| Mécanisme | Règle DNAT dans le noyau | Processus `forkproxy` de LXD |
| IP vue par hbbs | **IP du client** ✅ | IP de l'hôte ❌ |
| Connexion directe | Possible | Impossible, tout passe par le relais |
| IP fixe du conteneur | Obligatoire | Non requise |

---

# Réserve : sauvegarde

```bash
lxc snapshot rustdesk apres-install                     # point de reprise
lxc config set rustdesk snapshots.schedule "@daily"     # snapshot quotidien
lxc file pull rustdesk/opt/rustdesk/id_ed25519 ~/sauvegarde/   # copie de la clé
lxc export rustdesk rustdesk-sauvegarde.tar.gz          # export complet
```
