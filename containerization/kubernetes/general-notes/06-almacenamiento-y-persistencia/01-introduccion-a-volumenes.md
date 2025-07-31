# 📦 Introducción a los Volúmenes en Kubernetes

En Kubernetes, **los Pods son efímeros por naturaleza**: pueden eliminarse, reiniciarse o migrar de nodo en cualquier momento. Esto significa que **cualquier dato almacenado dentro del contenedor se pierde** si el Pod desaparece. 

> 🧨 Ejemplo clásico: una base de datos que escribe en el sistema de archivos del contenedor. Si el Pod reinicia, ¡los datos desaparecen!

Para solucionar este problema, Kubernetes introduce el concepto de **volúmenes**, una abstracción para manejar almacenamiento que **sobrevive al ciclo de vida del contenedor**.

---

## 🤔 ¿Por qué necesitamos Volúmenes?

Los volúmenes permiten separar los datos del ciclo de vida del contenedor. Entre los motivos principales para usarlos están:

1. **Persistencia de datos:** Evitar la pérdida de información crítica al reiniciar o recrear Pods.
2. **Compartir información:** Varios contenedores en un mismo Pod pueden acceder a un volumen común.
3. **Inyección de configuración o secretos:** ConfigMaps, Secrets o metadatos del Pod pueden montarse como archivos.
4. **Desacoplar almacenamiento del entorno de ejecución:** Mejora portabilidad y mantenibilidad.

---

## 📁 ¿Qué es un Volumen en Kubernetes?

Un **volumen** es un directorio accesible desde uno o más contenedores dentro de un Pod, montado en una ruta específica del sistema de archivos. Aunque su ciclo de vida depende del Pod, **su contenido puede persistir** dependiendo del tipo de volumen.

> 💡 No todos los volúmenes persisten datos entre reinicios del Pod. Algunos solo existen mientras el Pod está vivo.

---

## 📌 Tipos Básicos de Volúmenes en Kubernetes

### 1. `emptyDir` – Almacenamiento temporal compartido

- Se crea cuando el Pod es asignado a un nodo y se elimina cuando el Pod desaparece.
- Es útil para almacenamiento temporal o como medio de comunicación entre contenedores del mismo Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-con-emptydir
spec:
  containers:
  - name: main
    image: busybox
    command: ["/bin/sh", "-c", "echo 'Temporal' > /data/info.txt; sleep 3600"]
    volumeMounts:
    - name: temp-vol
      mountPath: /data
  volumes:
  - name: temp-vol
    emptyDir: {}
````

> ⚠️ Todo lo que se escriba en `emptyDir` se pierde cuando el Pod se destruye.

---

### 2. `hostPath` – Acceso directo al sistema de archivos del nodo

* Monta una ruta del nodo físico dentro del contenedor.
* Puede persistir datos incluso si el Pod se elimina.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-con-hostpath
spec:
  containers:
  - name: escritor-log
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo $(date) >> /var/log/app.log; sleep 5; done"]
    volumeMounts:
    - name: log-vol
      mountPath: /var/log
  volumes:
  - name: log-vol
    hostPath:
      path: /mnt/data/logs
      type: DirectoryOrCreate
```

> ⚠️ No recomendado para producción: compromete la portabilidad y seguridad del clúster.

---

### 3. Otros volúmenes útiles

| Tipo                    | Uso común                                                |
| ----------------------- | -------------------------------------------------------- |
| `configMap`             | Monta configuraciones externas como archivos             |
| `secret`                | Monta credenciales o claves de forma segura              |
| `downwardAPI`           | Expone metadatos del Pod (nombre, namespace, etiquetas)  |
| `persistentVolumeClaim` | Abstracción para usar almacenamiento persistente externo |

> 🛑 `gitRepo` fue deprecado. Para clonar código, considera `initContainers` o imágenes pre-construidas.

---

## 🧱 ¿Qué sigue?

Si bien `emptyDir` y `hostPath` pueden resolver casos simples o de desarrollo, **no son adecuados para producción** o para aplicaciones que necesitan almacenamiento duradero, replicable y portable.

Para eso, Kubernetes define una arquitectura de almacenamiento basada en:

* **PersistentVolumes (PV):** Recursos del clúster que representan almacenamiento físico.
* **PersistentVolumeClaims (PVC):** Solicitudes de almacenamiento que hacen los Pods.

> 🧩 Este modelo desacopla el almacenamiento de los Pods, permitiendo integración con proveedores externos como NFS, AWS EBS, Azure Disk, Google Persistent Disk, etc.

---

## ✅ Resumen

| Tipo                   | Persiste datos        | Ciclo de vida                  | Uso recomendado                  |
| ---------------------- | --------------------- | ------------------------------ | -------------------------------- |
| `emptyDir`             | ❌ No                  | Mientras viva el Pod           | Almacenamiento temporal          |
| `hostPath`             | ✅ Sí (en el nodo)     | Hasta que se borre manualmente | Desarrollo local, debugging      |
| `configMap` / `secret` | ⚠️ Solo configuración | Mientras viva el Pod           | Inyección de configuración       |
| `PVC`                  | ✅ Sí (externo)        | Independiente del Pod          | Producción y alta disponibilidad |

---

## 📚 Recursos adicionales

* [Documentación oficial sobre Volúmenes](https://kubernetes.io/es/docs/concepts/storage/volumes/)
* [Comparativa entre tipos de volúmenes](https://kubernetes.io/docs/concepts/storage/volumes/#types-of-volumes)
* [Guía de PersistentVolumes](https://kubernetes.io/es/docs/concepts/storage/persistent-volumes/)
