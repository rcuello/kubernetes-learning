# 🧰 kubectl: La herramienta CLI para Kubernetes

## 📌 ¿Qué es `kubectl`?

**`kubectl`** es la herramienta oficial de línea de comandos de Kubernetes. Permite interactuar con un clúster, ya sea local (como con `minikube`) o remoto (por ejemplo, en la nube), para consultar el estado de los recursos, desplegar aplicaciones, ver logs, escalar, actualizar y eliminar componentes.

---

## 🏭 Analogía: Kubernetes como una fábrica automatizada

Imagina que Kubernetes es una **fábrica automatizada de aplicaciones**. En esa fábrica:

- Cada **contenedor** es una **máquina** que realiza una tarea específica.
- Un **Pod** es como una **unidad de producción** que agrupa una o más máquinas que colaboran entre sí.
- Un **Deployment** es el **plan de producción**, que especifica cuántas unidades necesitas, cómo configurarlas y cómo reaccionar si fallan.
- Un **Service** es la **puerta de entrada o sistema de distribución**, que canaliza el tráfico hacia las unidades activas.
- Un **Ingress** es como una **recepción con políticas de acceso**, que decide quién entra, a qué parte y bajo qué condiciones.

Para operar esta fábrica, necesitas un **panel de control**. Ese panel es `kubectl`.

---

## 🧭 ¿Qué puedes hacer con `kubectl`?

Con `kubectl` puedes:

- Crear, actualizar o eliminar recursos como Pods, Deployments y Services.
- Consultar el estado del clúster y sus componentes.
- Escalar tus aplicaciones hacia arriba o abajo.
- Ver logs y eventos de ejecución.
- Ejecutar comandos dentro de los contenedores.

---

## ⚙️ Sintaxis básica de uso

```bash
kubectl get pods                         # Lista los pods activos
kubectl get services                     # Muestra los servicios expuestos
kubectl apply -f app.yaml                # Crea o actualiza recursos desde un manifiesto YAML
kubectl logs mi-pod                      # Muestra los logs de un pod específico
kubectl exec -it mi-pod -- bash          # Abre una terminal interactiva dentro del pod
````

---

## 📁 ¿Cómo sabe `kubectl` a qué clúster conectarse?

`kubectl` utiliza un archivo de configuración llamado `config`, que contiene información sobre:

* Qué clústeres están definidos.
* Qué usuarios pueden autenticarse.
* Qué contexto (combinación de clúster + usuario) está activo.

### 📍 Ubicación del archivo `config`

| Sistema Operativo          | Ruta por defecto             |
| -------------------------- | ---------------------------- |
| Linux / macOS              | `~/.kube/config`             |
| Windows (PowerShell / CMD) | `%USERPROFILE%\.kube\config` |
| Windows (WSL)              | `/home/usuario/.kube/config` |

> 🛠 Si usas **Minikube**, este archivo se crea automáticamente al ejecutar `minikube start`.

---

## 🧾 Ejemplo de contenido del archivo `config`

```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /home/usuario/.minikube/ca.crt
    server: https://192.168.49.2:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
users:
- name: minikube
  user:
    client-certificate: /home/usuario/.minikube/profiles/minikube/client.crt
    client-key: /home/usuario/.minikube/profiles/minikube/client.key
```

### 🔍 Descripción de campos

| Campo             | Descripción                                                            |
| ----------------- | ---------------------------------------------------------------------- |
| `clusters`        | Define los clústeres conocidos (endpoints, certificados, etc.).        |
| `contexts`        | Asocia un usuario y clúster, y puede incluir un namespace por defecto. |
| `current-context` | Define qué contexto está en uso actualmente por `kubectl`.             |
| `users`           | Define los usuarios y las credenciales que utilizarán.                 |

---

## 🔧 Comandos útiles de configuración

```bash
kubectl config view                    # Muestra el contenido del archivo config
kubectl config current-context         # Muestra el contexto activo
kubectl config get-clusters            # Lista los clústeres disponibles
kubectl config use-context <nombre>    # Cambia el contexto activo
```

> Aunque puedes editar el archivo `config` manualmente, se recomienda usar los comandos `kubectl config` o herramientas como `kubectx`.

---

## ✅ Buenas prácticas

* 🔐 **Nunca subas el archivo `config` a un repositorio**, ya que contiene credenciales o certificados sensibles.

* 🧩 Utiliza la variable de entorno `KUBECONFIG` si necesitas combinar múltiples archivos de configuración:

  ```bash
  export KUBECONFIG=~/.kube/config:~/.kube/dev.yaml:~/.kube/prod.yaml
  ```

* 🧭 Herramientas útiles:

  * [`kubectx`](https://github.com/ahmetb/kubectx) para cambiar de contexto fácilmente.
  * [`kubens`](https://github.com/ahmetb/kubectx) para cambiar de namespace.

---

## 🧩 Relación entre `kubectl` y `minikube`

| Herramienta | Función                                                  |
| ----------- | -------------------------------------------------------- |
| `minikube`  | Crea y gestiona clústeres de Kubernetes **locales**      |
| `kubectl`   | Se conecta al clúster (local o remoto) para **operarlo** |

> Al ejecutar `minikube start`, se configura automáticamente `kubectl` para apuntar al clúster recién creado.

---

## 🚀 Flujo típico de desarrollo local con `kubectl` y `minikube`

```bash
minikube start                                 # Inicia el clúster local
kubectl apply -f deployment.yaml               # Despliega una aplicación
kubectl get pods                               # Verifica que los pods estén corriendo
kubectl port-forward svc/mi-servicio 8080:80   # Redirige el servicio al puerto local
```

Este flujo es ideal para:

* Aprender Kubernetes desde tu entorno local.
* Simular entornos productivos antes de hacer despliegues reales.
* Probar manifiestos YAML y depurar configuraciones.

---

## 📚 Recursos útiles

* 📘 [Documentación oficial de kubectl](https://kubernetes.io/docs/reference/kubectl/)
* 📝 [Cheatsheet de comandos](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* 🗂️ [Guía del archivo kubeconfig](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
* 🔎 [kubectl explain](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#explain)

---

