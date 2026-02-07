Script Module – Android / ADB Shell

Descripción

Este módulo es un script autónomo ejecutado directamente en Android shell (/system/bin/sh). No utiliza framework, daemon ni sistema de módulos. La ejecución y los efectos dependen exclusivamente del entorno ADB / shell.

---

1. Alcance del módulo

· Ejecución directa en Android (adb shell)
· Sin dependencias externas
· Sin persistencia automática
· Control total del sistema según permisos del shell
· Reversión manual mediante script dedicado

---

2. Estructura de archivos

```
module/
├── Man.sh
└── uninstall.sh
```

Restricción: No se permiten archivos adicionales. Toda la lógica debe residir en estos dos scripts.

---

3. Intérprete obligatorio

Ambos archivos DEBEN iniciar con:

```bash
#!/system/bin/sh
```

Motivo:

· Android no garantiza /bin/sh
· BusyBox puede no estar presente
· /system/bin/sh es el intérprete estándar AOSP

---

4. Resolución del directorio del módulo

Todo script DEBE definir su ruta absoluta de ejecución:

```bash
# Directorio raíz del módulo
MODULE_DIR="$(cd "$(dirname "$0")" && pwd)"
```

Justificación técnica:

1. Evita fallos por cwd variable
2. Permite ejecución desde cualquier contexto:
   · adb shell
   · llamadas indirectas
   · enlaces simbólicos
3. Garantiza rutas determinísticas

IMPORTANTE: Toda referencia a archivos internos debe derivar de $MODULE_DIR.

---

5. Script principal (Man.sh)

Responsabilidades

· Aplicar modificaciones al sistema
· Crear archivos, enlaces o configuraciones
· Ejecutar comandos privilegiados
· Registrar cualquier cambio que requiera reversión

Restricciones

· No debe asumir estado previo del sistema
· No debe depender de variables de entorno externas
· No debe usar rutas relativas sin $MODULE_DIR

Ejemplo de encabezado mínimo válido

```bash
#!/system/bin/sh

MODULE_DIR="$(cd "$(dirname "$0")" && pwd)"

# A partir de este punto, el script opera sin restricciones,
# bajo el modelo de ADB Shell
```

---

6. Script de reversión (uninstall.sh)

Responsabilidades

· Revertir todas las acciones realizadas por Man.sh
· Eliminar archivos creados
· Restaurar configuraciones modificadas
· Dejar el sistema en estado previo

Reglas

1. No debe ejecutar lógica nueva
2. No debe depender de que Man.sh esté presente
3. Debe ser idempotente (ejecución múltiple sin efectos colaterales)

Encabezado obligatorio

```bash
#!/system/bin/sh
```

---

7. Modelo de ejecución

Ejemplo desde host

```bash
# Aplicar cambios
adb shell
cd /ruta/del/modulo
sh Man.sh

# Revertir cambios
sh uninstall.sh
```

No se soporta

· Ejecución automática
· Hooks de sistema
· Eventos de arranque

---

8. Consideraciones de seguridad

· El script se ejecuta con los privilegios del shell activo
· No existe sandbox
· Errores pueden causar:
  · Corrupción de archivos
  · Bootloops
  · Pérdida de datos

ADVERTENCIA: El autor asume responsabilidad total sobre los efectos del script.

---

9. Buenas prácticas técnicas

1. Uso explícito de set -e o control manual de errores
2. Comprobación de existencia antes de borrar ([ -e ])
3. Evitar rm -rf sin validación
4. Mantener simetría entre Man.sh y uninstall.sh

---

10. Licencia

Definir explícitamente el tipo de licencia si se distribuye el módulo.

---

Niveles de evolución sugeridos

Definir contrato entre Man.sh y uninstall.sh

· Establecer formato de registro de cambios
· Especificar estructura de metadatos
· Definir protocolo de comunicación entre scripts

Añadir sistema de registro de cambios

· Log estructurado de operaciones
· Backup automático de archivos modificados
· Sistema de versionado de cambios

Endurecer ejecución (checks de entorno)

· Verificación de permisos
· Validación de versión de Android
· Check de espacio disponible
· Verificación de integridad del script

Adaptarlo como base para otros sistemas

· Magisk Module Template
· Shizuku Script Base
· Rootless Implementation
· KernelSU Module Framework

---

NOTA: Este documento establece las especificaciones técnicas mínimas para un módulo de script autónomo en Android. Cualquier implementación debe respetar estas reglas para garantizar compatibilidad y seguridad.
