# Synchronisation upstream -> fork et workflow release

Ce document décrit une stratégie propre et durable pour ce fork :

- `upstream/main` : source officielle (repo public),
- `origin/main` : miroir read-only de `upstream/main`,
- `origin/cytadel-release` : branche de release de ton fork,
- `origin/feature/*` : branches d'évolution.

---

## 1) Préparation et contrôles de base

```bash
cd /Users/sylvain/git/ansible-roles/CytadelHosting.multi-php
git status
git remote -v
```

Vérifier que :

- `origin` pointe sur ton fork,
- `upstream` pointe sur le projet source.

---

## 2) Configurer ou corriger le remote `upstream`

Si `upstream` n'existe pas :

```bash
git remote add upstream https://github.com/lukasic/ansible-role-multi-php.git
```

Si `upstream` existe déjà mais est incorrect :

```bash
git remote set-url upstream https://github.com/lukasic/ansible-role-multi-php.git
```

Contrôle :

```bash
git remote -v
```

---

## 3) Récupérer les dernières données

```bash
git fetch origin --prune --tags
git fetch upstream --prune --tags
```

Contrôle optionnel des tags :

```bash
git tag -l "*1.4.0*"
git tag -l "*1.8.1*"
```

---

## 4) Renommer la branche release en `cytadel-release`

Etat cible :

- `main` reste read-only (miroir upstream),
- `cytadel-release` devient ta branche release active.

### 4.1 Renommer localement + publier

```bash
git checkout minimal-config-improvements
git pull --ff-only origin minimal-config-improvements
git branch -m minimal-config-improvements cytadel-release
git push -u origin cytadel-release
```

### 4.2 Basculer la branche par défaut GitHub

Dans GitHub :

- `Settings` -> `Branches` -> `Default branch` -> `cytadel-release`.

### 4.3 Nettoyage de l'ancienne branche distante (optionnel)

Après avoir changé la branche par défaut :

```bash
git push origin --delete minimal-config-improvements
git fetch origin --prune
git remote set-head origin -a
```

---

## 5) Synchroniser `origin/main` avec `upstream/main` (miroir read-only)

Cette étape met à jour uniquement le miroir.

```bash
git fetch upstream --prune --tags
git fetch origin --prune --tags

git checkout main
git pull --ff-only origin main
git merge --ff-only upstream/main
git push origin main
```

Notes :

- `--ff-only` garantit qu'aucun commit local parasite n'est ajouté au miroir.
- Si ça échoue, il faut d'abord analyser pourquoi `origin/main` a divergé.

---

## 6) Intégrer les nouveautés upstream dans `cytadel-release`

Deux options, selon la politique d'historique.

### Option A (historique linéaire, recommandé)

```bash
git checkout cytadel-release
git pull --ff-only origin cytadel-release
git rebase main
git push --force-with-lease origin cytadel-release
```

### Option B (sans réécriture d'historique)

```bash
git checkout cytadel-release
git pull --ff-only origin cytadel-release
git merge main
git push origin cytadel-release
```

---

## 7) Process d'évolution avec branches de feature

### 7.1 Créer une branche de feature depuis la release

```bash
git checkout cytadel-release
git pull --ff-only origin cytadel-release
git checkout -b feature/<nom-court-feature>
```

Exemples :

- `feature/php85-packages`
- `feature/fpm-pool-hardening`

### 7.2 Développer et pousser la feature

```bash
git add .
git commit -m "feat: <description courte>"
git push -u origin feature/<nom-court-feature>
```

### 7.3 Recaler la feature avant merge

```bash
git checkout feature/<nom-court-feature>
git fetch origin
git rebase origin/cytadel-release
```

Si conflit :

```bash
git add <fichier_corrige>
git rebase --continue
```

Annuler si nécessaire :

```bash
git rebase --abort
```

### 7.4 Merge de la feature vers `cytadel-release`

```bash
git checkout cytadel-release
git pull --ff-only origin cytadel-release
git merge --ff-only feature/<nom-court-feature>
git push origin cytadel-release
```

### 7.5 Nettoyer la branche feature après intégration

```bash
git branch -d feature/<nom-court-feature>
git push origin --delete feature/<nom-court-feature>
```

---

## 8) Vérifications utiles après synchro ou merge

```bash
git status -sb
git branch -vv
git log --oneline --decorate --graph --max-count=40
git diff --stat main...cytadel-release
```

---

## 9) Garde-fous importants

- Ne jamais développer directement sur `main`.
- Protéger `main` et `cytadel-release` dans GitHub (branch protection).
- Utiliser `--force-with-lease` uniquement après rebase maîtrisé.
- Faire une branche `backup/*` avant opérations sensibles.

Exemple :

```bash
git checkout cytadel-release
git branch backup/cytadel-release-$(date +%Y%m%d-%H%M)
```

---

## 10) Tagger une release

Tagger depuis `cytadel-release` uniquement, après validation.

```bash
git checkout cytadel-release
git pull --ff-only origin cytadel-release
git tag -a vX.Y.Z -m "Release vX.Y.Z"
git push origin vX.Y.Z
```

Vérifier localement et à distance :

```bash
git tag -l "v*"
git ls-remote --tags origin
```

Optionnel : pousser tous les tags locaux d'un coup.

```bash
git push origin --tags
```

