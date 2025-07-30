# 🔄 Rolling Update en Deployments de Kubernetes

## 🎯 Objetivo

Actualizar la imagen de un Deployment de forma segura y sin tiempo de inactividad utilizando el mecanismo de **Rolling Update** que Kubernetes ofrece por defecto.

---

## 🧠 ¿Qué es un Rolling Update?

Es el proceso automático mediante el cual Kubernetes:

1. **Crea nuevos Pods** con la nueva imagen.
2. **Elimina progresivamente** los Pods antiguos.
3. **Mantiene el estado deseado** (número de réplicas, disponibilidad, etc.).
4. **Soporta rollback automático** si algo falla (según configuración).

> 💡 Kubernetes garantiza que siempre haya el mínimo número de Pods disponibles durante la actualización.

---

## ⚙️ Cómo hacer un Rolling Update

### ✅ Opción 1: Usar `kubectl set image`

```bash
kubectl set image deployment/<deployment-name> <container-name>=<new-image>
```

**Ejemplo:**

```bash
kubectl set image deployment/hello-deployment hello-app=gcr.io/google-samples/hello-app:2.0
```

### ✅ Opción 2: Editar directamente el Deployment

```bash
kubectl edit deployment <deployment-name>
```

Busca la sección `spec.template.spec.containers.image` y actualiza la versión de la imagen.

---

## 🔍 Verificar el progreso del update

```bash
kubectl rollout status deployment/<deployment-name>
```

Ejemplo:

```bash
kubectl rollout status deployment/hello-deployment
```

---

## ⏪ Revertir cambios (Rollback)

Si algo sale mal, puedes deshacer la actualización:

```bash
kubectl rollout undo deployment/<deployment-name>
```

---

## 📌 Estrategia de actualización

El tipo de update por defecto de un Deployment es `RollingUpdate`. Puedes personalizarlo en el YAML:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

* `maxSurge`: Número de Pods nuevos que se pueden crear por encima del número deseado.
* `maxUnavailable`: Número de Pods existentes que pueden estar inactivos durante la actualización.

---

## 🧪 Validación final

```bash
kubectl get pods
kubectl describe deployment <deployment-name>
kubectl logs <pod-name>
```

Asegúrate de que los nuevos Pods están corriendo y la imagen fue actualizada correctamente.

---

## ✅ Buenas prácticas

* Siempre usa tags versionados de imágenes (`v1.0`, `v2.1`, etc.) y evita usar `:latest`.
* Automatiza pruebas de salud con probes (`livenessProbe`, `readinessProbe`).
* Usa `kubectl rollout pause` y `kubectl rollout resume` para actualizaciones controladas.
* Monitorea con herramientas como Prometheus, Grafana o Lens.

---

## 🧼 Comandos útiles

| Acción                           | Comando                                     |
| -------------------------------- | ------------------------------------------- |
| Ver historial de actualizaciones | `kubectl rollout history deployment/<name>` |
| Pausar actualización             | `kubectl rollout pause deployment/<name>`   |
| Reanudar actualización           | `kubectl rollout resume deployment/<name>`  |

---

Este documento complementa la explicación general sobre Deployments y cubre el proceso de actualización de forma segura y declarativa en entornos productivos.
