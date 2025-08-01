# 🧪 Prompt para Generar Laboratorios de Kubernetes

## 📋 Plantilla del Prompt

```
Genera un laboratorio práctico de Kubernetes sobre [TEMA]. Sigue esta estructura específica:

**ESTRUCTURA OBLIGATORIA:**

1. **🚫 Sección "El Problema"** (10-15% del contenido)
   - Muestra PRIMERO qué sucede cuando NO usas [TEMA] correctamente
   - Incluye un manifiesto YAML problemático con comentarios # ❌ 
   - Demuestra los fallos/limitaciones con comandos kubectl
   - Explica por qué no funciona

2. **✅ Sección "La Solución"** (60-70% del contenido)
   - Presenta el manifiesto YAML correcto con [TEMA]
   - Incluye comentarios explicativos # 🎯 en partes clave
   - Comandos paso a paso con salidas esperadas
   - Múltiples escenarios de prueba para demostrar características

3. **📊 Sección "Casos Prácticos"** (15-20% del contenido)
   - Al menos 3 ejemplos del mundo real
   - Comandos de verificación y validación
   - Diferentes configuraciones para casos específicos

4. **🧹 Sección "Limpieza"** (5% del contenido)
   - Comandos para limpiar todos los recursos creados

**REQUISITOS DE FORMATO:**

- Usa emojis para secciones (🚫, ✅, 📊, 🔍, etc.)
- Incluye bloques de código con sintaxis highlighting
- Agrega comentarios ❌ para partes problemáticas
- Agrega comentarios ✅ para ventajas/resultados esperados
- Incluye tablas comparativas cuando sea relevante
- Salidas esperadas después de comandos importantes

**CARACTERÍSTICAS OBLIGATORIAS:**

- **Progresivo**: De simple a complejo
- **Educativo**: Explica el "por qué", no solo el "cómo"
- **Práctico**: Comandos ejecutables reales
- **Realista**: Casos de uso del mundo real
- **Completo**: Desde despliegue hasta limpieza
- **Verificable**: Comandos para validar que funciona

**TONO Y ESTILO:**

- Dirigido a estudiantes/principiantes
- Explicaciones claras sin jerga excesiva
- Incluye tips y warnings importantes
- Formato consistente con laboratorios previos

**TEMA ESPECÍFICO:** [TEMA]

**CONTEXTO ADICIONAL:** [CONTEXTO_OPCIONAL]
```

---

## 🎯 Ejemplos de Uso del Prompt

### Para Secrets:
```
Genera un laboratorio práctico de Kubernetes sobre **Secrets**. [Usar estructura completa...]

**TEMA ESPECÍFICO:** Secrets
**CONTEXTO ADICIONAL:** Enfócate en diferentes tipos de secrets (generic, tls, docker-registry) y cómo las aplicaciones los consumen de forma segura vs. insegura.
```

### Para ConfigMaps:
```
Genera un laboratorio práctico de Kubernetes sobre **ConfigMaps**. [Usar estructura completa...]

**TEMA ESPECÍFICO:** ConfigMaps  
**CONTEXTO ADICIONAL:** Muestra la diferencia entre hardcodear configuración vs. usar ConfigMaps, incluyendo archivos de configuración completos y variables de entorno.
```

### Para Services:
```
Genera un laboratorio práctico de Kubernetes sobre **Services**. [Usar estructura completa...]

**TEMA ESPECÍFICO:** Services (ClusterIP, NodePort, LoadBalancer)
**CONTEXTO ADICIONAL:** Demuestra cómo exponer aplicaciones internamente y externamente, incluyendo problemas de conectividad sin Services.
```

### Para Ingress:
```
Genera un laboratorio práctico de Kubernetes sobre **Ingress**. [Usar estructura completa...]

**TEMA ESPECÍFICO:** Ingress Controllers y reglas de enrutamiento
**CONTEXTO ADICIONAL:** Muestra routing basado en host y path, SSL/TLS, y cómo manejar múltiples aplicaciones con un solo punto de entrada.
```

### Para Resource Limits:
```
Genera un laboratorio práctico de Kubernetes sobre **Resource Limits y Requests**. [Usar estructura completa...]

**TEMA ESPECÍFICO:** Resource Management (CPU, Memory limits/requests)
**CONTEXTO ADICIONAL:** Demuestra qué pasa sin límites (pods consumiendo todos los recursos) vs. con límites apropiados, incluyendo OOMKilled scenarios.
```

### Para RBAC:
```
Genera un laboratorio práctico de Kubernetes sobre **RBAC**. [Usar estructura completa...]

**TEMA ESPECÍFICO:** Role-Based Access Control
**CONTEXTO ADICIONAL:** Muestra acceso sin restricciones vs. usuarios con permisos específicos, incluyendo ServiceAccounts, Roles y RoleBindings.
```

### Para Jobs y CronJobs:
```
Genera un laboratorio práctico de Kubernetes sobre **Jobs y CronJobs**. [Usar estructura completa...]

**TEMA ESPECÍFICO:** Batch workloads (Jobs, CronJobs)
**CONTEXTO ADICIONAL:** Contrasta con Deployments para tareas que deben ejecutarse una vez o de forma programada, incluyendo backup jobs, data processing, etc.
```

### Para Networking:
```
Genera un laboratorio práctico de Kubernetes sobre **Network Policies**. [Usar estructura completa...]

**TEMA ESPECÍFICO:** Network Policies para microsegmentación
**CONTEXTO ADICIONAL:** Muestra comunicación sin restricciones vs. políticas de red específicas, incluyendo ingress/egress rules.
```

### Para Storage:
```
Genera un laboratorio práctico de Kubernetes sobre **Storage Classes y Dynamic Provisioning**. [Usar estructura completa...]

**TEMA ESPECÍFICO:** Storage Classes y provisioning automático
**CONTEXTO ADICIONAL:** Contrasta volumes manuales vs. provisioning dinámico, incluyendo diferentes tipos de storage (SSD, HDD, network storage).
```

### Para Helm:
```
Genera un laboratorio práctico de Kubernetes sobre **Helm Charts**. [Usar estructura completa...]

**TEMA ESPECÍFICO:** Package management con Helm
**CONTEXTO ADICIONAL:** Muestra despliegues manuales vs. Helm charts, incluyendo templating, values, y gestión de releases.
```

### Para Observabilidad:
```
Genera un laboratorio práctico de Kubernetes sobre **Monitoring con Prometheus**. [Usar estructura completa...]

**TEMA ESPECÍFICO:** Observabilidad y métricas
**CONTEXTO ADICIONAL:** Contrasta aplicaciones sin monitoreo vs. con métricas, alertas y dashboards completos.
```

### Para Seguridad:
```
Genera un laboratorio práctico de Kubernetes sobre **Pod Security Standards**. [Usar estructura completa...]

**TEMA ESPECÍFICO:** Pod Security (SecurityContext, PodSecurityPolicy/Standards)
**CONTEXTO ADICIONAL:** Muestra pods inseguros (root, privileged) vs. pods con security context apropiado.
```

---

## 📚 Lista de Temas Sugeridos para Laboratorios

### **Nivel Básico:**
- [ ] ConfigMaps y Secrets
- [ ] Services (ClusterIP, NodePort, LoadBalancer)
- [ ] Ingress y Ingress Controllers
- [ ] Resource Limits y Requests
- [ ] Namespaces y aislamiento
- [ ] Labels y Selectors
- [ ] Health Checks (Liveness/Readiness Probes)

### **Nivel Intermedio:**
- [ ] Jobs y CronJobs
- [ ] RBAC (Roles, RoleBindings, ServiceAccounts)
- [ ] Network Policies
- [ ] Storage Classes y Dynamic Provisioning
- [ ] Horizontal Pod Autoscaler (HPA)
- [ ] Pod Disruption Budgets
- [ ] Taints y Tolerations
- [ ] Node Affinity y Pod Affinity

### **Nivel Avanzado:**
- [ ] Custom Resource Definitions (CRDs)
- [ ] Operators y Controllers
- [ ] Admission Controllers
- [ ] Pod Security Standards
- [ ] Service Mesh (Istio basics)
- [ ] GitOps con ArgoCD/Flux
- [ ] Monitoring (Prometheus + Grafana)
- [ ] Logging (ELK/EFK stack)
- [ ] Backup y Disaster Recovery

### **Nivel DevOps:**
- [ ] Helm Charts avanzados
- [ ] CI/CD pipelines con Kubernetes
- [ ] Multi-cluster management
- [ ] Cluster autoscaling
- [ ] Performance tuning
- [ ] Troubleshooting avanzado
- [ ] Security scanning y compliance
- [ ] Cost optimization

---

## 💡 Tips para Usar el Prompt

1. **Personaliza el contexto** - Agrega información específica sobre tu entorno (minikube, cloud provider, etc.)

2. **Combina temas** - Puedes generar laboratorios que combinen múltiples conceptos:
   ```
   **TEMA ESPECÍFICO:** ConfigMaps + Secrets + Security Context
   **CONTEXTO ADICIONAL:** Muestra una aplicación web completa con configuración externa y credenciales seguras.
   ```

3. **Especifica el nivel** - Indica si quieres nivel básico, intermedio o avanzado

4. **Incluye herramientas específicas** - Menciona si quieres usar herramientas específicas (Helm, Kustomize, etc.)

5. **Casos de uso específicos** - Puedes especificar industria o tipo de aplicación (web apps, microservices, ML workloads, etc.)

---

## 🚀 Prompt Optimizado para Casos Específicos

```
Genera un laboratorio práctico de Kubernetes sobre **[TEMA]** para **[NIVEL]**. 

**ESCENARIO:** [DESCRIPCIÓN_DEL_ESCENARIO]
**APLICACIÓN:** [TIPO_DE_APP] 
**ENTORNO:** [MINIKUBE/EKS/GKE/AKS]
**HERRAMIENTAS:** [KUBECTL/HELM/KUSTOMIZE]

[Incluir estructura completa del prompt base...]

**ENFOQUE ESPECIAL:** [ASPECTOS_ESPECÍFICOS_A_CUBRIR]
```

¡Con este prompt podrás generar laboratorios consistentes y educativos para cualquier tema de Kubernetes que quieras aprender! 🎯