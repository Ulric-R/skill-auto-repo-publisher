---
name: Auto Repo Publisher
description: Automatiser tout le cycle de publication d'un dépôt GitHub — proposer des noms disponibles, vérifier les prérequis (git, gh, SSH), préparer le contenu sans secrets, créer le dépôt, pousser et vérifier. À utiliser quand on demande de publier, créer un dépôt, mettre sur GitHub ou pousser un projet pour la première fois.
---

# Auto Repo Publisher

Objectif : de « ce projet n'est pas sur GitHub » à « dépôt publié et
vérifié » en une seule passe, sans étape oubliée et sans secret publié.

## Sécurité non négociable

- **Aucun secret ni valeur `.env` commité** — vérifier avant tout add :
  `git status`, `git diff`, et chercher `.env` réel / clés en dur.
- **Jamais de force-push** ni de suppression de dépôt sans accord
  explicite.
- **Visibilité toujours demandée** : ne jamais choisir seul
  public/privé — présenter le choix à l'utilisateur avant création.
- Le dépôt local reste la source de vérité : rien d'irréversible
  côté local.

## Workflow

### 1. Prérequis (auto-diagnostic)

Vérifier dans l'ordre et réparer si possible :

| Vérification | Commande | Si KO |
| --- | --- | --- |
| git | `git --version` | demander l'installation |
| gh installé | `gh --version` | installer le binaire dans `~/.local/bin` (sans sudo) |
| gh authentifié | `gh auth status` | lancer `gh auth login --hostname github.com --git-protocol ssh --web` en arrière-plan, donner le code device à l'utilisateur |
| SSH fonctionnel | `ssh -T git@github.com -o BatchMode=yes` | si `Permission denied` : `gh ssh-key add ~/.ssh/id_ed25519.pub -t ...` (scope `admin:public_key` via `gh auth refresh -s admin:public_key`, nouveau code device), sinon demander l'ajout manuel sur github.com/settings/keys |
| Identité git | `git config user.name` + `user.email` | demander ou déduire du compte gh |

Ne jamais présumer qu'auth/SSH fonctionnent : **tester à chaque session**.

### 2. Préparer le contenu

1. Initialiser si besoin : `git init -b main` (ou adopter la branche existante).
2. **Nettoyage sécurité** : lister les fichiers candidats au commit ;
   exclure `.env`, clés, `node_modules/`, `dist/`, credentials.
   Proposer/compléter un `.gitignore` fiable.
3. **README** : s'il manque ou est vide, proposer le skill `git-readme`
   (sinon l'utilisateur peut le demander).
4. Commits propres (lien `git-discipline`) : message clair, un = une
   intention, `git add` ciblé — jamais `git add .` aveugle.
5. Si déjà remote configuré → signaler que le dépôt est déjà lié,
   ne pas récréer.

### 3. Proposer des noms

- Dérivation : nom du dossier/paquet → kebab-case, court, explicite
  (ex. : dossier `Gestion_Textile` → `gestion-textile`).
- **Tester la disponibilité** de chaque proposition :
  `gh repo view <nom>` → existe = KO, on recommence.
- Proposer **3 options** (principal + 2 alternatives), avec
  disponibilité réelle vérifiée — jamais un nom annoncé disponible
  sans test.
- Ajouter un préfixe si besoin (ex. : `skill-` pour des skills),
  selon l'usage de l'utilisateur.

### 4. Demander à l'utilisateur (obligatoire)

Avant toute création, confirmer en une seule question :
- **Nom** retenu parmi les propositions (ou le sien)
- **Visibilité** : public / privé (toujours demandé, jamais deviné)
- **Description** proposée (pitch du README) — valide ou corriger

### 5. Créer et pousser

```bash
gh repo create <nom> --public|--private \
  --source <chemin-local> --remote origin \
  --description "<description>"
git push -u origin main
```

- Préférer `gh repo create --source … --push` quand tout est prêt
  (création + remote + push en une passe).
- Repo déjà créé sans remote local → `git remote add origin git@github.com:<compte>/<nom>.git`
  puis push.
- **Erreurs connues** (voir `references/commands.md`) :
  - `Name already exists` → le repo existe déjà : vérifier si c'est le
    sien (`gh repo view`) et basculer sur l'ajout du remote, ou renommer
  - Erreur GraphQL transitoire → **réessayer une fois** (la création a
    parfois réussi côté serveur — vérifier avec `gh repo view` avant de
    rejouer)
  - Refus SSH → re-vérifier la clé (étape 1)

### 6. Vérifier et rapporter

- **Preuve de push** : `git ls-remote origin` ou `gh repo view <nom> --json url`
  montre un hash distant réel — un push silencieux n'est pas une preuve.
- Vérifier que le contenu attendu est bien présent (fichiers listés
  côté distant : `gh api repos/<compte>/<nom>/contents` si doute).
- Rapport final : URL, visibilité, branche, nombre de fichiers/commits,
  et tout ce qui n'a **pas** été inclus (périmètre).

## Interdits

- Choisir la visibilité sans demander
- Commiter un secret « temporairement »
- Écraser un dépôt existant, renommer/supprimer sans accord
- Déclarer « publié » sans vérification distante (lien `anti-dream`)
- Publier un README absent si l'utilisateur s'attend à en avoir un —
  le signaler et proposer `git-readme`
