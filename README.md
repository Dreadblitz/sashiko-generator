# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Descripción del Proyecto

**Sashiko Generator** es una aplicación web interactiva para generar patrones japoneses tradicionales de bordado Sashiko. La aplicación permite crear, personalizar y exportar patrones geométricos a PDF para uso en proyectos de bordado. Incluye un editor avanzado para crear patrones personalizados con herramientas de dibujo completas.

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
    patterns: { ... }           // Metadatos de patrones (incluye personalizados)
}
```

#### 2. Estado de la Aplicación (`state`)
Maneja todas las propiedades del patrón actual:
- `currentPattern`: Patrón seleccionado (asanoha, seigaiha, custom_*)
- `patternSize`: Tamaño del patrón individual (5-100mm)
- `spacingX/Y`: Espaciado horizontal/vertical independiente (5-75mm)
- `strokeWidth`: Grosor de línea en mm (0.1-3.0mm)
- `dashGap`: Separación de líneas segmentadas en mm (0.1-10mm)
- `workAreaWidth/Height`: Dimensiones del área de trabajo en cm
- `offsetX/Y`: Desplazamiento del patrón para arrastre

#### 3. Sistema de Patrones (`patterns`)
Cada patrón es una función que recibe:
```javascript
function patternName(ctx, x, y, size, strokeWidth, color, dashGap)
```

**Patrones predefinidos:**
- **Asanoha**: Hexágonos con líneas radiales internas
- **Seigaiha**: Arcos semicirculares concéntricos con offset alternado
- **Shippo**: Círculos entrelazados de la armonía
- **Yabane**: Plumas de flecha tradicionales
- **Uroko**: Escamas de dragón triangulares

**Patrones personalizados:**
- Almacenados en localStorage con ID único (`custom_timestamp`)
- Recreados dinámicamente desde formas guardadas
- Escalables y editables a través del editor modal

#### 4. Editor de Patrones Avanzado
Sistema completo de diseño modal con:

**Herramientas de dibujo:**
- **Línea**: Dibujo directo de líneas rectas
- **Círculo**: Círculos por centro y radio
- **Arco**: Proceso de 3 clics (centro → radio → ángulo final)
- **Lápiz**: Dibujo a mano alzada con optimización de puntos
- **Borrador**: Eliminación precisa con detección de colisión avanzada
- **Limpiar**: Borrado completo del canvas

**Estado del editor (`editorState`):**
```javascript
{
    currentTool: 'line',
    arcStep: 0,                    // 0=centro, 1=radio, 2=ángulos
    arcCenter: {x: 0, y: 0},       // Centro del arco en construcción
    arcRadius: 0,                  // Radio calculado
    arcStartAngle: 0,              // Ángulo inicial del arco
    drawnShapes: [],               // Formas completadas
    customPatterns: []             // Patrones guardados en localStorage
}
```

**Detección de colisión del borrador:**
- **Líneas**: Distancia punto-segmento con proyección
- **Círculos**: Detección de perímetro con tolerancia
- **Arcos**: Muestreo de puntos a lo largo de la curva
- **Trazos libres**: Verificación segmento por segmento

#### 5. Motor de Renderizado
- **Canvas principal**: Visualización escalada para la UI
- **Canvas temporal**: Renderizado en resolución completa para PDF
- **Canvas del editor**: 300x300px para diseño de patrones
- **Sistema de grilla**: Calcula posiciones con soporte para patrones offset
- **Optimización**: Solo renderiza elementos visibles + buffer expandido

#### 6. Sistema de Interacción
- **Arrastre de patrones**: Modifica `offsetX/Y` del estado con feedback visual
- **Controles independientes**: Espaciado X/Y separado para máxima flexibilidad
- **Tamaños de papel**: 7 formatos predefinidos + modo personalizado
- **Límite de tamaño**: Patrones de 5mm a 100mm (aumentado desde 50mm)

#### 7. Persistencia y Gestión
- **LocalStorage**: Almacena patrones personalizados como JSON
- **Carga automática**: Restaura patrones al inicializar la aplicación
- **Eliminación**: Botón de eliminar para patrones personalizados únicamente
- **Validación**: Verificación de nombre y contenido antes de guardar

#### 8. Exportación PDF
- **jsPDF**: Generación de documentos en dimensiones exactas
- **Alta resolución**: Canvas temporal con conversión mm→px precisa
- **Integración completa**: Soporte para todos los tipos de patrones
- **Nomenclatura**: Archivos con formato `sashiko-{patrón}-{dimensiones}.pdf`

## Flujo de Desarrollo

### Para abrir y probar:
```bash
# Abrir directamente en navegador
open sashiko-generator.html
# o usar servidor local
python -m http.server 8000
```

### Para modificar patrones predefinidos:
1. Agregar entrada en `CONFIG.patterns` con metadatos
2. Implementar función de patrón en objeto `patterns`
3. Agregar opción en HTML dentro del selector de patrones

### Para crear patrones personalizados:
1. Usar botón "✏️ Dibujar Patrón" en la interfaz
2. Seleccionar herramienta de dibujo en el modal
3. Crear formas en el canvas 300x300px
4. Asignar nombre y guardar
5. El patrón aparece automáticamente en el selector

### Para agregar tamaños de papel:
1. Definir dimensiones en `CONFIG.paperSizes`
2. Agregar `<option>` en el selector HTML

## Consideraciones Técnicas

### Sistema de Coordenadas
- **Unidades de entrada**: mm y cm (interfaz de usuario)
- **Unidades de cálculo**: píxeles (canvas)
- **Conversiones**: Usa constantes de CONFIG para mantener consistencia
- **Escalado**: Editor usa escala fija, patrones se adaptan al tamaño final

### Renderizado Eficiente
- Calcula rango expandido para cubrir área visible + offset
- Solo dibuja elementos dentro del área visible + buffer
- Usa `setLineDash()` para simular puntadas de bordado
- Optimización de trazos libres por distancia mínima

### Responsividad del Canvas
- Mantiene aspect ratio del área de trabajo
- Escala display máximo de 600px
- Canvas temporal usa resolución completa para PDF
- Editor modal responsivo para móviles

### Gestión de Estado
- Estado centralizado en objeto `state`
- Estado del editor separado en `editorState`
- Re-renderizado automático en cambios de estado
- Event listeners conectados a elementos de UI específicos

### Algoritmos de Colisión
- **Punto-línea**: Proyección vectorial con parámetros t
- **Punto-círculo**: Distancia euclidiana con tolerancia
- **Punto-arco**: Muestreo adaptativo basado en radio
- **Optimización**: Salida temprana en detección positiva

## Dependencias Externas

- **jsPDF**: Generación de documentos PDF (CDN)
- **Google Fonts**: Fuente Inter para tipografía moderna
- **Canvas API**: Renderizado de patrones (nativo del navegador)
- **LocalStorage**: Persistencia de patrones personalizados (nativo)

## Funcionalidades Clave Implementadas

### ✅ Herramienta de Arco
- Proceso de 3 clics: centro → radio → ángulo final
- Preview en tiempo real con guías visuales
- Integración completa con sistema de borrado
- Soporte para escalado en patrones finales

### ✅ Límite de Tamaño Aumentado
- Rango de patrones: 5mm - 100mm (anteriormente 50mm)
- Slider actualizado con nueva escala
- Validación en tiempo real

### ✅ Controles de Espaciado Independientes
- Espaciado horizontal y vertical separado
- Rango: 5mm - 75mm para cada eje
- Flexibilidad total en diseño de grillas

### ✅ Sistema de Patrones Personalizados
- Editor modal completo con 6 herramientas
- Persistencia automática en localStorage
- Gestión de eliminación para patrones custom
- Escalado automático desde canvas 300x300 a tamaño final

### ✅ Borrador Inteligente
- Detección precisa para todas las formas geométricas
- Algoritmos optimizados para cada tipo de trazo
- Círculo visual de borrado con feedback inmediato