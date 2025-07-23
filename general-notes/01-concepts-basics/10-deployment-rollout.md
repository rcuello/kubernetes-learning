# 🚀 Actualización de imagen en un Deployment (`kubectl set image`)

## 🎯 Objetivo

Actualizar la imagen de un `Deployment` en Kubernetes utilizando el comando `kubectl set image`, e ilustrar cómo Kubernetes maneja el proceso de **rollout** cuando se usa una imagen **inexistente** (para demostrar el control del estado).

---

## 📦 Prerrequisitos

Antes de ejecutar este comando, necesitas un `Deployment` existente. Supongamos que tienes el siguiente `Deployment` llamado `hello-deployment` que ejecuta una imagen válida:

```bash
kubectl create deployment hello-deployment --image=gcr.io/google-samples/hello-app:1.0
```

---

## 🔁 Comando de actualización (rollout)

Puedes actualizar la imagen del `Deployment` con:

```bash
kubectl set image deployment/hello-deployment hello-app=gcr.io/google-samples/hello-app:3.0
```

Este comando indica:

* **`deployment/hello-deployment`**: Nombre del deployment.
* **`hello-app=...`**: Contenedor llamado `hello-app` dentro del pod que usará la nueva imagen.
* **`gcr.io/google-samples/hello-app:3.0`**: Imagen objetivo (en este caso **inexistente**, lo cual forzará un fallo controlado del rollout).

---

## 📉 Resultado esperado (imagen inexistente)

El `Deployment` intentará iniciar nuevos pods con la imagen `3.0`, pero fallará porque no existe. Puedes monitorear el rollout con:

```bash
kubectl rollout status deployment/hello-deployment
```

Ejemplo de salida esperada:

```
Waiting for deployment "hello-deployment" rollout to finish: 1 out of 1 new replicas have been updated...
error: deployment "hello-deployment" exceeded its progress deadline
```

Verifica los eventos o errores en los pods:

```bash
kubectl get pods
kubectl describe pod <nombre-del-pod>
```

---

## 🧠 Comportamiento de Kubernetes

Cuando una imagen no se puede obtener:

* El rollout entra en estado **progresando** hasta que falla por timeout (`progressDeadlineSeconds`).
* Se mantiene la especificación del deployment con la imagen fallida.
* No se eliminan los pods anteriores si aún están disponibles (por `maxUnavailable`).

---

## 🛑 Reversión del cambio

Puedes deshacer el último rollout con:

```bash
kubectl rollout undo deployment/hello-deployment
```

Esto restaurará el deployment a la versión anterior (`1.0`).

---

## ✅ Prueba con una imagen válida

Una vez entendido el flujo de error, prueba de nuevo con una imagen existente:

```bash
kubectl set image deployment/hello-deployment hello-app=gcr.io/google-samples/hello-app:1.0
```
