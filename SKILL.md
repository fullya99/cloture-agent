---
name: cloture-agent
description: Clôture de session pour un agent personnel permanent, Hermes Agent ou OpenClaw. À lancer avant un /new, un /reset ou un /clear, et à la fin de toute unité de travail. Synchronise les 3 piliers du projet CODEMAP.md TODOS.md CHANGELOG.md, purge et corrige la mémoire de l'agent dans MEMORY.md et USER.md, range les skills que l'agent s'est écrits tout seul, et déplace vers archives/ tout ce qui est devenu faux ou périmé. Se déclenche sur : clôture ou fin de session, "avant de faire /new", "range tout", "mets à jour ta mémoire", "fais le ménage dans tes skills", "documente ce qu'on a fait", mémoire qui enfle, skills en doublon, doc vivante, CODEMAP, TODOS, CHANGELOG.
version: 1.0.0
author: fullya99
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [memory, skills, documentation, session, housekeeping]
    related_skills: [plan]
---

# Clôture de session, pour un agent qui vit en continu

> **Le contrat** : à tout instant, ce qui est écrit sur le disque doit suffire. Un `/new` ne doit
> jamais coûter une information, et une mémoire qui grossit sans être relue finit par mentir.

Tu tournes en service permanent, mais ta session se remet à zéro. Chez Hermes, `/new` ouvre une
session neuve, `/reset` vide l'historique. Chez OpenClaw, tu te réveilles à neuf à chaque session.
Ce qui n'est pas écrit avant ce moment-là n'existe plus après.

Deux risques, pas un. Le premier est de perdre du contexte à la coupure. Le second est propre aux
agents permanents : ta mémoire est réinjectée dans ton prompt à **chaque** appel, et rien ne la purge
tout seul. Une mémoire qui enfle coûte des tokens à chaque tour, et une mémoire fausse coûte bien
plus cher que ça.

**Avant d'écrire quoi que ce soit**, charge le skill `style-redaction` s'il est installé. Sinon le
minimum tient en une ligne : français, ton direct, aucun tiret cadratin, aucun point-virgule, aucune
tournure qui sent le texte généré.

---

## Étape 0. Où tu tournes, et où sont tes fichiers

Détecte la plateforme avant tout, les chemins en dépendent.

```bash
for base in "$HOME/.hermes" "$HOME/.openclaw/workspace"; do
  [ -d "$base" ] && echo "trouve : $base"
done
```

| | Hermes Agent | OpenClaw |
|---|---|---|
| Identité | `~/.hermes/SOUL.md` | `<workspace>/SOUL.md` |
| Profil utilisateur | `~/.hermes/memories/USER.md` | `<workspace>/USER.md` |
| Mémoire | `~/.hermes/memories/MEMORY.md` | `<workspace>/MEMORY.md` |
| Journal du jour | néant | `<workspace>/memory/AAAA-MM-JJ.md` |
| Skills | `~/.hermes/skills/<categorie>/<nom>/` | `<workspace>/skills/<nom>/` |
| Historique | `~/.hermes/state.db`, SQLite FTS5 | store de session |

Le workspace d'OpenClaw est `~/.openclaw/workspace` par défaut, mais il se configure dans
`~/.openclaw/openclaw.json` sous `agents.defaults.workspace`. Vérifie avant d'écrire, un agent
multi-workspace a un dossier par agent.

Si aucune des deux bases n'existe, tu tournes ailleurs. Fais la partie projet, dis que tu sautes la
partie mémoire, et n'invente pas de chemins.

---

## Les trois propriétaires d'une phrase

C'est la question qui tranche tout, et elle a trois réponses au lieu d'une.

| Ce que tu viens d'apprendre | Où ça va |
|---|---|
| Comment ce projet est fait, ce qui touche quoi | `CODEMAP.md` du projet |
| Ce qui reste à faire, ce qui bloque | `TODOS.md` du projet |
| Ce qui a changé et comment revenir en arrière | `CHANGELOG.md` du projet |
| Une décision et son pourquoi | `docs/decisions/` du projet |
| Une règle vraie en permanence sur ce projet | `CLAUDE.md` ou `AGENTS.md` du projet |
| Une préférence durable de l'utilisateur, valable partout | `USER.md` |
| Un fait durable sur l'environnement, les accès, les pièges machine | `MEMORY.md` |
| Qui tu es, ton ton, tes refus | `SOUL.md`, et seulement sur demande explicite |
| Ce qui s'est passé aujourd'hui, en brut | `memory/AAAA-MM-JJ.md` chez OpenClaw, le CHANGELOG sinon |

Deux erreurs à ne pas faire.

**Ne mets pas dans `MEMORY.md` ce qui appartient au projet.** Un fait sur un dépôt vit dans ce dépôt,
sinon il devient invisible pour qui lit le dépôt, et il te suit dans des sessions où il ne sert à
rien tout en coûtant des tokens.

**Ne touche pas à `SOUL.md` de ta propre initiative.** C'est ton identité, elle occupe le premier
emplacement de ton prompt système. Elle se modifie quand on te le demande, jamais parce que tu as
déduit quelque chose sur toi.

---

## Router

| Situation | Mode |
|---|---|
| Fin de session, avant un `/new` ou un `/reset`, fin d'unité de travail | **CLÔTURE** |
| Début de session, « où on en était » | **REPRISE** |
| Le projet n'a pas les 3 piliers | **INIT**, puis CLÔTURE dans la foulée |

---

## Mode INIT

```bash
bash "$SKILL/scripts/ctx-init.sh" --dry-run   # voir ce qui serait créé
bash "$SKILL/scripts/ctx-init.sh"             # créer, sans jamais écraser
```

`$SKILL` est le répertoire de ce skill. Tu travailles depuis le projet, pas depuis le skill, donc un
chemin relatif ne résout pas. Résous-le une fois :

```bash
SKILL="$(for d in "$HOME/.openclaw/workspace/skills/cloture-agent" \
  $(find "$HOME/.hermes/skills" -maxdepth 3 -type d -name cloture-agent 2>/dev/null) \
  $(find "$HOME/.openclaw" -maxdepth 4 -type d -name cloture-agent 2>/dev/null); do
  [ -f "$d/scripts/ctx-audit.sh" ] && echo "$d" && break
done)"
[ -n "$SKILL" ] || echo "skill introuvable, fais les etapes a la main sans le script"
```

Le script pose des gabarits vides. Le travail commence après. Lis le projet avant d'écrire,
arborescence, points d'entrée, fichier de build, CI, derniers commits. Ne remplis jamais le CODEMAP
depuis une supposition. Le détail est dans `references/CONVENTION.md`.

---

## Mode CLÔTURE

### Phase 0. Cadrer

```bash
git status -s && git diff --stat && git log --oneline -10
```

Repère ce qui a changé, y compris hors git, et **tous les niveaux touchés**. Une session partie d'un
sous-répertoire touche souvent au-dessus. Calibre l'effort sur la taille de la session, mais ne saute
ni la phase 3 ni la phase 4.

### Phase 1. Auditer le réel, sans croire la doc

```bash
bash "$SKILL/scripts/ctx-audit.sh"
```

Le script signale, il ne juge pas. Vérifie sur le terrain ce que la session a touché. Le service
tourne-t-il, le chemin cité existe-t-il encore, le test passe-t-il.

Note chaque écart sous la forme « c'est écrit X, la réalité est Y ». Chaque écart se termine par une
correction ou un archivage. Aucun ne reste ouvert.

### Phase 2. Les 3 piliers du projet

| Pilier | Ce que tu y fais |
|---|---|
| `CHANGELOG.md` | Entrée du jour en haut. Quoi, pourquoi, comment défaire |
| `CODEMAP.md` | La structure réelle. **Retire ce qui n'existe plus.** Ajoute les pièges aux zones sensibles |
| `TODOS.md` | Coche, **supprime** l'obsolète, ajoute les dettes trouvées, puis réécris le bloc d'état de reprise |

Le bloc d'état de reprise est un **état**, pas un journal. L'histoire vit au CHANGELOG. Si le bloc
raconte la session au lieu de dire où on en est, réécris-le.

### Phase 3. Ta mémoire, phase propre aux agents permanents

Elle est réinjectée à chaque appel. Relis-la en entier, à chaque clôture.

1. **Corrige ce qui est devenu faux.** Une affirmation fausse dans `MEMORY.md` te fera te tromper à
   chaque session, silencieusement. C'est le pire état possible.
2. **Supprime ce qui n'a plus d'usage.** Un fait qui ne servira plus jamais coûte des tokens à
   chaque tour. Suppression pure, pas d'archivage : `MEMORY.md` n'est pas un journal.
3. **Déplace ce qui appartient au projet** vers le dépôt concerné, selon le tableau des trois
   propriétaires.
4. **Fusionne les doublons.** Deux formulations d'un même fait vont finir par divergerv, et tu ne
   sauras plus laquelle croire.
5. **Mets à jour `USER.md`** avec les préférences exprimées cette session, corrections comprises.
   Une correction de l'utilisateur qui ne finit pas écrite se répétera.

Chez OpenClaw, le journal `memory/AAAA-MM-JJ.md` prend le brut de la journée. Il n'est pas une
sauvegarde de `MEMORY.md`, il n'a pas à être relu ni purgé, mais ce qui y devient durable remonte
dans `MEMORY.md`.

Si `MEMORY.md` dépasse ce que tu relirais volontiers en entier, il est trop gros. Coupe.

### Phase 4. Tes skills auto-créés, l'autre phase propre aux agents

Tu écris tes propres skills et tu les retouches. Personne ne fait le ménage derrière toi.

```bash
# Hermes
find "$HOME/.hermes/skills" -name SKILL.md | wc -l
# OpenClaw
find "$HOME/.openclaw/workspace/skills" -name SKILL.md | wc -l
```

Pour chaque skill que tu as créé ou modifié cette session :

- **Il marche et il sert** : vérifie que sa `description` dit vraiment quand le déclencher. Une
  description floue produit un skill qui ne part jamais, ou qui part tout le temps.
- **Il double un autre** : fusionne. Deux skills qui se chevauchent se déclenchent au hasard.
- **Il ne sert plus, ou il s'est révélé faux** : supprime-le. Dis-le, ne le fais pas en silence.
- **Il contient un secret, une clé, un chemin de home** : corrige tout de suite. Les skills se
  publient et se partagent.

Le critère qui tranche : un skill mérite d'exister si sa connaissance a été difficile à obtenir et
qu'elle resservira. Sinon c'est une note, et une note va dans `MEMORY.md`.

### Phase 5. Purger l'obsolète du projet

Fiches, procédures, scripts, plans abandonnés. Ce qui est faux, périmé ou remplacé part dans
`archives/` avec un en-tête qui dit la date, l'origine, la raison et le remplaçant. Une section
fausse dans un fichier encore utile se corrige sur place. Une règle devenue fausse dans `CLAUDE.md`
se supprime, une règle erronée est pire qu'une règle absente. Le détail est dans
`references/ARCHIVAGE.md`.

**Aucune suppression silencieuse.** Aucune affirmation connue-fausse laissée pour plus tard.

### Phase 6. Test « prêt pour /new »

- [ ] Les 3 piliers reflètent l'état vérifié en phase 1, sans se contredire.
- [ ] Le bloc d'état de reprise de `TODOS.md` permet de repartir sans cette conversation.
- [ ] Tout travail en cours non fini a une tâche écrite.
- [ ] `MEMORY.md` relu en entier, rien de faux, rien d'inutile, rien qui appartienne au projet.
- [ ] `USER.md` à jour des préférences et des corrections de cette session.
- [ ] Les skills touchés sont propres, sans doublon, sans secret.
- [ ] Aucun secret dans un fichier tracké. `git status` conforme à l'intention.

### Phase 7. Compte-rendu, puis commit si on te le demande

```
**Clôture, AAAA-MM-JJ**

Fait : [1 à 3 lignes factuelles]

Projet   : CHANGELOG ✓ · CODEMAP ✓ · TODOS ✓ · docs/ +[N] ~[N] · archives/ [N]
Mémoire  : MEMORY.md [N corrigés, N supprimés, N déplacés vers le projet] · USER.md [ce qui bouge]
Skills   : [N créés, N fusionnés, N supprimés] ou « rien touché »
Reste    : [2 ou 3 items, pointeur vers TODOS.md]

Prêt pour /new ✅
```

**Commit** : jamais sans demande explicite. Tu proposes, tu attends.

---

## Mode REPRISE

Dans cet ordre, et pas plus.

1. `CLAUDE.md` ou `AGENTS.md` du projet, pour les règles.
2. `TODOS.md`, le bloc d'état de reprise **d'abord**.
3. `CODEMAP.md`, pour localiser et pour les zones sensibles.
4. `CHANGELOG.md`, les 2 ou 3 dernières entrées seulement.
5. Les fiches `docs/` des seuls modules concernés par la tâche à venir.
6. `git log --oneline -10` et `git status -s`, pour vérifier que le dépôt dit la même chose.

Ta mémoire et ton `USER.md` sont déjà dans ton prompt, ne les relis pas depuis le disque.

**Ne lis pas `archives/`.** C'est du faux par construction.

Restitue en dix lignes maximum : où on en est, ce qui est en cours, la prochaine étape, les pièges.
Signale tout de suite une incohérence entre la doc et le dépôt, c'est le symptôme d'une clôture
bâclée et ça se corrige avant de coder.

---

## Invariants, jamais négociables

- ❌ **Aucune suppression silencieuse** dans le projet. `archives/` avec la raison.
- ❌ **Aucune affirmation connue-fausse** laissée dans les piliers, dans `MEMORY.md` ou dans `docs/`.
- ❌ **Aucune doc écrite depuis une supposition.**
- ❌ **Aucune modification de `SOUL.md`** sans demande explicite.
- ❌ **Aucun fait de projet rangé dans `MEMORY.md`**, ni l'inverse.
- ❌ **Aucun skill supprimé en silence.** Tu le dis.
- ❌ **Aucun commit ou push non demandé.** Aucun secret dans un fichier tracké ou publié.
- ❌ **Aucun texte qui sent le généré.** Voir `style-redaction`.

---

## Références, à charger au besoin

| Fichier | Quand le lire |
|---|---|
| `references/CONVENTION.md` | Spécification des 3 piliers, ce qui y va et ce qui n'y va pas |
| `references/DOCS-SYSTEM.md` | Organisation de `docs/`, critère « mérite une fiche », en-têtes |
| `references/ARCHIVAGE.md` | Archiver, corriger ou supprimer, format d'en-tête, index |
| `references/PROFILS-PROJETS.md` | Adapter la convention au type de projet |
| `references/MEMOIRE-AGENT.md` | Frontière entre projet, `MEMORY.md`, `USER.md` et `SOUL.md` |
| `templates/` | Gabarits des piliers, des fiches `docs/`, des index |
| `scripts/ctx-init.sh` | Scaffolder la convention, sans rien écraser |
| `scripts/ctx-audit.sh` | Rapport de dérive entre la doc et le dépôt |
