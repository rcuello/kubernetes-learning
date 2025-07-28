# 💾 Volúmenes persistentes en Kubernetes (PV y PVC)

En Kubernetes, los contenedores son efímeros: si un pod se reinicia o se reprograma, **todo su sistema de archivos desaparece**. Para solucionar esto, Kubernetes usa **volúmenes persistentes (PV)** y **reclamaciones de volumen persistente (PVC)**.

---

## 📦 ¿Qué es un volumen persistente (PV)?

Un **PersistentVolume (PV)** es un recurso del clúster que representa un **almacenamiento físico o lógico existente** (EBS en AWS, disco en GCP, NFS, etc.). Es **provisionado por un administrador** o por el propio clúster si hay soporte para aprovisionamiento dinámico.

> 🧠 **Analogía**: Imagina un disco duro externo conectado al clúster.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mi-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
````

---

## 🧾 ¿Qué es una PVC (PersistentVolumeClaim)?

Un **PVC (PersistentVolumeClaim)** es una **solicitud de almacenamiento** que hace un pod. El usuario no se preocupa por los detalles del volumen: solo dice cuánto necesita y cómo quiere acceder.

> 🧠 **Analogía**: El PVC es como una reserva de hotel: pides una habitación con ciertas características, y el sistema te asigna una disponible.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mi-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

---

## 🔗 ¿Cómo se conectan PV y PVC?

* Kubernetes empareja automáticamente un PVC con un PV disponible y compatible.
* El pod **nunca usa el PV directamente**: accede al volumen a través del PVC.

---

## 🧪 Ejemplo de uso en un Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-con-storage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ejemplo
  template:
    metadata:
      labels:
        app: ejemplo
    spec:
      containers:
        - name: app
          image: nginx
          volumeMounts:
            - mountPath: "/usr/share/nginx/html"
              name: almacenamiento
      volumes:
        - name: almacenamiento
          persistentVolumeClaim:
            claimName: mi-pvc
```

---

## 📚 Tipos de acceso (`accessModes`)

| Modo            | Descripción                                          |
| --------------- | ---------------------------------------------------- |
| `ReadWriteOnce` | Montado en solo un nodo en modo lectura/escritura    |
| `ReadOnlyMany`  | Montado en múltiples nodos en modo lectura           |
| `ReadWriteMany` | Montado en múltiples nodos en modo lectura/escritura |

---

## ⚙️ Aprovisionamiento dinámico

Puedes usar **StorageClasses** para que Kubernetes cree automáticamente los PV cuando creas un PVC.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
```

Y luego el PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mi-pvc-dinamico
spec:
  storageClassName: fast-ssd
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

---

## 🧭 Buenas prácticas

✅ Usa StorageClass para manejar distintos tipos de disco por entorno.
✅ Limpia los PVCs que ya no uses para liberar recursos.
✅ Usa `ReadWriteMany` con precaución: no todos los proveedores lo soportan.
✅ Para backups considera herramientas como Velero o snapshots nativas de tu nube.

---

## 🔄 Comandos útiles

```bash
# Ver volúmenes y reclamos
kubectl get pv
kubectl get pvc

# Ver uso en pods
kubectl describe pod <nombre-pod>

# Eliminar un PVC (no borra el PV automáticamente si es estático)
kubectl delete pvc mi-pvc
```

---

## 📌 Conclusión

Los volúmenes persistentes son clave para aplicaciones con estado (bases de datos, CMS, sistemas de archivos compartidos). Kubernetes ofrece una abstracción potente y flexible para conectarse a distintas soluciones de almacenamiento en la nube o locales.

📄 [Siguiente: 05-statefulset.md →](./05-statefulset.md)

