# 📦 Lab: Atributos de Manifiesto (YAML): Labels y Selectors - Organización y Gestión de Recursos


Este laboratorio te guía paso a paso en la comprensión de la estructura de un manifiesto YAML en Kubernetes. Aprenderás a usar `labels` y `selectors` para organizar y gestionar tus recursos de manera eficiente, lo cual es un conocimiento crucial para escalar y mantener aplicaciones en un entorno de producción.

> **Pre-requisitos:** Asegúrate de que Minikube esté iniciado (`minikube start`) y que tu terminal esté configurada para usar el clúster (`kubectl config use-context minikube`).

---

## 🎯 1. El problema: Falta de Organización y de identidad

Imagina que tienes una aplicación web simple que necesita dos pods. Sin etiquetas, ¿cómo los agruparías para gestionarlos juntos? Es una tarea ineficiente.

A continuación, crearemos un pod sin ninguna etiqueta, lo que dificultará su identificación y gestión en un entorno más grande.

### 📄 Manifiesto problemático (`pod-sin-label.yaml`)

```yaml
# ❌ Este manifiesto crea un pod, pero no incluye etiquetas (labels)
# ❌ Esto lo hace difícil de identificar y gestionar en grupo.
apiVersion: v1
kind: Pod
metadata:
  name: webserver-pod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```
---

### 🧪 1.1. Despliegue y observación del problema

```bash
# Aplica el manifiesto problemático
kubectl apply -f pod-sin-label.yaml

# Obtiene la información del pod. Nota la ausencia de la columna LABELS.
kubectl get pods --show-labels
```

**Salida esperada:**

```
NAME            READY   STATUS    RESTARTS   AGE   LABELS
webserver-pod   1/1     Running   0          5s    <none>
```

> ⚠️ **Problemas observados:**
> Como puedes ver, no hay información para identificar de qué "tipo" de aplicación es este pod. Si tuvieras 100 pods, sería imposible gestionarlos de manera eficiente por sus funciones.

---

## ✅ 2. La solución: Etiquetas y Selectores

La solución es simple: **etiquetar los recursos**. Un `label` es un par clave-valor que puedes asignar a cualquier recurso de Kubernetes. Un `selector` es la herramienta que usa otros recursos (como los **Services**) para encontrar y agrupar recursos con etiquetas específicas.

Vamos a reescribir nuestro manifiesto para incluir etiquetas.

### 📄 Archivo `pod-con-labels.yaml`

```yaml
# ✅ Este manifiesto crea un pod con etiquetas para identificarlo.
apiVersion: v1
kind: Pod
metadata:
  name: webserver-pod-labeled
  labels: # 🌟 Aquí definimos los labels.
    app: webserver # 🌟 Etiqueta "app" con valor "webserver"
    tier: frontend # 🌟 Etiqueta "tier" con valor "frontend"
    version: v1    # 🌟 Etiqueta "version" con valor "v1"
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```
---

## 🚀 3. Despliegue del Pod con Etiquetas
```bash
# Crea el pod etiquetado
kubectl apply -f pod-con-labels.yaml
```

---
## 👀 4. Observa las características clave del Pod

```bash
# Obtiene todos los pods y sus labels. Nota la columna LABELS ahora.
kubectl get pods --show-labels
```

**Salida esperada:**

```
NAME                      READY   STATUS    RESTARTS   AGE     LABELS
webserver-pod             1/1     Running   0          2m      <none>
webserver-pod-labeled     1/1     Running   0          5s      app=webserver,tier=frontend,version=v1
```

Ahora, con las etiquetas, puedes filtrar y gestionar los pods de manera mucho más efectiva.
---

## 🛸 5. Usando Selectores para filtrar recursos

El verdadero poder de los labels viene cuando los usas con selectores.

```bash
# Obtiene los pods con la etiqueta 'app=webserver'
kubectl get pods -l app=webserver

# Obtiene los pods con la etiqueta 'tier=frontend'
kubectl get pods -l tier=frontend

# Combina selectores: obtén pods con 'app=webserver' Y 'tier=frontend'
kubectl get pods -l 'app=webserver,tier=frontend'

# Obtiene pods con la etiqueta 'version' que no sea 'v2'
kubectl get pods -l '!version=v2' # 🌟 El signo '!' indica "no tiene este valor"
```

**Salida esperada para el último comando (todos los pods, ya que ninguno tiene v2):**

```
NAME                      READY   STATUS    RESTARTS   AGE     LABELS
webserver-pod             1/1     Running   0          4m      <none>
webserver-pod-labeled     1/1     Running   0          2m      app=webserver,tier=frontend,version=v1
```


## 🛠️ 6. Casos Prácticos

Los labels son la columna vertebral de la organización en Kubernetes. Aquí hay algunos ejemplos de su uso en escenarios del mundo real.


### 6.1. Gestión de entornos de desarrollo/producción

```yaml
# Creamos un pod para desarrollo
apiVersion: v1
kind: Pod
metadata:
  name: dev-pod
  labels:
    env: dev
spec:
  containers:
  - name: nginx
    image: nginx
---
# Creamos un pod para producción
apiVersion: v1
kind: Pod
metadata:
  name: prod-pod
  labels:
    env: prod
spec:
  containers:
  - name: nginx
    image: nginx
```

Ahora puedes gestionar tus pods por entorno.

```bash
# Aplica los manifiestos
kubectl apply -f dev-prod-pods.yaml

# Obtiene solo los pods de desarrollo
kubectl get pods -l env=dev

# Obtiene solo los pods de producción
kubectl get pods -l env=prod
```


### 6.2. Identificación de componentes de una aplicación compleja

```yaml
# Un pod para la base de datos
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
  labels:
    app: myapp
    component: database
spec:
  containers:
  - name: mysql
    image: mysql
---
# Un pod para la API backend
apiVersion: v1
kind: Pod
metadata:
  name: api-pod
  labels:
    app: myapp
    component: api
spec:
  containers:
  - name: backend
    image: my-backend-image
```

Aquí usamos `app: myapp` para agrupar todo lo que pertenece a la aplicación, y `component` para diferenciar las partes.

### 6.3. Control de versiones de despliegues

Imagina que tienes una nueva versión de tu aplicación (v2) y quieres probarla junto con la v1.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver-v2
  labels:
    app: webserver
    version: v2 # 🌟 Etiqueta de versión actualizada
spec:
  containers:
  - name: nginx
    image: nginx:1.25.3 # Imagen diferente
```

```bash
# Crea el pod de la versión 2
kubectl apply -f pod-v2.yaml

# Obtén todos los pods de la aplicación web
kubectl get pods -l app=webserver
```

**Salida esperada:**

```
NAME                      READY   STATUS    RESTARTS   AGE     LABELS
webserver-pod-labeled     1/1     Running   0          25m     app=webserver,tier=frontend,version=v1
webserver-v2              1/1     Running   0          1m      app=webserver,version=v2
```

### 6.4.Visualización y filtrado con `kubectl`

```bash
kubectl get pods -l app=nginx
kubectl get pods -l 'tier notin (frontend)'
kubectl get pods --selector='app=nginx,tier=frontend'
```

> 🎯 Aprende a filtrar tus recursos con `-l` y `--selector`
-----

## 7. 🧹 Limpieza 

Es una buena práctica limpiar los recursos después de un laboratorio.

```bash
kubectl delete -f pods-sin-labels.yaml
kubectl delete -f pods-con-labels.yaml
kubectl delete -f service-con-selector.yaml
kubectl delete -f deployment.yaml
```

**Tip:** También puedes eliminar pods por selectores, ¡pero ten cuidado\!

```bash
# Esto eliminaría todos los pods con la etiqueta 'app=webserver'
# kubectl delete pods -l app=webserver
```

---


## ✅ ¿Qué aprendiste?

* Aprendiste a usar **labels** (etiquetas) para clasificar y organizar recursos de Kubernetes.
* Dominaste el uso de **selectors** para filtrar y operar sobre grupos de recursos basados en sus etiquetas.
* Viste cómo los **labels** y **selectors** son fundamentales para el funcionamiento de recursos clave como **Services** y **Deployments**.
* Comprendiste la diferencia entre **labels** (para selección) y **annotations** (para metadatos informativos).

> 🎯 **Regla de oro:** Siempre etiqueta tus recursos. Las etiquetas son la forma en que Kubernetes entiende y gestiona las relaciones entre sus componentes.

