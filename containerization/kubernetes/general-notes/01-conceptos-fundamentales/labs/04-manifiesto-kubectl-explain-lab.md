# 📦 Lab: La anatomía de un objeto de Kubernetes y cómo dominarla

Este laboratorio te guiará a través de la estructura esencial de un manifiesto YAML. Aprenderás a usar la herramienta más poderosa para principiantes, `kubectl explain`, para entender cualquier atributo de un objeto y validar tus propios manifiestos.

> **Pre-requisitos:**
>
>   * Minikube iniciado: `minikube start`.
>   * `kubectl` configurado para usar Minikube.
>   * Un editor de texto simple (ej. Visual Studio Code) para crear archivos YAML.

-----

### 1 🚫 El Problema: El Manifiesto Mal Formado

Como principiante, es fácil cometer errores de sintaxis o adivinar los nombres de los atributos. Intentemos crear un `Pod` con un manifiesto incorrecto.

Crea un archivo llamado `pod-error-yaml` con el siguiente contenido. Nota que el atributo de imagen se llama `image-name`, que no es correcto.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-con-error
spec:
  containers:
  - image-name: nginx # ❌ Campo 'image-name' incorrecto
```

Ahora, intenta aplicar este manifiesto.

```bash
kubectl apply -f pod-error-yaml
```

**Salida esperada:**

```
error: error validating "pod-error-yaml": error validating data: ValidationError(Pod.spec.containers[0]): unknown field "image-name" in io.k8s.api.core.v1.Container; if you choose to ignore these errors, turn validation off with --validate=false
```

**Análisis del problema:** Kubernetes ha rechazado el manifiesto. El error es claro: `unknown field "image-name"`. Esto demuestra que no puedes adivinar los campos; debes conocer la estructura correcta. Aquí es donde `kubectl explain` se vuelve indispensable.

-----

### 2 ✅ La Solución: Usando `kubectl explain`

`kubectl explain` es tu manual de referencia. Nos dice exactamente qué campos son válidos para cada objeto y cómo se anidan.

1.  Usa `kubectl explain` para ver la estructura de un `Pod`:

    ```bash
    kubectl explain pod
    ```

    **Salida esperada:**
    (Verás una descripción general del objeto Pod y sus campos principales: `metadata`, `spec`, `status`).

2.  Ahora, usa un punto (`.`) para profundizar en la estructura. Investiguemos el campo `spec`:

    ```bash
    kubectl explain pod.spec
    ```

    **Salida esperada:**
    (Verás una descripción de la sección `spec` y los atributos que contiene, como `containers`, `nodeName`, etc.).

3.  Continuemos profundizando hasta encontrar la sección del contenedor:

    ```bash
    kubectl explain pod.spec.containers
    ```

    **Salida esperada:**
    (Verás la descripción de la sección `containers`. ¡Aquí encontrarás el campo `image`\!).

4.  Ahora que conocemos la estructura correcta, podemos escribir el manifiesto correctamente.

Crea un archivo llamado `nginx-pod.yaml`:

```yaml
apiVersion: v1 # 🎯 apiVersion: Versión de la API de Kubernetes.
kind: Pod # 🎯 kind: El tipo de objeto que estamos creando.
metadata: # 🎯 metadata: Información sobre el objeto, como el nombre y las etiquetas.
  name: nginx-pod
  labels:
    app: webserver
spec: # 🎯 spec: La especificación, o "estado deseado", de nuestro objeto.
  containers:
    - name: nginx-container
      image: nginx:1.14.2 # El campo correcto es 'image'
```

1.  Aplica el manifiesto correcto:
    ```bash
    kubectl apply -f nginx-pod.yaml
    ```
    **Salida esperada:**
    ```
    pod/nginx-pod created
    ```

-----

### 3 📊 Verificación y Casos Prácticos

#### Explorando atributos de metadata

Las etiquetas (`labels`) en la sección `metadata` son cruciales para organizar y seleccionar objetos en Kubernetes. Usaremos `kubectl` para verlas y filtrarlas.

1.  Muestra todas las etiquetas del `Pod`:

    ```bash
    kubectl get pods --show-labels
    ```

    **Salida esperada:**

    ```
    NAME        READY   STATUS    RESTARTS   AGE    LABELS
    nginx-pod   1/1     Running   0          ...    app=webserver
    ```

2.  Usa la etiqueta para filtrar los Pods:

    ```bash
    kubectl get pods -l app=webserver
    ```

    **Salida esperada:**
    (Solo se mostrará el Pod con esa etiqueta).

#### Entendiendo la sección `spec` de forma recursiva

Puedes usar el flag `--recursive` para ver una lista completa y anidada de todos los campos posibles para un objeto, lo cual es muy útil.

1.  Explora la estructura completa del `Pod.spec`:
    ```bash
    kubectl explain pod.spec --recursive
    ```
    **Salida esperada:**
    (Verás un listado muy largo, mostrando todas las posibles opciones, como `volumes`, `serviceAccountName`, `terminationGracePeriodSeconds`, etc.).

#### Validación de manifiestos sin aplicarlos

Para evitar errores, puedes validar tu manifiesto YAML antes de aplicarlo.

1.  Usa el flag `--dry-run=client` para verificar la sintaxis del manifiesto:
    ```bash
    kubectl apply -f nginx-pod.yaml --dry-run=client
    ```
    **Salida esperada:**
    (No se creará el Pod, pero si hay algún error de sintaxis, el comando fallará aquí. Si no hay errores, devolverá un mensaje de éxito simulado).

-----

### 4. 📋 Comparación: Manifiesto YAML vs. Dockerfile

| Característica | Dockerfile | Manifiesto YAML de Kubernetes |
|:---|:---|:---|
| **Propósito** | Define cómo construir una imagen de Docker. | Define el "estado deseado" de un objeto en el clúster. |
| **Sintaxis** | Conjunto de comandos con una sintaxis específica (`FROM`, `RUN`, `CMD`). | Formato de serialización de datos (`clave: valor`) con indentación. |
| **Unidad de enfoque** | Una imagen de contenedor. | Un objeto de Kubernetes (Pod, Deployment, Service, etc.). |

-----

### 5. 🧹 Limpieza

Elimina el Pod creado en el laboratorio usando el manifiesto:

```bash
kubectl delete -f nginx-pod.yaml
```

**Salida esperada:**

```
pod "nginx-pod" deleted
```

-----

### 6\. 🎓 Qué Aprendiste

  * Todo en Kubernetes se gestiona como un **objeto**, y estos se definen con **manifiestos YAML**.
  * Un manifiesto YAML tiene cuatro atributos principales: **`apiVersion`**, **`kind`**, **`metadata`** y **`spec`**.
  * **`kubectl explain`** es tu herramienta de auto-aprendizaje más valiosa para descubrir y comprender la estructura de cualquier objeto.
  * Las **etiquetas** (`labels`) en `metadata` son un método clave para organizar y buscar recursos.

> 🎯 **Regla de oro:** Si tienes dudas sobre un campo en un manifiesto, `kubectl explain [objeto].[campo]` es tu mejor amigo. Siempre valida tus manifiestos antes de aplicarlos.
