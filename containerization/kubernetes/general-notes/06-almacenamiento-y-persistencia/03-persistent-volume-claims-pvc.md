# 🧾 PersistentVolumeClaims (PVCs): Solicitando Almacenamiento Persistente en Kubernetes

En Kubernetes, los **PersistentVolumes (PVs)** representan las unidades de almacenamiento disponibles en un clúster. Sin embargo, los Pods no interactúan directamente con estos recursos. En su lugar, utilizan un mecanismo intermedio: los **PersistentVolumeClaims (PVCs)**.

Un **PersistentVolumeClaim (PVC)** es una **solicitud de almacenamiento persistente hecha por un usuario o una aplicación**. Permite que un Pod acceda a un volumen sin necesidad de conocer detalles sobre el almacenamiento físico subyacente.

---

## 📌 ¿Qué es un PVC?

Un PVC es un objeto de Kubernetes que:

* Define **cuánto almacenamiento necesita** una aplicación.
* Especifica **cómo se accederá** a ese almacenamiento (modo de acceso).
* Puede referirse opcionalmente a una **clase de almacenamiento** (`StorageClass`) específica.

💡 **Metáfora útil:** si un PV es una "oferta de almacenamiento", un PVC es una "solicitud de almacenamiento".

---

## 🎯 ¿Por Qué Usar PVCs?

Los PVCs ofrecen una abstracción que aporta importantes beneficios:

| Beneficio                            | Descripción                                                                                                |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------- |
| **Simplicidad para Desarrolladores** | Los desarrolladores solo definen sus necesidades de almacenamiento sin preocuparse por cómo se implementa. |
| **Desacoplamiento**                  | Separa la solicitud de almacenamiento (PVC) de su aprovisionamiento físico (PV).                           |
| **Portabilidad**                     | Las aplicaciones se pueden mover entre clústeres mientras haya una `StorageClass` compatible.              |
| **Aprovisionamiento Dinámico**       | Si no existe un PV adecuado, Kubernetes puede crear uno automáticamente usando la `StorageClass`.          |

---

## 🔍 Atributos Clave de un PVC

* `name`: Nombre único del claim.
* `namespace`: PVCs son objetos con ámbito de namespace.
* `accessModes`: Modo de acceso deseado:

  * `ReadWriteOnce` (RWO): Lectura/escritura por un único nodo.
  * `ReadOnlyMany` (ROX): Lectura por múltiples nodos.
  * `ReadWriteMany` (RWX): Lectura/escritura por múltiples nodos.
  * `ReadWriteOncePod` (RWOP): Exclusivo de un solo Pod.
* `resources.requests.storage`: Cantidad solicitada (e.g. `1Gi`, `500Mi`).
* `storageClassName`: Opcional. Define el tipo de almacenamiento deseado.

---

## 🔄 Ciclo de Vida de un PVC

1. **Creación:**

   * El usuario/aplicación define un objeto PVC con sus requisitos.

2. **Vinculación (Binding):**

   * Kubernetes encuentra un PV compatible.
   * Si hay una `StorageClass` con aprovisionamiento automático, se crea un PV dinámicamente.

3. **Uso:**

   * Un Pod utiliza el PVC para montar el volumen persistente.

4. **Eliminación:**

   * Al borrar el PVC, se libera el PV.
   * La política de `reclaim` del PV (`Retain`, `Delete`) determina si se eliminan o retienen los datos.

---

## 🧪 Ejemplo de PVC (Estático o Dinámico)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mi-pvc-para-app
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual-nfs
```

🔎 En este ejemplo:

* El PVC solicita 1GiB de almacenamiento.
* Requiere acceso de lectura/escritura desde un solo nodo.
* Se vinculará con un PV que tenga la `StorageClass` `manual-nfs` o activará un aprovisionador dinámico.

---

## 📦 Uso de un PVC desde un Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mi-aplicacion-con-estado
spec:
  containers:
  - name: servidor-web
    image: nginx:latest
    volumeMounts:
    - name: contenido-persistente
      mountPath: "/usr/share/nginx/html"
  volumes:
  - name: contenido-persistente
    persistentVolumeClaim:
      claimName: mi-pvc-para-app
```

✅ Esto le dice a Kubernetes que monte el volumen asociado al PVC `mi-pvc-para-app` dentro del contenedor, en la ruta especificada.

---

## 🧠 Buenas Prácticas

* Usa **`storageClassName` explícita** si necesitas un tipo de almacenamiento específico.
* Define **acceso mínimo necesario** (`RWO`, `ROX`, `RWX`) para mejorar seguridad y rendimiento.
* Asegúrate de tener una política de `reclaim` adecuada según el uso del volumen (`Retain` para producción crítica, `Delete` para entornos efímeros).
* Si usas aprovisionamiento dinámico, asegúrate de que la `StorageClass` exista y tenga un aprovisionador configurado.

---

## 🧩 Relación entre PV, PVC y Pod

```
[Pod] ─► [PVC] ─► [PV] ─► [Almacenamiento Físico o Lógico]
```

* **Pod** solicita un volumen usando un PVC.
* **PVC** encuentra o genera un PV compatible.
* **PV** se conecta al almacenamiento real (EBS, NFS, etc.).

---

## 🧭 Resumen

Los **PVCs** son el mecanismo que conecta a las aplicaciones con el almacenamiento persistente en Kubernetes. Junto con los **PVs**, permiten separar las responsabilidades entre desarrolladores y administradores:

* 👨‍💻 *Desarrolladores* → crean PVCs, usan almacenamiento sin preocuparse por su origen.
* 👩‍💼 *Administradores* → gestionan PVs y StorageClasses.

Esta arquitectura habilita la **automatización**, **portabilidad** y **escalabilidad** de aplicaciones con estado (`stateful`) dentro de Kubernetes.

