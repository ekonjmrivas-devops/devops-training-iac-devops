# DevOps Training - IaC y estrategia DevOps

Repositorio base de infraestructura como código (IaC) y estrategia DevOps usada en el curso.

## Contexto del curso
- Herramientas: Jenkins (on-prem), GitHub/GitLab (SaaS) y Azure DevOps (plataforma cloud 360).
- La rama `feat/base` contiene lo mínimo para empezar y el enunciado de todas las prácticas.
- Cada práctica tiene su `.md` de enunciado en `feat/base`. La solución vive en la rama `training-x-title` de esa práctica.
- Este README se actualizará de forma incremental durante el curso.

## Repositorios del curso (ramas base)
- App Python: https://github.com/contreras-adr/devops-training-python-app/tree/feat/base
- App Java: https://github.com/contreras-adr/devops-training-java-app/tree/feat/base
- IaC/DevOps: https://github.com/contreras-adr/devops-training-iac-devops/tree/feat/base

## Propósito del repositorio
- Despliegue local de Jenkins y dependencias.
- Material base para credenciales, pipelines y estrategia CI/CD.

## Stack local (Docker Compose)
El stack del laboratorio se levanta con:

```bash
docker-compose up -d
```

Servicios incluidos en `docker-compose.yml`:
- `jenkins` (UI `localhost:8080`)
- `dind` (Docker-in-Docker para ejecución de pipelines)
- `registry` (`local-registry:5000`)
- `artifactory` (`artifactory:8081`)

Servicios opcionales (comentados en `docker-compose.yml`):
- `sonarqube` + `sonardb` para análisis de calidad si se quiere ampliar el stack local.

Nota de prácticas:
- En la **Práctica 1** el foco es la instalación y configuración base de Jenkins.
- `registry` y `artifactory` pasan a ser necesarios desde la **Práctica 2** (publicación de imágenes y artefactos).

## Casos prácticos (5)
Habrá cinco casos prácticos, cada uno con una única rama de solución `training-x-title`.
- training-1-jenkins-config - enunciado: [training-1-jenkins-config.md](training-1-jenkins-config.md)
- training-2-piplines-ci-cd-jenkins  - enunciado: [training-2-piplines-ci-cd-jenkins.md](training-2-piplines-ci-cd-jenkins.md)
- training-3-github-actions  - enunciado: [training-3-github-actions.md](training-3-github-actions.md)
- training-4-gitlab-ci-cd  - enunciado: [training-4-gitlab-ci-cd.md](training-4-gitlab-ci-cd.md)
- training-5-azure-devops-pipelines  - enunciado: [training-5-azure-devops-pipelines.md](training-5-azure-devops-pipelines.md)
