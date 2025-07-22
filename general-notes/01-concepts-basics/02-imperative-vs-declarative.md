# 🎯 Declaración Imperativa vs Declarativa en Kubernetes

Cuando trabajamos con Kubernetes, existen dos enfoques principales para interactuar con los recursos del clúster: **imperativo** y **declarativo**. Ambos métodos tienen ventajas y limitaciones según el contexto, y es fundamental comprender las diferencias para elegir el más adecuado en cada caso.

---

## 🧱 1. ¿Qué es el enfoque Imperativo?

El enfoque **imperativo** se basa en emitir comandos que le dicen al sistema **cómo** lograr un estado deseado de forma inmediata. Es más procedural.

### ✅ Características:

* Se utiliza `kubectl` directamente desde línea de comandos.
* Los cambios son inmediatos.
* No se requiere mantener archivos de configuración.
* Más rápido para tareas puntuales o exploratorias.

### 🧪 Ejemplo:

```bash
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=3
kubectl delete pod nginx-abc123
```

Este estilo es muy útil para entornos de desarrollo y pruebas rápidas.

---

## 📄 2. ¿Qué es el enfoque Declarativo?

El enfoque **declarativo** se centra en describir **el estado deseado del sistema**, dejando que Kubernetes determine cómo lograrlo.

### ✅ Características:

* Se usa `kubectl apply -f archivo.yaml`.
* Es ideal para entornos productivos y GitOps.
* Permite control de versiones y trazabilidad.
* Facilita la automatización y la idempotencia.

### 🧪 Ejemplo:

Archivo `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

Comando para aplicar:

```bash
kubectl apply -f deployment.yaml
```

---

## 📊 3. Comparación Directa

| Característica          | Imperativo                  | Declarativo                        |
| ----------------------- | --------------------------- | ---------------------------------- |
| Enfoque                 | Procedural (cómo hacer)     | Declarativo (qué queremos)         |
| Uso                     | Comandos puntuales          | Archivos YAML mantenibles          |
| Reproducibilidad        | Limitada                    | Alta (infraestructura como código) |
| Control de cambios      | Manual                      | Versionable (ej. Git)              |
| Idempotencia            | No garantizada              | Garantizada                        |
| Escalabilidad operativa | Limitada                    | Alta                               |
| Ideal para              | Desarrollo, pruebas rápidas | Producción, CI/CD, GitOps          |

---

## 🔧 4. ¿Cuál usar?

| Contexto                   | Recomendado |
| -------------------------- | ----------- |
| Prototipado rápido         | Imperativo  |
| Automatización CI/CD       | Declarativo |
| Tareas administrativas     | Imperativo  |
| Gestión de clústeres/infra | Declarativo |
| Trabajar en equipo         | Declarativo |

---

## 🧠 Recomendación práctica

> ✅ Usa enfoque **imperativo** para tareas puntuales o exploratorias.
> ✅ Usa enfoque **declarativo** para definir recursos persistentes, gestionables y trazables en el tiempo (infraestructura como código).

En general, Kubernetes está diseñado para ser usado de forma declarativa, alineado con prácticas modernas como GitOps, donde los archivos de configuración viven en repositorios y los entornos se sincronizan automáticamente.

---

## 📚 Referencias

* [Kubernetes Official Docs: Declarative Management](https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/#declarative-object-configuration)
* [Imperative vs Declarative – ArgoCD & GitOps](https://argo-cd.readthedocs.io/en/stable/user-guide/declarative-setup/)
