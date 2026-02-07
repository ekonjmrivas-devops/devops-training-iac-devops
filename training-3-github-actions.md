# Práctica 3 - Pipelines CI/CD con GitHub Actions (Guiada)

En esta práctica harás lo mismo que en la Práctica 2, pero usando **GitHub Actions** en lugar de Jenkins. El objetivo es aprender, paso a paso, cómo se define un workflow YAML, cómo se usan parámetros, variables de entorno y condiciones por rama, y cómo usar Docker como “agente” de ejecución (jobs en contenedor).

Además, al final montarás un **runner self-hosted** en local con Docker Compose para poder ejecutar los pipelines con conectividad a herramientas on-prem (por ejemplo un registry y Artifactory levantados en tu red).

## Prerrequisitos
- Tener acceso a los repositorios del curso en GitHub.
- GitHub Actions habilitado en el repositorio.
- Ramas `develop` (DEV) y `master` (PRO) creadas en el repo (si tu repo usa `main`, adapta la práctica a `main`).
- (Recomendado) Conocer la Práctica 2 (stages, CI/CD, cleanup).

## Conceptos clave (mini teoría)
Antes de tocar el YAML, conviene entender estas piezas:
- **Workflow**: fichero YAML en `.github/workflows/*.yml` que define automatizaciones.
- **Event triggers** (`on:`): cuándo se ejecuta (push, PR, manual, schedule, etc.).
- **Jobs**: unidades de ejecución paralelizables (`jobs.<job_id>`).
- **Steps**: comandos o acciones dentro de un job.
- **Runner**: la “máquina” donde se ejecutan los jobs:
  - GitHub-hosted (en la nube de GitHub)
  - Self-hosted (en tu máquina/VM/red)
- **Actions**: piezas reutilizables (por ejemplo `actions/checkout@v4`).
- **Contexts**: variables internas (`github.*`, `env.*`, `secrets.*`, `inputs.*`) que puedes usar en `if:` o en scripts.

## Estrategia Gitflow simulada (DEV y PRO)
Simulamos 2 entornos:
- **DEV**: despliegue desde `develop`
- **PRO**: despliegue desde `master`

Reglas:
- **CI** se ejecuta en cualquier rama.
- **CD** solo se ejecuta si la rama es `develop` o `master` y el parámetro `RUN_CD` está activo.

## Punto de partida: workflow dummy (010)
En cada repo de proyecto (Python/Java) tienes ficheros de referencia:
- `.github/workflows/010-workflow-dummy.yml` (plantilla inicial)
- `.github/workflows/020-workflow-solution.yml` (estructura final / guía, sin “código copiable”)

Tu fichero “real” para que GitHub lo ejecute debe ser:
- `.github/workflows/ci.yml`

Ejercicio 0 (setup):
1) Crea la carpeta `.github/workflows/` (si no existe).
2) Copia `.github/workflows/010-workflow-dummy.yml` a `.github/workflows/ci.yml`.
3) Haz commit y push. Comprueba en GitHub:
   - Actions -> workflow ejecutado

## Ejercicios (montando el puzle)

### Ejercicio 1 - Triggers: `push`, `pull_request` y `workflow_dispatch`
Objetivo: entender cuándo se ejecuta un workflow.

Tarea:
- Configura el workflow para ejecutarse en:
  - `push` (todas las ramas)
  - `pull_request` (hacia `develop` y `master`)
  - `workflow_dispatch` (ejecución manual) con un input booleano `RUN_CD` (default `false`).

Referencia:
- https://docs.github.com/actions/using-workflows/events-that-trigger-workflows
- https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_dispatch

### Ejercicio 2 - Estructura de un workflow: `name`, `on`, `jobs`, `steps`
Objetivo: entender el esqueleto y validar el flujo con logs.

Tarea:
- Añade un job `ci` con steps:
  - `actions/checkout@v4`
  - Un step `Info` que imprima:
    - `github.ref_name`, `github.sha`
    - `runner.os`, `runner.name`

Pista:
- En GitHub Actions, la mayoría de valores se usan con la sintaxis: `${{ ... }}`.
- Para imprimir en logs, puedes usar `run: echo ...` (bash).

Referencia:
- https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions

### Ejercicio 3 - Variables de entorno: `env` (workflow/job/step)
Objetivo: definir variables y reutilizarlas sin duplicar strings.

Tarea:
- Define `env` a nivel de workflow con:
  - `IMAGE_NAME`
  - `APP_URL`
- Sobrescribe una variable a nivel de job (para ver precedencia).
- Usa las variables dentro de `run:` (shell) y dentro de `${{ env.VAR }}`.

Referencia:
- https://docs.github.com/actions/learn-github-actions/variables

### Ejercicio 4 - Parámetros (inputs) con `workflow_dispatch`
Objetivo: aprender a ejecutar manualmente con parámetros, como “Build with Parameters” en Jenkins.

Tarea:
- En `workflow_dispatch`, define inputs:
  - `RUN_CD` (boolean, default `false`)
  - `DEPLOY_ENV` (choice: DEV/PRO, opcional para simular)
- Imprime en un step el valor de `inputs.RUN_CD` y `inputs.DEPLOY_ENV`.

Pista:
- Los inputs se acceden como `${{ inputs.RUN_CD }}` y `${{ inputs.DEPLOY_ENV }}`.

Referencia:
- https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions#onworkflow_dispatchinputs

### Ejercicio 5 - Gestión de variables y secretos en GitHub (documentación + configuración)
Objetivo: aprender dónde se guardan variables y secretos, y cómo aplicarlos a nivel repo/entorno.

Qué documentar en el repo (README o en esta práctica):
- Variables:
  - Settings -> Secrets and variables -> Actions -> **Variables**
- Secrets:
  - Settings -> Secrets and variables -> Actions -> **Secrets**
- Niveles:
  - **Repository**: aplica a todos los workflows del repo.
  - **Environment**: aplica solo cuando el job usa `environment:` (permite protecciones).
  - **Organization** (si aplica): centraliza para muchos repos.

Tarea:
1) Crea variables (Repository variables), por ejemplo:
   - `REGISTRY_HOST` (ej. `local-registry:5000` o `ghcr.io`)
   - `ARTIFACTORY_URL` (ej. `http://artifactory:8081/artifactory`)
2) Crea secrets (Repository secrets), por ejemplo:
   - `REGISTRY_USER`, `REGISTRY_PASSWORD`
   - `ARTIFACTORY_USER`, `ARTIFACTORY_PASSWORD` (o token)
3) En el workflow, úsalo así:
   - Variables: `${{ vars.REGISTRY_HOST }}` / `${{ vars.ARTIFACTORY_URL }}`
   - Secrets: `${{ secrets.REGISTRY_PASSWORD }}`

Notas importantes (seguridad):
- Los secrets se enmascaran en logs, pero no imprimas secretos deliberadamente.
- Usa **Environments** (DEV/PRO) si quieres separar credenciales y proteger PRO con aprobaciones.

Referencia:
- https://docs.github.com/actions/security-guides/encrypted-secrets
- https://docs.github.com/actions/deployment/targeting-different-environments/using-environments-for-deployment

### Ejercicio 6 - Docker como agente: job en contenedor
Objetivo: ejecutar CI dentro de un contenedor, igual que hacíamos con `agent docker` en Jenkins.

Tarea:
- Python: ejecuta tests dentro de `python:3.6-slim`.
- Java: ejecuta `make lint` y `make test` dentro de `maven:3.8.6-openjdk-11-slim`.

Pista:
- En GitHub Actions puedes usar `container:` a nivel de job.

Referencia:
- https://docs.github.com/actions/using-jobs/running-jobs-in-a-container

### Ejercicio 7 - Build de imagen Docker en CI
Objetivo: construir la imagen Docker del repo (sin despliegue aún).

Tarea:
- Construye la imagen con el mismo tag que usa tu `docker-compose.yml`, para que luego CD pueda arrancar por compose.

### Ejercicio 8 - Condiciones por rama: CD solo en `develop/master`
Objetivo: simular despliegue por entornos.

Tarea:
- Crea un job `cd` (o stage equivalente) que solo se ejecute si:
  - `github.ref_name` es `develop` o `master`, y
  - `inputs.RUN_CD == true` (workflow dispatch)

Pista:
- Usa `if:` a nivel de job.

Referencia:
- https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idif

### Ejercicio 9 - Simulación de despliegue con Docker Compose + cleanup
Objetivo: simular CD en un runner efímero:
1) `docker compose up -d`
2) test e2e con `curl`
3) cleanup siempre, aunque falle

Tarea:
- Implementa el cleanup con un step `if: always()`.

Referencia:
- https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsif

## Opcionales

### Opcional A - Push a un registry
Objetivo: publicar la imagen construida.

Notas:
- En GitHub Actions, el caso más sencillo es **GHCR** (`ghcr.io`) usando `GITHUB_TOKEN`.
- Un registry “local” solo tiene sentido si usas un self-hosted runner en tu red.

Referencia:
- https://docs.github.com/packages/working-with-a-github-packages-registry/working-with-the-container-registry

### Opcional B (Java) - Subir el `.jar` a Artifactory con `curl` (sin plugin)
Igual que en Práctica 2, pero ejecutándolo desde un step de GitHub Actions usando secretos.

## Ejercicio final - Runner self-hosted en local con Docker Compose (on-prem)
Objetivo: ejecutar workflows de GitHub Actions **desde tu máquina/red** para tener conectividad con:
- Registry privado local
- Artifactory local
- Cualquier herramienta on-prem (Jenkins, SonarQube, bases de datos, etc.)

Por qué hace falta:
- Los runners GitHub-hosted NO pueden acceder a servicios de tu red local (registry/artifactory “local-registry”, IPs privadas, etc.).
- Un runner self-hosted en tu red sí puede, porque comparte conectividad (LAN/VPN) y puedes unirlo a tus redes Docker.

### Paso 1 - Crear un runner self-hosted en el repo
En GitHub:
- Settings -> Actions -> Runners -> New self-hosted runner
- Elige Linux y sigue las instrucciones.

Nota:
- Para este curso lo vamos a ejecutar dentro de un contenedor con Docker Compose.

### Paso 2 - Docker Compose del runner (plantilla)
Crea un fichero `docker-compose.runner.yml` (por ejemplo en el repo IaC o en tu entorno local):
```yaml
version: "3.8"

services:
  gha-runner:
    image: myoung34/github-runner:latest
    restart: unless-stopped
    environment:
      # TODO: configura estas variables según el asistente de GitHub
      REPO_URL: "https://github.com/<org>/<repo>"
      RUNNER_NAME: "local-runner-01"
      RUNNER_WORKDIR: "/tmp/runner"
      RUNNER_LABELS: "self-hosted,linux,docker"
      # Requiere token de registro (de GitHub) o PAT según el modo
      ACCESS_TOKEN: "${GITHUB_RUNNER_TOKEN}"
    volumes:
      # Permite ejecutar docker/docker compose desde el runner usando el Docker del host
      - /var/run/docker.sock:/var/run/docker.sock
      - gha-runner-work:/tmp/runner
    networks:
      - devops_training_net

volumes:
  gha-runner-work:

networks:
  # Reutiliza la red donde viven registry/artifactory en tu stack IaC (ajusta el nombre si cambia)
  devops_training_net:
    external: true
```

### Paso 3 - Ejecutar el runner
```bash
export GITHUB_RUNNER_TOKEN="<token>"
docker compose -f docker-compose.runner.yml up -d
docker compose -f docker-compose.runner.yml logs -f
```

### Paso 4 - Usar el runner en el workflow
En tu `.github/workflows/ci.yml`, cambia:
- `runs-on: ubuntu-latest`
por:
- `runs-on: [self-hosted, linux, docker]`

Así, los jobs correrán en tu runner local y tendrán conectividad con:
- `local-registry:5000` (si está levantado en la red Docker)
- `artifactory:8081` (si está levantado en la red Docker)

### Ventajas del runner self-hosted (para el curso y para empresa)
- **Personalización**: instalas herramientas específicas (docker compose, CLIs, SDKs, etc.).
- **Conectividad on-prem**: acceso a servicios internos (registry, Artifactory, SonarQube, DBs, etc.).
- **Seguridad**: secretos y accesos permanecen en tu red (mejor control de egress/ingress).
- **Rendimiento/caché**: puedes persistir caches (Maven, pip, Docker layers) para builds más rápidos.
- **Cumplimiento**: más fácil integrar con redes segregadas, proxies, políticas internas.

Notas de seguridad:
- Montar `/var/run/docker.sock` da mucho poder al runner (equivalente a root sobre Docker). En entornos reales:
  - aislar runners por proyecto
  - usar hosts dedicados
  - rotar tokens y aplicar hardening

## Entregables
- `.github/workflows/ci.yml` creado y evolucionado en ambos repos (Python y Java).
- CI funcionando en cualquier rama.
- CD condicionado por rama (`develop/master`) y por parámetro (`RUN_CD`).
- Cleanup garantizado (aunque falle el e2e).
