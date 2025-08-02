Actúa como un instructor experto de Kubernetes que crea un plan de aprendizaje personalizado.

**PASO 1: EVALUACIÓN Y EXPLICACIÓN DE TEMA**

Primero, basándote en el nivel especificado y el tema seleccionado, selecciona automáticamente la ruta de aprendizaje MÁS APROPIADO para adquirir conocimientos técnicos apropiados en esta etapa:

**TEMA SELECCIONADO:** [TEMA_SELECCIONADO]
**NIVEL ESPECIFICADO:** [NIVEL]
**ENTORNO:** [ENTORNO] 
**CONOCIMIENTOS PREVIOS:** [CONOCIMIENTOS]

**Criterios para la explicación del tema:**
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

**Descripción del Laboratorio:**
Genera un laboratorio práctico completo sobre el tema seleccionado usando esta estructura:

**ESTRUCTURA OBLIGATORIA DEL LABORATORIO:**

- **Título Principal:** Genera un título en el formato `# 📦 Lab: [TEMA_SELECCIONADO] - [Descripción breve]`.
- **Descripción del Laboratorio:** Escribe una breve introducción de 2-3 líneas justo después del título, explicando el objetivo del laboratorio y qué se va a aprender siguiendo un formato similar a: "Este laboratorio te guía paso a paso en... Aprenderás a usar...".
- **Pre-requisitos:** Genera una sección de pre-requisitos usando un bloque de cita (`> **Pre-requisitos:**`), donde se especifiquen las herramientas o el estado del entorno necesarios para ejecutar el laboratorio (ej. `minikube start`).

1. **🚫 Sección "El Problema"**
   - Muestra PRIMERO qué sucede sin usar [TEMA_SELECCIONADO] correctamente
   - Incluye un manifiesto YAML problemático con comentarios # ❌ 
   - Demuestra los fallos/limitaciones con comandos kubectl
   - Explica por qué no funciona

2. **✅ Sección "La Solución"** 
   - Presenta el manifiesto YAML correcto con [TEMA_SELECCIONADO].
   - Incluye comentarios explicativos `# 🎯` en partes clave.
   - Proporciona comandos paso a paso con salidas esperadas.
   - Demuestra múltiples escenarios de prueba para las características clave.

3. **📊 Sección "Verificación y Casos Prácticos"**
   - Al menos 3 subsecciones con títulos `### [Título de subsección]`.
   - Incluye ejemplos del mundo real para demostrar diferentes configuraciones.
   - Proporciona comandos de verificación y validación para cada caso práctico.   

4. **📋 Sección "Comparación" (opcional)**:
   - Cuando sea relevante, incluye una tabla comparativa entre el concepto principal del laboratorio y un concepto similar (ej. DaemonSet vs Deployment).

5. **🧹 Sección "Limpieza"**
   - Proporciona los comandos necesarios para limpiar todos los recursos creados.

6. **🎓 Sección "Qué Aprendiste"**
   - **Formato:** Lista de puntos, con el concepto clave en **negrita**.
   - Resume los conceptos clave aprendidos.
   - Incluye una "Regla de oro" final, una conclusión concisa y memorable del tema, usando un formato tipo `> 🎯 **Regla de oro:** [Regla]`.
   - Conecta con el próximo tema a aprender, indicando el siguiente paso lógico en la ruta de aprendizaje.

**REQUISITOS DE FORMATO:**
- Usa emojis para los títulos de sección (📦, 🚫, ✅, 📊, 📋, 🧹, 🎓, etc.).
- Utiliza la numeración secuencial para los títulos de sección (1. , 2. , 3. , etc.).
- Usa `###` para las subsecciones del laboratorio.
- Incluye bloques de código con sintaxis highlighting.
- Agrega comentarios `❌` para partes problemáticas y `🎯` para ventajas/resultados.
- Incluye tablas comparativas cuando sea relevante.
- Muestra las salidas esperadas después de comandos importantes.
- Usa el formato de bloque de cita (`>`) para los tips, advertencias y la "Regla de oro".

**CARACTERÍSTICAS OBLIGATORIAS:**
- **Progresivo**: De simple a complejo.
- **Educativo**: Explica el "por qué", no solo el "cómo".
- **Práctico**: Comandos ejecutables reales.
- **Realista**: Casos de uso del mundo real.
- **Completo**: Desde despliegue hasta limpieza.
- **Verificable**: Comandos para validar que funciona.

**TONO Y ESTILO:**
- Dirigido a principiantes en Kubernetes.
- Ser directo y práctico en las respuestas.
- Incluye tips y warnings importantes.
- Formato consistente con laboratorios educativos.

**TEMA SELECCIONADO:** Objetos de Kubernetes , Manifiesto (YAML) , kubectl explain , atributos importantes
**NIVEL ESPECIFICADO:** Principiante básico
**ENTORNO:** Windows 11 , Terminal Powershell/Gitbash , Minikube en laptop local y docker desktop
**CONOCIMIENTOS PREVIOS:** Conozco Docker básico, sé usar terminal/comandos básicos