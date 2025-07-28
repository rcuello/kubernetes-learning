# 📡 Qué es un Service en Kubernetes

## 📘 Definición general

Un **Service** en Kubernetes es un recurso que proporciona una forma **estable y confiable de acceder a uno o más Pods** que ejecutan una aplicación. Aunque los Pods son efímeros y cambian de IP constantemente, el Service actúa como un **puente estable** que garantiza la conectividad sin importar esos cambios.

> 🔄 **Analogía urbana**: Piensa en los Pods como locales dentro de una ciudad (el clúster). Cada local puede cerrar o cambiar de dirección, pero tú usas un número de teléfono central (el Service) para contactarlos sin saber su ubicación exacta. El Service se encarga de redirigir las llamadas al local disponible.

---

## 🧠 ¿Qué hace un Service?

- Asocia una dirección IP fija (ClusterIP o externa) y un nombre DNS a un conjunto dinámico de Pods.
- Realiza **balanceo de carga** entre los Pods disponibles.
- Usa etiquetas (`labels`) para seleccionar dinámicamente los Pods a los que se conecta.
- Permite comunicación **interna o externa** al clúster, según su tipo.


---

## 🧬 Tipos de Service

| Tipo             | Uso principal                                         | Alcance         |
|------------------|-------------------------------------------------------|------------------|
| `ClusterIP`      | Acceso interno al clúster                             | Interno          |
| `NodePort`       | Expone el servicio en un puerto de cada nodo         | Interno / Externo |
| `LoadBalancer`   | Usa un balanceador de carga externo (como AWS ELB)   | Externo          |
| `ExternalName`   | Redirige a un nombre DNS externo                      | Externo          |

### 📦 1. ClusterIP (por defecto)
- Expone el Service solo dentro del clúster.
- Ideal para comunicación entre microservicios.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mi-servicio
spec:
  selector:
    app: mi-app
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
````

---

### 📦 2. NodePort

* Asigna un puerto en cada nodo (por ejemplo, el `:30080`) que redirige al Service.
* Permite acceso externo simple, pero poco flexible.

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

> 🧠 Úsalo con precaución: requiere que conozcas la IP del nodo del clúster.

---

### 📦 3. LoadBalancer

* Solo funciona en proveedores de nube compatibles (AWS, Azure, GCP).
* Crea un balanceador externo que direcciona al NodePort interno.

```yaml
spec:
  type: LoadBalancer
```

---

### 📦 4. ExternalName

* No crea redirecciones dentro del clúster, solo traduce a un nombre DNS externo.

```yaml
spec:
  type: ExternalName
  externalName: api.misitio.com
```

---

## ⚙️ ¿Cómo selecciona Pods un Service?

Usa **label selectors** para identificar a qué Pods dirigir el tráfico:

```yaml
selector:
  app: mi-app
```

Todos los Pods que tengan `labels: { app: mi-app }` serán parte del grupo balanceado por el Service.

---

## 🎯 Casos de uso comunes

* Servicios internos para comunicación entre microservicios.
* Exposición de APIs REST a través de LoadBalancer.
* Acceso externo para pruebas o desarrollo.
* Servicios tipo DB que solo deberían estar disponibles dentro del clúster.

---

## 📌 Recomendaciones

* Usa `ClusterIP` siempre que sea posible para mantener la seguridad interna.
* Usa `LoadBalancer` con controladores de Ingress cuando tu infraestructura lo permita.
* Evita `NodePort` en producción salvo en entornos controlados.
* Configura `readinessProbes` en los Pods para evitar que Services balanceen a instancias que aún no están listas.

---

