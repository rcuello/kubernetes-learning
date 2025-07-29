Claro, aquí tienes una versión clara, didáctica y estructurada del archivo `01-que-es-helm.md`, ideal para tu repositorio `kubernetes-learning/general-notes/04-gestion-aplicaciones/`:

---

# 📦 ¿Qué es Helm?

## 🧠 Introducción

**Helm** es el **gestor de paquetes oficial de Kubernetes**. Su función principal es ayudarte a **definir, instalar y actualizar aplicaciones Kubernetes** de forma reproducible y estandarizada. Helm empaqueta todos los recursos necesarios de Kubernetes en un solo paquete llamado **Chart**.

Es similar a lo que hacen los gestores de paquetes como `apt`, `yum` o `npm`, pero orientado a aplicaciones que se ejecutan dentro de un clúster Kubernetes.

---

## 🎯 ¿Por qué usar Helm?

* 🔁 Despliegue reproducible: facilita el versionamiento y control de cambios.
* 📦 Empaquetado completo: todo lo necesario (Deployments, Services, ConfigMaps, etc.) en un solo Chart.
* 🔄 Actualizaciones simplificadas: permite upgrades y rollbacks automáticos.
* ⚙️ Parametrización: puedes usar valores dinámicos en tus manifiestos YAML.
* 📁 Reutilización: Charts pueden ser compartidos en repositorios públicos o privados.

---

## 🧾 Analogía

Imagina que desplegar una app en Kubernetes manualmente es como **cocinar una receta desde cero**: compras ingredientes, mezclas cantidades, ajustas el sabor…

Helm sería como usar una **caja de receta lista para usar** donde solo defines algunas preferencias (`values.yaml`) y el resto se instala automáticamente con un solo comando:

```bash
helm install mi-app .
```

---

## 📐 Componentes de Helm

| Componente    | Descripción                                                               |
| ------------- | ------------------------------------------------------------------------- |
| `Chart`       | Un paquete Helm: contiene todos los recursos K8s + metadatos + valores    |
| `values.yaml` | Archivo con parámetros configurables (ej. replicas, image, puertos, etc.) |
| `templates/`  | Archivos YAML con lógica templating (`{{ }}`) que generan manifiestos K8s |
| `Chart.yaml`  | Archivo con metadatos del Chart (nombre, versión, descripción)            |
| `helm` CLI    | Herramienta de línea de comandos para instalar y gestionar Charts         |

---

## 🚀 Comandos básicos de Helm

```bash
# Instalar un Chart (por nombre o ruta)
helm install mi-app ./mi-chart

# Ver los releases instalados
helm list

# Ver valores por defecto de un Chart
helm show values bitnami/nginx

# Desinstalar una app
helm uninstall mi-app

# Actualizar una app con nuevos valores
helm upgrade mi-app ./mi-chart -f nuevos-valores.yaml

# Hacer rollback a una versión anterior
helm rollback mi-app 1
```

---

## 🔍 Ejemplo visual de estructura de un Chart

```
mi-chart/
├── charts/             # Dependencias (otros charts)
├── templates/          # YAMLs con lógica Helm
│   └── deployment.yaml
│   └── service.yaml
├── values.yaml         # Variables que puede personalizar el usuario
└── Chart.yaml          # Metadatos del Chart
```

---

## 📚 Recursos adicionales

* Sitio oficial: [https://helm.sh](https://helm.sh)
* Repositorio de Charts oficiales: [https://artifacthub.io](https://artifacthub.io)
* Curso gratuito recomendado: *Helm para desarrolladores de Kubernetes (Udemy / YouTube)*

---

## 🧪 ¿Qué sigue?

En el siguiente archivo (`02-instalar-helm.md`) aprenderás a instalar Helm en tu entorno y a usar un Chart oficial en Minikube o cualquier otro clúster Kubernetes.

