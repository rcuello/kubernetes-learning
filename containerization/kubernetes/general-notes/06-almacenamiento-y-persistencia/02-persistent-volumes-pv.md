# 💾 Volúmenes Persistentes (PVs): La Fundación del Almacenamiento en Kubernetes

En Kubernetes, los **Pods** son efímeros por naturaleza: pueden ser creados, eliminados o reprogramados en cualquier nodo en cualquier momento. Esto implica que cualquier dato almacenado localmente dentro de un Pod se pierde cuando dicho Pod deja de existir.

Para que las aplicaciones conserven sus datos de manera **persistente y desacoplada del ciclo de vida de los Pods**, Kubernetes proporciona una solución robusta: los **PersistentVolumes (PVs)**.

---

## 📦 ¿Qué es un PersistentVolume (PV)?

Un **PersistentVolume (PV)** es una unidad de almacenamiento provista en el clúster, ya sea de forma manual por un administrador o de manera automática mediante aprovisionamiento dinámico.

Funciona como un **recurso de almacenamiento compartido a nivel de clúster**, y puede estar respaldado por distintas tecnologías: discos locales, NFS, o volúmenes administrados por proveedores cloud como AWS EBS, Google Persistent Disk, Azure Disk, entre otros.

> Piensa en un PV como un disco disponible en la red que ha sido registrado en el clúster y está **listo para ser utilizado por las aplicaciones**, pero sin estar vinculado aún a ninguna.

---

## 🧩 ¿Dónde encajan los Pods y las aplicaciones?

Aunque los PVs representan el almacenamiento disponible, **los Pods no se conectan directamente a ellos**. En su lugar, utilizan un recurso llamado **PersistentVolumeClaim (PVC)** para **solicitar** almacenamiento, con ciertas características (tamaño, tipo de acceso, clase de almacenamiento, etc.).

🔄 Esta separación entre PV (oferta) y PVC (demanda) permite:

* Asignación flexible del almacenamiento.
* Reutilización de recursos.
* Separación de roles entre desarrolladores y administradores.

> En este documento nos enfocamos en los PVs. Veremos los **PVCs** y su rol específico en el siguiente capítulo.

---

## 🔑 Características Clave de un PV

* **Independencia de los Pods**: El PV sobrevive a la destrucción o recreación de los Pods. La persistencia de los datos está garantizada mientras el volumen no se libere o elimine.

* **Recurso de clúster**: Los PVs no pertenecen a un solo namespace; están disponibles a nivel de clúster y pueden ser consumidos por cualquier Pod que cumpla las condiciones necesarias.

* **Capacidad y Modo de Acceso**: Cada PV declara:

  * `storage`: tamaño del volumen (por ejemplo, `10Gi`).
  * `accessModes`: cómo puede montarse el volumen:

    * `ReadWriteOnce` (RWO): lectura/escritura por un único nodo.
    * `ReadOnlyMany` (ROX): solo lectura por múltiples nodos.
    * `ReadWriteMany` (RWX): lectura/escritura por múltiples nodos (no siempre soportado).
    * `ReadWriteOncePod` (RWOP): lectura/escritura por un único Pod (desde Kubernetes 1.22).

* **Tipo de Almacenamiento**: Especifica el backend del volumen (NFS, iSCSI, AWS EBS, Azure Disk, etc.).

* **Clase de Almacenamiento (`storageClassName`)**:

  * Define cómo debe aprovisionarse el volumen.
  * Permite controlar aspectos como el rendimiento, replicación o zona de disponibilidad.
  * Es fundamental para el aprovisionamiento dinámico.

---

## 🔁 Ciclo de Vida de un PV

1. ### Aprovisionamiento (`Provisioning`)

   * **Estático**: El administrador crea manualmente el objeto `PersistentVolume`, apuntando a un volumen ya existente.
   * **Dinámico**: Cuando un usuario crea un PVC que especifica una `StorageClass`, Kubernetes solicita automáticamente al proveedor de almacenamiento crear un volumen físico y genera el PV correspondiente.

2. ### Vinculación (`Binding`)

   * Cuando se crea un PVC, Kubernetes busca un PV que **cumpla con los requisitos solicitados** (capacidad, modo de acceso, clase de almacenamiento).
   * Si lo encuentra, los recursos se enlazan de forma exclusiva: un PV queda **asociado a un único PVC**.

3. ### Uso (`Using`)

   * Una vez vinculado, el PVC puede ser referenciado por un Pod.
   * El volumen se monta en el contenedor y la aplicación puede leer o escribir en él según lo configurado.

4. ### Liberación (`Releasing`)

   * Cuando se elimina el PVC, el PV queda en estado `Released`.
   * El recurso ya no puede ser reclamado por otro PVC hasta que sea reciclado o eliminado manualmente, dependiendo de su política.

5. ### Reclamación (`Reclaiming`)

   Cada PV tiene una política de `persistentVolumeReclaimPolicy` que define qué hacer con los datos cuando el PVC se elimina:

   * **`Retain`**: Los datos permanecen. El administrador debe limpiarlos y liberar el recurso manualmente.
   * **`Delete`**: El volumen subyacente y el PV se eliminan automáticamente. Útil con aprovisionamiento dinámico.
   * **`Recycle`**: (Obsoleto) Eliminaba el contenido y dejaba el volumen listo para ser reutilizado. Reemplazado por `Delete`.

---

## 🧪 Ejemplo de PV con aprovisionamiento estático (NFS)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mi-pv-nfs
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual-nfs
  nfs:
    path: /export/data-mi-app
    server: 192.168.1.100
```

📌 En este ejemplo:

* Se define un volumen NFS de 10Gi.
* Permite acceso `ReadWriteMany`, útil para múltiples Pods.
* Tiene política de `Retain`, por lo que se requiere gestión manual tras la liberación.
* Solo puede ser reclamado por PVCs que especifiquen la `storageClassName: manual-nfs`.

---

## 🧭 Conclusión y Próximos Pasos

Los **PersistentVolumes (PVs)** permiten desacoplar el almacenamiento del ciclo de vida de los Pods y facilitan una gestión estructurada y segura de datos persistentes en Kubernetes.

En el siguiente documento exploraremos cómo las aplicaciones **solicitan este almacenamiento** a través de los **PersistentVolumeClaims (PVCs)**: la interfaz con la que los Pods acceden a estos volúmenes.
