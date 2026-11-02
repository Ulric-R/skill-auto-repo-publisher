# Auto Repo Publisher

> Skill OpenCode qui automatise tout le cycle de publication d'un dépôt GitHub — prérequis, noms disponibles, préparation sans secrets, création, push et vérification.

**ID** : `auto-repo-publisher` · **Portée** : globale (`~/.config/opencode/skills/`) · **Langue** : FR

## Ce que fait le skill

De « ce projet n'est pas sur GitHub » à « dépôt publié et vérifié » :

1. **Auto-diagnostic** — `git`, `gh` installé, `gh auth`, SSH, identité ; réparation proposée (device flow, ajout de clé via `gh ssh-key add`)
2. **Préparation** — `git init -b main`, `.gitignore`, exclusion stricte des secrets (`.env` jamais commité), README via `git-readme`, commits propres (`git-discipline`)
3. **Noms proposés** — 3 options en kebab-case, **disponibilité réellement testée** (`gh repo view`)
4. **Confirmation obligatoire** — nom + **visibilité toujours demandée** + description, en une question
5. **Création + push** — `gh repo create … --source … --push`, gestion des erreurs connues
6. **Vérification** — preuve distante réelle (`git ls-remote`, hash) avant tout « publié »

## Sécurité non négociable

- Jamais de secret ni de valeur `.env` commité
- **Visibilité toujours demandée** — jamais devinée
- Aucun force-push ni suppression sans accord
- « Publié » dit seulement après vérification distante

## Structure

```text
auto-repo-publisher/
├── SKILL.md                    # Workflow 6 étapes + interdits
└── references/
    └── commands.md             # Commandes exactes + table de dépannage
```

## Déclenchement

- **Automatique** : demande de publier / créer un dépôt / pousser sur GitHub pour la première fois
- **Manuel** : `@auto-repo-publisher`

## Dépannage intégré

`Name already exists` → vérifier puis rattacher le remote · erreur GraphQL transitoire → réessayer après vérification · `Permission denied` → ajouter la clé · scope manquant → `gh auth refresh`

## Skills liés

- `git-readme` (README manquant), `git-discipline` (commits), `pre-ship-review` (check avant push), `anti-dream` (aucune affirmation sans preuve)
