# ☁️ ¿Qué es AKS (Azure Kubernetes Service)?

AKS (Azure Kubernetes Service) es un **servicio gestionado de Kubernetes** que ofrece Microsoft Azure. Permite crear, configurar y administrar clústeres de Kubernetes de forma sencilla y con alta integración a la infraestructura de Azure.

---

## 🎯 Objetivo de AKS

Facilitar la adopción de Kubernetes sin tener que preocuparse por:

- El aprovisionamiento de nodos o VMs.
- La instalación y actualización del plano de control (control plane).
- La gestión de certificados, autoescalado, seguridad y monitoreo.

---

## ⚙️ ¿Qué gestiona Azure por ti?

| Componente                | Gestionado por Azure | Gestionado por el usuario |
|--------------------------|----------------------|----------------------------|
| Control Plane            | ✅ Sí                | ❌ No                      |
| Nodos Worker             | ❌ No (tú los defines) | ✅ Sí                      |
| Networking (CNI)         | ✅ Parcialmente       | ✅ (si usas avanzadas)     |
| Monitoreo (con Azure Monitor) | ✅ Opcional       | ✅ Personalizable          |

---

## 🧱 Arquitectura de AKS

```plaintext
+---------------------------+
|        Azure CLI / Portal|
+------------+-------------+
             |
             v
     +------------------+           +--------------------------+
     | Control Plane    | <-------> | Azure Managed Resources |
     | (K8s API, etcd)  |           +--------------------------+
     +------------------+
             |
             v
     +------------------+
     | Worker Nodes     |
     | (tu aplicación)  |
     +------------------+
````

---

## 🚀 Beneficios principales

* ✅ **Escalabilidad automática** (autoscaler).
* 🔐 **Integración con Azure Active Directory** (AAD).
* 📈 **Monitoreo con Azure Monitor y Log Analytics**.
* 💳 **Costo eficiente**: solo pagas por los nodos, no por el control plane.
* 🔄 **Actualizaciones automáticas y con cero downtime**.

---

## 📦 Casos de uso comunes

* Empresas que ya usan Azure y desean una solución Kubernetes nativa.
* Despliegue de microservicios en producción sin administrar infraestructura.
* Integración de CI/CD con GitHub Actions o Azure DevOps.

---

## 🔗 Recursos adicionales

* [Documentación oficial de AKS](https://learn.microsoft.com/es-es/azure/aks/)
* [Tutorial de AKS en Azure Portal](https://learn.microsoft.com/es-es/azure/aks/tutorial-kubernetes-deploy-cluster)
* [Azure CLI: crear clúster de AKS](https://learn.microsoft.com/es-es/cli/azure/aks)

---

## 📚 Siguiente paso

➡️ Lee el archivo `02-deploy-cluster-aks.md` para aprender cómo crear tu primer clúster AKS paso a paso.
