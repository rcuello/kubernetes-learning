# 🌐 Configurar Ingress Controller en Minikube

Este documento explica cómo habilitar y verificar el funcionamiento del Ingress Controller en un entorno local con Minikube. Está diseñado como una guía independiente, pero también puede ser referenciada desde otros documentos.

---

## 📥 Requisitos Previos

- Tener instalado [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- Tener instalado [kubectl](https://kubernetes.io/docs/tasks/tools/)
- Haber inicializado tu clúster con `minikube start`

---

## 🔌 Habilitar Ingress Controller en Minikube

Minikube proporciona un addon que permite activar rápidamente un NGINX Ingress Controller:

```bash
minikube addons enable ingress
````

---

## 🔍 Verificar que el Ingress Controller esté activo

Una vez habilitado el addon, Minikube instalará el controlador **NGINX Ingress** en el namespace `ingress-nginx`. Puedes verificar que los pods estén funcionando correctamente con:

```bash
kubectl get pods -n ingress-nginx
```

Deberías ver algo similar a:

```bash
NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-ldlpd       0/1     Completed   0          14m
ingress-nginx-admission-patch-89f59        0/1     Completed   0          14m
ingress-nginx-controller-67c5cb88f-lnd7r   1/1     Running     0          14m
```

Asegúrate de que el pod del controlador esté en estado `Running`. Si ves un estado como `CrashLoopBackOff`, algo salió mal con la instalación o configuración.

También puedes verificar los **servicios expuestos** por el Ingress Controller con:

```bash
kubectl get svc -n ingress-nginx
```

Ejemplo de salida:

```bash
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.96.122.242   <none>        80:32344/TCP,443:32614/TCP   15m
ingress-nginx-controller-admission   ClusterIP   10.100.77.104   <none>        443/TCP                      15m
```

> El servicio `ingress-nginx-controller` es el que expone los puertos 80 y 443. En entornos locales como Minikube, puedes obtener la IP pública con `minikube ip` y acceder a tus rutas ingresadas desde `http://<minikube-ip>:80`.

---

## 🌐 Exponer Servicios con Ingress

Una vez activo, puedes comenzar a definir recursos de tipo `Ingress`. Aquí tienes un ejemplo mínimo para exponer un servicio:

### 📄 `ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: demo.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: demo-service
            port:
              number: 80
```

Aplica el recurso:

```bash
kubectl apply -f ingress.yaml
```

---

## 🧪 Probar Ingress desde tu máquina

### 1. Obtener IP del nodo Minikube:

```bash
minikube ip
```

### 2. Agregar entrada en tu archivo `hosts`

Edita tu archivo `/etc/hosts` (Linux/Mac) o `C:\Windows\System32\drivers\etc\hosts` (Windows):

```
<MINIKUBE_IP>   demo.local
```

Ejemplo:

```
192.168.49.2    demo.local
```

---

## 🚧 Problemas comunes

* ❗ El pod de `ingress-nginx-controller` no arranca: revisa con `kubectl describe pod ...` y valida recursos de red.
* ❗ Ingress no enruta correctamente: asegúrate de que el servicio backend esté expuesto en el puerto correcto y que el path coincida.
* ❗ 404 Not Found: valida que tu host (`demo.local`) esté correctamente mapeado y que el Ingress esté aplicado.

---

## 📚 Recursos adicionales

* [Documentación oficial de Minikube Ingress](https://minikube.sigs.k8s.io/docs/handbook/ingress/)
* [NGINX Ingress Controller GitHub](https://github.com/kubernetes/ingress-nginx)

---

## 🧼 Limpieza (opcional)

```bash
kubectl delete ingress demo-ingress
minikube addons disable ingress
```
