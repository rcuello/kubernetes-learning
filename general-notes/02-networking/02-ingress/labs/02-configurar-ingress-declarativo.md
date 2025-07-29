# 🧪 Laboratorio: Exponer Servicios con Ingress (Manifiesto Declarativo)

Este laboratorio demuestra cómo exponer una aplicación en Kubernetes usando un manifiesto **todo-en-uno** que incluye el `Deployment`, `Service` e `Ingress`. Ideal para aplicar buenas prácticas de despliegue y facilitar la automatización.

---

## 🎯 Objetivo

* Activar el Ingress Controller en Minikube.
* Desplegar una aplicación de ejemplo (`hello-app`).
* Exponerla mediante `Ingress` y un nombre de dominio local (`hello-world.example`).
* Acceder desde navegador o `curl`.

---

## 🧱 Prerrequisitos

* Tener instalado:
  * [Minikube](https://minikube.sigs.k8s.io/docs/start/)
  * [`kubectl`](https://kubernetes.io/docs/tasks/tools/)
* Tener permisos de administrador para editar el archivo `hosts`.

---

## ⚙️ Paso 1: Habilitar el Ingress Controller

Activa el addon NGINX incluido en Minikube:

```bash
minikube addons enable ingress
````

Verifica que esté corriendo:

```bash
kubectl get pods -n ingress-nginx
```

---

## 📄 Paso 2: Crear manifiesto único `hello-app-ingress.yaml`

Crea un archivo `hello-app-ingress.yaml` con el siguiente contenido:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: hello
          image: gcr.io/google-samples/hello-app:1.0
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: hello
  ports:
    - port: 8080
      targetPort: 8080
---
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
kubectl apply -f hello-app-ingress.yaml
```

---

## 🔍 Paso 3: Verificar recursos

Verifica que los recursos estén creados correctamente:

```bash
kubectl get deployments
kubectl get svc
kubectl get ingress
```

Ejemplo de salida esperada:

```
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
web               1/1     1            1           20s

NAME   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
web    ClusterIP   10.98.141.47    <none>        8080/TCP   20s

NAME              CLASS    HOSTS                 ADDRESS         PORTS   AGE
example-ingress   nginx    hello-world.example   192.168.49.2    80      20s
```

---

## 🌍 Paso 4: Obtener la IP de Minikube

```bash
minikube ip
```

Guarda esta IP para usarla en el siguiente paso (por ejemplo, `192.168.49.2`).

---

## 🗂️ Paso 5: Modificar archivo `hosts`

Edita el archivo `hosts`:

* **Windows**: `C:\Windows\System32\drivers\etc\hosts`
* **Linux/macOS**: `/etc/hosts`

Agrega esta línea:

```
127.0.0.1 hello-world.example
```

> ⚠️ Usa `127.0.0.1` si estás usando `minikube tunnel`, o la IP del clúster (`minikube ip`) si no usas túnel.

---

## ✅ Paso 6: Verificar resolución DNS local

Confirma que el nombre de dominio se resuelva correctamente:

```bash
ping hello-world.example
```

Salida esperada:

```
Haciendo ping a hello-world.example [127.0.0.1] con 32 bytes de datos:
Respuesta desde 127.0.0.1: bytes=32 tiempo<1ms TTL=128
...
```

---

## 🔌 Paso 7: Asegúrate de que el puerto 80 esté libre

Si estás en **Windows** y tienes IIS activo, deténlo:

```cmd
iisreset /stop
```

O desactívalo por completo:

```powershell
Stop-Service W3SVC
Set-Service W3SVC -StartupType Disabled
```

---

## 🚇 Paso 8: Ejecutar `minikube tunnel`

Ejecuta en una terminal separada:

```bash
minikube tunnel
```

Esto crea una redirección entre tu máquina local y los servicios `LoadBalancer` de Minikube.

---

## 🌐 Paso 9: Probar el acceso

Accede desde tu navegador o con `curl`:

```bash
curl http://hello-world.example
```

Respuesta esperada:

```
Hello, world!
Version: 1.0.0
Hostname: web-xxxxx
```

---

## 🧹 Limpieza (opcional)

```bash
kubectl delete -f hello-app-ingress.yaml
```

---

## 🧠 ¿Qué aprendiste?

* Cómo empaquetar `Deployment`, `Service` e `Ingress` en un solo manifiesto.
* Cómo activar el controlador Ingress en Minikube.
* Cómo simular nombres de dominio personalizados en entornos locales.
* Cómo usar `minikube tunnel` para exponer recursos desde tu clúster local.

---
