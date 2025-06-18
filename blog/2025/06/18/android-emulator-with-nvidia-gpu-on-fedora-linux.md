# Ejecutar el emulador de Android con soporte de aceleración gráfica por hardware usando una GPU NVIDIA dedicada

Ejecutar el emulador de Android con soporte de aceleración gráfica por hardware usando una GPU NVIDIA dedicada (RTX 4000 Ada) con Vulkan en Fedora Linux, sin configurar variables de entorno globales que puedan romper otros programas.

* * * * *

🖥️ Entorno del sistema
-----------------------

-   **Sistema operativo**: Fedora Linux (Xorg, no Wayland)

-   **CPU**: Intel i9 vPro

-   **RAM**: 64 GB

-   **GPU**: NVIDIA RTX 4000 Ada Generation (Laptop)

-   **Driver NVIDIA**: Propietario, instalado correctamente

-   **Mesa**: Instalado por defecto (proporciona Intel/llvmpipe fallback)

* * * * *

⚙️ Problemas encontrados
------------------------

### 1\. `vulkaninfo` no detectaba la GPU NVIDIA

```bash
vulkaninfo | grep deviceName
deviceName = Intel(R) Graphics (RPL-S)
deviceName = llvmpipe (LLVM 20.1.5, 256 bits)
```

### 2\. El emulador fallaba con `VK_ERROR_INCOMPATIBLE_DRIVER`

```bash
failed to open device /dev/dri/renderD129 (VK_ERROR_INCOMPATIBLE_DRIVER)
```

### 3\. Android Studio no levantaba correctamente el emulador

-   Se congelaba o no mostraba gráficos acelerados.

-   O el emulador crasheaba (`Segmentation fault`) al no encontrar `libX11` o `libGL`.

* * * * *

✅ Solución definitiva
---------------------

### 1\. Evitar usar variables de entorno globales en `.bashrc`

Al establecer estas variables globalmente, algunos programas como Android Studio o el escritorio pueden comportarse de forma errática.

```bash
# ❌ No poner en ~/.bashrc:
export __NV_PRIME_RENDER_OFFLOAD=1
export __GLX_VENDOR_LIBRARY_NAME=nvidia
```
### 2\. Crear script personalizado para lanzar el emulador

```bash
#!/bin/bash

# ------------------------------------------------------------------------------
# Android Emulator Launcher Script (Vulkan + NVIDIA GPU Acceleration)
# ------------------------------------------------------------------------------
#
# This script launches an Android emulator with Vulkan support and NVIDIA GPU
# hardware acceleration. A device name must be provided as the first parameter.
#
# Usage:
#   ./launch_emulator.sh <device_name>
#
# Example:
#   ./launch_emulator.sh Pixel_8_Pro
#
# Environment Notes:
# - VK_ICD_FILENAMES and VK_LAYER_PATH must point to the correct NVIDIA Vulkan files.
# - __NV_PRIME_RENDER_OFFLOAD and __GLX_VENDOR_LIBRARY_NAME are **not set globally**
#   here because we have observed system-level issues when they are exported system-wide.
#   Instead, we pass them inline to the emulator command.
#
# Verification:
#   glxinfo | grep "OpenGL renderer"
#     → should return: OpenGL renderer string: NVIDIA RTX 4000 Ada ...
#   vulkaninfo | grep deviceName
#     → should return: deviceName = NVIDIA RTX 4000 Ada Generation Laptop GPU
# ------------------------------------------------------------------------------

# Fail if no device name is provided
if [ -z "$1" ]; then
  echo "❌ Error: No AVD device name provided."
  echo "Usage: $0 <device_name>"
  echo "Example: $0 Pixel_8_Pro"
  exit 1
fi
deviceName="$1"

# Path to emulator tool
EMULATOR_PATH="$HOME/Android/Sdk/emulator/emulator"

# Check if emulator tool exists
if [ ! -x "$EMULATOR_PATH" ]; then
  echo "❌ Error: Emulator not found at: $EMULATOR_PATH"
  exit 1
fi

# Validate that the given AVD exists
if ! "$EMULATOR_PATH" -list-avds | grep -q "^${deviceName}$"; then
  echo "❌ Error: AVD device '$deviceName' not found."
  echo "Available devices:"
  "$EMULATOR_PATH" -list-avds
  exit 1
fi

# Vulkan environment configuration for NVIDIA driver
export VK_ICD_FILENAMES="/usr/share/vulkan/icd.d/nvidia_icd.x86_64.json"
export VK_LAYER_PATH="/usr/share/vulkan/implicit_layer.d"

# Launch emulator using Vulkan + NVIDIA GPU acceleration
# Avoid exporting GPU offload variables globally due to known system conflicts
__NV_PRIME_RENDER_OFFLOAD=1 \
__GLX_VENDOR_LIBRARY_NAME=nvidia \
"$EMULATOR_PATH" \
  -avd "$deviceName" \
  -gpu host \
  -verbose

```

> Guarda esto como `launch_emulator.sh`, hazlo ejecutable: `chmod +x launch_emulator.sh`, y luego lánzalo con `./launch_emulator.sh`.

* * * * *

🧩 Dependencias necesarias
--------------------------

Asegúrate de tener instalados:

```bash
sudo dnf install vulkan-loader vulkan-tools vulkan-validation-layers
sudo dnf install libX11 libX11-devel libGL libGLU libglvnd libglvnd-devel
```
**Opcional** (para ver info de Vulkan):

bash

CopiarEditar

`vulkaninfo`

* * * * *

🧪 Cómo verificar que funciona
------------------------------

```bash
vulkaninfo | grep deviceName
# debe mostrar -> deviceName = NVIDIA RTX 4000 Ada Generation Laptop GPU
```

Solo para probar, se puede setear las variables de forma global y ver si el rendered cambia adecuadamente
```bash
export __NV_PRIME_RENDER_OFFLOAD=1
export __GLX_VENDOR_LIBRARY_NAME=nvidia

glxinfo | grep "OpenGL renderer"
# debe mostrar -> OpenGL renderer string: NVIDIA RTX 4000 Ada Generation Laptop GPU/PCIe/SSE2
```

En la salida del emulador veriamos algo asi:

```bash
INFO | Selecting Vulkan device: NVIDIA RTX 4000 Ada Generation Laptop GPU
```

* * * * *

🧰 Notas extra
--------------

-   Puedes lanzar el emulador desde terminal y **seguir usándolo desde Android Studio o IntelliJ** para debuggear Flutter o apps nativas.

-   Android Studio puede usar el emulador que ya esté corriendo si está correctamente registrado como AVD.

-   Si usas `flutter`, asegúrate de que detecte el emulador con:

```bash
flutter devices
```