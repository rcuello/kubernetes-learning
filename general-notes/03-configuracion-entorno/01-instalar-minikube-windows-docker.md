# ⚙️ Instalar Minikube en Windows (usando Docker como driver)

Esta guía te ayudará a instalar **Minikube** en Windows utilizando **Docker como driver**. Es la forma más sencilla si ya tienes Docker Desktop instalado, permitiéndote levantar un clúster Kubernetes local sin depender de máquinas virtuales adicionales.

---

## 📋 Requisitos previos

| Requisito       | Estado | Notas |
|-----------------|--------|-------|
| ✅ Windows 10/11 64-bit | Obligatorio | Con WSL2 activado |
| ✅ Docker Desktop | Obligatorio | Con Kubernetes desactivado desde Docker |
| ✅ Virtualización habilitada | Obligatorio | Para que Docker funcione |
| ✅ PowerShell o CMD | Recomendado | Para ejecutar comandos |
| ❌ Hyper-V o VirtualBox | **No necesarios** | Gracias al driver Docker |

---

## 🛠️ Paso a paso: Instalación

### 1️⃣ Instalar Docker Desktop

Descarga desde [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

- Habilita la opción de **WSL 2** durante la instalación.
- Asegúrate de que Docker esté corriendo correctamente.

### 2️⃣ Instalar Minikube

#### Opción A: Usando Chocolatey

```powershell
choco install minikube
````

#### Opción B: Manual

* Descarga desde: [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)
* Agrega el ejecutable a la variable de entorno `PATH`.

### 3️⃣ Verificar instalación

```powershell
minikube version
```

---

## 🚀 Crear el clúster con Docker como driver

```powershell
minikube start --driver=docker
```

Esto lanzará un contenedor que actúa como un nodo Kubernetes local.

---

## ✅ Verificar el clúster

```powershell
kubectl get nodes
```

Deberías ver algo como:

```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   1m    v1.30.1
```

---

## 🧪 Desplegar una app de prueba

```powershell
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
kubectl expose deployment hello-minikube --type=NodePort --port=8080
minikube service hello-minikube
```

Esto abrirá el navegador con un mensaje de prueba.

---

## 📁 Sobre el archivo de configuración `~/.kube/config`

Este archivo almacena las credenciales y contexto de acceso a tu clúster Kubernetes.

### 📍 En Windows, lo encuentras en:

```text
C:\Users\<TU_USUARIO>\.kube\config
```

### 📄 Ejemplo de contenido:

```yaml
apiVersion: v1
clusters:
- cluster:
    server: https://127.0.0.1:6443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: ...
    client-key: ...
```

Puedes editar este archivo o usar comandos como:

```bash
kubectl config get-contexts
kubectl config use-context minikube
```

---

## 🧯 Comandos útiles

```bash
minikube stop            # Detener el clúster
minikube delete          # Eliminar el clúster y contenedor
minikube dashboard       # Abrir UI web de Kubernetes
```

---

## 🧠 Analogía para entender Minikube

> **Minikube es como tener un parque de diversiones portátil en tu laptop.**
> En lugar de construir todo un parque (clúster) en la nube, lo haces en miniatura usando contenedores. Tus juegos (aplicaciones) siguen siendo reales, las reglas (Kubernetes) son las mismas, pero puedes probarlo todo localmente, más rápido y sin costo.

---

## 📚 Recursos

* [Sitio oficial de Minikube](https://minikube.sigs.k8s.io/docs/)
* [kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [Minikube drivers](https://minikube.sigs.k8s.io/docs/drivers/)
