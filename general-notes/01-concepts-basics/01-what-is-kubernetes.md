# 🧠 Introducción a Kubernetes

## ¿Qué es Kubernetes?

Kubernetes (abreviado como **K8s**) es una plataforma **de código abierto** que automatiza el despliegue, el escalado y la gestión de **aplicaciones contenerizadas**. En esencia, convierte tu infraestructura en una plataforma **autosuficiente**, donde las aplicaciones pueden ejecutarse con alta disponibilidad, escalabilidad y recuperación automática.

Fue desarrollado originalmente por Google y ahora es mantenido por la **Cloud Native Computing Foundation (CNCF)**.

---

## 🚀 ¿Por qué usar Kubernetes?

Kubernetes resuelve muchos problemas reales que surgen cuando se despliegan aplicaciones modernas en producción:

| Problema                   | ¿Cómo lo resuelve Kubernetes?                                                                 |
|---------------------------|-----------------------------------------------------------------------------------------------|
| **Despliegues manuales**  | Automatiza los despliegues con control de versiones, rollback y actualizaciones progresivas. |
| **Escalado complejo**     | Escala automáticamente la cantidad de réplicas según el uso de CPU o reglas personalizadas.  |
| **Fallas de servicio**    | Detecta fallos y reinicia los contenedores automáticamente para mantener la aplicación viva. |
| **Configuraciones frágiles** | Gestiona configuraciones externas con ConfigMaps y Secrets.                              |
| **Ruteo y balanceo**      | Usa Services para exponer Pods con IPs estables, DNS y balanceo de carga interno.           |
| **Orquestación de múltiples contenedores** | Controla cientos o miles de contenedores distribuidos entre nodos.        |

---

## 🧱 Arquitectura de Kubernetes

```mermaid
graph TD
  subgraph Cluster
    A[Usuario] -->|kubectl| B[API Server]
    B --> C[Scheduler]
    B --> D[Controller Manager]
    B --> E["etcd (base de datos)"]
    B --> F[Worker Node]
    F --> G[Pod: App 1]
    F --> H[Pod: App 2]
  end
````

---

## 🔑 Conceptos clave

| Concepto       | Descripción                                                                                                |
| -------------- | ---------------------------------------------------------------------------------------------------------- |
| **Cluster**    | Grupo de nodos donde se ejecutan tus aplicaciones.                                                         |
| **Node**       | Máquina física o virtual dentro del cluster (puede ser de control o de trabajo).                           |
| **Pod**        | Unidad mínima de ejecución. Puede contener uno o más contenedores. Es efímero y reemplazable.              |
| **Deployment** | Objeto que gestiona la creación, actualización y escalado de Pods. Permite hacer despliegues sin downtime. |
| **Service**    | Exposición lógica de un conjunto de Pods. Ofrece una IP estable, DNS y balanceo de carga.                  |
| **kubectl**    | CLI oficial para interactuar con el clúster (ver, desplegar, modificar recursos).                          |
| **YAML**       | Formato declarativo usado para definir objetos Kubernetes: Pods, Deployments, Services, ConfigMaps, etc.   |

---

## 🛠️ Flujo de trabajo básico

```bash
# Iniciar minikube
minikube start

# Desplegar una aplicación
kubectl create deployment hello-k8s --image=nginx

# Exponerla como servicio
kubectl expose deployment hello-k8s --type=NodePort --port=80

# Ver el servicio
kubectl get services

# Abrir en el navegador (minikube)
minikube service hello-k8s
```

---

## 🧠 ¿Cómo piensa Kubernetes?

Kubernetes trabaja de forma **declarativa**: tú defines **el estado deseado** del sistema (por ejemplo: "quiero 3 instancias de mi aplicación corriendo"), y el *Control Plane* se encarga de alcanzar y mantener ese estado, incluso si hay fallos.

* Si un contenedor falla, lo reinicia.
* Si falta una réplica, la crea.
* Si actualizas la imagen, hace el cambio de forma controlada.
* Si algo sale mal, puede hacer rollback.

---

## 📦 Archivos YAML de ejemplo

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mi-app
  template:
    metadata:
      labels:
        app: mi-app
    spec:
      containers:
      - name: mi-contenedor
        image: nginx
        ports:
        - containerPort: 80
```

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mi-servicio
spec:
  type: NodePort
  selector:
    app: mi-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30001
```

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

## 📚 Recursos adicionales

* [Documentación oficial de Kubernetes](https://kubernetes.io/es/)
* [Guía interactiva: Kubernetes Playground](https://labs.play-with-k8s.com/)
* [Libro gratuito: The Illustrated Children's Guide to Kubernetes](https://www.cncf.io/phippy/)
* [Cheat Sheet de `kubectl`](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

