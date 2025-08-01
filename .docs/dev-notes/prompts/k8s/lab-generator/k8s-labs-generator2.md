# 🤖 Prompt Autónomo para Laboratorios de Kubernetes

## 🎯 Prompt Principal - Generación Automática de Temas

```
Actúa como un instructor experto de Kubernetes que crea un plan de aprendizaje personalizado.

**PASO 1: EVALUACIÓN Y SELECCIÓN DE TEMA**

Primero, basándote en el nivel especificado, selecciona automáticamente el tema MÁS APROPIADO para aprender en esta etapa:

**NIVEL ESPECIFICADO:** [NIVEL]
**ENTORNO:** [ENTORNO] 
**CONOCIMIENTOS PREVIOS:** [CONOCIMIENTOS]

**Criterios para selección del tema:**
- Debe ser apropiado para el nivel especificado
- Debe seguir una progresión lógica de aprendizaje
- Debe tener casos de uso prácticos y relevantes
- Debe ser demostrable en el entorno especificado

**PASO 2: JUSTIFICACIÓN DEL TEMA**

Explica en 2-3 líneas:
- Por qué este tema es el siguiente paso lógico
- Qué habilidades/conocimientos se van a adquirir
- Cómo se conecta con conceptos anteriores o posteriores

**PASO 3: GENERACIÓN DEL LABORATORIO**

Genera un laboratorio práctico completo sobre el tema seleccionado usando esta estructura:

**ESTRUCTURA OBLIGATORIA:**

1. **🚫 Sección "El Problema"** (10-15% del contenido)
   - Muestra PRIMERO qué sucede sin usar [TEMA_SELECCIONADO] correctamente
   - Incluye un manifiesto YAML problemático con comentarios # ❌ 
   - Demuestra los fallos/limitaciones con comandos kubectl
   - Explica por qué no funciona

2. **✅ Sección "La Solución"** (60-70% del contenido)
   - Presenta el manifiesto YAML correcto con [TEMA_SELECCIONADO]
   - Incluye comentarios explicativos # 🎯 en partes clave
   - Comandos paso a paso con salidas esperadas
   - Múltiples escenarios de prueba para demostrar características

3. **📊 Sección "Casos Prácticos"** (15-20% del contenido)
   - Al menos 3 ejemplos del mundo real
   - Comandos de verificación y validación
   - Diferentes configuraciones para casos específicos

4. **🧹 Sección "Limpieza"** (5% del contenido)
   - Comandos para limpiar todos los recursos creados

5. **🎓 Sección "Qué Aprendiste"** (Nuevo - 5% del contenido)
   - Resumen de conceptos clave
   - Conexión con próximos temas a aprender
   - Sugerencia del siguiente tema lógico

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
- Dirigido a principiantes en Kubernetes
- Explicaciones claras sin jerga excesiva
- Incluye tips y warnings importantes
- Formato consistente con laboratorios educativos
```

---

## 🎯 Ejemplos de Uso del Prompt Autónomo

### Para Principiante Absoluto:
```
[Usar prompt completo arriba]

**NIVEL ESPECIFICADO:** Principiante absoluto (nunca he usado Kubernetes)
**ENTORNO:** Minikube en laptop local
**CONOCIMIENTOS PREVIOS:** Conozco Docker básico, sé usar terminal/comandos básicos
```

**Resultado esperado:** El LLM seleccionaría probablemente **Pods** como primer tema, ya que son la unidad básica de Kubernetes.

### Para Alguien que ya Conoce Pods:
```
[Usar prompt completo arriba]

**NIVEL ESPECIFICADO:** Principiante (conozco pods básicos)
**ENTORNO:** Minikube en laptop local  
**CONOCIMIENTOS PREVIOS:** He creado pods simples, sé usar kubectl run y kubectl get pods
```

**Resultado esperado:** Probablemente seleccionaría **Services** o **Deployments** como siguiente paso lógico.

### Para Nivel Intermedio:
```
[Usar prompt completo arriba]

**NIVEL ESPECIFICADO:** Intermedio
**ENTORNO:** Cluster de Google GKE
**CONOCIMIENTOS PREVIOS:** Manejo Pods, Deployments, Services básicos. He hecho aplicaciones simples.
```

**Resultado esperado:** Podría seleccionar **ConfigMaps/Secrets**, **Ingress**, o **Persistent Volumes**.

### Para Enfoque Específico:
```
[Usar prompt completo arriba]

**NIVEL ESPECIFICADO:** Intermedio con enfoque en seguridad
**ENTORNO:** Amazon EKS
**CONOCIMIENTOS PREVIOS:** Tengo aplicaciones funcionando, ahora quiero hacerlas más seguras
```

**Resultado esperado:** Seleccionaría **RBAC**, **Network Policies**, o **Security Contexts**.

---

## 🚀 Prompt Simplificado para Uso Rápido

```
Actúa como instructor experto de Kubernetes. Selecciona automáticamente el tema MÁS APROPIADO para mi nivel y genera un laboratorio completo.

**MI SITUACIÓN:**
- Nivel: [PRINCIPIANTE/INTERMEDIO/AVANZADO]
- Entorno: [MINIKUBE/EKS/GKE/AKS]
- Conocimientos previos: [DESCRIBE_TU_EXPERIENCIA]

**PASO 1:** Dime qué tema seleccionaste y por qué es el siguiente paso lógico para mí.

**PASO 2:** Genera un laboratorio completo con la estructura problema→solución→casos prácticos→limpieza, incluyendo:
- YAMLs problemáticos y correctos
- Comandos paso a paso
- Salidas esperadas
- Casos del mundo real
- Próximo tema sugerido

Formato educativo con emojis, comentarios explicativos y comandos verificables.
```

---

## 🎓 Prompt para Rutas de Aprendizaje Completas

```
Actúa como arquitecto de currículo de Kubernetes. Crea una ruta de aprendizaje personalizada de 10 laboratorios.

**MI PERFIL:**
- Nivel actual: [DESCRIBE_TU_NIVEL]
- Objetivo: [QUÉ_QUIERES_LOGRAR]
- Tiempo disponible: [HORAS_POR_SEMANA]
- Entorno: [TU_ENTORNO]

**ENTREGABLES:**

1. **📋 Ruta de Aprendizaje (Tabla)**
   | Lab | Tema | Objetivo | Prerrequisitos | Tiempo Est. |
   |-----|------|----------|----------------|-------------|
   | 1   | ?    | ?        | ?              | ? horas     |

2. **🎯 Selecciona el Lab #1** y justifica por qué empezar ahí

3. **📦 Genera el Lab #1 completo** con estructura problema→solución→práctica→limpieza

4. **🔗 Conexión al Lab #2** - Breve descripción de qué viene después

**CRITERIOS:**
- Progresión lógica y pedagógica
- Cada lab construye sobre el anterior
- Balance entre teoría y práctica
- Casos de uso relevantes y motivadores
```

---

## 💡 Variaciones por Contexto

### Para DevOps/SRE:
```
**CONTEXTO ADICIONAL:** Enfoque en operaciones, monitoreo, automatización y reliability
```

### Para Desarrolladores:
```
**CONTEXTO ADICIONAL:** Enfoque en desarrollo de aplicaciones, CI/CD, debugging
```

### Para Arquitectos:
```
**CONTEXTO ADICIONAL:** Enfoque en diseño de sistemas, patrones, governance
```

### Para Seguridad:
```
**CONTEXTO ADICIONAL:** Enfoque en security policies, compliance, threat modeling
```

---

## 🎯 Ejemplo Completo Listo para Usar

```
Actúa como instructor experto de Kubernetes. Selecciona automáticamente el tema MÁS APROPIADO para mi nivel y genera un laboratorio completo.

**MI SITUACIÓN:**
- Nivel: Principiante (sé usar Docker, pero Kubernetes es nuevo para mí)
- Entorno: Minikube en Windows con WSL2
- Conocimientos previos: He usado docker run, docker build. Entiendo contenedores básicos. No he usado kubectl nunca.

**PASO 1:** Dime qué tema seleccionaste y por qué es el siguiente paso lógico para mí.

**PASO 2:** Genera un laboratorio completo con estructura problema→solución→casos prácticos→limpieza, incluyendo:
- YAMLs problemáticos y correctos con comentarios explicativos
- Comandos kubectl paso a paso con salidas esperadas  
- Al menos 3 casos del mundo real
- Comandos de verificación y troubleshooting
- Sección de limpieza completa
- Sugerencia del próximo tema a aprender

Usa formato educativo con emojis, comentarios # ❌ y # ✅, y comandos completamente verificables en minikube.
```

---

## ✅ Ventajas del Prompt Autónomo

🎯 **Selección inteligente** - El LLM elige el tema más apropiado
📚 **Progresión lógica** - Sigue una secuencia pedagógica correcta  
🔄 **Adaptativo** - Se ajusta a tu nivel y contexto específico
🎓 **Educativo** - Explica por qué ese tema es el siguiente paso
🚀 **Listo para usar** - No necesitas conocer todos los temas de Kubernetes

¡Con este prompt el LLM será tu instructor personal que sabe exactamente qué enseñarte en cada momento! 🤖