# 🔐 Importar certificados corporativos en Minikube (Zscaler, proxies SSL)

## 📌 Objetivo

Permitir que un entorno local de **Minikube** acceda correctamente a servicios HTTPS externos —como **pull de imágenes Docker**, peticiones `curl`, entre otros— cuando se encuentra detrás de una red corporativa que intercepta tráfico TLS/SSL usando soluciones como **Zscaler** o proxies transparentes similares.

---

## 🧾 1. Exportar el certificado corporativo (Zscaler)

### En Windows:

1. Ejecuta `certmgr.msc`.
2. Navega a: `Entidades de certificación raíz de confianza > Certificados`.
3. Busca el certificado **Zscaler Root**.
4. Haz clic derecho sobre él → `Todas las tareas > Exportar`.
5. Elige:

   * *Sin clave privada*
   * *Formato: Base-64 encoded X.509 (.CER)*

> 💡 Guárdalo con nombre claro como:
> `C:\vault\certificates\zscaler.cer`

---

## 🚀 2. Iniciar Minikube y copiar el certificado

### Asegúrate de que Minikube está activo:

```powershell
minikube start
```

### Copia el certificado desde tu sistema Windows al entorno virtualizado de Minikube:

```powershell
# Ruta relativa desde PowerShell
minikube cp .\zscaler.cer /home/docker/zscaler.cer

# O con ruta completa de Windows
minikube cp "C:\vault\certificates\zscaler.cer" /home/docker/zscaler.cer
```

> 📌 El destino `/home/docker/` es el home del usuario dentro del nodo Minikube.

---

## 🛠️ 3. Instalar el certificado dentro de Minikube

### Accede a la shell del nodo Minikube:

```powershell
minikube ssh
```

### Mueve el certificado a la ubicación estándar del sistema:

```bash
sudo mv /home/docker/zscaler.cer /etc/ssl/certs/zscaler.crt
```

### Actualiza la base de certificados del sistema:

```bash
sudo update-ca-certificates
```

Deberías ver algo como:

```
Updating certificates in /etc/ssl/certs...
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

---

## ✅ 4. Validar conectividad HTTPS

### Desde dentro de Minikube, ejecuta:

```bash
curl https://kubernetes.io
```

Si el certificado fue correctamente reconocido, verás una respuesta HTML válida. Si no, verás errores como `SSL certificate problem: unable to get local issuer certificate`.

---

## 🐳 5. Verificar descarga de imágenes Docker

### Usa `crictl` o `docker` (si está disponible) para testear acceso a imágenes:

```bash
# Desde crictl
sudo crictl pull busybox
sudo crictl pull gcr.io/google-samples/hello-app:1.0

# También puedes probar:
docker pull nginx:alpine
```

> ⚠️ Si falla con `x509: certificate signed by unknown authority`, el certificado aún no fue cargado o reconocido correctamente.

---

## 📌 Consideraciones importantes

* 🔁 **Los cambios no persisten** entre reinicios del clúster o `minikube delete`.

  * Para persistencia, considera usar:

    * Volúmenes montados
    * `cloud-init` o scripts bootstrap personalizados
    * Imágenes base de Minikube modificadas

* 🐳 Si usas el driver Docker, también puedes importar el certificado en el *host Docker* ejecutando Minikube (`certs.d`, `daemon.json`, etc.).

* 🔐 Algunas redes corporativas también requieren variables de entorno `HTTPS_PROXY` o `NO_PROXY` configuradas.

---

## 🧼 Limpieza (opcional)

Para remover el certificado:

```bash
sudo rm /etc/ssl/certs/zscaler.crt
sudo update-ca-certificates --fresh
```

