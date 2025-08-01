# 📦 Lab: Persistencia de Datos con MySQL, PV, PVC y Volumen Manual (`hostPath`)

En este laboratorio, primero experimentarás la **pérdida de datos** que ocurre por defecto en Kubernetes. Luego, aprenderás a resolver ese problema desplegando un servidor de bases de datos MySQL con almacenamiento persistente mediante un volumen manual montado en el host de Minikube.

---

## 🚫 1. El problema: Pérdida de datos sin persistencia

Antes de usar la persistencia, vamos a desplegar un Pod de MySQL temporal para ver qué sucede con los datos cuando el Pod es eliminado.

### 📄 Manifiesto del Pod temporal (`mysql-temporal.yaml`)

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

### 🧪 1.1. Despliegue e inserción de datos

```bash
# Despliega el Pod temporal
kubectl apply -f mysql-temporal.yaml

# Espera a que el Pod esté en estado "Running"
kubectl get pod mysql-temporal-pod

# Conéctate al Pod e inserta datos
kubectl exec -it mysql-temporal-pod -- mysql -uroot -ppassword temp_db
```

```sql
SHOW DATABASES;
USE temp_db;
CREATE TABLE usuarios (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255));
INSERT INTO usuarios (name) VALUES ('Juan'), ('Maria'), ('Pedro');
SELECT * FROM usuarios;
exit
```

> ✅ **Resultado:** Los datos son visibles dentro del Pod.

### ❌ 1.2. Demostración de pérdida de datos

```bash
# Elimina el Pod
kubectl delete pod mysql-temporal-pod

# Vuelve a crear el Pod
kubectl apply -f mysql-temporal.yaml

# Espera a que el nuevo Pod esté listo
kubectl get pod mysql-temporal-pod

# Conéctate e intenta ver los datos
kubectl exec -it mysql-temporal-pod -- mysql -uroot -ppassword temp_db
```

```sql
SHOW DATABASES;
USE temp_db;
SHOW TABLES;
SELECT * FROM usuarios;
```

> ⚠️ **Resultado:** La tabla no existe. ¡Los datos se han perdido!

---

## ✅ 2. La solución: Persistencia con volumen manual

Ahora solucionaremos el problema anterior con un volumen manual basado en `hostPath`, garantizando que los datos sobrevivan al reinicio del Pod.

### 📄 Archivo `mysql-persistence-hostpath.yaml`

```yaml
# --- 1. PersistentVolume manual con hostPath ---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /data/mysql
  volumeMode: Filesystem

---

# --- 2. PersistentVolumeClaim que se enlaza al PV por storageClassName ---
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
  volumeName: mysql-pv
  storageClassName: manual
  volumeMode: Filesystem

---

# --- 3. Deployment de MySQL que monta el PVC ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
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

---

## 🚀 3. Despliegue del entorno persistente

```bash
kubectl apply -f mysql-persistence-hostpath.yaml
```

Salida esperada:

```
persistentvolume/mysql-pv created
persistentvolumeclaim/mysql-pvc created
deployment.apps/mysql-deployment created
```

---

## ✍️ 4. Inserta datos en la base persistente

```bash
# Obtiene el nombre del Pod
kubectl get pods -l app=mysql -o jsonpath="{.items[0].metadata.name}"

# Verifica el status del pod
kubectl get pod <pod-name>
kubectl get pod mysql-deployment-6d5df5969b-ttgnj

# Ver logs del pod
kubectl logs <pod-name>
kubectl logs mysql-deployment-6d5df5969b-ttgnj

# Conéctate al Pod
kubectl exec -it <pod-name> -- mysql -uroot -ppassword mydatabase

# Ejemplo:
kubectl exec -it mysql-deployment-6d5df5969b-ttgnj -- mysql -uroot -ppassword mydatabase
```

```sql
SHOW DATABASES;
USE mydatabase;
CREATE TABLE usuarios (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255));
INSERT INTO usuarios (name) VALUES ('Alice'), ('Bob'), ('Charlie');
SELECT * FROM usuarios;
exit
```

---

## 🔄 5. Reinicia el Deployment

```bash
# Elimina el Deployment (pero no el volumen)
kubectl delete deployment mysql-deployment

# Reaplica el manifiesto
kubectl apply -f mysql-persistence-hostpath.yaml
```

---

## 🔍 6. Verifica que los datos persisten

```bash
# Obtiene el nuevo nombre del Pod
kubectl get pods -l app=mysql -o jsonpath="{.items[0].metadata.name}"

# Conéctate nuevamente
kubectl exec -it <new-pod-name> -- mysql -uroot -ppassword mydatabase
```

```sql
SELECT * FROM mydatabase.usuarios;
```

> ✅ **Resultado esperado:** Los datos de `Alice`, `Bob`, y `Charlie` están intactos.

---

## 🧭 7. Visualización desde el host Minikube

```bash
minikube ssh
ls -lh /data/mysql
```

Deberías ver archivos como:

```bash
ibdata1  ib_logfile0  mysql/  mydatabase/  ...
```

> ⚠️ No edites directamente los archivos desde el host. Úsalo solo para inspección.

---

## 🧹 8. Limpieza

```bash
kubectl delete -f mysql-persistence-hostpath.yaml
kubectl delete -f mysql-temporal.yaml
```

---

## 🧹 9. Limpieza avanzada del volumen manual (`hostPath`)

Aunque hayas eliminado el Deployment, es posible que los archivos de la base de datos **aún existan** en el disco del host de Minikube. Esto se debe a la configuración del volumen persistente.

### 🔍 9.1. Verifica si aún existen archivos

```bash
minikube ssh
ls -lh /data/mysql
```

Si ves archivos como:

```bash
ibdata1  ib_logfile0  mysql/  mydatabase/
```

Significa que los datos aún están físicamente presentes en el host.

---

### ❓ ¿Por qué no se eliminaron automáticamente?

Esto ocurre porque el `PersistentVolume` se creó con esta política:

```yaml
persistentVolumeReclaimPolicy: Retain
```

Esta opción **conserva el volumen y sus datos** incluso si se elimina el `PersistentVolumeClaim`. Esto es útil para evitar pérdidas accidentales de datos, pero implica que debes **eliminar manualmente los archivos** si ya no los necesitas.

> 🧠 Además, como estás usando `hostPath`, Kubernetes **no puede gestionar dinámicamente** la provisión y destrucción del volumen, como lo haría con un storage class administrado en la nube.

---

### 🗑️ 9.2. Elimina manualmente los archivos

Si estás seguro de que no necesitas más esos datos:

```bash
# Desde la VM de Minikube
minikube ssh

# Borra todo el contenido del directorio
sudo rm -rf /data/mysql

# Verifica que esté vacío o que el directorio no exista
ls -lh /data/mysql
```

---

### ✅ Resultado esperado

Después de la limpieza, deberías ver:

```bash
ls: cannot access '/data/mysql': No such file or directory
```

> Ahora el entorno está completamente limpio para futuros laboratorios o pruebas.

---

## ✅ ¿Qué aprendiste?

* Que los Pods son efímeros y pierden su estado por defecto.
* Cómo usar un `PersistentVolume` manual con `hostPath`.
* Que los datos pueden sobrevivir al reinicio de un Pod.
* Cómo inspeccionar los datos directamente desde el host Minikube.
