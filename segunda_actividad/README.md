# README

## Pasos para Instalar y Configurar el Entorno

### 1. Instalar Minikube

#### Descarga e instalación:

1. **Descargar Minikube**:
   - Descarga el instalador de la última versión desde [Minikube Releases](https://github.com/kubernetes/minikube/releases).

2. **Usando PowerShell** (Ejecutar como Administrador):
   ```powershell
   New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
   Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing
   ```

3. **Agregar Minikube a la variable PATH**:
   ```powershell
   $oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
   if ($oldPath.Split(';') -inotcontains 'C:\minikube'){
       [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine)
   }
   ```

### 2. Instalar y Configurar ArgoCD

#### Crear el namespace y aplicar el manifiesto:

1. **Crear namespace**:
   ```bash
   kubectl create namespace argocd
   ```

2. **Aplicar instalación de ArgoCD**:
   ```bash
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. **Verificar pods**:
   ```bash
   kubectl -n argocd get pods
   ```

#### Exponer el servidor ArgoCD:

1. Crear un archivo `argocd-server-patch.yaml` con el contenido:
   ```yaml
   spec:
     type: NodePort
   ```

2. Aplicar el parche:
   ```bash
   kubectl -n argocd patch svc argocd-server --patch-file argocd-server-patch.yaml
   ```

3. Obtener la URL pública para acceder al servicio:
   ```bash
   minikube service argocd-server -n argocd --url
   ```

#### Obtener la contraseña del administrador:

```powershell
[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String((kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}")))
```

#### Validar los servicios:

```bash
kubectl -n argocd get svc
```

### 3. Dockerizar un Proyecto Node.js

#### Crear archivos requeridos:

1. **Crear un `Dockerfile` con el siguiente contenido**:
   ```dockerfile
   FROM node:18-alpine

   WORKDIR /app

   COPY package*.json ./

   RUN npm install && npm audit fix --force

   COPY . .

   RUN npm run build

   EXPOSE 3000

   CMD ["npm", "start"]
   ```

2. **Crear un archivo `.dockerignore`**:
   ```
   node_modules
   npm-debug.log
   .next
   ```

#### Construir la imagen Docker:

```bash
docker build -t frontend-challenge-base:latest .
```

#### Etiquetar y publicar la imagen:

```bash
docker tag frontend-challenge-base:latest orangearmandi/frontend-challenge-base:latest
docker push orangearmandi/frontend-challenge-base:latest
```

### 4. Desplegar la Aplicación con ArgoCD

#### Crear manifiestos de Kubernetes:

1. Crear un archivo `application.yaml`:
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: frontend-challenge
     namespace: argocd
   spec:
     project: default
     source:
       repoURL: 'https://github.com/usuario/repo'
       targetRevision: HEAD
       path: '.'
     destination:
       server: 'https://kubernetes.default.svc'
       namespace: default
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
   ```

2. Aplicar el manifiesto:
   ```bash
   kubectl apply -f application.yaml
   ```

### 5. Crear Pipeline CI/CD con GitHub Actions

#### Crear un archivo `.github/workflows/docker-deploy.yaml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Log in to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker image
        run: |
          docker build -t frontend-challenge-base:latest .
          docker tag frontend-challenge-base:latest orangearmandi/frontend-challenge-base:latest

      - name: Push Docker image
        run: docker push orangearmandi/frontend-challenge-base:latest
```

### Notas Finales

1. Cambia `usuario/repo` en el archivo `application.yaml` por tu repositorio.
2. Configura tus secretos `DOCKER_USERNAME` y `DOCKER_PASSWORD` en GitHub Actions.
3. Asegúrate de que el clúster Kubernetes y ArgoCD estén configurados correctamente antes de realizar el despliegue.
