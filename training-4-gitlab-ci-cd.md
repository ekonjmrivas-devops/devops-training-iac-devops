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
- `.gitlab/gitlab-ci-dummy.yml` (plantilla inicial)
- `.gitlab/gitlab-ci-template.yml` (estructura final / guía, sin “código copiable”)

El fichero real que ejecuta GitLab CI/CD es:
- `.gitlab-ci.yml`

Ejercicio 0 (setup):
1) Copia `.gitlab/gitlab-ci-dummy.yml` a `.gitlab-ci.yml`
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

## GitLab Runners (igual que el runner self-hosted de GitHub Actions)
Un **GitLab Runner** es el componente que ejecuta los jobs de CI/CD. Sin runner, GitLab no puede correr tu pipeline.

Tipos comunes:
- **Shared runners** (GitLab.com): disponibles si GitLab los ofrece para tu plan/proyecto.
- **Specific runners** (self-hosted): los instalas tú en tu máquina/VM/red (ideal para conectividad on-prem).

Ventajas de usar runners self-hosted en el curso/empresa
- **Personalización**: instalas herramientas específicas (docker compose, CLIs, SDKs).
- **Conectividad on-prem**: acceso a servicios internos (registry, Artifactory, SonarQube, BBDD, etc.).
- **Seguridad**: control de egress/ingress y posibilidad de aislar runners por proyecto.
- **Rendimiento/caché**: caches persistentes (Maven/pip/Docker layers) para builds más rápidos.
- **Cumplimiento**: integración con redes segregadas, proxies, políticas internas.

Notas de seguridad:
- Si usas Docker executor y montas `/var/run/docker.sock`, el job tiene mucho poder sobre el host Docker.
- En entornos reales: usar máquinas dedicadas, aislar runners, rotar tokens y aplicar hardening.

## Ejercicio final (opcional) - Desplegar un GitLab Runner en local con Docker Compose
Objetivo: ejecutar pipelines desde tu máquina/red para tener conectividad con herramientas on-prem (registry/Artifactory).

### Paso 1 - Crear/obtener el token de registro del runner
En GitLab (UI):
- Settings -> CI/CD -> Runners
- En “Set up a specific Runner manually” copia:
  - URL de GitLab (por ejemplo `https://gitlab.com/` o tu GitLab on-prem)
  - Token de registro (registration token)

### Paso 2 - Registrar el runner (Docker)
Ejecuta una vez (en tu máquina):
```bash
docker run --rm -it \
  -v gitlab-runner-config:/etc/gitlab-runner \
  gitlab/gitlab-runner:alpine \
  register
```

Durante el asistente:
- URL: la de tu GitLab
- Token: el registration token del proyecto/grupo
- Executor: `docker`
- Default image: `alpine:3.19` (o la que uses en CI)

### Paso 3 - Ejecutar el runner como servicio (Docker Compose)
Crea un fichero `docker-compose.gitlab-runner.yml` (por ejemplo en el repo IaC o en tu entorno local):
```yaml
version: "3.8"

services:
  gitlab-runner:
    image: gitlab/gitlab-runner:alpine
    restart: unless-stopped
    volumes:
      - gitlab-runner-config:/etc/gitlab-runner
      # Permite ejecutar docker/docker compose desde los jobs (Docker executor + docker.sock)
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - devops_training_net

volumes:
  gitlab-runner-config:

networks:
  # Reutiliza la red donde viven registry/artifactory en tu stack IaC (ajusta el nombre si cambia)
  devops_training_net:
    external: true
```

Arranque:
```bash
docker compose -f docker-compose.gitlab-runner.yml up -d
docker compose -f docker-compose.gitlab-runner.yml logs -f
```

### Paso 4 - Usar el runner en `.gitlab-ci.yml` (tags)
Cuando registras el runner puedes asignar **tags** (por ejemplo `local`, `docker`, `onprem`).

En tu job:
```yaml
ci:build-image:
  stage: ci
  tags: [local, docker]
  image: docker:27
  services:
    - docker:27-dind
  script:
    - echo "TODO: docker build ..."
```

Así fuerzas a que el job se ejecute en tu runner local (con conectividad a `local-registry` y `artifactory`).

### Paso 5 - Conectividad con registry/Artifactory
Si levantas registry/Artifactory en el stack IaC y el runner está en la misma red Docker (`devops_training_net`), tus jobs podrán acceder a:
- `local-registry:5000`
- `artifactory:8081`

Si usas GitLab.com con runners compartidos, NO tendrás acceso a esos hosts internos.

Referencias:
- GitLab Runner: https://docs.gitlab.com/runner/
- Install/Run with Docker: https://docs.gitlab.com/runner/install/docker.html
- Executors (docker): https://docs.gitlab.com/runner/executors/docker.html

## Entregables (opcional ejecutar)
- `.gitlab-ci.yml` en ambos repos (Python/Java) con:
  - stages CI y CD
  - variables y reglas por rama
  - cleanup con `after_script`
- Alternativamente (si no ejecutas):
  - `.gitlab/gitlab-ci-dummy.yml` y `.gitlab/gitlab-ci-template.yml` + `.gitlab-ci.yml` con la estructura final comentada
