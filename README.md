# .github — workflows partagés

Ce repo contient les **workflows GitHub Actions réutilisables** de l'organisation `glop2026shoploc`. Les repos des services les appellent au lieu de dupliquer leur CI.

| Fichier | Utilisé par | Ce qu'il fait |
|---|---|---|
| `.github/workflows/ci-back.yml` | users, fidelity, statistiques, commande, catalogue | Java 21 → `mvn package` (compilation + tests + jar) → build de l'image → push sur GHCR |
| `.github/workflows/ci-front.yml` | front | Node 22 → `npm ci` → lint → tests → `npm run build` → build de l'image → push sur GHCR |

---

## Utilisation dans un repo

`.github/workflows/ci.yml` du repo du service :

```yaml
name: CI

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: glop2026shoploc/.github/.github/workflows/ci-back.yml@main   # ou ci-front.yml
    with:
      image-name: users                                       # nom de l'image sur GHCR
    permissions:
      contents: read
      packages: write
```

### Paramètres

| Workflow | Paramètre | Obligatoire | Défaut |
|---|---|---|---|
| `ci-back.yml` | `image-name` | ✅ | — |
| `ci-back.yml` | `java-version` | | `21` |
| `ci-front.yml` | `image-name` | ✅ | — |
| `ci-front.yml` | `node-version` | | `22` |

---

## Comportement

| Événement | Tests / build | Image publiée |
|---|---|---|
| Pull request vers `main` | ✅ | ❌ (construite seulement) |
| Push / merge sur `main` | ✅ | ✅ `sha-<commit>` + `latest` |
| Tag `vX.Y.Z` | ✅ | ✅ `X.Y.Z` |

Les images sont publiées sur `ghcr.io/glop2026shoploc/<image-name>` avec le `GITHUB_TOKEN` (aucun secret à configurer).

---

## Règles importantes

- **Nom du check** : `ci / build-and-push` = `<job du repo appelant> / <job du workflow>`. Il est utilisé par la ruleset qui protège `main` : **ne pas renommer** le job `ci` (repos) ni `build-and-push` (workflows), sinon les merges seront bloqués.
- Toute modification ici s'applique **à tous les repos** qui utilisent `@main` ! 
- Premier push d'une nouvelle image : vérifier dans *Package settings → Manage Actions access* que le repo du service a les droits d'écriture.
