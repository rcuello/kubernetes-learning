# 🌐 ¿Qué es un Ingress en Kubernetes?

## 📘 Definición general

Un **Ingress** en Kubernetes es un recurso que permite **exponer múltiples servicios HTTP/HTTPS externos** desde un clúster, utilizando una única IP externa y reglas de enrutamiento.

> 🛣️ **Analogía urbana**: Imagina una avenida principal (la IP pública del clúster) con varios carteles que indican cómo llegar a distintos locales (Services). El **Ingress** actúa como ese sistema de señalización, indicando a qué servicio debe dirigirse una petición HTTP según su URL o dominio.

---

## 🧠 ¿Qué hace un Ingress?

- Define **reglas de enrutamiento** basadas en el host y/o el path.
- Permite **TLS (HTTPS)** usando certificados.
- Centraliza el acceso externo, evitando un LoadBalancer por cada servicio.
- Se apoya en un componente llamado **Ingress Controller** que lo implementa (como NGINX, Traefik o HAProxy).

---

## 🧬 Componentes del Ingress

| Componente           | Función clave                                              |
|----------------------|------------------------------------------------------------|
| Ingress              | Recurso que define las rutas de acceso HTTP(S)             |
| Ingress Controller   | Pod que interpreta y ejecuta las reglas definidas en Ingress |
| Cert-Manager (opcional) | Automatiza emisión y renovación de certificados TLS         |

---

## ✍️ Ejemplo básico de Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mi-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: miapp.local
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /frontend
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
````

---

## 🔐 Soporte para HTTPS (TLS)

El siguiente ejemplo permite HTTPS usando un certificado TLS cargado como `Secret`:

```yaml
spec:
  tls:
    - hosts:
        - miapp.local
      secretName: miapp-certificado
```

> 🧰 Para obtener certificados válidos con Let's Encrypt, se suele usar [cert-manager](https://cert-manager.io/).

---

## ⚙️ Requisitos

1. **Tener un Ingress Controller instalado**:

   * `NGINX Ingress Controller`: uno de los más usados.
   * Se instala como Deployment y escucha los recursos Ingress.

2. **DNS configurado** (o usar `/etc/hosts` en entornos locales).

3. **Servicios definidos previamente** para enrutar el tráfico.

---

## 🗺️ Ventajas del uso de Ingress

* Ahorra costos en LoadBalancers en la nube.
* Consolida todo el enrutamiento HTTP/S en un solo punto.
* Permite usar dominios personalizados por servicio o path.
* Facilita integración con herramientas de autenticación y TLS.

---

## 🚫 Limitaciones

* No soporta TCP/UDP (solo tráfico HTTP/S).
* Requiere un Ingress Controller compatible instalado y configurado.
* Puede tener complejidad adicional si usas múltiples namespaces o certificados.

---

## 🧪 Buenas prácticas

✅ Usa `pathType: Prefix` y sé explícito con las rutas.
✅ Define nombres DNS claros y organizados.
✅ Centraliza certificados TLS con `cert-manager`.
✅ Usa Ingress solo para servicios HTTP/HTTPS (usa `LoadBalancer` o `NodePort` para otros).

---

## 📍 Casos de uso típicos

* Exponer un frontend React desde `/` y una API desde `/api`.
* Usar dominios como `api.midominio.com` y `app.midominio.com`.
* Configurar HTTPS para todos los entornos con certificados gratuitos.
* Controlar el acceso con autenticación básica o JWT en NGINX.

---

## 🧭 ¿Qué sigue?

Ahora que entiendes cómo exponer servicios externos con Ingress, el siguiente paso es aprender sobre **ConfigMaps y Secrets**, recursos clave para manejar configuraciones y datos sensibles en tus aplicaciones.

📄 [Siguiente: 03-configmap-secret.md →](./03-configmap-secret.md)

