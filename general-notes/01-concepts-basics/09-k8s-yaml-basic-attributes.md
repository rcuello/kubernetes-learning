## 📘 Atributos importantes en archivos YAML de Kubernetes: ReplicaSet y Deployment

Este documento describe los campos clave que se utilizan al definir recursos `ReplicaSet` y `Deployment` en Kubernetes mediante archivos YAML.

---

### 🧱 Estructura general de un archivo YAML

Todo manifiesto de Kubernetes tiene una estructura básica común:

```yaml
apiVersion: <versión de la API>
kind: <tipo de recurso>
metadata:
  name: <nombre>
  labels:
    <clave>: <valor>
spec:
  ...
```

---

## 🔁 ReplicaSet

Un **ReplicaSet** asegura que haya un número específico de *Pods idénticos* ejecutándose en todo momento.

### 📄 Ejemplo básico

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

### 🧩 Atributos clave

| Atributo                        | Descripción                                                         |
| ------------------------------- | ------------------------------------------------------------------- |
| `apiVersion`                    | Versión de la API (para ReplicaSet es `apps/v1`)                    |
| `kind`                          | Tipo de recurso (`ReplicaSet`)                                      |
| `metadata.name`                 | Nombre único del ReplicaSet                                         |
| `spec.replicas`                 | Número deseado de réplicas (Pods)                                   |
| `spec.selector.matchLabels`     | Selector que identifica los Pods que este ReplicaSet debe gestionar |
| `spec.template`                 | Plantilla para los Pods que serán creados                           |
| `spec.template.metadata.labels` | Etiquetas de los Pods generados                                     |
| `spec.template.spec.containers` | Lista de contenedores a ejecutar por Pod                            |

---

## 🚀 Deployment

Un **Deployment** gestiona ReplicaSets y ofrece funcionalidades adicionales como *actualizaciones declarativas* (rolling updates) y *rollback*.

### 📄 Ejemplo básico

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deployment
spec:
  replicas: 4
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello-app
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
```

### 🧩 Atributos clave

| Atributo                                 | Descripción                                               |
| ---------------------------------------- | --------------------------------------------------------- |
| `apiVersion`                             | Versión de la API (para Deployment es `apps/v1`)          |
| `kind`                                   | Tipo de recurso (`Deployment`)                            |
| `metadata.name`                          | Nombre del Deployment                                     |
| `spec.replicas`                          | Cantidad de Pods deseada                                  |
| `spec.selector.matchLabels`              | Cómo seleccionar los Pods gestionados                     |
| `spec.template`                          | Definición del Pod a crear (igual que en ReplicaSet)      |
| `spec.strategy` *(opcional)*             | Estrategia de actualización (`RollingUpdate` por defecto) |
| `spec.revisionHistoryLimit` *(opcional)* | Cuántas versiones anteriores mantener para rollback       |

---

## 🧠 Recomendaciones para aprender

* Comienza usando `kubectl explain` para ver detalles de cada atributo:

  ```bash
  kubectl explain deployment.spec
  kubectl explain replicaset.spec.template
  ```
* Usa `kubectl get deployment -o yaml` para ver cómo Kubernetes estructura el recurso internamente.

