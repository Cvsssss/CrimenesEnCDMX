# SOW: Proyecto ZONA-A: Pipeline batch de cálculo de riesgo delictivo por zona y fecha

**Consultora:** Data Bloom
**Cliente:** SAFEZONE MÉXICO (Área de Inteligencia de Datos)
**Respuesta al RFP del:** 6 de octubre de 2026
**Versión:** 1.0
**Equipo:**
- Amir Cuevas Quintana — Ing. en Computación — Líder de proyecto y arquitecto de datos
- Uriel Ivan Guerrero Lopez — Ingeniería en Computación (UNAM) — Ingeniero de datos (ingesta y orquestación)
- Axel Alejandro Quintos Delgadillo — Ing. en Computación — Ingeniero de datos (calidad y modelo de datos)
- Aaron Rodrigo Leon Pérez — Ing. en Computación — Científico de datos
- Estrada Luna Eduardo Antonio — Ing. en Computación — DevOps, pruebas y documentación

---

## 1. Resumen ejecutivo

SAFEZONE realiza recorridos diariamente para transporte de efectivo, por que se usaron datos públicos de delitos repartidos en varias fuentes, con formatos y ubicaciones distintas. Con ello vamos a construir un proceso automático que todos los días descargue esos datos, los limpie, los guarde con su historial y calcule un nivel de riesgo por zona y periodo.
Los resultados quedarán disponibles en un servicio de consulta y en archivos listos para sus herramientas de visualización, siempre con su fecha, origen y limitaciones.
Entregamos en 8 semanas (6 hitos), con capacitación y 30 días de soporte posterior.
El precio total es de **$1,216,800 MXN + IVA** ($1,411,488 con IVA) y la operación mensual estimada es de **~$7,500 MXN**, ambos por debajo de los límites del RFP.

## 2. Alcance

**Dentro de alcance:**
- Ingesta automatizada de fuentes públicas autorizadas (SESNSP, INEGI y las fuentes abiertas que se confirmen en el Hito 0), guardando siempre una copia del dato original sin modificar.
- Limpieza, normalización y validación de fecha, categoría delictiva y ubicación, con reporte de calidad por lote.
- Cálculo de riesgo por zona y periodo con parámetros configurables y metodología documentada.
- Base de datos histórica con registros procesados, indicadores y bitácora de ejecuciones.
- Proceso batch programado (mínimo una vez al día), con alertas, registro de errores y reintentos sin duplicar registros.
- Servicio de consulta (API REST documentada) y exportación de archivos estructurados (CSV, GeoJSON y Parquet).
- Infraestructura en AWS definida como código, en una cuenta propiedad del cliente.
- Manual técnico y operativo, documentación de arquitectura, decisiones técnicas (ADR) y metodología.
- Sesión de transferencia de conocimiento y capacitación remota.
- 30 días naturales de soporte posterior a la puesta en producción para corrección de defectos.

**Fuera de alcance:**
- Aplicación móvil o página web para usuarios finales: el RFP lo excluye; entregamos datos listos para las herramientas de visualización que ya usa SAFEZONE.
- Cámaras, sensores, GPS o infraestructura física: el proyecto trabaja solo con datos ya publicados.
- Bases policiales restringidas, expedientes judiciales o datos personales: el RFP lo prohíbe y no aportan a un indicador agregado por zona.
- Identificación de personas o predicción de delitos individuales: el resultado es un nivel de riesgo agregado por zona y periodo, no una predicción sobre personas.
- Respuesta policial, despacho de emergencias o intervención en campo: no es un servicio tecnológico de datos.
- Sesiones presenciales: el trabajo es remoto; si se requieren, se cotizan aparte (ver sección 10).
- Fuentes de paga o que requieran convenio: solo se integran fuentes públicas cuyas condiciones de uso lo permitan.

## 3. Entregables y criterios de aceptación

| # | Entregable | Criterio de aceptación | Hito |
|---|---|---|---|
| E1 | Documento de arranque: fuentes confirmadas, nivel geográfico, reglas de "registro válido" y "zona con información suficiente", ADRs iniciales | Documento firmado por el responsable del cliente | 0 |
| E2 | Pipeline de ingesta con zona de datos originales (inmutable) | Al menos 90% de los registros válidos de cada lote se procesa con éxito en 3 lotes consecutivos (N1); el dato original de cada lote queda guardado y es recuperable | 1 |
| E3 | Proceso de limpieza, normalización y validación + reporte de calidad por lote | Al menos 95% de los registros utilizables tiene fecha, categoría y ubicación normalizadas (N2); cada lote genera su reporte de calidad con registros rechazados y motivo | 2 |
| E4 | Motor de cálculo de riesgo con parámetros configurables y metodología documentada | Se generan indicadores para al menos 85% de las zonas con información suficiente (N3); cambiar un parámetro no requiere modificar código; metodología revisada por el cliente | 3 |
| E5 | Base de datos histórica (registros, indicadores, bitácora de lotes) | El 100% de los lotes registra fuente, fecha de actualización, versión de metodología y estado (N6) | 4 |
| E6 | Orquestación batch programada con reintentos | 7 días consecutivos de ejecución diaria automática; en una prueba de falla forzada el reintento termina sin registros duplicados (mismo conteo antes y después) (N4) | 4 |
| E7 | Servicio de consulta (API REST) y exportación de archivos | Consulta por zona y rango de fechas; los archivos se cargan sin ajustes en la herramienta de visualización indicada por el cliente; cada resultado incluye fecha de actualización y limitaciones (N5) | 4 |
| E8 | Plan y reporte de pruebas (calidad, batch, rendimiento, recuperación, validación de resultados) | Todas las pruebas críticas pasan; reporte firmado por el cliente | 5 |
| E9 | Manual técnico y operativo, documentación de arquitectura y ADRs | El equipo del cliente ejecuta, reintenta y agrega una fuente siguiendo solo el manual, sin apoyo del proveedor | 5 |


## 4. Trazabilidad con el RFP

| Necesidad del RFP | Entregable(s) que la cubre(n) | Estado |
|---|---|---|
| N1 Integración automatizada de fuentes públicas | E1, E2 | Cubierta |
| N2 Limpieza y estandarización | E3 | Cubierta |
| N3 Cálculo de riesgo por zona y fecha | E1, E4 | Cubierta (la definición de "información suficiente" se acuerda en E1) |
| N4 Automatización por lotes | E6 | Cubierta |
| N5 Consulta de resultados | E7 | Cubierta |
| N6 Trazabilidad y confiabilidad | E2, E5 | Cubierta |

## 5. Enfoque técnico (alto nivel)

```
 Fuentes públicas            AWS (cuenta de SAFEZONE)
 (SESNSP, INEGI,     ┌───────────────────────────────────────────────────────────┐
  datos abiertos)    │  EventBridge Scheduler (diario)                           │
        │            │          │                                                │
        ▼            │          ▼                                                │
   [Ingesta] ───────►│  Step Functions ──► Tareas (Python)               │
                     │      │  reintentos      1. Descarga    → S3 /crudo        │
                     │      │  y alertas       2. Limpieza    → S3 /limpio       │
                     │      ▼                  3. Cálculo     → PostgreSQL+PostGIS│
                     │  CloudWatch + SNS        4. Exportación → S3 /resultados   │
                     │                                                           │
                     │  API Gateway + Lambda (solo lectura) ◄── clientes con     │
                     │  Secrets Manager (credenciales)          API key          │
                     └───────────────────────────────────────────────────────────┘
                                          │
                                          ▼
                      Herramientas de visualización existentes del cliente
```

Decisiones clave:
- **Cómputo que solo cobra cuando corre** (Fargate + Step Functions): el proceso es diario y corto, así que no pagamos servidores encendidos todo el día. Esto mantiene la operación muy por debajo de $25,500 MXN.
- **PostgreSQL con PostGIS**: permite cruzar puntos con colonias, municipios o cuadrícula y guardar historial con una herramienta conocida por el equipo del cliente.
- **Dato original intocable**: cada descarga se guarda tal cual en S3 con versionado; si cambia la metodología, se puede recalcular todo el histórico.
- **Reintentos sin duplicados**: cada lote tiene un identificador y cada registro una huella (hash); se inserta o actualiza, nunca se duplica.
- **Unidad geográfica en dos niveles**: municipio como base nacional (SESNSP) y colonia o cuadrícula hexagonal donde haya coordenadas.
- **Metodología de riesgo explicable**: tasa por 100 mil habitantes ponderada por tipo de delito, ajustada para zonas con pocos casos, comparada contra su propio historial, más una tendencia de corto plazo con su rango de incertidumbre. Resultado: nivel (bajo, medio, alto, muy alto), puntaje, tendencia y nota de confianza. 
- **Infraestructura como código** (Terraform) y todo el código en Git del cliente, para que no dependa de nosotros al terminar.

## 6. Plan por hitos

Fechas calculadas suponiendo firma del contrato el lunes 9 de noviembre de 2026.

| Hito | Qué se entrega | Evidencia | Fecha | Horas |
|---|---|---|---|---|
| 0 Arranque | E1: fuentes confirmadas, nivel geográfico, reglas de validación, cuenta AWS lista, ADRs iniciales | Documento de arranque firmado | Semana 1 — 13 nov 2026 | 120 |
| 1 Ingesta y datos originales | E2: descarga automática de fuentes y almacenamiento crudo | Demo de 3 lotes con métricas de procesamiento ≥ 90% | Semana 3 — 27 nov 2026 | 280 |
| 2 Limpieza y calidad | E3: normalización, validación y reportes de calidad | Reporte de calidad con ≥ 95% de registros normalizados | Semana 4 — 4 dic 2026 | 280 |
| 3 Motor de riesgo | E4: cálculo por zona y periodo, parámetros configurables, metodología | Indicadores en ≥ 85% de zonas con información suficiente; revisión metodológica con el cliente | Semana 6 — 18 dic 2026 | 300 |
| 4 Batch, histórico y consulta | E5, E6, E7: orquestación diaria, reintentos, bitácora, API y archivos | Bitácora de 7 días de ejecución, prueba de falla y reintento, demo de API | Semana 7 — 23 dic 2026 | 260 |
| 5 Pruebas, documentación y producción | E8, E9, E10: pruebas finales, manuales, capacitación y salida a producción | Reporte de pruebas firmado, sesión grabada, sistema en producción | Semana 8 — 31 dic 2026 | 160 |
| **Total** | | | | **1,400** |

Revisión con el cliente cada semana y demo al cierre de cada hito. El soporte de 30 días inicia al aceptar el Hito 5.

## 7. Equipo y responsabilidades

| Rol | Nombre | Responsabilidad principal | Reporta a |
|---|---|---|---|
| Líder de proyecto y arquitecto de datos | [Nombre] | Plan, arquitectura, ADRs, relación con el cliente, control de cambios | Responsable del proyecto en SAFEZONE |
| Ingeniero de datos (ingesta y orquestación) | Amir Cuevas Quintana | Conectores a fuentes, almacenamiento crudo, Step Functions, reintentos y bitácora de lotes | Líder de proyecto |
| Ingeniero de datos (calidad y modelo) | [Nombre] | Reglas de limpieza y validación, modelo de base de datos, reportes de calidad | Líder de proyecto |
| Científico de datos geoespaciales | [Nombre] | Unidad geográfica, metodología de riesgo, validación de resultados y limitaciones | Líder de proyecto |
| DevOps, pruebas y documentación | [Nombre] | Terraform, CI/CD, monitoreo, plan de pruebas, manuales y capacitación | Líder de proyecto |

## 8. Supuestos y dependencias del cliente

**Supuestos:**
- Las fuentes públicas (SESNSP, INEGI y datos abiertos que se confirmen) siguen disponibles y permiten su uso para este fin.
- SAFEZONE aporta o autoriza una cuenta de AWS a su nombre; los costos de nube se facturan directo al cliente.
- Los precios son en MXN y antes de IVA.
- Las sesiones de trabajo y capacitación son remotas.
- Tipo de cambio de referencia para estimar AWS: $19 MXN por dólar.

**Datos que no cuadran (y cómo los resolvemos):**
- **Ejecución diaria vs. fuentes mensuales:** el SESNSP publica incidencia por municipio de forma mensual. El batch correrá todos los días como pide N4, pero solo recalculará cuando detecte datos nuevos; el riesgo "por fecha" será mensual donde la fuente sea mensual y más fino solo donde existan fechas diarias.
- **"Predictivo" vs. "sin predicción determinista":** entregamos nivel de riesgo y tendencia con rango de incertidumbre por zona, no predicciones de delitos concretos.
- **Términos sin definir en N1 y N3:** "registro válido" y "zona con información suficiente" se definen y firman en el Hito 0; sin eso no se pueden medir los porcentajes.
- **Nivel geográfico abierto:** colonia, cuadrícula o municipio depende de qué fuentes traen coordenadas; se decide en el Hito 0.
- **Cifra negra:** los registros oficiales no incluyen delitos no denunciados; cada indicador lo dirá en su nota de limitaciones.
- **Plazo con temporada decembrina:** las semanas 7 y 8 caen en fiestas; por eso concentramos el trabajo pesado antes del 18 de diciembre.

**Dependencias del cliente:**
- Un responsable del proyecto con autoridad para aprobar hitos y cambios.
- Respuesta a validaciones en un máximo de 3 días hábiles.
- Acceso a la cuenta de AWS en la semana 1.
- Nombre y formato de la herramienta de visualización destino antes del Hito 3.
- Aprobación de la metodología de riesgo en el Hito 3.
- Disponibilidad del equipo de ingeniería y analítica para la capacitación en la semana 8.

## 9. Riesgos

| Riesgo | Probabilidad | Impacto | Qué haremos |
|---|---|---|---|
| Cambia el formato o la URL de una fuente pública | Media | Alto | Validación de estructura antes de procesar; el lote falla de forma controlada, alerta al equipo y no contamina la base; conectores separados por fuente |
| Una fuente deja de publicarse o se retrasa | Media | Medio | Indicadores muestran la fecha del último dato; se marca la zona como "desactualizada" en lugar de inventar valores |
| Calidad baja de datos (ubicaciones vacías o colonias mal escritas) | Alta | Alto | Catálogos oficiales de INEGI para normalizar, reglas de corrección documentadas y reporte de registros rechazados con su motivo |
| Sesgo por zonas con más denuncias o más cobertura | Alta | Alto | Ajuste para zonas con pocos casos, nota de confianza en cada indicador, revisión de resultados con analistas del cliente |
| Interpretación errónea del indicador por clientes corporativos | Media | Alto | Cada resultado incluye fecha, versión de metodología y limitaciones; guía breve de interpretación en el manual |
| Retraso en definiciones del Hito 0 (fuentes, nivel geográfico) | Media | Alto | Propuesta inicial preparada por nosotros para que el cliente solo valide; cualquier retraso mayor a 3 días hábiles se trata por control de cambios |
| Costos de nube por encima de lo estimado | Baja | Medio | Alertas de presupuesto en AWS al 50%, 80% y 100% de $25,500 MXN; cómputo bajo demanda y sin NAT Gateway |
| Ausencias por temporada decembrina | Media | Medio | Trabajo crítico terminado antes del 18 de diciembre; fechas de revisión acordadas desde el arranque |
| Uso de una fuente fuera de sus condiciones de uso | Baja | Alto | Revisión de términos de cada fuente en el Hito 0; registro de licencia por fuente en la bitácora |

## 10. Presupuesto

| Concepto | Horas | Tarifa MXN/h | Subtotal MXN |
|---|---|---|---|
| Líder de proyecto y arquitecto de datos | 240 | 900 | 216,000 |
| Ingeniero de datos (ingesta y orquestación) | 320 | 700 | 224,000 |
| Ingeniero de datos (calidad y modelo) | 320 | 700 | 224,000 |
| Científico de datos geoespaciales | 280 | 800 | 224,000 |
| DevOps, pruebas y documentación | 240 | 600 | 144,000 |
| Colchón de imprevistos (15%) | | | 154,800 |
| Costo estimado de AWS (desarrollo y producción durante 8 semanas) | | | 30,000 |
| **Total** | **1,400** | | **1,216,800** |

- **Comparación contra el RFP:** el total es de $1,216,800 MXN + IVA ($1,411,488 MXN con IVA), por debajo del máximo de $1,500,000 MXN en ambos casos. No fue necesario recortar alcance.
- El colchón cubre también los 30 días de soporte posterior. Si no se usa, no se cobra.
- **Sesión presencial opcional** (no incluida): $15,000 MXN por sesión en la CDMX, solo si el cliente la solicita.

**Costo mensual estimado de operación (después de la entrega):**

| Servicio AWS | MXN/mes |
|---|---|
| PostgreSQL + PostGIS (RDS, 100 GB, respaldos de 7 días) | 2,500 |
| Tareas batch en Fargate (1 a 2 horas diarias) | 600 |
| Step Functions y EventBridge | 100 |
| S3 (datos originales, limpios y resultados, con versionado) | 300 |
| API Gateway + Lambda (consultas de clientes) | 700 |
| CloudWatch y SNS (bitácoras, métricas y alertas) | 800 |
| Secrets Manager y KMS | 150 |
| Red, respaldos adicionales y transferencia | 600 |
| Margen de seguridad (30%) | 1,750 |
| **Total estimado** | **~7,500** |

Equivale a cerca del 30% del límite de $25,500 MXN; la cifra se valida con la calculadora de AWS en el Hito 0.

## 11. Control de cambios y comunicación

- **Pedir algo nuevo:** el cliente envía una solicitud de cambio por correo al líder de proyecto, indicando qué necesita y por qué.
- **Evaluación:** en 2 días hábiles respondemos con el impacto en horas, costo, fechas y riesgos.
- **Aprobación:** el cambio solo entra si lo firman el responsable de SAFEZONE y nuestro líder de proyecto. Lo no aprobado no se trabaja.
- **Registro:** todos los cambios quedan en una bitácora compartida con número, fecha y estado.
- **Revisión semanal:** reunión virtual de 45 minutos cada martes con avance, riesgos y pendientes del cliente.
- **Demo por hito:** sesión al cierre de cada hito para revisar la evidencia de aceptación.
- **Reporte semanal:** resumen por correo cada viernes. Canal de chat compartido para dudas del día a día; las decisiones formales siempre por correo.

## 12. Aceptación y pagos

- Un hito se da por terminado cuando el cliente firma el acta de aceptación con base en los criterios de la sección 3.
- El cliente tiene 5 días hábiles para revisar. Si hay observaciones, las corregimos en un máximo de 5 días hábiles y se vuelve a revisar solo lo observado.
- Si pasan 5 días hábiles sin respuesta ni observaciones, el hito se considera aceptado.
- El costo de AWS durante el proyecto se paga directo en la cuenta del cliente y no forma parte de estos pagos.

| Hito | % de honorarios | Monto MXN (antes de IVA) |
|---|---|---|
| 0 Arranque | 10% | 118,680 |
| 1 Ingesta y datos originales | 15% | 178,020 |
| 2 Limpieza y calidad | 15% | 178,020 |
| 3 Motor de riesgo | 20% | 237,360 |
| 4 Batch, histórico y consulta | 20% | 237,360 |
| 5 Pruebas, documentación y producción | 20% | 237,360 |
| **Total honorarios** | **100%** | **1,186,800** |

Los porcentajes se aplican a los honorarios ($1,186,800 MXN = horas + colchón). Los $30,000 MXN estimados de AWS se cobran directo en la cuenta del cliente, con lo que el total del proyecto queda en $1,216,800 MXN + IVA.
