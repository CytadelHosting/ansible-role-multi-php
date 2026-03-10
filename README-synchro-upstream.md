# Synchroniser le fork avec l'upstream et réintégrer les évolutions métier

Ce guide documente la procédure utilisée pour :

- récupérer les évolutions du dépôt source (`upstream/main`),
- rebaser la branche d'évolution locale dessus,
- pousser le résultat proprement sur `origin`.

Contexte de référence :

- branche d'évolution : `minimal-config-improvements`
- base historique de cette branche : tag `1.4.0`
- upstream actuel : tag `1.8.1` (et branche `main` à jour)

---

## 1) Se placer dans le dépôt et vérifier l'état

```bash
cd /Users/sylvain/git/ansible-roles/CytadelHosting.multi-php
git status
git remote -v
```

Objectif :

- confirmer que le dépôt est propre ou identifier les changements en cours,
- voir les remotes existants avant ajout de `upstream`.

---

## 2) Configurer le remote `upstream`

Si `upstream` n'existe pas encore :

```bash
git remote add upstream <URL_DU_REPO_SOURCE>
```

Si `upstream` existe déjà et doit être corrigé :

```bash
git remote set-url upstream <URL_DU_REPO_SOURCE>
```

Contrôle :

```bash
git remote -v
```

---

## 3) Récupérer les dernières données (branches + tags)

```bash
git fetch origin --prune --tags
git fetch upstream --prune --tags
```

Vérifier la présence des tags utiles :

```bash
git tag -l "*1.4.0*"
git tag -l "*1.8.1*"
```

---

## 4) Sauvegarder la branche d'évolution avant rebase

```bash
git checkout minimal-config-improvements
git branch backup/minimal-config-improvements-$(date +%Y%m%d-%H%M)
```

Cette sauvegarde permet un rollback rapide si nécessaire.

---

## 5) Rebase des commits métier sur `upstream/main` (Option A)

> Hypothèse : la branche `minimal-config-improvements` a bien été créée depuis `1.4.0`.

```bash
git checkout minimal-config-improvements
git rebase --onto upstream/main 1.4.0 minimal-config-improvements
```

### En cas de conflits

1. Corriger les fichiers en conflit
2. Marquer les résolutions
3. Continuer le rebase

```bash
git add <fichier_corrige>
git rebase --continue
```

Annuler complètement le rebase si besoin :

```bash
git rebase --abort
```

---

## 6) Vérifier le résultat du rebase

```bash
git status -sb
git branch -vv
git log --oneline --decorate --graph --max-count=30
git diff --stat upstream/main...minimal-config-improvements
```

---

## 7) Pousser la branche réécrite sur `origin`

Le rebase réécrit l'historique : il faut pousser avec `--force-with-lease` (sécurisé).

```bash
git push --force-with-lease origin minimal-config-improvements
```

Version explicite (équivalente) :

```bash
git push --force-with-lease origin minimal-config-improvements:minimal-config-improvements
```

---

## 8) Erreur courante rencontrée et correction

Erreur observée :

```text
erreur : le spécificateur de référence source minimal-config-improvement ne correspond à aucune référence
```

Cause :

- mauvais nom de branche (`minimal-config-improvement` au singulier),
- alors que la branche réelle est `minimal-config-improvements` (avec `s`).

Commande correcte :

```bash
git push --force-with-lease origin minimal-config-improvements
```

---

## 9) Préconisations de nommage (pour éviter les confusions)

Bon schéma recommandé :

- `upstream/main` : référence source (lecture/intégration),
- `main` (fork) : branche stable de ton fork,
- `feature/*` : branches d'évolution (ex: `feature/minimal-config-improvements`),
- `release/*` : branches de préparation de release si nécessaire.

Conseil pratique :

- garder des noms explicites et consistants (singulier/pluriel),
- éviter un nom trop proche entre branches critiques.

