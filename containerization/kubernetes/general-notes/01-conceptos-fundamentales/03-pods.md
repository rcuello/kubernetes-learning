# 🧱 Pods: La Unidad Fundamental de Ejecución en Kubernetes

> Los Pods son la unidad más pequeña y básica que puedes desplegar en Kubernetes. Son la abstracción que encapsula uno o más contenedores, junto con recursos compartidos y opciones de configuración.

-----

## 🧠 ¿Qué es un Pod y para qué sirve?

Un **Pod** representa una sola instancia de una aplicación o de una parte de una aplicación. Piensa en él como un "envoltorio" alrededor de tus contenedores. Aunque un Pod puede contener múltiples contenedores, lo más común es que encapsule **un solo contenedor principal**, a menudo acompañado de contenedores "sidecar" que le brindan servicios auxiliares.

Los contenedores dentro del mismo Pod **comparten recursos vitales**:

  * **El mismo espacio de red:** Comparten la misma dirección IP (interna al clúster) y puertos de red. Esto significa que pueden comunicarse entre sí usando `localhost`.
  * **El mismo almacenamiento:** Pueden compartir volúmenes de almacenamiento, permitiendo que los datos se persistan o se compartan entre ellos.
  * **El mismo ciclo de vida:** Son co-ubicados y co-programados en el mismo nodo. Si el Pod muere, todos los contenedores dentro de él mueren juntos y se recrea el Pod completo.

> 🚀 **Analogía Práctica**: Un Pod es como una casa móvil que se estaciona en un terreno (nodo). Dentro de esa casa móvil, tienes diferentes habitaciones (contenedores) que comparten la misma dirección, la misma conexión a servicios (red) y pueden tener acceso a los mismos armarios (volúmenes de almacenamiento). Si la casa se mueve o se destruye, todas las habitaciones se ven afectadas al mismo tiempo.

-----

## 🔑 Características Clave de los Pods

| Característica        | Descripción                                                                                                                                                                                                                                                        |
| :-------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Unidad Mínima** | Es la unidad más pequeña de despliegue en Kubernetes. No puedes desplegar contenedores directamente; siempre lo haces dentro de un Pod.                                                                                                                                     |
| **IP Única** | Cada Pod recibe una dirección IP única y efímera dentro de la red del clúster. Otros Pods y Services pueden comunicarse con él a través de esta IP o de su nombre DNS (si está asociado a un Service).                                                                        |
| **Ciclo de Vida** | Los Pods son efímeros por naturaleza. Si un Pod muere (por fallo de la aplicación, recursos insuficientes, etc.) o se elimina, desaparece y un nuevo Pod con una nueva IP es creado por un controlador (como un Deployment o ReplicaSet) para reemplazarlo. |
| **Recursos Compartidos** | Los contenedores dentro del mismo Pod comparten el mismo `localhost`, el mismo espacio de puertos, y pueden acceder a los mismos volúmenes de almacenamiento, facilitando la comunicación y el intercambio de datos entre ellos.                                 |
| **Co-ubicación** | Todos los contenedores de un Pod se programan juntos en el mismo Nodo, lo que garantiza que estén siempre físicamente cerca y minimiza la latencia en su comunicación.                                                                                                  |

-----

## 📄 Ejemplo YAML de un Pod

Aunque no es común crear Pods directamente en producción (ver buenas prácticas), es fundamental entender su estructura.

```yaml
apiVersion: v1     # Versión de la API de Kubernetes
kind: Pod          # Tipo de recurso: Pod
metadata:
  name: mi-primer-pod # Nombre único para el Pod
  labels:          # Etiquetas para organizar y seleccionar el Pod
    app: demo-app
    environment: dev
spec:
  containers:      # Lista de contenedores que se ejecutarán dentro de este Pod
    - name: nginx-webserver # Nombre del contenedor
      image: nginx:1.25     # Imagen Docker a utilizar
      ports:
        - containerPort: 80 # Puerto que el contenedor expone
      resources:          # (Opcional) Límites y solicitudes de recursos para el contenedor
        limits:
          memory: "128Mi"
          cpu: "500m"
        requests:
          memory: "64Mi"
          cpu: "250m"
    # Puedes añadir más contenedores aquí si son "sidecars"
    # - name: sidecar-logger
    #   image: fluentd:latest
    #   volumeMounts:
    #     - name: log-volume
    #       mountPath: /var/log/app
  # volumes: (Opcional) Volúmenes compartidos por los contenedores del Pod
  #   - name: log-volume
  #     emptyDir: {}
```

-----

## 🛠️ Comandos Básicos con Pods

Aquí tienes los comandos esenciales para interactuar directamente con los Pods:

1.  **Crear un Pod desde un archivo YAML (Declarativo):**

    ```bash
    kubectl apply -f pod.yaml
    ```

2.  **Crear un Pod de forma imperativa (Solo para pruebas rápidas/desarrollo):**
    Este método crea un Pod directamente desde la línea de comandos. No se recomienda para producción, ya que no permite una gestión declarativa y carece de funcionalidades como la autorecuperación.

    ```bash
    kubectl run nginx-test-pod --image=nginx --restart=Never --port=80
    
    # Para especificar un namespace:
    kubectl run nginx-test-pod --image=nginx --restart=Never --port=80 --namespace=desarrollo
    ```

3.  **Listar Pods en el namespace actual:**

    ```bash
    kubectl get pods
    ```

4.  **Listar Pods en un namespace específico:**

    ```bash
    kubectl get pods -n <nombre-del-namespace>
    # Ejemplo:
    kubectl get pods -n desarrollo
    ```

5.  **Listar Pods en todos los namespaces:**

    ```bash
    kubectl get pods --all-namespaces
    ```

6.  **Ver detalles completos de un Pod:**

    ```bash
    kubectl describe pod <nombre-del-pod>
    # Ejemplo:
    kubectl describe pod mi-primer-pod
    ```

    (Esto proporciona información vital como el estado actual, eventos, imágenes usadas, puertos, volúmenes, asignación de nodo, etc.)

7.  **Eliminar un Pod:**

    ```bash
    kubectl delete pod <nombre-del-pod>
    # Para eliminar en un namespace específico:
    kubectl delete pod <nombre-del-pod> --namespace=<nombre-del-namespace>
    ```

-----

## 🧪 Inspección y Depuración de Pods

Cuando un Pod no funciona como esperas, estos comandos son tus mejores amigos:

1.  **Ver los logs de un contenedor dentro de un Pod:**

    ```bash
    kubectl logs <nombre-del-pod>
    # Si hay múltiples contenedores en el Pod, especifica el nombre del contenedor:
    kubectl logs <nombre-del-pod> -c <nombre-del-contenedor>
    # Para ver los logs en tiempo real (follow):
    kubectl logs -f <nombre-del-pod>
    ```

2.  **Acceder al shell de un contenedor dentro de un Pod:**
    Esto te permite ejecutar comandos directamente dentro del contenedor, como si hubieras hecho un `docker exec`.

    ```bash
    kubectl exec -it <nombre-del-pod> -- /bin/bash
    # Si el contenedor usa sh en lugar de bash:
    kubectl exec -it <nombre-del-pod> -- /bin/sh
    # Si hay múltiples contenedores en el Pod, especifica el nombre del contenedor:
    kubectl exec -it <nombre-del-pod> -c <nombre-del-contenedor> -- /bin/bash
    ```

3.  **Ver Pods con su IP y el Nodo donde se ejecutan:**

    ```bash
    kubectl get pods -o wide
    # En un namespace específico:
    kubectl get pods -o wide --namespace=desarrollo
    ```

4.  **Descargar imagen manualmente en Minikube (para entornos offline/aislados):**
    Si estás usando Minikube y tienes problemas de conectividad o restricciones para pull de imágenes, puedes precargar una imagen en el daemon Docker de Minikube. Esto la hace disponible para tus Pods sin necesidad de pull desde internet en tiempo de ejecución.

    ```bash
    docker pull nginx:latest      # Primero, descarga la imagen a tu daemon Docker local
    minikube image load nginx:latest # Luego, carga esa imagen al daemon Docker de Minikube
    ```

    Ahora, puedes usar esta imagen en tus Pods:

    ```bash
    kubectl run my-offline-nginx --image=nginx:latest --restart=Never
    ```

-----

## 📚 Pods y su Relación con Otros Recursos

Los Pods rara vez se gestionan de forma aislada en un entorno de producción. Se utilizan en conjunto con **Controladores** de Kubernetes que les añaden robustez y funcionalidades de gestión:

| Recurso           | Descripción                                                                 |
| :---------------- | :-------------------------------------------------------------------------- |
| **Pod** | La unidad atómica que encapsula uno o más contenedores, red y almacenamiento. Es efímero y por sí solo no garantiza disponibilidad. |
| **ReplicaSet** | Un controlador de bajo nivel que asegura que un número deseado de Pods esté siempre ejecutándose. Gestiona la estabilidad y el escalado simple de Pods. |
| **Deployment** | Un controlador de alto nivel que gestiona la vida de los Pods a través de ReplicaSets. Permite actualizaciones declarativas (rolling updates) y reversiones (rollbacks) de tu aplicación. **Es el recurso preferido para cargas de trabajo sin estado.** |
| **StatefulSet** | Diseñado para cargas de trabajo con estado (ej. bases de datos). Asegura un orden de despliegue y eliminación predecible, y asigna identidades de red y almacenamiento persistentes a sus Pods. |
| **DaemonSet** | Garantiza que una copia de un Pod se ejecute en **todos (o un subconjunto específico)** de los Nodos del clúster. Ideal para agentes de monitoreo, log collectors, etc. |
| **Job / CronJob** | Ejecuta Pods que realizan una tarea hasta su finalización exitosa. Los Jobs ejecutan la tarea una vez; los CronJobs la ejecutan en un horario recurrente. |

-----

## 📌 Buenas Prácticas con Pods

  * **No gestiones Pods directamente en Producción:** Siempre utiliza controladores de más alto nivel como **Deployments** (para la mayoría de las aplicaciones), **StatefulSets** (para bases de datos y apps con estado) o **DaemonSets** (para agentes por nodo). Esto asegura alta disponibilidad, escalabilidad y una gestión de ciclo de vida adecuada.
  * **Define `resources.limits` y `requests`:** Especifica los requisitos de CPU y memoria para tus contenedores. Esto ayuda al planificador de Kubernetes a colocar tus Pods eficientemente y previene que un Pod consuma todos los recursos de un Nodo.
  * **Implementa `livenessProbe` y `readinessProbe`:** Configura estos "health checks" para que Kubernetes sepa cuándo un contenedor debe ser reiniciado (liveness) o cuándo está listo para recibir tráfico (readiness), mejorando la fiabilidad del servicio.
  * **Usa `labels` y `annotations`:** Aplica etiquetas descriptivas a tus Pods para facilitar su organización y selección por otros recursos (como Services). Usa anotaciones para metadatos que no son esenciales para la operación, pero útiles para herramientas o información.
  * **Un proceso por contenedor (regla general):** Aunque un Pod puede tener múltiples contenedores, lo ideal es que cada contenedor ejecute un solo proceso principal. Los contenedores adicionales (sidecars) deben ser procesos auxiliares que complementan al principal.

-----