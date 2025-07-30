# 🧩 ¿Qué es un ConfigMap en Kubernetes?

En Kubernetes, un **ConfigMap** es un recurso que permite almacenar **pares clave-valor** de configuración **no confidencial**, separados del código fuente.

> 🧠 **Analogía**: Es como un archivo `.env` o `.properties` que puedes montar en tu aplicación desde fuera del contenedor.

---

## 📦 ¿Qué tipo de datos almacena?

* URLs
* Flags de comportamiento (`DEBUG=true`)
* Rutas internas
* Cadenas de conexión (sin credenciales)
* Cualquier dato de configuración no sensible

---

## 🧪 Ejemplo de ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: debug
  API_URL: https://api.midominio.com
```

---

## 🛒 Ejemplo avanzado: ConfigMap orientado a un ecommerce

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ecommerce-config
data:
  APP_NAME: "my-ecommerce-app"
  ENVIRONMENT: "development"
  MAX_CONNECTIONS: "100"
  DEBUG_MODE: "true"
  SUPPORTED_LANGUAGES: "es,en,pt"

  FEATURE_FLAGS: |
    {
      "enableDiscounts": true,
      "multiCurrencySupport": true,
      "betaCheckoutFlow": false,
      "availablePromos": ["welcome10", "summer25"]
    }

  LOG_PATH: "/var/log/ecommerce"
  TIMEOUT_SECONDS: "30"
  PRODUCT_API_URL: "https://api.example.com/products"
  PAYMENT_GATEWAY_URL: "https://payments.example.com"

  taxes.properties: |
    country.default=CO
    tax.rate=0.19
    tax.inclusive=false

  ui.properties: |
    theme=modern
    show.out.of.stock=true
    currency.symbol=$
    default.language=es

  email.properties: |
    smtp.server=smtp.example.com
    smtp.port=587
    smtp.user=noreply@example.com
    smtp.tls=true
```

---

## 🧩 Cómo usarlo en un Pod

```yaml
env:
  - name: LOG_LEVEL
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: LOG_LEVEL
```

O como archivo montado:

```yaml
volumes:
  - name: config-volume
    configMap:
      name: ecommerce-config

volumeMounts:
  - name: config-volume
    mountPath: /etc/config
```

---

## ⚙️ Comandos útiles

```bash
# Crear ConfigMap desde archivo .env
kubectl create configmap app-config --from-env-file=app.env

# Crear ConfigMap desde manifiesto YAML (declarativo)
kubectl apply -f ecommerce-config.yaml

```

```bash
# Ver configuración del ConfigMap 
kubectl describe configmap app-config
```

---

## 🧭 ¿Qué sigue?

Para manejar datos sensibles, consulta el archivo [`secrets.md`](./secrets.md).

