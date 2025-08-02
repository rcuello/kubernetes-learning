# 📦 Lab: ReplicaSet - Garantizando la alta disponibilidad de tus Pods

Este laboratorio te guiará paso a paso para entender y usar **ReplicaSet** en Kubernetes. Aprenderás a asegurar que una aplicación siempre tenga el número de réplicas deseado, incluso si los pods fallan o se eliminan.

> **Pre-requisitos:**
>
>   * Docker Desktop y Minikube instalados y ejecutándose.
>   * Una terminal (PowerShell o Git Bash) con `kubectl` configurado para interactuar con Minikube.
>   * Inicia Minikube con el comando `minikube start`.

-----

1.  🚫 El Problema: Un Pod solitario no es resiliente

Imagina que despliegas un Pod para tu aplicación, pero este Pod por alguna razón deja de funcionar. En un entorno de producción, esto significa que tu servicio estaría caído. Sin un controlador que lo supervise, Kubernetes no hará nada para recuperarlo.

Vamos a probarlo. Crea el siguiente manifiesto `unstable-pod.yaml`:

```yaml
# unstable-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mi-pod-solitario
  labels:
    app: mi-app
spec:
  containers:
  - name: servidor-web
    image: nginx
    ports:
    - containerPort: 80
```

```bash
# Crea el Pod
kubectl apply -f unstable-pod.yaml

# Revisa el estado del Pod
kubectl get pods
# 🎯 Salida esperada: El Pod está en estado "Running"
# NAME                READY   STATUS    RESTARTS   AGE
# mi-pod-solitario    1/1     Running   0          10s
```

Ahora, simulemos un fallo eliminando el Pod manualmente.

```bash
# ❌ Eliminemos el Pod
kubectl delete pod mi-pod-solitario

# Verifiquemos si se ha recuperado
kubectl get pods
# ❌ Salida esperada: No hay pods con ese nombre.
# No resources found in default namespace.
```

**❌ ¿Por qué no funciona?** El Pod ha sido eliminado y Kubernetes no ha hecho nada para reemplazarlo. Un Pod por sí solo no tiene capacidad de autocuración. Necesitamos un controlador que se encargue de "observar" y "reconciliar" el estado deseado.

-----

2.  ✅ La Solución: Presentamos ReplicaSet

**ReplicaSet** es un controlador que asegura que un número específico de réplicas de pods (especificado en el campo `spec.replicas`) se estén ejecutando en todo momento. Si un pod falla, ReplicaSet lo crea de nuevo automáticamente.

Creemos un manifiesto llamado `my-replicaset.yaml` para nuestra aplicación:

```yaml
# my-replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mi-replicaset
spec:
  replicas: 3 # 🎯 Número de réplicas que queremos mantener
  selector:
    matchLabels:
      app: mi-app # 🎯 Selector para que ReplicaSet encuentre sus Pods
  template: # 🎯 Plantilla para crear nuevos Pods
    metadata:
      labels:
        app: mi-app
    spec:
      containers:
      - name: servidor-web
        image: nginx
        ports:
        - containerPort: 80
```

```bash
# Aplica el manifiesto
kubectl apply -f my-replicaset.yaml

# Verifica que el ReplicaSet se ha creado y sus Pods
kubectl get rs
# 🎯 Salida esperada: Un ReplicaSet con 3 réplicas listas
# NAME            DESIRED   CURRENT   READY   AGE
# mi-replicaset   3         3         3       10s

# Revisa los Pods creados por el ReplicaSet
kubectl get pods -l app=mi-app
# 🎯 Salida esperada: 3 Pods en estado "Running"
# NAME                      READY   STATUS    RESTARTS   AGE
# mi-replicaset-abcde       1/1     Running   0          10s
# mi-replicaset-fghij       1/1     Running   0          10s
# mi-replicaset-klmno       1/1     Running   0          10s
```

Ahora, **simulemos el fallo** de uno de los pods eliminándolo.

```bash
# ❌ Elimina uno de los Pods (el nombre será diferente)
kubectl delete pod mi-replicaset-abcde

# Observa cómo ReplicaSet crea uno nuevo casi inmediatamente
kubectl get pods -l app=mi-app
# 🎯 Salida esperada: A pesar de la eliminación, el número de pods sigue siendo 3. ReplicaSet ha creado uno nuevo.
# NAME                      READY   STATUS    RESTARTS   AGE
# mi-replicaset-fghij       1/1     Running   0          1m
# mi-replicaset-klmno       1/1     Running   0          1m
# mi-replicaset-pqrst       1/1     Running   0          5s
```

**🎯 Resultado:** ReplicaSet ha actuado como un guardián, detectando que el número de réplicas no coincidía con el deseado (`replicas: 3`) y creando un nuevo Pod para restaurar el estado deseado.

-----

3.  📊 Verificación y Casos Prácticos

#### Escalar horizontalmente

Una de las grandes ventajas de ReplicaSet es su capacidad para escalar tu aplicación de forma sencilla. Aumentemos el número de réplicas a 5.

```bash
# Escala el ReplicaSet a 5 réplicas
kubectl scale --replicas=5 rs mi-replicaset

# Verifica que se han creado los nuevos Pods
kubectl get rs mi-replicaset
# 🎯 Salida esperada: El número de réplicas deseadas y actuales es 5.
# NAME            DESIRED   CURRENT   READY   AGE
# mi-replicaset   5         5         5       2m

kubectl get pods -l app=mi-app
# 🎯 Salida esperada: Ahora hay 5 pods en total
```

#### Etiquetado y Selectores

Los `labels` y `selectors` son la clave para que ReplicaSet sepa qué Pods debe gestionar. Cambia una etiqueta de un Pod para ver cómo ReplicaSet lo "abandona".

```bash
# Obtén el nombre de un Pod
kubectl get pods -l app=mi-app -o jsonpath='{.items[0].metadata.name}'
# 🎯 Salida esperada: mi-replicaset-xxxxx

# Remueve la etiqueta `app: mi-app` de un pod
kubectl label pod [NOMBRE_DEL_POD] app-

# Ahora, verifica el estado. ReplicaSet creará un nuevo Pod para mantener el total de 5.
kubectl get pods -l app=mi-app
# 🎯 Salida esperada: 5 Pods con la etiqueta `app: mi-app`.
# El Pod que modificaste ya no está en la lista.
```

#### Múltiples ReplicaSets

Puedes tener múltiples ReplicaSets, cada uno gestionando su propio conjunto de Pods. Esto es útil para separar entornos o versiones. Crea uno nuevo para una "versión 2".

```yaml
# replicaset-v2.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mi-replicaset-v2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mi-app-v2
  template:
    metadata:
      labels:
        app: mi-app-v2
    spec:
      containers:
      - name: servidor-web
        image: busybox # Usamos una imagen diferente para distinguir
        command: ["sh", "-c", "echo 'Hola desde la v2!' && sleep 3600"]
```

```bash
kubectl apply -f replicaset-v2.yaml

kubectl get rs
# 🎯 Salida esperada: Dos ReplicaSets, cada uno con sus réplicas.
# NAME                 DESIRED   CURRENT   READY   AGE
# mi-replicaset        5         5         5       10m
# mi-replicaset-v2     2         2         2       10s
```

-----

5.  🧹 Limpieza

Elimina los ReplicaSets para limpiar todos los recursos, incluyendo los Pods que crearon.

```bash
# Elimina los ReplicaSets
kubectl delete rs mi-replicaset mi-replicaset-v2

# Confirma que los recursos se han eliminado
kubectl get rs
# 🎯 Salida esperada: No se encuentran recursos.
# No resources found in default namespace.
```

-----

6.  🎓 Qué Aprendiste

  * Aprendiste que los **Pods** por sí solos no son resilientes.
  * Descubriste cómo **ReplicaSet** actúa como un controlador para mantener un número fijo de réplicas de tus Pods.
  * Comprendiste la importancia de los **selectores (selectors)** y las **etiquetas (labels)** para que ReplicaSet identifique qué pods debe gestionar.
  * Practicaste cómo **escalar** tus aplicaciones horizontalmente usando `kubectl scale`.
  * Viste cómo ReplicaSet automáticamente **reemplaza** un Pod fallido o eliminado, garantizando la alta disponibilidad.

> 🎯 **Regla de oro:** ReplicaSet es tu guardián. Su único trabajo es asegurar que el número de réplicas sea siempre el que has deseado, sin importar lo que pase.
