## 🧱 Estructura General de un Archivo YAML de Kubernetes

Todo manifiesto de Kubernetes sigue una estructura básica y declarativa. Es como un "plano" de lo que quieres que exista en tu clúster.

```yaml
apiVersion: <versión-de-la-API> # Define el grupo de API y la versión (ej. v1, apps/v1)
kind: <tipo-de-recurso>       # El tipo de objeto de Kubernetes que estás creando (ej. Pod, Service, Deployment)
metadata:                     # Metadatos que ayudan a identificar y organizar el recurso
  name: <nombre-del-recurso>  # Nombre único para este recurso dentro de su Namespace
  namespace: <nombre-namespace> # (Opcional) Namespace donde se creará el recurso (por defecto: 'default')
  labels:                     # Etiquetas: pares clave-valor para organizar y seleccionar (ej. app: mi-app)
    <clave>: <valor>
  annotations:                # Anotaciones: pares clave-valor para metadatos no-identificadores (ej. información de CI/CD)
    <clave>: <valor>
spec:                         # Especificación: La definición del estado deseado del recurso
  # Atributos específicos del tipo de recurso (ej. replicas, selector, containers)
  ...
```

-----

## 🔁 ReplicaSet: Asegurando la Cantidad Deseada de Pods

Un **ReplicaSet** es un controlador que garantiza que un número específico de Pods idénticos (`replicas`) esté siempre ejecutándose. Si un Pod falla, el ReplicaSet lo reemplaza; si hay demasiados, los elimina.

### 📄 Ejemplo Básico de ReplicaSet

```yaml
apiVersion: apps/v1        # API Group para ReplicaSets
kind: ReplicaSet
metadata:
  name: nginx-replicaset  # Nombre de tu ReplicaSet
  labels:
    app: nginx-frontend   # Etiquetas para este ReplicaSet
spec:
  replicas: 3             # Queremos 3 Pods de Nginx
  selector:               # CRÍTICO: Cómo el ReplicaSet encuentra sus Pods
    matchLabels:
      app: nginx-frontend # ¡Las etiquetas de los Pods deben coincidir con esto!
  template:               # La plantilla que el ReplicaSet usa para crear nuevos Pods
    metadata:
      labels:             # Las etiquetas que se aplicarán a los Pods creados
        app: nginx-frontend # ¡DEBE COINCIDIR con spec.selector.matchLabels!
    spec:
      containers:         # Define los contenedores dentro de cada Pod
      - name: nginx-container # Nombre del contenedor
        image: nginx:latest   # Imagen Docker a usar
        ports:
        - containerPort: 80 # Puerto que el contenedor expone
```

### 🧩 Atributos Clave de ReplicaSet (`spec`)

| Atributo                      | Descripción                                                                                                                                                                                                                                                                          |
| :---------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`spec.replicas`** | **Obligatorio.** Un entero que especifica el **número deseado de Pods** que el ReplicaSet debe mantener activos en todo momento.                                                                                                                                                      |
| **`spec.selector.matchLabels`** | **Obligatorio.** Un mapa de etiquetas. El ReplicaSet usará estas etiquetas para **identificar qué Pods están bajo su control**. Es **fundamental** que estas etiquetas coincidan *exactamente* con las etiquetas definidas en `spec.template.metadata.labels`. Si no coinciden, el ReplicaSet no funcionará. |
| **`spec.template`** | **Obligatorio.** Esta sección contiene la **definición completa de un Pod**. El ReplicaSet utiliza esta plantilla para crear nuevas instancias de Pods. Incluye `metadata` (para etiquetas de Pod) y `spec` (para la configuración de contenedores, volúmenes, etc. del Pod). |
| `spec.template.metadata.labels` | **Obligatorio (dentro de `template`).** Las etiquetas que se aplicarán a cada Pod creado por este ReplicaSet. **DEBEN COINCIDIR** con el `spec.selector.matchLabels` del ReplicaSet.                                                                                             |
| `spec.template.spec.containers` | **Obligatorio (dentro de `template.spec`).** Una lista de definiciones de contenedores que se ejecutarán dentro de cada Pod. Cada contenedor tiene un `name`, una `image`, y opcionalmente `ports`, `resources`, `env`, etc.                                                     |

-----

## 🚀 Deployment: Gestionando el Despliegue de Aplicaciones

Un **Deployment** es un controlador de nivel superior que gestiona `ReplicaSets`. Es la forma **recomendada** de desplegar y actualizar aplicaciones sin estado en Kubernetes, ofreciendo funcionalidades como `rolling updates` y `rollbacks`.

### 📄 Ejemplo Básico de Deployment

```yaml
apiVersion: apps/v1      # API Group para Deployments
kind: Deployment
metadata:
  name: hello-deployment # Nombre de tu Deployment
  labels:
    app: hello-web
spec:
  replicas: 4            # Queremos 4 Pods para nuestra app 'hello-web'
  selector:              # Cómo el Deployment selecciona los Pods (y el ReplicaSet)
    matchLabels:
      app: hello-web     # ¡Las etiquetas de los Pods deben coincidir!
  template:              # La plantilla para los Pods
    metadata:
      labels:
        app: hello-web   # ¡DEBE COINCIDIR con spec.selector.matchLabels!
    spec:
      containers:
      - name: hello-app-container # Nombre del contenedor
        image: gcr.io/google-samples/hello-app:1.0 # Imagen Docker (versión 1.0)
        ports:
        - containerPort: 8080
  strategy:              # (Opcional) Define cómo se realizan las actualizaciones
    type: RollingUpdate  # Estrategia por defecto: RollingUpdate (actualización gradual)
    rollingUpdate:
      maxSurge: 1        # Número máximo de Pods que pueden crearse por encima de 'replicas'
      maxUnavailable: 1  # Número máximo de Pods que pueden estar no disponibles durante la actualización
  revisionHistoryLimit: 5 # (Opcional) Número de versiones de ReplicaSets a mantener para rollbacks
```

### 🧩 Atributos Clave de Deployment (`spec`)

Los Deployments comparten muchos atributos con los ReplicaSets, ya que los gestionan. Sin embargo, añaden otros importantes para el control del ciclo de vida:

| Atributo                      | Descripción                                                                                                                                                                                                                                                                                         |
| :---------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`spec.replicas`** | **Obligatorio.** Número deseado de Pods. El Deployment usará su ReplicaSet(s) subyacente(s) para mantener esta cantidad.                                                                                                                                                                         |
| **`spec.selector.matchLabels`** | **Obligatorio.** Define las etiquetas que el Deployment (y sus ReplicaSets) usarán para encontrar y gestionar los Pods. **DEBE COINCIDIR** con las etiquetas en `spec.template.metadata.labels`.                                                                                                  |
| **`spec.template`** | **Obligatorio.** La plantilla de Pods. Al igual que en ReplicaSet, define cómo se crearán los Pods. **Las etiquetas del Pod dentro de `template` deben coincidir con el `selector.matchLabels` del Deployment.** |
| **`spec.strategy`** | **Opcional.** Define la estrategia para reemplazar Pods existentes por nuevos durante una actualización. El valor por defecto es `RollingUpdate`. \<br/\> - `type: RollingUpdate`: Realiza una actualización gradual. \<br/\> - `type: Recreate`: Termina todos los Pods viejos antes de crear los nuevos (hay downtime). |
| `spec.strategy.rollingUpdate.maxSurge` | **Opcional (con `RollingUpdate`).** El número máximo o porcentaje de Pods que pueden crearse *por encima* del número de réplicas deseado durante una actualización. (Ej: `1` o `"25%"`)                                                                                                |
| `spec.strategy.rollingUpdate.maxUnavailable` | **Opcional (con `RollingUpdate`).** El número máximo o porcentaje de Pods que pueden estar *no disponibles* durante una actualización. (Ej: `1` o `"25%"`)                                                                                                                    |
| `spec.revisionHistoryLimit`     | **Opcional.** Un entero que especifica cuántas versiones antiguas de ReplicaSets debe conservar el Deployment. Esto permite realizar `rollbacks` a versiones anteriores. Por defecto es 10. Mantener demasiadas puede consumir recursos.                                                         |
| `spec.minReadySeconds`          | **Opcional.** Un entero. El número mínimo de segundos que un Pod recién creado debe estar listo (healthy) sin que ninguno de sus contenedores se reinicie, para ser considerado disponible. Por defecto es 0.                                                                                  |

-----

## 🧠 Recomendaciones para un Aprendizaje Profundo

  * **Usa `kubectl explain`:** Esta es tu mejor herramienta para entender cada campo en un manifiesto. Por ejemplo:
    ```bash
    kubectl explain deployment.spec
    kubectl explain deployment.spec.template.spec.containers.ports
    kubectl explain replicaset.spec.selector
    ```
    Te mostrará la descripción oficial de cada campo y si es obligatorio o no.
  * **Inspecciona Recursos Existentes:** Crea un Deployment o ReplicaSet y luego míralo en YAML para entender cómo Kubernetes lo interpreta y añade valores por defecto:
    ```bash
    kubectl get deployment <nombre-de-tu-deployment> -o yaml
    ```
  * **Experimenta:** Modifica los valores de `replicas`, `image`, o las estrategias de `rollingUpdate` y observa cómo Kubernetes reacciona usando `kubectl get pods -w` y `kubectl describe deployment`.

Dominar estos atributos te dará el control total sobre el despliegue y la gestión de tus aplicaciones en Kubernetes.

-----