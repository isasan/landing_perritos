# README - Flujo de trabajo de GitHub Actions

Este repositorio contiene los siguientes GitHub Actions activos para facilitar la validación de ramas y la gestión de versiones:

## 🚦 Workflows Activos

### 1. **Validación de Nombres de Ramas**

Este workflow asegura que las ramas sigan una convención de nombres antes de ser mergeadas. Las reglas son:

- Las ramas `feature` deben seguir el formato `feature/nombre_descriptivo`.
- Las ramas `release` deben seguir el formato `release/vX.Y.Z`.

Si el nombre de la rama no sigue estas convenciones, el pull request será bloqueado.

**Archivo: `.github/workflows/validate-branches.yml`**

```yaml
name: Validar nombres de ramas

on:
  pull_request:
    branches:
      - main
      - develop

jobs:
  check-branch-name:
    runs-on: ubuntu-latest
    steps:
      - name: Validar nombre de rama
        run: |
          BRANCH_NAME="${GITHUB_HEAD_REF}"
          echo "Rama: $BRANCH_NAME"

          if [[ "$BRANCH_NAME" =~ ^feature\/[a-z0-9_]+$ ]]; then
            echo "Rama de feature válida."
          elif [[ "$BRANCH_NAME" =~ ^release\/v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
            echo "Rama de release válida."
          else
            echo "❌ Nombre de rama no válido. Debe ser 'feature/nombre' o 'release/vX.Y.Z'"
            exit 1
          fi
```

### 2. **Validación de Tag en Releases**

Antes de mergear una rama `release/*` a `main`, este workflow comprueba que exista un tag con la versión correspondiente de la release (`vX.Y.Z`). Si no existe el tag, el pull request será bloqueado.

**Archivo: `.github/workflows/validate-release-tag.yml`**

```yaml
name: Validar existencia de tag en releases

on:
  pull_request:
    branches:
      - main

jobs:
  check-release-tag:
    if: startsWith(github.head_ref, 'release/')
    runs-on: ubuntu-latest
    steps:
      - name: Comprobar que existe un tag para la release
        run: |
          RELEASE_NAME="${GITHUB_HEAD_REF}"
          VERSION_TAG="${RELEASE_NAME##release/}"

          echo "Buscando tag: $VERSION_TAG"

          TAG_EXIST=$(git ls-remote --tags origin "refs/tags/$VERSION_TAG")

          if [ -z "$TAG_EXIST" ]; then
            echo "❌ No release tag matching '$VERSION_TAG' found. Create it before merging to main."
            exit 1
          else
            echo "✅ Tag $VERSION_TAG encontrado."
          fi
```

---

## 📑 Convenciones de Ramas

- **Feature**: Las ramas de características deben tener el formato `feature/nombre_descriptivo`.
  
  Ejemplo:
  ```
  feature/cambiar-colores
  ```

- **Release**: Las ramas de release deben tener el formato `release/vX.Y.Z`, donde `X`, `Y` y `Z` son números.

  Ejemplo:
  ```
  release/v1.0.0
  ```

---

## 📌 Pasos para Crear una Release

1. Crea la rama de release:
   ```bash
   git checkout -b release/v1.0.0
   git push origin release/v1.0.0
   ```

2. Crea el tag correspondiente:
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

3. Abre un pull request de la rama `release` hacia `main`.

---

## ⚠️ Consejos

- Asegúrate de crear el **tag** antes de abrir el PR de una rama `release`.
- Si un PR es bloqueado por la validación de la rama o el tag, revisa que ambos sigan las convenciones descritas anteriormente.

---

¡Así de sencillo! 😎
