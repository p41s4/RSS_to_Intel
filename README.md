# Threat Intelligence RSS Automation – Daily Feed to Teams

## Descripción general

Este proyecto implementa una pipeline automatizada de Threat Intelligence construida en n8n, cuyo objetivo es:

- Recopilar diariamente noticias de ciberseguridad desde múltiples feeds RSS
- Almacenarlas de forma centralizada
- Analizarlas automáticamente con IA
- Publicar únicamente los artículos relevantes y aprobados en Microsoft Teams

El sistema está dividido en dos workflows independientes pero complementarios, que deben ejecutarse en el siguiente orden:

1. RSS Daily Feed  
2. Feed to Teams  

---

## Arquitectura del sistema

RSS Feeds → RSS Daily Feed → Data Table  
                              ↓  
                        Feed to Teams  
                              ↓  
                      AI Threat Analysis  
                              ↓  
                   Teams/Slack/Telegram/etc

---

## 1. RSS Daily Feed

### Objetivo

Recolectar noticias de ciberseguridad publicadas durante el día, filtrarlas por fecha y almacenarlas en una tabla central para su posterior análisis.

### Funcionamiento

1. Ejecución programada  
   - Se ejecuta automáticamente todos los días a las 11:00 mediante un nodo Cron.

2. Lectura de feeds RSS  
   - Consume múltiples fuentes, por ejemplo:
     - BleepingComputer
     - The Hacker News

3. Unificación de resultados  
   - Los artículos de todos los feeds se combinan en un único flujo.

4. Normalización de fechas  
   - Se convierte el campo isoDate a formato ISO estándar.

5. Filtrado temporal  
   - Solo se procesan artículos:
     - Con fecha válida
     - Publicados después del inicio del día actual

6. Persistencia  
   - Cada artículo se inserta en una Data Table con los campos:
     - Titulo
     - Enlace
     - Resumen
     - Approve = false
     - processed = false

Resultado: una cola diaria de artículos pendientes de análisis.

---

## 2. Feed to Intel

### Objetivo

Analizar los artículos recopilados, evaluar su nivel de amenaza y relevancia, y publicar automáticamente los más relevantes en Microsoft Teams/Slack/Telegram/etc.

### Funcionamiento

1. Lectura de artículos  
   - Se recuperan las filas de la Data Table.

2. Filtrado inicial  
   - Solo se procesan artículos que cumplan:
     - Approve = true
     - processed = false

3. Descarga del contenido  
   - Se accede al enlace del artículo vía HTTP.
   - Se extrae el texto principal mediante parsing HTML (etiqueta article).

4. Análisis con IA  
   - Un AI Agent actúa como Threat Intelligence Analyst y evalúa:
     - Resumen del artículo
     - Prerrequisitos del ataque
     - Actores de amenaza
     - Victimología (industrias, países o regiones)
     - Capability, Opportunity e Intent
     - IOCs y artefactos cazables
     - Técnicas de ataque explicadas en lenguaje no técnico

   La amenaza se clasifica como:
   - Potential
   - Impending
   - Insubstantial

5. Actualización de estado  
   - El resultado del análisis IA se guarda en la tabla.
   - El artículo se marca como:
     - processed = true
     - Approve = false

6. Publicación  
   - El contenido analizado puede enviarse automáticamente a Microsoft Teams como mensaje estructurado.

---

## Resultado final

El sistema entrega:

- Un feed diario curado de Threat Intelligence
- Artículos analizados y clasificados por riesgo
- Información enriquecida con contexto técnico y estratégico
- Contenido listo para:
  - SOC
  - Threat Hunters
  - Analistas CTI
  - Consumo ejecutivo

---

## Data Table – Esquema

| Campo     | Descripción |
|----------|-------------|
| Titulo   | Título del artículo |
| Enlace   | URL original |
| Resumen  | Extracto del feed RSS |
| IA       | Análisis generado por IA |
| Approve  | Control manual de aprobación |
| processed| Evita reprocesamiento |
| rowID    | Identificador interno |

---

## Requisitos

- Instancia funcional de n8n
- Acceso a:
  - Feeds RSS públicos
  - OpenAI API
  - Microsoft Teams (Webhook o conector)
- Conocimientos básicos de:
  - Automatización con n8n

---

## Posibles extensiones

- Aprobación automática basada en IA
- Integración con SIEM o SOAR
- Scoring de riesgo
- Dashboard de métricas CTI
- Evolución hacia una plataforma RAG de Threat Intelligence

