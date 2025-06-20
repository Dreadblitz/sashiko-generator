# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Descripción del Proyecto

**Sashiko Generator** es una aplicación web interactiva para generar patrones japoneses tradicionales de bordado Sashiko. La aplicación permite crear, personalizar y exportar patrones geométricos a PDF para uso en proyectos de bordado.

## Arquitectura de la Aplicación

### Estructura Monolítica
- **Archivo principal**: `sashiko-generator.html` - Aplicación completa en un solo archivo HTML
- **Carpeta de respaldos**: `backups/` - Contiene versiones anteriores del generador

### Componentes Principales

#### 1. Sistema de Configuración Global (`CONFIG`)
```javascript
const CONFIG = {
    DPI: 96,
    CM_TO_PX: 37.795275591,    // Conversión 96 DPI
    MM_TO_PX: 3.7795275591,    // Conversión mm a píxeles
    paperSizes: { ... },        // Tamaños de papel predefinidos
    patterns: { ... }           // Metadatos de patrones
}
```

#### 2. Estado de la Aplicación (`state`)
Maneja todas las propiedades del patrón actual:
- `currentPattern`: Patrón seleccionado (asanoha, seigaiha)
- `scale`: Escala del patrón en mm
- `strokeWidth`: Grosor de línea en px
- `dashGap`: Separación de líneas segmentadas en mm
- `workAreaWidth/Height`: Dimensiones del área de trabajo en cm
- `offsetX/Y`: Desplazamiento del patrón para arrastre

#### 3. Sistema de Patrones (`patterns`)
Cada patrón es una función que recibe:
```javascript
function patternName(ctx, x, y, size, strokeWidth, color, dashGap)
```

**Patrones implementados:**
- **Asanoha**: Hexágonos con líneas radiales internas
- **Seigaiha**: Arcos semicirculares concéntricos con offset alternado

#### 4. Motor de Renderizado
- **Canvas principal**: Visualización escalada para la UI
- **Canvas temporal**: Renderizado en resolución completa para PDF
- **Sistema de grilla**: Calcula posiciones con soporte para patrones offset
- **Optimización**: Solo renderiza elementos visibles + buffer

#### 5. Sistema de Interacción
- **Arrastre de patrones**: Modifica `offsetX/Y` del estado
- **Controles en tiempo real**: Sliders y inputs actualizan el estado y re-renderizan
- **Tamaños de papel**: Predefinidos + modo personalizado

#### 6. Exportación PDF
- Usa jsPDF para crear documentos en las dimensiones exactas
- Renderiza patrón completo en canvas temporal de alta resolución
- Convierte canvas a imagen PNG y la embebe en PDF

## Flujo de Desarrollo

### Para abrir y probar:
```bash
# Abrir directamente en navegador
open sashiko-generator.html
# o usar servidor local
python -m http.server 8000
```

### Para modificar patrones:
1. Agregar entrada en `CONFIG.patterns` con metadatos
2. Implementar función de patrón en objeto `patterns`
3. Agregar opción en HTML dentro de `.pattern-selector`

### Para agregar tamaños de papel:
1. Definir dimensiones en `CONFIG.paperSizes`
2. Agregar `<option>` en el selector HTML

## Consideraciones Técnicas

### Sistema de Coordenadas
- **Unidades de entrada**: mm y cm (interfaz de usuario)
- **Unidades de cálculo**: píxeles (canvas)
- **Conversiones**: Usa constantes de CONFIG para mantener consistencia

### Renderizado Eficiente
- Calcula rango expandido para cubrir área visible + offset
- Solo dibuja elementos dentro del área visible + buffer
- Usa `setLineDash()` para simular puntadas de bordado

### Responsividad del Canvas
- Mantiene aspect ratio del área de trabajo
- Escala display máximo de 600px
- Canvas temporal usa resolución completa para PDF

### Gestión de Estado
- Estado centralizado en objeto `state`
- Re-renderizado automático en cambios de estado
- Event listeners conectados a elementos de UI específicos

## Dependencias Externas

- **jsPDF**: Generación de documentos PDF (CDN)
- **Google Fonts**: Fuente Inter para tipografía moderna
- **Canvas API**: Renderizado de patrones (nativo del navegador)