# 📝 Ejemplo: Prompt para Laboratorio de Deployments

## 🎯 Prompt Específico para Deployments

```
Genera un laboratorio práctico de Kubernetes sobre **Deployments**. Sigue esta estructura específica:

**ESTRUCTURA OBLIGATORIA:**

1. **🚫 Sección "El Problema"** (10-15% del contenido)
   - Muestra PRIMERO qué sucede cuando NO usas Deployments correctamente
   - Incluye un manifiesto YAML problemático con comentarios # ❌ 
   - Demuestra los fallos/limitaciones con comandos kubectl
   - Explica por qué no funciona

2. **✅ Sección "La Solución"** (60-70% del contenido)
   - Presenta el manifiesto YAML correcto con Deployments
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

**TEMA ESPECÍFICO:** Deployments vs Pods directos

**CONTEXTO ADICIONAL:** 
- Entorno: Minikube local
- Nivel: Básico (principiantes)
- Enfócate en: 
  * Por qué NO crear pods directamente
  * Ventajas de self-healing y scaling
  * Rolling updates básicos
  * Rollbacks simples
  * Comparación directa Pod vs Deployment
- Aplicaciones: Usar aplicaciones web simples (nginx, apache)
- Incluir comandos específicos de minikube cuando sea relevante
- Mostrar cómo ver el dashboard de minikube para visualización
- Incluir troubleshooting básico común en minikube
```

---

## 🎯 Resultado Esperado del Prompt

Al usar este prompt, obtendrías un laboratorio que cubriría:

### **🚫 Sección "El Problema"**
- Crear un pod directamente con `kubectl run`
- Demostrar qué pasa cuando el pod se elimina accidentalmente
- Mostrar que no hay auto-recuperación
- Explicar las limitaciones de escalamiento manual

### **✅ Sección "La Solución"**
- Manifiesto YAML de Deployment básico
- Comandos para desplegar y verificar el Deployment
- Demostración de self-healing (eliminar pod y ver que se recrea)
- Escalamiento horizontal (`kubectl scale`)
- Rolling updates con nueva imagen
- Rollback a versión anterior
- Comparación visual de comportamientos

### **📊 Sección "Casos Prácticos"**
- Deployment de aplicación web frontend
- Deployment de API backend con múltiples réplicas
- Deployment con configuración específica de recursos

### **🧹 Sección "Limpieza"**
- Comandos para eliminar todos los recursos
- Verificación de limpieza completa

---

## 🔧 Variaciones del Prompt para Otros Casos

### Para Nivel Intermedio:
```
**CONTEXTO ADICIONAL:** 
- Nivel: Intermedio
- Enfócate en: 
  * Estrategias de deployment (RollingUpdate vs Recreate)
  * Health checks (liveness/readiness probes)
  * Resource limits y requests
  * Selector labels avanzados
  * Deployment history y anotaciones
```

### Para Entorno Cloud:
```
**CONTEXTO ADICIONAL:** 
- Entorno: Amazon EKS
- Enfócate en:
  * Load balancers externos
  * Persistent storage con EBS
  * Auto-scaling con múltiples AZs
  * Integración con AWS services
```

### Para Aplicaciones Específicas:
```
**CONTEXTO ADICIONAL:** 
- Aplicación: Microservicios de e-commerce
- Enfócate en:
  * Frontend (React/Vue)
  * Backend API (Node.js/Python)
  * Base de datos (con StatefulSet)
  * Comunicación entre servicios
```

---

## 💡 Tips para Personalizar Prompts

### 1. **Ajusta el Nivel de Complejidad**
```
- Básico: Conceptos fundamentales, comandos simples
- Intermedio: Configuraciones avanzadas, best practices
- Avanzado: Integración con otras herramientas, casos complejos
```

### 2. **Especifica el Entorno**
```
- Minikube: Comandos específicos, limitaciones locales
- Cloud Provider: Servicios específicos, integración nativa
- Bare Metal: Configuración manual, networking
```

### 3. **Define el Contexto de Aplicación**
```
- Web apps: Frontend/backend separation
- Microservices: Service discovery, communication
- Data processing: Batch jobs, streaming
- ML workloads: GPU resources, model serving
```

### 4. **Incluye Herramientas Específicas**
```
- Helm: Package management, templating
- Kustomize: Configuration management
- GitOps: ArgoCD, Flux integration
- Monitoring: Prometheus, Grafana
```

---

## 🚀 Prompt Listo para Usar

**Copia y pega este prompt personalizado:**

```
Genera un laboratorio práctico de Kubernetes sobre **Deployments**. [Incluir estructura completa...]

**TEMA ESPECÍFICO:** Deployments vs Pods directos
**CONTEXTO ADICIONAL:** 
- Entorno: Minikube local
- Nivel: Básico (principiantes)
- Enfócate en diferencias fundamentales entre pods directos y deployments
- Incluir auto-recuperación, escalamiento y rolling updates básicos
- Usar aplicaciones web simples como nginx
- Comandos específicos para minikube cuando sea relevante
```

¡Con este prompt específico obtendrás un laboratorio perfecto para aprender Deployments desde cero en Minikube! 🎯