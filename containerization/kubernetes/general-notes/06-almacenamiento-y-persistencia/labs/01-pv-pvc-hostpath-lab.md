# 📦 Lab: PersistentVolume y PersistentVolumeClaim en Kubernetes

Este laboratorio te guía paso a paso en la creación de almacenamiento persistente en un entorno local con Minikube. Aprenderás a usar `PersistentVolume (PV)` y `PersistentVolumeClaim (PVC)` para montar un directorio del host (`hostPath`) en un Pod de Nginx que servirá un `index.html` personalizado.

---

## 🔧 1. Preparación del entorno en Minikube

Primero, crea el directorio en el nodo donde se alojará el volumen. Esto se hace directamente en la máquina virtual de Minikube.

```bash
minikube ssh

# Solo se puede manipular la carpeta "mnt" siendo administrador por eso se usa "sudo su"
sudo su

# Crea el directorio donde se alojará el volumen.
mkdir /mnt/nginx-pv-data

# Crea el archivo index.html que será servido por Nginx.
echo '<h1>Hello from Volume!</h1>' > /mnt/nginx-pv-data/index.html

# Verifica que el archivo y el directorio se crearon correctamente.
ls /mnt/nginx-pv-data/
cat /mnt/nginx-pv-data/index.html

# Sal de la sesión de root y de la VM de Minikube.
exit
exit
```

---

## 📄 2. Archivos de definición
Para este laboratorio, usaremos dos manifiestos.

### `pv-pvc.yaml`
Este manifiesto crea el volumen persistente y la solicitud de almacenamiento.

```yaml
# --- Definición del PersistentVolume (el "almacén") ---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv         # Nombre único para nuestro volumen persistente.
  labels:
    # Estas etiquetas se usan para que el PVC pueda "seleccionar" este PV específico.
    type: local
    app: nginx-storage
spec:
  # La capacidad total del volumen. El PVC debe solicitar una cantidad igual o menor.
  capacity:
    storage: 1Gi
  # Los modos de acceso que soporta este volumen.
  #   * "ReadWriteOnce" significa que un solo nodo puede montarlo como lectura/escritura.
  accessModes:
    - ReadWriteOnce
  # Esta clase de almacenamiento vincula el PV con el PVC.
  # Debe coincidir en ambos manifiestos.
  storageClassName: manual
  # Aquí se define el tipo de almacenamiento. En este caso, un directorio local en el nodo.
  hostPath:
    # La ruta del directorio en la máquina virtual de Minikube que se usará como almacenamiento.
    path: /mnt/nginx-pv-data
    type: Directory
---
# --- Definición del PersistentVolumeClaim (la "solicitud" o "llave") ---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc  # Nombre de la solicitud que usará nuestro Pod.
spec:
  storageClassName: manual
  # El selector de etiquetas que busca un PV con la etiqueta "type: local" y "app: nginx-storage".
  selector:
    matchLabels:
      type: local
      app: nginx-storage
  # Los modos de acceso que solicita el Pod. Deben ser compatibles con los del PV.    
  accessModes:
    - ReadWriteOnce
  # El tamaño de almacenamiento que se solicita. Debe ser igual o menor que la capacidad del PV.
  resources:
    requests:
      storage: 1Gi
```

### `pod.yaml`

Este manifiesto crea un `Pod` que utiliza la `PVC` que acabas de definir.

```yaml
# --- Definición del Pod que consume el almacenamiento ---
apiVersion: v1
kind: Pod
metadata:
  name: my-pod  # Nombre del Pod.
spec:
  containers:
    - name: my-container  # Nombre del contenedor.
      image: nginx        # La imagen de Nginx que se ejecutará.
      volumeMounts:
        # Aquí se especifica cómo se monta el volumen dentro del contenedor.
        - mountPath: /usr/share/nginx/html
          name: my-storage  # El nombre del volumen (debe coincidir con la sección "volumes").
  volumes:
    # Se define un volumen para el Pod.
    - name: my-storage
      persistentVolumeClaim:
        # Se especifica que este volumen se obtiene a través de un PVC.
        claimName: my-pvc
```

---

## 🚀 3. Despliegue de los componentes

### Aplica el PV y PVC:

```bash
kubectl apply -f pv-pvc.yaml
```

**Salida esperada:**

```
persistentvolume/my-pv created
persistentvolumeclaim/my-pvc created
```

### Crea el Pod:
Luego, despliega el **Pod**, que usará la PVC.

```bash
kubectl apply -f pod.yaml
```

**Salida esperada:**

```
pod/my-pod created
```

---

## 🔎 4. Verificación inicial

### Verifica el estado del almacenamiento:
Es crucial verificar que el volumen se ha vinculado correctamente.

```bash
kubectl get pv,pvc
```

**Salida esperada:**

```
NAME                     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM            STORAGECLASS   AGE
persistentvolume/my-pv   1Gi        RWO            Retain           Bound    default/my-pvc   manual         1m

NAME                           STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/my-pvc   Bound    my-pv    1Gi        RWO            manual         1m
```
> **Nota:** El estado `Bound` (`Vinculado`) confirma que el `PVC` `my-pvc` se ha enlazado exitosamente al `PV` `my-pv`.

### Verifica el estado del pod:

```bash
kubectl get pod my-pod
```

**Salida esperada:**

```
NAME     READY   STATUS    RESTARTS   AGE
my-pod   1/1     Running   0          1m
```

---

## 🧠 5. Verifica el montaje del volumen

```bash
kubectl describe pod my-pod
```

**Fragmento relevante:**

```
Volumes:
  my-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  my-pvc
    ReadOnly:   false
```

> **Nota:** Esta salida confirma que el Pod ha montado el volumen a través del PVC.

---

## 🔍 6. Explora el contenido dentro del Pod

```bash
kubectl exec my-pod -- ls -la /usr/share/nginx/html

# En Git Bash (Windows), usa:
kubectl exec my-pod -- sh -c "ls -la /usr/share/nginx/html"
```

**Salida esperada:**

```
-rw-r--r-- 1 root root    28 Jul 31 19:37 index.html
```

> **Observación:** La salida muestra el archivo `index.html` que creaste en el paso de preparación, lo que confirma que el volumen se montó correctamente.

---

## 🌐 7. Prueba el servidor Nginx

```bash
kubectl port-forward my-pod 8080:80
```

Visita en tu navegador:

```
http://localhost:8080
```

**Resultado esperado:**

```
Hello from Volume!
```

---

## 🧹 8. Limpieza y comprobación de persistencia

Elimina los componentes para ver cómo el volumen persiste.

```bash
# Elimina el Pod
kubectl delete pod my-pod
```

```bash
# El Pod se ha ido, pero ¿el contenido?
minikube ssh
ls /mnt/nginx-pv-data/
cat /mnt/nginx-pv-data/index.html
exit
exit
```

> **Observación:** Verás que el directorio y el archivo `index.html` siguen en la máquina virtual de Minikube. Esto demuestra la persistencia del volumen.

Finalmente, elimina el PV y PVC para limpiar completamente el entorno.

```bash
# Elimina el PV y PVC
kubectl delete -f pv-pvc.yaml
```

---

## ✅ ¿Qué aprendiste?

  * Cómo configurar un **PV** y un **PVC** y vincularlos manualmente.
  * La importancia de usar `hostPath` con una ruta accesible en el nodo.
  * Que los datos persisten en el volumen (`PV`) incluso si el Pod es eliminado.
  * Cómo validar y depurar problemas de almacenamiento y acceso.


