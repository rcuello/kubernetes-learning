# 📦 Deployments en Kubernetes: Orquestando tus Aplicaciones

> Descubre los Deployments, el recurso central para gestionar el ciclo de vida de tus aplicaciones en Kubernetes. Aprende cómo automatizan actualizaciones, escalado y garantizan la disponibilidad de tus Pods de forma declarativa y segura.

-----

## 🧠 ¿Qué es un Deployment y para qué sirve?

Un **Deployment** es un recurso de nivel superior en Kubernetes que te permite describir el **estado deseado** de tu aplicación. Su propósito principal es gestionar el despliegue, la actualización y el escalado de un conjunto de Pods, asegurando que tu aplicación se ejecute de manera confiable y con el mínimo tiempo de inactividad.

En esencia, un Deployment:

  * **Orquesta ReplicaSets:** Crea y administra automáticamente uno o varios **ReplicaSets**. Aunque los ReplicaSets se encargan de mantener el número deseado de Pods, los Deployments son los que controlan los ReplicaSets para habilitar funcionalidades avanzadas.
  * **Mantiene Réplicas Deseadas:** Garantiza que siempre haya el número especificado de Pods ejecutándose, igual que un ReplicaSet.
  * **Actualizaciones sin Interrupción (Rolling Updates):** Permite actualizar tu aplicación a una nueva versión de forma gradual, reemplazando Pods antiguos por nuevos sin que el servicio deje de estar disponible.
  * **Rollbacks Sencillos:** Si una actualización sale mal, puedes revertir fácilmente a una versión anterior y estable de tu aplicación con un solo comando.
  * **Autorecuperación:** Si un Pod falla o se elimina, el Deployment (a través de su ReplicaSet subyacente) asegura que se inicie un nuevo Pod para mantener el número de réplicas.

> 🚀 **Analogía Práctica**: Piensa en un Deployment como el director de orquesta de tu aplicación. No solo se asegura de que haya suficientes músicos (Pods) tocando, sino que también gestiona las "audiciones" para nuevos músicos (actualizaciones), permite que los músicos antiguos se retiren sin parar el concierto (rolling updates), y puede traer de vuelta a los músicos originales si la nueva interpretación no funciona (rollbacks).

-----

## 🏗️ Estructura de un Archivo `deployment.yaml`

Un manifiesto de Deployment es donde declaras cómo quieres que se vea tu aplicación.

```yaml
apiVersion: apps/v1     # API Group y versión para Deployments
kind: Deployment        # Tipo de recurso: Deployment
metadata:
  name: mi-aplicacion-deployment # Nombre único para tu Deployment
  labels:
    app: mi-aplicacion # Etiquetas para identificar el Deployment
spec:
  replicas: 3           # Número deseado de Pods (réplicas) que el Deployment debe mantener
  selector:             # Selector: Crucial para que el Deployment sepa qué Pods gestionar
    matchLabels:        # Los Pods con estas etiquetas serán controlados por este Deployment
      app: mi-aplicacion
  template:             # Plantilla de Pod: Define cómo serán los Pods que creará el Deployment
    metadata:
      labels:           # ¡IMPORTANTE! Las etiquetas de la plantilla deben COINCIDIR con el selector
        app: mi-aplicacion
    spec:
      containers:
        - name: contenedor-principal # Nombre del contenedor dentro del Pod
          image: mi-imagen:1.0      # Imagen Docker a utilizar para el contenedor
          ports:
            - containerPort: 8080   # Puerto que la aplicación escucha dentro del contenedor
          resources:              # (Opcional) Límites de recursos para el contenedor
            limits:
              memory: "128Mi"
              cpu: "500m"
            requests:
              memory: "64Mi"
              cpu: "250m"
```

### Explicación de Campos Clave:

  * **`replicas`**: Indica cuántas instancias de tu aplicación (`Pods`) deseas que se estén ejecutando en todo momento.
  * **`selector`**: Es la parte más crítica para la conexión. Le dice al Deployment qué Pods "pertenecen" a este Deployment. Utiliza `matchLabels` para especificar qué etiquetas deben tener los Pods para ser gestionados.
  * **`template`**: Es una plantilla completa de un Pod. Cuando el Deployment necesita crear nuevos Pods (por ejemplo, para escalar o reemplazar Pods durante una actualización), utiliza esta plantilla como "plano".
      * **¡Atención\!** Las `metadata.labels` dentro de la `template` del Pod **DEBEN COINCIDIR** con el `selector.matchLabels` del Deployment. Si no coinciden, el Deployment no podrá encontrar ni gestionar sus propios Pods.

-----

## 🔁 La Relación entre Deployment y ReplicaSet

Cuando creas un **Deployment**, en realidad, Kubernetes hace el "trabajo sucio" por ti:

1.  El Deployment crea un **ReplicaSet**.
2.  Este ReplicaSet es el que se encarga de crear y mantener el número de Pods especificado en el `replicas` del Deployment.
3.  Durante una actualización, el Deployment creará un **nuevo ReplicaSet** con la nueva versión de los Pods y, gradualmente, escalará el nuevo ReplicaSet mientras escala el antiguo a cero. Esto permite las *rolling updates*.

> ✅ **En resumen:** Tú interactúas con el Deployment porque te ofrece funcionalidades de alto nivel (actualizaciones, rollbacks). El Deployment, a su vez, utiliza uno o más ReplicaSets para manejar la creación y el mantenimiento de tus Pods. ¡No necesitas crear los ReplicaSets manualmente\!

-----

## ⚙️ Comandos Útiles para Deployments

Aquí tienes los comandos esenciales para gestionar tus Deployments:

1.  **Crear un Deployment desde un archivo YAML:**

    ```bash
    kubectl apply -f mi-aplicacion-deployment.yaml
    ```

2.  **Listar todos los Deployments en el namespace actual:**

    ```bash
    kubectl get deployments
    # Para ver también los ReplicaSets asociados:
    kubectl get deployments,replicasets
    ```

3.  **Ver detalles de un Deployment específico:**

    ```bash
    kubectl describe deployment mi-aplicacion-deployment
    ```

    (Esto te mostrará el estado actual, el ReplicaSet que está gestionando, los eventos, etc.).

4.  **Escalar un Deployment (cambiar el número de réplicas):**

    ```bash
    kubectl scale deployment mi-aplicacion-deployment --replicas=5
    ```

    (Esto ajustará el número de Pods a 5).

5.  **Actualizar la imagen de un contenedor en un Deployment (Rolling Update):**

    ```bash
    kubectl set image deployment/mi-aplicacion-deployment contenedor-principal=mi-imagen:2.0
    ```

      * **Ejemplo Práctico:** Si tu Deployment se llama `hello-deployment` y el contenedor se llama `hello-app`, y quieres actualizarlo a la versión `2.0`:
        ```bash
        kubectl set image deployment/hello-deployment hello-app=gcr.io/google-samples/hello-app:2.0
        ```
      * Puedes ver cómo el Deployment orquesta un nuevo ReplicaSet y escala gradualmente:
        ```bash
        kubectl get deployments -w # -w para watch, ver los cambios en tiempo real
        kubectl get replicasets -w
        kubectl get pods -w
        ```

6.  **Ver el historial de revisiones del Deployment:**

    ```bash
    kubectl rollout history deployment/mi-aplicacion-deployment
    ```

7.  **Deshacer la última actualización (Rollback):**

    ```bash
    kubectl rollout undo deployment/mi-aplicacion-deployment
    ```

    (También puedes especificar una revisión específica para revertir).

8.  **Eliminar un Deployment (y todos los ReplicaSets y Pods asociados):**

    ```bash
    kubectl delete deployment mi-aplicacion-deployment
    ```

-----

## 🚀 Crear un Deployment y Exponerlo Rápidamente desde la CLI

Puedes crear Deployments básicos y exponerlos rápidamente directamente desde la línea de comandos, útil para pruebas rápidas:

1.  **Crear un Deployment simple:**

    ```bash
    kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
    ```

      * Este comando crea un Deployment llamado `web`.
      * Despliega un Pod con un contenedor usando la imagen `gcr.io/google-samples/hello-app:1.0`.
      * Por defecto, se crea con **una réplica**.
      * **No expone puertos automáticamente.**

2.  **Exponer el Deployment como un Service (para acceso interno o externo):**
    Para que tu aplicación sea accesible, necesitas un Service. Puedes crearlo rápidamente desde el Deployment:

      * **Como Service interno (ClusterIP):**

        ```bash
        kubectl expose deployment web --port=80 --target-port=8080 --type=ClusterIP
        ```

        Esto creará un Service de tipo `ClusterIP` que balanceará el tráfico del puerto `80` a los Pods del Deployment `web` en su puerto `8080`.

      * **Como Service externo (NodePort - para Minikube, etc.):**

        ```bash
        kubectl expose deployment web --port=80 --target-port=8080 --type=NodePort --name=web-service-np
        ```

        Esto creará un Service de tipo `NodePort` (llamado `web-service-np` para evitar conflictos si ya existe un `web` Service). Podrás acceder a él vía `http://<IP_del_Nodo>:<NodePort_asignado>`.

-----

## 📌 Buenas Prácticas con Deployments

  * **Siempre usa Deployments:** Son el recurso estándar para gestionar aplicaciones sin estado y son la base para la mayoría de las cargas de trabajo en Kubernetes.
  * **Define Recursos (CPU/Memoria):** Incluye `resources.limits` y `resources.requests` en tus Pod templates para asegurar un rendimiento predecible y evitar el agotamiento de recursos en los Nodos.
  * **Estrategia de Actualización:** Familiarízate con `spec.strategy.rollingUpdate` para controlar cómo se realizan las actualizaciones. Puedes ajustar `maxSurge` (pods adicionales permitidos) y `maxUnavailable` (pods no disponibles permitidos) para equilibrar la velocidad y la disponibilidad.
  * **Health Checks:** Incluye `livenessProbe` y `readinessProbe` en tus Pods para que Kubernetes sepa cuándo un Pod está listo para recibir tráfico y cuándo debe reiniciarse.
  * **Etiquetas y Anotaciones:** Usa etiquetas consistentes para organizar y seleccionar tus Pods. Utiliza anotaciones para metadatos adicionales de herramientas o información de contacto.

-----