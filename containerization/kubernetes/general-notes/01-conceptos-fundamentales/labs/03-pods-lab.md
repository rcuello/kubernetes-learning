# 📦 Lab: Pod - Tu primera unidad de despliegue en Kubernetes

Este laboratorio te guiará paso a paso para crear, gestionar y entender tu primer Pod. Aprenderás a definir Pods usando YAML y a usar los comandos esenciales de `kubectl` para interactuar con ellos, sentando las bases de tu viaje en Kubernetes.

> **Pre-requisitos:**
>
>   * Docker Desktop instalado y funcionando.
>   * Minikube instalado y configurado.
>   * Ejecuta `minikube start` para iniciar tu clúster.
>   * `kubectl` configurado para usar Minikube (generalmente se hace automáticamente).
>   * Una terminal como Git Bash o PowerShell en Windows 11.

-----

### 1. 🚫 El Problema: El Manifiesto Incompleto

A menudo, los principiantes cometen errores al definir sus primeros manifiestos. Un error común es olvidar que un contenedor dentro de un Pod necesita un nombre. Intentemos crear un Pod con un manifiesto incompleto para ver qué sucede.

Crea un archivo llamado `pod-error.yaml` con el siguiente contenido:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-de-prueba
spec:
  containers: # ❌ Falta el campo "name" dentro del contenedor
    - image: nginx
```

Ahora, intenta aplicarlo con `kubectl`:

```bash
kubectl apply -f pod-error.yaml
```

**Salida esperada:**

```
The Pod "pod-de-prueba" is invalid: spec.containers[0].name: Required value
```

**Análisis del problema:** Kubernetes valida el manifiesto YAML antes de intentar crear el recurso. El error nos dice claramente que falta el campo `name` para el contenedor. Todos los contenedores dentro de un Pod deben tener un nombre único para ser referenciados.

-----

### 2. ✅ La Solución: Creando tu Primer Pod

Ahora, corregiremos el manifiesto para crear un Pod funcional. Este Pod ejecutará una sola instancia del contenedor Nginx.

Crea un archivo llamado `nginx-pod.yaml` con el siguiente contenido:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-container # 🎯 Un nombre único para el contenedor
      image: nginx:1.14.2  # 🎯 La imagen de Docker a usar
      ports:
        - containerPort: 80 # 🎯 El puerto que expone el contenedor
```

**Pasos para el despliegue:**

1.  Aplica el manifiesto para crear el Pod:

    ```bash
    kubectl apply -f nginx-pod.yaml
    ```

    **Salida esperada:**

    ```
    pod/nginx-pod created
    ```

2.  Verifica que el Pod se esté ejecutando:

    ```bash
    kubectl get pods
    ```

    **Salida esperada:**

    ```
    NAME        READY   STATUS    RESTARTS   AGE
    nginx-pod   1/1     Running   0          ...
    ```

    > 💡 **Tip:** El estado `Running` y `READY 1/1` nos dice que el Pod está saludable y todos sus contenedores están listos.

3.  Obtén información detallada sobre el Pod:

    ```bash
    kubectl describe pod nginx-pod
    ```

    **Salida esperada:**
    (Verás una gran cantidad de información, incluyendo eventos, estado, contenedores y más. Esta es una herramienta crucial para la depuración).

4.  Visualiza los logs del contenedor dentro del Pod:

    ```bash
    kubectl logs nginx-pod
    ```

    **Salida esperada:**
    (Verás los logs de Nginx, confirmando que la aplicación está funcionando).

-----

### 3. 📊 Verificación y Casos Prácticos

#### Accediendo a tu Pod con `port-forward`

Los Pods tienen su propia IP interna, pero no son directamente accesibles desde fuera del clúster de Kubernetes. Usaremos `kubectl port-forward` para redirigir el tráfico de un puerto en tu máquina local al puerto del Pod.

1.  En una terminal **nueva** (sin cerrar la anterior), ejecuta el siguiente comando:
    ```bash
    kubectl port-forward nginx-pod 8080:80
    ```
    **Salida esperada:**
    ```
    Forwarding from 127.0.0.1:8080 -> 80
    Forwarding from [::1]:8080 -> 80
    ```
2.  Ahora, abre tu navegador web y navega a `http://localhost:8080`. ¡Deberías ver la página de bienvenida de Nginx\!

#### Caso de uso: Patrón Sidecar con múltiples contenedores

Una de las ventajas clave de un Pod es que puede contener múltiples contenedores que comparten recursos. Un ejemplo común es el patrón "Sidecar", donde un contenedor auxiliar se ejecuta junto al contenedor principal para realizar tareas como la gestión de logs, la sincronización de archivos, etc.

Crea un archivo llamado `sidecar-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  volumes: # 🎯 Se define un volumen que será compartido
  - name: shared-data
    emptyDir: {}
  containers:
  - name: main-container # 🎯 El contenedor principal (nginx)
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html # Monta el volumen en el directorio de servicio de Nginx

  - name: sidecar-container # 🎯 El contenedor "sidecar" (busybox)
    image: busybox
    command: ["sh", "-c", "echo '<h1>¡Hola desde el sidecar!</h1>' > /var/www/index.html && sleep 3600"]
    volumeMounts:
    - name: shared-data
      mountPath: /var/www # Monta el mismo volumen
```

1.  Aplica el manifiesto y verifica que el Pod esté funcionando:
    ```bash
    kubectl apply -f sidecar-pod.yaml
    kubectl get pod sidecar-pod
    ```
2.  Usa `port-forward` para acceder al Pod:
    ```bash
    kubectl port-forward sidecar-pod 8081:80
    ```
3.  Abre tu navegador y navega a `http://localhost:8081`. ¡Deberías ver la página que creó el contenedor "sidecar"\! Esto demuestra cómo dos contenedores dentro del mismo Pod pueden colaborar y compartir un volumen.

-----

### 4. 📋 Comparación: Pod vs. Contenedor de Docker

| Característica | Docker Container | Kubernetes Pod |
|:---|:---|:---|
| **Unidad de ejecución** | Una única imagen de contenedor | Uno o más contenedores |
| **Recursos compartidos** | No comparte red ni almacenamiento con otros contenedores por defecto | Todos los contenedores comparten red, IP y volúmenes |
| **Escalabilidad** | Gestionado manualmente con `docker run`/`docker-compose` | Gestionado por recursos de Kubernetes como `Deployments` |
| **Orquestación** | No hay orquestación nativa, requiere herramientas externas (`docker-compose`) | Es la unidad básica de orquestación en el clúster |

-----

### 5. 🧹 Limpieza

Cuando termines, es importante limpiar los recursos que creaste para evitar que se ejecuten innecesariamente.

```bash
kubectl delete -f nginx-pod.yaml
kubectl delete -f sidecar-pod.yaml
```

-----

### 6. 🎓 Qué Aprendiste

  * Un **Pod** es la unidad más pequeña de despliegue en Kubernetes y actúa como un envoltorio para uno o más contenedores.
  * Los contenedores dentro de un mismo Pod comparten la misma red, almacenamiento y recursos.
  * Aprendiste a usar comandos de `kubectl` como **`apply`**, **`get`**, **`describe`** y **`logs`** para gestionar los Pods.
  * El patrón **Sidecar** te mostró cómo usar múltiples contenedores en un Pod para colaborar en una tarea.

> 🎯 **Regla de oro:** En un entorno real, casi nunca gestionas Pods de forma individual. En su lugar, usas recursos de nivel superior como **Deployments** o **ReplicaSets** para gestionar automáticamente el ciclo de vida y la escalabilidad de tus Pods.
