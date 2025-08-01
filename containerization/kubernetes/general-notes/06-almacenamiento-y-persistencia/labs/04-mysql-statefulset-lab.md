# 📦 Lab: StatefulSet con MySQL - Identidad Persistente y Almacenamiento Automático

En este laboratorio aprenderás la diferencia entre un **Deployment** y un **StatefulSet** desplegando MySQL. Verás cómo StatefulSet proporciona **identidad persistente**, **almacenamiento automático por pod** y **nombres DNS estables**.

---

## 🚫 1. El problema: Deployment con múltiples réplicas de MySQL

Primero vamos a intentar desplegar MySQL con un Deployment normal para ver por qué **NO funciona** con bases de datos.

### 📄 Manifiesto problemático (`mysql-deployment-problema.yaml`)

```yaml
# ❌ ESTO NO FUNCIONARÁ CORRECTAMENTE
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment-problema
spec:
  replicas: 2  # ❌ Múltiples réplicas con la misma base de datos
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
          value: "testdb"
        ports:
        - containerPort: 3306
```

### 🧪 1.1. Despliegue y observación del problema

```bash
# Despliega el Deployment problemático
kubectl apply -f mysql-deployment-problema.yaml

# Observa los pods - nombres aleatorios
kubectl get pods -l app=mysql

# Ejemplo de salida:
# mysql-deployment-problema-7d4b5c8f9-abc12
# mysql-deployment-problema-7d4b5c8f9-def34

kubectl exec -it mysql-deployment-problema-7d4b5c8f9-abc12  -- mysql -uroot -ppassword testdb
kubectl exec -it mysql-deployment-problema-7d4b5c8f9-def34  -- mysql -uroot -ppassword testdb

kubectl delete pod mysql-deployment-problema-7d4b5c8f9-abc12

# Observa los pods - se creó un nuevo pod
kubectl get pods -l app=mysql

kubectl exec -it mysql-deployment-problema-7959f8868f-phz45  -- mysql -uroot -ppassword testdb

```

### Conecta al primer pod (mysql-deployment-problema-7d4b5c8f9-abc12)

```bash
kubectl exec -it mysql-deployment-problema-7d4b5c8f9-abc12  -- mysql -uroot -ppassword testdb
```

```sql
CREATE TABLE usuarios (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255), pod VARCHAR(50));
INSERT INTO usuarios (name, pod) VALUES ('Alice', 'pod-0'), ('Bob', 'pod-0');
SELECT * FROM usuarios;
exit
```

### Conecta al segundo pod (mysql-deployment-problema-7d4b5c8f9-def34)

```bash
kubectl exec -it mysql-deployment-problema-7d4b5c8f9-def34  -- mysql -uroot -ppassword testdb
```

```sql
CREATE TABLE productos (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255), precio DECIMAL(10,2));
INSERT INTO productos (name, precio) VALUES ('Laptop', 999.99), ('Mouse', 25.50);
SELECT * FROM productos;
exit
```

### Elimina al primer pod (mysql-deployment-problema-7d4b5c8f9-abc12)

```bash
kubectl get pods -l app=mysql
kubectl delete pod mysql-deployment-problema-7d4b5c8f9-abc12
```

### Consulta los pods

```bash
kubectl get pods -l app=mysql

# Ejemplo de salida:
# mysql-deployment-problema-7959f8868f-qbn4c (nuevo pod) (Alerta de Spoiler: la tabla y los datos no existen!)
# mysql-deployment-problema-7d4b5c8f9-def34

```

### Conecta al nuevo pod (mysql-deployment-problema-7959f8868f-qbn4c)

```bash
kubectl exec -it mysql-deployment-problema-7959f8868f-qbn4c  -- mysql -uroot -ppassword testdb
```

```sql
use testdb;
show tables;
exit
```

> ⚠️ **Problemas observados:**
> - Los pods tienen nombres **aleatorios** e impredecibles
> - **No hay almacenamiento persistente** - los datos se pierden
> - **Múltiples instancias** de MySQL intentando usar la misma base de datos
> - **No hay coordinación** entre las réplicas

```bash
# Limpia este experimento fallido
kubectl delete -f mysql-deployment-problema.yaml
```

---

## ✅ 2. La solución: StatefulSet con MySQL

Ahora despleguemos MySQL correctamente usando **StatefulSet** que resuelve todos los problemas anteriores.

### 📄 Archivo `mysql-statefulset.yaml`

```yaml
# --- 1. Service para acceso estable ---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql-sts
  ports:
  - port: 3306
    targetPort: 3306
  clusterIP: None  # Headless service para StatefulSet

---

# --- 2. StatefulSet de MySQL ---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
spec:
  serviceName: mysql-service
  replicas: 3
  selector:
    matchLabels:
      app: mysql-sts
  template:
    metadata:
      labels:
        app: mysql-sts
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
        - name: MYSQL_DATABASE
          value: "mydb"
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  # 🎯 CLAVE: volumeClaimTemplates crea un PVC por cada pod
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

---

## 🚀 3. Despliegue del StatefulSet

```bash
kubectl apply -f mysql-statefulset.yaml
```

Salida esperada:
```
service/mysql-service created
statefulset.apps/mysql-statefulset created
```

---

## 👀 4. Observa las diferencias clave

### 4.1. Nombres predecibles y ordenados

```bash
# Observa los nombres de los pods
kubectl get pods -l app=mysql-sts

# Salida esperada:
# mysql-statefulset-0   1/1     Running   0          2m
# mysql-statefulset-1   1/1     Running   0          1m
# mysql-statefulset-2   1/1     Running   0          30s
```

> ✅ **Ventaja:** Los pods tienen nombres **predecibles** (`-0`, `-1`, `-2`) en lugar de aleatorios.

### 4.2. Creación secuencial ordenada

```bash
# Observa cómo se crean en orden
kubectl get pods -l app=mysql-sts -w

# Verás que se crean uno por uno: primero -0, luego -1, luego -2
```

### 4.3. Almacenamiento persistente automático

```bash
# Cada pod tiene su propio PVC automáticamente
kubectl get pvc

# Salida esperada:
# mysql-data-mysql-statefulset-0   Bound    pvc-abc123   1Gi
# mysql-data-mysql-statefulset-1   Bound    pvc-def456   1Gi  
# mysql-data-mysql-statefulset-2   Bound    pvc-ghi789   1Gi
```

> ✅ **Ventaja:** **Cada pod tiene su propio almacenamiento** - no hay conflictos.

---

## 📊 5. Inserta datos diferentes en cada pod

Vamos a demostrar que cada pod mantiene sus **propios datos independientes**.

### 5.1. Conecta al primer pod (mysql-statefulset-0)

```bash
kubectl exec -it mysql-statefulset-0 -- mysql -uroot -ppassword mydb
```

```sql
CREATE TABLE usuarios (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255), pod VARCHAR(50));
INSERT INTO usuarios (name, pod) VALUES ('Alice', 'pod-0'), ('Bob', 'pod-0');
SELECT * FROM usuarios;
exit
```

### 5.2. Conecta al segundo pod (mysql-statefulset-1)

```bash
kubectl exec -it mysql-statefulset-1 -- mysql -uroot -ppassword mydb
```

```sql
CREATE TABLE productos (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255), precio DECIMAL(10,2));
INSERT INTO productos (name, precio) VALUES ('Laptop', 999.99), ('Mouse', 25.50);
SELECT * FROM productos;
exit
```

### 5.3. Conecta al tercer pod (mysql-statefulset-2)

```bash
kubectl exec -it mysql-statefulset-2 -- mysql -uroot -ppassword mydb
```

```sql
CREATE TABLE pedidos (id INT AUTO_INCREMENT PRIMARY KEY, cliente VARCHAR(255), total DECIMAL(10,2));
INSERT INTO pedidos (cliente, total) VALUES ('Juan Pérez', 150.00), ('María García', 89.99);
SELECT * FROM pedidos;
exit
```

---

## 🔍 6. Verifica la identidad DNS estable
En este paso, confirmaremos que los pods pueden encontrarse entre sí utilizando nombres de red predecibles.

### 📄 Manifiesto de Pod de depuración (si es necesario)

Si el comando `nslookup` o `ping` no está disponible en la imagen de MySQL, puedes usar un Pod temporal con la imagen `busybox` para realizar la prueba. Crea un archivo llamado `debug-pod.yaml`.

```yaml
# debug-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-pod
spec:
  containers:
  - name: debug-container
    image: busybox
    command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
```

```bash
# Aplica el manifiesto para crear el Pod de depuración
kubectl apply -f debug-pod.yaml
```

### 6.1. Prueba la resolución DNS desde un pod

Ahora, desde tu Pod de depuración o directamente desde el Pod de MySQL si tiene `nslookup`, verifica la conectividad.

```bash
# Si tienes nslookup en el pod de mysql (poco probable)
# kubectl exec -it mysql-statefulset-0 -- nslookup mysql-statefulset-1.mysql-service.default.svc.cluster.local

# ✅ Si usas el Pod de depuración (MÉTODO RECOMENDADO):
# Ejecuta este comando para ver la dirección IP del Pod mysql-statefulset-1
kubectl exec -it debug-pod -- nslookup mysql-statefulset-1.mysql-service.default.svc.cluster.local
```

**Salida esperada (ejemplo):**

```
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      mysql-statefulset-1.mysql-service.default.svc.cluster.local
Address 1: 172.17.0.6 mysql-statefulset-1.mysql-service.default.svc.cluster.local
```

> ✅ **Ventaja:** Cada pod tiene un **nombre DNS estable** que no cambia, lo que permite la comunicación interna entre las réplicas.

### 6.2. Limpieza del Pod de depuración

```bash
# Elimina el Pod de depuración cuando hayas terminado
kubectl delete -f debug-pod.yaml
```

-----

## 🔄 7. Prueba la persistencia eliminando un pod

### 7.1. Elimina el pod-1

```bash
# Elimina específicamente el pod-1
kubectl delete pod mysql-statefulset-1

# Abre otra terminal para observar cómo se recrea AUTOMÁTICAMENTE con el mismo nombre
kubectl get pods -l app=mysql-sts -w
```

### 7.2. Verifica que los datos persisten

```bash
# Una vez que el nuevo pod-1 esté Running, conecta y verifica los datos
kubectl exec -it mysql-statefulset-1 -- mysql -uroot -ppassword mydb
```

```sql
SELECT * FROM productos;  -- Los datos siguen ahí!
exit
```

> ✅ **Resultado:** El nuevo pod mantiene **el mismo nombre**, **el mismo almacenamiento** y **los mismos datos**.

---

## 📈 8. Escalamiento ordenado

### 8.1. Escala hacia arriba

```bash
# Escala a 5 réplicas
kubectl scale statefulset mysql-statefulset --replicas=5

# Observa cómo se crean en orden: -3, luego -4
kubectl get pods -l app=mysql-sts -w
```

### 8.2. Escala hacia abajo

```bash
# Escala hacia abajo a 2 réplicas
kubectl scale statefulset mysql-statefulset --replicas=2

# Observa cómo se eliminan en ORDEN INVERSO: -4, luego -3
kubectl get pods -l app=mysql-sts -w
```

> ✅ **Ventaja:** El escalamiento es **ordenado y predecible**.

---

## 🔍 9. Inspección del almacenamiento

```bash
# Los PVCs persisten incluso si reduces las réplicas
kubectl get pvc

# Verás que los PVCs de los pods eliminados siguen existiendo
# Esto permite recuperar datos si vuelves a escalar hacia arriba
```

---

## 🧹 10. Limpieza

```bash
# Elimina el StatefulSet
kubectl delete -f mysql-statefulset.yaml

# Elimina el deployment problemático
kubectl delete -f mysql-deployment-problema.yaml


# Los PVCs NO se eliminan automáticamente (por seguridad)
kubectl get pvc

# Si quieres eliminar también los datos:
kubectl delete pvc mysql-data-mysql-statefulset-0
kubectl delete pvc mysql-data-mysql-statefulset-1
kubectl delete pvc mysql-data-mysql-statefulset-2
# ... y cualquier otro que exista
```

---

## 📋 11. Comparación: Deployment vs StatefulSet

| Aspecto | Deployment | StatefulSet |
|---------|------------|-------------|
| **Nombres de pods** | Aleatorios (`pod-abc123`) | Ordenados (`pod-0`, `pod-1`) |
| **Creación** | Paralela | Secuencial |
| **DNS** | No garantizado | Estable y predecible |
| **Almacenamiento** | Compartido o manual | Automático por pod |
| **Escalamiento** | Aleatorio | Ordenado |
| **Uso ideal** | Apps stateless | Bases de datos, apps stateful |

---

## ✅ ¿Qué aprendiste?

* **StatefulSet** proporciona identidad persistente a los pods
* **Cada pod obtiene su propio almacenamiento** automáticamente con `volumeClaimTemplates`
* Los pods tienen **nombres DNS estables** para comunicación directa
* El **escalamiento es ordenado** (secuencial hacia arriba, inverso hacia abajo)
* Es **ideal para bases de datos** y aplicaciones que requieren identidad única
* Los **datos persisten** incluso si el pod se elimina y recrea

> 🎯 **Caso de uso real:** Este patrón es perfecto para bases de datos como MySQL, PostgreSQL, MongoDB, sistemas de colas como Kafka, o cualquier aplicación que necesite identidad persistente y almacenamiento dedicado por instancia.