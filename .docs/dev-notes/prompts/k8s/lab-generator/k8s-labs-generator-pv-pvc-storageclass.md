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
- Ser directo y práctico en las respuestas
- Incluye tips y warnings importantes
- Formato consistente con laboratorios educativos

**TEMA SELECCIONADO:** PersistentVolumes (PVs),PersistentVolumeClaims (PVCs),StorageClass
**NIVEL ESPECIFICADO:** Principiante básico
**ENTORNO:** Windows 11 , Terminal Powershell/Gitbash , Minikube en laptop local y docker desktop
**CONOCIMIENTOS PREVIOS:** Conozco Docker básico, sé usar terminal/comandos básicos