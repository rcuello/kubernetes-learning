# Kubernetes Namespaces

## ¿Qué es un Namespace en Kubernetes?

Un **Namespace** en Kubernetes es una forma de dividir un clúster en múltiples espacios lógicos, permitiendo que diferentes equipos o proyectos trabajen de forma aislada dentro del mismo clúster. Es útil en entornos multiusuario o donde se requieren entornos separados (producción, pruebas, desarrollo, etc.).

---

## Beneficios de usar Namespaces

* **Aislamiento lógico**: recursos como pods, servicios, secrets y configmaps se agrupan dentro de un namespace.
* **Organización**: útil para separar por equipos, entornos, aplicaciones.
* **Control de acceso**: se pueden definir políticas RBAC (Role-Based Access Control) específicas por namespace.
* **Límites de recursos**: se pueden definir quotas de CPU/memoria por namespace.

---

## Namespaces por defecto en Kubernetes

Cuando se crea un clúster de Kubernetes, se generan automáticamente los siguientes namespaces:

* `default`: usado si no se especifica otro.
* `kube-system`: contiene los recursos del sistema (DNS, scheduler, etc.).
* `kube-public`: accesible públicamente, útil para compartir info entre usuarios.
* `kube-node-lease`: usado para detectar el estado de los nodos.

---

## Comandos útiles con kubectl

### Ver los namespaces existentes

```bash
kubectl get namespaces
```

### Crear un namespace

```bash
kubectl create namespace desarrollo
```

### Usar un namespace temporalmente

```bash
kubectl get pods -n desarrollo
```

### Establecer un namespace por defecto en el contexto actual

```bash
kubectl config set-context --current --namespace=desarrollo
```

---

## Ejemplo YAML para crear un Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: desarrollo
```

Aplica con:

```bash
kubectl apply -f namespace-desarrollo.yaml
```

---

## Buenas prácticas

* Usa namespaces por entorno (`dev`, `staging`, `prod`).
* Establece cuotas (`ResourceQuota`) y límites (`LimitRange`).
* Usa `NetworkPolicies` para aislar el tráfico entre namespaces si es necesario.
* Aplica etiquetas (`labels`) para clasificar y buscar recursos fácilmente.

---
