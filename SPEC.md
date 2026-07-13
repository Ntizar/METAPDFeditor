# VisorMetadatosPDF — SPEC

## Visión
Herramienta HTML client-side zero-install para visualizar, editar y sobreescribir metadatos de PDFs. Arrastra un PDF o carpeta completa → tabla editable → descarga PDFs con metadatos modificados.

## Alcance

### Sí hace
- Drag & drop de PDFs individuales o carpetas completas
- Extracción y visualización de TODOS los metadatos del PDF en tabla editable
- Campos estándar: Título, Autor, Asunto, Palabras Clave, Creador, Productor, Fecha Creación, Fecha Modificación
- Campos extra: todos los campos adicionales que detecte el PDF (custom metadata)
- XMP fields: campos XMP embebidos en el PDF
- Page labels: etiquetas de páginas si existen
- Edición inline de cualquier campo directamente en la tabla
- Sobreescribir PDF individual con metadatos modificados (descarga)
- Descarga en lote: ZIP con todos los PDFs modificados
- Vista previa: nombre de archivo, tamaño, número de páginas
- Indicador de cambios sin guardar por archivo
- Búsqueda/filtrado de archivos en la tabla
- Contador de archivos procesados / con cambios

### NO hace (non-goals)
- No modifica contenido de páginas (solo metadatos)
- No hace OCR ni extracción de texto
- No tiene backend ni servidor
- No tiene login ni usuarios
- No tiene modo offline (usa CDN para librerías)
- No procesa PDFs protegidos con contraseña
- No genera PDFs desde cero
- No soporta arrastrar archivos no-PDF (los ignora con aviso)

## Pantallas

1. **Pantalla principal**: Drag-drop zone + tabla de metadatos + barra de herramientas
2. **Sin state adicional**: todo en una pantalla, sin tabs ni navegación

## Datos

| Fuente | Tipo | Actualización | Volumen |
|--------|------|---------------|---------|
| PDFs del usuario | File API / drag-drop | On-demand | 1-500 archivos |
| Metadatos extraídos | pdf-lib en memoria | On-demand | 8-50 campos por PDF |

## Arquitectura

### Stack técnico
| Librería | CDN | Uso |
|----------|-----|-----|
| **pdf-lib** | cdn.jsdelivr.net/npm/pdf-lib@1.17.1/dist/pdf-lib.min.js | Leer/escribir metadatos PDF |
| **pdf.js** | cdn.jsdelivr.net/npm/pdfjs-dist@3.11.174/build/pdf.min.js | Info extendida (page labels, XMP) |
| **JSZip** | cdn.jsdelivr.net/npm/jszip@3.10.1/dist/jszip.min.js | Empaquetar ZIP descargable |

### Capas (dentro de un solo HTML)

| Capa | Sección JS | Responsabilidad |
|------|-----------|----------------|
| Config | `CONFIG` | Constantes, mapeo de campos, estilos |
| Estado | `STATE` | Array de archivos, metadatos, cambios pendientes |
| Parser | `parsePDF()` | Extrae metadatos completos de un PDF via pdf-lib + pdf.js |
| Editor | `renderTable()` | Genera tabla editable con todos los campos |
| Export | `exportPDF()` / `exportZIP()` | Sobreescribe metadatos y genera descarga |
| UI | `initDragDrop()` | Drag-drop, filtrado, contadores, eventos |

### Metadatos extraídos

**Campos estándar PDF (pdf-lib):**
- `/Title` → Título
- `/Author` → Autor
- `/Subject` → Asunto
- `/Keywords` → Palabras Clave
- `/Creator` → Creador (aplicación que creó el original)
- `/Producer` → Productor (aplicación que generó el PDF)
- `/CreationDate` → Fecha de Creación
- `/ModDate` → Fecha de Última Modificación

**Campos extra (pdf-lib raw metadata):**
- Todos los campos del dictionary de metadatos que no sean los 8 estándar
- Se muestran como "Campo personalizado: valor"

**XMP metadata (pdf.js):**
- Metadatos XMP embebidos (Dublin Core, PDF/X, etc.)
- Se parsean como pares clave-valor

**Page Labels (pdf.js):**
- Etiquetas de páginas si existen (ej: "i, ii, iii, 1, 2, 3")

### Estado global

```javascript
window.STATE = {
    files: [
        {
            id: 'uuid',
            file: File,                    // Referencia al archivo original
            name: 'documento.pdf',         // Nombre del archivo
            size: 1234567,                 // Tamaño en bytes
            pageCount: 5,                  // Número de páginas
            metadata: {
                standard: { title: '', author: '', subject: '', keywords: '', creator: '', producer: '', creationDate: '', modDate: '' },
                custom: { 'campo1': 'valor1', ... },
                xmp: { 'dc:title': '...', ... },
                pageLabels: ['i', 'ii', '1', '2', '3']
            },
            originalMetadata: { ... },     // Copia para detectar cambios
            hasChanges: false,             // ¿Ha sido editado?
            status: 'loaded'               // loaded | exporting | exported | error
        }
    ],
    filter: '',                           // Filtro de búsqueda
    selectedCount: 0                      // Archivos seleccionados para export
};
```

### Interfaces entre módulos

```
initDragDrop() → handleFiles() → parsePDF() → addToState() → renderTable()
renderTable() → editField() → updateState() → markChanged()
exportPDF() → applyMetadata() → downloadBlob()
exportZIP() → JSZip → downloadBlob()
```

## Diseño visual (Kaizen)

- **Colores**: Azul #2563eb (primario), Naranja #f97316 (acentos), Gris #f8fafc (fondo)
- **Fondo**: Blanco #ffffff para la tabla
- **Tipografía**: Inter o system-ui
- **Tabla**: Bordes sutiles, hover highlight, editable cells con borde azul al focus
- **Drag-drop zone**: Borde punteado grande, icono de carpeta, texto "Arrastra PDFs aquí"
- **Botones**: Primario azul (exportar), Secundario gris (filtrar), Peligro rojo (eliminar)
- **Badges**: Contador de archivos, cambios pendientes, estado
- **Responsive**: Tabla scrollable horizontal en móvil

## Criterios de éxito
- [ ] Arrastra 1 PDF → tabla visible en < 1s
- [ ] Arrastra carpeta con 50 PDFs → tabla completa en < 5s
- [ ] Edita un campo → badge de "cambios" visible
- [ ] Exporta PDF → descarga con metadatos modificados
- [ ] Exporta ZIP → todos los PDFs modificados en un ZIP
- [ ] Funciona desde file:// (doble clic, sin servidor)
- [ ] Sin errores en consola

## Anti-patrones (lo que evitamos)
- NO usar pdf.js para escribir metadatos (pdf.js solo lee)
- NO usar pdf-lib para leer XMP (pdf-lib no soporta XMP completo)
- NO usar CDN sin fallback (si CDN falla, aviso claro)
- NO procesar PDFs > 50MB sin aviso previo
- NO sobreescribir el archivo original (siempre descargar nuevo)
- NO usar `const charts = {}` pattern (usar `var` o `window.`)

## Referencias
- **browser-local-tools** → patrón HTML autocontenido, pdf-lib para metadatos
- **PDFtoMeta** → proyecto anterior de David para metadatos PDF
- **pdf-lib docs**: https://pdf-lib.js.org/
- **pdf.js docs**: https://mozilla.github.io/pdf.js/

## Estructura de archivos

```
VisorMetadatosPDF/
├── SPEC.md              ← Este archivo
├── index.html           ← HTML completo autocontenido
└── README.md            ← Docs del proyecto
```

**Nota:** Al ser una herramienta single-page, TODO va en un solo `index.html` autocontenido. Las librerías se embeben inline para que funcione offline y sea compartible.
