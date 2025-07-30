# 🚀 Ejemplo: Desplegar NGINX con Helm en Minikube

Este documento te guía paso a paso para desplegar un servidor **NGINX** en Kubernetes usando **Helm**, personalizando la instalación mediante el uso de un archivo `values.yaml`.

---

## 🧱 Paso 1: Agregar el repositorio Bitnami (si no lo tienes)

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

---

## 📂 Paso 2: Crear una carpeta de proyecto

```bash
mkdir nginx-chart-example && cd nginx-chart-example
```

---

## 📝 Paso 3: Crear un archivo `values.yaml`

Crea un archivo llamado `values.yaml` con la siguiente configuración personalizada:

```yaml
replicaCount: 2

image:
  registry: docker.io
  repository: bitnami/nginx
  tag: "1.25.3"
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80

ingress:
  enabled: false

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

> 💡 Esto define 2 réplicas de NGINX, usando NodePort para exponer el servicio, y asigna límites de recursos.

---

## 🚀 Paso 4: Instalar el chart usando `values.yaml`

```bash
helm install nginx-demo bitnami/nginx -f values.yaml
```

---

## 🔍 Paso 5: Verificar la instalación

```bash
helm list
kubectl get pods
kubectl get svc
```

---

## 🌐 Paso 6: Acceder a NGINX en el navegador

Si estás usando **Minikube**, expón el servicio con:

```bash
minikube service nginx-demo
```

Esto abrirá automáticamente tu navegador con la IP y puerto de acceso al servicio.

---

## 🧼 Paso 7: Desinstalar

Cuando termines, puedes limpiar tu clúster:

```bash
helm uninstall nginx-demo
```

---

## 📚 Recomendaciones adicionales

* Explora los valores que puedes personalizar:

  ```bash
  helm show values bitnami/nginx > default-values.yaml
  ```

* Documentación oficial del chart:
  [https://github.com/bitnami/charts/tree/main/bitnami/nginx](https://github.com/bitnami/charts/tree/main/bitnami/nginx)
