# Conversión Word a Markdown - Guía Completa
## Migrar Documentación Existente sin Perder Información

> **Objetivo:** Convertir documentos Word (.docx) a Markdown (.md) de forma automatizada  
> **Herramienta principal:** Pandoc (el estándar de facto)  
> **Tiempo:** 5-10 min por documento manual, segundos si automatizas

---

## 📋 Índice

1. [Herramientas Recomendadas](#1-herramientas-recomendadas)
2. [Pandoc - Setup e Instalación](#2-pandoc-setup)
3. [Conversión Básica](#3-conversión-básica)
4. [Conversión por Lotes](#4-conversión-por-lotes)
5. [Manejo de Elementos Complejos](#5-elementos-complejos)
6. [Scripts de Automatización](#6-scripts-de-automatización)
7. [Validación Post-Conversión](#7-validación)
8. [Azure DevOps Integration](#8-azure-devops-integration)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. Herramientas Recomendadas

### Comparación Rápida

| Herramienta | Calidad | Velocidad | Batch | Gratis | Recomendado |
|-------------|---------|-----------|-------|--------|-------------|
| **Pandoc** | ⭐⭐⭐⭐⭐ | ⚡⚡⚡ | ✅ | ✅ | ✅ **SÍ** |
| Writage | ⭐⭐⭐⭐ | ⚡⚡ | ❌ | ✅ | Para pocos docs |
| Online converters | ⭐⭐⭐ | ⚡ | ❌ | ✅ | Emergencias |
| Word2md | ⭐⭐⭐ | ⚡⚡ | ✅ | ✅ | Alternativa |
| Mammoth | ⭐⭐⭐⭐ | ⚡⚡ | ✅ | ✅ | Para Python |

**Recomendación: Pandoc** - Es el más completo, potente y automatizable

---

## 2. Pandoc - Setup e Instalación

### 2.1 Instalación por Sistema Operativo

#### Windows

**Opción 1: Instalador oficial (Recomendado)**
```powershell
# Descargar instalador
# https://github.com/jgm/pandoc/releases/latest
# pandoc-X.XX-windows-x86_64.msi

# Instalar haciendo doble click

# Verificar instalación
pandoc --version
```

**Opción 2: Chocolatey**
```powershell
# Si tienes Chocolatey instalado
choco install pandoc

# Verificar
pandoc --version
```

**Opción 3: Winget**
```powershell
# Windows 11 / Windows 10 con App Installer
winget install --id=JohnMacFarlane.Pandoc -e

# Verificar
pandoc --version
```

#### macOS

```bash
# Con Homebrew (recomendado)
brew install pandoc

# O descargar el .pkg desde
# https://github.com/jgm/pandoc/releases/latest

# Verificar
pandoc --version
```

#### Linux (Ubuntu/Debian)

```bash
# Desde repositorio (puede ser versión vieja)
sudo apt update
sudo apt install pandoc

# O versión más reciente desde GitHub
wget https://github.com/jgm/pandoc/releases/download/3.1.11/pandoc-3.1.11-1-amd64.deb
sudo dpkg -i pandoc-3.1.11-1-amd64.deb

# Verificar
pandoc --version
```

### 2.2 Verificar Instalación

```bash
# Debe mostrar versión 2.x o superior
pandoc --version

# Expected output:
# pandoc 3.1.11
# Features: +server +lua
# ...
```

---

## 3. Conversión Básica

### 3.1 Comando Simple

**Convertir un archivo:**

```bash
# Sintaxis básica
pandoc input.docx -o output.md

# Ejemplo real
pandoc "Arquitectura del Sistema.docx" -o arquitectura-sistema.md
```

**¡Listo! Ya tienes tu Markdown** ✅

### 3.2 Conversión Mejorada

**Con opciones recomendadas:**

```bash
pandoc input.docx -o output.md \
  --wrap=none \
  --extract-media=./images \
  --markdown-headings=atx \
  --standalone
```

**Explicación de opciones:**
- `--wrap=none`: No cortar líneas (mejor para Git)
- `--extract-media=./images`: Extraer imágenes a carpeta
- `--markdown-headings=atx`: Usar `#` en lugar de underlines
- `--standalone`: Documento completo con metadata

### 3.3 Ejemplo Completo

```bash
# Crear carpeta para imágenes
mkdir -p images

# Convertir con todas las opciones
pandoc "Mi Documento Técnico.docx" \
  -f docx \
  -t markdown \
  -o mi-documento-tecnico.md \
  --wrap=none \
  --extract-media=./images \
  --markdown-headings=atx \
  --standalone \
  --toc

# Opciones adicionales:
# -f docx: Formato origen explícito
# -t markdown: Formato destino explícito
# --toc: Generar tabla de contenidos
```

### 3.4 Preview Antes de Convertir

```bash
# Ver cómo quedará sin crear archivo
pandoc input.docx -t markdown --wrap=none | head -50

# Útil para verificar formato antes de conversión masiva
```

---

## 4. Conversión por Lotes (Automatizada)

### 4.1 Script para Windows (PowerShell)

```powershell
# convert-all-docx.ps1
# Convierte todos los .docx en carpeta actual a .md

param(
    [string]$InputFolder = ".",
    [string]$OutputFolder = "./markdown",
    [string]$ImagesFolder = "./images"
)

# Crear carpetas si no existen
New-Item -ItemType Directory -Force -Path $OutputFolder | Out-Null
New-Item -ItemType Directory -Force -Path $ImagesFolder | Out-Null

# Obtener todos los archivos .docx
$docxFiles = Get-ChildItem -Path $InputFolder -Filter "*.docx" -Recurse

$total = $docxFiles.Count
$current = 0

Write-Host "🔄 Encontrados $total archivos .docx"
Write-Host ""

foreach ($file in $docxFiles) {
    $current++
    $percentage = [math]::Round(($current / $total) * 100)
    
    # Nombre del archivo de salida
    $outputName = [System.IO.Path]::GetFileNameWithoutExtension($file.Name) + ".md"
    $outputPath = Join-Path $OutputFolder $outputName
    
    Write-Host "[$current/$total] ($percentage%) Converting: $($file.Name)"
    
    # Convertir con pandoc
    pandoc $file.FullName `
        -f docx `
        -t markdown `
        -o $outputPath `
        --wrap=none `
        --extract-media=$ImagesFolder `
        --markdown-headings=atx `
        --standalone
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "  ✅ Success: $outputName"
    } else {
        Write-Host "  ❌ Error: $outputName" -ForegroundColor Red
    }
    Write-Host ""
}

Write-Host "🎉 Conversión completa!"
Write-Host "📁 Archivos markdown en: $OutputFolder"
Write-Host "🖼️  Imágenes en: $ImagesFolder"
```

**Uso:**

```powershell
# Convertir todos los .docx en carpeta actual
.\convert-all-docx.ps1

# Especificar carpetas
.\convert-all-docx.ps1 -InputFolder "C:\docs" -OutputFolder "C:\markdown"

# Con imágenes en otra ubicación
.\convert-all-docx.ps1 -ImagesFolder "C:\images"
```

---

### 4.2 Script para Linux/macOS (Bash)

```bash
#!/bin/bash
# convert-all-docx.sh
# Convierte todos los .docx a .md

INPUT_DIR="${1:-.}"
OUTPUT_DIR="${2:-./markdown}"
IMAGES_DIR="${3:-./images}"

# Crear carpetas
mkdir -p "$OUTPUT_DIR"
mkdir -p "$IMAGES_DIR"

# Contador
total=$(find "$INPUT_DIR" -name "*.docx" -type f | wc -l)
current=0

echo "🔄 Encontrados $total archivos .docx"
echo ""

# Convertir cada archivo
find "$INPUT_DIR" -name "*.docx" -type f | while read -r docx_file; do
    current=$((current + 1))
    percentage=$((current * 100 / total))
    
    # Nombre del archivo de salida
    filename=$(basename "$docx_file" .docx)
    output_file="$OUTPUT_DIR/${filename}.md"
    
    echo "[$current/$total] ($percentage%) Converting: $(basename "$docx_file")"
    
    # Convertir con pandoc
    pandoc "$docx_file" \
        -f docx \
        -t markdown \
        -o "$output_file" \
        --wrap=none \
        --extract-media="$IMAGES_DIR" \
        --markdown-headings=atx \
        --standalone
    
    if [ $? -eq 0 ]; then
        echo "  ✅ Success: ${filename}.md"
    else
        echo "  ❌ Error: ${filename}.md"
    fi
    echo ""
done

echo "🎉 Conversión completa!"
echo "📁 Archivos markdown en: $OUTPUT_DIR"
echo "🖼️  Imágenes en: $IMAGES_DIR"
```

**Uso:**

```bash
# Dar permisos de ejecución
chmod +x convert-all-docx.sh

# Convertir todos los .docx en carpeta actual
./convert-all-docx.sh

# Especificar carpetas
./convert-all-docx.sh /path/to/docs /path/to/markdown /path/to/images

# Convertir y ver progreso en tiempo real
./convert-all-docx.sh | tee conversion.log
```

---

### 4.3 Script Python (Cross-platform)

```python
#!/usr/bin/env python3
"""
convert_docx_to_md.py
Convierte archivos .docx a Markdown usando Pandoc
Requiere: pip install tqdm
"""

import os
import subprocess
import sys
from pathlib import Path
from tqdm import tqdm

def convert_docx_to_md(docx_path, output_dir, images_dir):
    """Convierte un archivo .docx a .md"""
    
    docx_path = Path(docx_path)
    output_dir = Path(output_dir)
    images_dir = Path(images_dir)
    
    # Crear carpetas si no existen
    output_dir.mkdir(parents=True, exist_ok=True)
    images_dir.mkdir(parents=True, exist_ok=True)
    
    # Nombre del archivo de salida
    output_file = output_dir / f"{docx_path.stem}.md"
    
    # Comando pandoc
    cmd = [
        'pandoc',
        str(docx_path),
        '-f', 'docx',
        '-t', 'markdown',
        '-o', str(output_file),
        '--wrap=none',
        f'--extract-media={images_dir}',
        '--markdown-headings=atx',
        '--standalone'
    ]
    
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, check=True)
        return True, output_file
    except subprocess.CalledProcessError as e:
        return False, str(e)

def find_docx_files(directory):
    """Encuentra todos los archivos .docx"""
    directory = Path(directory)
    return list(directory.rglob("*.docx"))

def main():
    # Argumentos
    input_dir = sys.argv[1] if len(sys.argv) > 1 else "."
    output_dir = sys.argv[2] if len(sys.argv) > 2 else "./markdown"
    images_dir = sys.argv[3] if len(sys.argv) > 3 else "./images"
    
    print("🔍 Buscando archivos .docx...")
    docx_files = find_docx_files(input_dir)
    
    if not docx_files:
        print("❌ No se encontraron archivos .docx")
        sys.exit(1)
    
    print(f"✅ Encontrados {len(docx_files)} archivos")
    print("")
    
    # Convertir con barra de progreso
    successful = 0
    failed = 0
    
    for docx_file in tqdm(docx_files, desc="Convirtiendo", unit="archivo"):
        success, result = convert_docx_to_md(docx_file, output_dir, images_dir)
        
        if success:
            successful += 1
            tqdm.write(f"✅ {docx_file.name} → {result.name}")
        else:
            failed += 1
            tqdm.write(f"❌ {docx_file.name} - Error: {result}")
    
    # Resumen
    print("")
    print("=" * 50)
    print(f"🎉 Conversión completa!")
    print(f"✅ Exitosos: {successful}")
    if failed > 0:
        print(f"❌ Fallidos: {failed}")
    print(f"📁 Markdown en: {output_dir}")
    print(f"🖼️  Imágenes en: {images_dir}")
    print("=" * 50)

if __name__ == "__main__":
    main()
```

**Instalación y uso:**

```bash
# Instalar dependencia (opcional, para barra de progreso)
pip install tqdm

# Dar permisos
chmod +x convert_docx_to_md.py

# Convertir
python3 convert_docx_to_md.py

# Con argumentos
python3 convert_docx_to_md.py /path/to/docs /path/to/markdown /path/to/images
```

---

## 5. Manejo de Elementos Complejos

### 5.1 Tablas

**Word → Markdown:**

Word table:
```
| Header 1 | Header 2 |
|----------|----------|
| Cell 1   | Cell 2   |
```

**Pandoc lo convierte automáticamente** ✅

**Tablas complejas (merged cells):**
```bash
# Usar formato pipe_tables (mejor para tablas complejas)
pandoc input.docx -o output.md --columns=200

# Si tabla es muy compleja, Pandoc usa HTML <table>
# Esto es OK, Markdown soporta HTML
```

### 5.2 Imágenes

**Pandoc extrae imágenes automáticamente:**

```bash
# Extraer a carpeta específica
pandoc input.docx -o output.md --extract-media=./images

# Resultado en Markdown:
# ![](images/image1.png)
```

**Renombrar imágenes automáticamente:**

```python
# rename_images.py
import re
from pathlib import Path

def rename_images_in_md(md_file):
    """Renombra imágenes a nombres más descriptivos"""
    
    content = Path(md_file).read_text(encoding='utf-8')
    
    # Buscar imágenes: ![alt](path)
    def replace_image(match):
        alt_text = match.group(1)
        old_path = match.group(2)
        
        # Generar nuevo nombre desde alt text
        if alt_text:
            new_name = re.sub(r'[^\w\s-]', '', alt_text.lower())
            new_name = re.sub(r'[-\s]+', '-', new_name)
            ext = Path(old_path).suffix
            new_path = f"images/{new_name}{ext}"
            
            # Renombrar archivo físico
            old_file = Path(old_path)
            new_file = Path(new_path)
            if old_file.exists():
                old_file.rename(new_file)
            
            return f"![{alt_text}]({new_path})"
        
        return match.group(0)
    
    # Reemplazar en contenido
    new_content = re.sub(r'!\[(.*?)\]\((.*?)\)', replace_image, content)
    
    # Guardar
    Path(md_file).write_text(new_content, encoding='utf-8')
    print(f"✅ Imágenes renombradas en {md_file}")

# Uso
rename_images_in_md("output.md")
```

### 5.3 Estilos y Formato

**Preservados automáticamente:**

- ✅ **Negrita:** `**texto**`
- ✅ **Cursiva:** `*texto*`
- ✅ **Código inline:** `` `código` ``
- ✅ **Listas:** `- item` o `1. item`
- ✅ **Headings:** `# H1`, `## H2`, etc.
- ✅ **Links:** `[texto](url)`
- ✅ **Citas:** `> quote`

**Colores y fuentes específicas:**
- ❌ Se pierden (Markdown no soporta)
- ℹ️ No es problema para docs técnicos

### 5.4 Código y Bloques de Código

**Si usaste estilo "Código" en Word:**

```bash
# Pandoc lo detecta automáticamente
```python
def example():
    return "Hello"
```

**Si no:**
- Revisa manualmente
- Busca secciones de código
- Añade ``` manualmente

### 5.5 Notas al Pie

**Word:** Nota al pie automática  
**Markdown:** Pandoc convierte a `[^1]`

```markdown
Texto con nota[^1]

[^1]: Contenido de la nota
```

### 5.6 Metadatos del Documento

**Extraer metadatos:**

```bash
# Pandoc puede preservar metadata en YAML front matter
pandoc input.docx -o output.md \
  --standalone \
  --metadata-file=metadata.yaml

# output.md tendrá:
# ---
# title: Título del documento
# author: Autor
# date: 2024-03-15
# ---
```

---

## 6. Scripts de Automatización Avanzada

### 6.1 Script con Post-Processing

```bash
#!/bin/bash
# convert-and-clean.sh
# Convierte y limpia Markdown automáticamente

INPUT_DOCX="$1"
OUTPUT_MD="${INPUT_DOCX%.docx}.md"

echo "📄 Convirtiendo: $INPUT_DOCX"

# 1. Convertir con Pandoc
pandoc "$INPUT_DOCX" \
  -f docx \
  -t markdown \
  -o "$OUTPUT_MD" \
  --wrap=none \
  --extract-media=./images \
  --markdown-headings=atx \
  --standalone

# 2. Post-processing con sed/awk
echo "🔧 Limpiando formato..."

# Eliminar líneas vacías múltiples
sed -i '/^$/N;/^\n$/D' "$OUTPUT_MD"

# Asegurar espacio antes de headers
sed -i 's/^#/\n#/g' "$OUTPUT_MD"

# Convertir espacios múltiples a uno
sed -i 's/  */ /g' "$OUTPUT_MD"

# 3. Aplicar markdownlint automáticamente
if command -v markdownlint &> /dev/null; then
    echo "✨ Aplicando markdownlint..."
    markdownlint --fix "$OUTPUT_MD"
fi

echo "✅ Listo: $OUTPUT_MD"
```

### 6.2 Script con Validación

```python
#!/usr/bin/env python3
"""
convert_and_validate.py
Convierte y valida que no se perdió información
"""

import subprocess
import sys
from pathlib import Path
import zipfile
import xml.etree.ElementTree as ET

def count_words_in_docx(docx_path):
    """Cuenta palabras en .docx"""
    try:
        with zipfile.ZipFile(docx_path) as docx:
            xml_content = docx.read('word/document.xml')
            tree = ET.fromstring(xml_content)
            
            # Namespace
            ns = {'w': 'http://schemas.openxmlformats.org/wordprocessingml/2006/main'}
            
            # Extraer texto
            texts = []
            for paragraph in tree.findall('.//w:p', ns):
                para_text = []
                for text in paragraph.findall('.//w:t', ns):
                    if text.text:
                        para_text.append(text.text)
                if para_text:
                    texts.append(' '.join(para_text))
            
            full_text = ' '.join(texts)
            return len(full_text.split())
    except Exception as e:
        print(f"⚠️  No se pudo contar palabras en .docx: {e}")
        return None

def count_words_in_md(md_path):
    """Cuenta palabras en .md"""
    content = Path(md_path).read_text(encoding='utf-8')
    # Eliminar código markdown para contar solo texto real
    import re
    # Remover code blocks
    content = re.sub(r'```.*?```', '', content, flags=re.DOTALL)
    # Remover inline code
    content = re.sub(r'`[^`]+`', '', content)
    # Remover markdown syntax
    content = re.sub(r'[#*\[\]()!]', '', content)
    
    return len(content.split())

def convert_docx(docx_path, output_path):
    """Convierte .docx a .md"""
    cmd = [
        'pandoc',
        str(docx_path),
        '-f', 'docx',
        '-t', 'markdown',
        '-o', str(output_path),
        '--wrap=none',
        '--extract-media=./images',
        '--markdown-headings=atx',
        '--standalone'
    ]
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.returncode == 0

def main():
    if len(sys.argv) < 2:
        print("Uso: python convert_and_validate.py input.docx [output.md]")
        sys.exit(1)
    
    docx_path = Path(sys.argv[1])
    output_path = Path(sys.argv[2]) if len(sys.argv) > 2 else Path(docx_path.stem + '.md')
    
    print(f"📄 Convirtiendo: {docx_path.name}")
    
    # Contar palabras en original
    original_words = count_words_in_docx(docx_path)
    if original_words:
        print(f"📊 Palabras en original: {original_words}")
    
    # Convertir
    if not convert_docx(docx_path, output_path):
        print("❌ Error en conversión")
        sys.exit(1)
    
    print(f"✅ Convertido a: {output_path.name}")
    
    # Contar palabras en markdown
    md_words = count_words_in_md(output_path)
    print(f"📊 Palabras en markdown: {md_words}")
    
    # Validar
    if original_words and md_words:
        diff_percent = abs(original_words - md_words) / original_words * 100
        
        if diff_percent < 5:
            print(f"✅ Validación OK (diferencia: {diff_percent:.1f}%)")
        elif diff_percent < 15:
            print(f"⚠️  Revisar (diferencia: {diff_percent:.1f}%)")
        else:
            print(f"❌ Posible pérdida de contenido (diferencia: {diff_percent:.1f}%)")
    
    print("")
    print("🔍 Revisa manualmente:")
    print("  - Tablas complejas")
    print("  - Imágenes")
    print("  - Formato especial")

if __name__ == "__main__":
    main()
```

---

## 7. Validación Post-Conversión

### 7.1 Checklist Manual

**Para cada documento convertido, verificar:**

```
□ Headings preservados (H1, H2, H3, etc.)
□ Listas (bullets y numeradas) correctas
□ Tablas renderizadas correctamente
□ Imágenes extraídas y vinculadas
□ Links funcionando
□ Código en bloques de código
□ Negrita/cursiva preservadas
□ No hay texto perdido (comparar word count)
```

### 7.2 Script de Validación Automática

```bash
#!/bin/bash
# validate-conversion.sh
# Valida que la conversión fue exitosa

MD_FILE="$1"

echo "🔍 Validando: $MD_FILE"
echo ""

# 1. Check que archivo existe y no está vacío
if [ ! -s "$MD_FILE" ]; then
    echo "❌ Archivo vacío o no existe"
    exit 1
fi

echo "✅ Archivo existe y tiene contenido"

# 2. Check headings
HEADINGS=$(grep -c "^#" "$MD_FILE")
echo "📑 Headings encontrados: $HEADINGS"

# 3. Check imágenes
IMAGES=$(grep -c "!\[.*\](.*)" "$MD_FILE")
echo "🖼️  Imágenes: $IMAGES"

# 4. Check tablas
TABLES=$(grep -c "^|" "$MD_FILE")
echo "📊 Filas de tabla: $TABLES"

# 5. Check links
LINKS=$(grep -c "\[.*\](.*)" "$MD_FILE")
echo "🔗 Links: $LINKS"

# 6. Check bloques de código
CODE_BLOCKS=$(grep -c "^```" "$MD_FILE")
echo "💻 Bloques de código: $((CODE_BLOCKS / 2))"

# 7. Word count
WORDS=$(wc -w < "$MD_FILE")
echo "📝 Palabras: $WORDS"

# 8. Run markdownlint
if command -v markdownlint &> /dev/null; then
    echo ""
    echo "🔍 Ejecutando markdownlint..."
    if markdownlint "$MD_FILE"; then
        echo "✅ Markdown válido"
    else
        echo "⚠️  Hay warnings de formato"
    fi
fi

echo ""
echo "✅ Validación completa"
```

### 7.3 Comparación Visual

**Comparar side-by-side:**

```bash
# Generar HTML desde Markdown para comparar
pandoc output.md -o output.html

# Abrir ambos en browser
# Word (Export as PDF) vs output.html
```

**O usar herramienta de diff:**

```bash
# Convertir ambos a texto plano y comparar
pandoc input.docx -t plain -o original.txt
pandoc output.md -t plain -o converted.txt

# Diff
diff original.txt converted.txt

# O con herramienta visual
code --diff original.txt converted.txt  # VS Code
```

---

## 8. Azure DevOps Integration

### 8.1 Pipeline de Conversión Automática

**Opción: Convertir docs cuando se suben al repo**

```yaml
# azure-pipelines-convert.yml
# Pipeline que convierte .docx a .md automáticamente

trigger:
  paths:
    include:
      - 'word-docs/*.docx'

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: UsePythonVersion@0
    inputs:
      versionSpec: '3.x'
    displayName: 'Setup Python'
  
  - bash: |
      # Instalar pandoc
      sudo apt update
      sudo apt install -y pandoc
      pandoc --version
    displayName: 'Install Pandoc'
  
  - bash: |
      # Crear carpetas
      mkdir -p markdown-output
      mkdir -p images
      
      # Convertir todos los .docx
      find word-docs -name "*.docx" -type f | while read docx; do
        filename=$(basename "$docx" .docx)
        echo "Converting: $filename"
        
        pandoc "$docx" \
          -f docx \
          -t markdown \
          -o "markdown-output/${filename}.md" \
          --wrap=none \
          --extract-media=./images \
          --markdown-headings=atx \
          --standalone
      done
      
      echo "Conversión completa"
    displayName: 'Convert DOCX to Markdown'
  
  - bash: |
      # Copiar archivos convertidos al repo
      cp -r markdown-output/* ./docs/
      
      # Configurar Git
      git config user.email "pipeline@azuredevops.com"
      git config user.name "Azure Pipeline"
      
      # Commit y push
      git add docs/
      git add images/
      git commit -m "Auto-convert: Update Markdown from DOCX [skip ci]"
      git push origin HEAD:$(Build.SourceBranchName)
    displayName: 'Commit Converted Files'
    condition: and(succeeded(), ne(variables['Build.Reason'], 'PullRequest'))
```

**Setup:**

1. Crear carpeta `word-docs/` en repo
2. Usuarios suben .docx ahí
3. Pipeline convierte automáticamente
4. Resultado aparece en `docs/`

### 8.2 Pull Request con Conversión

**Workflow:**

```
Usuario → Sube .docx a branch
       → Crea PR
       → Pipeline convierte a .md
       → Reviewer ve el .md
       → Aprobar y merge
```

---

## 9. Troubleshooting

### 9.1 Problemas Comunes

#### "Pandoc not found"

```bash
# Verificar instalación
which pandoc  # Linux/Mac
where pandoc  # Windows

# Si no está, reinstalar
# Ver Sección 2.1
```

#### "Permission denied"

```bash
# Linux/Mac: Dar permisos al script
chmod +x convert-all-docx.sh

# Windows: Ejecutar PowerShell como Admin
```

#### "Failed to extract images"

```bash
# Asegurar que carpeta existe
mkdir -p images

# Dar permisos de escritura
chmod 755 images
```

#### "Table looks broken"

```bash
# Tablas muy complejas → Usar más ancho
pandoc input.docx -o output.md --columns=200

# O acepta HTML table
# Markdown soporta HTML nativo
```

#### "Caracteres raros (�)"

```bash
# Problema de encoding
pandoc input.docx -o output.md --from docx+smart

# O especificar UTF-8
iconv -f ISO-8859-1 -t UTF-8 output.md > output-utf8.md
```

### 9.2 Archivos Muy Grandes

**Para .docx > 50MB:**

```bash
# Aumentar memoria de Pandoc
pandoc input.docx -o output.md +RTS -M4G -RTS

# O dividir el documento primero en Word
```

### 9.3 Conversión Parcial

**Si solo quieres ciertas secciones:**

```bash
# Convertir y extraer secciones con awk
pandoc input.docx -t markdown | awk '/^## Sección 2/,/^## Sección 3/' > section2.md
```

---

## 10. Herramientas Alternativas

### 10.1 Writage (Plugin de Word)

**Pros:**
- ✅ Plugin dentro de Word
- ✅ UI familiar
- ✅ Vista previa en tiempo real

**Contras:**
- ❌ Solo manual, no batch
- ❌ Solo Windows/Mac

**Instalación:**
```
https://www.writage.com/
Descargar → Instalar → Restart Word
File → Save As → Markdown
```

### 10.2 Mammoth (Python)

**Para integrar en scripts Python:**

```python
# Instalar
pip install mammoth

# Uso
import mammoth

with open("input.docx", "rb") as docx_file:
    result = mammoth.convert_to_markdown(docx_file)
    markdown = result.value
    
    with open("output.md", "w") as md_file:
        md_file.write(markdown)
```

**Pros:**
- ✅ Fácil integración Python
- ✅ Control programático

**Contras:**
- ❌ Menos features que Pandoc
- ❌ No extrae imágenes automáticamente

### 10.3 Online Converters

**Solo para emergencias:**

- CloudConvert: https://cloudconvert.com/docx-to-md
- Zamzar: https://www.zamzar.com/
- ConvertIO: https://convertio.co/

**Contras:**
- ❌ No batch
- ❌ Upload a terceros (security concern)
- ❌ Límites de tamaño

---

## 11. Workflow Recomendado

### 11.1 Para Pocos Documentos (<10)

```bash
# 1. Crear carpeta de trabajo
mkdir conversion
cd conversion
mkdir images

# 2. Copiar .docx
cp path/to/*.docx .

# 3. Convertir uno por uno
for file in *.docx; do
    pandoc "$file" -o "${file%.docx}.md" \
        --wrap=none \
        --extract-media=./images \
        --markdown-headings=atx
done

# 4. Revisar manualmente cada uno
# 5. Ajustar formato si necesario
# 6. Copiar a repo Git
```

### 11.2 Para Muchos Documentos (>10)

```bash
# 1. Usar script de batch (Sección 4)
./convert-all-docx.sh

# 2. Validar con script (Sección 7)
for md in markdown/*.md; do
    ./validate-conversion.sh "$md"
done

# 3. Spot check (revisar algunos manualmente)
# 4. Aplicar markdownlint
markdownlint --fix markdown/*.md

# 5. Copiar a repo Git
```

### 11.3 Flujo Continuo

**Para docs que se actualizan frecuentemente:**

```
1. Editar en Word (formato familiar)
2. Guardar .docx en carpeta watched
3. Script automático convierte
4. Push a Git repo
5. CI/CD valida Markdown
```

**Setup:**

```bash
# Con inotify (Linux) o fswatch (Mac)
# watch-and-convert.sh

while inotifywait -e modify word-docs/*.docx; do
    ./convert-all-docx.sh word-docs markdown
    git add markdown/
    git commit -m "Auto-convert: $(date)"
    git push
done
```

---

## 12. Best Practices

### ✅ DO

- **Usar Pandoc** - Es el estándar de facto
- **Extraer imágenes** - Siempre usa `--extract-media`
- **Validar después** - No asumas que salió bien
- **Revisar manualmente** - Al menos spot check
- **Versionar en Git** - Commit antes y después de convertir
- **Mantener originales** - Backup de .docx por si acaso
- **Nombres descriptivos** - Renombrar archivos a kebab-case
- **Aplicar linting** - markdownlint después de convertir

### ❌ DON'T

- **No borrar .docx originales** - Mantener como backup
- **No confiar ciegamente** - Siempre validar
- **No usar converters online** - Para docs confidenciales
- **No ignorar warnings** - Si Pandoc advierte algo, revisar
- **No mezclar formatos** - Decide: ¿Word o Markdown?
- **No sobre-editar Word** - Formato simple convierte mejor

### 📝 Preparar Word para Mejor Conversión

**Antes de convertir, en Word:**

1. **Usar estilos estándar** (Heading 1, 2, 3, etc.)
2. **Tablas simples** (sin merged cells si es posible)
3. **Imágenes inline** (no flotantes)
4. **Links válidos** (no texto que parece link)
5. **Código en estilo Código** (no solo Courier)

---

## 13. Resumen Ejecutivo

### Método Recomendado

```bash
# 1. Instalar Pandoc (una vez)
# Ver Sección 2.1

# 2. Convertir (por cada archivo o batch)
pandoc input.docx -o output.md \
  --wrap=none \
  --extract-media=./images \
  --markdown-headings=atx \
  --standalone

# 3. Validar
./validate-conversion.sh output.md

# 4. Limpiar
markdownlint --fix output.md

# 5. Commit a Git
git add output.md images/
git commit -m "docs: Migrate from Word"
```

### Tiempo Estimado

| Tarea | Tiempo |
|-------|--------|
| Instalar Pandoc | 5 min (una vez) |
| Convertir 1 doc | 10 segundos |
| Convertir 100 docs (batch) | 5 min |
| Validar 1 doc | 2 min |
| Ajustar formato 1 doc | 5-10 min |

**Total para 10 docs:** ~1 hora  
**Total para 100 docs:** ~3-4 horas

---

## 14. Checklist de Migración

### Preparación
- [ ] Pandoc instalado y verificado
- [ ] Scripts preparados (batch, validación)
- [ ] Carpeta de trabajo creada
- [ ] Backup de archivos originales

### Conversión
- [ ] Todos los .docx convertidos a .md
- [ ] Imágenes extraídas a carpeta
- [ ] Nombres de archivo normalizados
- [ ] Encoding UTF-8 verificado

### Validación
- [ ] Spot check manual (10% de archivos)
- [ ] Scripts de validación ejecutados
- [ ] Markdownlint aplicado
- [ ] Word count comparado (±10% es OK)

### Ajuste
- [ ] Tablas complejas revisadas
- [ ] Imágenes renombradas si necesario
- [ ] Links verificados
- [ ] Código en bloques correctos

### Finalización
- [ ] Archivos en estructura correcta de repo
- [ ] Committed a Git
- [ ] Pipeline CI/CD valida correctamente
- [ ] .docx originales archivados

---

## 🎉 Conclusión

**Pandoc es tu mejor amigo para esta tarea.**

**Resumen:**
- ✅ Instala Pandoc (5 min)
- ✅ Usa scripts de batch (Sección 4)
- ✅ Valida automáticamente (Sección 7)
- ✅ Ajusta manualmente lo necesario
- ✅ Migra en pocas horas, no días

**Para tu caso específico:**

Si tienes **10-20 documentos:**
→ Conversión manual uno por uno (1-2 horas)

Si tienes **50-100 documentos:**
→ Script de batch + validación automática (3-4 horas)

Si tienes **500+ documentos:**
→ Batch + Azure Pipeline automático (1 día setup, luego automático)

**¡Pandoc hace el 95% del trabajo por ti!** 🚀

---

**Versión:** 1.0  
**Última actualización:** 2024-03-15  
**Herramienta principal:** Pandoc 3.x  
**Status:** ✅ Production Ready
