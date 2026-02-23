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

Servicio opcional por perfil (Práctica 3):
- `github-runner` (runner self-hosted de GitHub Actions en Docker)
  - Perfil de compose: `github-runner`
  - Usa variables de entorno:
    - `GITHUB_RUNNER_REPO_URL`
    - `GITHUB_RUNNER_TOKEN`
    - `GITHUB_RUNNER_NAME` (opcional, default `local-runner-01`)
    - `GITHUB_RUNNER_LABELS` (opcional, default `self-hosted,linux,docker`)
  - Plantilla de variables: `.env.github-runner.example`

Arranque del runner de GitHub:
```bash
cp .env.github-runner.example .env.github-runner
# editar .env.github-runner con valores reales
docker compose --env-file .env.github-runner --profile github-runner up -d github-runner
docker compose --env-file .env.github-runner logs -f github-runner
```

Servicio opcional por perfil (Práctica 4):
- `gitlab-runner` (runner local de GitLab CI/CD)
  - Perfil de compose: `gitlab-runner`
  - Registro automático al arrancar (si no existe `config.toml`)
  - Usa variables de entorno:
    - `GITLAB_URL`
    - `GITLAB_REGISTRATION_TOKEN`
    - `GITLAB_RUNNER_NAME` (opcional)
    - `GITLAB_RUNNER_TAGS` (opcional, default `local,docker`)
    - `GITLAB_DOCKER_IMAGE` (opcional, default `docker:27`)
  - Plantilla de variables: `.env.gitlab-runner.example`

Arranque del runner de GitLab:
```bash
cp .env.gitlab-runner.example .env.gitlab-runner
# editar .env.gitlab-runner con valores reales
docker compose --env-file .env.gitlab-runner --profile gitlab-runner up -d gitlab-runner
docker compose --env-file .env.gitlab-runner logs -f gitlab-runner
```

Servicio opcional por perfil (Práctica 5):
- `azure-agent` (agente self-hosted para Azure DevOps)
  - Perfil de compose: `azure-agent`
  - Usa variables de entorno:
    - `AZP_URL`
    - `AZP_TOKEN`
    - `AZP_POOL` (opcional, default `local-docker`)
    - `AZP_AGENT_NAME` (opcional)
  - Plantilla de variables: `.env.azure-agent.example`

Arranque del agente Azure DevOps:
```bash
cp .env.azure-agent.example .env.azure-agent
# editar .env.azure-agent con valores reales
docker compose --env-file .env.azure-agent --profile azure-agent up -d azure-agent
docker compose --env-file .env.azure-agent logs -f azure-agent
```

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
