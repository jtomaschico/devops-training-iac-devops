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
- **CD** se ejecuta si:
  - `RUN_CD=true` (en cualquier rama), **o**
  - la rama es `develop` o `master` (aunque `RUN_CD=false`).

## Qué vas a construir (resultado final)

### Pipeline Python
- CI
  - Ejecutar `make lint` en un contenedor `python:3.6-slim` (imagen de CI desde `devops/ci.Dockerfile`).
  - Ejecutar tests unitarios en el mismo contenedor.
  - Construir la imagen Docker de la app (sin push).
- CD
  - Levantar BBDD + App con Docker Compose en el runner.
  - Mostrar logs del contenedor principal.
  - Limpiar recursos con `docker compose down --volumes`.

### Pipeline Java
- CI
  - Ejecutar `make lint` y `make test` en un contenedor `maven:3.8.6-openjdk-11-slim` (imagen de CI desde `devops/ci.Dockerfile`).
  - Construir la imagen Docker de la app (sin push).
- CD
  - Levantar App con Docker Compose en el runner.
  - Mostrar logs del contenedor principal.
  - Limpiar recursos con `docker compose down --volumes`.

## Punto de partida: workflow dummy (010)
En cada repo de proyecto (Python/Java) tienes ficheros de referencia:
- `.github/workflows/workflow-dummy.yml.template` (plantilla inicial)
- `.github/workflows/workflow.yml.template` (estructura final / guía, sin “código copiable”)
- `.github/workflows/workflow-full.yml.template` (estructura completa con TODOs y opcionales)

Tu fichero “real” para que GitHub lo ejecute debe ser:
- `.github/workflows/ci.yml`

Ejercicio 0 (setup):
1) Crea la carpeta `.github/workflows/` (si no existe).
2) Copia `.github/workflows/workflow-dummy.yml.template` a `.github/workflows/ci.yml`.
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
- Environments (recomendado en este curso):
  - Settings -> Environments -> **New environment**
  - Crea `DEV` y `PRO`
  - En cada environment:
    - Añade **Environment variables** y **Environment secrets**
    - (Opcional) añade protection rules si quieres aprobaciones manuales
- Niveles:
  - **Repository**: aplica a todos los workflows del repo.
  - **Environment**: aplica solo cuando el job usa `environment:` (permite protecciones).
  - **Organization** (si aplica): centraliza para muchos repos.

Tarea (usando Environments):
1) Crea environments `DEV` y `PRO`:
   - Settings -> Environments -> New environment
2) Dentro de cada environment crea variables, por ejemplo:
   - `REGISTRY_HOST` (ej. `local-registry:5000` o `ghcr.io`)
   - `ARTIFACTORY_URL` (ej. `http://artifactory:8081/artifactory`)
3) Dentro de cada environment crea secrets, por ejemplo:
   - `REGISTRY_USER`, `REGISTRY_PASSWORD`
   - `ARTIFACTORY_USER`, `ARTIFACTORY_PASSWORD` (o token)
4) En el workflow, declara el environment en el job:
   - `environment: DEV` o `environment: PRO`
   - Recomendado: seleccionar dinámicamente según rama:
     - `environment: ${{ github.ref_name == 'master' && 'PRO' || 'DEV' }}`
5) En el workflow, referencia así:
   - Variables: `${{ vars.REGISTRY_HOST }}` / `${{ vars.ARTIFACTORY_URL }}`
   - Secrets: `${{ secrets.REGISTRY_PASSWORD }}`

Importante:
- Estas variables/secrets son **por repositorio**. Debes crearlas en **cada proyecto** (Python y Java) para que los workflows funcionen.

Notas:
- Si no quieres usar environments, puedes crear variables/secrets a nivel Repository.
- Las variables/secretos de Environment solo están disponibles si el job define `environment:`.

Notas importantes (seguridad):
- Los secrets se enmascaran en logs, pero no imprimas secretos deliberadamente.
- Usa **Environments** (DEV/PRO) si quieres separar credenciales y proteger PRO con aprobaciones.

Referencia:
- https://docs.github.com/actions/security-guides/encrypted-secrets
- https://docs.github.com/actions/deployment/targeting-different-environments/using-environments-for-deployment

Nota equivalente en Azure DevOps (para práctica 5):
- Crear variables:
  - Pipeline -> Edit -> Variables (UI) o en YAML con `variables:`
  - Variable Groups: Pipelines -> Library -> Variable groups
- Referenciar variables en YAML:
  - En scripts: `$(VAR_NAME)`
  - En `env:`: `VAR_NAME: $(VAR_NAME)`
- Variables secretas:
  - Marca como secret en UI o en Variable Group
  - Se enmascaran en logs por defecto

Referencias Azure DevOps:
- https://learn.microsoft.com/azure/devops/pipelines/process/variables
- https://learn.microsoft.com/azure/devops/pipelines/library/variable-groups

### Ejercicio 6 - Docker como agente: job en contenedor
Objetivo: ejecutar CI dentro de un contenedor, igual que hacíamos con `agent docker` en Jenkins.

Tarea:
- Python: construye una imagen de CI (`devops/ci.Dockerfile`) y ejecuta `make lint`/`make test` dentro de esa imagen.
- Java: construye una imagen de CI (`devops/ci.Dockerfile`) y ejecuta `make lint`/`make test` dentro de esa imagen.

Pista:
- En GitHub Actions puedes usar `container:` a nivel de job o ejecutar `docker run` con la imagen de CI.

Referencia:
- https://docs.github.com/actions/using-jobs/running-jobs-in-a-container

### Ejercicio 7 - Build de imagen Docker en CI
Objetivo: construir la imagen Docker del repo (sin despliegue aún).

Tarea:
- Construye la imagen con el mismo tag que usa tu `docker-compose.yml`, para que luego CD pueda arrancar por compose.

### Ejercicio 8 - Condiciones por rama: CD solo en `develop/master`
Objetivo: simular despliegue por entornos.

Tarea:
- Crea un job `cd` (o stage equivalente) que se ejecute si:
  - `inputs.RUN_CD == true` (workflow dispatch), **o**
  - `github.ref_name` es `develop` o `master`

Pista:
- Usa `if:` a nivel de job.

Referencia:
- https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idif

### Ejercicio 9 - Simulación de despliegue con Docker Compose + cleanup
Objetivo: simular CD en un runner efímero:
1) `docker compose up -d`
2) mostrar logs del contenedor principal
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

### Opcional C - Librería común (composite action) en repo IaC
Objetivo: reutilizar pasos comunes entre Python y Java.

Tarea:
1) usando una acción compuesta en el repo IaC:
   - Ruta: `.github/actions/devops-lib/action.yml`
2) Encapsula al menos:
   - Build de la imagen de CI (`devops/ci.Dockerfile`)
   - Ejecución de comandos dentro de esa imagen
3) Usa esa acción en los workflows de Python y Java (opcional):
   - `uses: <org>/devops-training-iac-devops/.github/actions/devops-lib@<ref>`

Configuración en GitHub (librería común):
1) Publica el repo IaC con la acción compuesta en una rama o tag estable:
   - Recomendado: crear un tag (ej. `v1.0.0`) o usar una rama específica (`main`/`develop`).
2) En el repo consumidor (Python/Java), referencia la acción:
   - `uses: <org>/devops-training-iac-devops/.github/actions/devops-lib@<ref>`
   - `@<ref>` debe ser un tag o branch existente.
3) Verifica permisos:
   - Actions -> General -> Workflow permissions: `Read repository contents`
   - Si el repo IaC es privado, el repo consumidor debe tener acceso (mismo org o permisos explícitos).

Gestión de variables de entorno y secretos:
- Variables (Actions -> Variables):
  - Settings -> Secrets and variables -> Actions -> Variables
  - Usa `${{ vars.VAR_NAME }}` en el workflow.
  - Útil para `REGISTRY_HOST`, `ARTIFACTORY_URL`, `REGISTRY_REPO`, etc.
- Secrets (Actions -> Secrets):
  - Settings -> Secrets and variables -> Actions -> Secrets
  - Usa `${{ secrets.SECRET_NAME }}` en el workflow.
  - Útil para `REGISTRY_USER`, `REGISTRY_PASSWORD`, `ARTIFACTORY_USER`, `ARTIFACTORY_PASSWORD`.
- Scope recomendado:
  - Repository: para prácticas del curso.
  - Environment: si quieres diferenciar DEV/PRO con aprobaciones.

Referencias oficiales:
- Reusable workflows/composite actions: https://docs.github.com/actions/creating-actions/creating-a-composite-action
- Sharing actions in a repo: https://docs.github.com/actions/creating-actions/sharing-actions
- Variables: https://docs.github.com/actions/learn-github-actions/variables
- Secrets: https://docs.github.com/actions/security-guides/encrypted-secrets

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

### Paso 2 - Usar el runner ya incluido en el stack IaC
En `devops-training-iac-devops/docker-compose.yml` ya existe el servicio `github-runner` con perfil opcional `github-runner`.

Variables necesarias:
- `GITHUB_RUNNER_REPO_URL` (ej. `https://github.com/<org>/<repo>`)
- `GITHUB_RUNNER_TOKEN` (token de registro o PAT según tu estrategia)
- `GITHUB_RUNNER_NAME` (opcional)
- `GITHUB_RUNNER_LABELS` (opcional, recomendado: `self-hosted,linux,docker`)

Plantilla incluida:
- `devops-training-iac-devops/.env.github-runner.example`

### Paso 3 - Ejecutar el runner con Docker Compose
Desde `devops-training-iac-devops/`:
```bash
cp .env.github-runner.example .env.github-runner
# Edita .env.github-runner con valores reales
docker compose --env-file .env.github-runner --profile github-runner up -d github-runner
docker compose --env-file .env.github-runner logs -f github-runner
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
