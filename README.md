# IngsisPrintScript: Workflows & Actions

Repositorio centralizado para los **workflows**, **formatters**, **linters**, **checkers**, y **coverage** compartidas entre todos los proyectos de la organización **IngsisPrintScript**.

---

## Estructura del repositorio

```
.github/
 └── workflows/
     └── ci.yml                # Workflow reutilizable con build, format, test y coverage
.githooks/
 ├── pre-commit                # Ejecuta spotlessApply y agrega cambios formateados
 └── pre-push                  # Ejecuta spotlessCheck + tests
config/
 ├── checkstyle/
 │   └── checkstyle.xml        # Reglas de estilo Java
 └── jacoco.gradle             # Configuración de cobertura mínima (80%)
scripts/
 └── install-hooks.sh          # Script para instalar hooks de Git
README.md
```

---



### Workflow principal: `.github/workflows/ci.yml`

Incluye los siguientes pasos automáticos:

1. **Spotless Apply:** aplica formato a todo el código (`spotlessApply`).
2. **Build:** compila el proyecto con Gradle (`clean build`).
3. **Tests:** ejecuta todos los tests de JUnit.
4. **Coverage Check:** falla el build si la cobertura total es menor al 80 %.
5. **Reportes:** genera reportes HTML y XML con JaCoCo.

---

## Cómo integrarlo en otros repositorios

Cada microservicio de la organización debe tener un archivo `.github/workflows/ci.yml` con el siguiente contenido:

```yaml
name: CI

on:
  push:
    branches: [ "main", "dev" ]
  pull_request:
    branches: [ "main", "dev" ]

jobs:
  call-central-workflow:
    uses: IngsisPrintScript/workflows-and-actions/.github/workflows/ci.yml@main
    secrets: inherit
```

---

## Integrar Checkstyle y JaCoCo locales

Si querés que tu proyecto use los mismos estándares de Checkstyle y cobertura definidos en este repo, agregá en tu `build.gradle`:

```groovy
checkstyle {
    configFile = file("${rootProject.projectDir}/../workflows-and-actions/config/checkstyle/checkstyle.xml")
}

apply from: "${rootProject.projectDir}/../workflows-and-actions/config/jacoco.gradle"
```

*(ajustá la ruta si tu repo no está en el mismo nivel que `workflows-and-actions`)*

---

## Instalación de Git Hooks

Para usar los hooks locales de formateo y verificación antes de commit/push:

```bash
chmod +x scripts/install-hooks.sh
./scripts/install-hooks.sh
```

Esto configura:

- `pre-commit`: aplica `spotlessApply` automáticamente y agrega los cambios formateados.
- `pre-push`: ejecuta `spotlessCheck` y `test` antes de permitir el push.

