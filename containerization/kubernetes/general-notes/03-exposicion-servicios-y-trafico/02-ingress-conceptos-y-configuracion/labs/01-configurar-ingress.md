# 🧪 Laboratorio: Exponer Servicios con Ingress en Kubernetes (Minikube)

Este laboratorio te guía para instalar un **controlador Ingress**, desplegar una aplicación de ejemplo y acceder a ella mediante un **nombre de dominio personalizado** dentro de un clúster local con **Minikube**.

---

## 🎯 Objetivo

* Activar el Ingress Controller NGINX en Minikube.
* Desplegar una aplicación expuesta como servicio.
* Configurar un recurso `Ingress` para exponerla vía dominio personalizado (`hello-world.example`).
* Acceder desde navegador o `curl`.

---

## 🧱 Prerrequisitos

Tener instalado:

* [Minikube](https://minikube.sigs.k8s.io/docs/start/)
* [`kubectl`](https://kubernetes.io/docs/tasks/tools/) configurado con el clúster local de Minikube.
* Editor de texto con permisos para modificar el archivo `hosts` (`/etc/hosts` en Linux/macOS o `C:\Windows\System32\drivers\etc\hosts` en Windows).

---

## ⚙️ Paso 1: Crear el Deployment y Service `web`

Desplegamos una pequeña aplicación de ejemplo y la exponemos mediante un `Service`.

### 🔍 Verifica si ya existe un recurso llamado `web`

```bash
kubectl get svc
kubectl get deployment web
```

Si ya existe, elimínalo para evitar conflictos:

```bash
kubectl delete svc web
kubectl delete deployment web
```

### 🚀 Crea el Deployment y el Service

```bash
kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
kubectl expose deployment web --port=8080 --target-port=8080
```

### ✅ Verifica el estado del Service

```bash
kubectl get svc web
```

Deberías ver algo similar:

```
NAME   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
web    ClusterIP   10.98.141.47   <none>        8080/TCP   10s
```

> ⚠️ Asegúrate de que el tipo de servicio sea `ClusterIP`. No uses `NodePort` si vas a trabajar con Ingress.

---

## 📥 Paso 2: Habilitar el Ingress Controller

Minikube incluye un addon de NGINX para actuar como controlador Ingress:

```bash
minikube addons enable ingress
```

Verifica que los pods estén activos:

```bash
kubectl get pods -n ingress-nginx
```

---

## 📄 Paso 3: Crear el recurso Ingress

Crea un archivo llamado `example-ingress.yaml` con este contenido:

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

---

## 🔎 Paso 4: Verificar el Ingress

Revisa los detalles del recurso creado:

```bash
kubectl describe ingress example-ingress
```

Verifica los siguientes puntos:

* `Address`: debe mostrar la IP del Ingress (generalmente la IP de Minikube).
* `Host`: debe coincidir con `hello-world.example`.
* `Backends`: debe mostrar el servicio `web:8080`.
* `Events`: confirma que el Ingress ha sido sincronizado.

---

## 🌍 Paso 5: Obtener la IP de Minikube

```bash
minikube ip
```

Ejemplo:

```
192.168.49.2
```

---

## 🗂️ Paso 6: Modificar el archivo `hosts`

Edita tu archivo `hosts` para que el dominio personalizado apunte a la IP de Minikube.

Agrega la línea:

```
127.0.0.1 hello-world.example
```

En Windows, el archivo está en:

```
C:\Windows\System32\drivers\etc\hosts
```

> ⚠️ Necesitas permisos de administrador para modificar este archivo.

---

### ✅ Verifica resolución de DNS local

Usa el siguiente comando para verificar:

```bash
ping hello-world.example
```

Ejemplo de salida esperada:

```bash
Haciendo ping a hello-world.example [127.0.0.1] con 32 bytes de datos:
Respuesta desde 127.0.0.1: bytes=32 tiempo<1m TTL=128
Respuesta desde 127.0.0.1: bytes=32 tiempo<1m TTL=128
Respuesta desde 127.0.0.1: bytes=32 tiempo<1m TTL=128
Respuesta desde 127.0.0.1: bytes=32 tiempo<1m TTL=128
```

Si obtienes respuestas similares, tu dominio local está correctamente configurado y redirigiendo a tu servicio expuesto por `minikube`.

---

## 🚧 Paso 7: Verifica que el puerto 80 esté libre

Si estás en **Windows**, asegúrate de que **IIS (Internet Information Services)** no esté usando el puerto 80.

### 🛑 Detener IIS temporalmente

Desde CMD como administrador:

```cmd
iisreset /stop
```

### ❌ O deshabilitar IIS completamente (opcional):

```powershell
Stop-Service W3SVC
Set-Service W3SVC -StartupType Disabled
```

> Puedes verificar quién está usando el puerto 80 con:
>
> ```cmd
> netstat -ano | findstr :80
> ```

---

## 🚇 Paso 8: Iniciar `minikube tunnel`

Este paso expone los servicios `LoadBalancer` del clúster a tu red local, permitiendo que el Ingress funcione correctamente:

```bash
minikube tunnel
```

Deberías ver:

```
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...
```

> 🔐 **Mantén esta consola abierta** mientras haces pruebas con el Ingress.

---

## 🧪 Paso 9: Probar el acceso

Abre tu navegador y visita:

```
http://hello-world.example
```

O usa `curl`:

```bash
curl http://hello-world.example
```

Respuesta esperada:

```
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

* Cómo desplegar una aplicación y exponerla mediante Ingress.
* Cómo configurar Minikube y el controlador NGINX.
* Cómo simular nombres de dominio en local modificando el archivo `hosts`.
* Cómo usar `minikube tunnel` para exponer servicios desde el clúster.

