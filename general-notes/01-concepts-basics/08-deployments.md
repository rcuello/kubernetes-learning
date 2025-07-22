# 📦 Deployments en Kubernetes

Un **Deployment** en Kubernetes es un recurso que permite administrar y mantener de forma declarativa un conjunto de Pods y sus respectivos ReplicaSets. Es uno de los controladores más comunes, ya que automatiza muchas tareas operativas como actualizaciones, rollbacks y autoescalado.

## 🧠 ¿Qué hace un Deployment?

- Crea y administra un ReplicaSet.
- Asegura que siempre exista la cantidad deseada de Pods (`replicas`).
- Permite hacer *rolling updates* (actualizaciones sin downtime).
- Permite hacer *rollbacks* a versiones anteriores si algo falla.

## 🏗️ Estructura básica

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
````

## 🔁 Relación con ReplicaSet

Cada Deployment crea y gestiona internamente un **ReplicaSet**, que a su vez se encarga de mantener la cantidad de Pods activos. Tú no creas el ReplicaSet directamente; Kubernetes lo hace al procesar el Deployment.

## 🔧 Comandos útiles

* Crear desde archivo:

  ```bash
  kubectl apply -f deployments.yaml
  ```

* Ver los deployments:

  ```bash
  kubectl get deployments
  ```

* Ver detalles:

  ```bash
  kubectl describe deployment nombre-del-deployment
  ```

* Escalar:

  ```bash
  kubectl scale deployment nombre-del-deployment --replicas=5
  ```

* Eliminar:

  ```bash
  kubectl delete deployment nombre-del-deployment
  ```

## 📌 Recomendaciones

* Usa Deployments para cualquier aplicación que necesite ser actualizada, escalada o desplegada con alta disponibilidad.
* Para cargas simples que no requieran actualización, podrías usar directamente un ReplicaSet o incluso un Pod.

---

