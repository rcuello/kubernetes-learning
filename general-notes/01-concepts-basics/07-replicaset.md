# 🔁 Kubernetes - ReplicaSets

## 📌 ¿Qué es un ReplicaSet?

Un **ReplicaSet** es un recurso de Kubernetes que garantiza que una cantidad específica de réplicas de un Pod estén siempre en ejecución.

> 🚀 **Analogía**: Si un Pod es un trabajador, un ReplicaSet es el supervisor que asegura que siempre haya una cantidad fija de trabajadores disponibles. Si uno se va, se contrata a otro.

---

## 🧠 Características Clave

| Característica          | Descripción                                                                 |
| ----------------------- | --------------------------------------------------------------------------- |
| Alta disponibilidad     | Si un Pod falla o se elimina, el ReplicaSet crea uno nuevo automáticamente. |
| Escalabilidad           | Permite aumentar o reducir el número de réplicas según la carga.            |
| Selección por etiquetas | Usa `selector` para identificar qué Pods controlar.                         |
| Control de estado       | Monitorea constantemente que el número de réplicas deseado esté activo.     |

---

## 📄 Ejemplo YAML de ReplicaSet

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mi-replicaset
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
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Explicación de campos importantes:

| Campo      | Descripción                                                               |
| ---------- | ------------------------------------------------------------------------- |
| `replicas` | Número de Pods que se desea tener activos.                                |
| `selector` | Define qué Pods están bajo el control del ReplicaSet (por sus etiquetas). |
| `template` | Plantilla que el ReplicaSet usará para crear nuevos Pods si es necesario. |

> ⚠️ El `selector.matchLabels` **debe coincidir exactamente** con los `labels` del `template`.

---

## 🛠️ Comandos Básicos

### Ver ReplicaSets disponibles

```bash
kubectl get rs
```

### Describir un ReplicaSet en detalle

```bash
kubectl describe rs mi-replicaset
```

### Escalar un ReplicaSet manualmente

```bash
kubectl scale rs mi-replicaset --replicas=5
```

### Eliminar un ReplicaSet desde archivo YAML

```bash
kubectl delete -f replicaset.yml
```

### Eliminar un ReplicaSet por nombre

```bash
kubectl delete rs mi-replicaset
```

---

## 🔍 Inspección

### Ver los Pods gestionados por el ReplicaSet

```bash
kubectl get pods -l app=nginx
```

---

## 📚 ReplicaSet vs Deployment

| Recurso    | Descripción                                                              |
| ---------- | ------------------------------------------------------------------------ |
| ReplicaSet | Asegura que haya N Pods activos. No maneja actualizaciones ni versiones. |
| Deployment | Usa ReplicaSets, pero además permite actualizaciones, rollback y más.    |

> ✅ **Recomendado:** Usa **Deployments** en lugar de ReplicaSets directamente para facilitar actualizaciones y gestión de versiones.

---

## 🧠 Buenas prácticas

* No uses ReplicaSets directamente para producción, mejor usa **Deployments**.
* Verifica que el `selector` coincida con los `labels` del template para evitar errores.
* Supervisa eventos con `kubectl describe` si los Pods no se crean.
* Usa etiquetas (`labels`) descriptivas y consistentes para facilitar la selección de Pods.

---

¿Te gustaría que preparemos un documento complementario para Deployments o YAML comparativo? 🚀
