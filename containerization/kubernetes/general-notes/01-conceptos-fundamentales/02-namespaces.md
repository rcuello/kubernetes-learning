# 🌌 Namespaces: Organizando tu Clúster de Kubernetes

> Los **Namespaces** son una herramienta fundamental en Kubernetes para dividir tu clúster en múltiples "espacios virtuales" o áreas lógicas. Te permiten organizar y aislar recursos, facilitando la gestión en entornos complejos o multiusuario.

-----

## 🧠 ¿Qué es un Namespace y para qué sirve?

Imagina tu clúster de Kubernetes como un gran edificio de oficinas. Sin una organización, todos los equipos pondrían sus escritorios y equipos en cualquier lugar, lo que llevaría al caos. Los **Namespaces** son como los diferentes pisos o departamentos del edificio.

Cada Namespace proporciona un ámbito único para los nombres de los recursos. Esto significa que puedes tener un Pod llamado `mi-app` en el Namespace `desarrollo` y otro Pod también llamado `mi-app` en el Namespace `produccion` sin que haya conflictos.

Los Namespaces son cruciales para:

  * **Aislamiento Lógico:** Agrupan y separan recursos como Pods, Services, Deployments, Secrets y ConfigMaps. Esto evita colisiones de nombres y mejora la claridad.
  * **Organización:** Permiten estructurar tu clúster por equipos, entornos (desarrollo, pruebas, producción), aplicaciones o incluso por clientes.
  * **Control de Acceso (RBAC):** Puedes definir políticas de **Control de Acceso Basado en Roles (RBAC)** que otorgan permisos específicos a usuarios o equipos solo dentro de un Namespace particular. Esto mejora la seguridad al limitar el alcance de las acciones.
  * **Límites de Recursos:** Puedes establecer **cuotas de recursos (`ResourceQuotas`)** para limitar la cantidad total de CPU y memoria que los Pods pueden consumir dentro de un Namespace, lo que ayuda a prevenir que una aplicación acapare los recursos de todo el clúster.

-----

## 🏛️ Namespaces por Defecto en Kubernetes

Cuando instalas un clúster de Kubernetes, vienen preconfigurados con algunos Namespaces esenciales para su funcionamiento:

  * `default`: Es el Namespace donde se crean los recursos si no especificas uno. Para entornos de desarrollo o pruebas sencillas, puede ser suficiente, pero no se recomienda para producción.
  * `kube-system`: Contiene todos los recursos y componentes que forman parte del propio sistema de control de Kubernetes (por ejemplo, el scheduler, el controller-manager, CoreDNS, kube-proxy). Es fundamental no modificar recursos en este Namespace a menos que sepas exactamente lo que estás haciendo.
  * `kube-public`: Este Namespace es de lectura para todos los usuarios (incluso los no autenticados). Se utiliza para recursos que deben ser accesibles públicamente, como un `ConfigMap` con la información del clúster.
  * `kube-node-lease`: Se utiliza para almacenar objetos `Lease` (arrendamientos) para cada nodo. Los nodos actualizan estos objetos periódicamente para indicar su estado de "salud" al plano de control, lo que mejora el rendimiento y la escalabilidad de la detección de fallos de los nodos.

-----

## 🛠️ Comandos Útiles con `kubectl` para Namespaces

Manejar Namespaces con `kubectl` es muy sencillo:

1.  **Ver todos los Namespaces existentes:**

    ```bash
    kubectl get namespaces
    # O su forma abreviada:
    kubectl get ns
    ```

2.  **Crear un nuevo Namespace:**

    ```bash
    kubectl create namespace mi-equipo-desarrollo
    # O la forma abreviada:
    kubectl create ns mi-equipo-desarrollo
    ```

3.  **Especificar un Namespace para un comando (temporalmente):**
    Cuando ejecutas un comando `kubectl`, por defecto opera en el Namespace configurado en tu contexto actual (a menudo `default`). Puedes usar la bandera `-n` o `--namespace` para apuntar a un Namespace diferente:

    ```bash
    kubectl get pods -n mi-equipo-desarrollo  # Muestra los Pods en 'mi-equipo-desarrollo'
    kubectl create deployment mi-app --image=nginx -n mi-equipo-desarrollo # Crea un Deployment en ese Namespace
    ```

4.  **Establecer un Namespace por defecto en tu contexto actual:**
    Si trabajas mucho en un Namespace específico, puedes configurarlo como predeterminado para no tener que usar `-n` en cada comando.

    ```bash
    # Primero, asegúrate de saber qué contexto estás usando:
    kubectl config current-context

    # Luego, establece el Namespace por defecto para ese contexto:
    kubectl config set-context --current --namespace=mi-equipo-desarrollo
    ```

    Ahora, cualquier comando `kubectl` que ejecutes (como `kubectl get pods`) operará en `mi-equipo-desarrollo` a menos que lo sobrescribas con `-n`.

5.  **Eliminar un Namespace:**

    ```bash
    kubectl delete namespace mi-equipo-desarrollo
    ```

    > ⚠️ **¡Precaución\!** Eliminar un Namespace **eliminará automáticamente todos los recursos (Pods, Services, Deployments, etc.)** que contiene. ¡Úsalo con extremo cuidado, especialmente en entornos de producción\!

-----

## 📄 Ejemplo YAML para Crear un Namespace

Puedes crear un Namespace de forma declarativa con un archivo YAML:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: qa-testing # Nombre del nuevo Namespace
```

Guarda este contenido como `qa-namespace.yaml` y aplícalo:

```bash
kubectl apply -f qa-namespace.yaml
```

-----

## 💡 Buenas Prácticas con Namespaces

  * **Organiza por Propósito:** Utiliza Namespaces para separar lógicamente tus cargas de trabajo, ya sea por entorno (`dev`, `staging`, `prod`), por equipo (`equipo-frontend`, `equipo-backend`), o por aplicación.
  * **Define Cuotas de Recursos (`ResourceQuotas`):** Implementa cuotas de CPU y memoria a nivel de Namespace para evitar que una aplicación consuma demasiados recursos y afecte a otras.
  * **Controla el Acceso con RBAC:** Define Roles y RoleBindings para limitar las acciones de los usuarios y ServiceAccounts a Namespaces específicos. Esto es vital para la seguridad en entornos multiusuario.
  * **Aísla el Tráfico con `NetworkPolicies`:** Si necesitas un aislamiento de red más estricto entre Namespaces (o dentro de ellos), usa `NetworkPolicies` para controlar qué Pods pueden comunicarse entre sí.
  * **Sé Explícito:** Siempre que sea posible, especifica el Namespace al crear o interactuar con recursos (usando `-n`). Esto ayuda a evitar errores y asegura que los recursos se desplieguen donde deben ir.
  * **Etiquetas y Anotaciones Complementarias:** Aunque los Namespaces ya organizan, puedes usar **Labels** en tus recursos dentro de los Namespaces para una organización aún más granular, y **Annotations** para metadatos adicionales útiles para herramientas o documentación.

-----