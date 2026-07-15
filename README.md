# 📄 METAPDFeditor

**Herramienta zero-install** para visualizar, editar y sobreescribir metadatos de PDFs desde el navegador.

## ✨ Características

- **Drag & drop** de PDFs individuales o carpetas completas
- **Metadatos completos**: estándar (Título, Autor, Asunto, Keywords, Creador, Productor, Fechas) + campos personalizados + XMP + Page Labels
- **Edición inline** de cualquier campo directamente en la tabla
- **Export individual** — descarga un PDF con metadatos modificados
- **Export masivo** — ZIP con todos los PDFs modificados
- **Filtrado** de archivos por nombre
- **Detección de cambios** — badge visual por archivo modificado
- **Reset** — vuelve a los metadatos originales con un clic
- **100% client-side** — sin servidor, sin instalación, funciona con doble clic

## 🚀 Uso

1. Abre `index.html` en tu navegador (doble clic)
2. Arrastra PDFs o carpetas a la zona de carga
3. Haz clic en un archivo para ver/editar sus metadatos
4. Modifica los campos que quieras
5. Haz clic en **📥 Descargar PDF modificado** o **📥 Exportar todo (ZIP)**

## 🛠️ Stack

| Librería | Uso |
|----------|-----|
| [pdf-lib](https://pdf-lib.js.org/) | Leer/escribir metadatos PDF |
| [pdf.js](https://mozilla.github.io/pdf.js/) | XMP metadata, page labels, page count |
| [JSZip](https://stuk.github.io/jszip/) | Empaquetar ZIP descargable |

## 📋 Metadatos soportados

### Estándar (editables)
- Título (`/Title`)
- Autor (`/Author`)
- Asunto (`/Subject`)
- Palabras Clave (`/Keywords`)
- Fecha Creación (`/CreationDate`)
- Fecha Modificación (`/ModDate`)

### Solo lectura (generados por el sistema)
- Creador (`/Creator`) — aplicación que creó el original
- Productor (`/Producer`) — aplicación que generó el PDF

### Extra
- Campos personalizados del Info dictionary
- XMP Metadata (Dublin Core, PDF/X, etc.)
- Page Labels (i, ii, 1, 2, 3...)

## ⚠️ Limitaciones

- No modifica contenido de páginas (solo metadatos)
- No procesa PDFs protegidos con contraseña
- PDFs > 50MB pueden ser lentos
- XMP metadata es solo lectura
- Las firmas digitales existentes se pierden al re-encode

## 📝 Licencia

MIT

---

Hecho con ❤️ por David Antizar
