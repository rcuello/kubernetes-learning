# ⚠️ Troubleshooting: Fallos de Conectividad en Laboratorio Ingress (Minikube)

> ¿Tu `Ingress` no responde? Usa esta guía de depuración capa por capa para resolver problemas comunes de red en tu entorno Minikube en Windows.

-----

## 🎯 El Objetivo de Este Documento

Este documento te ayudará a diagnosticar y resolver los problemas más frecuentes que impiden que tu `Ingress` funcione correctamente en Minikube, especialmente en Windows. Nos enfocaremos en usar un enfoque sistemático basado en el **modelo TCP/IP**, moviéndonos desde lo que ve tu navegador hasta las capas más profundas de la red.

## 🧱 Pre-verificación Esencial

Antes de empezar a depurar, asegúrate de que los componentes básicos de Kubernetes están funcionando:

  * **Minikube:** ¿Está corriendo?
    ```bash
    minikube status
    ```
    (Debe mostrar `host: Running`, `kubelet: Running`, `apiserver: Running`). Si no, intenta `minikube stop` y luego `minikube start`.
  * **Pods del Ingress Controller:** ¿Están los Pods del `ingress-nginx` en estado `Running`?
    ```bash
    kubectl get pods -n ingress-nginx
    ```
    Si no, revisa sus logs: `kubectl logs -n ingress-nginx <nombre-del-pod-ingress-nginx>`.
  * **Service `web` y su Deployment:** ¿Están `Running` y `Ready`?
    ```bash
    kubectl get svc web
    kubectl get deployment web
    kubectl get pods -l app=web # Para ver los pods específicos de tu app
    ```
    Si el `Service` no tiene `ENDPOINT`s, significa que los `Pod`s asociados no están `Ready`. Revisa los logs de tus `Pod`s (`kubectl logs <nombre-del-pod-web>`).
  * **Recurso Ingress:** ¿Existe y apunta correctamente a tu `Service`?
    ```bash
    kubectl describe ingress example-ingress
    ```
    Verifica que `Host` sea `hello-world.example` y `Backend` apunte a `web:8080`.

Si estos elementos básicos están en orden, entonces el problema es probable que esté en la **conectividad de red** entre tu host (Windows) y Minikube/Ingress. ¡Aquí es donde el modelo TCP/IP entra en juego\!

-----

## 🔎 Diagnóstico Capa por Capa (Modelo TCP/IP)

### **Paso 1: Capa de Aplicación (Tu Navegador / `curl`)**

Aquí es donde se manifiesta el problema. El navegador no carga la página o `curl` no obtiene respuesta HTTP.

  * **Síntoma:** Navegador carga infinitamente o muestra error "Este sitio no puede ser alcanzado", "Connection timed out". `curl` muestra `Connection timed out` o se queda esperando.
  * **Pregunta clave:** ¿La solicitud HTTP (`hello-world.example`) está llegando a algún lado y recibiendo una respuesta HTTP válida?
  * **Acción:** Intenta acceder a tu dominio con la opción verbose de `curl`. Esto te dará más detalles sobre dónde falla la conexión.
    ```bash
    curl -v http://hello-world.example
    ```
  * **Resultados y lo que significa:**
      * **Si obtienes una respuesta HTTP (200 OK, 404, 500, etc.):** ¡Excelente\! El problema está en la lógica de tu aplicación (ej. una ruta que no existe, un error interno). La conectividad de red básica funciona. Revisa los logs de tu aplicación (`kubectl logs <pod-web>`).
      * **Si `curl` muestra `Connection timed out`, `Failed to connect`, o se queda colgado:** El problema no es tu aplicación, sino que la solicitud HTTP ni siquiera está llegando o siendo respondida. Esto apunta a un problema en una capa **inferior**. Continúa al Paso 2.

-----

### **Paso 2: Capa de Transporte (Puertos TCP - El Ingress y `minikube tunnel`)**

Aquí verificamos si la conexión TCP al puerto 80 (el puerto estándar de HTTP) se puede establecer desde tu host Windows.

  * **Síntoma:** El `curl` del Paso 1 falla con `Connection timed out` o `Connection refused`.

  * **Pregunta clave:** ¿Está el puerto 80 de tu máquina Windows (donde `minikube tunnel` debería escuchar) libre y disponible para el tráfico del Ingress?

  * **Acciones:**

    1.  **Verificar si el puerto 80 está en uso en tu sistema Windows:**
        Este es un problema **MUY común** en Windows. IIS (Internet Information Services) u otros servicios (como WAMP, XAMPP, Skype antiguo, etc.) pueden estar usando el puerto 80. `minikube tunnel` necesita este puerto libre para redirigir el tráfico.
        Abre **CMD como Administrador** y ejecuta:
        ```cmd
        netstat -ano | findstr :80
        ```
          * **Busca:** Una línea que diga `LISTENING` para `0.0.0.0:80` o `127.0.0.1:80`. El número al final de la línea es el **PID** (ID de Proceso).
          * **Identifica el proceso:** Abre el Administrador de Tareas, ve a la pestaña "Detalles" y busca el PID. Si es `w3svc` (IIS), lo has encontrado.
    2.  **Probar conectividad directa al puerto 80 local:**
        Desde CMD normal:
        ```bash
        # Intentar telnet (si está instalado)
        telnet localhost 80
        # O netcat (si está instalado, en netshoot también funciona)
        nc -vz localhost 80
        ```
          * **Resultado esperado (si el puerto está libre y minikube tunnel funciona):** Conecta o indica éxito.
          * **Si ves "Connection refused":** Algo está bloqueando la conexión al puerto 80 localmente. Podría ser el firewall de Windows.
          * **Si ves una conexión establecida pero no responde (como si un servidor web estuviera esperando):** ¡Otro programa está escuchando en el puerto 80\!

  * **Solución para la Capa de Transporte:**

      * **Liberar el Puerto 80:**
          * **Detener IIS (CMD como Administrador):** `iisreset /stop`
          * **Deshabilitar IIS (PowerShell como Administrador):**
            ```powershell
            Stop-Service W3SVC
            Set-Service W3SVC -StartupType Disabled
            ```
          * Para otros programas, tendrás que identificarlos por su PID y cerrarlos o reconfigurarlos.
      * **Reiniciar `minikube tunnel`:** Una vez que el puerto 80 esté libre, detén cualquier instancia de `minikube tunnel` que tengas y reiníciala.
        ```bash
        # En la consola donde minikube tunnel está corriendo: Ctrl+C
        # Luego, inicia de nuevo en una nueva consola:
        minikube tunnel
        ```

### **Paso 3: Capa de Internet (IPs y DNS)**

Aquí confirmamos que el nombre de dominio se resuelve correctamente a una IP que `minikube tunnel` puede manejar, y que hay una ruta hacia ella.

  * **Síntoma:** Aunque el puerto 80 esté libre, la conexión sigue sin establecerse, o `ping hello-world.example` no muestra la IP esperada.
  * **Pregunta clave:** ¿Tu sistema operativo está dirigiendo `hello-world.example` a la IP correcta donde `minikube tunnel` espera el tráfico?
  * **Acciones:**
    1.  **Verificar la resolución de `hello-world.example` en tu `hosts`:**
        Abre `C:\Windows\System32\drivers\etc\hosts` (necesitas permisos de administrador).
          * **Error Común:** Puedes haber puesto la IP real de la VM de Minikube (ej. `192.168.49.2 hello-world.example`), y en algunos entornos de Windows, `minikube tunnel` tiene más facilidad para interceptar el tráfico que va a `127.0.0.1`.
          * **Acción:** Si tenías la IP de Minikube, cámbiala a `127.0.0.1 hello-world.example`. Guarda el archivo.
    2.  **Verificar la resolución de DNS local (después de modificar `hosts`):**
        Abre CMD y ejecuta:
        ```bash
        ping hello-world.example
        ```
          * **Resultado esperado:** Debe mostrar `Respuesta desde 127.0.0.1` (si usaste `127.0.0.1` en `hosts`) o la IP de Minikube (si esa configuración te funciona). Si apunta a otra IP o falla, tu archivo `hosts` no se guardó correctamente o hay otro problema de DNS.
  * **Solución para la Capa de Internet:**
      * Asegúrate de que la entrada en `hosts` apunte a `127.0.0.1` para `hello-world.example`. Esta es la configuración más robusta para `minikube tunnel` en Windows.

### **Paso 4: Capa de Acceso a la Red (Conectividad Subyacente)**

Esta capa es la menos probable que falle específicamente para Ingress si Minikube en general está funcionando, pero es el último recurso.

  * **Síntoma:** Minikube no inicia correctamente, o `minikube tunnel` falla con errores relacionados con la interfaz de red.
  * **Pregunta clave:** ¿La máquina virtual de Minikube tiene conectividad básica y sus interfaces de red virtuales están bien?
  * **Acciones:**
    1.  **Reinicia Minikube completamente:**
        ```bash
        minikube stop
        minikube start
        ```
    2.  **Verifica los logs del driver de Minikube:**
        Si estás usando VirtualBox o Hyper-V, revisa los logs de esas herramientas para ver si hay errores al iniciar la VM o al configurar la red.
  * **Solución:** A menudo, un reinicio completo de Minikube resuelve problemas de conectividad de bajo nivel. Si persisten, podría ser una configuración de firewall muy estricta en Windows o problemas con el software de virtualización.

-----

## 🚀 Volver a Probar

Después de cada solución aplicada (especialmente después de liberar el puerto 80 o modificar el archivo `hosts` y reiniciar `minikube tunnel`), vuelve al **Paso 1** y reintenta acceder a `http://hello-world.example` con tu navegador o `curl`.

Al seguir esta guía capa por capa, podrás identificar y resolver rápidamente la mayoría de los problemas de conectividad que encuentres en tus laboratorios de Kubernetes con Minikube. ¡La clave es la paciencia y la sistematicidad\!

-----