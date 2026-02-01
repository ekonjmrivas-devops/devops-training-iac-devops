# Practica 1 - Instalacion de Jenkins en local y configuracion base

En esta practica instalaremos y configuraremos Jenkins en local para poder ejecutar el resto del curso.

## Objetivos
- Instalar y arrancar Jenkins en local con los plugins minimos.
- Configurar acceso a Git (SSH) para que Jenkins pueda leer repositorios.
- Preparar Multi-branch Pipelines para los repos de Python y Java.
- Dejar Jenkins listo para ejecutar steps en contenedores (Docker) de forma aislada.

## Repos relacionados y ramas
- Repo Python (base `feat/base`): https://github.com/contreras-adr/devops-training-2025-python-app/tree/feat/base
- Repo Python (solucion `training-1-jenkins-config`): https://github.com/contreras-adr/devops-training-2025-python-app/tree/training-1-jenkins-config
- Repo Java (base `feat/base`): https://github.com/contreras-adr/devops-training-2025-java-app/tree/feat/base
- Repo Java (solucion `training-1-jenkins-config`): https://github.com/contreras-adr/devops-training-2025-java-app/tree/training-1-jenkins-config
- Repo IaC/DevOps (base `feat/base`): https://github.com/contreras-adr/devops-training-2025-iac-devops/tree/feat/base
- Repo IaC/DevOps (solucion `training-1-jenkins-config`): https://github.com/contreras-adr/devops-training-2025-iac-devops/tree/training-1-jenkins-config

## Prerequisitos
- Docker y Docker Compose instalados.
- Opcional (si estas en Windows): WSL2 Ubuntu 22.04.
  - https://gist.github.com/Adhjie/8dcab8ef69a82e0b35d017725f20de19
  - https://documentation.ubuntu.com/wsl/en/latest/howto/install-ubuntu-wsl2/

## Docker Compose (stack de Jenkins) - fichero existente
En este curso Jenkins se despliega con Docker Compose junto a un daemon Docker-in-Docker (DinD). Esto permite crear “entornos virtualizados” (contenedores) para ejecutar steps de los pipelines sin depender del Docker del host.

Consulta el fichero:
- `devops-training-2025-iac-devops/docker-compose.yml`

Resumen de lo que despliega:
- Red `devops_training_net` (bridge) con subnet `172.16.236.0/24`
- Volumen `jenkins-data` para persistir `/var/jenkins_home`
- Volumen `jenkins-docker-certs` para certificados TLS entre Jenkins y DinD
- Servicio `dind` (Docker API por TLS en 2376)
- Servicio `jenkins` (UI en 8080 y agentes inbound en 50000)

## Configurar plugins de Jenkins
Antes de levantar Jenkins, define plugins en:
- `devops-training-2025-iac-devops/jenkins/plugins.txt`

Nota:
- Si cambias esta lista y ya tienes Jenkins con volumen persistente, deberas borrar el volumen para aplicar bien el cambio.

## Arrancar Jenkins
```bash
docker-compose up -d
docker-compose logs jenkins
```

## Desbloquear Jenkins (primer arranque)
Cuando Jenkins arranca por primera vez, muestra la contrasena inicial en los logs. La ruta dentro del contenedor suele ser:
```
/var/jenkins_home/secrets/initialAdminPassword
```

En la UI de Jenkins, copia y pega esa contrasena en la pantalla de “Unlock Jenkins” y continua el asistente.

## Crear clave SSH de GitHub para Jenkins
```bash
ssh-keygen -C "contreras.adr@outlook.com" -f ~/.ssh/jenkins-github
cat ~/.ssh/jenkins-github.pub
```

## Multi-branch pipelines + Jenkinsfile dummy
En Jenkins, crea un Multi-branch Pipeline para cada repo (Python y Java) y apunta al fichero:
- `devops/jenkinsfile`

En Practica 2, ese Jenkinsfile pasara de “dummy” a un pipeline real.

