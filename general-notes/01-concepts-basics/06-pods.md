# 🧱 Kubernetes - Pods

## 📌 ¿Qué es un Pod?

Un **Pod** es la unidad más pequeña desplegable en Kubernetes. Representa uno o más contenedores que comparten:

* El mismo espacio de red (IP y puerto)
* El mismo almacenamiento (volúmenes)
* El mismo ciclo de vida

> 🚀 **Analogía**: Un Pod es como una habitación que puede tener una o varias personas (contenedores) que comparten el mismo aire (red) y algunos muebles (almacenamiento).

## 🧠 Características Clave

| Característica | Descripción                                                       |
| -------------- | ----------------------------------------------------------------- |
| Un solo Pod    | Puede contener uno o varios contenedores. El uso común es uno.    |
| IP única       | Cada Pod recibe una IP interna única en el clúster.               |
| Ciclo de vida  | Se recrea cuando falla (mediante un controlador como Deployment). |
| Compartición   | Contenedores en un Pod comparten volúmenes y red.                 |

---

## 🛠️ Comandos Básicos

### Crear un Pod de forma imperativa

Para crear un Pod directamente desde la línea de comandos sin necesidad de un archivo YAML:

```bash
kubectl run nginx-nodeport --image=nginx --restart=Never --port=80
# Para colocar el namespace
kubectl run nginx-nodeport --image=nginx --restart=Never --port=80 --namespace=desarrollo

```

> ⚠️ Este método es útil solo para pruebas rápidas o entornos de desarrollo. No se recomienda en producción.

---

### Crear Pod desde archivo YAML (declarativo)

```bash
kubectl apply -f pod.yaml
```

---

### Ver Pods en el namespace actual

```bash
kubectl get pods
```

### Ver Pods en un namespace particular

```bash
kubectl get pods -n <nombre-namespace>
```

### Ver Pods en todos los namespaces

```bash
kubectl get pods --all-namespaces
```

### Ver detalles de un Pod

```bash
kubectl describe pod <nombre>

# Ejemplo:
kubectl describe pod nginx-pod
```
> Incluye información sobre eventos, imagen, puertos, volúmenes, etc.

### Eliminar un Pod

```bash
kubectl delete pod <nombre>
kubectl delete pod <nombre> --namespace=<namespace>
```

---

## 📄 Ejemplo YAML de un Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mi-primer-pod
  labels:
    app: demo
spec:
  containers:
    - name: nginx-container
      image: nginx:1.25
      ports:
        - containerPort: 80
```

---

## 🧪 Inspección y depuración

### Ver logs del contenedor

```bash
kubectl logs <nombre-del-pod>
#Ejemplo:
kubectl logs nginx-pod
```

### Acceder al shell del contenedor

```bash
kubectl exec -it <nombre-del-pod> -- /bin/bash
```

### Ver Pods con su IP y nodo

```bash
kubectl get pods -o wide
kubectl get pods -o wide --namespace=desarrollo
```

### Descargar imagen manualmente en Minikube (modo offline)
Esto permite evitar problemas de acceso a internet o restricciones corporativas:

```bash
docker pull nginx
minikube image load nginx
```
Luego se puede usar el Pod normalmente:

```bash
kubectl run nginx --image=nginx --restart=Never
```
---

## 📦 Pods vs Otros Recursos

| Recurso         | Descripción                                               |
| --------------- | --------------------------------------------------------- |
| **Pod**         | Unidad básica de ejecución de contenedores.               |
| **ReplicaSet**  | Asegura que cierto número de Pods estén corriendo.        |
| **Deployment**  | Gestiona actualizaciones declarativas de Pods.            |
| **StatefulSet** | Ideal para cargas de trabajo con identidad estable (DBs). |

---

## 📚 Buenas prácticas

* **No crear Pods directamente** en producción. Usa **Deployments** o **StatefulSets**.
* Define recursos (`resources.limits` y `requests`) para evitar sobrecarga de nodos.
* Nombra tus Pods con lógica de negocio (`web-api`, `auth-svc`, etc.).
* Usa `labels` para agrupar y seleccionar Pods fácilmente.
