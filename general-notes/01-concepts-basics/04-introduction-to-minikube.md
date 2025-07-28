# 🐳 Introducción a Minikube

## ¿Qué es Minikube?

**Minikube** es una herramienta de código abierto que permite ejecutar un clúster de **Kubernetes** localmente en tu equipo (Windows, macOS o Linux). Está diseñada para facilitar el desarrollo, la experimentación y las pruebas sin necesidad de usar infraestructura en la nube.

Minikube simula un entorno Kubernetes real al crear una máquina virtual (o contenedor, dependiendo del driver) con todos los componentes necesarios para desplegar, exponer y monitorear aplicaciones.

Imagina que Kubernetes es como una gran **fábrica automatizada** donde se construyen y gestionan aplicaciones (contenedores) con reglas muy precisas.

Pero esa fábrica normalmente está en la nube (como en Amazon EKS, Google GKE, Azure AKS), lo que puede hacerla difícil y costosa de usar si solo quieres practicar, aprender o hacer pruebas.

Aquí es donde entra **Minikube**.

---

### 🧠 Analogía: Minikube es como un "simulador de vuelo"

* **Kubernetes real** es como un **avión comercial** gigante que necesita una pista, torre de control, permisos, y cuesta mucho operarlo.
* **Minikube** es como un **simulador de vuelo** que puedes correr en tu laptop. Tiene los mismos controles y comportamientos básicos del avión real, pero todo está contenido y simplificado.
* Puedes **practicar, fallar, probar maniobras** y aprender a volar sin arriesgar nada ni pagar por el uso del avión real.

---

## 🚀 Principales características

* 🧩 **Kubernetes real en local**: Minikube ejecuta una versión real de Kubernetes, permitiendo probar tus manifiestos y controladores sin depender de servicios cloud.
* ⚙️ **Compatibilidad multiplataforma**: Disponible para Windows, macOS y Linux.
* 🧪 **Ideal para pruebas y desarrollo local**: Entorno rápido, desechable y fácil de configurar.
* 🔌 **Soporte para funcionalidades clave de Kubernetes**:

  * Ingress controllers
  * Servicios `LoadBalancer` (mediante túneles)
  * Volúmenes persistentes
  * Add-ons como `dashboard`, `metrics-server`, `ingress`, etc.
* 🛠️ **Montaje de archivos locales en contenedores**: útil para desarrollo iterativo.
* 🔁 **Reinicios rápidos del clúster**: puedes detener e iniciar el clúster sin perder estado.

---

## 🛠️ ¿Cómo funciona Minikube?

Minikube inicia una máquina virtual o contenedor que actúa como un nodo de Kubernetes (control plane + worker) usando un **driver**. Este puede ser `docker`, `virtualbox`, `hyperkit`, entre otros.

```bash
minikube start --driver=docker
```

Este comando levanta un clúster local usando Docker como backend. Una vez iniciado, puedes aplicar manifests de Kubernetes como si estuvieras en un entorno productivo.

---

## 🧰 Valores comunes para `--driver`

El flag `--driver` define el backend que Minikube usará para crear el nodo de Kubernetes. Aquí una tabla con los valores más usados:

| Driver       | Descripción                                                          | Plataformas      | Recomendado si...                                         |
| ------------ | -------------------------------------------------------------------- | ---------------- | --------------------------------------------------------- |
| `docker`     | Usa contenedores Docker como entorno del clúster                     | Win/macOS/Linux  | Ya tienes Docker instalado y quieres una opción liviana   |
| `virtualbox` | Utiliza VirtualBox como hipervisor para correr una VM con Kubernetes | Win/macOS/Linux  | No usas Docker o prefieres aislamiento completo           |
| `hyperkit`   | Driver nativo para macOS basado en Hypervisor.framework              | macOS            | Estás en macOS y buscas mejor rendimiento que VirtualBox  |
| `hyperv`     | Usa Hyper-V para ejecutar una VM con Kubernetes                      | Windows          | Tienes Hyper-V habilitado y prefieres usarlo sobre Docker |
| `qemu`       | Usa QEMU para entornos más personalizados o en Linux                 | Linux (avanzado) | Necesitas emulación más configurable o portable           |
| `vmware`     | Usa VMware Fusion o Workstation como hipervisor                      | Win/macOS        | Ya trabajas con VMware y tienes licencias                 |
| `none`       | Ejecuta directamente en el host (sin virtualización)                 | Linux            | Tienes Linux y quieres evitar overhead de VMs o Docker    |
| `podman`     | Similar a Docker, pero usa Podman como backend                       | Linux/macOS      | Trabajas con Podman y deseas rootless containers          |
| `wsl2`       | Usa WSL2 como entorno para el clúster Kubernetes                     | Windows 10/11    | Trabajas con Windows moderno y tienes WSL2 disponible     |

> ⚠️ **Nota**: No todos los drivers están disponibles en todas las plataformas. Algunos requieren configuraciones previas (por ejemplo, habilitar Hyper-V o instalar VirtualBox).

Puedes ver los drivers disponibles en tu sistema con:

```bash
minikube start --help
```
> Busca una línea similar a: `--driver string VM driver is one of: [docker, kvm2, virtualbox, ...]`. La lista exacta variará según tu sistema operativo y las dependencias instaladas.

---

## 📦 Comandos básicos

| Comando                     | Descripción                                        |
| --------------------------- | -------------------------------------------------- |
| `minikube start`            | Inicia el clúster local                            |
| `minikube stop`             | Detiene el clúster sin eliminarlo                  |
| `minikube delete`           | Elimina el clúster y su configuración              |
| `minikube dashboard`        | Abre la UI gráfica de Kubernetes en el navegador   |
| `minikube service <nombre>` | Abre un servicio tipo `NodePort` o `LoadBalancer`  |
| `minikube ssh`              | Accede a la VM o contenedor donde corre Kubernetes |
| `minikube addons list`      | Muestra add-ons disponibles                        |
| `minikube status`           | Verifica el estado actual del clúster              |

---

## 🔍 Comando `minikube status`

Este comando permite comprobar si el clúster de Minikube está operativo:

```bash
minikube status
```

### 🔎 ¿Qué muestra?

| Componente     | Descripción                                                   |
| -------------- | ------------------------------------------------------------- |
| **host**       | Máquina virtual o contenedor del clúster                      |
| **kubelet**    | Servicio que ejecuta y gestiona los pods en el nodo           |
| **apiserver**  | Servidor API de Kubernetes que coordina todos los componentes |
| **kubeconfig** | Indica si el archivo `~/.kube/config` apunta a este clúster   |

#### 📘 Ejemplo:

```bash
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

---

## 🎯 ¿Cuándo usar Minikube?

Minikube es ideal en los siguientes escenarios:

* 📚 **Aprendizaje y entrenamiento**: para familiarizarse con Kubernetes sin gastar recursos en la nube.
* 🧪 **Testing y validación local**: para probar nuevos deployments o configuraciones.
* 💻 **Desarrollo local**: para simular cómo se comportaría una app en producción.

---

## ⚡ Recomendaciones adicionales

* Usa `--driver=docker` si ya tienes Docker instalado para una integración más liviana.
* Configura CPU, memoria y disco con flags como `--cpus`, `--memory`, `--disk-size` si tu entorno lo requiere.
* Puedes crear perfiles múltiples con `--profile` para simular distintos entornos.

---
