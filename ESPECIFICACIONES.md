# ESPECIFICACIONES DEL SISTEMA — Encuestas Políticas y de Opinión Pública

> **Versión:** 1.1.0  
> **Fecha:** Junio 2026  
> **Filosofía:** Spec-as-Source — este documento es la única fuente de verdad del sistema.  
> **Nota:** El historial de cambios se gestiona exclusivamente mediante commits de Git.

---

## Tabla de Contenidos

1. [Visión General](#1-visión-general)
2. [Stack Tecnológico](#2-stack-tecnológico)
3. [Arquitectura del Sistema](#3-arquitectura-del-sistema)
4. [Estructura del Monorepo](#4-estructura-del-monorepo)
5. [Usuarios y Roles](#5-usuarios-y-roles)
6. [Autenticación y Seguridad](#6-autenticación-y-seguridad)
7. [Trabajos de Campo](#7-trabajos-de-campo)
8. [Puntos de Muestreo](#8-puntos-de-muestreo)
9. [Encuestas](#9-encuestas)
10. [Toma de Muestras — PWA Encuestadores](#10-toma-de-muestras--pwa-encuestadores)
11. [Resultados y Análisis](#11-resultados-y-análisis)
12. [Generación de Informes](#12-generación-de-informes)
13. [Notificaciones](#13-notificaciones)
14. [Sitio Web de Publicación](#14-sitio-web-de-publicación)
15. [PWA de Encuestadores — UX/UI](#15-pwa-de-encuestadores--uxui)
16. [PWA de Administración — UX/UI](#16-pwa-de-administración--uxui)
17. [API RESTful](#17-api-restful)
18. [Infraestructura y Despliegue](#18-infraestructura-y-despliegue)
19. [Testing](#19-testing)
20. [Filosofía Spec-as-Source y Desarrollo con IA](#20-filosofía-spec-as-source-y-desarrollo-con-ia)
21. [Alcance Geográfico y Escalabilidad](#21-alcance-geográfico-y-escalabilidad)
22. [Consideraciones Futuras](#22-consideraciones-futuras)

---

## 1. Visión General

### 1.1 Descripción

Sistema integral para la organización, ejecución y análisis de encuestas políticas y de opinión pública. Permite gestionar trabajos de campo con encuestadores en terreno, capturar datos en tiempo real (incluso sin conexión a internet), analizar resultados con herramientas avanzadas y generar informes profesionales.

### 1.2 Usos Principales

- Organización y gestión de encuestas políticas y de opinión pública.
- Recolección de datos en campo, en tiempo real, con soporte offline.
- Análisis de resultados con segmentación, ponderación y visualizaciones avanzadas.
- Generación de informes con conclusiones, gráficos y tablas exportables.
- Publicación de resultados e informes en un sitio web accesible al público.

### 1.3 Idioma

Toda la interfaz de usuario, el código fuente (variables, funciones, comentarios, documentación) y los archivos de especificación estarán en **español**. La arquitectura estará preparada para incorporar internacionalización (i18n) en el futuro, aunque en la primera versión solo se soportará español.

---

## 2. Stack Tecnológico

### 2.1 Backend

| Componente          | Tecnología                         |
|---------------------|------------------------------------|
| Entorno de ejecución | Node.js                           |
| Framework           | Fastify                            |
| Lenguaje            | TypeScript                         |
| ORM                 | Prisma                             |
| Base de datos       | MySQL / MariaDB                    |
| Caché / PubSub      | Redis                              |
| Almacenamiento de archivos | MinIO (self-hosted, S3-compatible) |
| Autenticación       | JWT (access + refresh tokens)      |
| Documentación API   | Swagger / OpenAPI (autogenerado)   |
| Emails transaccionales | SMTP propio                     |
| WebSockets          | Fastify WebSocket plugin + Redis   |
| Generación de PDF   | Puppeteer / Playwright (server-side) |

### 2.2 Frontend (todas las apps)

| Componente          | Tecnología                         |
|---------------------|------------------------------------|
| Framework           | React + Vite                       |
| Lenguaje            | TypeScript                         |
| Estilos             | Tailwind CSS + componentes propios |
| Gráficos            | Recharts                           |
| Editor de informes  | Editor de bloques tipo Notion (TipTap o similar, con bloques customizados) |
| Mapas               | Google Maps JavaScript API         |
| PWA (offline)       | Service Worker + IndexedDB (ambas PWAs) |
| PDF (cliente)       | react-to-pdf o similar (vista previa) |

### 2.3 Infraestructura y Herramientas

| Componente          | Tecnología                         |
|---------------------|------------------------------------|
| Gestión de monorepo | Nx                                 |
| Contenedores        | Docker + Docker Compose            |
| Despliegue          | VPS propio                         |
| Backup              | Automático diario, retención 30 días |
| Testing unitario/integración | Vitest                  |
| Testing E2E         | Playwright                         |
| CI/CD               | No definido para la primera versión |

---

## 3. Arquitectura del Sistema

El sistema sigue una arquitectura de **múltiples aplicaciones desacopladas** que se comunican a través de una **API RESTful centralizada**.

```
┌────────────────────────────────────────────────────────────────┐
│                        CLIENTE                                 │
│                                                                │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐  │
│  │ PWA Encuestadores│  │  PWA Administrac.│  │ Sitio Web    │  │
│  │  (React + Vite)  │  │  (React + Vite)  │  │ Público      │  │
│  │  Service Worker  │  │  Service Worker  │  │ (React+Vite) │  │
│  │  IndexedDB       │  │  IndexedDB       │  │              │  │
│  └────────┬─────────┘  └────────┬─────────┘  └──────┬───────┘  │
│           │ HTTPS               │ HTTPS             │ HTTPS    │
└───────────┼─────────────────────┼───────────────────┼──────────┘
            │                     │                   │
┌───────────┼─────────────────────┼───────────────────┼──────────┐
│           │         SERVIDOR    │                   │          │
│  ┌────────▼─────────────────────▼───────────────────▼───────┐  │
│  │                     API RESTful                          │  │
│  │              Node.js + Fastify + TypeScript              │  │
│  │              Swagger/OpenAPI autogenerado                │  │
│  └──────────────────────────────────────────────────────┬───┘  │
│                          │                              │      │
│              ┌───────────┴───────────┐                  │      │
│              │                       │                  │      │
│  ┌───────────▼──────┐  ┌─────────────▼─────┐            │      │
│  │  MySQL/MariaDB   │  │     Redis         │            │      │
│  │  (Prisma ORM)    │  │  Caché + WebSocket│            │      │
│  └──────────────────┘  └───────────────────┘            │      │
│                                                         │      │
│  ┌──────────────────────────────────────────────────────▼──┐   │
│  │                    MinIO                                │   │
│  │       Almacenamiento de archivos (imágenes, PDFs)       │   │
│  └─────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────┘
```

### 3.1 Comunicación en Tiempo Real

Los supervisores y administradores pueden monitorear el progreso de los trabajos de campo en tiempo real. Se utilizan **WebSockets** (gestionados con Redis como broker) para transmitir:

- Nuevas encuestas completadas.
- **Encuestas iniciadas y en proceso** (quién está tomando una encuesta en este momento).
- Cambios en el estado de los trabajos de campo.
- Ubicación en tiempo real de encuestadores activos (cuando GPS disponible).
- Alertas e incidencias reportadas por encuestadores.

> **Nota:** El tracking GPS en tiempo real y el seguimiento de encuestas en proceso requieren conexión a Internet activa tanto en el dispositivo del encuestador como en el del supervisor/administrador. En ausencia de conexión, estos datos no se transmiten en vivo, aunque las respuestas se almacenan localmente y se sincronizan al reconectarse.

---

## 4. Estructura del Monorepo

El proyecto es un **monorepo gestionado con Nx** con la siguiente estructura:

```
/
├── apps/
│   ├── api/                    # API RESTful (Node.js + Fastify)
│   ├── pwa-encuestadores/      # PWA para encuestadores (React + Vite)
│   ├── pwa-administracion/     # PWA de administración (React + Vite)
│   └── sitio-publico/          # Sitio web de publicación (React + Vite)
├── libs/
│   ├── tipos-compartidos/      # Tipos TypeScript compartidos entre apps
│   ├── validaciones/           # Esquemas de validación (Zod o similar)
│   └── utilidades/             # Funciones utilitarias comunes
├── .spec/                      # Especificaciones Spec-as-Source (Markdown)
│   ├── ESPECIFICACIONES.md     # Documento principal (este archivo)
│   ├── api/                    # Specs de endpoints y modelos de datos
│   ├── pwa-encuestadores/      # Specs de la PWA de encuestadores
│   ├── pwa-administracion/     # Specs de la PWA de administración
│   └── sitio-publico/          # Specs del sitio público
├── docker-compose.yml          # Configuración de servicios (MySQL, Redis, MinIO)
├── nx.json
└── package.json
```

---

## 5. Usuarios y Roles

### 5.1 Gestión de Usuarios

- **Creación:** Solo los administradores (y súper administradores) pueden crear nuevos usuarios. No existe registro libre. Al crear un usuario, el administrador puede:
  - Asignar una **contraseña inicial de forma manual**, o
  - Enviar un **enlace por correo electrónico** para que el propio usuario establezca su contraseña la primera vez que accede.
- **Eliminación:** Borrado lógico (soft delete). Los registros se marcan como eliminados pero permanecen en la base de datos para mantener integridad histórica y facilitar auditoría.
- **Archivado:** Los usuarios pueden ser archivados (desactivados) sin eliminarse.

### 5.2 Roles del Sistema

Los roles pueden **acumularse**, excepto para administradores y súper administradores, que son exclusivos.

| Rol                  | Descripción |
|----------------------|-------------|
| **Encuestador**      | Realiza encuestas en el campo. Accede a la PWA de encuestadores. Ve trabajos asignados, toma encuestas y reporta incidencias. |
| **Supervisor**       | Accede a la PWA de administración con permisos limitados. Puede gestionar la asignación de encuestadores a puntos de muestreo, supervisar el progreso de los trabajos de campo y monitorear a todos los encuestadores del sistema. |
| **Analista de datos** | Accede a la PWA de administración con permisos limitados. Puede consultar y analizar resultados, generar informes y hacer seguimiento de trabajos de campo. No puede gestionar usuarios ni configurar el sistema. |
| **Administrador**    | Acceso completo a la PWA de administración. Gestiona usuarios (excepto administradores), trabajos de campo, encuestas, puntos de muestreo, resultados y configuración del sistema. No puede crear ni eliminar administradores. |
| **Súper Administrador** | Mismas capacidades que el administrador, con la diferencia de que es el único que puede crear y eliminar cuentas de administrador. Gestiona la organización y la administración global del sistema. |

### 5.3 Información del Perfil de Usuario

Cada usuario tendrá un perfil con los siguientes campos:

| Campo                | Obligatorio | Notas |
|----------------------|-------------|-------|
| Nombre completo      | Sí          | |
| Username             | Sí          | Identificador único en el sistema. Usado para login. Solo letras, números y guiones bajos. |
| Correo electrónico   | Sí          | Debe validarse. Usado para login y notificaciones. |
| Número de teléfono   | Sí          | Usado para login. |
| Contraseña           | Sí          | Hasheada, nunca almacenada en texto plano. |
| Imagen de perfil     | No          | Almacenada en MinIO. |
| Rol(es) asignado(s)  | Sí          | Asignado por administradores. |
| Estado               | Sí          | Activo, archivado, eliminado (soft). |

El propio usuario puede actualizar su nombre, teléfono, imagen de perfil y contraseña. Los administradores pueden actualizar cualquier campo e impersonar o archivar cualquier usuario (excepto súper administradores).

### 5.4 Grupos de Encuestadores

Los encuestadores pueden organizarse en **grupos** para facilitar la asignación masiva a los trabajos de campo.

- Los grupos son entidades gestionables (crear, editar, eliminar) con nombre y descripción.
- Un encuestador puede pertenecer a más de un grupo.
- Al asignar encuestadores a un trabajo de campo, se puede asignar directamente un grupo completo, en lugar de hacerlo individualmente.
- Los grupos son independientes de los trabajos de campo: se crean una vez y se reutilizan en múltiples trabajos.

---

## 6. Autenticación y Seguridad

### 6.1 Métodos de Autenticación

- **Teléfono / Email / Username + contraseña:** el usuario puede identificarse con cualquiera de los tres identificadores.
- **Google OAuth:** inicio de sesión mediante botón de Google.

### 6.1.1 Flujo de Login en Steps

El proceso de inicio de sesión se divide en **dos pasos** en todas las aplicaciones:

- **Paso 1 — Identificación:** El usuario ingresa su teléfono, correo electrónico o username. El sistema valida la existencia del usuario y carga su información de perfil (nombre e imagen).
- **Paso 2 — Contraseña:** Se muestra la imagen de perfil y el nombre del usuario reconocido. El usuario ingresa su contraseña.

**Persistencia del último usuario:** Cuando una sesión se cierra (por el usuario o por expiración automática), la PWA mantiene en almacenamiento local los datos del último usuario que inició sesión (nombre e imagen de perfil). Al volver a abrir la aplicación, el login arranca **directamente en el Paso 2**, mostrando la imagen de perfil y el nombre del usuario, sin necesidad de volver a ingresar el identificador. El usuario puede optar por cambiar de usuario, lo que lo lleva de vuelta al Paso 1.

### 6.2 Gestión de Sesiones — JWT

Se utiliza un esquema de **doble token**:

| Token          | Duración         | Descripción |
|----------------|------------------|-------------|
| Access Token   | 15–60 minutos    | Token de corta duración para acceder a los endpoints protegidos. |
| Refresh Token  | 7–30 días        | Token de larga duración almacenado de forma segura (httpOnly cookie). Permite renovar el access token sin volver a pedir credenciales. |

### 6.3 Recuperación de Contraseña

Mediante **enlace de recuperación enviado por email** al correo registrado del usuario. El enlace tiene expiración de tiempo (por definir en la implementación, se recomienda 1 hora).

### 6.4 Seguridad Adicional

- **2FA:** No se implementa en la primera versión.
- **Comunicaciones:** Toda la comunicación entre cliente y servidor debe realizarse sobre **HTTPS**.
- **Contraseñas:** Almacenadas con hashing robusto (bcrypt o argon2).
- **Rate limiting:** La API deberá implementar límites de solicitudes para prevenir ataques de fuerza bruta, especialmente en los endpoints de autenticación.
- **Audit log:** Se registran automáticamente todas las acciones críticas (ver sección [13.2](#132-registro-de-auditoría)).

---

## 7. Trabajos de Campo

### 7.1 Definición

Un trabajo de campo es una unidad organizativa que agrupa una o más encuestas a realizarse en un conjunto de puntos de muestreo durante un período de tiempo determinado.

### 7.2 Ciclo de Vida

```
Borrador → Publicado → Activo → Pausado → Finalizado → Archivado
```

| Estado       | Descripción |
|--------------|-------------|
| **Borrador** | En configuración. Se pueden editar todos los parámetros. Los encuestadores no tienen acceso aún. |
| **Publicado** | Configuración bloqueada. Los encuestadores asignados pueden ver el trabajo pero no comenzar a tomar datos. |
| **Activo**   | Toma de datos habilitada. Se generan actualizaciones en tiempo real en el dashboard. |
| **Pausado**  | Toma de datos temporalmente suspendida. Las encuestas en curso pueden completarse. |
| **Finalizado** | No se pueden tomar más encuestas. Los datos quedan disponibles para análisis. |
| **Archivado** | Trabajo removido de la vista principal, pero conservado en el sistema. |

### 7.3 Configuración de un Trabajo de Campo

Un trabajo de campo incluye:

- **Nombre y descripción.**
- **Período:** fecha y hora de inicio y finalización.
- **Grupos de puntos de muestreo** asignados.
- **Encuestas asignadas**, con sus objetivos de muestra.
- **Asignación de encuestadores a puntos de muestreo** (ver sección [7.5](#75-asignación-de-encuestadores-a-puntos-de-muestreo)).
- **Variables demográficas adicionales** (además de edad y género, que son fijas).
- **Cuotas de muestra** por punto de muestreo (ver sección [7.4](#74-cuotas-de-muestra)).

### 7.4 Cuotas de Muestra

Las cuotas se definen **por punto de muestreo** y combinan:

- **Cantidad total de respuestas objetivo.**
- **Cuotas demográficas** (ej.: 50% mujeres, 30% entre 18–35 años, etc.).

Cuando una cuota está llena, el sistema **advierte al encuestador** antes de iniciar la encuesta, pero **permite continuar** si así lo decide.

### 7.5 Asignación de Encuestadores a Puntos de Muestreo

La asignación de encuestadores a los puntos de muestreo dentro de un trabajo de campo es flexible y admite dos modalidades:

#### 7.5.1 Modalidad Asignada

Un encuestador (o grupo de encuestadores) es asignado explícitamente a uno o más puntos de muestreo específicos. Las reglas de esta modalidad son:

- **Un encuestador puede tener múltiples puntos de muestreo asignados** dentro del mismo trabajo de campo. En la PWA, podrá visualizarlos todos en un mapa y elegir desde cuál trabajar.
- **Un punto de muestreo puede tener múltiples encuestadores asignados**, trabajando de forma simultánea o en distintos turnos/horarios.
- La asignación puede realizarse encuestador por encuestador, o de forma masiva asignando un **grupo de encuestadores** a uno o varios puntos.
- Opcionalmente, se puede indicar un horario de atención sugerido para cada encuestador en cada punto.

#### 7.5.2 Modalidad Abierta

Un grupo de puntos de muestreo (o puntos individuales) puede marcarse como **abierto**, indicando que cualquier encuestador del trabajo de campo puede tomar muestras allí, sin necesidad de asignación nominal previa.

- En la PWA, los puntos abiertos con cupos disponibles aparecen listados y en el mapa para todos los encuestadores activos del trabajo de campo.
- El sistema gestiona los cupos del punto en tiempo real: cuando el total o una cuota demográfica se completa, el punto deja de ofrecerse como disponible (mostrando advertencia si ya está lleno).
- Ambas modalidades pueden coexistir dentro de un mismo trabajo de campo.

### 7.6 Multiasignación de Encuestadores

Un encuestador puede estar asignado y trabajar en **múltiples trabajos de campo activos simultáneamente**.

---

## 8. Puntos de Muestreo

### 8.1 Definición

Los puntos de muestreo son ubicaciones físicas donde se realizan las encuestas. Son entidades reutilizables: un mismo punto puede usarse en múltiples trabajos de campo.

### 8.2 Información de un Punto de Muestreo

| Campo                | Obligatorio | Descripción |
|----------------------|-------------|-------------|
| Nombre               | Sí          | Identificador descriptivo del punto. |
| Dirección            | Sí          | Dirección textual. |
| Coordenadas GPS      | Sí          | Latitud y longitud. Seleccionables en mapa. |
| Horarios de operación | No         | Horarios sugeridos para realizar encuestas. |
| Notas                | No          | Información adicional relevante. |
| Grupo(s)             | Sí          | Al menos un grupo asignado. |
| Estado               | Sí          | Activo, archivado, eliminado (soft delete). |

### 8.3 Grupos de Puntos de Muestreo

Los puntos de muestreo se organizan en **grupos** para facilitar su asignación a trabajos de campo. Un punto puede pertenecer a más de un grupo.

Los grupos son entidades gestionables (crear, editar, eliminar) con nombre y descripción.

### 8.4 Gestión con Google Maps

La interfaz de gestión de puntos de muestreo utiliza la **Google Maps JavaScript API** para:

- Visualizar todos los puntos en un mapa interactivo.
- Seleccionar coordenadas haciendo clic en el mapa al crear/editar un punto.
- Ver el estado (activo/con cuota completada/etc.) de cada punto mediante marcadores con colores.

---

## 9. Encuestas

### 9.1 Ciclo de Vida

```
Borrador → Publicada → Activa → Pausada → Finalizada → Archivada
```

Mismo comportamiento que el ciclo de vida de los trabajos de campo. Una encuesta **publicada genera automáticamente una nueva versión**, preservando la versión anterior para mantener la integridad de los datos históricos.

### 9.2 Tipos de Preguntas

| Tipo                        | Descripción |
|-----------------------------|-------------|
| **Opción múltiple**         | El encuestado puede seleccionar varias respuestas de una lista. |
| **Opción única**            | El encuestado selecciona una sola respuesta de una lista. |
| **Respuesta abierta**       | Campo de texto libre. |
| **Escala Likert**           | Escala de actitud (configurable: 1–5 o 1–10 con etiquetas extremas). |
| **Escala numérica (slider)** | El encuestado elige un valor dentro de un rango numérico. |
| **Ranking**                 | El encuestado ordena opciones por preferencia. |
| **Matriz**                  | Conjunto de sub-preguntas que comparten las mismas opciones de respuesta. |
| **Con imagen en opciones**  | Opción única o múltiple donde cada opción tiene una imagen asociada (ej.: fotos de candidatos, logos de partidos). |

**Imágenes en preguntas:** Cualquier tipo de pregunta puede incluir una imagen en su enunciado, ya sea de forma ilustrativa o como elemento central de la pregunta (por ejemplo: mostrar una imagen y preguntar sobre ella). El enunciado textual puede ser complementario o incluso omitirse si la imagen es suficientemente descriptiva. Las imágenes de enunciado y de opciones son independientes y ambas se almacenan en MinIO.

### 9.3 Lógica Condicional

Las encuestas soportan lógica condicional de dos tipos:

- **Skip logic (salto):** Redirigir al encuestado a una pregunta o sección específica según su respuesta.
- **Display logic (visualización):** Mostrar u ocultar preguntas según condiciones definidas.

Las condiciones pueden combinar múltiples criterios con operadores lógicos **AND / OR**.

### 9.4 Secciones

Las preguntas pueden organizarse en **secciones** para estructurar visualmente la encuesta y facilitar la navegación y la lógica condicional.

### 9.5 Versionado

Cuando una encuesta en estado **Publicada** es modificada, el sistema crea automáticamente una nueva versión. Las respuestas históricas siempre quedan vinculadas a la versión de la encuesta con la que fueron tomadas.

### 9.6 Duplicación y Plantillas

- Las encuestas pueden **duplicarse** para crear una nueva encuesta con las mismas preguntas como punto de partida.
- Las encuestas pueden marcarse como **plantilla reutilizable**, apareciendo en una galería de plantillas para crear nuevas encuestas a partir de ellas.

### 9.7 Metadatos y Etiquetas

Cada encuesta tiene los siguientes metadatos:

- Nombre y descripción.
- Fecha de creación, publicación, inicio y finalización.
- Estado.
- Versión actual.
- **Etiquetas/categorías** (asignadas por administradores para clasificación y búsqueda).
- Trabajo(s) de campo al que está asignada.

Las **preguntas individuales** también pueden tener etiquetas propias, facilitando el análisis por categoría (ej.: filtrar todas las preguntas de intención de voto en múltiples encuestas).

---

## 10. Toma de Muestras — PWA Encuestadores

### 10.1 Capacidades Offline

#### 10.1.1 PWA de Encuestadores (offline completo)

La PWA de encuestadores está diseñada para funcionar **sin conexión a internet**:

- Los trabajos de campo, encuestas y configuración se sincronizan al dispositivo cuando hay conexión.
- Las respuestas y datos recopilados se almacenan localmente en **IndexedDB** mediante **Service Workers**.
- Al restablecer la conexión, los datos se sincronizan automáticamente con el servidor.
- La app es **instalable** como PWA en el dispositivo del encuestador (Android/iOS/escritorio).

#### 10.1.2 PWA de Administración (offline limitado)

La PWA de administración también es **instalable** y cuenta con soporte offline básico mediante Service Worker + IndexedDB:

- En modo offline, los usuarios pueden **consultar datos previamente cargados** (trabajos de campo, encuestas, resultados cacheados, informes guardados).
- Las acciones que requieren escritura en el servidor (crear, editar, publicar, etc.) **no están disponibles** sin conexión y se muestra un mensaje claro al usuario.
- El dashboard muestra los últimos datos sincronizados con indicación del momento de la última actualización.
- Las funcionalidades de tiempo real (tracking GPS, encuestas en proceso) quedan suspendidas sin conexión.

### 10.2 Datos Recopilados Automáticamente

Al completar (o al iniciar/finalizar) cada encuesta, el sistema registra automáticamente:

| Dato                     | Descripción |
|--------------------------|-------------|
| Fecha y hora de inicio   | Timestamp del momento en que el encuestador presionó "Iniciar Encuesta". |
| Fecha y hora de finalización | Timestamp del momento en que se completó la encuesta. |
| Ubicación GPS (al inicio) | Coordenadas del encuestador al iniciar, si tiene permiso y GPS disponible. |
| Ubicación GPS (al finalizar) | Coordenadas del encuestador al finalizar. |
| Estado del GPS           | `disponible` / `no disponible` / `sin permiso`. |
| Encuestador              | ID del usuario que tomó la muestra. |
| Punto de muestreo        | Punto desde el cual se tomó la muestra (asignado o abierto). |
| Versión de la encuesta   | Versión exacta de la encuesta utilizada. |

Si el GPS no está disponible o el usuario no otorgó permisos, la encuesta **puede continuar** pero la ubicación queda marcada como `no disponible`.

### 10.3 Datos Demográficos del Encuestado

Al iniciar cada encuesta, el encuestador registra datos básicos del encuestado:

**Fijos (siempre requeridos):**
- Edad (número entero o rango etario, según configuración).
- Género.

**Adicionales configurables por trabajo de campo** (ejemplos: nivel educativo, ocupación, zona de residencia, etc.).

### 10.4 Cuotas Demográficas

Antes de iniciar una encuesta, la app muestra al encuestador el estado de las cuotas del punto de muestreo. Las cuotas se muestran con:

- **Cantidad efectiva y objetivo** (ej.: `5 / 10`).
- **Porcentaje de cumplimiento** (ej.: `50%`).
- **Indicador de color:** verde (dentro de cuota), amarillo (≥ 80% de la cuota), rojo (cuota completada o superada).

Si una cuota ya está completa para el perfil demográfico del encuestado, se **muestra una advertencia visible** pero se **permite continuar** la encuesta.

### 10.5 Pausa y Reanudación

Las encuestas en curso pueden **pausarse y retomarse posteriormente**, incluso offline. El estado de la encuesta pausada queda guardado localmente.

### 10.6 Incidencias

Los encuestadores pueden reportar incidencias desde la app mediante:

- **Tipo de incidencia:** selección de un formulario predefinido de tipos (configurable por administradores, ej.: "Punto de muestreo cerrado", "Encuestado rechazó participar", "Problema técnico", etc.).
- **Descripción libre:** campo de texto adicional opcional.

Las incidencias quedan asociadas al trabajo de campo y al punto de muestreo correspondiente y son visibles para supervisores y administradores en tiempo real.

### 10.7 Tracking GPS en Tiempo Real

Cuando el encuestador tiene GPS habilitado y está con una sesión activa, su **ubicación se actualiza periódicamente** en el servidor y es visible para supervisores y administradores en el mapa de monitoreo en tiempo real. Este tracking **solo funciona con conexión a Internet activa**.

### 10.8 Seguimiento de Encuestas en Proceso

Cuando un encuestador inicia una encuesta (presiona "Iniciar Encuesta"), se emite un evento en tiempo real al servidor que es visible para supervisores y administradores. El sistema permite ver en vivo:

- Cuántas encuestas están siendo realizadas en este momento en cada trabajo de campo.
- Qué encuestador está tomando cada encuesta activa (nombre, punto de muestreo, encuesta en curso).

Al finalizar la encuesta, el evento se actualiza reflejando la finalización. Este seguimiento **solo funciona con conexión a Internet activa**; si el encuestador está offline, la encuesta se registra localmente y los eventos de inicio/finalización se sincronizan al reconectarse.

### 10.8 Estadísticas del Encuestador

Los encuestadores **no visualizan estadísticas de desempeño** propias en la PWA. Solo ven los trabajos asignados, las encuestas pendientes y el progreso de cuotas del punto donde trabajan.

---

## 11. Resultados y Análisis

### 11.1 Módulo de Análisis

El módulo de análisis de resultados está diseñado para analistas y administradores. Permite trabajar con datos de una o múltiples encuestas y trabajos de campo combinados.

### 11.2 Tipos de Gráficos Disponibles

| Categoría         | Tipos |
|-------------------|-------|
| Básicos           | Barras, Torta / Dona, Línea, Área |
| Estadísticos      | Mapa de calor, Radar, Burbuja, Treemap |
| Geográficos       | Mapa coroplético (resultados por zona geográfica) |
| Tabulares         | Tablas de frecuencia y porcentajes |

### 11.3 Análisis Avanzado

- **Tablas cruzadas (crosstabulation):** Cruce de cualquier par de preguntas de encuesta o variables demográficas. Por ejemplo: intención de voto cruzada por género, franja etaria o zona geográfica.
- **Filtros combinados:** Filtrar resultados según respuestas a preguntas específicas, rango de fechas, puntos de muestreo, encuestadores, etc.
- **Comparaciones:** Comparar resultados entre diferentes encuestas, versiones de encuesta o trabajos de campo.
- **Evolución histórica:** Visualizar cómo cambian los resultados a lo largo del tiempo (gráficos de línea por fecha de recolección).
- **Segmentación:** Por demografía (edad, género, nivel educativo, etc.), geografía (punto de muestreo, zona) o cualquier otra variable relevante.

### 11.4 Ponderación de Datos

El sistema ofrece dos métodos de ponderación:

| Método                    | Descripción |
|---------------------------|-------------|
| **Ponderación manual**    | El analista asigna manualmente pesos a grupos demográficos o geográficos. |
| **Rim Weighting / Raking** | Algoritmo iterativo que ajusta automáticamente los pesos para que múltiples variables demográficas simultáneas coincidan con targets de población definidos. |

Los analistas pueden aplicar una ponderación a un conjunto de resultados y guardarla como **escenario nombrado** para reutilizarla o compararla.

#### 11.4.1 Ponderación por Punto de Muestreo

Adicionalmente, el sistema permite asignar **pesos específicos a los puntos de muestreo**, permitiendo simular distintos niveles de participación o relevancia geográfica. Esto es especialmente útil en encuestas de intención de voto, donde cada zona puede tener una **"asistencia electoral esperada"** diferente según datos históricos.

- El analista puede definir un escenario de ponderación geográfica asignando un coeficiente de peso a cada punto de muestreo (o grupo de puntos).
- Los resultados agregados reflejarán la ponderación aplicada, ajustando la contribución de cada zona al resultado final.
- Estos escenarios geográficos se pueden combinar con la ponderación demográfica (manual o rim weighting) para obtener proyecciones más precisas.
- Los escenarios pueden guardarse, nombrarse y compararse entre sí.

### 11.5 Exportación de Resultados

Los resultados pueden exportarse en:

- **CSV** (datos crudos, para uso en herramientas externas como Excel, R, SPSS, etc.)
- **Excel (.xlsx)** (con formato, múltiples hojas por pregunta o segmento)
- **PDF** (informe de resultados con gráficos, generado en el servidor con Puppeteer)

---

## 12. Generación de Informes

### 12.1 Editor de Informes

Los analistas y administradores pueden crear informes utilizando un **editor de bloques tipo Notion**, donde cada bloque puede ser:

| Tipo de bloque       | Descripción |
|----------------------|-------------|
| Párrafo de texto     | Texto enriquecido (negrita, cursiva, listas, etc.) |
| Título / Subtítulo   | Encabezados jerárquicos |
| Gráfico del sistema  | Gráfico generado desde los datos de las encuestas, incrustado y configurable (tipo, segmentación, filtros) |
| Tabla                | Tabla de datos o de resultados |
| Imagen               | Imagen personalizada cargada por el usuario |
| Separador            | Línea divisoria |
| Cita / Destacado     | Bloque de texto destacado visualmente |

Los gráficos incluidos en los informes son **configurables directamente desde el editor**: el analista puede elegir el tipo de gráfico, la pregunta fuente, los filtros y la ponderación aplicada.

### 12.2 Origen de Datos

Un informe puede combinar datos de **múltiples encuestas y trabajos de campo**.

### 12.3 Visibilidad

| Tipo       | Descripción |
|------------|-------------|
| **Privado** | Solo accesible para usuarios del sistema con permisos específicos, o mediante enlace seguro (con token). |
| **Público** | Publicado en el sitio web de publicación de resultados, accesible por cualquier persona sin autenticación. |

### 12.4 Exportación y Compartición

- **PDF:** Generado en el servidor (Puppeteer) para máxima fidelidad visual. Vista previa en cliente con resolución rápida.
- **Word (.docx):** Exportación editable.
- **Enlace web público:** URL pública del informe en el sitio de publicación (solo para informes públicos).
- **Enlace privado con token:** URL segura con token de acceso de tiempo limitado para compartir informes privados con personas externas al sistema.

---

## 13. Notificaciones y Auditoría

### 13.1 Sistema de Notificaciones

**Canales:**
- **Push notifications (navegador):** Requieren que el usuario haya otorgado permisos al navegador.
- **Notificaciones in-app:** Centro de notificaciones dentro de la PWA.

**Eventos que generan notificaciones** (lista no exhaustiva, configurable):

| Evento                                        | Destinatario |
|-----------------------------------------------|--------------|
| Nuevo trabajo de campo asignado               | Encuestador  |
| Encuesta iniciada (en proceso)                | Supervisor, Administrador |
| Encuesta completada (hito de progreso)        | Supervisor, Administrador |
| Cuota de muestra completada en un punto       | Supervisor, Administrador |
| Incidencia reportada por un encuestador       | Supervisor, Administrador |
| Trabajo de campo próximo a finalizar          | Supervisor, Administrador |
| Usuario creado / contraseña restablecida      | Usuario afectado |
| Informe publicado                             | Roles configurables |

Los usuarios pueden configurar qué notificaciones desean recibir desde su perfil.

### 13.2 Registro de Auditoría

Todas las **acciones críticas** quedan registradas automáticamente en un log de auditoría. Cada entrada incluye:

- Usuario que realizó la acción.
- Fecha y hora (timestamp UTC).
- Tipo de acción (ej.: `USUARIO_CREADO`, `ENCUESTA_PUBLICADA`, `RESULTADOS_EXPORTADOS`).
- Descripción detallada (entidad afectada, valores anteriores y nuevos).
- IP de origen.

El log es visible por administradores y súper administradores desde la pantalla de Registro de Auditoría. No es editable.

---

## 14. Sitio Web de Publicación

> **Prioridad: BAJA — No se implementa en la primera versión.**

### 14.1 Descripción General

Sitio web de acceso público construido como una **aplicación React + Vite separada** dentro del monorepo. Su propósito es mostrar informes y resultados de encuestas marcados como públicos por los administradores.

### 14.2 Funcionalidades Previstas

- Listado de informes públicos con filtros básicos (fecha, etiquetas, trabajo de campo).
- Vista de detalle de cada informe con sus gráficos y textos.
- Sin autenticación requerida para el acceso a contenido público.
- Diseño profesional y compatible con todos los dispositivos.

---

## 15. PWA de Encuestadores — UX/UI

### 15.1 Principios de Diseño

- Interfaz **extremadamente simple e intuitiva**, diseñada para uso en campo con dispositivos móviles.
- Optimizada para pantallas pequeñas (teléfonos).
- Mínimas distracciones: solo muestra lo necesario para la tarea actual.
- Funcional con **una sola mano** en la medida de lo posible.
- Tiempos de respuesta rápidos incluso con conexiones lentas.

### 15.2 Pantallas

#### 15.2.1 Pantalla de Login

El login se implementa en **dos pasos**:

- **Paso 1 — Identificación:** Campo único para ingresar teléfono, email o username. Botón "Continuar". Botón "Iniciar sesión con Google".
- **Paso 2 — Contraseña:** Se muestra la imagen de perfil y el nombre del usuario identificado. Campo de contraseña. Enlace a recuperación de contraseña. Opción "No soy yo" para volver al Paso 1.

**Persistencia:** Si el usuario cierra sesión o la sesión expira, la app recuerda al último usuario. Al volver a abrir la app, el login arranca directamente en el **Paso 2** mostrando imagen de perfil y nombre, evitando reingresar el identificador.

#### 15.2.2 Pantalla de Inicio
- Listado de **trabajos de campo asignados** con:
  - Nombre del trabajo.
  - Estado (Activo, Pausado, Pendiente).
  - Progreso (encuestas completadas / total objetivo en los puntos del encuestador).
- Acceso rápido al centro de notificaciones.
- Acceso al perfil de usuario.
- **Navegación automática:** Si el encuestador tiene un único trabajo de campo activo, la app navega automáticamente a la pantalla de ese trabajo, omitiendo la selección manual. Se muestra una breve confirmación visual con opción de "Ver todos los trabajos".

#### 15.2.3 Pantalla de Selección de Trabajo de Campo
- Detalle del trabajo seleccionado:
  - Fechas de inicio y fin.
  - **Puntos de muestreo** del encuestador en este trabajo (puede ser uno o varios):
    - Si el encuestador tiene múltiples puntos asignados o hay puntos abiertos disponibles, se muestran en un **mapa interactivo** (Google Maps) y como lista, con el estado de cuotas de cada uno.
    - Cada punto muestra su progreso de cuotas con cantidades y porcentajes.
  - Encuestas disponibles en este trabajo.
- Botón para reportar una incidencia.

#### 15.2.4 Pantalla de Selección de Encuesta
- Lista de encuestas disponibles para el trabajo seleccionado.
- Indicador del objetivo de muestra y el progreso actual.
- Indicador de cuotas demográficas con:
  - Cantidad efectiva y objetivo (ej.: `5 / 10`).
  - Porcentaje de cumplimiento (ej.: `50%`).
  - Estado visual por color (verde / amarillo / rojo).
- **Navegación automática:** Si solo hay una encuesta disponible en el trabajo, la app navega directamente a ella, omitiendo la selección manual. Se muestra confirmación visual con nombre de la encuesta.

#### 15.2.5 Pantalla de Registro Demográfico
- Formulario para registrar los datos del encuestado (edad, género + variables adicionales configuradas).
- Si alguna cuota ya está completada para este perfil demográfico, se muestra una **advertencia visible** con la opción de continuar o cancelar.

#### 15.2.6 Pantalla de Toma de Encuesta
- Visualización clara y limpia de cada pregunta con sus opciones.
- **Imagen en el enunciado:** si la pregunta tiene una imagen asociada al enunciado, se muestra de forma prominente antes de las opciones. El enunciado textual puede ser complementario.
- **Imagen en las opciones:** para preguntas con imágenes en opciones, las imágenes se muestran junto a (o en lugar de) el texto de cada opción.
- Barra de progreso con el número de pregunta actual y el total.
- Navegación hacia atrás (para corregir respuestas anteriores).
- Lógica condicional aplicada automáticamente (saltos y visualización según respuestas).
- Botón **"Finalizar Encuesta"** visible solo en la última pregunta.
- El inicio se marca con **"Iniciar Encuesta"** (botón en la pantalla de registro demográfico o en una pantalla de confirmación previa a las preguntas).
- Las fechas/horas y GPS se registran automáticamente al iniciar y finalizar.

#### 15.2.7 Pantalla de Confirmación de Encuesta Completada
- Mensaje de éxito.
- Resumen: tiempo empleado, punto de muestreo.
- Indicador de estado de sincronización (sincronizado / pendiente de sync offline).
- Opciones: "Tomar otra encuesta" o "Volver al inicio".

#### 15.2.8 Pantalla de Reporte de Incidencia
- Selector de tipo de incidencia (lista configurable).
- Campo de descripción libre (opcional).
- Confirmación y envío (funciona también offline).

#### 15.2.9 Pantalla de Perfil de Usuario
- Edición de nombre, teléfono e imagen de perfil.
- Cambio de contraseña.
- Configuración de notificaciones.
- Cierre de sesión.

---

## 16. PWA de Administración — UX/UI

### 16.1 Principios de Diseño

- Interfaz rica y funcional, pero intuitiva.
- Responsive: funciona en dispositivos móviles, tablets y computadoras de escritorio.
- Sidebar de navegación colapsable.
- Accesos rápidos (shortcuts) a las funciones más utilizadas.
- Dashboard con visión general del estado del sistema.

### 16.2 Pantallas

#### 16.2.1 Pantalla de Login

El login se implementa en **dos pasos**, idéntico al de la PWA de encuestadores:

- **Paso 1 — Identificación:** Campo único para teléfono, email o username. Botón "Continuar". Botón "Iniciar sesión con Google".
- **Paso 2 — Contraseña:** Se muestra imagen de perfil y nombre del usuario reconocido. Campo de contraseña. Enlace a recuperación de contraseña. Opción "No soy yo".

**Persistencia:** Al volver a la app tras cierre de sesión o expiración, el login arranca en el **Paso 2** con los datos del último usuario, evitando reingresar el identificador.

#### 16.2.2 Dashboard (Pantalla de Inicio)
- **Resumen estadístico:** trabajos de campo activos, encuestas completadas hoy, encuestadores activos en campo.
- **Encuestas en proceso en tiempo real:** contador y listado de las encuestas que se están tomando en este momento (encuestador, punto de muestreo, encuesta en curso). Requiere conexión activa.
- **Gráficos rápidos:** progreso de los trabajos de campo activos.
- **Mapa thumbnail:** vista reducida de los puntos de muestreo activos con encuestadores.
- **Notificaciones recientes** e incidencias sin resolver.
- **Accesos rápidos** a las secciones más usadas.

#### 16.2.3 Gestión de Usuarios
> *Acceso: Administrador, Súper Administrador*

- Tabla con todos los usuarios (con filtros y búsqueda por nombre, username, email, rol, estado).
- Crear / editar / archivar / restaurar usuarios.
  - Al crear: opción de asignar contraseña manual o enviar enlace de activación por email.
  - Campos: nombre completo, username, email, teléfono, rol(es), imagen de perfil.
- Asignación de roles.
- Gestión de **Grupos de Encuestadores**: CRUD de grupos, asignación de encuestadores a grupos, vista de encuestadores por grupo.
- Vista de historial de actividad del usuario.

#### 16.2.4 Gestión de Trabajos de Campo
> *Acceso: Administrador, Supervisor (lectura + gestión de asignaciones)*

- Tabla de trabajos de campo con filtros por estado, fecha, etc.
- Crear / editar / archivar trabajos de campo.
- Asignar grupos de puntos de muestreo.
- **Gestión de asignación de encuestadores a puntos:**
  - Asignación nominal: asignar encuestadores (individualmente o por grupo) a puntos específicos, con horario opcional. Relación muchos a muchos (múltiples encuestadores por punto, múltiples puntos por encuestador).
  - Asignación abierta: marcar uno o varios puntos como "abiertos" para que cualquier encuestador activo en el trabajo pueda tomarlos.
  - Ambas modalidades pueden coexistir en el mismo trabajo.
- Configurar encuestas y objetivos de muestra (totales + cuotas demográficas por punto).
- Seguimiento del progreso en tiempo real (barras de progreso, alertas de cuotas, encuestas en proceso).

#### 16.2.5 Gestión de Encuestas
> *Acceso: Administrador*

- Tabla de encuestas con filtros por estado, etiquetas, trabajo de campo.
- Crear / editar / duplicar / archivar encuestas.
- **Constructor de encuestas:**
  - Añadir, editar, reordenar y eliminar secciones y preguntas.
  - Configurar todos los tipos de pregunta.
  - Definir lógica condicional (skip logic + display logic) mediante interfaz visual.
  - Subir imágenes para preguntas y opciones.
  - Asignar etiquetas a preguntas.
- Publicar encuesta (genera nueva versión).
- Galería de plantillas.

#### 16.2.6 Gestión de Puntos de Muestreo
> *Acceso: Administrador*

- **Mapa interactivo (Google Maps):** vista de todos los puntos con marcadores.
- Tabla/lista de puntos con filtros por grupo, estado, etc.
- Crear / editar / archivar puntos de muestreo (con selección de coordenadas en el mapa).
- Gestión de grupos de puntos.

#### 16.2.7 Análisis de Resultados
> *Acceso: Administrador, Analista de Datos*

- Selector de encuesta(s) y/o trabajo(s) de campo a analizar.
- Panel de filtros: rango de fechas, puntos de muestreo, encuestadores, segmentos demográficos, respuestas específicas.
- **Visualizaciones interactivas:** todos los tipos de gráficos disponibles (sección [11.2](#112-tipos-de-gráficos-disponibles)).
- **Constructor de tablas cruzadas:** selección de variable fila y variable columna.
- **Panel de ponderación:** aplicar ponderación manual, rim weighting y/o ponderación geográfica por punto de muestreo. Guardar y comparar escenarios nombrados.
- **Evolución histórica:** gráfico de evolución de resultados por fecha.
- Exportación de resultados (CSV, Excel, PDF).
- Botón "Crear informe a partir de este análisis" (pre-carga gráficos actuales en el editor de informes).

#### 16.2.8 Generación de Informes
> *Acceso: Administrador, Analista de Datos*

- Lista de informes (propios y con acceso) con estado (borrador / publicado / privado / público).
- Crear nuevo informe.
- **Editor de bloques tipo Notion:** composición libre de texto, gráficos, imágenes y tablas.
- Configuración de visibilidad (privado / público).
- Exportar (PDF, Word, enlace).
- Vista previa antes de publicar.

#### 16.2.9 Mapa de Monitoreo en Tiempo Real
> *Acceso: Administrador, Supervisor*

- **Mapa de Google Maps** con:
  - Marcadores de puntos de muestreo con estado de cuotas (coloreados), mostrando cantidades y porcentajes.
  - Íconos de encuestadores activos con su posición GPS en tiempo real (cuando disponible y conectados).
  - Popup al hacer clic en un encuestador: nombre, trabajo actual, encuestas completadas hoy, encuesta en curso (si la hay), última actividad.
  - Popup al hacer clic en un punto: cuotas completadas con cantidades/porcentajes, encuestadores asignados, encuestas en proceso ahora, incidencias.
- **Panel lateral de encuestas en proceso:** listado en tiempo real de todas las encuestas que se están tomando en este momento en los trabajos activos (encuestador, punto, encuesta, tiempo transcurrido).
- Filtros: por trabajo de campo, por zona, por estado.
- Requiere conexión activa para datos en tiempo real. Sin conexión muestra los últimos datos cacheados con indicación de la última actualización.

#### 16.2.10 Gestión de Etiquetas y Categorías
> *Acceso: Administrador*

- CRUD de etiquetas para encuestas y preguntas.
- Agrupación de etiquetas por categoría.
- Vista de uso de cada etiqueta (cuántas encuestas/preguntas la tienen asignada).

#### 16.2.11 Gestión de Plantillas de Encuestas
> *Acceso: Administrador*

- Galería visual de plantillas disponibles.
- Crear nueva encuesta desde plantilla.
- Promover una encuesta existente a plantilla.
- Archivar / eliminar plantillas.

#### 16.2.12 Configuración del Sistema
> *Acceso: Administrador, Súper Administrador*

- **Variables demográficas:** gestionar las variables adicionales configurables para trabajos de campo.
- **Tipos de incidencias:** CRUD de tipos de incidencias predefinidas para encuestadores.
- **Configuración de notificaciones:** reglas globales de notificaciones.
- **Gestión de Google Maps:** configuración de la API key.
- **Configuración de ponderación:** targets de población para rim weighting (ej.: distribución por edad y género de la población de referencia).
- **Ajustes de seguridad:** políticas de contraseñas, expiración de tokens, etc.

#### 16.2.13 Registro de Auditoría
> *Acceso: Administrador, Súper Administrador*

- Tabla completa del log de acciones críticas.
- Filtros: por usuario, por tipo de acción, por fecha, por entidad afectada.
- Vista de detalle de cada entrada del log.
- No editable.

#### 16.2.14 Centro de Notificaciones
> *Acceso: Todos los roles*

- Bandeja de notificaciones in-app.
- Marcar como leída / no leída.
- Filtros por tipo.
- Enlace directo a la entidad relacionada con cada notificación.
- Configuración personal de qué notificaciones recibir.

#### 16.2.15 Perfil de Usuario
> *Acceso: Todos los roles*

- Edición de datos personales (nombre, teléfono, imagen).
- Cambio de contraseña.
- Configuración de notificaciones.
- Cierre de sesión.

---

## 17. API RESTful

### 17.1 Principios

- **RESTful:** recursos bien definidos, uso correcto de verbos HTTP (GET, POST, PUT, PATCH, DELETE).
- **Versionada:** todos los endpoints bajo `/api/v1/`.
- **Documentación:** Swagger/OpenAPI autogenerado y accesible en `/api/docs`.
- **Autenticación:** Bearer token (JWT access token) en todos los endpoints protegidos.
- **Respuestas consistentes:** estructura estándar de respuesta `{ data, meta, error }`.
- **Paginación:** basada en cursor o en offset/limit, consistente en todos los listados.

### 17.2 Módulos Principales

| Módulo                  | Descripción |
|-------------------------|-------------|
| `/auth`                 | Login (steps), logout, refresh token, Google OAuth, recuperación de contraseña, activación de cuenta |
| `/usuarios`             | CRUD de usuarios, gestión de roles, perfil, username |
| `/grupos-encuestadores` | CRUD de grupos de encuestadores, asignación de miembros |
| `/trabajos-de-campo`    | CRUD, asignaciones (modalidad asignada y abierta), progreso, cuotas |
| `/puntos-de-muestreo`   | CRUD, grupos |
| `/encuestas`            | CRUD, constructor, versionado, plantillas |
| `/preguntas`            | CRUD de preguntas dentro de encuestas |
| `/respuestas`           | Registro de respuestas (desde PWA encuestadores), sync offline |
| `/sesiones-encuesta`    | Inicio/fin de encuestas en proceso (para seguimiento en tiempo real) |
| `/resultados`           | Análisis, agregaciones, exportaciones, ponderación demográfica y geográfica |
| `/informes`             | CRUD, publicación, exportación |
| `/incidencias`          | Registro y consulta de incidencias |
| `/notificaciones`       | Listado, marcado, configuración |
| `/auditoria`            | Consulta del log de auditoría |
| `/configuracion`        | Gestión de variables demográficas, tipos de incidencias, etc. |
| `/archivos`             | Upload/download de imágenes y archivos (via MinIO) |

### 17.3 Seguridad de la API

- **HTTPS obligatorio** en producción.
- **Rate limiting:** límites por IP y por usuario autenticado, más estrictos en endpoints de autenticación.
- **CORS:** configurado para aceptar solo los orígenes de las apps del sistema.
- **Validación de entrada:** todos los payloads son validados con esquemas estrictos (Zod).
- **Autorización por rol:** cada endpoint verifica que el usuario tenga el rol requerido.

---

## 18. Infraestructura y Despliegue

### 18.1 Servicios Docker

```yaml
# docker-compose.yml — servicios principales
servicios:
  - api:          Node.js + Fastify (API RESTful)
  - mysql:        Base de datos MySQL/MariaDB
  - redis:        Caché y broker WebSocket
  - minio:        Almacenamiento de archivos (S3-compatible)
  - nginx:        Reverse proxy y servicio de las PWAs compiladas
```

### 18.2 Despliegue

- Toda la plataforma se despliega en un **VPS propio** mediante Docker Compose.
- Las apps React (PWA encuestadores, PWA administración, sitio público) se compilan a assets estáticos servidos por Nginx.
- La API corre como contenedor Node.js detrás de Nginx como reverse proxy.

### 18.3 Backup

- **Frecuencia:** Backup automático diario de la base de datos MySQL.
- **Retención:** 30 días.
- **Almacenamiento de backups:** ubicación segura separada del servidor principal (a definir en implementación: bucket MinIO separado, almacenamiento externo, etc.).

### 18.4 Variables de Entorno

Todas las configuraciones sensibles (credenciales de base de datos, claves API, secretos JWT, etc.) se gestionan mediante variables de entorno y **nunca se incluyen en el código fuente ni en el repositorio Git**.

---

## 19. Testing

### 19.1 Estrategia General

| Tipo                | Framework   | Alcance |
|---------------------|-------------|---------|
| Unit tests          | Vitest      | Lógica de negocio, utilidades, transformaciones de datos |
| Integration tests   | Vitest      | Endpoints de la API, interacciones con BD y Redis |
| E2E tests           | Playwright  | Flujos completos de usuario en las PWAs |

### 19.2 Cobertura Mínima Objetivo

*(A definir en especificaciones de implementación de cada módulo)*

### 19.3 Datos de Prueba

Se proveerá un script de **seed de base de datos** con datos de prueba realistas para facilitar el desarrollo y los tests de integración.

---

## 20. Filosofía Spec-as-Source y Desarrollo con IA

### 20.1 Principios Core

1. **Las Specs son la única fuente de verdad:** El código es un artefacto generado para cumplir con las especificaciones. Ningún componente del sistema existe sin un documento de spec que lo respalde.

2. **Flujo de trabajo obligatorio:** Todo cambio o nueva funcionalidad DEBE seguir este ciclo:
   ```
   Requerimiento → Actualización/Creación de Spec → Revisión → Generación de Código → Verificación
   ```

3. **Sin Specs no hay Código:** Si se escribe código que no esté documentado en la carpeta `.spec/`, se considera un defecto y debe eliminarse o documentarse inmediatamente.

4. **Git maneja el historial:** No se mantienen changelogs dentro de los archivos Markdown. La especificación siempre refleja el **estado actual y deseado** del sistema. La evolución histórica se gestiona exclusivamente mediante commits de Git.

### 20.2 Estructura de la Carpeta `.spec/`

```
.spec/
├── ESPECIFICACIONES.md          # Documento maestro (este archivo)
├── api/
│   ├── autenticacion.md
│   ├── usuarios.md
│   ├── trabajos-de-campo.md
│   ├── puntos-de-muestreo.md
│   ├── encuestas.md
│   ├── respuestas.md
│   ├── resultados.md
│   ├── informes.md
│   └── ...
├── modelos/
│   ├── esquema-bd.md            # Definición de tablas y relaciones
│   └── tipos-compartidos.md     # Tipos TypeScript compartidos
├── pwa-encuestadores/
│   ├── pantallas.md
│   ├── flujos.md
│   └── offline.md
├── pwa-administracion/
│   ├── pantallas.md
│   ├── dashboard.md
│   ├── analisis.md
│   └── informes.md
└── infraestructura/
    ├── docker.md
    ├── variables-de-entorno.md
    └── backup.md
```

### 20.3 Sub-Agentes de Desarrollo

La organización de sub-agentes especializados de IA para el desarrollo del sistema **se definirá en una especificación separada** antes de comenzar el desarrollo. Se contempla la división por dominios (Frontend, Backend, Base de Datos, Testing, DevOps), pero los detalles de responsabilidades, herramientas y protocolos de comunicación entre agentes se detallarán en `.spec/agentes/` en una fase posterior.

---

## 21. Alcance Geográfico y Escalabilidad

### 21.1 Despliegue Inicial

- **Geografía:** Provincia de Santa Fe, con foco en la ciudad de **Rosario**, Argentina.
- Las divisiones políticas y zonas geográficas del sistema estarán configuradas para Argentina desde el inicio.

### 21.2 Escalabilidad Prevista

- **Geográfica:** El sistema está diseñado para escalar a cobertura nacional (toda Argentina) sin cambios arquitecturales mayores. Las zonas geográficas (provincias, municipios, barrios) serán configurables.
- **De usuarios:** La infraestructura soporta la escala mediana (50–200 encuestadores activos simultáneamente) definida para el inicio, con capacidad de crecimiento mediante escalado vertical del VPS o migración a orquestación de contenedores (Kubernetes) en el futuro.
- **De datos:** El versionado de encuestas, el borrado lógico y el diseño relacional de la base de datos garantizan la integridad a largo plazo a medida que crece el volumen de respuestas históricas.

---

## 22. Consideraciones Futuras

Las siguientes funcionalidades están identificadas como **deseables para versiones futuras** pero **no forman parte del alcance de la primera versión**:

| Funcionalidad                          | Notas |
|----------------------------------------|-------|
| Encuestas telefónicas                  | Integración con plataformas de llamadas, IVR. |
| Encuestas en línea (autoadministradas) | Formulario web accesible por URL pública. |
| Autenticación de dos factores (2FA)    | Para mayor seguridad en cuentas de admin. |
| CI/CD automatizado                     | Pipeline de GitHub Actions para tests y deploy. |
| Aplicación móvil nativa (iOS/Android)  | Para mejor experiencia offline y GPS. |
| Extensión geográfica nacional          | Configuración completa de provincias y municipios argentinos. |
| Internacionalización (i18n)            | Soporte de múltiples idiomas en la interfaz. |
| Integración con herramientas de BI     | Conexión directa con Power BI, Tableau, etc. |
| Sub-agentes de IA definidos            | Organización detallada del flujo de desarrollo con IA. |

---

*Fin del documento de especificaciones v1.1.0*
