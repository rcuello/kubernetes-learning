-----

# Diagnóstico de Conectividad a un Servicio Externo en Kubernetes

> Guía paso a paso para desarrolladores que enfrentan problemas de conexión desde sus Pods a recursos externos, utilizando el modelo TCP/IP para un diagnóstico sistemático.

-----

## El Problema: "Mi Aplicación no Conecta al Exterior"

Como desarrollador, es frustrante cuando tu aplicación funciona localmente, pero al desplegarla en Kubernetes, **no logra conectar con servicios externos**. Puedes ver errores en los logs como:

  * `Connection timed out`
  * `Network unreachable`
  * `Could not resolve host`
  * `Connection refused`
  * `SSL handshake failed`

Este documento te guiará a través de un proceso de diagnóstico sistemático, aprovechando el entendimiento del **modelo TCP/IP**, para identificar dónde se rompe la comunicación.

-----

## La Metodología: Capa por Capa del TCP/IP

El modelo TCP/IP divide la comunicación de red en 4 capas. Al diagnosticar, avanzaremos desde la capa de **Aplicación** (la más cercana a tu código) hacia abajo, hasta la capa de **Acceso a la Red** (la más fundamental). Esto te permitirá descartar problemas en cada nivel.

### **Herramientas Clave:**

Necesitarás acceso a la terminal dentro de tu Pod. Usa `kubectl exec` para abrir una shell:

```bash
kubectl exec -it <nombre-de-tu-pod> -- bash
# O si tu imagen base no tiene bash, prueba sh:
# kubectl exec -it <nombre-de-tu-pod> -- sh
```

Si tu Pod no tiene herramientas de red como `curl`, `nslookup`, `ping`, `nc` (netcat) o `telnet`, puedes:

1.  **Instalarlas temporalmente** (si la imagen base lo permite, ej. `apt update && apt install curl iputils-ping dnsutils netcat-traditional`).
2.  **Usar un Pod de depuración con `nicolaka/netshoot`:**
    ```bash
    kubectl run debug-toolkit --rm -ti --image=nicolaka/netshoot -- bash
    ```
    Luego, desde este Pod de depuración, puedes intentar las conexiones.
3.  **Usar `kubectl debug` (si tu versión de Kubernetes lo soporta):**
    ```bash
    kubectl debug -it <nombre-de-tu-pod> --image=nicolaka/netshoot --target=<nombre-de-tu-contenedor-principal>
    ```

-----

## Pasos de Diagnóstico

### **Capa 4: Aplicación (Tu Código y los Servicios)**

Esta es la primera capa a revisar porque es donde tu aplicación interactúa directamente.

  * **¿Qué verificar?**
      * **URL/IP y Puerto:** ¿La aplicación está configurada para conectarse a la dirección y puerto correctos del servicio externo?
      * **Protocolo:** ¿Está usando el protocolo adecuado (HTTP/S, gRPC, una conexión de base de datos específica)?
      * **Certificados TLS/SSL:** Si es HTTPS, ¿hay problemas con certificados (expirados, no confiables)?
      * **Lógica de Reintentos/Timeouts:** ¿Tu aplicación tiene timeouts demasiado agresivos o carece de lógica de reintentos?
  * **Comandos/Acciones:**
      * **Revisa los logs de tu aplicación:** Busca el error exacto. A menudo, el mensaje de error de la librería HTTP/DB ya te da pistas sobre la causa raíz.
        ```bash
        kubectl logs <nombre-de-tu-pod> -f
        ```
      * **Prueba de `curl` (o herramienta equivalente para tu protocolo) desde el Pod:** Intenta acceder al servicio externo directamente.
        ```bash
        # Para HTTP/S
        curl -v https://api.mi-servicio-externo.com/health

        # Para una base de datos, quizás un cliente específico si está instalado, o al menos un telnet/nc
        ```
  * **Resultados y Siguientes Pasos:**
      * **`curl` funciona / La aplicación sigue fallando:** El problema es casi seguro en el **código de tu aplicación** (variables de entorno incorrectas, librerías mal configuradas, lógica de negocio defectuosa). La red funciona bien hasta este punto.
      * **`curl` falla con un `Connection timed out`, `Could not resolve host` o similar:** El problema está en una capa inferior. ¡Continuamos\!
      * **`curl` falla con `SSL handshake failed` o `certificate verify failed`:** El problema está en la gestión de certificados TLS/SSL de tu aplicación o del entorno del Pod (Capa de Aplicación/Presentación, a veces en Capa de Transporte si no se negocia bien el TLS).

### **Capa 3: Transporte (Puertos y Conexiones)**

Aquí verificamos si se puede establecer una conexión al puerto remoto.

  * **¿Qué verificar?**
      * **Firewalls:** ¿Existe un firewall (en Kubernetes, en tu proveedor de nube, o en el destino) bloqueando el puerto?
      * **Servicio Escuchando:** ¿El servicio externo realmente está escuchando en el puerto esperado?
  * **Comandos/Acciones:**
      * **`nc` (netcat) o `telnet` al puerto remoto:** Esta es la prueba definitiva para saber si puedes *tocar* el puerto remoto.
        ```bash
        # Prueba si el puerto 443 está abierto en el dominio externo
        nc -vz api.mi-servicio-externo.com 443
        # Ejemplo de salida exitosa: Connection to api.mi-servicio-externo.com 443 port [tcp/https] succeeded!
        ```
  * **Resultados y Siguientes Pasos:**
      * **`nc` / `telnet` se conecta (`succeeded!`):** La conectividad al puerto está bien (Capa de Transporte y Capas inferiores OK). El problema vuelve a ser en la **Capa de Aplicación** (Capa 4 de TCP/IP) o problemas de certificado (Capa 4.5/5/6 en OSI). Revisa los logs de tu aplicación y la configuración de SSL/TLS.
      * **`nc` / `telnet` falla (`Connection refused`, `Timed out`):** El problema está en la Capa de Transporte o inferior.
          * **`Connection refused`**: El servidor remoto recibió la conexión pero la rechazó activamente. Esto a menudo significa que el servicio no está escuchando en ese puerto o que un firewall lo bloqueó activamente.
          * **`Timed out`**: La conexión nunca llegó al destino o no se recibió respuesta. Esto apunta a un bloqueo en la ruta (firewall intermedio) o que el destino no existe o no responde.
          * **`Network unreachable`**: Problema en la capa de Internet. ¡Continuamos\!

### **Capa 2: Internet (Direcciones IP y Enrutamiento)**

Aquí nos aseguramos de que tu Pod pueda resolver el nombre del servicio a una IP y que haya una ruta para llegar a esa IP.

  * **¿Qué verificar?**
      * **Resolución DNS:** ¿Tu Pod puede convertir el nombre de dominio (`api.mi-servicio-externo.com`) a una dirección IP?
      * **Rutas:** ¿Existe una ruta de red válida desde tu Pod hasta la IP del servicio externo?
      * **Firewalls de Red:** ¿Hay un firewall (NetworkPolicy en K8s, Security Group en la nube) que bloquea el tráfico a nivel de IP?
  * **Comandos/Acciones:**
      * **`nslookup` o `dig` para resolución DNS:**
        ```bash
        nslookup api.mi-servicio-externo.com
        # o
        dig api.mi-servicio-externo.com
        ```
          * **Si falla `nslookup` (`server can't find`):** El problema es de **DNS**. Tu Pod no puede resolver nombres de dominio. Revisa la configuración DNS de tu clúster (CoreDNS) o del Pod (`/etc/resolv.conf`).
          * **Si funciona `nslookup`:** Toma la IP que te da.
      * **`ping` a la IP resuelta:**
        ```bash
        ping -c 4 <IP-del-servicio-externo>
        ```
          * **Si `ping` funciona:** La conectividad IP básica está bien. El problema es más probable en la **Capa de Transporte** (firewall en el puerto, servicio no escuchando).
          * **Si `ping` falla (`Host unreachable`, `Request timed out`):** El problema está en el enrutamiento o un firewall bloqueando el ICMP (ping).
      * **`traceroute` a la IP resuelta (si `ping` falla):**
        ```bash
        traceroute <IP-del-servicio-externo>
        ```
        `traceroute` te mostrará la ruta que toman los paquetes y dónde se detienen, lo que es invaluable para identificar firewalls o routers que no reenvían el tráfico.
  * **Resultados y Siguientes Pasos:**
      * **DNS falla:** Resuelve el problema de DNS en tu clúster.
      * **Ping/Traceroute falla:** Contacta a tu equipo de red/infraestructura. Podría ser un **firewall a nivel de red** (NetworkPolicy en Kubernetes que impide la salida del tráfico del Pod, Security Groups de la VPC, firewall empresarial) o un problema de **enrutamiento** más complejo.

### **Capa 1: Acceso a la Red (Hardware y Enlaces Lógicos)**

Esta es la capa más baja y, a menudo, menos probable que sea la causa de un problema de conectividad externa si el resto del clúster funciona. Sin embargo, si los pasos anteriores no revelan nada, puede ser el momento de mirar aquí.

  * **¿Qué verificar?**
      * **Estado del Nodo:** ¿El nodo donde está tu Pod está "Ready"?
      * **Interfaces de Red:** ¿Las interfaces de red del nodo están levantadas y correctamente configuradas?
      * **Plugin CNI:** ¿Hay problemas con el plugin de red de contenedores (CNI) que asigna IPs a los Pods y configura sus interfaces virtuales?
  * **Comandos/Acciones (Normalmente requieren acceso SSH al nodo):**
      * **Estado del nodo:**
        ```bash
        kubectl get nodes
        ```
      * **Interfaces de red en el nodo:**
        ```bash
        ip a show
        ```
      * **Logs del CNI:** Dependiendo de tu CNI (Calico, Flannel, Cilium, etc.), revisa sus logs en el namespace `kube-system`.
        ```bash
        kubectl logs -n kube-system -l k8s-app=<nombre-app-cni>
        ```
  * **Resultados:**
      * Problemas aquí suelen indicar fallos de infraestructura mayores (configuración de red del nodo, cables, virtualización). Si llegaste a este punto, es probable que no sea un problema exclusivo de tu aplicación, sino del clúster entero o del nodo.

-----

## Patrones Comunes y Dónde Buscarlos

| Síntoma/Error Típico | Capa TCP/IP Probable | Causa Más Común | Sugerencia Rápida |
| :------------------- | :-------------------- | :--------------- | :---------------- |
| `Connection refused` | **Transporte (3)** | Firewall de destino bloqueando puerto, o servicio no escuchando en el puerto. | Usa `nc -vz <host> <puerto>`. Revisa firewalls destino. |
| `Connection timed out` | **Aplicación (4), Transporte (3), Internet (2)** | Firewall intermedio, servicio no existe/no responde, red lenta. | Prueba con `curl`, `nc -vz`, `ping`, `traceroute`. |
| `Could not resolve host` | **Internet (2)** | Problema de DNS. | Usa `nslookup` o `dig`. Revisa configuración DNS del Pod (`/etc/resolv.conf`) y CoreDNS. |
| `Network unreachable` | **Internet (2)** | Problema de enrutamiento o firewall a nivel de red. | Usa `ping` y `traceroute`. Revisa NetworkPolicies. |
| `SSL handshake failed` | **Aplicación (4)** | Certificados SSL/TLS incorrectos, expirados, o no confiables. | Revisa logs de aplicación para errores de certificado, verifica secretos de TLS. |
| `404 Not Found`, `500 Internal Server Error` | **Aplicación (4)** | La conexión de red funciona, pero la lógica de la aplicación remota falla. | Revisa logs de la aplicación externa (si tienes acceso) o la configuración de tu propia aplicación. |

-----

## Kit de Emergencia del Desarrollador para Debugging de Red

Un Pod con herramientas de red preinstaladas puede ser un salvavidas:

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: debug-toolkit
  labels:
    app: debug-toolkit
spec:
  containers:
  - name: toolkit
    image: nicolaka/netshoot # Incluye ping, curl, nslookup, telnet, tcpdump, etc.
    command: ["/bin/bash", "-c", "sleep infinity"] # Mantén el Pod vivo
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
  restartPolicy: Never
EOF
```

Para usarlo:

```bash
kubectl exec -it debug-toolkit -- bash
```

Cuando termines, no olvides limpiarlo:

```bash
kubectl delete pod debug-toolkit
```

-----

Al seguir estos pasos sistemáticos, transformarás la frustración de un error de conectividad en un proceso de diagnóstico claro y eficiente. ¡Dominar el modelo TCP/IP es una de las mejores habilidades que un desarrollador puede tener para trabajar con sistemas distribuidos como Kubernetes\!