# Guide utilisateur

Tu es lycéen·ne, tu veux apprendre Bash, et tu n'as jamais vraiment ouvert un terminal ? Ce guide est pour toi.

## Installation rapide via les binaires 

Pas besoin de disposer de Linux dans ta machine. Les binaires sont publiés sur la page [Releases](https://github.com/openDataSenegal/PanongbeneLearningLinux-PLL/releases) — un seul fichier à télécharger, zéro dépendance Python.

| Plateforme | Binaire | Cible |
|---|---|---|
| Linux x86_64 | `pll-linux-x86_64` | Ubuntu 22.04+ / glibc récent |
| macOS Apple Silicon | `pll-macos-arm64` | M1 / M2 / M3 |
| Windows x86_64 | `pll-windows-x86_64.exe` | Windows 10 / 11 |

### Linux / macOS

```bash
# 1. Récupérer le binaire (remplace VERSION par la dernière, ex. 1.2.0)
curl -L -o pll \
  https://github.com/openDataSenegal/PanongbeneLearningLinux-PLL/releases/download/VERSION/pll-linux-x86_64
# (sur macOS Apple Silicon : pll-macos-arm64)

# 2. Rendre exécutable et installer dans le PATH
chmod +x pll
sudo mv pll /usr/local/bin/

# 4. Lancer
pll --help
```

Sur **macOS**, Gatekeeper bloque le binaire au premier lancement (il n'est pas signé Apple Developer). Lève la quarantaine une fois :

```bash
xattr -d com.apple.quarantine /usr/local/bin/pll
```

### Windows (PowerShell)

```powershell
# 1. Télécharger
Invoke-WebRequest `
  -Uri "https://github.com/openDataSenegal/PanongbeneLearningLinux-PLL/releases/download/VERSION/pll-windows-x86_64.exe" `
  -OutFile "pll.exe"

# 3. Placer dans un dossier du PATH (ex. %USERPROFILE%\bin) puis ouvrir un nouveau terminal
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\bin" | Out-Null
Move-Item pll.exe "$env:USERPROFILE\bin\pll.exe"

# 4. Lancer
pll --help
```

## Démarrer une session

```bash
pll
```

Au premier lancement, l'agent crée ton profil et te pose une seule question : **quel est ton niveau ?**

| Niveau | Quand le choisir |
|---|---|
| 1 — Découverte | Tu n'as jamais tapé `ls` de ta vie |
| 2 — Flux de données | Tu connais `cd`, `ls`, `grep` et tu veux chaîner des outils |
| 3 — Scripting | Tu veux écrire des scripts `.sh` avec des boucles |
| 4 — Puissance | Tu maîtrises les scripts, tu veux du `sed`/`awk`/`regex` |
| 5 — Maîtrise | Tu veux scripter proprement avec gestion d'erreurs et sécurité |

Tu peux changer de niveau à tout moment :

- `pll --niveau 2` → force au démarrage
- `pll --change-niveau` → relance le diagnostic interactif
- `:niveau 2` (ou `:level 2`) pendant une session → bascule au vol, dès le prochain exercice

Et si tu préfères laisser l'agent décider : voir [Auto-détection de niveau](#auto-détection-de-niveau) plus bas.

## Le déroulé d'un exercice

```
🌱 Exercice 1/5  ·  Niveau 1 — Découverte
┌────────────────────────────────────────┐
│ Liste le contenu du dossier            │
│ /workspace/home en affichant les       │
│ détails (permissions, taille, date).   │
└────────────────────────────────────────┘

attempt 1/6 $ ls -l /workspace/home
```

À chaque tour :

1. L'agent te propose un exercice
2. Tu tapes ta commande — elle est exécutée **pour de vrai** dans un sandbox sécurisé
3. Tu vois la vraie sortie
4. L'agent t'explique si c'est juste, ou ce qui ne va pas

## Indices progressifs

Quand tu bloques :

| Tape | Ce que tu obtiens | Coût |
|---|---|---|
| `?` | Indice léger (une piste) | -2 XP |
| `??` | Indice fort (presque la solution) | -5 XP |
| `???` | Solution commentée | -10 XP |

Exemple :

```
attempt 2/6 $ ?
💡 Indice léger
`ls` affiche le contenu. Il existe une option pour le format *long*.

attempt 3/6 $ ls -l /workspace/home
✅ Correct — tu as compris le concept de navigation.   +13 XP
```

Demander plusieurs fois le même niveau d'indice ne coûte **rien** : le tarif n'est payé qu'une fois par exercice.

## Poser une question libre

Les indices sont ciblés sur l'exercice en cours. Parfois tu as juste besoin de **demander un truc** : « quelle commande pour la taille des fichiers ? », « pourquoi ma commande plante ? », « c'est quoi `awk` ? ». Tu as quatre entrées, du plus naturel au plus explicite.

### Tape ta question naturellement (nouveau)

Pendant un exercice, si ce que tu tapes ressemble à une question en français, l'agent s'en rend compte : il répond avec le **contexte de l'exo en cours** (énoncé + ta dernière commande + le message d'erreur), sans décompter ta tentative ni coûter d'XP.

```
attempt 2/6 $ kat /data/log.txt
❌ Ce n'est pas la bonne approche.

attempt 2/6 $ comment afficher le contenu d'un fichier ?
╭── ❓ comment afficher le contenu d'un fichier ? ──╮
│ Utilise `cat <fichier>` pour afficher le contenu  │
│ complet. Sur un gros fichier, `less <fichier>`    │
│ permet de défiler. Relis ta dernière commande :   │
│ une petite faute de frappe sur le nom de la       │
│ commande, peut-être ?                             │
╰───────────────────────────────────────────────────╯
attempt 2/6 $ cat /data/log.txt
✅ Correct — tu as compris le concept de read.   +10 XP
```

Détection déclenchée par un interrogatif en tête (`comment`, `pourquoi`, `quelle`, `pourrais-tu`, `explique`, `dis-moi`…) ou par une phrase complète finissant par `?`. Les globs shell courts (`ls ?`, `echo a?`) restent bien interprétés comme commandes.

### `:chat` — dialogue multi-tours sur l'exo

Pour creuser un concept en plusieurs questions, ouvre une discussion. L'agent garde en tête tes **6 derniers échanges** et le contexte de l'exercice.

```
attempt 2/6 $ :chat
ℹ Mode discussion — `:back` pour reprendre, `:quit` pour sortir.
💬 Toi › c'est quoi la différence entre cat et less ?
╭── ❓ … ──╮
│ `cat` dump tout d'un coup — pratique pour un fichier court. `less`   │
│ permet de défiler avec les flèches, très utile sur un gros log.      │
╰──────────────────────────────────────────────────────────────────────╯
💬 Toi › et pour voir juste le début ?
╭── ❓ … ──╮
│ `head -n 5 fichier` montre les 5 premières lignes, `tail -n 5` les   │
│ dernières. Sans option, `head`/`tail` en affichent 10.                │
╰──────────────────────────────────────────────────────────────────────╯
💬 Toi › :back
ℹ Retour à l'exercice.
attempt 2/6 $
```

Aucune tentative n'est consommée pendant le chat. Entrée vide ou `:back` reprend le prompt de commande ; `:quit` sort de la session entière.

### `:ask` — question ponctuelle explicite

Quand tu veux forcer le mode question (ex. ta phrase ne commence pas par un interrogatif), garde le préfixe historique :

```
attempt 2/6 $ :ask quelle commande pour connaître la taille des fichiers ?
```

Tu peux aussi taper `:ask` tout seul : l'agent ouvre un prompt dédié pour saisir la question. Comme la détection auto, `:ask` utilise le contexte de l'exo courant.

### Depuis le shell (sans démarrer de session)

**One-shot** — réponse courte puis exit :

```bash
pll --ask "quelle commande pour connaître la taille des fichiers"
```

**REPL** — enchaîner plusieurs questions (chaque question est traitée isolément) :

```bash
pll --qa
```

Tape tes questions, Entrée vide ou `:quit` pour sortir.

**Chat libre** — dialogue multi-tours avec **mémoire conversationnelle** :

```bash
pll --chat
```

Contrairement à `--qa`, le chat garde le fil : l'agent se souvient des 12 derniers messages et répond en tenant compte du contexte ("et comment je filtre ça ?" après une question sur `ls` sera bien interprété). Le mode s'appuie sur l'endpoint `/api/chat` natif d'Ollama.

| Commande | Action |
|---|---|
| _texte libre_ | Envoie le message à PLL |
| `:clear` | Efface l'historique de la conversation |
| `:quit` / `:exit` / `:back` / Entrée vide | Sort du chat |

Équivalents en langage naturel : `pll chat`, `pll discute avec moi`, `pll mode chat`, `pll on chatte`.

> **Mode hors-ligne** : si Ollama n'est pas lancé, l'agent tombe sur un mini-dictionnaire intégré qui couvre les questions les plus fréquentes (identité — `qui es tu`, `présente-toi` —, taille, permissions, processus, recherche, dates, variables d'env…). Pour des réponses complètes et adaptées, lance `ollama serve`.

## Commandes spéciales

À taper à la place d'une commande bash. **Toutes les options de la CLI `pll`** sont accessibles ici via le préfixe `:` — tape `:help` (ou `:?`) à tout moment pour la liste complète.

### Aide & cycle de vie

| Commande | Action |
|---|---|
| `:help` / `:?` | Affiche le tableau récapitulatif de toutes les commandes méta |
| `:skip` | Passe l'exercice (-15 XP) |
| `:quit` / `:exit` | Quitte la session (profil sauvegardé) |

### Indices et solution

| Commande | Action |
|---|---|
| `?` / `??` / `???` | Indices progressifs (coût XP croissant) |
| `:hint [1-3]` / `:indice [1-3]` | Demande d'indice explicite — équivalent de `?`/`??`/`???` |
| `:solution` | Affiche la solution canonique sans rien facturer |
| `:man <cmd>` | Affiche la page man (ex. `:man grep`) |

### Conversation avec le mentor

| Commande | Action |
|---|---|
| _question en français_ | Détectée automatiquement (`comment…`, `pourquoi…`, `…?`) et routée vers `:ask` |
| `:ask <question>` | Pose une question libre (ex: `:ask comment voir la taille ?`) — contextualisée à l'exo, sans consommer la tentative |
| `:chat` | Dialogue multi-tours **avec mémoire** sur l'exo courant (`:back` pour reprendre) |
| `:qa` | Q&R indépendantes, sans mémoire — chaque question est traitée isolément |

### Profil et progression

| Commande | Action |
|---|---|
| `:stats` | Affiche ta progression sans quitter |
| `:niveau N` / `:level N` | Change de niveau à la volée (1-5) |
| `:prof` | Vue enseignant — tableau de la classe en lecture seule |

### Configuration de la session

| Commande | Action |
|---|---|
| `:sujet <topic>` / `:topic <topic>` | Filtre les prochains exos par sujet (ex. `:sujet pipes`). `:sujet none` retire le filtre. |
| `:rephrase on\|off` | Active/désactive la reformulation LLM des énoncés |
| `:auto-level on\|off` | Active/désactive l'ajustement automatique du niveau |
| `:challenge on\|off` | Bascule le mode défi chronométré (effet sur les prochains exos) |

### Modèle LLM

| Commande | Action |
|---|---|
| `:model` / `:models` | Affiche le modèle Ollama courant et la liste des modèles installés |
| `:model <nom>` | Change de modèle Ollama à la volée (ex. `:model mistral`) — mémorisé dans le profil |

### Bibliothèque d'exos

| Commande | Action |
|---|---|
| `:generated` / `:list-generated` | Liste les exercices générés stockés localement |

> Une commande inconnue (`:plop`) ne consomme pas la tentative : un avertissement t'oriente vers `:help`. **Tab** complète les commandes méta ; **↑/↓** rejoue l'historique.

## XP et niveaux

Chaque exercice rapporte entre 10 et 80 XP selon sa difficulté. Les seuils :

| Niveau | XP requis |
|---|---|
| 1 | 0 |
| 2 | 100 |
| 3 | 300 |
| 4 | 700 |
| 5 | 1500 |

Quand tu franchis un seuil d'XP, tu passes au niveau suivant avec une petite animation. Tu peux aussi perdre de l'XP (indices, skip, erreurs en mode challenge) — l'XP seul ne te fait pas redescendre, mais l'auto-détection peut te ramener d'un niveau si tu bloques trop (voir [Auto-détection de niveau](#auto-détection-de-niveau)).

## Auto-détection de niveau

Pendant une session, l'agent observe tes 5 derniers exercices :

- Si tu réussis **≥ 4 sur 5** dont **≥ 3 sans aucun indice** → promotion au niveau suivant ✨
- Si tu réussis **≤ 1 sur 5** (après au moins 3 essais) → retour au niveau précédent pour consolider

Dans les deux cas, la fenêtre est remise à zéro pour que tu aies de l'air au nouveau niveau. Si tu veux garder la main, désactive-la :

```bash
pll --no-auto-level
```

Tu peux aussi reprendre le contrôle à tout moment avec `:niveau N` en session.

## Variantes d'énoncés

Beaucoup d'exercices ne sont pas figés mot pour mot — ils se présentent chaque fois avec des valeurs différentes (un nom de dossier, un nombre de lignes, un chemin). L'idée : que tu apprennes **le concept** (créer un dossier) plutôt que la réponse (`mkdir mes_notes`).

En plus, si Ollama tourne, l'agent reformule l'énoncé en tenant compte de ton niveau. Les éléments entre backticks (commandes, chemins, noms de fichiers) sont toujours préservés à l'identique — seul l'habillage varie.

Si tu préfères un énoncé strictement stable (mode offline, ou pour ne pas être perturbé par une reformulation qui glisserait) :

```bash
pll --no-rephrase
```

## Parler à pll en français (sans les flags)

Pas envie de mémoriser les options ? Écris simplement ce que tu veux, pll devine :

```bash
pll augmente le niveau              # → passe au niveau supérieur
pll niveau 3                        # → force le niveau 3
pll entraîne-moi sur les pipes      # → --sujet pipes
pll pratique grep                   # → --sujet grep
pll mes stats                       # → --stats
pll mode défi                       # → --challenge
pll relance le diagnostic           # → --change-niveau
pll "comment voir la taille ?"      # → --ask "..."
pll discute avec moi                # → --chat (dialogue multi-tours)
pll mode chat                       # → --chat
pll efface tout                     # → --reset-all
pll reset                           # → --reset
pll utilise mistral                 # → --model mistral
pll "change le modèle pour qwen2.5" # → --model qwen2.5
pll quels modèles sont installés    # → --list-models
```

Avant d'exécuter, pll affiche ce qu'il a compris :

```
🧠 Interprété comme : niveau → 3 (Scripting)
```

Comme ça, pas de surprise. Si rien ne matche, pll te renvoie vers `pll --help`.

**Quelques règles utiles** :

- « augmente le niveau » / « baisse le niveau » te font monter/descendre d'un cran (borné 1-5). Le mot *niveau* (ou *level*, *palier*) doit être présent, sinon « augmente » tout seul est trop ambigu.
- « reset all » / « efface tout » / « wipe » **écrasent tous les profils** — ça demande toujours une confirmation (taper `WIPE`).
- « reset » / « supprime mon profil » n'efface **que ton profil** (confirmation : retaper ton nom).
- Toute phrase de ≥ 3 mots qui ne matche aucune action est traitée comme une question libre → `--ask`.

Tu peux bien sûr mélanger — `pll --niveau 3 --challenge` marche comme avant. Les flags explicites ne sont **jamais** écrasés par l'interprétation.

## Options de la CLI

```bash
pll                         # session normale
pll --niveau 3              # force le niveau de départ
pll --change-niveau         # relance le diagnostic interactif
pll --no-auto-level         # désactive l'auto-détection de niveau
pll --sujet pipes           # entraînement ciblé sur un thème
pll --exercices 10          # plus d'exos par session (défaut 5)
pll --challenge             # mode chronométré, bonus de vitesse +25%
pll --stats                 # affiche ta progression et quitte
pll --ask "…"               # pose une question libre et quitte
pll --qa                    # mode questions/réponses (stateless)
pll --chat                  # mode chat libre (mémoire conversationnelle)
pll --nom alice             # choisir un profil différent
pll --no-docker             # désactive Docker (fallback local)
pll --no-rephrase           # désactive la reformulation LLM des énoncés
pll --reset                 # supprime le profil courant (confirmation)
pll --reset-all             # wipe complet de la base locale (tous profils)
pll --model mistral         # change le modèle Ollama (mémorisé dans ton profil)
pll --list-models           # liste les modèles Ollama installés et quitte
pll --generate "<sujet>"    # génère un exo via LLM, validé sandbox, stocké en SQLite
pll --practice "<sujet>"    # rejoue un exo généré (XP ×0.5, auto-level off)
pll --practice-count N      # nombre d'exos enchaînés en --practice (défaut 1)
pll --list-generated        # liste les exos générés stockés localement
pll --explain-script <PATH> # explique un script bash long (PATH ou "-" pour stdin)
pll --generate-script "..." # génère un script bash long et complet
pll --output script.sh      # fichier de sortie pour --generate-script (sinon stdout)
```

### Thèmes disponibles pour `--sujet`

- `navigation`, `files`, `read` (niveau 1)
- `redirections`, `pipes`, `grep`, `find`, `sort`, `uniq` (niveau 2)
- `variables`, `loops`, `conditions`, `functions`, `codes_retour` (niveau 3)
- `sed`, `awk`, `xargs`, `substitution`, `processus`, `regex` (niveau 4)
- `droits`, `defensive`, `reseau`, `securite` (niveau 5)

## Exos générés à la volée (mode practice)

Besoin d'un exo sur un sujet qui n'est pas pris en compte officiel ? Demande à PLL de t'en fabriquer un :

```bash
pll --generate "comptage de lignes dans un log" --niveau 2
```

### Rejouer un exo généré

```bash
pll --practice "comptage"              # 1 exo (tiré au hasard parmi ceux stockés)
pll --practice "log" --practice-count 3 # 3 exos d'affilée sur le thème
```

Le match sur le sujet est partiel et insensible à la casse. Les exos les moins utilisés sont prioritaires (pour tourner sur ton catalogue plutôt que toujours le même).

### Voir ce que tu as dans la base

```bash
pll --list-generated
```

Affiche un tableau avec id, sujet, niveau, nombre d'utilisations, statut validé, et un extrait de l'énoncé.

### Règles du mode practice

- **XP réduite à 50 %** par rapport aux exos curés. Ces exos ne sont pas calibrés par un humain — tu ne peux pas « grinder » ton niveau avec.
- **L'auto-détection de niveau est désactivée** dans ce mode. Réussir ou rater un exo généré n'impacte pas ta progression officielle.
- **Il faut Ollama.** Sans modèle LLM actif, `--generate` refuse proprement et te demande de lancer `ollama serve`. Rien de pire qu'un exo « généré » par un fallback scripté qui serait mal évalué.

### Limites

- La qualité dépend du modèle que tu utilises. Un modèle 1B te sortira des exos simplistes, un 7B+ sera mieux.
- Si un sujet ne s'y prête pas (ex. « firewall iptables ») le sandbox sans réseau va rejeter systématiquement — la génération finira par abandonner au bout de 3 essais avec un message clair.
- L'évaluation utilise le regex dérivé de la commande canonique du LLM : l'ordre des arguments compte (comme pour les exos curés).

## Scripts bash longs : expliquer ou générer

Au-delà des one-liners qu'on tape à l'invite, deux flags traitent des **scripts bash complets** — utile pour décortiquer un script qu'on lit dans une codebase, ou pour partir d'un squelette propre quand on doit en écrire un.

### `--explain-script` — décortiquer un script

```bash
pll --explain-script ~/scripts/backup.sh         # depuis un fichier
cat /etc/init.d/foo | pll --explain-script -     # depuis stdin
pll --explain-script deploy.sh --niveau 4        # ajuste le niveau pédagogique
```

Le LLM produit une analyse structurée en Markdown :

1. **Vue d'ensemble** — 2 ou 3 phrases sur le but global du script.
2. **Décomposition** — bloc par bloc, avec citations exactes du code et explication du *pourquoi*.
3. **Concepts clés** — les notions Bash mobilisées (redirections, traps, substitutions, etc.).
4. **Points de vigilance** — bugs potentiels, edge cases, problèmes de robustesse, sécurité.
5. **Suggestions** — 1 à 3 améliorations concrètes (idiomes, lisibilité, robustesse).

Le niveau pédagogique se calibre sur `--niveau N` (ou ton profil par défaut). Les très gros scripts sont tronqués à 12 000 caractères avant envoi (variable d'env `PLL_MAX_SCRIPT_CHARS` pour ajuster) — un avertissement t'indique si la coupe a eu lieu.

### `--generate-script` — produire un script complet

```bash
pll --generate-script "rotation de logs avec compression gzip et purge >30j" \
    --output rotate.sh --niveau 4
pll --generate-script "scan de ports rapide d'une plage CIDR" --niveau 5
```

Le script généré respecte les conventions de production :

- shebang `#!/usr/bin/env bash` + `set -euo pipefail` ;
- fonctions nommées, variables `local` dans les fonctions, constantes en MAJUSCULES ;
- gestion d'erreur (codes de retour, messages sur `stderr` via `>&2`) ;
- validation des arguments (usage / help) si le script en prend ;
- commentaires `#` qui expliquent le *pourquoi* des étapes non triviales.

Cible : 40 à 150 lignes selon la complexité demandée. La syntaxe est **automatiquement vérifiée avec `bash -n`** : si le script ne passe pas le check, tu vois l'erreur signalée et le code retour passe à `2` — utile en CI ou en boucle. Sortie par défaut : impression colorée avec numéros de ligne dans le terminal. Avec `--output chemin.sh`, le script est écrit sur disque et marqué exécutable (`chmod 755`).

### Règles communes

- **Il faut Ollama.** Les deux modes refusent net si Ollama est injoignable — pas de fallback offline, ces tâches n'ont aucun sens sans modèle. Lance `ollama serve` puis réessaie.
- **Choix du modèle.** La qualité dépend fortement du modèle. Un 7B+ (`llama3.1:8b`, `qwen2.5-coder:7b`) donne des explications/scripts nettement supérieurs à un 1-3B. Bascule via `pll --model <nom>` (persisté).
- **Sécurité.** Le prompt système interdit `sudo`, accès réseau (`curl`, `wget`, `apt`…), `rm -rf /` et `eval` sur entrée non vérifiée. Mais comme partout : **relis avant d'exécuter** un script généré par un LLM.

## Mode challenge

```bash
pll --challenge
```

Le chrono démarre à chaque exercice. Si tu réponds en moins de 20 secondes, tu gagnes **+25% d'XP bonus**. L'objectif : devenir réflexe.

## Recommencer à zéro

Si tu veux effacer ta progression et repartir d'un profil vierge :

```bash
pll --reset         # supprime uniquement ton profil (+ historique)
pll --reset-all     # wipe complet : tous les profils de la machine
```

Les deux demandent une confirmation explicite (retaper le nom du profil, ou taper `WIPE` en majuscules) — pas de suppression accidentelle. Une fois fait, le prochain lancement te fera repasser par le diagnostic de niveau.

## Reprendre une session

Ton profil est persistant. À chaque lancement, il affiche :

```
Bon retour, Panongbene ! Niveau courant : 2 (Flux de données)  ·  XP : 145
```

Tes exos déjà réussis ne te sont pas redonnés en priorité — sauf si le stock du niveau est épuisé.

## Choisir le modèle Ollama

PLL utilise Ollama en local pour générer explications, feedback et réponses libres. Le modèle par défaut est `llama3.2` — mais tu peux le changer si tu en as un autre installé (`mistral`, `qwen2.5`, `llama3.1:8b`…).

**Trois façons de changer :**

1. **Au lancement** — persiste dans ton profil pour les prochaines sessions :
   ```bash
   pll --model mistral
   ```

2. **En langage naturel** :
   ```bash
   pll utilise qwen2.5
   pll "change le modèle pour llama3.1:8b"
   ```

3. **Pendant une session** :
   ```
   attempt 1/6 $ :model              # affiche le modèle courant + la liste
   attempt 1/6 $ :model mistral      # bascule au vol, mémorisé dans le profil
   ```

**Voir ce qui est installé :**

```bash
pll --list-models
```

Ou, en session, tape simplement `:model` (sans argument).

**Priorité de résolution** (du plus fort au plus faible) :

1. `--model` passé en ligne de commande
2. `preferred_model` mémorisé dans le profil
3. Variable d'environnement `OLLAMA_MODEL`
4. Défaut : `llama3.2`

**Si le modèle demandé n'est pas installé**, PLL se débrouille tout seul au démarrage :

- S'il y a **d'autres modèles** sur Ollama, il bascule automatiquement sur le plus familier installé (priorité à `llama3.2 → llama3.1 → mistral → qwen2.5 → qwen3 → gemma2 → gemma3`) et mémorise ce nouveau choix dans ton profil :

  ```
  ⚠  Le modèle « fantome » n'est pas installé côté Ollama.
     Bascule automatique sur « mistral:latest ».
  ```

- Si **aucun modèle** n'est installé, PLL télécharge `mistral` (~4 Go) via `ollama pull` automatique, avec barre de progression (taille, vitesse, ETA) :

  ```
  ⚠  Aucun modèle Ollama installé. Téléchargement de « mistral »…
  mistral · downloading ████████████░░░░░  2.1/4.1 GB  45.2 MB/s  0:00:42
  ℹ  Modèle « mistral » installé et sélectionné.
  ```

- Si le téléchargement échoue (pas de réseau, disque plein), PLL tombe en mode dégradé (mini-dictionnaire intégré) et t'indique la commande manuelle à lancer (`ollama pull mistral`).

## Quand ça ne marche pas

- **"Ollama indisponible"** → l'agent tourne en mode dégradé, tu perds juste les explications IA personnalisées. Lance `ollama serve` à côté.
- **"Docker indisponible"** → passage en sandbox local, moins isolé mais fonctionnel.
- **Commande bloquée** → tu as tapé un truc dangereux (`rm -rf /`, fork bomb). Reformule.
- **Timeout** → ta commande tourne en boucle ou attend une entrée. Termine-la proprement.

## Auteur

- GitHub : https://github.com/Panongbene
- Portfolio : https://panongbene.com
- Contact : amt1900@gmail.com
- Linkedin : www.linkedin.com/in/panongbene-jean-mohamed-sawadogo-33234a168
