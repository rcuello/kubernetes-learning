# ☁️ ¿Qué es GKE (Google Kubernetes Engine)?

GKE (Google Kubernetes Engine) es un **servicio gestionado de Kubernetes** ofrecido por Google Cloud Platform (GCP). Permite desplegar, administrar y escalar aplicaciones en contenedores utilizando la potencia de Kubernetes con la simplicidad de un servicio gestionado.

Google fue el creador de Kubernetes, por lo que GKE es uno de los servicios más maduros y profundamente integrados con el ecosistema de Kubernetes.

---

## 🎯 Objetivo de GKE

Permitir a los equipos de desarrollo y operaciones concentrarse en sus aplicaciones sin preocuparse por la complejidad de administrar el clúster subyacente.

---

## ⚙️ ¿Qué incluye GKE?

| Componente                  | ¿Gestionado por GKE? |
|----------------------------|----------------------|
| Plano de control (control plane) | ✅ Sí                |
| Nodos worker (instancias)        | ✅ (autopilot) o manual |
| Escalamiento automático          | ✅ Nodos y pods         |
| Monitoreo (Cloud Monitoring)     | ✅ Integrado            |
| Seguridad con IAM y Workload Identity | ✅ Integrado     |

---

## 🧱 Modos de operación

GKE ofrece dos modos para crear clústeres:

### 1. **Modo Estándar**
- Tú administras los nodos.
- Mayor control, ideal para entornos personalizados.

### 2. **Modo Autopilot**
- Google administra los nodos por ti.
- Solo pagas por los recursos consumidos por los pods.
- Ideal para workloads sin necesidad de control detallado.

---

## 🧰 Integraciones clave

- 🔐 IAM de GCP para control de acceso.
- 📈 Cloud Monitoring & Logging para métricas y logs.
- 🔄 Cloud Build y Artifact Registry para CI/CD.
- 🌍 Cloud Load Balancing para exponer servicios.

---

## 🖼️ Arquitectura simplificada

```plaintext
+------------------------------+
|   Google Cloud Console / CLI |
+---------------+--------------+
                |
                v
        +------------------+
        | Control Plane    |
        | (API, etcd, etc) |
        +------------------+
                |
                v
    +--------------------------+
    | Worker Nodes (VMs)      |
    | o Autopilot Pods        |
    +--------------------------+
````

---

## 🚀 Beneficios de GKE

* 🌐 Kubernetes nativo con soporte oficial de Google.
* 🔄 Actualizaciones automáticas del plano de control.
* 💵 Facturación por segundo en modo Autopilot.
* 🧩 Escalabilidad automática horizontal y vertical.
* 🧠 Seguridad avanzada con integraciones GCP.

---

## 📦 Casos de uso típicos

* Migración de microservicios a contenedores.
* Procesamiento de datos y pipelines.
* Aplicaciones con picos impredecibles de tráfico.
* Plataformas multi-tenant que requieren aislamiento.

---

## 🔗 Recursos adicionales

* [Documentación oficial de GKE](https://cloud.google.com/kubernetes-engine/docs)
* [Comparación Autopilot vs Estándar](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview)
* [Quickstart GKE con gcloud CLI](https://cloud.google.com/kubernetes-engine/docs/quickstarts)

---

## 📚 Siguiente paso

➡️ Lee el archivo `02-deploy-cluster-gke.md` para crear tu primer clúster GKE desde la consola o con `gcloud`.

