# 🧩 Configurar `kubectl` en Windows para trabajar con Minikube

`kubectl` es la herramienta de línea de comandos para interactuar con clústeres de Kubernetes. Esta guía explica cómo instalarla, configurarla y verificar que esté funcionando correctamente en un entorno Windows con Minikube.

---

## 📋 Requisitos previos

Antes de continuar, asegúrate de:

- Tener instalado Minikube con Docker como driver. (Ver `01-instalar-minikube-windows-docker.md`)
- Tener acceso a PowerShell, CMD o una terminal WSL.
- Tener conexión a Internet para descargar herramientas.

---

## ✅ 1. Instalar `kubectl` en Windows

### 🔧 Opción A: Usando Chocolatey

```powershell
choco install kubernetes-cli
````

### 🧰 Opción B: Manual

1. Ve a la [página oficial de descargas](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/).
2. Descarga el binario de `kubectl.exe`.
3. Coloca el archivo en una carpeta como `C:\kubectl`.
4. Agrega esta ruta a la variable de entorno `PATH`.

---

## 🔍 2. Verificar instalación

En una terminal, ejecuta:

```powershell
kubectl version --client
```

Debe mostrar la versión del cliente, por ejemplo:

```
Client Version: v1.30.1
```

---

## 🔧 3. Configurar `kubectl` para hablar con Minikube

Cuando instalas Minikube y lo inicias con:

```powershell
minikube start --driver=docker
```

automáticamente genera y configura un archivo:

```
C:\Users\<TU_USUARIO>\.kube\config
```

Este archivo contiene el contexto, credenciales y clúster actual.

Puedes verificar el contexto activo con:

```powershell
kubectl config get-contexts
kubectl config current-context
```

Si por alguna razón `kubectl` no reconoce Minikube, puedes forzar la configuración con:

```powershell
minikube update-context
```

---

## 🧪 4. Probar que `kubectl` funciona

```powershell
kubectl get nodes
```

Si todo está bien, deberías ver algo como:

```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   2m    v1.30.1
```

---

## 💡 Comandos comunes de `kubectl`

| Comando                          | Descripción                                 |
| -------------------------------- | ------------------------------------------- |
| `kubectl get pods`               | Lista los pods                              |
| `kubectl get svc`                | Lista los servicios                         |
| `kubectl apply -f archivo.yaml`  | Aplica un manifiesto                        |
| `kubectl delete -f archivo.yaml` | Elimina recursos definidos en el manifiesto |
| `kubectl describe pod <nombre>`  | Detalla información de un pod               |
| `kubectl logs <pod>`             | Muestra logs de un pod                      |

---

## 🎯 Analogía con el mundo real

> **`kubectl` es como el control remoto de tu clúster de Kubernetes.**
> Así como con un control remoto puedes cambiar de canal, subir el volumen o apagar la TV, con `kubectl` puedes manejar los recursos de tu clúster: lanzar nuevas apps (pods), ver su estado, reiniciarlas o eliminarlas.

---

## 📚 Recursos útiles

* [Documentación oficial de `kubectl`](https://kubernetes.io/docs/reference/kubectl/)
* [Cheat sheet de kubectl](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [Archivos de configuración de kube](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
