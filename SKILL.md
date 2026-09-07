---
name: cloture-agent
description: "Clôture de session pour un agent personnel permanent, Hermes Agent ou OpenClaw. À lancer avant un /new, un /reset ou un /clear, et à la fin de toute unité de travail. Synchronise les 3 piliers du projet CODEMAP.md TODOS.md CHANGELOG.md, purge et corrige la mémoire de l'agent dans MEMORY.md et USER.md, range les skills que l'agent s'est écrits tout seul, et déplace vers archives/ tout ce qui est devenu faux ou périmé, et tient chaque fichier du projet dans son budget de contexte : bloc de reprise borné, journal qui tourne, carte par zone sur les grosses codebases. Se déclenche sur : clôture ou fin de session, \"avant de faire /new\", \"range tout\", \"mets à jour ta mémoire\", \"fais le ménage dans tes skills\", \"documente ce qu'on a fait\", \"la doc gonfle\", \"le CODEMAP déborde\", budget de contexte, mémoire qui enfle, skills en doublon, doc vivante, CODEMAP, TODOS, CHANGELOG."
version: 2.0.0
author: fullya99
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [memory, skills, documentation, session, housekeeping, context-budget]
    related_skills: [plan]
---

# Clôture de session, pour un agent qui vit en continu

> **Le contrat** : à tout instant, ce qui est écrit sur le disque doit suffire. Un `/new` ne doit
> jamais coûter une information, et une mémoire qui grossit sans être relue finit par mentir. Le
> disque doit aussi rester lisible : une doc vraie mais trop longue est une doc perdue.

Tu tournes en service permanent, mais ta session se remet à zéro. Chez Hermes, `/new` ouvre une
session neuve, `/reset` en est un alias, `/clear` fait la même chose en nettoyant l'écran. Chez
OpenClaw, tu te réveilles à neuf à chaque session. Ce qui n'est pas écrit avant ce moment-là n'existe
plus après.

Une précision qui compte, parce qu'elle change ce que tu as à sauver. **Aucune de ces commandes ne
touche ta mémoire sur disque.** `MEMORY.md`, `USER.md` et `SOUL.md` sont là après comme avant. Ce que
la coupure emporte, c'est le contexte de conversation, donc tout ce qui n'a été que dit. La clôture ne
sert pas à protéger ta mémoire d'un `/new`, elle sert à ce que le disque contienne ce que la
conversation savait.

Deux risques, pas un. Le premier est de perdre du contexte à la coupure. Le second est propre aux
agents permanents : ta mémoire est réinjectée dans ton prompt à **chaque** appel, et rien ne la purge
tout seul. Une mémoire qui enfle coûte des tokens à chaque tour, et une mémoire fausse coûte bien
plus cher que ça.

**Avant d'écrire quoi que ce soit**, charge le skill de style s'il est installé. Il s'appelle
`style-redaction-agent` chez un agent permanent et `style-redaction` chez Claude Code. Sinon le
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
| Comment ce projet est fait, ce qui touche quoi | `CODEMAP.md` du projet, ou la carte de sa zone `<zone>/CODEMAP.md` pour un piège de zone |
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

## Zones et budget

Sur un gros projet, un répertoire de code trop volumineux pour le CODEMAP racine reçoit sa propre
carte, `<zone>/CODEMAP.md`. L'état et le journal restent uniques à la racine, une zone n'a jamais
son propre TODOS ni son propre CHANGELOG. Un `<zone>/CLAUDE.md`, facultatif, ne se charge tout
seul que chez Claude Code ou Codex (`AGENTS.md` au même endroit). Ailleurs, rien ne le charge : la
carte est donc la partie portable de la zone, celle qui compte sur toute plateforme.

Chaque fichier a un seuil de blocage. `CLAUDE.md` ou `AGENTS.md` racine, 100 lignes de cible.
CODEMAP racine, 250. Bloc d'état de reprise, 40. CHANGELOG, 30 entrées. Une fiche `docs/`, 200
lignes. Le détail complet, avec les seuils d'alerte, est dans `references/BUDGET.md`. Le critère
pour créer une zone, ce qu'elle porte et ce qui reste à la racine sont dans `references/ZONES.md`.

Le seuil est un détecteur, pas la règle. Le critère de découpe est sémantique, un fichier répond
à une question, c'est ce que disent Diátaxis et DITA qui ne fixent aucune longueur maximale.

---

## Mode INIT

Installer la convention sans rien écraser. Le terrain n'est jamais le même projet nu deux fois, le
script le détecte et l'annonce avant d'écrire. Le premier geste dépend de ce qu'il trouve.

| Ce que tu trouves | Ce que ça veut dire | Le premier geste |
|---|---|---|
| Rien, ou presque rien | Projet neuf, rien à greffer | Le kit se pose, tu remplis depuis le code réel |
| `doc/`, `documentation/`, `adr/` | Une doc rangée autrement | Le script l'adopte et déclare les vrais chemins, il ne pose pas de gabarits vides à côté |
| `AGENTS.md` sans `CLAUDE.md` | `AGENTS.md` est déjà le porteur des règles | Le script y écrit, c'est lui qui fait foi |
| `memory-bank/`, `.clinerules`, `.cursor/rules` | Une autre convention d'agent, deux sources de vérité | Une seule question, qui fait foi. Celle qui perd est archivée |
| `MEMORY.md` de projet | Version antérieure de cette convention | Le contenu se répartit chez les propriétaires. À ne pas confondre avec TON `MEMORY.md` d'agent, qui vit hors du dépôt |
| Documentation déjà abondante | Gros projet déjà obèse | La greffe est une clôture, pas une installation. Pose le kit, mesure, va en phase 6 |

Le script détecte le terrain et l'annonce avant d'écrire. Il ne touche jamais un fichier existant.

```bash
bash "$SKILL/scripts/ctx-init.sh" --dry-run   # voir ce qui serait créé
bash "$SKILL/scripts/ctx-init.sh"             # créer, sans jamais écraser
```

`$SKILL` est le répertoire de ce skill. Tu travailles depuis le projet, pas depuis le skill, donc un
chemin relatif ne résout pas. Résous-le une fois :

```bash
SKILL="$(for d in \
  $(find "$HOME/.hermes/skills" "$HOME/.openclaw" -maxdepth 4 -type d -name cloture-agent 2>/dev/null) \
  $(find skills -maxdepth 2 -type d -name cloture-agent 2>/dev/null) \
  "$HOME/.claude/skills/cloture-agent"; do
  [ -f "$d/scripts/ctx-audit.sh" ] && echo "$d" && break
done)"
[ -n "$SKILL" ] || echo "skill introuvable, fais les etapes a la main sans le script"
```

Le premier `find` couvre Hermes, `~/.hermes/skills/` avec ou sans catégorie, et OpenClaw à son
emplacement par défaut. Le second est relatif et sert quand le workspace OpenClaw a été déplacé par
`agents.defaults.workspace` : ce workspace est le seul répertoire de travail de l'agent, donc
`skills/<nom>/` y résout depuis le cwd quel que soit son emplacement réel.

Le script pose des gabarits vides et mesure la taille du projet en fichiers. Le travail commence
après. Lis le projet avant d'écrire, arborescence, points d'entrée, fichier de build, CI, derniers
commits. Ne remplis jamais le CODEMAP depuis une supposition. Le détail est dans
`references/CONVENTION.md`.

**Sur un grand projet, plus de 600 fichiers**, pose une carte par zone dès l'INIT plutôt que
d'attendre que le CODEMAP racine déborde :

```bash
bash "$SKILL/scripts/ctx-audit.sh" --budget      # liste les zones candidates
bash "$SKILL/scripts/ctx-init.sh" --zone <dir>   # pose la carte de chacune
```

---

## Mode CLÔTURE

### Phase 0. Cadrer et mesurer

```bash
git status -s && git diff --stat && git log --oneline -10
bash "$SKILL/scripts/ctx-audit.sh" --budget
```

Repère ce qui a changé, y compris hors git. Calibre l'effort sur la taille de la session, mais ne
saute ni la phase 3 ni la phase 4.

La section 10 de l'audit dit ce qui déborde. Note-le, la phase 6 le traite. Un `[!!]` ne se
reporte pas à la clôture suivante.

**Puis repère tous les niveaux.** Tu tournes en continu et tu passes d'un projet à l'autre sans
frontière visible, donc c'est ici que ça se perd le plus facilement. Une session peut avoir touché
un monorepo et deux de ses sous-projets, ou trois dépôts sans rapport.

```bash
find . -mindepth 2 -name CODEMAP.md -not -path './.git/*' -not -path './archives/*' \
  -not -path './node_modules/*' | sed 's|/CODEMAP.md$||'
```

Chaque dossier qui porte les trois piliers est un niveau **autonome**. Il se clôture séparément, avec
son propre passage en phases 1, 2 et 5. Le parent **pointe** vers lui, il ne l'absorbe pas, et
recopier la doc d'un sous-projet chez son parent est exactement l'erreur à ne pas faire.

L'audit de la phase 1 le signale de lui-même en section « Niveaux imbriqués », et il prévient aussi
quand tu l'as lancé depuis un sous-projet alors qu'il a audité le parent. Sans `--root`, la racine se
déduit de git, donc elle remonte au dépôt entier.

Un dossier qui ne porte qu'un `CODEMAP.md` est une **zone**, pas un niveau. Elle se met à jour dans
cette même clôture, elle ne se clôture pas à part. La section 1b de l'audit liste les zones à côté
des niveaux.

Ne documente jamais que le répertoire d'où tu es parti.

### Phase 1. Auditer le réel, sans croire la doc

```bash
bash "$SKILL/scripts/ctx-audit.sh"                      # le niveau courant
bash "$SKILL/scripts/ctx-audit.sh" --root packages/api  # un niveau imbriqué
```

**Une passe par niveau touché**, pas une seule à la racine. Le script audite un niveau à la fois, et
sans `--root` il remonte à la racine git.

Le script signale, il ne juge pas. Vérifie sur le terrain ce que la session a touché. Le service
tourne-t-il, le chemin cité existe-t-il encore, le test passe-t-il.

Note chaque écart sous la forme « c'est écrit X, la réalité est Y ». Chaque écart se termine par une
correction ou un archivage. Aucun ne reste ouvert.

### Phase 2. Les 3 piliers du projet

| Pilier | Ce que tu y fais |
|---|---|
| `CHANGELOG.md` | Entrée du jour en haut. Quoi, pourquoi, comment défaire. **25 lignes au plus.** |
| `CODEMAP.md` | La structure réelle. **Retire ce qui n'existe plus.** Un piège va dans la carte de sa zone ou la fiche du module, aux zones sensibles racine seulement s'il n'a ni l'un ni l'autre |
| `TODOS.md` | Coche le fait puis **supprime-le**, supprime l'obsolète, ajoute les dettes trouvées, puis réécris le bloc d'état de reprise |

Le bloc de reprise tient en 40 lignes. Il se **réécrit depuis l'état courant**, jamais en éditant
le bloc précédent. Ce qu'il en sort a un propriétaire, classe avant de retirer : un événement est
déjà au CHANGELOG, un piège va dans la carte de sa zone ou la fiche du module, une tâche dans la
liste avec son ancrage, une décision en ADR. Le récit, l'insistance, les majuscules ne se gardent
pas. Rien de ce qui est dans TODOS ne raconte une session, pas de section « ce que cette session a
produit », c'est du CHANGELOG.

**Applique ce tableau à chaque niveau touché**, repéré en phase 0. Un sous-projet a sa propre doc
vivante et son propre bloc d'état de reprise. Le parent gagne au mieux une ligne qui pointe vers lui.

### Phase 3. Ta mémoire, phase propre aux agents permanents

Elle est réinjectée à chaque appel. Relis-la en entier, à chaque clôture.

1. **Corrige ce qui est devenu faux.** Une affirmation fausse dans `MEMORY.md` te fera te tromper à
   chaque session, silencieusement. C'est le pire état possible.
2. **Supprime ce qui n'a plus d'usage.** Un fait qui ne servira plus jamais coûte des tokens à
   chaque tour. Suppression pure, pas d'archivage : `MEMORY.md` n'est pas un journal.
3. **Déplace ce qui appartient au projet** vers le dépôt concerné, selon le tableau des trois
   propriétaires.
4. **Fusionne les doublons.** Deux formulations d'un même fait vont finir par diverger, et tu ne
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

Fiches, procédures, scripts, plans abandonnés. Le critère porte sur les références entrantes, pas
sur une impression. Pour chaque référence : est-elle opérative, quelqu'un a besoin du document
pour agir maintenant, ou est-ce du contexte historique. On archive quand toutes les références
restantes sont historiques. Un lien markdown vivant bloque l'archivage.

```bash
bash "$SKILL/scripts/ctx-archive.sh" --verifier <chemin>
bash "$SKILL/scripts/ctx-archive.sh" <chemin> --raison "..." --remplace-par "..."
```

Le premier appel n'écrit rien, il classe les références et dit si un lien vivant bloque. Le second
déplace par `git mv`, pose l'en-tête, résout les liens du fichier déplacé, ajoute la ligne d'index.
Sa dernière étape est une vérification : elle sort en erreur s'il reste une occurrence de l'ancien
chemin hors des archives.

Une section fausse dans un fichier encore utile se corrige sur place, pas de déplacement. Une règle
devenue fausse dans `CLAUDE.md` ou `AGENTS.md` se supprime, une règle erronée est pire qu'une règle
absente. Un pilier ne s'archive pas, le script le refuse. Le détail est dans
`references/ARCHIVAGE.md`.

**Aucune suppression silencieuse.** Aucune affirmation connue-fausse laissée pour plus tard.

### Phase 6. Tenir le budget

```bash
bash "$SKILL/scripts/ctx-audit.sh" --budget
```

Chaque ligne en `[!]` se traite maintenant, ou reçoit une tâche datée dans TODOS, une seule fois.
Une alerte qui revient à la clôture suivante est un blocage. Chaque ligne en `[!!]` se traite
maintenant, la clôture n'est pas finie avant.

| Ce qui déborde | Le geste |
|---|---|
| bloc de reprise | réécrire depuis l'état courant, classer ce qui sort (phase 2) |
| TODOS.md | supprimer les cases cochées, déplacer les sections de session au CHANGELOG, dégraisser les tâches sans ancrage |
| CHANGELOG.md, ou tout journal à entrées datées | `bash "$SKILL/scripts/ctx-rotation.sh" --fichier <journal>`, puis raccourcir les entrées de plus de 25 lignes en sortant le détail vers une fiche |
| CODEMAP.md racine | créer les zones candidates avec `ctx-init.sh --zone`, y descendre leurs zones sensibles et leurs commandes, laisser un pointeur |
| CLAUDE.md ou AGENTS.md | sans `clawd-audit-md`, qui n'existe que chez Claude Code, la procédure de `references/BUDGET.md` : l'état va dans TODOS, le récit dans la fiche, la règle de zone dans `<zone>/CLAUDE.md`, chaque ligne restante passe le test « la retirer ferait-il faire une erreur ? » |
| une fiche `docs/` | scinder en deux sujets, ou sortir vers `archives/` le récit qui ne sert plus à décider |
| une carte de zone | scinder la zone en deux |

La rotation se lance en `--dry-run` d'abord, tu lis ce qu'elle annonce, puis pour de vrai. Elle
n'archive pas du faux, elle range du froid : les entrées gardent leur valeur, elles ne coûtent plus
rien à la reprise.

Ta mémoire d'agent a le même problème que ces fichiers. Sa phase 3 est son budget, relue en entier
à chaque clôture pour la même raison : une mémoire qui grossit sans être coupée coûte des tokens à
chaque tour, qu'elle serve ou non.

### Phase 7. Test « prêt pour /new »

- [ ] **Chaque niveau touché** a eu sa passe, pas seulement celui d'où tu es parti.
- [ ] Les 3 piliers reflètent l'état vérifié en phase 1, sans se contredire.
- [ ] Le bloc d'état de reprise de `TODOS.md` permet de repartir sans cette conversation.
- [ ] Tout travail en cours non fini a une tâche écrite.
- [ ] `MEMORY.md` relu en entier, rien de faux, rien d'inutile, rien qui appartienne au projet.
- [ ] `USER.md` à jour des préférences et des corrections de cette session.
- [ ] Les skills touchés sont propres, sans doublon, sans secret.
- [ ] Aucun secret dans un fichier tracké. `git status` conforme à l'intention.
- [ ] `ctx-audit.sh --budget` ne rend aucun `[!!]`, chaque `[!]` a son geste ou sa tâche.
- [ ] Chaque zone touchée a sa carte à jour et est citée dans le CODEMAP racine.

### Phase 8. Compte-rendu, puis commit si on te le demande

```
**Clôture, AAAA-MM-JJ**

Fait : [1 à 3 lignes factuelles]

Niveaux  : [chemin de chaque niveau clôturé, un par un]
Projet   : CHANGELOG ✓ · CODEMAP ✓ · TODOS ✓ · docs/ +[N] ~[N] · archives/ [N]
Mémoire  : MEMORY.md [N corrigés, N supprimés, N déplacés vers le projet] · USER.md [ce qui bouge]
Skills   : [N créés, N fusionnés, N supprimés] ou « rien touché »
Budget   : [ce qui a été réduit, tourné ou découpé] ou « tout tient »
Reste    : [2 ou 3 items, pointeur vers TODOS.md]

Prêt pour /new ✅
```

**Commit** : jamais sans demande explicite. Tu proposes, tu attends.

---

## Mode REPRISE

Dans cet ordre, et pas plus.

1. `CLAUDE.md` ou `AGENTS.md` du projet, pour les règles.
2. Le bloc d'état de reprise de `TODOS.md`, **d'abord et rien d'autre** :

   ```bash
   awk '/^## /{n++} n<=1' TODOS.md
   ```
3. Le plan de `CODEMAP.md`, puis la seule section utile, puis la carte de la zone concernée :

   ```bash
   grep -n '^## ' CODEMAP.md
   ```
4. Les 2 ou 3 dernières entrées de `CHANGELOG.md`, pas tout l'historique :

   ```bash
   awk '/^## /{n++} n<=3' CHANGELOG.md
   ```
5. Les fiches `docs/` des seuls modules concernés par la tâche à venir.
6. `git log --oneline -10` et `git status -s`, pour vérifier que le dépôt dit la même chose.

Tu ne lis jamais un pilier en entier. Un pilier qui déborde se signale, il ne se lit pas jusqu'au
bout pour compenser.

Ta mémoire et ton `USER.md` sont déjà dans ton prompt, ne les relis pas depuis le disque.

**Ne lis pas `archives/`.** C'est du faux par construction. On n'y va que sur demande explicite
d'archéologie, et jamais par un `grep` nu : il rend une ligne sans son en-tête, et tu prends alors
du froid pour du courant. `bash "$SKILL/scripts/ctx-archeo.sh" <motif>` rend la provenance avant
l'extrait, avec une sortie bornée.

Restitue en dix lignes maximum : où on en est, ce qui est en cours, la prochaine étape, les pièges.
Signale tout de suite une incohérence entre la doc et le dépôt, c'est le symptôme d'une clôture
bâclée et ça se corrige avant de coder.

---

## Invariants, jamais négociables

- ❌ **Aucune suppression silencieuse** dans le projet. `archives/` avec la raison.
- ❌ **Aucune affirmation connue-fausse** laissée dans les piliers, dans `MEMORY.md` ou dans `docs/`.
- ❌ **Aucune doc écrite depuis une supposition.**
- ❌ **Aucun `archives/` lu en reprise.** Toute fouille passe par `ctx-archeo.sh`, qui rend la
  provenance avec l'extrait.
- ❌ **Aucune modification de `SOUL.md`** sans demande explicite.
- ❌ **Aucun fait de projet rangé dans `MEMORY.md`**, ni l'inverse.
- ❌ **Aucun skill supprimé en silence.** Tu le dis.
- ❌ **Aucun commit ou push non demandé.** Aucun secret dans un fichier tracké ou publié.
- ❌ **Aucun fichier au-dessus de son seuil de blocage** à la fin d'une clôture. Ce qui déborde
  descend d'un étage, il ne s'efface pas.
- ❌ **Aucun journal de session dans TODOS, aucun état ni récit dans `CLAUDE.md` ou `AGENTS.md`.**
- ❌ **Aucun texte qui sent le généré.** Voir `style-redaction`.

---

## Références, à charger au besoin

| Fichier | Quand le lire |
|---|---|
| `references/CONVENTION.md` | Spécification des 3 piliers, ce qui y va et ce qui n'y va pas |
| `references/BUDGET.md` | Les seuils, leur pourquoi, le geste par fichier qui déborde |
| `references/ZONES.md` | Quand un répertoire mérite sa carte, ce qu'elle porte, ce qui reste à la racine |
| `references/DOCS-SYSTEM.md` | Organisation de `docs/`, critère « mérite une fiche », en-têtes |
| `references/ARCHIVAGE.md` | Archiver, corriger ou supprimer, format d'en-tête, index |
| `references/PROFILS-PROJETS.md` | Adapter la convention au type de projet |
| `references/MEMOIRE-AGENT.md` | Frontière entre projet, `MEMORY.md`, `USER.md` et `SOUL.md` |
| `templates/` | Gabarits des piliers, des cartes de zone, des fiches `docs/`, des index |
| `scripts/ctx-init.sh` | Scaffolder la convention ou une zone, sans rien écraser |
| `scripts/ctx-audit.sh` | Rapport de dérive entre la doc et le dépôt, et mesure du budget avec `--budget` |
| `scripts/ctx-rotation.sh` | Faire tourner un journal vers `archives/`, `--dry-run` d'abord |
| `scripts/ctx-archive.sh` | Archiver un document en une passe : références entrantes, déplacement, en-tête, liens, index, vérification |
| `scripts/ctx-archeo.sh` | Le seul chemin sanctionné pour fouiller `archives/`, provenance avant extrait, sortie bornée |
