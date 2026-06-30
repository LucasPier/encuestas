# Sistema de Gestión de Encuestas — Resumen para Profesionales

> **Versión:** 1.1.0 · **Fecha:** Junio 2026

---

## ¿Qué es?

Plataforma integral para **organizar, ejecutar y analizar encuestas políticas y de opinión pública**. Permite gestionar trabajos de campo con encuestadores en terreno, capturar datos en tiempo real incluso sin conexión a internet, analizar resultados con herramientas avanzadas y generar informes profesionales.

El alcance inicial es la **Provincia de Santa Fe** (con foco en Rosario), con diseño preparado para escalar a cobertura nacional.

---

## Roles

| Rol | Responsabilidades principales |
|---|---|
| **Encuestador** | Toma encuestas en el campo desde su celular. Ve sus trabajos asignados y el progreso de cuotas. |
| **Supervisor** | Monitorea el progreso de los trabajos de campo, gestiona la asignación de encuestadores a puntos y supervisa incidencias en tiempo real. |
| **Analista de Datos** | Consulta resultados, construye tablas cruzadas, aplica ponderaciones y genera informes. |
| **Administrador** | Gestión completa: usuarios, encuestas, trabajos de campo, puntos de muestreo y configuración. |

---

## Trabajos de Campo

Un **trabajo de campo** agrupa una o más encuestas, un conjunto de puntos de muestreo y los encuestadores asignados, con fechas definidas.

Tiene un ciclo de vida claro: `Borrador → Publicado → Activo → Pausado → Finalizado → Archivado`. Las encuestas en el campo solo pueden tomarse cuando el trabajo está **Activo**.

**Cuotas de muestra:** Se definen por punto de muestreo, combinando un total objetivo y cuotas demográficas (ej.: 50% mujeres, 30% entre 18–35 años). El sistema alerta al encuestador si una cuota ya está completa, pero le permite continuar si así lo decide.

**Asignación de encuestadores:** Se soportan dos modalidades que pueden coexistir:
- **Asignada:** encuestadores asignados nominalmente a puntos específicos (con horario opcional).
- **Abierta:** cualquier encuestador activo puede tomar muestras en puntos marcados como abiertos.

---

## Puntos de Muestreo

Ubicaciones físicas reutilizables (con nombre, dirección y coordenadas GPS seleccionables en mapa). Se organizan en **grupos** para facilitar su asignación a trabajos de campo. Un mismo punto puede usarse en múltiples trabajos.

---

## Encuestas

### Tipos de Preguntas

| Tipo | Descripción |
|---|---|
| **Opción múltiple** | El encuestado puede seleccionar varias respuestas de una lista. |
| **Opción única** | Selección de una sola respuesta. |
| **Respuesta abierta** | Campo de texto libre. |
| **Escala Likert** | Configurable: 1–5 o 1–10, con etiquetas en los extremos. |
| **Escala numérica (slider)** | El encuestado elige un valor dentro de un rango numérico. |
| **Ranking** | El encuestado ordena opciones por preferencia. |
| **Matriz** | Conjunto de sub-preguntas que comparten las mismas opciones de respuesta. |
| **Con imagen en opciones** | Opción única o múltiple donde cada opción lleva una imagen (ej.: fotos de candidatos, logos de partidos). |

Además, **cualquier tipo de pregunta** puede incluir una imagen en el enunciado (ilustrativa o como elemento central de la pregunta).

> 💬 **Consulta:** ¿Los tipos de pregunta disponibles cubren sus necesidades habituales? ¿Hay algún tipo que utilicen frecuentemente y no esté contemplado?

### Lógica Condicional

El sistema soporta dos mecanismos de ramificación:

- **Skip logic (salto):** redirigir al encuestado a una pregunta o sección específica según su respuesta (ej.: si responde "No", saltar directamente a la pregunta 12).
- **Display logic (visualización):** mostrar u ocultar preguntas según condiciones (ej.: mostrar la pregunta solo si el encuestado es mayor de 35 años).

Las condiciones pueden combinar múltiples criterios con operadores **AND / OR**.

Las preguntas también pueden organizarse en **secciones**, lo que facilita la navegación y la aplicación de lógica condicional a bloques completos.

> 💬 **Consulta:** ¿La combinación de skip logic y display logic con AND/OR es suficiente para los diseños de encuesta que habitualmente trabajan? ¿Necesitan algún tipo de ramificación más compleja?

### Otras características de encuestas

- **Versionado automático:** al modificar una encuesta ya publicada, se crea una nueva versión preservando las respuestas históricas vinculadas a la versión original.
- **Plantillas reutilizables:** las encuestas pueden marcarse como plantillas para usarlas como base en trabajos futuros.
- **Duplicación:** copiar una encuesta existente como punto de partida.
- **Etiquetas:** tanto las encuestas como las preguntas individuales pueden etiquetarse para facilitar la búsqueda y el análisis cruzado entre trabajos.

---

## Aplicación para Encuestadores (campo)

La app está diseñada para ser **extremadamente simple** de usar en campo, desde un celular y con una sola mano. Funciona **sin conexión a internet**: los trabajos y encuestas se sincronizan previamente y las respuestas se guardan en el dispositivo hasta poder subirse.

**Al registrar cada encuesta, el sistema captura automáticamente:**
- Fecha y hora de inicio y fin.
- Coordenadas GPS (si están disponibles y el encuestador otorgó permiso).
- Punto de muestreo y encuestador.

**El encuestador registra al inicio de cada entrevista:** edad y género del encuestado (fijos), más las variables demográficas adicionales que se hayan configurado para ese trabajo (ej.: nivel educativo, zona de residencia, etc.).

**Otras capacidades:**
- Ver el estado de cuotas del punto donde está trabajando, con indicadores de color (verde / amarillo / rojo).
- Pausar una encuesta en curso y retomarla después.
- Reportar incidencias (punto cerrado, rechazo del encuestado, problemas técnicos, etc.) desde la misma app.

**Supervisores y administradores** pueden ver en tiempo real en un mapa: la ubicación de los encuestadores activos, el estado de cuotas por punto y las encuestas que se están tomando en ese momento.

---

## Análisis de Resultados

El módulo de análisis permite trabajar con datos de una o múltiples encuestas y trabajos de campo combinados.

**Herramientas disponibles:**
- **Tablas cruzadas:** cruce de cualquier par de preguntas o variables demográficas (ej.: intención de voto por género, franja etaria o zona).
- **Filtros combinados:** por rango de fechas, puntos de muestreo, encuestadores, segmentos demográficos o respuestas específicas.
- **Evolución histórica:** cómo cambian los resultados a lo largo del tiempo.
- **Comparaciones** entre distintas encuestas, versiones o trabajos de campo.

**Ponderación:**
- **Manual:** el analista asigna pesos a grupos demográficos o geográficos.
- **Rim Weighting / Raking:** ajuste iterativo automático para que múltiples variables demográficas simultáneas coincidan con los targets de población definidos.
- **Ponderación por punto de muestreo:** asignar pesos a zonas geográficas (ej.: distintas "asistencias electorales esperadas" por barrio o localidad según datos históricos). Los escenarios se guardan con nombre y pueden compararse entre sí.

**Gráficos disponibles:** barras, torta/dona, línea, área, mapa de calor, radar, burbuja, treemap, mapa coroplético (resultados por zona geográfica) y tablas de frecuencias y porcentajes.

**Exportación:** CSV (para uso en Excel, R, SPSS, etc.), Excel (.xlsx) y PDF.

---

## Informes

Los analistas y administradores pueden crear informes mediante un **editor visual**, combinando libremente texto enriquecido, títulos, gráficos generados desde los datos, tablas e imágenes.

Los gráficos dentro del informe son **configurables directamente en el editor**: tipo de gráfico, pregunta fuente, filtros y ponderación aplicada. Un informe puede combinar datos de múltiples encuestas y trabajos de campo.

**Visibilidad:**
- **Privado:** accesible solo para usuarios del sistema o mediante enlace seguro con expiración.
- **Público:** publicado en el sitio web de acceso público (en una versión futura del sistema).

**Exportación:** PDF, Word (.docx) y enlace web.

---

## Funcionalidades previstas para versiones futuras

Entre las más relevantes para el trabajo estadístico:

| Funcionalidad | Notas |
|---|---|
| Encuestas telefónicas | Integración con plataformas de llamadas. |
| Encuestas en línea (autoadministradas) | Formulario web accesible por URL pública. |
| Integración con herramientas de BI | Conexión directa con Power BI, Tableau, etc. |
| Extensión geográfica nacional | Configuración completa de provincias y municipios argentinos. |

---

> **Queda abierta la consulta** sobre cualquier aspecto del diseño, especialmente los tipos de pregunta y la lógica condicional, donde la experiencia de los profesionales que trabajan en campo y en análisis puede aportar mejoras muy valiosas antes de comenzar el desarrollo.
