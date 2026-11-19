# Guía del Debugger en Neovim (DAP)

Esta documentación detalla los atajos de teclado, las acciones y el flujo de trabajo configurado para depurar proyectos de **C/C++** (usando `codelldb`) y **C# / .NET** (usando `netcoredbg`) en Neovim.

---

## ⌨️ Atajos de Teclado (Keymaps)

Las siguientes teclas están configuradas globalmente para controlar la sesión de depuración:

| Atajo | Acción (DAP) | Descripción |
| :--- | :--- | :--- |
| `<F5>` | `dap.continue` | Inicia la depuración, reanuda la ejecución tras un punto de interrupción o continúa hasta el siguiente. |
| `<F10>` | `dap.step_over` | Ejecuta la siguiente línea de código (salta por encima de las funciones sin entrar en ellas). |
| `<F11>` | `dap.step_into` | Entra dentro de la función ubicada en la línea actual para inspeccionar su ejecución. |
| `<F12>` | `dap.step_out` | Sale de la función actual y regresa al contexto de la función que la llamó. |
| `<Leader>b` | `dap.toggle_breakpoint` | Añade o quita un punto de interrupción (*breakpoint*) en la línea actual del cursor. |
| `<Leader>du` | `dapui.toggle` | Abre o cierra manualmente la interfaz gráfica flotante del depurador (`nvim-dap-ui`). |

---

## 🖥️ Interfaz Gráfica (DAP UI) y Ventanas

La interfaz de usuario se gestiona mediante `nvim-dap-ui` y `nvim-dap-virtual-text`.

* **Apertura y Cierre Automático:** La interfaz se abrirá automáticamente al iniciar o conectar un proceso de depuración y se cerrará por completo al finalizar o salir del programa.
* **Texto Virtual:** Las variables actuales y sus valores se mostrarán directamente en línea como texto virtual junto a tu código fuente mientras depuras.

---

## 🛠️ Guía de Uso por Lenguaje

### 1. C / C++ (`codelldb`)
1. Abre cualquier archivo con extensión `.c` o `.cpp`.
2. Coloca un punto de interrupción con `<Leader>b`.
3. Presiona `<F5>`.
4. Introduce o confirma la ruta del binario compilado cuando el sistema te lo solicite en la línea de comandos de Neovim.

### 2. C# / .NET (`netcoredbg`)
1. Abre cualquier archivo con extensión `.cs`.
2. Coloca un punto de interrupción con `<Leader>b` en la línea de código que desees analizar.
3. Presiona `<F5>`.
4. **Compilación opcional:** El sistema te preguntará si deseas compilar el proyecto (`s/n`). Si indicas que sí, ejecutará `dotnet build` automáticamente.
5. **Ruta del DLL:** Te solicitará la ruta del archivo `.dll` compilado (por defecto sugerirá la carpeta `bin/Debug/`). Presiona `Enter` para aceptar la ruta sugerida.