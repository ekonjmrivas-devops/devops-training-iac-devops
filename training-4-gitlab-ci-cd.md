# Práctica 4 - Pipelines CI/CD con GitLab CI/CD (Guiada)

En esta práctica harás lo mismo que en la Práctica 2 (Jenkins) y Práctica 3 (GitHub Actions), pero usando **GitLab CI/CD**.

IMPORTANTE:
- Esta práctica es **opcional**: no es obligatorio crear una cuenta de GitLab ni ejecutar los pipelines.
- Para entregar la práctica, es suficiente con presentar la **solución teórica** (los ficheros `.gitlab-ci.yml` con comentarios/estructura final).

## Prerrequisitos
- Conocer la Práctica 2 (CI/CD por stages, Docker, cleanup, ramas).
- (Opcional) Tener una cuenta de GitLab.com o acceso a un GitLab on-prem.

## Por qué hay que crear repositorios desde cero en GitLab
En el curso partimos de repositorios en GitHub. Para usar GitLab CI/CD, asumimos que:
- Vas a crear **nuevos repositorios Git en GitLab** (vacíos).
- Vas a copiar el código fuente de los repos actuales (Python/Java) y hacer un primer push.

Esto te obliga a practicar una situación real:
- migración/copia de un repo a otra plataforma
- primera configuración de CI/CD en un repo nuevo

## Cómo crear los repositorios Git en GitLab (paso a paso)
Repite estos pasos para **Python** y **Java**:

1) Crear un proyecto vacío en GitLab
- New project -> Create blank project
- Nombre sugerido:
  - `devops-training-python-app` (o añade sufijo `-gitlab`)
  - `devops-training-java-app` (o añade sufijo `-gitlab`)

2) Inicializar el repo local (si no tienes `.git`)
En el directorio del proyecto:
```bash
git init
git add .
git commit -m "Initial import for GitLab CI"
```

3) Añadir remote de GitLab y hacer push
```bash
git remote add origin <URL_DEL_REPO_GITLAB>
git branch -M master
git push -u origin master
```

Notas:
- Si tu repo usa `main`, adapta el nombre de rama.
- Si quieres simular Gitflow como en Práctica 2/3, crea también `develop`:
  - `git checkout -b develop`
  - `git push -u origin develop`

## Estrategia Gitflow simulada (DEV y PRO)
Simulamos 2 entornos:
- **DEV**: despliegue desde la rama `develop`
- **PRO**: despliegue desde la rama `master`

Reglas:
- **CI** se ejecuta en cualquier rama.
- **CD** solo se ejecuta si la rama es `develop` o `master`.

## Punto de partida: pipeline dummy (010)
En cada repo (Python/Java) crea dos ficheros de referencia:
- `.gitlab/010-gitlab-ci-dummy.yml` (plantilla inicial)
- `.gitlab/020-gitlab-ci-solution.yml` (estructura final / guía, sin “código copiable”)

El fichero real que ejecuta GitLab CI/CD es:
- `.gitlab-ci.yml`

Ejercicio 0 (setup):
1) Copia `.gitlab/010-gitlab-ci-dummy.yml` a `.gitlab-ci.yml`
2) Haz commit y push
3) (Si ejecutas la práctica) revisa en GitLab:
   - CI/CD -> Pipelines

## Ejercicios (montando el puzle)

### Ejercicio 1 - Estructura mínima: `stages` y un job
Objetivo: entender el YAML mínimo de GitLab CI/CD.

Tarea:
- Define `stages: [ci]`
- Crea un job `ci:info` que imprima variables (rama/commit).

Referencia:
- https://docs.gitlab.com/ee/ci/yaml/

### Ejercicio 2 - Variables: `variables:` y variables predefinidas
Objetivo: aprender a definir variables y usar las predefinidas de GitLab.

Tarea:
- Define variables globales:
  - `IMAGE_NAME`
  - `APP_URL`
- Imprime en logs:
  - `$CI_COMMIT_BRANCH`
  - `$CI_COMMIT_SHA`

Referencia:
- Variables predefinidas: https://docs.gitlab.com/ee/ci/variables/predefined_variables.html

### Ejercicio 3 - CI con contenedores (Docker como agente)
Objetivo: ejecutar jobs en contenedores (similar a Jenkins agent docker / GHA container jobs).

Tarea:
- Python: job que use `image: python:3.6-slim` y ejecute tests.
- Java: jobs que usen `image: maven:3.8.6-openjdk-11-slim` y ejecuten `make lint` y `make test`.

### Ejercicio 4 - Build de imagen Docker en CI
Objetivo: construir la imagen Docker del repo.

Tarea:
- Añade un job `ci:build-image` que ejecute `docker build`.

Nota:
- Para construir imágenes con Docker dentro de GitLab CI normalmente necesitas:
  - GitLab Runner con Docker executor, y/o
  - Docker-in-Docker (`services: [docker:dind]`)

Referencia:
- https://docs.gitlab.com/ee/ci/docker/using_docker_build.html

### Ejercicio 5 - Condiciones por rama: ejecutar CD solo en `develop/master`
Objetivo: simular despliegue por entornos con reglas.

Tarea:
- Crea un stage `cd` y un job `cd:deploy` que solo ejecute cuando la rama sea:
  - `develop` o `master`

Pista:
- Usa `rules:` con `if:`.

Referencia:
- https://docs.gitlab.com/ee/ci/yaml/#rules

### Ejercicio 6 - Simulación de despliegue con Docker Compose + cleanup
Objetivo: simular CD (up -> e2e -> down).

Tarea:
- En `cd:deploy`, ejecuta:
  - `docker compose up -d`
  - `curl` a `APP_URL`
  - cleanup (siempre) con `after_script:`

Referencia:
- `after_script`: https://docs.gitlab.com/ee/ci/yaml/#after_script

## Gestión de variables y secretos en GitLab
Objetivo: entender dónde se guardan y cómo se protegen.

Dónde:
- Settings -> CI/CD -> Variables

Recomendación:
- Usa variables “Masked” para secretos.
- Usa variables “Protected” para que solo se expongan en ramas protegidas (por ejemplo `master`).

Referencia:
- https://docs.gitlab.com/ee/ci/variables/

## Entregables (opcional ejecutar)
- `.gitlab-ci.yml` en ambos repos (Python/Java) con:
  - stages CI y CD
  - variables y reglas por rama
  - cleanup con `after_script`
- Alternativamente (si no ejecutas):
  - `.gitlab/010-gitlab-ci-dummy.yml` y `.gitlab/020-gitlab-ci-solution.yml` + `.gitlab-ci.yml` con la estructura final comentada

