# Guía del Presentador - Estandarización de Documentación IT
## Speaker Notes para Presentación de 30 Minutos

> **Duración total:** 30 minutos (25 min presentación + 5 min Q&A)  
> **Audiencia:** Equipos técnicos (Hardware, Virtual, WAN, LAN, Windows, Linux, DevOps, Cloud)  
> **Objetivo:** Conseguir buy-in y compromiso para empezar Fase 1

---

## 🎬 Preparación Pre-Presentación

### Checklist 24h Antes:

- [ ] **Enviar agenda y materiales** a los asistentes
  - PDF de la presentación
  - Link al repositorio de templates
  - Plan completo (opcional, para quien quiera profundizar)

- [ ] **Confirmar asistencia** de stakeholders clave:
  - Leads de cada departamento
  - Executive sponsor (CTO/VP)
  - Arquitectos principales
  - SRE leads

- [ ] **Preparar demos:**
  - Template de Architecture Design abierto
  - Ejemplo de un servicio documentado
  - Dashboard de métricas (mockup si es necesario)

- [ ] **Setup técnico:**
  - Proyector/pantalla compartida funcionando
  - Backup de slides en PDF
  - Links funcionando
  - Ejemplos listos para mostrar

---

## 📊 Estructura de Tiempo

| Sección | Duración | Slides | Timing |
|---------|----------|--------|--------|
| Intro + Agenda | 1 min | 1-2 | 0:00-1:00 |
| **El Problema** | 3 min | 3-5 | 1:00-4:00 |
| **La Solución** | 5 min | 6-9 | 4:00-9:00 |
| **Templates** | 5 min | 10-15 | 9:00-14:00 |
| **Plan 12M** | 5 min | 16-20 | 14:00-19:00 |
| **Estructura Repo** | 5 min | 21-25 | 19:00-24:00 |
| **Primeros Pasos** | 3 min | 26-28 | 24:00-27:00 |
| **FAQ** | 2 min | 29-34 | 27:00-29:00 |
| Cierre | 1 min | 35-37 | 29:00-30:00 |

**Total: 30 minutos**

---

## 🎤 Speaker Notes Detallados

### SLIDE 1-2: Título + Agenda (1 min)

**Qué decir:**
```
"Buenos días/tardes a todos. Gracias por estar aquí.

Voy a presentarles nuestro plan para estandarizar la documentación 
de arquitectura IT. Es un tema que todos hemos sentido - información 
dispersa, difícil de encontrar, y frecuentemente obsoleta.

Tenemos 30 minutos en total. Voy a dedicar 25 a la presentación y 
dejar 5 minutos al final para preguntas. Pero si hay algo crítico 
durante la presentación, por favor interrumpan.

[Mostrar agenda rápidamente]

Empecemos por el problema..."
```

**Tips:**
- ✅ Tono conversacional, no formal
- ✅ Reconocer el problema que todos sienten
- ✅ Dejar claro que habrá tiempo para preguntas
- ❌ No disculparse por "otro proyecto más"

---

### SLIDE 3-5: El Problema Actual (3 min)

**SLIDE 3: Situación Actual**

**Qué decir:**
```
"Todos hemos experimentado esto. Levanten la mano si en la última 
semana han buscado documentación y...
- No la encontraron [pausa para manos]
- La encontraron pero estaba obsoleta [pausa]
- Encontraron 3 versiones diferentes [pausa]

[Mostrar tabla]

Nuestro análisis muestra que tenemos documentación dispersa en al 
menos 5 lugares diferentes. Wiki, SharePoint, algunos en Confluence, 
emails archivados, y mensajes de Slack.

El 60% no se ha actualizado en 6 meses o más. Y el 40% de nuestros 
servicios ni siquiera tienen documentación básica.

El resultado: gastamos 8 horas por semana, POR PERSONA, buscando 
información. Eso es un día completo de trabajo perdido."
```

**Tips:**
- ✅ Hacer participar con la pregunta de manos levantadas
- ✅ Dar tiempo para que procesen los números
- ✅ Conectar con su experiencia diaria
- ❌ No culpar a nadie específicamente

---

**SLIDE 4: Coste Real del Problema**

**Qué decir:**
```
"Pero el problema no es solo de eficiencia. Tiene un coste real en euros.

[Mostrar cálculo]

Tenemos 15 incidentes al mes que no tienen runbook. El MTTR promedio 
es 2 horas. Si valoramos una hora de downtime en €1,000 - que es 
conservador - estamos hablando de €30,000 al mes.

Eso es €360,000 al año solo en downtime prevenible.

Y esto no incluye:
- Onboarding lento que retrasa productividad
- Conocimiento tribal que se va con la gente
- Auditorías de compliance complejas porque no encontramos evidencia
- Decisiones lentas porque nadie tiene el contexto completo

Esta presentación no es sobre crear más trabajo. Es sobre AHORRAR 
tiempo y dinero mientras mejoramos nuestra calidad operacional."
```

**Tips:**
- ✅ Usar números conservadores (más creíbles)
- ✅ Enfatizar que es dinero REAL, no abstracto
- ✅ Mencionar el compliance (importante para managers)
- ⚠️ Si alguien cuestiona los números, decir "podemos validar juntos después"

---

### SLIDE 6-9: La Solución (5 min)

**SLIDE 6: Visión SSOT**

**Qué decir:**
```
"Nuestra solución es simple en concepto: crear un Single Source of Truth.

Un repositorio centralizado donde:
- TODA la arquitectura importante está documentada
- Siempre actualizada porque lo automatizamos
- Fácil de encontrar porque tiene estructura clara
- Ownership claro - sabemos quién mantiene qué
- Y es parte de nuestro workflow diario, no algo separado

El principio fundamental es: 'Si no está en el repo, no existe.'

Esto no es un wiki más. Es integrar documentación en cómo trabajamos."
```

**Tips:**
- ✅ Repetir "Single Source of Truth" - es el concepto clave
- ✅ Enfatizar "parte del workflow" - no es extra work
- ❌ No entrar en detalles técnicos todavía

---

**SLIDE 7: Objetivos del Proyecto**

**Qué decir:**
```
"Tenemos objetivos medibles para 12 meses:

[Mostrar tabla fila por fila]

MTTR de 2 horas a 45 minutos. ¿Cómo? Con runbooks claros que 
funcionan, no hay que investigar desde cero cada vez.

Onboarding de 4 semanas a 1.5 semanas. Los nuevos pueden aprender 
del repo en lugar de preguntar a 10 personas diferentes.

Del 40% al 90% de servicios documentados. Coverage real.

Y el ROI: €215,000 neto en el primer año. Esto se paga solo en 6 meses.

[Pausa para que procesen]

Estos no son números aspiracionales. Están basados en benchmarks de 
empresas similares que han hecho esto - Google, Netflix, Uber todos 
publican sobre sus procesos SRE."
```

**Tips:**
- ✅ Ir despacio en los números, dejar que procesen
- ✅ Explicar el "cómo" brevemente para cada métrica
- ✅ Mencionar las referencias (Google SRE Book) - da credibilidad
- ⚠️ Si cuestionan el ROI, recordar que hay documento detallado

---

**SLIDE 8: Los 3 Pilares**

**Qué decir:**
```
"La solución tiene tres pilares:

Primero: Templates estandarizados. Hemos creado 8 templates 
profesionales basados en best practices de SRE. No tienen que 
inventar la rueda, solo llenar los templates.

Segundo: Proceso simple. Todo integrado en CI/CD. Automatización 
donde sea posible. Hemos visto que reduce el esfuerzo manual en 80%.

Tercero: Cultura doc-first. Hacer de la documentación parte del 
Definition of Done, con incentivos y soporte continuo.

Estos tres pilares se refuerzan entre sí."
```

**Tips:**
- ✅ Usar los dedos: "Primero... Segundo... Tercero..."
- ✅ Brevedad aquí - los detalles vienen después
- ✅ "80% menos esfuerzo" es un soundbite importante

---

### SLIDE 10-15: Los 8 Templates (5 min)

**Estrategia general para esta sección:**
- Ir RÁPIDO en slides 10-13 (templates 1-4)
- 1 minuto cada uno
- MÁS rápido en slides 14-15 (templates 5-8)
- El objetivo NO es explicar todo, sino dar sabor

**SLIDE 10: Template 1 - Architecture Design**

**Qué decir:**
```
"Vamos a ver rápidamente los 8 templates. No voy a entrar en cada 
detalle - los documentos completos están disponibles.

El primero es Architecture Design. Este contiene todo lo que 
necesitas para entender un servicio:
- Requisitos funcionales y no funcionales
- SLOs - Service Level Objectives, no solo SLAs
- Diagramas usando el estándar C4
- Dependencias claras
- Capacity planning y disaster recovery
- Y cumplimiento - GDPR, SOC2, lo que aplique

Todo servicio crítico debe tener este documento. Es el 'manual de 
usuario' del servicio para otros equipos."
```

**Tips:**
- ✅ Mencionar "SLOs vs SLAs" - diferenciador importante
- ✅ "C4" si la audiencia es técnica (sino skip)
- ❌ No leer la lista completa, solo highlights

---

**SLIDE 11: Template 2 - Deployment & Runbooks**

**Qué decir:**
```
"Deployment & Runbooks. Este es crítico para operaciones.

La clave aquí es Infrastructure as Code como fuente de verdad. 
No documentas manualmente cómo deployar - el IaC ES la documentación.

Incluye procedimientos para deployment normal, hotfixes, rollbacks.
Runbooks operacionales - qué hacer cuando algo falla.
Health checks y troubleshooting.

Lo importante: estos runbooks son TESTEABLES. No son teóricos, 
son procedimientos que realmente funcionan."
```

**Tips:**
- ✅ "IaC como fuente de verdad" - concepto moderno
- ✅ "Testeables" - palabra clave
- ✅ Conectar con su dolor: "cuando algo falla a las 3 AM"

---

**SLIDE 12: Template 3 - Service Ownership & RACI**

**Qué decir:**
```
"Service Ownership y RACI.

Este responde una pregunta crítica: cuando hay un problema a las 
3 de la mañana, ¿quién es responsable?

Incluye:
- Service Owner identificado - una persona, no un equipo
- Matriz RACI completa - quién hace qué
- On-call rotation y procedimientos
- Escalation paths claros
- Contactos actualizados

Si no hay owner claro, el servicio no debería estar en producción."
```

**Tips:**
- ✅ La pregunta "3 AM" resuena con todos
- ✅ "Una persona, no un equipo" - específico
- ⚠️ Esto puede generar debate - OK, pero no ahora

---

**SLIDE 13: Template 4 - Observability & Monitoring**

**Qué decir:**
```
"Observability y Monitoring.

El lema: Si no lo puedes medir, no lo puedes operar.

Este template define:
- SLIs - las métricas medibles
- Golden Signals - Latency, Traffic, Errors, Saturation
- Links a dashboards
- Alerting con runbooks asociados
- Logs, traces, metrics

La clave es: cada alerta debe tener un runbook. Nada de alertas 
que nadie sabe cómo resolver."
```

**Tips:**
- ✅ "Si no lo mides, no lo operas" - principio SRE
- ✅ "Cada alerta → runbook" - conexión importante
- ✅ Golden Signals si audiencia técnica (sino skip detalles)

---

**SLIDE 14-15: Templates 5-8**

**Qué decir:**
```
"Los otros cuatro templates son de soporte pero igual de importantes:

Service Catalog Entry - el registro centralizado de cada servicio.
Metadata, links, contactos. El índice de todo.

Incident Management - cómo responder cuando las cosas fallan. 
Severity matrix, workflows, post-mortem templates.

Change Management - tipos de cambios, aprobaciones, RFC templates.
Proceso formal para cambios de alto riesgo.

Y API Documentation - si tu servicio expone APIs, cómo documentarlas 
correctamente con OpenAPI/Swagger standards.

No todos los servicios necesitan los 8. Pero los críticos sí."
```

**Tips:**
- ✅ IR RÁPIDO aquí - 30 segundos para los 4
- ✅ "No todos necesitan los 8" - reduce la intimidación
- ❌ No entrar en detalles - solo overview

---

### SLIDE 16-20: Plan 12 Meses (5 min)

**SLIDE 16: Roadmap Overview**

**Qué decir:**
```
"Ahora el plan. 12 meses, divididos en 4 fases trimestrales.

[Mostrar el diagrama Q1-Q4]

Q1 es Foundation. Validamos el enfoque con servicios Tier 1.
100% de servicios críticos documentados.

Q2 es Expansion. Escalamos a 40% de servicios totales.
Training completo, automatización operativa.

Q3 es Scale. Llegamos a 60%. Compliance ready.
La cultura doc-first empieza a ser real.

Q4 es Excellence. Alcanzamos 90%. Process optimizado.
Y lo convertimos en sustainable - solo 0.5 FTE para mantenimiento.

Es un crescendo. Empezamos pequeño, validamos, y luego escalamos."
```

**Tips:**
- ✅ Usar las manos para mostrar crecimiento
- ✅ "Empezamos pequeño" - reduce ansiedad
- ✅ "0.5 FTE Year 2" - muestra sostenibilidad
- ⚠️ No prometer demasiado en Q1

---

**SLIDE 17: Fase 1 - Foundation (Q1)**

**Qué decir:**
```
"Vamos al detalle de Q1 porque es donde necesitamos su compromiso YA.

Mes 1: Setup y piloto con 3 servicios.
Formamos el task force, setup tooling, documentamos 3 servicios 
completamente para validar que funciona.

Mes 2: Expandimos a 5 servicios más.
Implementamos automated checks - linting, link checking, integración 
con CI/CD.

Mes 3: Completamos todos los Tier 1.
10-15 servicios totales documentados.
Templates refinados basado en aprendizaje real.

Y CRÍTICO - al final del mes 3 hay un go/no-go gate.
Evaluamos: ¿Funciona? ¿Los equipos están satisfechos? ¿Vale la pena?
Solo continuamos si la respuesta es sí."
```

**Tips:**
- ✅ "3 servicios" en Mes 1 suena manejable
- ✅ Enfatizar el go/no-go gate - muestra que no es compromiso ciego
- ✅ "CRÍTICO" para captar atención
- ⚠️ No comprometerse a servicios específicos aún

---

**SLIDE 18: Fases 2-4**

**Qué decir:**
```
"Q2, Q3 y Q4 escalan el modelo validado en Q1.

[No leer todo, solo highlights]

Q2: 20 servicios más. Automation tools. Training program.

Q3: Otros 35 servicios. Compliance templates. Policy formal.

Q4: Los 25 finales. Optimization. Sustainable.

El patrón es: cada trimestre builds on el anterior.
No hay Big Bang aquí. Es incremental y controlado."
```

**Tips:**
- ✅ IR MUY RÁPIDO aquí - 30 segundos max
- ✅ "Incremental y controlado" - palabras clave
- ❌ No detallar cada bullet

---

**SLIDE 19: Inversión y ROI**

**Qué decir:**
```
"Hablemos de dinero.

[Pausa para que vean los números]

Inversión Año 1: €280,000.
Eso es 1 Technical Writer full-time, medio SRE, cuarto de arquitecto, 
cuarto de PM. Más tooling y training.

Ahorro Año 1: €495,000.
Donde viene: reducción de MTTR, onboarding más rápido, menos riesgo 
de compliance, retención de conocimiento.

ROI neto: +€215,000 en el primer año.
Break-even a los 6 meses.

Año 2 solo requiere €120k - 57% menos. Porque ya está establecido.

[Pausa]

Y esto es solo el ahorro cuantificable. No incluye beneficios como 
mejor toma de decisiones, menos estrés del equipo, mejor onboarding 
de clientes enterprise."
```

**Tips:**
- ✅ Dar tiempo para procesar los números
- ✅ "6 meses break-even" - timeframe concreto
- ✅ "57% menos Año 2" - muestra sostenibilidad
- ⚠️ Si alguien cuestiona: "Tengo el breakdown detallado"

---

### SLIDE 21-25: Estructura del Repositorio (5 min)

**SLIDE 21: Organización por Dominios**

**Qué decir:**
```
"Ahora cómo nos organizamos. Esto es importante porque afecta 
ownership y workflow diario.

[Mostrar estructura]

Tenemos un repositorio Git con estructura clara:

Templates reutilizables arriba.
Global para políticas company-wide.
Infrastructure ORGANIZADO POR DEPARTAMENTO - y aquí están sus 8 
departamentos: Hardware, Virtualización, WAN, LAN, Windows, Linux, 
DevOps, Cloud Azure.

Cada departamento tiene su carpeta y es dueño de esa documentación.

Después servicios por aplicación.
Cross-cutting para temas transversales.
Y decisiones para Architecture Decision Records.

Es jerárquico pero flexible."
```

**Tips:**
- ✅ Señalar físicamente las secciones en la pantalla
- ✅ "Cada departamento es dueño" - ownership claro
- ✅ Nombrar los 8 departamentos específicamente
- ❌ No explicar CADA carpeta

---

**SLIDE 22: Ejemplo Departamento DevOps**

**Qué decir:**
```
"Un ejemplo concreto: el departamento de DevOps.

[Mostrar estructura]

Tienen su carpeta con:
- Architecture subfolder - diseños de Kubernetes, ingress strategy, etc.
- Clusters subfolder - documentación por cluster: prod-eu, prod-us, dev
- CICD subfolder - Jenkins, ArgoCD configs
- Observability - Prometheus, Grafana
- Y runbooks - procedimientos operacionales

Cada departamento puede estructurar su carpeta como tenga sentido 
para ellos. Pero siguen los templates estándar para los documentos.

Autonomía con consistencia."
```

**Tips:**
- ✅ Usar ejemplo concreto ayuda a visualizar
- ✅ "Autonomía con consistencia" - balance importante
- ✅ Si hay gente de DevOps, hacer contacto visual
- ⚠️ Ofrecer mostrar ejemplo de otro departamento si hay tiempo

---

**SLIDE 23: Governance CODEOWNERS**

**Qué decir:**
```
"Governance es crítico. Usamos CODEOWNERS de GitHub.

[Mostrar ejemplos]

Cada departamento es 'owner' de su carpeta en el repo.
Cuando alguien hace un cambio en tu documentación, TÚ tienes que 
aprobar el pull request.

Por ejemplo:
- Cambios en /infrastructure/devops/ requieren aprobación de @devops-team
- Cambios en /infrastructure/wan/ requieren @network-team

Cross-cutting como security requiere múltiples equipos.

Esto garantiza:
1. Ownership claro
2. Calidad - los expertos revisan
3. Conocimiento distribuido - el review es también training
4. Accountability - sabemos quién aprobó qué

No es burocracia. Es calidad y ownership."
```

**Tips:**
- ✅ Explicar CODEOWNERS brevemente - no todos lo conocen
- ✅ Los 3 beneficios son importantes - enfatizar
- ✅ "No es burocracia" - anticipar objeción
- ⚠️ Si hay resistencia, mencionar que es opcional para standard changes

---

**SLIDE 24: Automatización desde Día 1**

**Qué decir:**
```
"Y todo esto está automatizado desde el principio.

[Mostrar pipeline]

Cuando haces un pull request:
1. Markdown linting automático
2. Broken links check
3. Spell check
4. Valida que la estructura del template esté completa
5. Y requiere aprobación de CODEOWNERS

Cuando se mergea a main:
1. Auto-publica a Confluence o tu wiki
2. Actualiza el dashboard de métricas
3. Notifica en Slack
4. Genera reporte de coverage

80% menos trabajo manual. La máquina hace el trabajo tedioso.
Los humanos se enfocan en contenido de calidad."
```

**Tips:**
- ✅ "80% menos trabajo" - repetir este número
- ✅ "Máquina hace tedioso, humanos hacen calidad" - buen soundbite
- ✅ Mostrar entusiasmo - esto ES cool
- ❌ No entrar en detalles de GitHub Actions

---

### SLIDE 26-28: Primeros Pasos (3 min)

**SLIDE 26: Esta Semana**

**Qué decir:**
```
"OK, suficiente teoría. ¿Qué hacemos AHORA?

Esta semana necesitamos tres cosas del leadership:

Uno: Aprobar el plan y el presupuesto de €280k.
Sin esto, no podemos contratar el Technical Writer ni comprar tooling.

Dos: Asignar un executive sponsor.
Típicamente el CTO o VP of Engineering.
Alguien que puede desbloquear recursos y dar air cover.

Tres: Comunicar que esto es prioridad.
Si la gente no sabe que es importante, no dedicarán tiempo.

Y tres cosas del core team:

Uno: Formar el Documentation Task Force.
1 Tech Writer, 1 SRE Lead, champions por departamento.

Dos: Seleccionar 3 servicios piloto.
Tier 1, críticos, equipos comprometidos.

Tres: Kickoff meeting. 2 horas.
Alineamiento inicial.

Estas decisiones se toman esta semana. El mes que viene ya estaremos 
documentando."
```

**Tips:**
- ✅ "Esta semana" - urgencia
- ✅ Usar dedos: "Uno... dos... tres..."
- ✅ Ser específico: "€280k", "2 horas", "3 servicios"
- ⚠️ Pausar después de cada punto para que anoten

---

**SLIDE 27: Próximas 2 Semanas**

**Qué decir:**
```
"En las dos semanas siguientes:

Setup técnico:
- Crear el repositorio Git
- Implementar la estructura de carpetas que vieron
- Setup tooling básico: Confluence, CI/CD, Slack channel

Y empezar el piloto:
- Workshop de 4 horas con los 3 equipos piloto
- Hands-on: crear el primer doc juntos
- Documentar esos 3 servicios completamente
- Iterar los templates basado en su feedback

Al final de estas 2 semanas tenemos 3 servicios documentados 
y feedback real de equipos reales."
```

**Tips:**
- ✅ "Hands-on" - no es teórico
- ✅ "Feedback real" - se escucha su voz
- ✅ Brevedad aquí - están cansados de detalles
- ❌ No expandir más

---

**SLIDE 28: Mes 1 - Objetivos**

**Qué decir:**
```
"Y los objetivos del Mes 1 son claros:

[Leer lista]

Pero el MÁS importante es el último punto.

[Señalar "Decisión crítica"]

Al final del Mes 1 nos preguntamos:
- ¿Los templates funcionan en la realidad?
- ¿Los equipos están satisfechos o frustrados?
- ¿Las docs fueron útiles en incidentes reales?

Si la respuesta a las tres es sí, continuamos a Fase 1 completa.
Si no, pivotamos la estrategia.

No es un compromiso ciego de 12 meses.
Es un piloto de 1 mes con option to continue."
```

**Tips:**
- ✅ "Option to continue" - reduce riesgo percibido
- ✅ Enfatizar el go/no-go gate OTRA VEZ
- ✅ "No es compromiso ciego" - tranquiliza
- ⚠️ Esta es su safety net - hacerlo claro

---

**SLIDE 29: Roles y Responsabilidades**

**Qué decir:**
```
"Rápidamente sobre roles:

[Mostrar tabla]

Core team de 2 FTEs total:
1 Technical Writer full-time - crea standards, training, quality
0.5 SRE - automation, runbooks, tech guidance
0.25 Architect - architecture review, ADRs
0.25 PM - coordination, reporting

Y extended team:
Department champions al 10% - uno o dos por departamento
Service owners al 5% - mantienen sus docs

No es una army. Es un pequeño core team con support distribuido."
```

**Tips:**
- ✅ "2 FTEs total" - no suena como overhead grande
- ✅ "Small core + distributed" - modelo claro
- ❌ No leer cada responsabilidad

---

**SLIDE 30: Training & Enablement**

**Qué decir:**
```
"Y no los dejamos solos. Hay training:

Path 1 para todos los service owners: 4 horas, obligatorio.
Intro, hands-on workshop, tools, maintenance.

Path 2 para champions: 8 horas, avanzado.
Deep dive, train-the-trainer.

Path 3 para leadership: 1 hora, ejecutivo.
Business case, ROI, compliance.

Múltiples formatos: presencial, video, self-paced.
Office hours semanales para Q&A.

Nadie tiene que figurarlo out solo."
```

**Tips:**
- ✅ "4 horas obligatorio" - expectativa clara
- ✅ "Nadie solo" - mensaje de soporte
- ✅ Brevedad - solo overview
- ❌ No prometer fechas específicas aún

---

### SLIDE 31-34: FAQ (2 min)

**Estrategia:** Solo cubrir las 2-3 más importantes. El resto "tengo respuestas si hay preguntas".

**SLIDE 31: FAQ 1 - ¿No es mucho trabajo?**

**Qué decir:**
```
"Pregunta obvia: ya estamos sobrecargados, ¿cómo encontramos tiempo?

Tres respuestas:

Uno: Es inversión inicial, ahorro continuo.
Primera doc toma ~16 horas. Pero luego ahorras >100 horas al año 
en incidentes más rápidos.

Dos: Automatización reduce 80% del esfuerzo.
No estás escribiendo desde cero. Templates, auto-generation, AI assist.

Tres: No todos los servicios a la vez.
Fase 1 solo Tier 1. 12 meses para 90%.

Es como invertir en automation. Duele al principio, paga después."
```

**Tips:**
- ✅ "Pregunta obvia" - valida su preocupación
- ✅ Tres respuestas claras
- ✅ "16h vs >100h" - math simple
- ⚠️ Si insisten "no tenemos 16h", redirigir a después

---

**SLIDE 32: FAQ 2 - SharePoint vs Git**

**Qué decir:**
```
"¿Por qué Git y no solo SharePoint mejor?

Usamos AMBOS. Best of both worlds.

Git es la fuente de verdad: versionado, branching, PR reviews, CODEOWNERS.
Confluence/SharePoint para visualización: interfaz familiar para management.

Auto-sync entre ellos.

Developers editan en Git - su workflow normal.
Management lee en Confluence - interfaz familiar.
Automatic sync mantiene todo sincronizado.

No es Git OR SharePoint. Es Git AND SharePoint, cada uno para lo que es bueno."
```

**Tips:**
- ✅ "AMBOS" - importante clarificar
- ✅ "Git para developers, Confluence para management" - segmentación clara
- ✅ "Auto-sync" - resuelve la preocupación de duplicación
- ⚠️ Si insisten en "solo SharePoint", mencionar limitaciones (no IaC, no PR reviews)

---

**SLIDE 33-34: Otras FAQs**

**Qué decir:**
```
"Tengo respuestas preparadas para otras preguntas comunes:

[Solo leer títulos]

- ¿Cómo garantizamos adopción?
- ¿Quién mantiene esto actualizado?
- ¿Y los sistemas de vendors?

Pero en interés del tiempo, voy a pasar al cierre.
Si alguna de estas es crítica para ustedes, háganla en el Q&A."
```

**Tips:**
- ✅ Solo listar, no explicar
- ✅ "En interés del tiempo" - respeta su tiempo
- ✅ Invitar a Q&A - muestra apertura
- ❌ No empezar a responderlas si nadie pregunta

---

### SLIDE 35-37: Resumen y Cierre (1 min)

**SLIDE 35: Los 5 Puntos Clave**

**Qué decir:**
```
"Para cerrar, los 5 puntos que quiero que recuerden:

[Leer cada uno con pausa]

Uno: El problema es real. €360k/año en downtime prevenible.

Dos: La solución está probada. Google, Netflix, Uber - todos hacen esto.

Tres: El plan es realista. 12 meses, 4 fases, checkpoints.

Cuatro: El ROI es claro. €280k inversión, €495k ahorro, +€215k neto.

Cinco: Empezamos pequeño. 3 servicios piloto, scaling gradual.

Este no es un leap of faith. Es un piloto medible con opción de escalar."
```

**Tips:**
- ✅ Ir DESPACIO - están tomando notas mentalmente
- ✅ Pausar 2 segundos entre cada punto
- ✅ "Leap of faith" - reconocer la duda
- ✅ Contacto visual en cada punto

---

**SLIDE 36: Decisiones Necesarias HOY**

**Qué decir:**
```
"Necesitamos 4 decisiones hoy - no necesariamente en esta sala, 
pero esta semana:

[Mostrar checklist]

1. ¿Aprobamos el enfoque?
2. ¿Aprobamos €280k de presupuesto?
3. ¿Quién es el executive sponsor?
4. ¿Cuándo empezamos? ¿Q2 2024?

[Pausa]

Si la respuesta a las primeras tres es sí, la cuarta es automática: 
empezamos ya.

Mi ask específico a los aquí presentes:
- Leads de departamento: Nominar un champion de su equipo esta semana
- Architects: Ayudar a seleccionar servicios piloto
- Managers: Liberar tiempo para el piloto

Y a todos: Si esto tiene sentido, ayuden a evangelizar.
Si no tiene sentido, díganme por qué - mejoramos el plan juntos."
```

**Tips:**
- ✅ "Esta semana" - crear urgencia
- ✅ Ask específico por rol - cada quien sabe qué hacer
- ✅ "Si no tiene sentido, díganme" - invita feedback honesto
- ⚠️ Contacto visual con los decision makers

---

**SLIDE 37: Próximos Pasos Concretos**

**Qué decir:**
```
"Y los próximos pasos concretos:

[Leer rápido]

Esta semana: Approvals, nominar champions
2 semanas: Kickoff, pilotos
Mes 1: 3 servicios documentados, go/no-go

Simple. Claro. Medible.

[Pausa]

Preguntas ahora, o pueden contactarme después.
Email, Slack, office hours - puertas abiertas.

Construyamos juntos el Single Source of Truth."
```

**Tips:**
- ✅ "Simple. Claro. Medible." - tres adjetivos pegajosos
- ✅ "Puertas abiertas" - accesibilidad
- ✅ Terminar con el lema "Single Source of Truth"
- ✅ Sonreír - terminar en tono positivo

---

### SLIDE 38-39: Q&A (5 min)

**Qué decir:**
```
"Tienen 5 minutos para preguntas.

¿Qué preguntas tienen?"

[Esperar - silencio es OK por 5-10 segundos]

[Si silencio continúa:]
"O puedo empezar con las que usualmente escucho: ¿Qué pasa con 
los servicios legacy? ¿Cómo priorizamos qué documentar primero?"
```

---

## 🎯 Manejo de Objeciones Comunes

### Objeción 1: "No tenemos tiempo"

**Respuesta corta:**
```
"Entiendo. Pero el tiempo que invertimos ahora lo ahorramos 10x 
después. Y automatizamos 80% del esfuerzo. ¿Podemos al menos hacer 
el piloto de 1 mes y medir el impacto real?"
```

**Si insisten:**
```
"Pregunta: ¿Cuánto tiempo gastaron el mes pasado buscando 
documentación? ¿Debugging incidentes sin runbook? ¿Onboarding 
gente nueva? Eso es el tiempo que recuperamos."
```

---

### Objeción 2: "Ya intentamos esto antes y falló"

**Respuesta corta:**
```
"Buena pregunta. ¿Qué falló específicamente? [Escuchar]

Lo que es diferente ahora:
1. Automatización - no depende de disciplina manual
2. Templates - no empezar de cero
3. Integración CI/CD - parte del workflow, no aparte
4. Ownership claro - CODEOWNERS garantiza accountability"
```

---

### Objeción 3: "Esto es solo para desarrollo, no infraestructura"

**Respuesta corta:**
```
"Al contrario. Infraestructura NECESITA documentación más que nadie.

Switches, routers, firewall rules, VLAN configs - toda esa 
configuración es crítica y si se pierde, es disaster.

Los 8 departamentos están en el plan porque todos necesitan esto.
De hecho, hardware y networking típicamente se benefician más."
```

---

### Objeción 4: "El presupuesto es muy alto"

**Respuesta corta:**
```
"€280k suena alto, pero consideren:
- 1 FTE Technical Writer: €80k (standard market rate)
- 0.5 FTE SRE: €60k (ya lo tenemos, solo dedicamos tiempo)
- Tooling: €50k (Confluence ya lo tenemos, esto es extra features)

Y recuerden: ahorro de €495k en Año 1.
Break-even a los 6 meses.

¿Qué parte del presupuesto es la preocupación específica?"
```

**Si insisten:**
```
"OK, podemos hacer una versión lean:
- No contratar Tech Writer, usar contractor part-time: -€40k
- Solo tooling esencial: -€20k
- Reducir training: -€10k

€210k versión lean. ¿Eso funciona?"
```

---

### Objeción 5: "¿Por qué no [herramienta X] que ya tenemos?"

**Respuesta corta:**
```
"Gran punto. ¿Qué herramienta específicamente?

[Si dicen SharePoint/Confluence:]
Vamos a usar eso para visualización. Git es para source of truth 
con versionado y PR reviews. Auto-sync entre ambos.

[Si dicen otra herramienta:]
Podemos evaluarla. Los principios son más importantes que las 
herramientas específicas. Si [herramienta X] soporta lo que 
necesitamos, podemos adaptarnos."
```

---

## 📋 Post-Presentación Checklist

Inmediatamente después:

- [ ] **Enviar slides + grabación** (si se grabó) a todos los asistentes
- [ ] **Email de follow-up** con:
  - Summary de decisiones tomadas (o pendientes)
  - Próximos pasos con owners y deadlines
  - Links a documentos completos
  - Forma de dar feedback o hacer preguntas
- [ ] **Actualizar el plan** basado en feedback recibido
- [ ] **Schedule 1-on-1s** con key stakeholders que tuvieron objeciones
- [ ] **Crear tracking doc** de compromisos hechos en la reunión

Dentro de 48h:

- [ ] **Slack message** en canal relevante resumiendo meeting
- [ ] **Calendar invites** para próximos pasos (kickoff, etc.)
- [ ] **Document FAQs** que surgieron para futuras presentaciones

---

## 🎓 Tips Generales de Presentación

### Lenguaje Corporal:
- ✅ Pararse (no sentarse) - más energía
- ✅ Moverse naturalmente - no estar plantado
- ✅ Contacto visual con diferentes personas
- ✅ Gestos con las manos para enfatizar
- ❌ Dar la espalda a la audiencia (mirar pantalla)
- ❌ Leer las slides textualmente

### Voz:
- ✅ Variar tono y volumen
- ✅ Pausas estratégicas antes de puntos clave
- ✅ Enfatizar números importantes
- ❌ Monótono
- ❌ Hablar muy rápido por nervios

### Engagement:
- ✅ Hacer preguntas ("Levanten la mano si...")
- ✅ Invitar a interrumpir si hay dudas
- ✅ Reconocer objeciones con empatía
- ✅ Usar nombres de personas presentes
- ❌ Modo "lecture" unidireccional

### Timing:
- ✅ Practicar al menos 2 veces completas
- ✅ Tener reloj visible
- ✅ Saber qué slides se pueden skip si vas tarde
- ✅ Dejar SIEMPRE tiempo para Q&A
- ❌ Correr al final
- ❌ Pasar del tiempo asignado

---

## 🚨 Señales de Éxito en la Sala

Durante la presentación, buscar estos indicadores:

**Positivos:**
- 👍 Gente tomando notas
- 👍 Cabezas asintiendo en puntos clave
- 👍 Preguntas constructivas (no defensivas)
- 👍 Comentarios como "esto nos pasó la semana pasada"
- 👍 Gente haciendo preguntas de implementación (detalles)

**Neutrales/Preocupantes:**
- ⚠️ Silencio total sin preguntas
- ⚠️ Brazos cruzados (postura defensiva)
- ⚠️ Gente en teléfonos/laptops
- ⚠️ Miradas escépticas
- ⚠️ Preguntas de "por qué" repetidas (resistencia)

**Acciones correctivas en tiempo real:**
- Si ves señales negativas → cambiar a modo más conversacional
- Si gente desconectada → hacer una pregunta directa
- Si muchas objeciones → reconocer concerns, ofrecer 1-on-1s
- Si confusión → usar un ejemplo concreto

---

## 📞 Contactos y Recursos Post-Presentación

**Para seguimiento:**
```
📧 Email: docs-team@company.com
💬 Slack: #documentation (crear si no existe)
📅 Office Hours: [Establecer calendario]
📁 Documentos: [Link a repo/wiki]
👥 Core Team:
   - Program Lead: [Nombre] - [email]
   - Executive Sponsor: [Nombre] - [email]
```

---

## 🎬 Cierre de la Guía

**Recuerda:**
- 🎯 Tu objetivo NO es que digan "sí" inmediatamente
- 🎯 Tu objetivo ES que entiendan el problema y la solución
- 🎯 Que vean que es realista y bien pensado
- 🎯 Que se sientan escuchados en sus objeciones
- 🎯 Que quieran participar en el piloto

**El éxito no es consensus perfecto.**
**El éxito es commitment para probar el Mes 1.**

---

**¡Buena suerte! 🚀**

**Notas finales:**
- Respira antes de empezar
- Es normal estar nervioso
- Conoces el material - confía en eso
- Si algo sale mal, continúa con gracia
- Recuerda: estás resolviendo un problema real que todos sienten

*You got this!* 💪
