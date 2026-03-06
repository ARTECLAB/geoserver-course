# Módulo 2 · Publicación de Datos 🗄️

> **Clases 2 y 3 · 4 horas totales**
> El corazón del curso. Aprendes a llevar cualquier dato geográfico a un servicio publicado: desde shapefiles hasta capas directamente desde PostGIS.

---

## 🎯 Objetivos del Módulo

- Publicar datos vectoriales desde shapefiles de Bolivia
- Conectar GeoServer a una base de datos PostGIS
- Publicar capas directamente desde tablas y vistas de PostGIS
- Configurar metadatos profesionales en cada capa
- Crear grupos de capas para organizar servicios
- Entender cuándo usar Shapefile vs PostGIS

---

## Clase 02 — Publicación de Datos Vectoriales: Shapefile & Directory Store

### 🎯 Objetivo

Dominar la publicación de datos vectoriales desde archivos Shapefile, entender las limitaciones del formato y aprender a organizar múltiples capas eficientemente.

### Parte 1 — Repaso rápido y preparación de datos (10 min)

#### Datos de ejemplo para clase (Bolivia)

Preparar en una carpeta `C:\geodatos\bolivia\` (o `/home/usuario/geodatos/bolivia/`):

| Archivo | Geometría | Atributos clave |
|---------|-----------|-----------------|
| `departamentos.shp` | MultiPolygon | nombre, codigo, poblacion, superficie_km2 |
| `rios_principales.shp` | MultiLineString | nombre, longitud_km, cuenca |
| `capitales.shp` | Point | nombre, departamento, altitud_msnm, poblacion |
| `areas_protegidas.shp` | MultiPolygon | nombre, categoria, superficie_ha, año_creacion |

> **Tip para clase:** Tener los shapefiles descargados y verificados ANTES de la clase. Abrirlos en QGIS para confirmar que se ven bien y que los atributos no tienen caracteres rotos.

### Parte 2 — Shapefile Store: Un archivo, una capa (15 min)

Ya hicimos esto en la Clase 01 con un solo shapefile. Ahora profundizamos en la configuración.

#### Configuración detallada de la capa

Al publicar una capa, hay campos que DEBES configurar bien:

**Pestaña Data:**

```
Name:     departamentos
          → Nombre técnico: sin espacios, sin tildes, en minúsculas
          → Este nombre aparece en las URLs de los servicios

Title:    Departamentos de Bolivia
          → Nombre legible para humanos
          → Es lo que ven los usuarios en GetCapabilities

Abstract: Límites administrativos de los 9 departamentos de Bolivia.
          Fuente: Instituto Nacional de Estadística (INE), 2024.
          Sistema de referencia original: EPSG:32719 (UTM 19S).
          → Descripción técnica del dato. Sé específico.

Keywords: departamentos, límites, administrativo, bolivia, INE
          → Ayudan a buscar la capa en catálogos (CSW)

SRS declarado:  EPSG:4326
SRS nativo:     EPSG:32719 (o el que tenga el shapefile)
Acción:         Reproject native to declared
→ GeoServer reproyecta al vuelo de UTM a WGS84

Bounding Box:
  → "Compute from data" → "Compute from native bounds"
  → SIEMPRE hacer ambos clics
```

**Pestaña Publishing:**

```
Default Style:  polygon (por ahora, en Módulo 3 creamos estilos propios)
WMS Settings:   Queryable ✓ (obligatorio para GetFeatureInfo)
```

#### ⚠️ Limitaciones del Shapefile que debes conocer

| Limitación | Impacto |
|------------|---------|
| Nombres de campo máximo 10 caracteres | `superficie_km2` se trunca a `superfici` |
| No soporta valores nulos | Campos vacíos se llenan con 0 o espacios |
| Un solo tipo de geometría por archivo | No puedes mezclar puntos y polígonos |
| Tamaño máximo ~2 GB | Para datos grandes, usar PostGIS |
| No soporta fechas con hora | Solo fecha, sin hora ni zona horaria |
| Encoding problemático | Tildes y ñ se pueden corromper si el charset es incorrecto |

> **Tip real de producción:** El shapefile es un formato de los años 90. Para proyectos nuevos, usa PostGIS o GeoPackage. Pero vas a encontrar shapefiles en TODOS los proyectos existentes — saber publicarlos es obligatorio.

### Parte 3 — Directory Store: Muchos shapefiles de un golpe (20 min)

En vez de crear un Store por cada shapefile, puedes apuntar a una **carpeta entera**:

> Stores → Add new Store → **Directory of spatial files (shapefiles)**

```
Workspace:  datos_bolivia
Store name: vectoriales_bolivia
URL:        file:///C:/geodatos/bolivia/
            (la carpeta que contiene TODOS los shapefiles)
Charset:    UTF-8
```

GeoServer detecta todos los shapefiles de la carpeta y te permite publicar cada uno individualmente.

#### ¿Cuándo usar Directory Store?

- Tienes 5+ shapefiles del mismo proyecto
- Quieres agregar nuevos shapefiles sin crear stores nuevos (solo copias el .shp a la carpeta y publicas)
- Los datos son estáticos (no cambian frecuentemente)

#### ¿Cuándo NO usar Directory Store?

- Los shapefiles tienen proyecciones diferentes → conflictos
- Necesitas diferentes charsets por archivo
- Datos que se actualizan frecuentemente → mejor PostGIS

### Parte 4 — Metadatos: La diferencia entre amateur y profesional (15 min)

Un servicio WMS profesional tiene metadatos completos. Cuando alguien hace GetCapabilities, ve:

```xml
<Layer queryable="1">
  <Name>datos_bolivia:departamentos</Name>
  <Title>Departamentos de Bolivia</Title>
  <Abstract>Límites administrativos de los 9 departamentos.
    Fuente: INE, 2024. EPSG:4326.</Abstract>
  <KeywordList>
    <Keyword>departamentos</Keyword>
    <Keyword>bolivia</Keyword>
    <Keyword>administrativo</Keyword>
  </KeywordList>
  <BoundingBox>...</BoundingBox>
</Layer>
```

**Regla de oro:** Si mañana alguien ve tu servicio WMS sin poder preguntarte nada, ¿entendería qué datos tiene y de dónde vienen? Si no, tus metadatos son insuficientes.

> **Tip para trabajo real:** Muchas licitaciones y proyectos gubernamentales EXIGEN metadatos completos en los servicios WMS. Si no los pones, no apruebas la entrega.

---

### 💻 Práctica 02

1. Crea un Directory Store apuntando a una carpeta con al menos 3 shapefiles
2. Publica las 3 capas con metadatos completos: nombre técnico, título legible, abstract con fuente y fecha, keywords
3. Verifica cada capa en Layer Preview
4. Prueba la URL GetCapabilities: `http://localhost:8080/geoserver/mi_proyecto/wms?service=WMS&version=1.1.1&request=GetCapabilities`
5. Busca tus metadatos en el XML de respuesta

### ✅ Checklist Clase 02

- [ ] Directory Store creado con múltiples shapefiles
- [ ] Al menos 3 capas publicadas
- [ ] Metadatos completos en cada capa (title, abstract, keywords)
- [ ] SRS y Bounding Box correctamente configurados
- [ ] Todas las capas visibles en Layer Preview
- [ ] GetCapabilities devuelve un XML con tus capas

---

## Clase 03 — Conexión a PostGIS y Grupos de Capas

### 🎯 Objetivo

Conectar GeoServer a una base de datos PostGIS, publicar capas desde tablas y vistas SQL, y organizar todo en grupos de capas profesionales.

### Parte 1 — ¿Por qué PostGIS y no solo Shapefiles? (10 min)

| Característica | Shapefile | PostGIS |
|---------------|-----------|---------|
| Nombres de campo | Máximo 10 chars | Sin límite |
| Tamaño de datos | ~2 GB máximo | Sin límite práctico |
| Consultas SQL | No posible | SQL completo |
| Edición concurrente | No | Sí (múltiples usuarios) |
| Actualización | Reemplazar archivo | UPDATE/INSERT en vivo |
| Tipos de geometría | Uno por archivo | Mixtos en misma tabla |
| Relaciones entre tablas | No | JOINs, Foreign Keys |
| Vistas SQL | No | Sí — capas virtuales dinámicas |
| Índices espaciales | .shx (básico) | GiST (muy eficiente) |

> **Regla práctica:** Para datos estáticos simples → Shapefile está bien. Para CUALQUIER otra cosa → PostGIS.

### Parte 2 — Preparar PostGIS (15 min)

#### Verificar que PostGIS está instalado

```sql
-- En psql o PgAdmin:
SELECT PostGIS_Full_Version();
-- Debe devolver algo como: POSTGIS="3.4.0" GEOS="3.12.0" ...
```

#### Crear la base de datos del curso

```sql
-- Crear base de datos
CREATE DATABASE geoserver_curso;

-- Conectar a la base
\c geoserver_curso

-- Activar extensión PostGIS
CREATE EXTENSION postgis;

-- Verificar
SELECT PostGIS_Version();
```

#### Importar datos con shp2pgsql o QGIS

**Opción A — Línea de comandos:**
```bash
shp2pgsql -s 4326 -I departamentos.shp public.departamentos | psql -d geoserver_curso
shp2pgsql -s 4326 -I rios_principales.shp public.rios | psql -d geoserver_curso
shp2pgsql -s 4326 -I capitales.shp public.capitales | psql -d geoserver_curso
```

**Opción B — Desde QGIS:**
1. Cargar shapefile en QGIS
2. Clic derecho en la capa → Exportar → Guardar Features como...
3. Formato: PostgreSQL → Conexión a tu base → Nombre de tabla
4. CRS: EPSG:4326

### Parte 3 — Conectar GeoServer a PostGIS (20 min)

> Stores → Add new Store → **PostGIS**

```
Workspace:           datos_bolivia
Data Source Name:    postgis_bolivia
Descripción:         Conexión a la base PostGIS del curso

Connection Parameters:
  host:     localhost
  port:     5432
  database: geoserver_curso
  schema:   public
  user:     postgres
  passwd:   tu_contraseña

  Expose primary keys:         ✓
  Validate connections:        ✓
  max connections:             20
  min connections:             1
  Connection timeout:          20 (segundos)
```

> Clic en **Save**. Si la conexión es exitosa, verás la lista de tablas disponibles para publicar.

#### ⚠️ Error #1: "Cannot connect to database"

Causas más comunes:
1. PostgreSQL no está corriendo → `sudo systemctl status postgresql`
2. Puerto incorrecto → verificar `postgresql.conf` → `port = 5432`
3. Contraseña incorrecta
4. `pg_hba.conf` no permite conexiones locales → agregar `host all all 127.0.0.1/32 md5`
5. Firewall bloqueando el puerto

> **Tip:** Siempre prueba la conexión PRIMERO desde psql o PgAdmin antes de configurar en GeoServer. Si no funciona ahí, no va a funcionar en GeoServer.

### Parte 4 — Publicar capas desde PostGIS (20 min)

Después de crear el Store PostGIS, GeoServer lista todas las tablas con geometría.

> Clic en **Publish** junto a `departamentos`

La configuración es igual que con Shapefile, pero con ventajas:

1. **Nombres de campo completos** — `superficie_km2` se ve completo, no truncado
2. **SRS se detecta automáticamente** — PostGIS almacena el SRID en la tabla `geometry_columns`
3. **Actualizaciones en vivo** — Si modificas datos en PostGIS, GeoServer los refleja sin republicar

### Parte 5 — Vistas SQL: Capas virtuales dinámicas (15 min)

Una Vista SQL es una capa que no existe como tabla — se genera al vuelo desde una consulta SQL. Es una de las funcionalidades más potentes de GeoServer + PostGIS.

> Layers → Add new layer → Seleccionar tu Store PostGIS → **"Configure new SQL view..."**

#### Ejemplo: Solo municipios de La Paz con población > 10,000

```sql
SELECT
  gid,
  nombre,
  departamento,
  poblacion,
  superficie_km2,
  geom
FROM municipios
WHERE departamento = 'La Paz'
  AND poblacion > 10000
ORDER BY poblacion DESC
```

**Configuración de la Vista SQL:**
```
View Name:   municipios_lapaz_grandes
SQL:         (pegar la consulta de arriba)
Guess parameters from SQL:  → Clic
Refresh:     → Clic para detectar columnas
SRID:        4326
Geometry type: MultiPolygon
```

#### Ejemplo: Vista con JOIN — Municipios con datos de censo

```sql
SELECT
  m.gid,
  m.nombre,
  m.geom,
  c.poblacion_2024,
  c.tasa_crecimiento,
  c.densidad_hab_km2
FROM municipios m
JOIN censo_2024 c ON m.codigo_ine = c.codigo_municipio
```

> **Tip avanzado:** Las vistas SQL pueden recibir parámetros desde la URL con `%param%`. Esto permite capas dinámicas controladas por el usuario.

### Parte 6 — Grupos de Capas (15 min)

Un Layer Group agrupa varias capas en un solo servicio. El cliente pide UNA URL y recibe todas las capas superpuestas.

> Menú lateral → **Layer Groups** → **Add new layer group**

```
Name:       mapa_base_bolivia
Title:      Mapa Base de Bolivia
Workspace:  datos_bolivia

Layers (en orden de abajo hacia arriba):
  1. departamentos     (fondo)
  2. rios              (encima)
  3. carreteras        (encima)
  4. capitales         (encima, último = se dibuja arriba)
```

#### ¿Cuándo usar Layer Groups?

- Para ofrecer un "mapa base" completo con una sola URL WMS
- Para simplificar la vida del cliente que consume el servicio
- Para controlar el orden de renderizado de las capas

> **Tip real:** Muchas IDEs (Infraestructuras de Datos Espaciales) publican sus mapas base como Layer Groups. El consumidor solo agrega una URL y obtiene todo el mapa organizado.

---

### 💻 Práctica 03

1. Instala PostGIS si no lo tienes (o verifica que está activo)
2. Crea la base de datos `geoserver_curso` con extensión PostGIS
3. Importa al menos 2 shapefiles a PostGIS (con shp2pgsql o QGIS)
4. Crea un Store PostGIS en GeoServer y conecta a la base
5. Publica al menos 2 capas desde PostGIS
6. Crea una Vista SQL que filtre datos (ej: solo registros de una región)
7. Crea un Layer Group que agrupe 3+ capas con orden lógico
8. Verifica el Layer Group en Layer Preview

### 🚀 Reto Extra — Vista SQL parametrizada

Crea una vista SQL con un parámetro `%departamento%` que permita filtrar municipios por departamento desde la URL del servicio WMS:

```sql
SELECT gid, nombre, poblacion, geom
FROM municipios
WHERE departamento = '%departamento%'
```

Luego prueba acceder cambiando el parámetro en la URL.

### ✅ Checklist Clase 03

- [ ] PostGIS funcionando con datos importados
- [ ] Store PostGIS conectado en GeoServer
- [ ] Capas publicadas desde tablas PostGIS
- [ ] Al menos una Vista SQL funcional
- [ ] Layer Group creado con 3+ capas ordenadas
- [ ] Layer Group visible en Layer Preview

---

## 📝 Errores comunes del Módulo 2

1. **Shapefile con caracteres rotos** → Charset del Store en UTF-8, verificar encoding original del .dbf
2. **Cannot connect to PostGIS** → Verificar host, puerto, contraseña, pg_hba.conf
3. **Tabla sin geometría detectada** → Verificar que la columna geom tiene tipo `geometry` y está en `geometry_columns`
4. **Vista SQL sin SRID** → Especificar SRID manualmente al crear la vista
5. **Layer Group en blanco** → Verificar que las capas individuales funcionan primero
6. **Bounding Box negativo o inválido** → Siempre "Compute from data" antes de guardar
7. **PostGIS lento con datos grandes** → Verificar que la tabla tiene índice espacial: `CREATE INDEX idx_geom ON tabla USING GIST(geom);`

## 💡 Tips de producción

- **Siempre usa PostGIS para producción.** Shapefiles son para intercambio y datos estáticos pequeños.
- **Nombra tus tablas PostGIS en minúsculas sin espacios.** GeoServer respeta el case de PostgreSQL y las mayúsculas causan problemas.
- **Crea índices espaciales** en TODAS las tablas con geometría. Sin índice, las consultas WMS son 10-100x más lentas.
- **Las vistas SQL son tu arma secreta.** En vez de crear capas separadas para cada departamento, crea UNA vista parametrizada.
- **Documenta tus Layer Groups.** El abstract del grupo debe explicar qué capas incluye y para qué sirve.