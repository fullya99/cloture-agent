# cloture-agent

Un skill de clôture de session pour agent personnel permanent. Il marche sur **Hermes Agent** et sur
**OpenClaw**, et il suit le standard [agentskills.io](https://agentskills.io/specification).

L'idée tient en une phrase : **si une information ne vit que dans la conversation, elle n'existe
pas.** Tu tournes en continu, mais ta session se remet à zéro. Ce qui n'est pas écrit avant un `/new`
a disparu après.

## Installation

**Hermes Agent**

```bash
git clone https://github.com/fullya99/cloture-agent.git \
  ~/.hermes/skills/housekeeping/cloture-agent
```

Le skill devient une slash command au passage, Hermes expose les skills installés comme commandes. Le
dossier de catégorie est du rangement, pas une condition, le scan est récursif. Les mises à jour se
font par `git pull`.

Pas besoin de redémarrer. L'outil de liste voit le skill tout de suite, l'index de skills du prompt
système est construit en début de session et ne l'affichera qu'à la suivante.

### L'installation officielle est impossible, et c'est normal

`hermes skills install` refuse ce skill, et ce n'est pas contournable. Vérifié le 2026-08-13 sur une
installation réelle :

```
Decision: BLOCKED - Blocked (community source + dangerous verdict, 27 findings).
--force does not override a dangerous verdict.
rules: agent_config_mod, hermes_config_mod
```

Ni `--force`, ni `--yolo`, ni l'ajout du dépôt aux dépôts de confiance ne lèvent un verdict
`dangerous`. Il n'y a aucun réglage utilisateur pour ça.

Le scanner ne se trompe pas. Ce skill dit à l'agent de relire et de corriger sa mémoire, et ses
scripts écrivent des fichiers de contexte. C'est exactement la signature qu'un scanner d'installation
cherche, parce que c'est la signature d'une attaque par empoisonnement de mémoire : l'entrée AST01 de
l'OWASP Agentic Skills Top 10 cite l'injection dans `MEMORY.md` comme le premier cas de skill
malveillant. En février 2026, l'audit ToxicSkills de Snyk a trouvé un défaut de sécurité dans 36% de
3 984 skills publics, et la campagne ClawHavoc a placé 341 skills malveillants sur ClawHub, soit 12%
du catalogue de l'époque.

Un skill de clôture qui range la mémoire de l'agent ne passera donc jamais une installation
officielle, sur aucune plateforme sérieuse. Le `git clone` ci-dessus est la voie normale, pas un
pis-aller : c'est un outil personnel qui touche à ta mémoire, tu l'installes en le lisant, pas en
faisant confiance à un catalogue.

Son voisin `style-redaction-agent`, qui n'écrit que du texte que tu lui demandes, passe le scan sans
problème et s'installe officiellement.

**OpenClaw**

```bash
git clone https://github.com/fullya99/cloture-agent.git \
  ~/.openclaw/workspace/skills/cloture-agent
```

Si ton workspace n'est pas à l'emplacement par défaut, regarde `agents.defaults.workspace` dans
`~/.openclaw/openclaw.json`.

**Claude Code**, si tu veux la même chose côté session

```bash
cp -r . ~/.claude/skills/cloture-agent
```

Il y a une version dédiée à Claude Code, `cloture-session`, avec ses trois slash commands. Celle-ci
apporte en plus le rangement de la mémoire de l'agent, qui n'existe pas là-bas.

## Ce que fait la 2.0.0

Une doc vraie mais trop longue est une doc perdue. Sur un projet réel de 979 fichiers, après 83
clôtures, le bloc de reprise faisait 2 579 lignes et le CHANGELOG 8 283. Rien ne poussait à
retirer. La 2.0.0 pose le budget comme second invariant, au même rang que le vrai.

- **Un budget par surface**, mesuré par `ctx-audit.sh --budget`. Une clôture n'est pas finie tant
  qu'un fichier dépasse son seuil de blocage.
- **Un bloc de reprise réécrit**, jamais édité, borné à 40 lignes. Ce qui en sort a un
  propriétaire : événement au CHANGELOG, piège dans une fiche, tâche dans TODOS.
- **Un journal qui tourne.** `ctx-rotation.sh` déplace les entrées anciennes du CHANGELOG vers
  `archives/`, par mois, sans rien coûter à la reprise.
- **Des cartes de zone** sur les gros projets. Un répertoire qui le mérite reçoit son propre
  `CODEMAP.md`, et au besoin un `CLAUDE.md` de zone que seuls Claude Code et Codex chargent tout
  seuls. Ailleurs, rien ne le charge, la carte est donc la partie portable de la zone.
- **Un chemin vers les archives.** `archives/` ne se lit jamais en reprise, mais sur demande
  explicite d'archéologie `ctx-archeo.sh` rend la provenance avant l'extrait, sortie bornée. Sans
  lui, ne jamais lire les archives reviendrait à perdre ce qu'on prétend garder.

| Surface | Cible |
|---|---|
| `CLAUDE.md` ou `AGENTS.md` racine | 100 lignes |
| `CODEMAP.md` racine | 250 lignes |
| bloc « État à la reprise » | 40 lignes |
| `CHANGELOG.md` | 30 entrées |

## Ce qu'il fait

**Côté projet**, il maintient trois fichiers à la racine du dépôt sur lequel tu travailles.

| Fichier | Question |
|---|---|
| `CODEMAP.md` | Où ça se trouve, et qu'est-ce qui touche quoi ? |
| `TODOS.md` | Qu'est-ce que je fais maintenant ? |
| `CHANGELOG.md` | Qu'est-ce qui a changé, et comment revenir en arrière ? |

Plus `docs/`, qui ne contient que du vrai, et `archives/`, qui absorbe tout le reste avec un en-tête
disant la date, l'origine, la raison et le remplaçant. Rien n'est supprimé en douce.

**Côté agent**, et c'est ce qui le distingue, il range ce que tu portes en propre.

`MEMORY.md` est réinjecté dans ton prompt à chaque appel, et rien ne le purge tout seul. Un fait
inutile s'y paie indéfiniment, un fait faux te fait te tromper indéfiniment et sans bruit. Le skill
te fait le relire en entier à chaque clôture, en quatre passes : corriger le faux, supprimer le
périmé, remonter dans le dépôt ce qui appartient au projet, fusionner les doublons.

`USER.md` reçoit les préférences exprimées et **chaque correction que l'utilisateur t'a faite**. Une
correction qui ne finit pas écrite se répétera.

`SOUL.md` ne bouge que sur demande explicite. Un agent qui réécrit sa propre identité à partir de ses
observations dérive, et la dérive est invisible depuis l'intérieur.

**Tes skills auto-créés** enfin. Tu en écris, tu les modifies, personne ne fait le ménage derrière
toi. Le skill traque quatre défauts : la description floue qui empêche le déclenchement, le doublon
qui rend le comportement imprévisible, le skill mort écrit pour un besoin ponctuel, et le secret
oublié dedans alors que les skills se publient.

## Plusieurs projets dans la même session

C'est le cas courant chez un agent permanent, qui passe d'un projet à l'autre sans frontière visible.
Le skill traite chaque niveau séparément.

Un dossier qui porte les trois piliers est un niveau **autonome**. Un monorepo avec deux sous-projets
équipés, ça fait trois clôtures, pas une. Le parent pointe vers ses enfants, il ne recopie pas leur
doc, et recopier est l'erreur exacte que la convention veut éviter.

L'audit le gère de deux façons. Sa section « Niveaux imbriqués » liste les dossiers en dessous qui
portent leur propre kit, avec la commande pour chacun. Et il prévient quand tu l'as lancé depuis un
sous-projet alors qu'il a audité le parent, ce qui arrive sans `--root` puisque la racine se déduit de
git.

## Se greffer sur un projet existant

L'INIT ne pose jamais de gabarits vides à côté d'une doc qui existe déjà. Il détecte le terrain et
l'annonce avant d'écrire.

| Ce que le projet a déjà | Ce que le skill en fait |
|---|---|
| Rien, ou presque rien | Le kit se pose, tu remplis depuis le code réel |
| `doc/`, `documentation/`, `adr/` | Il adopte et déclare les vrais chemins |
| `AGENTS.md` sans `CLAUDE.md` | Il y écrit, `AGENTS.md` reste le porteur des règles |
| `memory-bank/`, `.clinerules`, `.cursor/rules` | Une seule question tranche qui fait foi, l'autre part en archives |
| `MEMORY.md` de projet | Son contenu se répartit chez les propriétaires, décisions en ADR, pièges dans les fiches |
| Documentation déjà abondante | La greffe est une clôture, pas une installation |

Dans tous les cas il ne touche jamais un fichier existant.

## Les modes

| Mode | Quand |
|---|---|
| CLÔTURE | avant un `/new`, un `/reset`, ou à la fin d'une unité de travail |
| REPRISE | en début de session, pour retrouver le fil en dix lignes |
| INIT | une fois, pour installer les trois piliers dans un projet neuf |

Il détecte sa plateforme et ses chemins tout seul. Si aucune base d'agent n'est trouvée, il fait la
partie projet et dit qu'il saute la partie mémoire, plutôt que d'inventer des chemins.

## Les scripts, utilisables seuls

```bash
bash scripts/ctx-init.sh --dry-run    # voir ce qui serait créé
bash scripts/ctx-init.sh              # créer, sans jamais écraser un fichier existant
bash scripts/ctx-init.sh --zone <dir> # poser la carte d'une zone
bash scripts/ctx-audit.sh             # rapport de dérive entre la doc et le dépôt
bash scripts/ctx-audit.sh --budget    # mesurer le budget de chaque surface
bash scripts/ctx-audit.sh --strict    # sortie non nulle s'il reste une alerte, pour la CI
bash scripts/ctx-rotation.sh --dry-run   # voir ce que la rotation du CHANGELOG déplacerait
bash scripts/ctx-rotation.sh             # déplacer les entrées froides vers archives/
bash scripts/ctx-archive.sh --verifier <chemin>                        # classer les références entrantes
bash scripts/ctx-archive.sh <chemin> --raison "..." --remplace-par "..." # archiver en une passe
bash scripts/ctx-archeo.sh <motif>          # chercher dans archives/, provenance puis extrait
bash scripts/ctx-archeo.sh --liste          # inventaire de ce qui est archivé
bash scripts/ctx-archeo.sh --fichier <chemin> # provenance d'une archive précise
```

Bash et git suffisent. Aucune dépendance, aucun serveur MCP.

## Ce qu'il ne fait pas

Il ne résume pas ta session. Le but est de rendre le dépôt exact, pas de raconter ce qui s'est passé.

Il ne touche pas à `SOUL.md` sans qu'on le lui demande.

Il ne devine pas. Ce qu'il ne peut pas déduire, il le demande, parce qu'une carte fausse est pire que
pas de carte.

Il ne commite pas, ne pousse pas, ne supprime pas un skill sans le dire.

## Licence

MIT.
