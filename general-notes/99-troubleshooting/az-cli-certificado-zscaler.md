# 🔐 Cómo solucionar el error de verificación de certificado en Azure CLI con Zscaler

Cuando estás detrás de un proxy corporativo como Zscaler, Azure CLI puede fallar en operaciones como `az login` debido a problemas de validación SSL:

```
[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1010)
```

Este documento explica cómo solucionar este problema integrando el certificado raíz de Zscaler en la configuración de Azure CLI.

---

## 🧾 Requisitos

* Certificado raíz del proxy (`zscaler.cer` o `zscaler.pem`)
* Acceso de administrador para modificar archivos dentro del directorio de instalación de Azure CLI
* PowerShell en Windows
* Opción: `openssl` si necesitas convertir el certificado

---

## 🛠️ Solución recomendada en PowerShell

### 1. Verifica que tienes el certificado Zscaler en formato PEM

Debe comenzar con:

```
-----BEGIN CERTIFICATE-----
```

Si está en formato DER, conviértelo con OpenSSL:

```bash
openssl x509 -inform DER -in zscaler.cer -out zscaler.pem
```

Guárdalo en una ubicación accesible, por ejemplo:

```powershell
C:\DevOps\certificates\zscaler.pem
```

---

### 2. Combina el certificado Zscaler con el `cacert.pem` de Azure CLI

Busca el archivo `cacert.pem` dentro del entorno virtual de Python que usa Azure CLI. Dos ubicaciones comunes:

* `C:\Program Files\Microsoft SDKs\Azure\CLI2\Lib\site-packages\certifi\cacert.pem`
* `C:\Program Files\Microsoft SDKs\Azure\CLI2\Lib\site-packages\pip\_vendor\certifi\cacert.pem`

> Revisa que el archivo esté presente y haz una copia de respaldo antes de modificarlo.

```powershell
# Ruta del cacert.pem (el que usa Azure CLI)
$certifiPath = "C:\Program Files\Microsoft SDKs\Azure\CLI2\Lib\site-packages\certifi\cacert.pem"

# Ruta de tu certificado Zscaler en formato PEM
$zscalerCert = "C:\DevOps\certificates\zscaler.pem"

# Combina ambos certificados en un nuevo archivo
Get-Content $certifiPath, $zscalerCert | Set-Content "$certifiPath.modified.pem"
```

---

### 3. Configura la variable de entorno `REQUESTS_CA_BUNDLE`

```powershell
$env:REQUESTS_CA_BUNDLE = "$certifiPath.modified.pem"
```

Esta variable indica al módulo `requests` de Python (usado por Azure CLI) que use el nuevo bundle con el certificado agregado.

---

### 4. Prueba la autenticación con Azure CLI

```powershell
az login
```

Si todo está correcto, se abrirá el navegador y no habrá error SSL.

---

## ✅ Verificación opcional

Puedes verificar si el bundle está funcionando correctamente con:

```powershell
python -c "import requests; print(requests.get('https://login.microsoftonline.com').status_code)"
```

---
