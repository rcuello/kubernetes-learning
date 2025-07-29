# 🧩 ¿Qué son los ConfigMap y los Secret en Kubernetes?

En Kubernetes, **ConfigMaps** y **Secrets** son objetos que permiten separar la configuración de una aplicación del código fuente de la misma. Son fundamentales para mantener una arquitectura limpia, segura y fácilmente adaptable a diferentes entornos (desarrollo, staging, producción).

---

## 🔧 ¿Qué es un ConfigMap?

Un **ConfigMap** es un recurso que almacena **pares clave-valor** de configuración no confidencial.

> 🧠 **Analogía**: Es como un archivo `.env` o de configuración `.properties` que puedes montar en tu aplicación desde fuera del contenedor.

### 📦 ¿Qué tipo de datos almacena?

- URLs
- Flags de comportamiento (`DEBUG=true`)
- Rutas internas
- Cadenas de conexión (sin credenciales)
- Cualquier dato de configuración no sensible

### 🧪 Ejemplo de ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: debug
  API_URL: https://api.midominio.com
````

### 🧩 Cómo usarlo en un Pod

```yaml
env:
  - name: LOG_LEVEL
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: LOG_LEVEL
```

---

## 🔐 ¿Qué es un Secret?

Un **Secret** es un recurso similar a un ConfigMap, pero **destinado a almacenar información sensible**, como:

* Contraseñas
* Tokens
* Claves API
* Certificados

> 🛡️ **Analogía**: Es como una caja fuerte dentro del clúster. Se accede igual que un ConfigMap, pero con protección adicional.

### 📦 ¿Cómo se ve un Secret?

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=    # base64 de "admin"
  password: c2VjcmV0    # base64 de "secret"
```

> ⚠️ Los valores deben estar codificados en **Base64**, pero **esto no es cifrado**, solo ofuscación. En producción se recomienda integrar soluciones como **Vault**, **SealedSecrets** o el cifrado nativo de etcd.

---

## 🔁 Diferencias entre ConfigMap y Secret

| Característica      | ConfigMap                 | Secret                        |
| ------------------- | ------------------------- | ----------------------------- |
| Uso principal       | Configuración no sensible | Información sensible          |
| Codificación Base64 | ❌ No requerida            | ✅ Obligatoria                 |
| Nivel de seguridad  | Bajo                      | Alto (uso restringido, RBAC)  |
| Visibilidad         | Clara                     | Protegida en el Dashboard/API |

---

## 💡 Mejores prácticas

✅ **No mezcles** datos sensibles con ConfigMaps.
✅ Usa Secrets incluso para tokens temporales o headers.
✅ Usa `kubectl create secret` con flags para evitar codificar manualmente en base64.
✅ Implementa rotación de Secrets con herramientas como **External Secrets Operator** o **Vault**.
✅ No los subas a Git, ni siquiera ofuscados.

---

## ⚙️ Comandos útiles

```bash
# Crear ConfigMap desde archivo .env
kubectl create configmap app-config --from-env-file=app.env

# Crear Secret desde línea de comandos
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secret

# Ver contenido decodificado (⚠️ solo para pruebas)
kubectl get secret db-secret -o jsonpath="{.data.password}" | base64 --decode
```

---

## 🧭 ¿Qué sigue?

Ahora que conoces cómo separar la configuración (y secretos) del código, el siguiente paso es entender cómo persistir información en Kubernetes con **volúmenes y storage classes**.

📄 [Siguiente: 04-volumenes-persistentes.md →](./04-volumenes-persistentes.md)

