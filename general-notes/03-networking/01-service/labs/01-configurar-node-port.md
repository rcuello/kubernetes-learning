# 🧪 Laboratorio: Exposición de una aplicación con `NodePort` en Kubernetes

Este laboratorio muestra cómo desplegar una aplicación sencilla y exponerla al exterior utilizando un **Service** del tipo `NodePort`.

---

## 🎯 Objetivo

* Desplegar una aplicación simple tipo "Hello, World".
* Exponerla mediante un servicio accesible desde fuera del clúster.

---

## 📦 Paso 1: Desplegar la aplicación

Usaremos una imagen de ejemplo proporcionada por Google:

```bash
kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
```

Verifica que el deployment se haya creado correctamente:

```bash
kubectl get deployments
```

Ejemplo de salida:

```text
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    1/1     1            1           86s
```

---

## 🌐 Paso 2: Exponer el deployment usando `NodePort`

Vamos a crear un **Service** de tipo `NodePort` que nos permita acceder a la aplicación desde fuera del clúster (por la IP de un nodo y un puerto abierto en el rango 30000–32767):

```bash
kubectl expose deployment web --type=NodePort --port=8080
```

Salida esperada:

```text
service/web exposed
```

---

## 🔍 Paso 3: Verificar el servicio creado

Listamos los servicios para identificar el puerto asignado por Kubernetes:

```bash
kubectl get svc
```

Ejemplo de salida:

```text
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          6d
web          NodePort    10.111.44.184   <none>        8080:31317/TCP   71s
```

### 📌 Interpretación:

* `8080`: Puerto del servicio (interno al clúster).
* `31317`: Puerto `NodePort` accesible desde fuera (asignado automáticamente).
* Puedes acceder a la app usando:

```text
http://<IP-del-nodo>:31317
```

---

## 🖥️ Acceso desde Minikube

Si estás trabajando con Minikube, puedes usar el siguiente comando para obtener la URL pública del servicio:

```bash
minikube service web --url
```

Ejemplo de salida:

```bash
$ minikube service web --url
http://127.0.0.1:61560
❗  Because you are using a Docker driver on windows, the terminal needs to be open to run it.
```

### ⚠️ ¿Qué significa esta salida?

Cuando usas Minikube con el **driver Docker**, el clúster de Kubernetes corre dentro de un contenedor y no expone puertos directamente al host. Para resolver esto, `minikube service` crea **un túnel temporal** que mapea el puerto del clúster a un puerto disponible en tu máquina local, como `127.0.0.1:61560`.

* Esa URL solo estará disponible **mientras la terminal donde se ejecutó el comando esté abierta**.
* Si cierras la terminal, el túnel se cierra y la aplicación dejará de ser accesible en ese puerto.

### 🛠️ Alternativas más permanentes

Si necesitas que el servicio esté siempre disponible:

* Usar el flag `--background`:

  ```bash
  minikube tunnel --background
  ```
* O cambiar el driver de Minikube a uno que soporte interfaces puenteadas, como VirtualBox o Hyper-V.

---

## ✅ Resultado esperado

Al acceder a la URL pública del servicio, deberías ver el mensaje:

```text
Hello, world!
Version: 1.0.0
Hostname: web-xxxxx
```

---

## 🧹 Limpieza (opcional)

Si deseas eliminar lo creado:

```bash
kubectl delete svc web
kubectl delete deployment web
```

