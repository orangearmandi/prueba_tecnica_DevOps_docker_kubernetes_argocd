# DOCKER_KUBERNETES_MONGO_ALTADISPONIBILIDAD_GRAFENA_PROMETEUS

# Conjunto de Réplicas de MongoDB con Herramientas de Monitoreo

Este repositorio contiene los archivos de configuración y pasos para desplegar y gestionar un conjunto de réplicas de MongoDB en un entorno de Kubernetes. Además, incluye instrucciones para configurar herramientas de monitoreo como Prometheus y Grafana.

## Requisitos Previos

- Cluster de Kubernetes
- Herramienta de línea de comandos `kubectl` instalada
- Archivos de configuración para el conjunto de réplicas de MongoDB, Mongo Express, Prometheus y Grafana

---

## Pasos para Desplegar y Gestionar los Servicios

### 1. Desplegar el Conjunto de Réplicas de MongoDB

Aplica el archivo de configuración para el conjunto de réplicas de MongoDB:
```bash
kubectl apply -f mongo-replica-set.yaml
```

### 2. Validar los Pods de MongoDB

Verifica si los pods de MongoDB están en ejecución:
```bash
kubectl get pods -l app=mongo
```
### RESPUESTA 

```bash
NAME                  READY   STATUS    RESTARTS      AGE
mongo-replica-set-0   1/1     Running   1 (19m ago)   8h
mongo-replica-set-1   1/1     Running   2 (19m ago)   8h
mongo-replica-set-2   1/1     Running   1 (19m ago)   8h   
```


### 3. Inicializar el Conjunto de Réplicas de MongoDB

Ejecuta el siguiente comando para emparejar los nodos del conjunto de réplicas de MongoDB:
```bash
kubectl exec -it mongo-replica-set-0 -- mongosh --eval "
rs.initiate({
  _id: 'rs0',
  members: [
    { _id: 0, host: 'mongo-replica-set-0.mongo:27017' },
    { _id: 1, host: 'mongo-replica-set-1.mongo:27017' },
    { _id: 2, host: 'mongo-replica-set-2.mongo:27017' }
  ]
})"
```

### 4. Escalar el Conjunto de Réplicas de MongoDB

Para escalar el conjunto de réplicas a un máximo de 3 réplicas y un mínimo de 0 (para desactivar todo):
```bash
kubectl scale statefulset mongo-replica-set --replicas=3
```

### 5. Habilitar Mongo Express

Aplica el archivo de configuración para habilitar el cliente Mongo Express:
```bash
kubectl apply -f mongo-express-no-auth.yaml
```

### 6. Habilitar Herramientas de Monitoreo

Despliega Prometheus y Grafana utilizando los archivos de configuración:
```bash
kubectl apply -f prometheus-configmap.yaml
kubectl apply -f prometheus-deployment.yaml
kubectl apply -f grafana-deployment.yaml
```

### 7. Validar Todos los Servicios y Pods

Lista todos los pods y servicios para asegurarte de que están en ejecución:
```bash
kubectl get pods
kubectl get services
```

### REPUESTA

```bash
NAME                             READY   STATUS    RESTARTS      AGE
grafana-68975f8b9b-5fb6q         1/1     Running   1 (34m ago)   8h
mongo-express-5dc9c6c76f-6dp6g   1/1     Running   1 (34m ago)   8h
mongo-replica-set-0              1/1     Running   1 (34m ago)   8h
mongo-replica-set-1              1/1     Running   2 (34m ago)   8h
mongo-replica-set-2              1/1     Running   1 (34m ago)   8h
prometheus-747775648b-pjvsk      1/1     Running   1 (34m ago)   8h
```
##
## Desactivar Pods y Contenedores
##

### Desactivar Servicios Individuales

Para eliminar servicios o despliegues específicos:
```bash
kubectl delete grafana
kubectl delete kubernetes 
kubectl delete mongo         
kubectl delete mongo-express   
kubectl delete prometheus
```

### Desactivar Todos los Servicios Globalmente

Para eliminar todos los jobs, statefulsets, despliegues y servicios:
```bash
kubectl delete job --all
kubectl delete statefulset --all
kubectl delete deployment --all
kubectl delete svc --all
```

---

## Documentación de Herramientas de Monitoreo

### Docker

Asegúrate de que Docker esté instalado en tu sistema y en ejecución. Para despliegues en Kubernetes, Docker se utiliza generalmente para construir y gestionar imágenes de contenedores.

### Kubernetes

- Gestiona tu clúster usando comandos de `kubectl`.
- Despliega configuraciones almacenadas en archivos YAML.
- Valida los servicios utilizando comandos `kubectl get`.

### Grafana

- Accede al panel de control de Grafana después del despliegue.
- Agrega fuentes de datos como Prometheus para visualizar métricas.
- Utiliza paneles predefinidos o personalizados para monitorear el conjunto de réplicas de MongoDB.

### Prometheus

- Prometheus recopila métricas de los pods de MongoDB.
- Asegúrate de que la configuración de Prometheus apunte a los endpoints correctos de métricas de MongoDB.
- Consulta y visualiza métricas usando Prometheus o Grafana.

---

## Notas

- Siempre valida las configuraciones antes de aplicarlas a tu clúster.
- Asegúrate de que el clúster de Kubernetes tenga suficientes recursos para todos los despliegues.
- Utiliza herramientas de monitoreo para rastrear el rendimiento y detectar problemas a tiempo.
