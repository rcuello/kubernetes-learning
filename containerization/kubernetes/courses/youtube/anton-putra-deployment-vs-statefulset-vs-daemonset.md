# Kubernetes: Deployment vs StatefulSet vs DaemonSet
*Guía completa para entender los diferentes tipos de workloads en Kubernetes*

## Conceptos Fundamentales

### ¿Qué es un Pod?
- **Definición**: El pod es la unidad más pequeña de computación y replicación en Kubernetes
- **Escalamiento**: Cuando una aplicación necesita escalar, simplemente se incrementa el número de pods
- **Limitación importante**: Todos los contenedores en un pod se escalan juntos, sin importar sus necesidades individuales
- **Mejor práctica**: Mantener los pods lo más pequeños posible, típicamente conteniendo solo:
  - Un proceso principal
  - Contenedores auxiliares estrechamente relacionados (sidecars)

---

## 1. Kubernetes Deployment

### ¿Qué es un Deployment?
Un Deployment es una abstracción de nivel superior que gestiona pods de manera declarativa. Su propósito principal es **declarar cuántas réplicas de un pod deben estar ejecutándose en cualquier momento**.

### Características Principales
- **Gestión automática**: Crea automáticamente el número solicitado de pods y los monitorea
- **Auto-recuperación**: Si un pod falla, el deployment lo recrea automáticamente
- **Estado declarativo**: Solo necesitas declarar el estado deseado del sistema
- **Pods efímeros**: Los pods no tienen identidad persistente y obtienen IDs únicos

### Exposición al Exterior
- Se utiliza un **Service** para exponer la aplicación
- Las peticiones se redirigen aleatoriamente a cualquiera de los pods
- **No es posible** dirigirse a un pod específico individualmente

### Persistencia de Datos en Deployments

#### Problema Principal
Los pods son efímeros por naturaleza. Cuando un pod se reprograma o elimina, **pierde todo el estado y datos almacenados localmente**.

#### Solución: Persistent Volume Claims (PVC)
```yaml
# Ejemplo conceptual
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-storage
spec:
  accessModes:
    - ReadWriteOnce  # Solo un pod puede montarlo
  resources:
    requests:
      storage: 10Gi
```

#### Limitaciones con ReadWriteOnce
- **Un solo pod**: Solo se puede adjuntar el volumen a un pod
- **Múltiples réplicas fallan**: Otros pods no pueden montar el volumen
- **Problemas en actualizaciones**: El pod nuevo intenta montar mientras el anterior aún lo tiene

#### Soluciones y Workarounds

**Estrategia Recreate:**
```yaml
spec:
  strategy:
    type: Recreate  # Destruye el pod antes de crear uno nuevo
```

**ReadWriteMany con almacenamiento de red:**
- Ejemplo: [Elastic File System (EFS)](https://aws.amazon.com/es/efs/) en AWS
- Permite montar el mismo volumen en múltiples pods
- **Problema**: Todos los pods comparten el mismo volumen (generalmente no deseado)

**Múltiples Deployments:**
- Crear tantos PVs como pods necesites
- Crear deployments separados con una réplica cada uno
- **Problema**: Difícil de escalar y gestionar

---

## 2. Kubernetes StatefulSet

### ¿Qué es un StatefulSet?
Un StatefulSet es similar a un Deployment, pero está diseñado específicamente para **aplicaciones que requieren identidad persistente y almacenamiento estable**.

### Características Clave

#### Volume Claim Templates
```yaml
# Ejemplo conceptual
spec:
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```
- **Creación dinámica**: Crea automáticamente PVCs para cada pod
- **Vinculación persistente**: Cada pod se vincula a un volumen específico
- **Persistencia**: Si el pod se reprograma, mantiene el mismo volumen

#### Identidad de Red Estable
- **DNS estable**: Cada pod obtiene un nombre DNS único y estable
- **Formato predecible**: `<pod-name>.<service-name>.<namespace>.svc.cluster.local`
- **Casos de uso**: Aplicaciones como Kafka que necesitan conocer la ubicación de líderes de partición

#### Registros DNS
- **Registros A**: Para resolución de nombres básica
- **Registros SRV**: Para casos de uso avanzados con clientes especializados

### Casos de Uso Comunes
- **Bases de datos**: MySQL, PostgreSQL, MongoDB
- **Sistemas de mensajería**: Kafka, RabbitMQ
- **Almacenamiento distribuido**: Cassandra, ElasticSearch
- **Aplicaciones stateless con identidad fuerte**: Cuando necesitas identidad persistente

### Estrategias de Despliegue

#### Despliegue Secuencial (Por Defecto)
- Los pods se crean uno por uno en orden
- Más lento pero más controlado

#### Despliegue Paralelo
```yaml
spec:
  podManagementPolicy: Parallel
```
- Todos los pods se crean simultáneamente
- Más rápido para casos que no requieren orden específico

---

## 3. Kubernetes DaemonSet

### ¿Qué es un DaemonSet?
Un DaemonSet asegura que **todos (o algunos) nodos ejecuten una copia de un pod específico**. Es ideal para servicios que deben ejecutarse en cada nodo del cluster.

### Características Principales
- **Un pod por nodo**: Automáticamente crea tantos pods como nodos hay en el cluster
- **Escalamiento automático**: Al agregar/quitar nodos, automáticamente agrega/quita pods
- **No gestión de réplicas**: No necesitas especificar el número de réplicas

### Casos de Uso Comunes

#### Monitoreo y Logging
- **Agentes de monitoreo**: Prometheus Node Exporter, Datadog Agent
- **Recolectores de logs**: Fluentd, Fluent Bit, Filebeat
- **Observabilidad**: Jaeger agents, APM agents

#### Almacenamiento Distribuido
- **Hadoop**: Nodos de datos distribuidos
- **MinIO**: Almacenamiento de objetos distribuido
- **Rook**: Orquestador de almacenamiento cloud-native
- **Local Persistent Volumes**: Abstracción de discos locales

### Consideraciones Especiales

#### Toleraciones (Taints)
```yaml
# Ejemplo conceptual
spec:
  template:
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
```
- **Problema**: Los nodos pueden tener taints que previenen el scheduling
- **Solución**: Agregar tolerations para todos los posibles taints

#### Casos de Uso de Almacenamiento
- **Nodos con discos adjuntos**: Crear nodos Kubernetes con discos locales
- **Abstracción de almacenamiento**: Usar DaemonSet para exponer discos locales como S3 o filesystem
- **Distribución de datos**: Proporcionar almacenamiento distribuido para aplicaciones

---

## Comparación Rápida

| Aspecto | Deployment | StatefulSet | DaemonSet |
|---------|------------|-------------|-----------|
| **Identidad de pods** | Efímera, intercambiable | Persistente, única | Una por nodo |
| **Almacenamiento** | Compartido o sin estado | Persistente por pod | Típicamente local |
| **Escalamiento** | Manual/automático por réplicas | Manual/automático por réplicas | Automático por nodos |
| **Orden de creación** | Paralelo | Secuencial (configurable) | Paralelo |
| **Casos de uso** | Apps stateless, microservicios | Bases de datos, apps stateful | Agentes, almacenamiento distribuido |
| **DNS** | No garantizado | Estable y predecible | Por nodo |
| **Actualizaciones** | Rolling update | Rolling update ordenado | Rolling update por nodo |

---

## Cuándo Usar Cada Uno

### Usa **Deployment** cuando:
- Tu aplicación es stateless
- Los pods son intercambiables
- No necesitas almacenamiento persistente por pod
- Quieres escalamiento simple y rápido
- Ejemplos: APIs REST, microservicios web, aplicaciones frontend

### Usa **StatefulSet** cuando:
- Tu aplicación requiere identidad persistente
- Necesitas almacenamiento persistente por pod
- Los pods no son intercambiables
- Requieres nombres DNS estables
- Ejemplos: Bases de datos, sistemas de colas, aplicaciones con estado

### Usa **DaemonSet** cuando:
- Necesitas ejecutar algo en cada nodo
- El servicio debe estar presente en todo el cluster
- No controlas el número de réplicas (depende de los nodos)
- Ejemplos: Agentes de monitoreo, recolectores de logs, drivers de red

---

## Consejos Prácticos

1. **Comienza simple**: Usa Deployments para la mayoría de aplicaciones web
2. **Evalúa el estado**: Si tu app guarda datos críticos, considera StatefulSet
3. **Monitoreo es clave**: Siempre usa DaemonSet para agentes de monitoreo
4. **Prueba la persistencia**: Verifica que los datos sobrevivan a reinicios de pods
5. **Considera los costos**: StatefulSets con almacenamiento pueden ser más costosos
6. **Planifica el escalamiento**: DaemonSets escalan con los nodos, no con la carga

---

*Esta guía está basada en el video de [Anton Putra](https://www.youtube.com/watch?v=30KAInyvY_o&ab_channel=AntonPutra) sobre Kubernetes workloads.*