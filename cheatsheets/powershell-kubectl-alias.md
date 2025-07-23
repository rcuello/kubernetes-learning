# 🧩 Crear alias `k` para `kubectl` en PowerShell

## ✅ Alias temporal (solo por la sesión actual)

Puedes crear un alias temporal ejecutando este comando en PowerShell:

```powershell
Set-Alias k kubectl
```

Ahora puedes usar `k` como alias de `kubectl`, por ejemplo:

```powershell
k get pods
```

> ⚠️ Este alias se perderá cuando cierres la terminal.

---

## 🔒 Alias persistente (para todas las sesiones)

Para mantener el alias disponible siempre, agrégalo al perfil de PowerShell.

### 1. Verificar si tienes un perfil:

```powershell
Test-Path $PROFILE
```

Si devuelve `False`, créalo con:

```powershell
New-Item -ItemType File -Path $PROFILE -Force
```

### 2. Editar el perfil:

```powershell
notepad $PROFILE
```

Agrega la siguiente línea al final del archivo:

```powershell
Set-Alias k kubectl
```

Guarda y cierra el archivo.

### 3. Aplicar los cambios:

```powershell
. $PROFILE
```

---

## 🧪 Probar alias

Ejecuta:

```powershell
k get nodes
```

Esto debería funcionar igual que:

```powershell
kubectl get nodes
```

¡Y listo! Ahora puedes usar `k` en lugar de `kubectl` también en PowerShell.
