# 🔌 `kubectl port-forward`

El comando `kubectl port-forward` permite redirigir puertos desde tu máquina local hacia un pod o recurso de Kubernetes como un Deployment o Service. Es útil para acceder a servicios que corren dentro del clúster sin necesidad de exponerlos externamente.

---

## 🧪 Ejemplo práctico: `hello-deployment`

Supongamos que tienes un Deployment llamado `hello-deployment` que ejecuta una aplicación en el puerto `8080`.

Puedes redirigir el puerto del pod hacia tu máquina local usando:

```bash
kubectl port-forward deploy/hello-deployment 8080:8080
```

Esto hará que cualquier tráfico dirigido a `localhost:8080` se reenvíe al puerto `8080` del pod que esté siendo gestionado por el Deployment `hello-deployment`.

---

## 🔍 Prueba del reenvío

Una vez ejecutado el comando anterior, abre tu navegador y visita:

```
http://localhost:8080
```

Deberías ver una respuesta como:

```
Hello, world!
Version: 1.0.0
Hostname: hello-deployment-577b47bd4c-n95qj
```

---

## 🧠 Notas importantes

* Si hay múltiples pods para el deployment, `kubectl` elegirá uno aleatoriamente para redirigir el tráfico.
* El puerto del contenedor debe coincidir con el segundo número (`8080:8080` → contenedor escucha en `8080`).
* Este comando es ideal para pruebas locales sin exponer servicios al exterior con un `LoadBalancer` o `NodePort`.

---

## 📄 Definición del Deployment `hello-deployment`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deployment
  labels:
    app: hello
spec:
  replicas: 4
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello-app
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
```

Este manifiesto define un Deployment que ejecuta 4 réplicas del contenedor `hello-app` basado en la imagen `gcr.io/google-samples/hello-app:1.0`, exponiendo el puerto `8080` para recibir peticiones.

```

