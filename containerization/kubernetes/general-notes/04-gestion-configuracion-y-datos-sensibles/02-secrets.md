# 🔐 ¿Qué es un Secret en Kubernetes?

Un **Secret** es un recurso que permite almacenar información **sensible** como:

* Contraseñas
* Tokens
* Claves API
* Certificados

> 🛡️ **Analogía**: Es como una caja fuerte dentro del clúster. Se accede igual que un ConfigMap, pero con protección adicional.

---

## 📦 ¿Cómo se ve un Secret? (Declarativo)

```yaml
# ecommerce-secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: ecommerce-secrets
type: Opaque
data:
  db_user: YWRtaW4=           # admin
  db_pass: c3VwZXJzZWNyZXQ=   # supersecret
  jwt_secret: c2VjdXJlanNvbjEyMw==   # securejson123
  stripe_key: c3RyaXBlX2tleV9saXZl   # stripe_key_live
```

> ⚠️ Los valores deben estar codificados en **Base64**, pero **esto no es cifrado**, solo ofuscación.

### ✅ Crear de forma declarativa

```bash
kubectl apply -f ecommerce-secrets.yaml
```

---

## ⚙️ Crear Secret desde línea de comandos (Imperativo)

```bash
kubectl create secret generic ecommerce-secrets \
  --from-literal=db_user=admin \
  --from-literal=db_pass=supersecret \
  --from-literal=jwt_secret=securejson123 \
  --from-literal=stripe_key=stripe_key_live
```

> 🧪 Para ver un valor específico (decodificado):

```bash
kubectl get secret ecommerce-secrets -o jsonpath="{.data.db_pass}" | base64 --decode
```

---

Excelente idea. Aquí tienes la sección actualizada agregando el uso de `kubectl edit secret` para modificar un Secret directamente desde el editor:

---

## ✏️ Editar un Secret en tiempo real

Puedes editar un Secret directamente desde tu editor (por defecto `vim`, o el que tengas configurado) usando:

```bash
kubectl edit secret ecommerce-secrets
```

Esto abrirá el recurso en formato YAML, pero **recuerda**: los valores en `data` están codificados en **Base64**, así que verás algo como esto:

```yaml
data:
  db_user: YWRtaW4=
  db_pass: c3VwZXJzZWNyZXQ=
```

> ✍️ Si editas manualmente los valores, asegúrate de codificarlos en Base64 antes de guardarlos.

### 👉 Ejemplo

Supongamos que quieres cambiar la contraseña `db_pass` de `supersecret` a `newpass123`. Primero conviértelo:

```bash
echo -n "newpass123" | base64
# Resultado: bmV3cGFzczEyMw==
```

Luego reemplaza el valor en el editor:

```yaml
data:
  db_pass: bmV3cGFzczEyMw==
```

Guarda y cierra. Kubernetes aplicará el cambio al Secret inmediatamente.
---

## 📊 Comparación rápida: ConfigMap vs Secret

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
✅ **No los subas a Git**, ni siquiera ofuscados.


