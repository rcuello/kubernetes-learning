# 📦 Deployments en Kubernetes

Un **Deployment** en Kubernetes es un recurso de nivel superior que permite gestionar de forma declarativa un conjunto de Pods y sus respectivos ReplicaSets. Automatiza tareas operativas clave como actualizaciones, escalado y recuperación ante fallos.

---

## 🧠 ¿Qué es y para qué sirve?

Un Deployment:

* Crea y administra automáticamente uno o varios ReplicaSets.
* Mantiene el número deseado de réplicas de Pods.
* Permite actualizaciones sin downtime (*rolling updates*).
* Permite revertir a versiones anteriores si ocurre un fallo (*rollback*).

Es ideal para gestionar aplicaciones de forma continua y segura.

---

## 🏗️ Estructura de un archivo `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ejemplo-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mi-app
  template:
    metadata:
      labels:
        app: mi-app
    spec:
      containers:
        - name: contenedor-app
          image: mi-imagen:1.0
          ports:
            - containerPort: 8080
```

* `replicas`: número deseado de Pods.
* `selector`: define qué Pods deben ser gestionados.
* `template`: especifica el contenido de los Pods.

---

## 🔁 Relación entre Deployment y ReplicaSet

Al crear un Deployment, Kubernetes genera y gestiona automáticamente un **ReplicaSet**, el cual es responsable de mantener los Pods activos.

> ✅ No necesitas crear manualmente el ReplicaSet; el Deployment lo hace por ti.

---
## ⚙️ Comandos útiles

### Crear un Deployment  

```bash
kubectl apply -f deployment.yaml
```

### Listar deployments

```bash
kubectl get deployments
```

### Ver detalles
```bash
kubectl describe deployment nombre-del-deployment
```

### Eliminar el Deployment
```bash
kubectl delete deployment nombre-del-deployment
```

### Modificar imagen del deployment
```bash
kubectl set image deployment/<nombre-del-deployment> <container>=<nueva-imagen>

# Ejemplo
kubectl set image deployment/hello-deployment hello-app=gcr.io/google-samples/hello-app:2.0
# Ver cambio
kubectl describe deployments hello-deployment
```

---

## 📌 Buenas prácticas

* Usa Deployments para aplicaciones que requieren actualización, escalabilidad o alta disponibilidad.
* Aprovecha el `strategy.rollingUpdate` para minimizar tiempos de inactividad.
* Aplica etiquetas y anotaciones descriptivas para facilitar la gestión y el monitoreo.

