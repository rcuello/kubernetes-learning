# 🌐 Introducción a Services e Ingress en Kubernetes

En Kubernetes, los Pods se crean y destruyen de manera dinámica. Para permitir que aplicaciones dentro y fuera del clúster se comuniquen con estos Pods, Kubernetes ofrece mecanismos para **exponer servicios** de forma controlada, escalable y segura. Los dos principales mecanismos son:

- **Services (Servicios)**: proporcionan una IP estable y un punto de acceso interno o externo para uno o varios Pods.
- **Ingress**: gestiona el enrutamiento HTTP/S desde el exterior hacia múltiples servicios dentro del clúster, actuando como un **controlador de entrada** con soporte para reglas, TLS y dominios personalizados.

---

## 🔗 ¿Por qué no exponer directamente un Pod?

Los Pods tienen IPs efímeras y no son accesibles directamente desde fuera del clúster. Por eso usamos:

| Mecanismo     | Uso principal                                      | Accesibilidad |
|---------------|-----------------------------------------------------|----------------|
| **ClusterIP** | Acceso interno dentro del clúster                   | Interno         |
| **NodePort**  | Expone el servicio en un puerto fijo de cada nodo  | Interno/Externo |
| **LoadBalancer** | Asigna IP pública mediante proveedor cloud       | Externo (Cloud) |
| **Ingress**   | Gestión avanzada de rutas HTTP/HTTPS               | Externo         |

---

## 🧭 ¿Cuándo usar Service vs Ingress?

| Escenario                                        | Usa `Service` | Usa `Ingress` |
|--------------------------------------------------|---------------|----------------|
| Acceso interno entre microservicios              | ✅ Sí         | ❌ No          |
| Exponer un servicio por puerto fijo (NodePort)   | ✅ Sí         | ❌ No          |
| Exponer múltiples rutas (ej. `/api`, `/web`)     | ❌ No         | ✅ Sí          |
| Soporte para certificados TLS                    | ❌ Limitado   | ✅ Completo     |
| Uso de balanceador de carga en la nube           | ✅ Sí (LoadBalancer) | ✅ Sí (con Ingress Controller) |

---

## 🧱 Componentes básicos a cubrir

Durante esta sección exploraremos:

1. `Service`:
   - ClusterIP
   - NodePort
   - LoadBalancer
   - Headless services (para StatefulSets)
2. `Ingress`:
   - Ingress básico
   - Ingress con TLS
   - Ingress con múltiples rutas
   - Controladores populares (NGINX, Traefik, etc.)
3. Comparaciones con otros métodos (port-forward, externalName)
4. Consideraciones de seguridad, escalabilidad y buenas prácticas

---

## ☁️ Consideraciones para la nube

En entornos cloud (como AKS, GKE o EKS), Kubernetes puede **provisionar balanceadores de carga reales** (tipo ELB, ALB, etc.) cuando usas un Service tipo `LoadBalancer` o un Ingress con el controlador adecuado. Esto facilita la exposición pública sin configurar IPs manualmente.

---

## 🚦 Prerrequisitos

Antes de avanzar, asegúrate de tener:

- Un clúster Kubernetes funcionando (local o en la nube)
- `kubectl` configurado
- Conocimientos básicos sobre Pods y Deployments
- (Opcional) Helm para instalar Ingress Controllers más fácilmente

---

## 📌 Conclusión

Dominar `Service` e `Ingress` es fundamental para construir aplicaciones modernas en Kubernetes. Aprenderás no solo a exponer tus servicios correctamente, sino a **rutearlos de forma segura, eficiente y escalable**, tanto dentro como fuera del clúster.
