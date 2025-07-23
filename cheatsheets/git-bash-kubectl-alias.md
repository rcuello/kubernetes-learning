# ⚡ Alias para `kubectl` en Git Bash (`k`)

En entornos donde se usa frecuentemente `kubectl`, es común definir un alias más corto para facilitar la escritura de comandos. Este documento explica cómo configurar el alias `k` para `kubectl` en **Git Bash** sobre **Windows**.

---

## ✅ Requisitos previos

- Tener instalado Git Bash para Windows.
- Tener instalado `kubectl` y disponible en el `PATH`.

---

## 🛠️ Pasos para configurar el alias `k`

### 1. Abrir Git Bash

Inicia Git Bash como lo haces normalmente.

### 2. Editar el archivo `.bashrc`

Ejecuta en la terminal:

```bash
nano ~/.bashrc
````

> Si el archivo no existe, este comando lo creará.

### 3. Agregar el alias

Desplázate al final del archivo y agrega la siguiente línea:

```bash
alias k='kubectl'
```

Este alias te permitirá usar `k` en lugar de escribir `kubectl` completo.

### 4. Guardar y cerrar

* Presiona `Ctrl + X` para salir.
* Pulsa `Y` para confirmar guardar.
* Presiona `Enter` para sobrescribir.

### 5. Aplicar los cambios

Ejecuta el siguiente comando para recargar la configuración del archivo `.bashrc`:

```bash
source ~/.bashrc
```

---

## 🚀 Verificar que funciona

Prueba si el alias funciona ejecutando:

```bash
k version
```

Deberías ver la misma salida que con:

```bash
kubectl version
```

---

## 📌 Consideraciones adicionales

* Este alias solo estará disponible cuando uses **Git Bash**.
* Si abres una nueva terminal de Git Bash, el alias seguirá disponible.
* Puedes agregar otros alias útiles como:

```bash
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kaf='kubectl apply -f'
```

Para agregarlos, simplemente edítalos en el mismo archivo `.bashrc`.

---

## 🧽 Deshacer los cambios

Si deseas eliminar el alias más adelante, simplemente:

1. Ejecuta `nano ~/.bashrc`.
2. Elimina o comenta (`#`) la línea `alias k='kubectl'`.
3. Guarda y cierra el archivo.
4. Ejecuta `source ~/.bashrc`.

---

Con esto, tu experiencia con `kubectl` en Git Bash será más fluida y productiva 🚀
