---
outline: deep
---

# Guía de Operación de Migración de Datos

Esta página cubre el flujo de trabajo práctico para la migración de directorios de datos. Para detalles de implementación técnica, consulte [Implementación Básica](/es/datamigrae/baseinfo).

## Encontrar Directorios de Datos Asociados a Aplicaciones

1. Cambie a la pestaña "Directorios de Datos" en la ventana principal de AppPorts
2. El panel izquierdo muestra todas las aplicaciones instaladas
3. Haga clic en una aplicación; el panel derecho muestra sus directorios de datos asociados bajo `~/Library/`

AppPorts escanea automáticamente los siguientes directorios, haciendo coincidir por Bundle ID o nombre de la aplicación:

| Ruta de Escaneo | Método de Coincidencia |
|-----------------|----------------------|
| `~/Library/Application Support/` | Bundle ID o nombre de app |
| `~/Library/Preferences/` | Bundle ID o nombre de app |
| `~/Library/Containers/` | Bundle ID |
| `~/Library/Group Containers/` | Bundle ID |
| `~/Library/Caches/` | Bundle ID o nombre de app |
| `~/Library/WebKit/` | Bundle ID |
| `~/Library/HTTPStorages/` | Bundle ID |
| `~/Library/Application Scripts/` | Bundle ID |
| `~/Library/Logs/` | Nombre de app |
| `~/Library/Saved Application State/` | Nombre de app |

## Directorios de Herramientas (Dot-Folders)

AppPorts puede detectar automáticamente dot-folders creados por herramientas de desarrollo comunes en el directorio home del usuario:

1. Cambie a la subpestaña "Directorios de Herramientas" en la pestaña Directorios de Datos
2. La página lista todos los directorios de herramientas detectados con sus tamaños
3. Cada directorio muestra un marcador de prioridad (recommended/optional) y estado

Si falta un directorio de herramienta local pero la ubicación canónica del almacenamiento externo seleccionado todavía contiene un directorio gestionado por AppPorts, el elemento aparece como "Necesita Revinculación". Al cambiar de almacenamiento externo, AppPorts vuelve a escanear los directorios de herramientas y actualiza este estado. Los archivos normales no se tratan como directorios revinculables.

Para la lista completa soportada, consulte [Detección de Directorios de Herramientas](/es/datamigrae/tools).

## Migración de Directorios (Carpetas Personalizadas)

La pestaña "Migración de Directorios" migra carpetas de usuario arbitrarias. Es útil para proyectos grandes, modelos, bibliotecas de recursos o cachés de herramientas que desea mover al almacenamiento externo.

1. Cambie a "Migración de Directorios" en la ventana principal
2. Haga clic en el botón "+" del encabezado "Carpetas Locales"
3. Elija la carpeta local a migrar y luego el directorio raíz de destino en el almacenamiento externo
4. AppPorts usa `raíz destino/nombre de carpeta local` como destino externo, guarda la configuración e inicia la migración

Para evitar copias recursivas, migración de directorios del sistema o tomar control de una ruta incorrecta, se aplican estas comprobaciones:

- La carpeta local debe estar dentro del directorio home del usuario actual y no puede ser todo el home
- La ruta local y sus rutas superiores no deben ser enlaces simbólicos
- La carpeta local no debe solaparse con un directorio de datos o entrada de migración ya gestionada
- La raíz de destino externa debe ser una carpeta y no debe estar dentro del home del usuario actual
- El destino externo final no debe estar dentro de la carpeta local, y la carpeta local no debe estar dentro del destino externo final

Después de la migración, el panel local muestra el estado de la ruta original y el panel externo muestra el estado de la copia externa. Seleccione elementos en el panel externo para "Revincular carpeta" o "Restaurar carpeta". Quitar una configuración solo la elimina de la lista de migración; no borra datos reales automáticamente.

## Operaciones de Migración

### Migración de Directorio Individual

1. Encuentre el directorio a migrar en la lista de directorios de datos
2. Haga clic en el botón "Migrar" a la derecha
3. AppPorts ejecuta los siguientes pasos:
   - Copia el directorio al almacenamiento externo
   - Escribe metadatos de enlace gestionado
   - Elimina el directorio local original
   - Crea un enlace simbólico

### Re-firmado Automático

Cuando "Re-firmado automático" está habilitado en la configuración, la migración del directorio de datos activa automáticamente el firmado para la app asociada:

1. **Antes de la migración**: Respalda la firma original de la **ruta real externa** de la app asociada (no el shell local)
2. **Después de la migración**: Ejecuta re-firmado Ad-hoc en la **app real externa** (modo silencioso; los fallos no muestran diálogo)

Para apps vinculadas, AppPorts resuelve automáticamente la ruta real de la app detrás del shell Stub Portal o el enlace simbólico, asegurando que los cambios de firma se apliquen al paquete de aplicación real en lugar de un shell local inválido.

::: tip 💡 No se requiere acción manual
Con el re-firmado automático habilitado, el flujo de trabajo de migración del directorio de datos está completamente automatizado. El respaldo de firma y el re-firmado ambos apuntan a la ruta real de la app — no se requiere intervención manual.
:::

### Contexto de Logs

Las operaciones de directorio de datos (migración, restauración, normalización, re-vinculación) incluyen automáticamente información de contexto de la app asociada en los logs:

| Campo | Descripción |
|-------|-------------|
| `app_name` | Nombre de la app asociada |
| `app_status` | Estado de la app (Vinculada, Local, etc.) |
| `app_is_resigned` | Si la app ha sido re-firmada |
| `app_bundle_id` | Bundle ID de la app (leído de la ruta real) |
| `app_real_path` | Ruta real externa de la app |

Estos campos ayudan a localizar problemas con más precisión al exportar paquetes de diagnóstico.

### Migración por Lotes

1. Marque múltiples directorios en la lista de directorios de herramientas
2. Haga clic en el botón "Migración por Lotes" en la parte inferior
3. AppPorts ejecuta la migración secuencialmente

::: tip 💡 Recomendaciones de Prioridad
Los directorios de datos se clasifican en tres niveles de prioridad:

- **Crítico** (`critical`): Debe funcionar después de la migración; afecta la funcionalidad principal de la aplicación
- **Recomendado** (`recommended`): Gran ahorro de espacio; alto beneficio de migración
- **Opcional** (`optional`): Tamaño pequeño o reconstruible

Se recomienda priorizar la migración de directorios marcados como "Recomendado".
:::

## Operaciones de Restauración

1. Encuentre el directorio migrado en la lista de directorios de datos (estado: "Vinculado")
2. Haga clic en el botón "Restaurar" a la derecha
3. AppPorts ejecuta los siguientes pasos:
   - Elimina el enlace simbólico local
   - Copia los datos del almacenamiento externo de vuelta a local
   - Elimina el directorio externo (mejor esfuerzo)

## Manejo de Estados Anormales

### Necesita Normalización

El directorio es gestionado por AppPorts, pero la ruta externa no está en la ubicación canónica. Haga clic en "Normalizar"; AppPorts moverá los datos externos a la ruta canónica y reconstruirá el enlace simbólico.

### Necesita Revinculación

El directorio de datos aún existe en el almacenamiento externo, pero el enlace simbólico local se perdió. Haga clic en "Revincular"; AppPorts recreará el enlace simbólico. La revinculación solo se aplica cuando el destino externo sigue siendo un directorio. Si un archivo normal ocupa el destino externo, AppPorts detiene la operación y conserva el archivo.

### Enlace Suave Existente

Un enlace simbólico creado por el usuario, no por AppPorts. Puede elegir "Tomar Control"; AppPorts escribirá metadatos de enlace gestionado y lo gestionará en adelante.

## Vista de Árbol

Para directorios de datos que contienen subdirectorios (ej., múltiples directorios de aplicaciones bajo `Application Support`), AppPorts proporciona una vista de agrupación en árbol:

- El directorio principal muestra flechas de expandir/colapsar a la izquierda
- Los subdirectorios muestran indentación jerárquica
- Cada nodo muestra independientemente el tamaño y estado
- Las operaciones de migración/restauración pueden realizarse en subdirectorios individuales
