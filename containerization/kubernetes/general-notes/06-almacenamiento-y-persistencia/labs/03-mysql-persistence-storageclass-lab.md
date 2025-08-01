# 📦 Lab: Persistencia de Datos con MySQL, PV, PVC y StorageClass

En este laboratorio, primero experimentarás la **pérdida de datos** que ocurre por defecto en Kubernetes. Luego, aprenderás a resolver ese problema desplegando un servidor de bases de datos MySQL con almacenamiento persistente.

-----

## 🚫 1. El problema: Pérdida de datos sin persistencia

Antes de usar la persistencia, vamos a desplegar un Pod de MySQL temporal para ver qué sucede con los datos cuando el Pod es eliminado.

### Manifiesto del Pod temporal (`mysql-temporal.yaml`)

```yaml
# Pod de MySQL sin ninguna configuración de volumen
apiVersion: v1
kind: Pod
metadata:
  name: mysql-temporal-pod
spec:
  containers:
  - name: mysql-temporal-container
    image: mysql:5.7
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    - name: MYSQL_DATABASE
      value: "temp_db"
    ports:
    - containerPort: 3306
```

### 1.1. Despliegue e inserción de datos

```bash
# Despliega el Pod temporal
kubectl apply -f mysql-temporal.yaml

# Espera a que el Pod esté en estado "Running"
kubectl get pod mysql-temporal-pod

# Obtiene logs del Pod temporal
kubectl logs mysql-temporal-pod

# Conéctate al Pod e inserta datos
kubectl exec -it mysql-temporal-pod -- mysql -uroot -ppassword temp_db
```

Una vez en la terminal de MySQL, ejecuta:

```sql
SHOW DATABASES;

USE temp_db;

CREATE TABLE usuarios (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255));

SHOW TABLES;

INSERT INTO usuarios (name) VALUES ('Juan'), ('Maria'), ('Pedro');

SELECT * FROM usuarios;

exit
```

> **Observación:** Los datos se ven perfectamente en el Pod.

### 1.2. Demostración de la pérdida de datos

```bash
# Elimina el Pod. ¡El Deployment no lo recreará porque es un Pod independiente!
kubectl delete pod mysql-temporal-pod

# Creamos nuevamente el Pod manualmente, simulando un reemplazo o reinicio sin persistencia.
kubectl apply -f mysql-temporal.yaml

# Espera a que el nuevo Pod esté en estado "Running".
kubectl get pod mysql-temporal-pod

# Conéctate al nuevo Pod y verifica si los datos persisten (spoiler: no lo harán).
kubectl exec -it mysql-temporal-pod -- mysql -uroot -ppassword temp_db
```

Una vez conectado, ejecuta :

```sql
SHOW DATABASES;

USE temp_db;

SHOW TABLES;

SELECT * FROM usuarios;

exit
```

> **Resultado:** La tabla y los datos **no existen**. Se perdieron para siempre al eliminar el Pod.

-----

## 📄 2. La solución: Aplica la persistencia

Ahora, vamos a resolver el problema anterior usando un `Deployment` con almacenamiento persistente.

### Archivo de definición (`mysql-persistence.yaml`)

```yaml
# --- 1. Definición de la StorageClass para el aprovisionamiento dinámico ---
# Minikube usa este provisioner para crear volúmenes hostPath de forma automática.
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: mysql-storage-class
provisioner: k8s.io/minikube-hostpath
# La política "Retain" mantendrá el volumen y los datos incluso si el PVC es eliminado.
reclaimPolicy: Retain
volumeBindingMode: Immediate

---

# --- 2. Definición del PersistentVolumeClaim (la "solicitud" de almacenamiento) ---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  storageClassName: mysql-storage-class

---

# --- 3. Definición del Deployment de MySQL ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  selector:
    matchLabels:
      app: mysql
  replicas: 1
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
        - name: MYSQL_DATABASE
          value: "mydatabase"
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-data
        persistentVolumeClaim:
          claimName: mysql-pvc
```

-----

## 🚀 3. Despliegue de los componentes

```bash
kubectl apply -f mysql-persistence.yaml
```

**Salida esperada:**

```
storageclass.storage.k8s.io/mysql-storage-class created
persistentvolumeclaim/mysql-pvc created
deployment.apps/mysql-deployment created
```

-----

## ✍️ 4. Inserta datos en la base de datos

```bash
# Obtiene el nombre del Pod
kubectl get pods -l app=mysql -o jsonpath="{.items[0].metadata.name}"

# Obtiene el nombre del Pod
# mysql-deployment-6d5df5969b-5gv9t
```

```bash
# Conéctate al Pod e inserta datos
kubectl exec -it mysql-deployment-6d5df5969b-5gv9t -- mysql -uroot -ppassword mydatabase
```

Una vez en la terminal de MySQL, ejecuta:

```sql
SHOW DATABASES;

USE mydatabase;

SHOW TABLES;

CREATE TABLE usuarios (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255));

INSERT INTO usuarios (name) VALUES ('Alice'), ('Bob'), ('Charlie');

SELECT * FROM usuarios;

exit
```

-----

## 🚀 5. Prueba la persistencia de los datos

```bash
# Elimina el Deployment. Esto también eliminará el Pod, pero no el volumen.
kubectl delete deployment mysql-deployment
```

```bash
# Vuelve a crear el Deployment.
kubectl apply -f mysql-persistence.yaml
```
Salida : 

```
storageclass.storage.k8s.io/mysql-storage-class unchanged
persistentvolumeclaim/mysql-pvc unchanged
deployment.apps/mysql-deployment created

```

```bash
# Espera a que el nuevo Pod esté en "Running".
kubectl get pods -l app=mysql
```

-----

## 🔍 6. Verifica que los datos persisten

Conéctate al nuevo Pod de MySQL para verificar que los datos están intactos.

```bash
# Obtiene el nombre del NUEVO Pod
kubectl get pods -l app=mysql -o jsonpath="{.items[0].metadata.name}"
# Obtiene el nombre del Pod
# mysql-deployment-6d5df5969b-g9qbh

# Conéctate al nuevo Pod de MySQL
kubectl exec -it mysql-deployment-6d5df5969b-g9qbh -- mysql -uroot -ppassword mydatabase
```

Una vez en la terminal, ejecuta :
```sql
SHOW DATABASES;

USE mydatabase;

SHOW TABLES;

SELECT * FROM usuarios;

exit
```

> **Resultado:** Los datos `Alice`, `Bob` y `Charlie` están intactos, confirmando la persistencia.

-----

## 🧹 7. Limpieza

```bash
# Elimina todos los recursos del laboratorio, incluyendo el Deployment y la PVC.
kubectl delete -f mysql-persistence-storageclass.yaml

# El PV se mantiene por la política "Retain". Debes eliminarlo manualmente.
# Encuentra el nombre del PV con: kubectl get pv
# Y luego elimínalo con:
kubectl delete pv <nombre-del-pv>

# Elimina el Pod temporal
kubectl delete -f mysql-temporal.yaml
```

## 🧾 8. Verificación y limpieza del volumen persistente

Incluso después de ejecutar `kubectl delete -f mysql-persistence-storageclass.yaml`, el volumen persistente no se elimina automáticamente.

### 🔍 8.1 Verifica que el PVC se liberó pero el PV sigue presente

```bash
kubectl get pv
```

> 🟡 El `STATUS` del volumen será `Released`, indicando que el PVC fue eliminado pero los datos permanecen.

---

### 🗂️ 8.2 Accede a Minikube y revisa si los datos persisten

```bash
minikube ssh
ls -lh /mnt/data
```

> ✅ Deberías ver una carpeta (por ejemplo `/mnt/data/default-mysql-pvc-xxxxx/`) que contiene los archivos de la base de datos MySQL.

---

### 🧹 8.3 Limpieza manual del volumen desde Minikube

Para borrar los archivos manualmente:

```bash
sudo rm -rf /mnt/data/default-mysql-pvc-*
```

> ⚠️ Este paso es **manual a propósito** porque Kubernetes **no elimina el contenido** del volumen cuando la política de retención es `Retain`. Esta política está diseñada para **conservar los datos aunque se elimine el PVC**, como medida de seguridad ante eliminaciones accidentales.

---

### 📘 ¿Por qué Kubernetes no borra los datos?

La política `persistentVolumeReclaimPolicy: Retain` le indica a Kubernetes:

> “Aunque se elimine el PVC, **no borres el volumen** automáticamente, ni su contenido.”

Esto permite recuperar o migrar los datos, lo cual es útil en entornos de producción o para auditorías. Pero también implica que **debes gestionar manualmente** esos datos cuando ya no se necesiten.

---

## ✅ ¿Qué aprendiste?

  * Que los Pods son efímeros y **pierden sus datos por defecto**.
  * La importancia de usar volúmenes persistentes para aplicaciones con estado.
  * Que un volumen persistente (`PV`) mantiene el estado de tus datos, incluso si el `Pod` es eliminado y recreado.
  * El uso de `StorageClass` y `PVC` para el aprovisionamiento dinámico.