# Materiales de Presentación - Estandarización de Documentación IT

## 📦 Contenido del Package

Has recibido 3 archivos para tu presentación de 30 minutos:

1. **10-presentation-slides.md** (39 slides)
   - Presentación completa en formato Marp
   - Listo para convertir a PowerPoint/PDF
   - 30 minutos con Q&A incluido

2. **11-speaker-notes-guide.md** (Guía completa del presentador)
   - Speaker notes slide-by-slide
   - Timing detallado (qué decir, cuándo)
   - Manejo de objeciones
   - Tips de presentación
   - Post-presentación checklist

3. **12-executive-summary-handout.md** (One-pager)
   - Resumen de 1 página para entregar
   - Puede imprimirse como handout
   - También funciona como leave-behind

---

## 🎨 Cómo Convertir las Slides a PowerPoint

### Opción 1: Usando Marp CLI (Recomendado)

**Instalar Marp:**
```bash
# Via npm
npm install -g @marp-team/marp-cli

# O via Homebrew (macOS)
brew install marp-cli
```

**Convertir a PowerPoint:**
```bash
# Generar PPTX
marp 10-presentation-slides.md --pptx -o presentation.pptx

# O generar PDF
marp 10-presentation-slides.md --pdf -o presentation.pdf

# O generar HTML (para presentar desde browser)
marp 10-presentation-slides.md -o presentation.html
```

**Resultado:** Archivo `.pptx` listo para abrir en PowerPoint/Google Slides

---

### Opción 2: Usando Marp for VS Code (Visual)

1. **Instalar VS Code** (si no lo tienes)
   - https://code.visualstudio.com/

2. **Instalar extensión Marp:**
   - Abrir VS Code
   - Extensions (Ctrl+Shift+X)
   - Buscar "Marp for VS Code"
   - Install

3. **Abrir archivo:**
   - Open `10-presentation-slides.md` en VS Code
   - Click en el ícono de Marp (arriba derecha)
   - Preview en vivo de las slides

4. **Exportar:**
   - Click derecho en el preview
   - "Export Slide Deck..."
   - Elegir formato: PowerPoint, PDF, o HTML

**Resultado:** Archivo exportado con preview visual previo

---

### Opción 3: Online (Sin Instalación)

1. **Ir a Marp Online:**
   - https://web.marp.app/

2. **Copiar contenido:**
   - Abrir `10-presentation-slides.md`
   - Copiar TODO el contenido
   - Pegar en Marp Web

3. **Exportar:**
   - File → Export
   - Elegir PowerPoint (.pptx)

**Resultado:** Descarga directa del archivo .pptx

---

### Opción 4: Conversión Manual (Último Recurso)

Si ninguna opción anterior funciona:

1. **Usar Pandoc:**
```bash
# Instalar Pandoc
brew install pandoc  # macOS
# o descargar desde https://pandoc.org/

# Convertir
pandoc 10-presentation-slides.md -o presentation.pptx
```

2. **O copiar manualmente:**
   - Crear PowerPoint vacío
   - Cada sección `---` es un nuevo slide
   - Copiar contenido slide por slide
   - Aplicar formato manualmente

---

## 🎯 Cómo Usar Estos Materiales

### Antes de la Presentación (48h antes)

1. **Practicar con speaker notes:**
   - Leer `11-speaker-notes-guide.md` completo
   - Practicar al menos 2 veces completo
   - Cronometrar (debe ser ~25 minutos sin Q&A)

2. **Customizar slides:**
   - Reemplazar `[TBD]` con datos reales
   - Añadir logos de la empresa
   - Ajustar números si es necesario

3. **Preparar demos:**
   - Tener un template abierto para mostrar
   - Ejemplo de un servicio documentado
   - Dashboard (mockup si es necesario)

4. **Distribuir materiales:**
   - Enviar `12-executive-summary-handout.md` como PDF a asistentes
   - Opcional: Slides completos 24h antes

---

### Durante la Presentación

**Setup:**
- Proyector/screen share funcionando
- Slides en modo presentador (speaker notes visibles para ti)
- Reloj visible
- Agua cerca

**Timing sugerido:**
```
00:00 - Intro (Slides 1-2)
01:00 - El Problema (Slides 3-5)
04:00 - La Solución (Slides 6-9)
09:00 - Templates (Slides 10-15)
14:00 - Plan 12M (Slides 16-20)
19:00 - Estructura Repo (Slides 21-25)
24:00 - Primeros Pasos (Slides 26-30)
27:00 - FAQ (Slides 31-34)
29:00 - Cierre (Slides 35-37)
30:00 - Q&A (Slides 38-39)
```

**Tips del speaker notes:**
- Seguir el timing indicado
- Usar objeciones preparadas en sección FAQ
- Pausar después de números importantes
- Invitar preguntas durante presentación

---

### Después de la Presentación

**Inmediato (mismo día):**
- [ ] Enviar slides + grabación (si se grabó)
- [ ] Email de follow-up con próximos pasos
- [ ] Actualizar tracking doc de compromisos

**48 horas:**
- [ ] 1-on-1s con stakeholders que tuvieron objeciones
- [ ] Slack message resumen
- [ ] Calendar invites para kickoff

**Referencias:**
- Ver "Post-Presentación Checklist" en `11-speaker-notes-guide.md`

---

## 📝 Customización de Contenido

### Áreas que DEBES personalizar:

**Slides con [TBD] o placeholders:**
- Slide 2: Contactos específicos
- Slide 7: Métricas actuales de tu empresa
- Slide 19: Presupuesto exacto
- Slide 29: Roles y nombres
- Slide 36: Fechas específicas
- Slide 38: Links y recursos

**Opcional pero recomendado:**
- Añadir logo de la empresa en header/footer
- Cambiar color scheme si tienes brand colors
- Añadir screenshots de tu infraestructura actual
- Incluir ejemplos específicos de tu empresa

### Cómo editar el Marp theme:

En `10-presentation-slides.md`, líneas 1-8:
```yaml
---
marp: true
theme: default        ← cambiar a: gaia, uncover, custom
paginate: true
backgroundColor: #fff  ← cambiar color de fondo
---
```

**Themes disponibles:**
- `default` - Clean, profesional (recomendado)
- `gaia` - Más colorido
- `uncover` - Minimalista

---

## 🎨 Tips de Diseño Visual

### Si editas en PowerPoint después de exportar:

**Hacer:**
- ✅ Añadir transiciones sutiles (fade, no "fly in")
- ✅ Usar imágenes/íconos donde tenga sentido
- ✅ Consistencia en fonts (max 2 tipos)
- ✅ High contrast (texto oscuro en fondo claro)
- ✅ Números grandes y visibles

**Evitar:**
- ❌ Animaciones excesivas (distractoras)
- ❌ Walls of text (máximo 6 bullets por slide)
- ❌ Colores muy brillantes/neón
- ❌ Fonts muy pequeños (<20pt)
- ❌ Demasiadas slides (máx 40)

---

## 🎤 Tips de Presentación

### Del speaker notes, lo MÁS importante:

**Lenguaje Corporal:**
- Pararse, no sentarse
- Contacto visual con audiencia
- Gestos naturales con manos
- NO dar espalda a audiencia

**Voz:**
- Variar tono y volumen
- Pausas antes de puntos clave
- NO monótono o muy rápido

**Engagement:**
- Hacer preguntas ("Levanten la mano si...")
- Invitar a interrumpir
- Reconocer objeciones con empatía
- Usar nombres de personas presentes

**Timing:**
- Practicar 2+ veces antes
- Tener reloj visible
- Saber qué slides skip si vas tarde
- SIEMPRE dejar tiempo para Q&A

---

## 📊 Estructura de las 39 Slides

### Sección 1: El Problema (4 slides)
- Agenda
- Situación actual (fragmentación)
- Coste real (€360k/año)

### Sección 2: La Solución (5 slides)
- Visión SSOT
- Objetivos medibles
- Los 3 pilares
- → **Objetivo:** Que entiendan QUÉ y POR QUÉ

### Sección 3: Templates (6 slides)
- Templates 1-4 (detalle)
- Templates 5-8 (overview)
- → **Objetivo:** Que vean el CÓMO (herramientas)

### Sección 4: Plan (5 slides)
- Roadmap 12 meses
- Fase 1 detalle
- Fases 2-4 overview
- Inversión y ROI
- → **Objetivo:** Que crean que es FACTIBLE

### Sección 5: Estructura (5 slides)
- Organización por dominios
- Ejemplo departamento
- Governance (CODEOWNERS)
- Automatización
- → **Objetivo:** Que entiendan la IMPLEMENTACIÓN

### Sección 6: Acción (5 slides)
- Esta semana
- 2 semanas
- Mes 1
- Roles
- Training
- → **Objetivo:** Que sepan QUÉ HACER

### Sección 7: FAQ (5 slides)
- 5 objeciones comunes
- → **Objetivo:** Desarmar RESISTENCIAS

### Sección 8: Cierre (4 slides)
- 5 puntos clave
- Decisiones HOY
- Próximos pasos
- Q&A

---

## 🚀 Variantes de Presentación

### Versión Corta (15 minutos)

Usar solo estos slides:
- 1-2 (Intro)
- 3-4 (Problema)
- 6-8 (Solución)
- 10-11 (Templates highlight)
- 16-17 (Plan)
- 21 (Estructura)
- 26-27 (Primeros pasos)
- 35-36 (Cierre)

**Resultado:** 12 slides, 15 min con Q&A

---

### Versión Larga (60 minutos)

Añadir a la presentación base:
- Deep dive en cada template (10 min)
- Live demo de documentar un servicio (15 min)
- Más tiempo para Q&A (20 min)

**Resultado:** 60 min interactivo

---

### Versión Ejecutiva (5 minutos)

Solo estos slides:
- 1 (Título)
- 4 (Coste del problema)
- 6 (Visión)
- 7 (Objetivos)
- 19 (ROI)
- 36 (Decisiones)

**Resultado:** 6 slides, 5 min para C-level

---

## 📁 Archivos Relacionados

En el mismo package deberías tener:

- `00-standardization-plan.md` - Plan completo 12 meses
- `01-08-*.md` - Los 8 templates
- `09-repository-structure-guide.md` - Guía de estructura repo

**Estos NO son para presentar, son documentos de referencia**

---

## 🆘 Troubleshooting

### "No puedo instalar Marp"
→ Usa opción 3 (Marp Online - web.marp.app)

### "El .pptx se ve raro"
→ Marp a veces tiene formatting issues. Abre en PowerPoint y ajusta manualmente.

### "Necesito cambiar mucho contenido"
→ Mejor editar el .md primero, luego re-exportar. No editar el .pptx directamente.

### "Las tablas no se ven bien"
→ En PowerPoint, las tablas de markdown a veces necesitan ajuste manual de ancho.

### "Quiero más slides"
→ Cada sección `---` es un nuevo slide. Añade donde necesites.

### "Quiero menos slides"
→ Elimina secciones completas (entre `---`). No elimines slides sueltos sin contexto.

---

## 📞 Soporte

**Para preguntas sobre contenido:**
- docs-team@company.com

**Para issues técnicos de Marp:**
- https://github.com/marp-team/marp
- https://marpit.marp.app/

**Para feedback sobre estos materiales:**
- Crea un issue o contáctanos directamente

---

## ✅ Checklist Pre-Presentación

24 horas antes:
- [ ] Slides exportados a .pptx
- [ ] Practicado 2+ veces
- [ ] Timing verificado (~25 min)
- [ ] [TBD] reemplazados con datos reales
- [ ] Demos preparados
- [ ] Handout enviado a asistentes
- [ ] Proyector/screen share testeado
- [ ] Backup PDF de slides
- [ ] Speaker notes impresos (opcional)
- [ ] Agua y reloj listos

Día de presentación:
- [ ] Llegar 15 min antes
- [ ] Test de proyector/audio
- [ ] Modo presentador activado
- [ ] Teléfono en silencio
- [ ] Postura, respiración, confianza 💪

---

## 🎯 Objetivo de la Presentación

**NO es:**
- ❌ Que todos digan "sí" inmediatamente
- ❌ Perfección en la delivery
- ❌ Zero objeciones

**ES:**
- ✅ Que entiendan el problema y la solución
- ✅ Que vean que es realista y bien pensado
- ✅ Que se sientan escuchados en concerns
- ✅ Que quieran probar el piloto de Mes 1

**El éxito = Commitment para el piloto**

---

## 🎬 ¡Listo para Presentar!

Tienes todo lo que necesitas:
- ✅ 39 slides profesionales
- ✅ Speaker notes detallados
- ✅ Handout para entregar
- ✅ Guía de customización
- ✅ Tips de presentación

**Remember:**
- Respira antes de empezar
- Conoces el material
- Estás resolviendo un problema real
- You got this! 💪

---

**Version:** 1.0  
**Created:** 2024-03-15  
**Format:** Markdown (Marp) → PowerPoint/PDF  
**Duration:** 30 minutes  
**Audience:** Technical teams  

**¡Buena suerte! 🚀**
