# 📦 StorageClass en Kubernetes

Una **StorageClass** en Kubernetes define **cómo debe aprovisionarse el almacenamiento persistente de manera dinámica**, ofreciendo distintos "perfiles" o "calidades" de almacenamiento según las necesidades de las aplicaciones.

---

## 🧠 ¿Por qué usar StorageClass?

Tradicionalmente, un administrador creaba manualmente volúmenes (`PersistentVolumes`) para que los desarrolladores los consumieran vía `PersistentVolumeClaims`. Con `StorageClass`, este proceso se **automatiza**, permitiendo:

* Provisionamiento dinámico de volúmenes.
* Abstracción de detalles del backend de almacenamiento.
* Configuración de políticas de retención y replicación.
* Soporte para múltiples tipos de almacenamiento (local, NFS, cloud, etc.).

---

## ⚙️ Estructura de un StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

### Campos clave

| Campo                  | Descripción                                                                                                                   |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `provisioner`          | Plugin de almacenamiento que crea el volumen (ej. `kubernetes.io/aws-ebs`, `kubernetes.io/gce-pd`, `csi.storage.k8s.io/...`). |
| `parameters`           | Parámetros específicos del proveedor (tipo de disco, zona, etc.).                                                             |
| `reclaimPolicy`        | Política de limpieza (`Delete`, `Retain`, `Recycle`).                                                                         |
| `volumeBindingMode`    | Cuándo se liga el volumen: `Immediate` o `WaitForFirstConsumer`.                                                              |
| `allowVolumeExpansion` | Si permite aumentar el tamaño del volumen.                                                                                    |

---

## 🧪 Ejemplo: StorageClass para NFS dinámico

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-dynamic
provisioner: nfs.csi.k8s.io
parameters:
  server: 10.0.0.20
  share: /export/data
reclaimPolicy: Retain
volumeBindingMode: Immediate
```

---

## 🔄 Flujo de uso con PVC

1. El usuario crea un `PersistentVolumeClaim` y especifica la `storageClassName`.
2. Kubernetes usa el `provisioner` definido en esa `StorageClass` para crear un `PersistentVolume`.
3. El `PV` es enlazado automáticamente al `PVC`.

> ⚠️ Si no se especifica `storageClassName`, Kubernetes usará la `StorageClass` marcada como `default` (si existe).

---

## 🔍 Ver las StorageClass disponibles

```bash
kubectl get storageclass
```

Salida típica:

```bash
NAME             PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   kubernetes.io/gce-pd   Delete          Immediate            true                  20d
fast-ssd             kubernetes.io/aws-ebs  Delete          WaitForFirstConsumer false                 10d
```

---

## ✅ Buenas prácticas

* Define una `StorageClass` por tipo de carga: `fast-ssd`, `backup-nfs`, `standard-hdd`, etc.
* Usa `WaitForFirstConsumer` si tu volumen debe estar en la misma zona que el Pod.
* Establece políticas de expansión (`allowVolumeExpansion: true`) cuando tus cargas crezcan con el tiempo.
* Controla la eliminación de volúmenes con `reclaimPolicy`.

---

## 📌 Conclusión

`StorageClass` permite a Kubernetes **provisionar almacenamiento de forma dinámica**, facilitando entornos self-service, despliegues automatizados y configuraciones multi-tenant más seguras y eficientes.

