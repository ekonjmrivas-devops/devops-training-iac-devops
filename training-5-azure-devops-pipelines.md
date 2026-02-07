# Práctica 5 - Azure DevOps Pipelines (Guiada)

En esta práctica aprenderás a montar pipelines CI/CD en **Azure DevOps** integrándolo directamente con **GitHub**. Repetiremos los conceptos de las prácticas anteriores (pipeline por etapas, variables, parámetros, condiciones por rama, templates reutilizables y agentes self-hosted), pero en el ecosistema de Azure DevOps.

IMPORTANTE:
- La ejecución de esta práctica es **opcional**: no es obligatorio crear una cuenta de Azure DevOps ni ejecutar los pipelines.
- Para entregar la práctica, es suficiente con presentar la **solución teórica** (los ficheros YAML con la estructura final y comentarios).

## Azure DevOps como plataforma SDLC 360 (visión general)
Azure DevOps es una plataforma “360” para el ciclo de vida de desarrollo (SDLC), porque integra en un mismo sitio:
- **Azure Boards**: planificación (backlog, issues, sprints, kanban).
- **Azure Repos**: repos Git (opcional si integras con GitHub).
- **Azure Pipelines**: CI/CD (YAML y pipelines clásicos).
- **Azure Test Plans**: gestión de pruebas (manuales/exploratorias).
- **Azure Artifacts**: repositorios de paquetes (similar a “artifact repository”).
- **Wiki**: documentación del proyecto.

## Capa gratuita (free tier)
Azure DevOps ofrece un uso gratuito para equipos pequeños y también permite ejecutar CI/CD en **agentes Microsoft-hosted** o **agentes self-hosted**.

Nota:
- Los límites (usuarios/minutos) cambian con el tiempo. Para la cifra exacta y condiciones actuales, revisa la página oficial de precios de Azure DevOps.
  - https://azure.microsoft.com/pricing/details/devops/azure-devops-services/

## Diferencias frente a Jenkins, GitHub Actions y GitLab CI/CD
Comparativa conceptual (lo importante para el curso):

- **Jenkins**:
  - Motor de automatización “plugin-based”.
  - Tú gestionas la infraestructura y la operación (actualizaciones, plugins, seguridad).
  - Muy flexible, pero más coste de mantenimiento.

- **GitHub Actions**:
  - CI/CD integrado en GitHub.
  - Excelente para automatizar repos en GitHub (workflows, marketplace de actions).
  - Los runners cloud no tienen acceso a tu red local (salvo runner self-hosted).

- **GitLab CI/CD**:
  - Plataforma integrada (repo + CI/CD + más).
  - Runner self-hosted muy común para on-prem.
  - Muy potente en reglas, pipelines y seguridad; depende del stack GitLab que tengas.

- **Azure DevOps**:
  - Suite SDLC completa (Boards, Pipelines, Artifacts, Test Plans, Wiki).
  - Puede trabajar con **repos en GitHub** sin mover el código (Azure Pipelines se integra).
  - La ejecución se controla por **agent pools** (Microsoft-hosted vs self-hosted).
  - Templates YAML muy usados para estandarizar pipelines entre repos.

## Importante: no es necesario clonar ni migrar repos
En esta práctica NO hace falta clonar ni mover repositorios:
- Vamos a crear pipelines en Azure DevOps apuntando directamente a los repos de GitHub.

Ejemplo (ajusta a tu organización):
- Python: `contreras-adr/devops-training-python-app`
- Java: `contreras-adr/devops-training-java-app`
- IaC: `contreras-adr/devops-training-iac-devops`

## Prerrequisitos
- Tener una organización y un proyecto en Azure DevOps.
- Repos en GitHub accesibles (Python y Java).
- Ramas `develop` (DEV) y `master` (PRO) en los repos (si usas `main`, adapta la práctica).

Opcional (para ejercicios finales):
- Tener un registry y un Artifactory levantados en local/on-prem (por ejemplo con el stack IaC del curso).
- Tener Docker y Docker Compose en la máquina donde correrá el agente self-hosted.

## Estrategia Gitflow simulada (DEV y PRO)
Simulamos 2 entornos:
- **DEV**: despliegue desde `develop`
- **PRO**: despliegue desde `master`

Reglas:
- **CI** se ejecuta en cualquier rama.
- **CD** solo se ejecuta si la rama es `develop` o `master` y el parámetro `RUN_CD` está activado.

## Punto de partida: pipeline dummy (010)
En cada repo de proyecto (Python/Java) crea dos ficheros de referencia:
- `azure-pipelines/010-pipeline-dummy.yml` (plantilla inicial)
- `azure-pipelines/020-pipeline-solution.yml` (estructura final / guía, sin “código copiable”)

Tu fichero “real” para que Azure DevOps ejecute el pipeline puede ser cualquiera, pero para el curso recomendamos:
- `azure-pipelines.yml` (en la raíz del repo)

Ejercicio 0 (setup):
1) Copia `azure-pipelines/010-pipeline-dummy.yml` a `azure-pipelines.yml`
2) Haz commit y push.

## Integración Azure DevOps -> GitHub (cómo conectarlo)
Objetivo: que Azure DevOps pueda leer tu repo de GitHub y ejecutar el YAML.

Pasos (alto nivel):
1) En Azure DevOps: Pipelines -> New pipeline
2) Selecciona **GitHub** como origen
3) Autoriza la conexión (GitHub App / OAuth)
4) Selecciona el repo (Python o Java)
5) Elige “Existing Azure Pipelines YAML file” y apunta a:
   - `azure-pipelines.yml`
6) Guarda y lanza el primer run.

Notas:
- Azure DevOps suele crear una “service connection” a GitHub para acceder al repo.
- Si cambias el nombre del fichero, actualiza la ruta en la configuración del pipeline.

## Ejercicios (montando el puzle)

### Ejercicio 1 - Estructura mínima: `trigger`, `pool` y `steps`
Objetivo: entender el YAML mínimo y ver un run exitoso.

Tarea:
- Usa `pool` con un agente Microsoft-hosted (por ejemplo `ubuntu-latest`).
- Crea steps que impriman variables del build (rama, commit, nombre del repo).

### Ejercicio 2 - Triggers: `trigger` y `pr`
Objetivo: controlar cuándo se dispara el pipeline.

Tarea:
- Configura:
  - `trigger` para `develop` y `master`
  - `pr` para PRs hacia `develop` y `master`

### Ejercicio 3 - Variables y secretos en Azure DevOps
Objetivo: aprender dónde se configuran y cómo se consumen.

Dónde se gestionan:
- Pipeline variables (por pipeline).
- Library -> **Variable groups** (reutilización entre pipelines).

Recomendaciones:
- Marca como “secret” cualquier credencial (no imprimirla en logs).
- Usa variable groups para compartir configuración entre repos.

Tarea:
1) Crea variables (no secret) como:
   - `IMAGE_NAME`
   - `APP_URL`
2) Crea secretos como:
   - `REGISTRY_USER`, `REGISTRY_PASSWORD`
   - `ARTIFACTORY_USER`, `ARTIFACTORY_PASSWORD` (o token)
3) Úsalos en scripts (sin imprimir secretos).

### Ejercicio 4 - Parámetros (`parameters`) y ejecución manual
Objetivo: ejecutar con inputs, similar a “Build with Parameters”.

Tarea:
- Añade `parameters`:
  - `RUN_CD` (boolean, default false)
  - `DEPLOY_ENV` (string/choice simulado: DEV/PRO)
- Usa el parámetro en condiciones (ejecutar CD solo si `RUN_CD` es true).

### Ejercicio 5 - CI por proyecto (tests en contenedor)
Objetivo: ejecutar CI en contenedores (equivalente a “Docker agent”).

Tarea:
- Python: ejecutar tests en `python:3.6-slim`.
- Java: ejecutar `make lint` y `make test` en `maven:3.8.6-openjdk-11-slim`.

Pista:
- Azure Pipelines permite `container:` a nivel de job.

### Ejercicio 6 - Build de imagen Docker en CI
Objetivo: construir la imagen Docker del repo.

Tarea:
- Añade un step `docker build` usando el Dockerfile de cada repo.

Opcional:
- Push a registry (si tienes credenciales y conectividad).

### Ejercicio 7 - CD condicionado por rama (`develop/master`) + parámetro `RUN_CD`
Objetivo: simular despliegue DEV/PRO.

Tarea:
- Crea una etapa/stage `CD` que solo se ejecute si:
  - rama `develop` o `master`, y
  - `RUN_CD == true`

### Ejercicio 8 - Templates reutilizables (final 1)
Objetivo: factorizar lógica repetida y reutilizarla en Python y Java.

Estrategia del curso:
- Los templates se guardan en el repo IaC:
  - `devops-training-iac-devops/azure-pipelines/templates/`

Tarea:
1) Crea 1 o más templates en el repo IaC (por ejemplo CI y/o CD).
2) Configura los pipelines de Python y Java para **importar** esos templates desde el repo IaC.

Pistas (concepto):
- En Azure Pipelines puedes usar:
  - `resources.repositories` para traer otro repo como recurso
  - `template:` para incluir plantillas de jobs/stages/steps

Ejemplo (estructura, no copiable tal cual):
```yaml
resources:
  repositories:
    - repository: iac
      type: github
      name: contreras-adr/devops-training-iac-devops
      ref: refs/heads/feat/base
      endpoint: <service-connection-github>

stages:
  - stage: CI
    jobs:
      - job: tests
        steps:
          - template: azure-pipelines/templates/ci-python-steps.yml@iac
            parameters:
              pythonImage: "python:3.6-slim"
```

Resultado:
- Los dos proyectos usan el mismo template sin duplicar YAML.

### Ejercicio 9 - Agente Docker en local (self-hosted) con conectividad on-prem (final 2)
Objetivo: ejecutar pipelines desde tu red para tener conectividad con:
- **Artifactory** on-prem (y/o **Azure Artifacts** como alternativa cloud).
- **Docker registry** on-prem (y/o **Azure Container Registry - ACR** como alternativa en Azure).

Por qué:
- Los agentes Microsoft-hosted no pueden acceder a `local-registry`, `artifactory` y redes privadas de tu laboratorio.
- Un agente self-hosted en tu red sí puede (LAN/VPN y redes Docker).

Qué documentar / hacer:
1) En Azure DevOps, crea un **Agent pool** (por ejemplo `local-docker`).
2) Descarga e instala el agente (Azure DevOps te da comandos/scripts).
3) Opción “Docker”: ejecuta el agente en un contenedor y monta:
   - `/var/run/docker.sock` para poder usar Docker/Docker Compose
   - una red Docker donde exista conectividad con `local-registry` y `artifactory`
4) En el YAML, apunta el pipeline al pool:
   - `pool: name: local-docker`

Ejemplo (docker-compose del agente, plantilla):
```yaml
version: "3.8"

services:
  azp-agent:
    # TODO: usa la imagen del agente de Azure Pipelines recomendada por Microsoft (puede variar).
    image: <azure-pipelines-agent-image>
    restart: unless-stopped
    environment:
      # URL de tu organización/proyecto
      AZP_URL: "https://dev.azure.com/<org>"
      # Token/PAT con permisos para registrar el agente
      AZP_TOKEN: "${AZP_TOKEN}"
      AZP_POOL: "local-docker"
      AZP_AGENT_NAME: "local-agent-01"
    volumes:
      # Permite ejecutar docker/docker compose desde el agente
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      # Red donde viven registry/artifactory del stack IaC (ajusta el nombre si cambia)
      - devops_training_net

networks:
  devops_training_net:
    external: true
```

Ventajas del agente local:
- **Conectividad on-prem**: registry/Artifactory/servicios internos.
- **Personalización**: herramientas instaladas a medida (docker compose, CLIs).
- **Seguridad**: control de red y de secretos dentro de tu entorno.
- **Rendimiento**: caches persistentes (Maven/pip/Docker layers).

Nota de seguridad:
- Montar `/var/run/docker.sock` da permisos elevados al job. En entornos reales, aísla runners/hosts por proyecto y aplica hardening.

## Entregables
- `azure-pipelines.yml` en ambos repos (Python/Java) evolucionado por ejercicios.
- Templates en `devops-training-iac-devops/azure-pipelines/templates/` y consumo desde ambos proyectos.
- Documentación breve de variables/secretos y de agente self-hosted (opcional ejecutar).
