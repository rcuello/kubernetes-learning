# 🧪 Laboratorio: Exponer servicios usando Ingress en Kubernetes

Este laboratorio muestra cómo instalar un **controlador Ingress**, configurar un recurso `Ingress` personalizado y acceder a una aplicación mediante nombre de dominio dentro de Minikube.

---

## 🎯 Objetivo

* Activar el controlador Ingress en Minikube.
* Desplegar una aplicación y exponerla vía Ingress.
* Acceder usando un nombre de host personalizado (`hello-world.example`).

---

## 🧱 Prerrequisitos

* Tener instalado:

  * [Minikube](https://minikube.sigs.k8s.io/docs/start/)
  * `kubectl` conectado al clúster local de Minikube.

---

## ⚙️ Paso 1: Crear el Deployment y el Service `web`

Vamos a desplegar una aplicación de ejemplo y exponerla como un servicio interno de Kubernetes.

### 🔍 Verifica si ya existe un servicio `web`

Antes de crear el nuevo servicio, asegúrate de que no haya uno existente con el mismo nombre:

```bash
kubectl get svc
kubectl get svc web
```

Si el servicio `web` ya existe, elimínalo para evitar conflictos:

```bash
kubectl delete svc web
```

> También puedes eliminar el `Deployment` asociado (opcional):
>
> ```bash
> kubectl delete deployment web
> ```

---

### 🚀 Crea el Deployment y el Service

1. Crea un `Deployment` que use una imagen de ejemplo proporcionada por Google:

```bash
kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
```

2. Expón el Deployment como un `Service` accesible dentro del clúster en el puerto 8080:

```bash
kubectl expose deployment web --port=8080
kubectl expose deployment web --type=NodePort --port=8080
```

---

### ✅ Verifica que el servicio esté activo

Consulta el estado del nuevo `Service`:

```bash
kubectl get svc web
```

Deberías obtener una salida similar a la siguiente:

```
NAME   TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
web    NodePort   10.98.141.47   <none>        8080:31604/TCP   11s
```

---


## 🚀 Paso 2: Activar el Ingress Controller en Minikube

Minikube incluye un **addon** para el controlador NGINX. Actívalo con:

```bash
minikube addons enable ingress
```

Verifica que el pod del controlador esté corriendo:

```bash
kubectl get pods -n ingress-nginx
```

---

## 📄 Paso 3: Crear el manifiesto Ingress

Crea un archivo llamado `example-ingress.yaml` con el siguiente contenido:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: hello-world.example
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 8080
```

Aplica el manifiesto:

```bash
kubectl apply -f example-ingress.yaml
```

Ejemplo de salida:

```text
ingress.networking.k8s.io/example-ingress created
```

---

## 🔎 Paso 4: Verifica los detalles del Ingress creado

Una vez desplegado el recurso `Ingress`, puedes inspeccionar sus detalles y verificar si ha sido correctamente configurado y sincronizado por el controlador NGINX:

```bash
kubectl describe ingress example-ingress
```

Una salida típica puede verse así:

```
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
Name:             example-ingress
Labels:           <none>
Namespace:        default
Address:          192.168.49.2
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host                 Path  Backends
  ----                 ----  --------
  hello-world.example
                       /     web:8080 (10.244.0.53:8080)
Annotations:           <none>
Events:
  Type    Reason  Age                    From                      Message
  ----    ------  ----                   ----                      -------
  Normal  Sync    5m14s (x2 over 5m29s)  nginx-ingress-controller  Scheduled for sync
```

### 🔍 ¿Qué revisar en esta salida?

* **Address**: Es la IP donde el Ingress está escuchando dentro del clúster (útil si estás usando Minikube).
* **Ingress Class**: Debe ser `nginx`, como se definió en el manifiesto.
* **Host y Path**: La URL `hello-world.example/` está direccionando correctamente al servicio `web` en el puerto `8080`.
* **Events**: El evento `Scheduled for sync` indica que el controlador NGINX ha procesado el Ingress.


---

## 🌐 Paso 5: Obtener la IP del Ingress Controller

Obtén la IP de Minikube:

```bash
minikube ip
```

Ejemplo de salida:

```
192.168.49.2
```

---

## 🗂️ Paso 6: Configurar el archivo `/etc/hosts`

Para que `hello-world.example` funcione, debes mapear el dominio a la IP de Minikube.

Agrega esta línea a tu archivo `/etc/hosts`:

```
192.168.49.2 hello-world.example
```

> En Windows, el archivo se encuentra en:
> `C:\Windows\System32\drivers\etc\hosts`

---

## 🌐 Paso 4: Exponer el Ingress hacia fuera del clúster con `minikube tunnel`

Para que puedas acceder a tu `Ingress` desde fuera del clúster en un entorno local como Minikube, necesitas iniciar un túnel. Esto redirige las peticiones externas a la IP del clúster a través de tu máquina local.

Ejecuta:

```bash
minikube tunnel
```

Deberías ver una salida como:

```
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

❗  Access to ports below 1024 may fail on Windows with OpenSSH clients older than v8.1. For more information, see: https://minikube.sigs.k8s.io/docs/handbook/accessing/#access-to-ports-1024-on-windows-requires-root-permission
🏃  Starting tunnel for service example-ingress.
```

### 🔒 Importante

* **Mantén esta terminal abierta**: Si la cierras, el túnel se cierra y no podrás acceder al Ingress desde tu navegador.
* **Permisos**: En algunos sistemas operativos (especialmente Windows), puede que se requieran permisos elevados para exponer puertos bajos (<1024).

---


## ✅ Paso 7: Probar el acceso

Abre tu navegador o usa `curl`:

```bash
curl http://hello-world.example
```

Deberías recibir:

```text
Hello, world!
Version: 1.0.0
Hostname: web-xxxx
```

---

## 🧹 Limpieza (opcional)

```bash
kubectl delete ingress example-ingress
kubectl delete svc web
kubectl delete deployment web
```

---

## 🧠 ¿Qué aprendiste?

* Cómo crear un servicio básico con Deployment + Service.
* Cómo activar y verificar un controlador Ingress con Minikube.
* Cómo configurar un recurso `Ingress` con `rules` por host.
* Cómo usar `/etc/hosts` para simular dominios en local.


