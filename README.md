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

Le skill devient une slash command au passage, Hermes expose les skills installés comme commandes.

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
bash scripts/ctx-init.sh --dry-run   # voir ce qui serait créé
bash scripts/ctx-init.sh             # créer, sans jamais écraser un fichier existant
bash scripts/ctx-audit.sh            # rapport de dérive entre la doc et le dépôt
bash scripts/ctx-audit.sh --strict   # sortie non nulle s'il reste une alerte, pour la CI
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
