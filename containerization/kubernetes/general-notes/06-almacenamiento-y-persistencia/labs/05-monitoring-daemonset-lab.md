# 📦 Lab: DaemonSet - Desplegando Agentes de Monitoreo en Cada Nodo

En este laboratorio aprenderás cómo **DaemonSet** despliega automáticamente un pod en **cada nodo** del cluster. Simularemos el despliegue de agentes de monitoreo y logging, casos de uso típicos en el mundo real.

---

## 🚫 1. El problema: Deployment no garantiza cobertura de todos los nodos

Primero vamos a intentar desplegar un agente de monitoreo con un **Deployment** normal para ver por qué **NO es la solución correcta**.

### 📄 Manifiesto problemático (`monitoring-deployment-problema.yaml`)

```yaml
# ❌ ESTO NO GARANTIZA COBERTURA COMPLETA
apiVersion: apps/v1
kind: Deployment
metadata:
  name: monitoring-agent-deployment
spec:
  replicas: 2  # ❌ ¿Qué pasa si tienes 3 nodos? ¿O 1 nodo?
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: nginx:alpine  # Simulamos un agente de monitoreo
        command: ['/bin/sh']
        args: ['-c', 'while true; do echo "$(date): Monitoring node $(hostname)"; sleep 10; done']
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
```

### 🧪 1.1. Despliegue y observación del problema

```bash
# Despliega el Deployment problemático
kubectl apply -f monitoring-deployment-problema.yaml

# Verifica cuántos nodos tienes
kubectl get nodes

# Observa DÓNDE se desplegaron los pods
kubectl get pods -o wide -l app=monitoring-agent
```

> ⚠️ **Problemas observados:**
> - Los pods pueden estar en el **mismo nodo** (sin cobertura completa)
> - Si agregas un **nuevo nodo**, no se despliega automáticamente
> - Si reduces las **réplicas**, podrías perder cobertura de algunos nodos
> - **No hay garantía** de que cada nodo tenga un agente

```bash
# Simula agregar réplicas manualmente
kubectl scale deployment monitoring-agent-deployment --replicas=3

# Aún así, no hay garantía de distribución por nodo
kubectl get pods -o wide -l app=monitoring-agent
```

```bash
# Limpia este experimento fallido
kubectl delete -f monitoring-deployment-problema.yaml
```

---

## ✅ 2. La solución: DaemonSet para cobertura automática

Ahora despleguemos correctamente usando **DaemonSet** que garantiza **un pod por nodo automáticamente**.

### 📄 Archivo `monitoring-daemonset.yaml`

```yaml
# --- 1. DaemonSet de agente de monitoreo ---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-agent-daemonset
  labels:
    app: monitoring-agent
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: nginx:alpine
        command: ['/bin/sh']
        args: ['-c', 'while true; do echo "$(date): Monitoring node $(hostname) - Node: $NODE_NAME"; sleep 15; done']
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
        # 🎯 Acceso a métricas del nodo host
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
      # Volúmenes del host para monitoreo real
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      # 🔑 IMPORTANTE: Tolera taints para ejecutarse en todos los nodos
      tolerations:
      - operator: Exists
        effect: NoSchedule
      - operator: Exists
        effect: PreferNoSchedule
      - operator: Exists
        effect: NoExecute
```

---

## 🚀 3. Despliegue del DaemonSet

```bash
kubectl apply -f monitoring-daemonset.yaml
```

Salida esperada:
```
daemonset.apps/monitoring-agent-daemonset created
```

---

## 👀 4. Observa las características clave del DaemonSet

### 4.1. Un pod por nodo automáticamente

```bash
# Cuenta los nodos
kubectl get nodes
echo "Número de nodos: $(kubectl get nodes --no-headers | wc -l)"

# Cuenta los pods del DaemonSet
kubectl get pods -l app=monitoring-agent
echo "Número de pods: $(kubectl get pods -l app=monitoring-agent --no-headers | wc -l)"
```

> ✅ **Resultado:** El número de pods **siempre coincide** con el número de nodos.

### 4.2. Distribución garantizada

```bash
# Verifica que hay exactamente UN pod por nodo
kubectl get pods -o wide -l app=monitoring-agent

# Verifica los nombres de los nodos
kubectl get pods -l app=monitoring-agent -o jsonpath='{range .items[*]}{.spec.nodeName}{"\n"}{end}' | sort
```

> ✅ **Resultado:** Cada nodo tiene **exactamente un pod**, no más, no menos.

### 4.3. No hay réplicas manuales

```bash
# Intenta escalar el DaemonSet (no funcionará como Deployment)
kubectl scale daemonset monitoring-agent-daemonset --replicas=5

# Verifica - el número de pods sigue siendo igual al número de nodos
kubectl get pods -l app=monitoring-agent
```

> ✅ **Comportamiento:** DaemonSet **ignora el parámetro replicas** - siempre mantiene uno por nodo.

---

## 📊 5. Monitorea la actividad de los agentes

### 5.1. Observa los logs de diferentes nodos

```bash
# Lista todos los pods y sus nodos
kubectl get pods -l app=monitoring-agent -o custom-columns="POD:metadata.name,NODE:spec.nodeName"

# Ve los logs de cada pod (reemplaza con tu nombre real)
kubectl logs -f <pod-name-from-node-1>
```

Salida esperada:
```
Mon Dec  4 10:15:23 UTC 2023: Monitoring node monitoring-agent-daemonset-abc12 - Node: minikube
Mon Dec  4 10:15:38 UTC 2023: Monitoring node monitoring-agent-daemonset-abc12 - Node: minikube
```

---

## 🔄 6. Prueba el escalamiento automático de nodos

### 6.1. Simula agregar un nodo (solo con minikube multi-nodo)

Si estás usando minikube, puedes simular esto:

```bash
# Agregar un nodo adicional (si tu entorno lo soporta)
minikube node add

# Observa cómo se crea automáticamente un nuevo pod
kubectl get pods -l app=monitoring-agent -w
```

> ✅ **Comportamiento:** Automáticamente se despliega un nuevo pod en el nodo nuevo.

### 6.2. Simula eliminar un nodo

```bash
# Lista los nodos actuales
kubectl get nodes

# Si tienes múltiples nodos, elimina uno
minikube node delete <node-name>

# Observa cómo el pod se elimina automáticamente
kubectl get pods -l app=monitoring-agent
```

---

## 🛠️ 7. Caso de uso real: Agente de logs con acceso al filesystem

Vamos a desplegar un ejemplo más realista: un recolector de logs que necesita acceso a los logs del host.

### 📄 Archivo `log-collector-daemonset.yaml`

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector-daemonset
  labels:
    app: log-collector
spec:
  selector:
    matchLabels:
      app: log-collector
  template:
    metadata:
      labels:
        app: log-collector
    spec:
      containers:
      - name: log-collector
        image: alpine:latest
        command: ['/bin/sh']
        args: ['-c', 'while true; do echo "$(date): Collecting logs from node $NODE_NAME"; ls -la /var/log/host/ 2>/dev/null | head -5; sleep 20; done']
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        # 🎯 Monta los logs del host
        volumeMounts:
        - name: varlog
          mountPath: /var/log/host
          readOnly: true
        - name: containers
          mountPath: /var/lib/docker/containers
          readOnly: true
        resources:
          requests:
            memory: "32Mi"
            cpu: "25m"
          limits:
            memory: "64Mi"
            cpu: "50m"
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: containers
        hostPath:
          path: /var/lib/docker/containers
      tolerations:
      - operator: Exists
        effect: NoSchedule
      - operator: Exists
        effect: PreferNoSchedule
```

### 7.1. Despliega el recolector de logs

```bash
kubectl apply -f log-collector-daemonset.yaml

# Observa los pods
kubectl get pods -l app=log-collector

# Ve los logs para verificar el acceso al filesystem del host
kubectl logs -f <log-collector-pod-name>
```

---

## 🔍 8. Casos de uso del mundo real

### 8.1. Agente de monitoreo de sistema

```yaml
# Ejemplo conceptual - no ejecutar
spec:
  containers:
  - name: node-exporter  # Prometheus Node Exporter
    image: prom/node-exporter:latest
    ports:
    - containerPort: 9100
    volumeMounts:
    - name: proc
      mountPath: /host/proc
    - name: sys  
      mountPath: /host/sys
```

### 8.2. Agente de seguridad

```yaml
# Ejemplo conceptual - no ejecutar  
spec:
  containers:
  - name: security-agent
    image: falcosecurity/falco:latest
    volumeMounts:
    - name: dev
      mountPath: /host/dev
    - name: proc
      mountPath: /host/proc
```

### 8.3. Driver de red

```yaml
# Ejemplo conceptual - no ejecutar
spec:
  containers:
  - name: network-driver
    image: calico/node:latest
    env:
    - name: NODENAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
```

---

## 📈 9. Verifica el comportamiento con taints

### 9.1. Agrega un taint a un nodo

```bash
# Agrega un taint que normalmente evitaría scheduling
kubectl taint nodes <node-name> special=monitoring:NoSchedule

# Verifica que el DaemonSet sigue funcionando (gracias a las tolerations)
kubectl get pods -l app=monitoring-agent -o wide
```

### 9.2. Quita el taint

```bash
# Remueve el taint
kubectl taint nodes <node-name> special=monitoring:NoSchedule-

# El comportamiento no cambia - el pod ya estaba ahí
kubectl get pods -l app=monitoring-agent -o wide
```

---

## 🧹 10. Limpieza

```bash
# Elimina ambos DaemonSets
kubectl delete -f monitoring-daemonset.yaml
kubectl delete -f log-collector-daemonset.yaml

# Verifica que todos los pods se eliminaron automáticamente
kubectl get pods -l app=monitoring-agent
kubectl get pods -l app=log-collector
```

---

## 📋 11. Comparación: Deployment vs DaemonSet

| Aspecto | Deployment | DaemonSet |
|---------|------------|-----------|
| **Réplicas** | Configurable manualmente | Automático = número de nodos |
| **Distribución** | Aleatoria | Garantizada (uno por nodo) |
| **Escalamiento** | Manual/HPA | Automático con nodos |
| **Scheduling** | Cualquier nodo disponible | Todos los nodos (con tolerations) |
| **Uso ideal** | Apps de negocio | Agentes de sistema |
| **Tolerations** | Opcional | Casi siempre necesario |

---

## 🎯 12. Casos de uso reales de DaemonSet

### **Monitoreo y Observabilidad:**
- **Prometheus Node Exporter** - Métricas de sistema
- **Datadog Agent** - APM y métricas
- **New Relic Agent** - Monitoreo de aplicaciones

### **Logging:**
- **Fluentd** - Recolección de logs
- **Fluent Bit** - Recolector ligero de logs  
- **Filebeat** - Envío de logs a Elasticsearch

### **Seguridad:**
- **Falco** - Runtime security monitoring
- **Twistlock** - Container security
- **Antivirus agents** - Protección en tiempo real

### **Networking:**
- **Calico** - CNI networking
- **Flannel** - Overlay network
- **Cilium** - eBPF-based networking

### **Storage:**
- **Local Persistent Volumes** - Gestión de almacenamiento local
- **Rook agents** - Distributed storage
- **Longhorn** - Cloud-native storage

---

## ✅ ¿Qué aprendiste?

* **DaemonSet** despliega automáticamente **un pod por nodo**
* **No necesitas gestionar réplicas** - se ajusta automáticamente al número de nodos
* **Escalamiento automático** cuando se agregan/quitan nodos del cluster
* Las **tolerations son cruciales** para ejecutarse en todos los nodos
* **Acceso al host** mediante `hostPath` volumes para monitoreo real
* Es **ideal para agentes de sistema** que deben estar presentes en cada nodo
* **Casos de uso reales**: monitoreo, logging, seguridad, networking y storage

> 🎯 **Regla de oro:** Si tu aplicación debe ejecutarse en **cada nodo** del cluster para funcionar correctamente, usa **DaemonSet**. Si solo necesitas N réplicas distribuidas, usa **Deployment**.