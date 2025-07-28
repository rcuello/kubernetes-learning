# ⚙️ Instalación de Helm

## 📦 ¿Qué vamos a hacer?

En este documento aprenderás cómo instalar Helm en tu máquina local (Linux, macOS, Windows), validar su funcionamiento, y conectarlo con un clúster Kubernetes (por ejemplo, usando Minikube).

---

## 🧰 Requisitos

Antes de instalar Helm asegúrate de tener:

* ✅ Un clúster Kubernetes en funcionamiento (Minikube, Docker Desktop, etc.)
* ✅ `kubectl` correctamente configurado
* ✅ Acceso a la terminal o consola con permisos de administrador

---

## 💻 Instalación por sistema operativo

### 🐧 Linux (vía script oficial)

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

O usando `apt` (Debian/Ubuntu):

```bash
sudo apt update
sudo apt install helm
```

### 🍏 macOS (Homebrew)

```bash
brew install helm
```

### 🪟 Windows (Chocolatey o Scoop)

Con Chocolatey:

```powershell
choco install kubernetes-helm
```

Con Scoop:

```powershell
scoop install helm
```

---

## 🧪 Verificar instalación

```bash
helm version
```

Salida esperada:

```
version.BuildInfo{Version:"v3.14.0", GitCommit:..., GoVersion:...}
```

---

## 📂 Validar conexión con tu clúster Kubernetes

```bash
kubectl config current-context
```

Asegúrate de estar conectado a un clúster válido. Si estás usando Minikube:

```bash
minikube start
kubectl config use-context minikube
```

Luego prueba que Helm se puede comunicar correctamente:

```bash
helm list
```

---

## 🔗 Agregar repositorios oficiales de Charts

Helm funciona con repositorios de Charts. Puedes agregar uno de los más usados:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

---

## 🧪 Prueba rápida: desplegar WordPress en Minikube

```bash
helm install mi-wordpress bitnami/wordpress
```

Luego puedes listar tu release:

```bash
helm list
```

Y acceder al servicio:

```bash
minikube service mi-wordpress
```

---

## 🧼 Limpiar (opcional)

```bash
helm uninstall mi-wordpress
```

---

## 📚 Recursos recomendados

* Documentación oficial: [https://helm.sh/docs/intro/install](https://helm.sh/docs/intro/install/)
* Charts oficiales: [https://artifacthub.io](https://artifacthub.io)


