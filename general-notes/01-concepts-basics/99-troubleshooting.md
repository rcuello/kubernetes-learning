## 🛠️ Troubleshooting: Error `ImagePullBackOff` en redes corporativas

En entornos con **restricciones de red o TLS interceptado por proxy corporativo**, Kubernetes puede fallar al descargar imágenes desde DockerHub con errores como:

```
Failed to pull image "nginx": tls: failed to verify certificate: x509: certificate signed by unknown authority
```

### 💡 Solución: Usar imágenes locales con `minikube`

1. **Descargar la imagen con Docker localmente** (fuera del clúster):

   ```bash
   docker pull nginx
   ```

2. **Importar esa imagen al entorno de Minikube**:

   ```bash
   minikube image load nginx
   ```

3. **Crear el Pod normalmente (usará la imagen local)**:

   ```bash
   kubectl run nginx-nodeport --image=nginx --restart=Never --port=80 --namespace=desarrollo
   ```

### 🔍 Verificación

Puedes confirmar que el pod usó la imagen cargada correctamente si ves eventos como:

```
Successfully pulled image "nginx"
Created container: nginx-nodeport
Started container nginx-nodeport
```

### ✅ Consideraciones

* Esta técnica es útil para entornos corporativos donde la verificación de certificados externos falla.
* Aplica tanto a imágenes públicas como privadas (si las cargas previamente).
* También puedes usar un **registry privado local** o configurar un proxy corporativo con certificados adecuados, si es una solución repetitiva.
