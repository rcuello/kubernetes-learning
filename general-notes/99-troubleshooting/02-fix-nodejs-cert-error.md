# 🛠️ Fix: Node.js / Python / Minikube - unable to get local issuer certificate

Este documento describe cómo solucionar errores relacionados con certificados en entornos empresariales que utilizan inspección HTTPS, como Zscaler. Estos errores se manifiestan típicamente en herramientas como Node.js, Python, Minikube, Prisma, etc., y generalmente muestran mensajes como:

```
Error: unable to get local issuer certificate
```

---

## 📍 Contexto del problema

En redes corporativas con inspección HTTPS (Zscaler, firewalls empresariales), los certificados presentados por los servidores pueden ser interceptados y firmados por una autoridad corporativa. Esto genera errores en herramientas que no confían en estos certificados por defecto.

---

## 🧾 Requisitos

* Tener el certificado raíz/intermedio en formato `.cer` o `.pem`, por ejemplo:

  * `C:\vault\certificates\zscaler.cer`

---

## 🔧 Solución general (Windows, permanente)

Usamos PowerShell para establecer variables de entorno a nivel de máquina:

```powershell
[System.Environment]::SetEnvironmentVariable("NODE_EXTRA_CA_CERTS", "C:\vault\certificates\zscaler.cer", [EnvironmentVariableTarget]::Machine)
[System.Environment]::SetEnvironmentVariable("REQUESTS_CA_BUNDLE", "C:\vault\certificates\zscaler.cer", [EnvironmentVariableTarget]::Machine)
[System.Environment]::SetEnvironmentVariable("SSL_CERT_FILE", "C:\vault\certificates\zscaler.cer", [EnvironmentVariableTarget]::Machine)
```

Estas variables afectan:

* `NODE_EXTRA_CA_CERTS`: Node.js (incluye `npm`, `yarn`, Prisma)
* `REQUESTS_CA_BUNDLE`: Python (requests, pip, etc.)
* `SSL_CERT_FILE`: OpenSSL (puede afectar otras herramientas como Minikube)

> 🔁 Reinicia terminales o la máquina para que los cambios surtan efecto.

---

## 🐳 Minikube (si usa Docker/Hyper-V y errores persisten)

Minikube también puede requerir configuración del certificado, especialmente si descarga imágenes o ejecuta comandos remotos.

1. Copia el certificado al path donde Minikube pueda leerlo:

```powershell
Copy-Item "C:\vault\certificates\zscaler.cer" "$env:USERPROFILE\.minikube\ca-cert.pem"
```

2. También puedes iniciar Minikube con configuración explícita:

```bash
minikube start --embed-certs --extra-config=kubelet.certificate-authority=C:\vault\certificates\zscaler.cer
```

---

## 🧪 Validación

* `npm install` / `prisma db pull` / `pip install` deberían funcionar sin errores de certificados.
* Puedes probar en Python:

```python
import requests
requests.get("https://www.google.com")  # debería funcionar sin error
```

---

## 📝 Notas adicionales

* Asegúrate de tener privilegios de administrador al establecer las variables a nivel `Machine`.
* Revisa que el certificado esté en formato correcto (PEM/DER). Puedes convertir con OpenSSL si es necesario.

```bash
openssl x509 -inform der -in zscaler.cer -out zscaler.pem
```

