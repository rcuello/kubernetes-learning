# 🌐 Exponer Servicios en Kubernetes con Ingress

En Kubernetes, existen varias formas de exponer aplicaciones hacia el exterior. Este documento detalla las diferencias entre los tipos de `Service` y cómo utilizar `Ingress` como una forma más flexible y declarativa de enrutar tráfico.

---

## 📦 Tipos de `Service`

### 1. `ClusterIP` (por defecto)

* Solo accesible dentro del clúster.
* No expone puertos externamente.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

🔒 **Ideal para comunicación entre pods.**

---

### 2. `NodePort`

* Expone el servicio en un puerto estático de cada nodo (`nodeIP:nodePort`).
* Se puede acceder desde fuera del clúster si conoces el IP del nodo y el puerto.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

⚠️ **Limitado en control de rutas y requiere exponer puertos manualmente.**

---

### 3. `LoadBalancer`

* Proporciona una IP pública a través del balanceador de carga del proveedor cloud (solo funciona si estás en un entorno como AKS, EKS, GKE o usas MetalLB en local).

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

🌍 **Ideal para producción en la nube, pero puede generar costos.**

---

## 🚪 `Ingress`: Enrutamiento HTTP Avanzado

El `Ingress` permite exponer múltiples servicios HTTP en el mismo dominio o IP con reglas por ruta o subdominio.

### Requisitos

* Tener un **controlador de ingress** (como `ingress-nginx`, `traefik`, `istio`, etc.)
* Tener servicios tipo `ClusterIP` (generalmente) que serán referenciados por el `Ingress`.

---

### 🧪 Ejemplo Completo con `ingress-nginx`

#### 1. **Servicio expuesto internamente (ClusterIP)**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

#### 2. **Ingress básico por ruta**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: myapp.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
```

📌 Accede con `http://myapp.local` (requiere modificar el `/etc/hosts` si es local).

---

### 🔐 Ejemplo con TLS

```yaml
spec:
  tls:
    - hosts:
        - myapp.local
      secretName: myapp-tls
  rules:
    - host: myapp.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
```

🔐 **Requiere un `Secret` con el certificado TLS** (`myapp-tls`).

---

### 📥 Ruta a múltiples servicios

```yaml
rules:
  - host: mydomain.local
    http:
      paths:
        - path: /api
          pathType: Prefix
          backend:
            service:
              name: api-service
              port:
                number: 80
        - path: /app
          pathType: Prefix
          backend:
            service:
              name: frontend-service
              port:
                number: 80
```

🔁 **Permite enrutar `/api` al backend y `/app` al frontend.**

---

## ✅ Recomendaciones

| Escenario                  | Recomendación         |
| -------------------------- | --------------------- |
| Solo interno               | `ClusterIP`           |
| Acceso puntual desde fuera | `NodePort`            |
| En la nube                 | `LoadBalancer`        |
| Varios servicios HTTP      | `Ingress + ClusterIP` |

---

¿Deseas agregar ejemplos con `traefik`, `istio`, autenticación básica o integración con cert-manager para TLS automático? Puedo complementar este documento según tu stack.

---
