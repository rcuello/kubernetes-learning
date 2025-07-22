## 🧰 ¿Qué es kubectl?

**`kubectl`** es la **herramienta de línea de comandos** para interactuar con un cluster de Kubernetes.

Te permite **crear, leer, actualizar y eliminar** (CRUD) recursos como pods, deployments, services, configmaps, etc.

### 📘 Sintaxis básica:

```bash
kubectl get pods             # Lista los pods
kubectl get services         # Lista servicios
kubectl apply -f app.yaml    # Aplica un manifiesto
kubectl logs mi-pod          # Ver logs de un pod
kubectl exec -it mi-pod -- bash # Entrar al pod
```

> `kubectl` usa un archivo llamado `~/.kube/config` para saber **a qué cluster se está conectando**.

---

## 🧩 Relación entre minikube y kubectl

| Herramienta | Rol                                                  |
| ----------- | ---------------------------------------------------- |
| `minikube`  | Crea y gestiona el cluster local                     |
| `kubectl`   | Se conecta al cluster (local o remoto) para operarlo |

Cuando ejecutas `minikube start`, este configura automáticamente `kubectl` para que se conecte a ese cluster.

---

## 📌 Ejemplo completo de flujo local

```bash
minikube start                        # Levantas el cluster local
kubectl apply -f deployment.yaml     # Desplegas tu app
kubectl get pods                     # Verificas que esté corriendo
kubectl port-forward svc/mi-servicio 8080:80  # Expones localmente
```

