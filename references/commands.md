# Commandes et dépannage — Auto Repo Publisher

Aide-mémoire des commandes réelles et de leurs pannes courantes.

## 1. Prérequis

```bash
git --version
gh --version                    # absent → télécharger le binaire gh dans ~/.local/bin
gh auth status                   # non connecté → login device flow
ssh -T git@github.com -o BatchMode=yes   # Permission denied → ajouter la clé
git config --get user.name; git config --get user.email
```

### Login gh (device flow, non interactif)

```bash
yes '' | gh auth login --hostname github.com --git-protocol ssh --web
# → affiche un code à donner à l'utilisateur : https://github.com/login/device
```

### Ajouter la clé SSH via gh

```bash
gh auth refresh -h github.com -s admin:public_key   # nouveau code device
gh ssh-key add ~/.ssh/id_ed25519.pub -t "machine-$(hostname)"
ssh -T git@github.com -o BatchMode=yes              # doit répondre « Hi <user>! »
```

### Installer gh sans sudo

```bash
# Télécharger depuis https://github.com/cli/cli/releases (tar.gz linux_amd64)
# puis : tar -xzf gh_*.tar.gz && cp gh_*/bin/gh ~/.local/bin/gh
```

## 2. Préparation

```bash
git status --short                 # fichiers candidats
git check-ignore -v .env           # vérifier que .env est ignoré
git init -b main                   # si pas de dépôt
git add <fichiers ciblés>          # jamais `git add .` aveugle
git commit -m "feat: initialiser le projet"
```

## 3. Noms disponibles

```bash
gh repo view <nom> >/dev/null 2>&1 && echo "PRISE" || echo "LIBRE"
# Tester chaque proposition avant de la proposer
```

## 4. Création + push

```bash
# Une passe (création + remote + push) :
gh repo create <nom> --public --source . --remote origin \
  --description "<pitch>" --push

# Ou séquencé :
gh repo create <nom> --public --source . --remote origin --description "<pitch>"
git push -u origin main
```

## 5. Dépannage

| Symptom | Cause | Remède |
| --- | --- | --- |
| `Name already exists` | nom pris (parfois créé à un essai précédent) | `gh repo view <nom>` → si c'est le sien : `git remote add origin git@github.com:<compte>/<nom>.git` + push ; sinon renommer |
| `GraphQL: Something went wrong` | erreur transitoire API | **réessayer une fois** ; vérifier d'abord si le repo existe (`gh repo view`) — il a parfois été créé |
| `Permission denied (publickey)` | clé absente du compte | `gh ssh-key add` (étape 1) ou ajout manuel sur github.com/settings/keys |
| `HTTP 404 … admin:public_key` | scope manquant | `gh auth refresh -s admin:public_key` |
| `Repository not found` au push | compte/remote erroné | vérifier `git remote -v` et le compte (`gh auth status`) |
| Push refusé (protected branch) | règle de branche | vérifier les protections, ou pousser sur une autre branche après accord |
| `remote origin already exists` | remote déjà configuré | `git remote set-url origin <url>` si mauvais, sinon ne rien changer |

## 6. Vérification (preuve)

```bash
git ls-remote origin | head -3        # hash distant réel
gh repo view <nom> --json url,visibility,defaultBranchRef
gh api repos/<compte>/<nom>/contents --jq '.[].name'   # fichiers côté distant
```

## 7. Cas particuliers

- **Repo existant à mettre à jour** : pas de création — `git push`
  seulement (et `git pull --rebase` si distant a bougé).
- **Premier push d'un dépôt ancien** : vérifier `.gitignore` avant,
  l'historique complet part.
- **Monorepo** : un seul dépôt à la racine, pas de sous-push.
