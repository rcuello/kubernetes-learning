
# 🐮 ¿Qué es Rancher?

**Rancher** es una plataforma de gestión de clústeres Kubernetes de código abierto que permite a los equipos de desarrollo, operaciones y seguridad administrar de manera centralizada múltiples entornos Kubernetes, ya sea en la nube, on-premise o en entornos híbridos.

---

## 🌎 ¿Qué problema resuelve?

Aunque Kubernetes es poderoso, su gestión puede volverse compleja cuando se tienen múltiples clústeres distribuidos en diferentes entornos. Rancher centraliza esta gestión y ofrece una **experiencia de usuario visual, segura y eficiente** para operar Kubernetes a escala.

---

## 🎯 Funcionalidades principales

| Funcionalidad               | Descripción                                                                |
| --------------------------- | -------------------------------------------------------------------------- |
| 🌐 UI web                   | Interfaz gráfica para gestionar clústeres, namespaces, pods y más          |
| 🔐 Control de acceso (RBAC) | Gestión de usuarios y permisos integrada con SSO, LDAP, GitHub, etc.       |
| ⚙️ Gestión de clústeres     | Importar, crear y administrar clústeres Kubernetes (EKS, AKS, GKE, RKE...) |
| 🧱 Catálogo de aplicaciones | Instalar aplicaciones vía Helm Charts con un solo clic                     |
| 🔄 Actualizaciones seguras  | Control de versiones, upgrades de clúster y monitoreo                      |
| 📊 Monitoreo integrado      | Soporte para Prometheus, Grafana, alertas y logs                           |

---

## 📦 ¿Qué es RKE?

Rancher incluye su propia herramienta llamada **RKE (Rancher Kubernetes Engine)** para crear clústeres Kubernetes fácilmente sobre servidores físicos o virtuales.

---

## 🧪 ¿Cómo usar Rancher en local (con Docker)?

Puedes levantar Rancher para pruebas locales con un solo comando:

```bash
docker run -d --restart=unless-stopped \
  -p 8080:80 -p 8443:443 \
  --name rancher \
  rancher/rancher:latest
```

Una vez ejecutado, accede a:

```
https://localhost:8443
```

> 🛑 Usa certificados autofirmados para pruebas locales. En producción se recomienda usar TLS válido.

---

## 🧠 Analogía de Kubernetes:

> Si Kubernetes es un sistema operativo para el centro de datos, **Rancher es el panel de control visual para operar múltiples sistemas operativos Kubernetes a la vez.**

Así como un **sistema operativo** te permite ejecutar procesos, controlar recursos y gestionar usuarios en un solo equipo, Rancher te permite **orquestar y supervisar clústeres Kubernetes enteros** con una experiencia más accesible y colaborativa.

---

## 📚 Referencias

* Sitio oficial: [https://rancher.com](https://rancher.com)
* GitHub: [https://github.com/rancher/rancher](https://github.com/rancher/rancher)
* Documentación: [https://docs.ranchermanager.rancher.io](https://docs.ranchermanager.rancher.io)
