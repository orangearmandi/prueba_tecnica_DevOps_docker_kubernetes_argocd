# Instrucciones para Configuración y Despliegue

## Instalar Minikube
Descarga e instala la última versión de Minikube.

### Usando PowerShell
Ejecuta los siguientes comandos en PowerShell:

```powershell
New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing
```

### Agregar `minikube.exe` al PATH
Asegúrate de ejecutar PowerShell como Administrador. Luego, agrega Minikube al PATH del sistema:

```powershell
$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine)
}
```

## Configurar ArgoCD

### Crear el Namespace para ArgoCD
```bash
kubectl create namespace argocd
```

### Instalar ArgoCD
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Verificar los Pods de ArgoCD
```bash
kubectl -n argocd get pods
```

### Actualizar el Servicio `argocd-server`
```bash
kubectl -n argocd patch svc argocd-server --patch-file argocd-server-patch.yaml
```

### Obtener la URL pública del servicio
```bash
minikube service argocd-server -n argocd --url
```

### Generar la contraseña del administrador
```powershell
[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String((kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}")))
```

### Validar los Servicios
```bash
kubectl -n argocd get svc
```

### Aplicar configuración personalizada
```bash
kubectl apply -n argocd -f .\service.yaml
kubectl apply -n argocd -f .\deployment.yaml
```

### Acceder al servicio de la aplicación
```bash
minikube service frontend-challenge-service -n argocd
kubectl get svc frontend-challenge-service -n argocd
```

### Sincronizar con ArgoCD
`Applications>NEW APP>EDIT AS YAML>COPIAR ARCHIVO>CREATE>SYNCRONIZARLO`
Agrega `fronted-application.yaml` a ArgoCD y sincronízalo.

## Dockerizar un Proyecto

### Crear Archivos Necesarios
1. Crea un archivo `Dockerfile` con la configuración adecuada.
2. Crea un archivo `.dockerignore`.

### Construir la Imagen Docker
```bash
docker build -t frontend-challenge-base:latest .
```

### Etiquetar la Imagen
```bash
docker tag frontend-challenge-base:latest orangearmandi/frontend-challenge-base:latest
```

### Subir la Imagen al Registro
```bash
docker push orangearmandi/frontend-challenge-base:latest