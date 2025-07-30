# 🌐 Visión General de Tipos de Services en Kubernetes

> Comprende los cuatro tipos principales de Service en Kubernetes, su alcance y cómo se utilizan para exponer tus aplicaciones.

---

## 📘 Recordatorio: ¿Qué es un Service?

Un **Service** en Kubernetes es una abstracción que proporciona una forma **estable y confiable de acceder a uno o más Pods**. Actúa como un proxy virtual y balanceador de carga, permitiendo que tu aplicación se comunique sin preocuparse por la naturaleza efímera de los Pods. El **`type`** de un Service es lo que define su método de exposición.

Para una introducción más detallada, consulta: [Qué es un Service en Kubernetes](./00-que-es-service.md)

---

## 🧠 Los Cuatro Tipos Principales de Service

Kubernetes ofrece cuatro tipos principales de Service, cada uno diseñado para un escenario de exposición diferente:

| Tipo           | Uso Principal                                     | Alcance           | Documentación Detallada |
|----------------|---------------------------------------------------|-------------------|-------------------------|
| `ClusterIP`    | Acceso **interno** al clúster                     | Solo Clúster      | [Detalles de ClusterIP](./02-clusterip-service.md) |
| `NodePort`     | Exposición básica **desde fuera** del clúster via los Nodos | Clúster / Externo | [Detalles de NodePort](./03-nodeport-service.md) |
| `LoadBalancer` | Exposición **externa avanzada** usando un Balanceador de Carga de la Nube | Externo           | [Detalles de LoadBalancer](./04-loadbalancer-service.md) |
| `ExternalName` | Redirección DNS a un nombre de host **externo** | Externo (DNS)     | [Detalles de ExternalName](./05-externalname-service.md) |

---

## 🧐 ¿Cuál Service Usar?

La elección del tipo de Service depende enteramente de tus necesidades:

* Utiliza **`ClusterIP`** para la comunicación entre tus microservicios dentro del clúster (por ejemplo, tu frontend hablando con tu backend). Es el tipo por defecto y el más seguro para comunicación interna.
* Considera **`NodePort`** para entornos de desarrollo, pruebas o si estás en un clúster on-premise sin un balanceador de carga dedicado. Permite un acceso rápido desde fuera del clúster a través de la IP de cualquier nodo y un puerto asignado.
* Si estás en un proveedor de nube (AWS, GCP, Azure), **`LoadBalancer`** es la opción ideal para exponer servicios al mundo exterior de forma robusta y escalable. Kubernetes se encarga de provisionar un balanceador de carga nativo de la nube.
* **`ExternalName`** es perfecto cuando tus aplicaciones en Kubernetes necesitan conectarse a servicios que residen *fuera* del clúster, como bases de datos gestionadas o APIs de terceros, sin proxyar el tráfico a través del clúster.

Para la mayoría de los casos de uso en producción, especialmente para aplicaciones web, la combinación de un **`ClusterIP` Service** para tu aplicación junto con un recurso **`Ingress`** (que a su vez es expuesto por un `LoadBalancer` o un `NodePort`) es la práctica recomendada.

---

## ➡️ Próximos Pasos: Explora Cada Tipo en Detalle

Haz clic en los enlaces de la tabla anterior para profundizar en cada tipo de Service, entender su funcionamiento interno, sus casos de uso específicos y ver ejemplos de manifestos.