# 🌐 Servicios en Kubernetes e Ingress

En Kubernetes, los **servicios** son una abstracción que expone una aplicación en ejecución en un conjunto de Pods como un servicio de red. Para exponerlos externamente, se pueden utilizar varios tipos de servicios, y una capa adicional llamada **Ingress** para gestionar el acceso HTTP.

---

## 🛠️ Tipos de Servicios en Kubernetes

### 1. ClusterIP (por defecto)

Expone el servicio solo dentro del clúster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: servicio-clusterip
spec:
  type: ClusterIP
  selector:
    app: mi-app
  ports:
    - port: 80
      targetPort: 8080
```

### 2. NodePort

Expone el servicio en una IP estática de cada nodo del clúster, accesible mediante `<NodeIP>:<nodePort>`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: servicio-nodeport
spec:
  type: NodePort
  selector:
    app: mi-app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30007
```

### 3. LoadBalancer

Proporciona una IP externa que balancea carga a los pods. Requiere integración con un proveedor de nube.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: servicio-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: mi-app
  ports:
    - port: 80
      targetPort: 8080
```

### 4. ExternalName

Mapea un servicio a un nombre DNS externo.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: servicio-externalname
spec:
  type: ExternalName
  externalName: ejemplo.com
```

---

## 🌍 Exponer Servicios vía Ingress

Ingress permite acceder a servicios internos a través de reglas HTTP/HTTPS. Se requiere un **Ingress Controller** desplegado en el clúster.

### 📦 Requisitos Previos

* Tener instalado un Ingress Controller (por ejemplo, NGINX Ingress Controller).
* Asegurarse que los servicios a exponer estén configurados correctamente (generalmente `ClusterIP`).

### 📄 Ejemplo de Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingreso-ejemplo
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: app.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: servicio-clusterip
                port:
                  number: 80
```

Este manifiesto permite acceder al servicio `servicio-clusterip` mediante la URL `http://app.local/`.

> 🔧 Si estás usando Minikube, puedes habilitar Ingress con:
>
> ```bash
> minikube addons enable ingress
> ```

---

## ✅ Consideraciones Finales

* Utiliza `ClusterIP` junto con Ingress para mantener la arquitectura limpia y controlada.
* `NodePort` y `LoadBalancer` son útiles en desarrollo o nubes públicas.
* Ingress permite aplicar reglas complejas de enrutamiento y seguridad.

Puedes agregar certificados TLS, autenticación básica y políticas de red usando anotaciones o configuraciones avanzadas del controlador Ingress.

## 🧭 Conclusión

- `Service` permite exponer Pods de forma estable.
- `Ingress` permite enrutar tráfico HTTP de forma avanzada.
- Para entornos locales como Minikube, `NodePort` es útil. Para producción, se prefiere `Ingress` + balanceadores.



