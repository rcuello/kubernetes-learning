# 🔐 Cómo solucionar el error de verificación de certificado en Azure CLI con Zscaler

Cuando estás detrás de un proxy corporativo como Zscaler, Azure CLI puede fallar en operaciones como `az login` debido a problemas de validación SSL:

```

\[SSL: CERTIFICATE\_VERIFY\_FAILED] certificate verify failed: unable to get local issuer certificate (\_ssl.c:1010)

```

Este documento explica cómo solucionar este problema integrando el certificado raíz de Zscaler en la configuración de Azure CLI.

---

## 🧾 Requisitos

* Certificado raíz del proxy (`zscaler.cer` o `zscaler.pem`)
* Acceso de administrador para modificar archivos dentro del directorio de instalación de Azure CLI
* PowerShell en Windows
* Opción: `openssl` si necesitas convertir el certificado

---

## 📤 Exportar el certificado corporativo (zscaler.cer)

### En Windows:

1. Ejecuta `certmgr.msc`.
2. Navega a: `Entidades de certificación raíz de confianza > Certificados`.
3. Busca el certificado **Zscaler Root**.
4. Haz clic derecho sobre él → `Todas las tareas > Exportar`.
5. Elige:
   * *Sin clave privada*
   * *Formato: Base-64 encoded X.509 (.CER)*

💡 Guarda el archivo como:
```

C:\vault\certificates\zscaler.cer

```

---

## 🔄 Convertir el certificado a formato PEM (si aplica)

Debe comenzar con:
```

\-----BEGIN CERTIFICATE-----

````

Si está en formato DER, conviértelo con:

```bash
openssl x509 -inform DER -in zscaler.cer -out zscaler.pem
````

Guárdalo como:

```
C:\vault\certificates\zscaler.pem
```

---

## 🛠️ Combinar certificados y configurar Azure CLI

### Paso 1: Ubica el cacert.pem utilizado por Azure CLI

Rutas comunes:

* `C:\Program Files\Microsoft SDKs\Azure\CLI2\Lib\site-packages\certifi\cacert.pem`
* `C:\Program Files\Microsoft SDKs\Azure\CLI2\Lib\site-packages\pip\_vendor\certifi\cacert.pem`

### Paso 2: Combina el certificado Zscaler con el bundle de certificados

```powershell
# Ruta del cacert.pem (el que usa Azure CLI)
$certifiPath = "C:\Program Files\Microsoft SDKs\Azure\CLI2\Lib\site-packages\certifi\cacert.pem"

# Ruta de tu certificado Zscaler en formato PEM
$zscalerCert = "C:\vault\certificates\zscaler.pem"

# Combina ambos certificados en un nuevo archivo
Get-Content $certifiPath, $zscalerCert | Set-Content "$certifiPath.modified.pem"
```

---

### Paso 3: Configura la variable de entorno

```powershell
$env:REQUESTS_CA_BUNDLE = "$certifiPath.modified.pem"
```

Esto indica a `requests` (usado internamente por Azure CLI) que utilice el nuevo bundle con el certificado de Zscaler incluido.


> Nota: En windows , puedes configurar la variable de entorno `REQUESTS_CA_BUNDLE` con el comando `sysdm.cpl`.

---


### Paso 4: Ejecuta `az login`

```powershell
az login
```

Deberías poder iniciar sesión sin errores de verificación SSL.

---

## ✅ Verificación opcional

Para verificar que el bundle está siendo usado correctamente:

```powershell
python -c "import requests; print(requests.get('https://login.microsoftonline.com').status_code)"
```

Un código `200` confirma que la conexión es exitosa con el nuevo certificado.

---

## 📎 Notas adicionales

* No sobreescribas `cacert.pem` directamente; mantener una copia modificada reduce el riesgo de errores con futuras actualizaciones de Azure CLI.
* Si deseas aplicar esto de forma permanente, puedes configurar la variable de entorno global en el sistema o script de inicio de PowerShell.

---
