# Módulo 4 · Servicios WMS, WFS & Filtros CQL ⚡

> **Clase 5 · 2 horas**
> Dominas los dos servicios más importantes de GeoServer: WMS para mapas y WFS para datos. Y aprendes CQL — el arma secreta para hacer consultas dinámicas sin tocar el servidor.

---

## 🎯 Objetivos

- Entender a fondo las operaciones WMS: GetCapabilities, GetMap, GetFeatureInfo, GetLegendGraphic
- Configurar y exponer servicios WFS: GetCapabilities, GetFeature, DescribeFeatureType
- Dominar filtros CQL para consultas dinámicas desde la URL
- Combinar CQL con operadores lógicos y espaciales
- Probar y validar servicios desde el navegador y QGIS
- Entender la diferencia práctica entre WMS y WFS y cuándo usar cada uno

---

## Parte 1 — WMS a fondo: Las 4 operaciones que necesitas (25 min)

### 1.1 — GetCapabilities: "¿Qué tienes disponible?"

Es la primera petición que hace cualquier cliente. Devuelve un XML con el catálogo completo: qué capas hay, qué formatos soporta, qué SRS acepta.

```
http://localhost:8080/geoserver/datos_bolivia/wms?
  service=WMS
  &version=1.1.1
  &request=GetCapabilities
```

**¿Para qué sirve en la práctica?**
- Cuando agregas un WMS en QGIS, internamente hace GetCapabilities para listar las capas
- Si tu servicio no devuelve GetCapabilities correctamente, ningún cliente puede conectarse
- Los metadatos que configuraste en el Módulo 2 (Title, Abstract, Keywords) aparecen aquí

> **Tip:** Abre el XML en el navegador y busca tus capas con Ctrl+F. Verifica que los metadatos se ven completos. Si están vacíos, vuelve a la configuración de la capa.

### 1.2 — GetMap: "Dame una imagen del mapa"

La operación principal de WMS. Recibe parámetros y devuelve una imagen PNG o JPEG.

```
http://localhost:8080/geoserver/datos_bolivia/wms?
  service=WMS
  &version=1.1.1
  &request=GetMap
  &layers=datos_bolivia:departamentos
  &styles=estilo_departamentos
  &srs=EPSG:4326
  &bbox=-69.7,-22.9,-57.4,-9.6
  &width=800
  &height=600
  &format=image/png
  &transparent=true
```

**Parámetros que puedes jugar:**

| Parámetro | Opciones | Efecto |
|-----------|----------|--------|
| `format` | `image/png`, `image/jpeg`, `image/png8` | PNG transparente, JPEG comprimido, PNG 8-bit (más liviano) |
| `transparent` | `true`, `false` | Fondo transparente o blanco |
| `bgcolor` | `0xFFFFFF`, `0x000000` | Color de fondo cuando transparent=false |
| `width/height` | Cualquier número | Tamaño de la imagen. Más grande = más detalle pero más pesado |
| `styles` | Nombre del estilo o vacío | Vacío usa el estilo por defecto |
| `srs` | `EPSG:4326`, `EPSG:3857` | Sistema de coordenadas de salida |

**Ejemplo práctico — Múltiples capas en una sola imagen:**

```
&layers=datos_bolivia:departamentos,datos_bolivia:rios,datos_bolivia:capitales
&styles=estilo_departamentos,estilo_rios,estilo_capitales
```

> **Tip:** El orden de `layers` importa. El primero se dibuja abajo, el último arriba. Los estilos deben coincidir en cantidad y orden con las capas.

### 1.3 — GetFeatureInfo: "¿Qué hay en este punto?"

Consulta los atributos del feature en una posición específica del mapa. Es el equivalente a "Identificar" en QGIS.

```
http://localhost:8080/geoserver/datos_bolivia/wms?
  service=WMS
  &version=1.1.1
  &request=GetFeatureInfo
  &layers=datos_bolivia:departamentos
  &query_layers=datos_bolivia:departamentos
  &info_format=application/json
  &srs=EPSG:4326
  &bbox=-69.7,-22.9,-57.4,-9.6
  &width=800
  &height=600
  &x=400
  &y=300
  &feature_count=5
```

**Formatos de respuesta:**

| Formato | Uso |
|---------|-----|
| `application/json` | Para apps que parsean datos (el más útil) |
| `text/html` | Para mostrar directamente en un popup web |
| `text/plain` | Texto simple |
| `application/vnd.ogc.gml` | GML para clientes GIS |

**Ejemplo de respuesta JSON:**

```json
{
  "type": "FeatureCollection",
  "features": [{
    "type": "Feature",
    "id": "departamentos.3",
    "geometry": { "type": "MultiPolygon", "coordinates": [...] },
    "properties": {
      "nombre": "La Paz",
      "codigo": "02",
      "poblacion": 2926996,
      "superficie_km2": 133985,
      "capital": "La Paz"
    }
  }]
}
```

> **Tip:** Prueba GetFeatureInfo en el navegador ANTES de integrarlo en una app. Si no devuelve datos, verifica que la capa tiene `Queryable` activado en Publishing.

### 1.4 — GetLegendGraphic: "Dame la leyenda"

Genera una imagen de la leyenda basada en el estilo SLD de la capa.

```
http://localhost:8080/geoserver/wms?
  service=WMS
  &version=1.1.1
  &request=GetLegendGraphic
  &layer=datos_bolivia:departamentos
  &style=estilo_departamentos
  &format=image/png
  &width=20
  &height=20
  &LEGEND_OPTIONS=fontName:Arial;fontSize:12;fontAntiAliasing:true
```

**LEGEND_OPTIONS — Personalización avanzada:**

| Opción | Ejemplo | Efecto |
|--------|---------|--------|
| `fontName` | `Arial` | Fuente de la leyenda |
| `fontSize` | `12` | Tamaño del texto |
| `fontAntiAliasing` | `true` | Texto suavizado |
| `forceLabels` | `on` | Forzar etiquetas aunque sean muchas |
| `bgColor` | `0xFFFFFF` | Color de fondo |
| `fontColor` | `0x333333` | Color del texto |

> **Tip:** GetLegendGraphic es lo que usan los visores web para mostrar la leyenda del mapa. La leyenda se genera automáticamente desde los `<Title>` de cada Rule en el SLD — por eso es importante que cada Rule tenga un Title descriptivo.

---

## Parte 2 — WFS: Acceso a los datos vectoriales (20 min)

### ¿Cuándo usar WFS en vez de WMS?

| Necesitas... | Usa |
|-------------|-----|
| Ver el mapa como imagen | WMS |
| Descargar datos para análisis | WFS |
| Hacer consultas espaciales desde el cliente | WFS |
| Mostrar popups con información | WMS GetFeatureInfo o WFS |
| Editar features remotamente | WFS-T (transaccional) |
| Exportar a GeoJSON/Shapefile | WFS |

### 2.1 — GetCapabilities (WFS)

```
http://localhost:8080/geoserver/datos_bolivia/wfs?
  service=WFS
  &version=2.0.0
  &request=GetCapabilities
```

### 2.2 — DescribeFeatureType: "¿Qué campos tiene esta capa?"

Devuelve el schema (estructura) de una capa: nombres de campos, tipos de datos, si son obligatorios.

```
http://localhost:8080/geoserver/datos_bolivia/wfs?
  service=WFS
  &version=2.0.0
  &request=DescribeFeatureType
  &typeName=datos_bolivia:departamentos
```

**Respuesta (simplificada):**
```xml
<element name="nombre" type="string"/>
<element name="poblacion" type="int"/>
<element name="superficie_km2" type="double"/>
<element name="geom" type="gml:MultiPolygonPropertyType"/>
```

> **Tip:** Usa DescribeFeatureType para saber los nombres EXACTOS de los campos antes de escribir filtros CQL. Si el campo se llama `poblacion` y escribes `Poblacion`, no va a funcionar (PostGIS es case-sensitive).

### 2.3 — GetFeature: "Dame los datos"

La operación principal de WFS. Devuelve los features con geometría y atributos.

**Todos los departamentos (GeoJSON):**
```
http://localhost:8080/geoserver/datos_bolivia/wfs?
  service=WFS
  &version=2.0.0
  &request=GetFeature
  &typeName=datos_bolivia:departamentos
  &outputFormat=application/json
```

**Solo campos específicos:**
```
&propertyName=nombre,poblacion,geom
```

**Limitar cantidad:**
```
&count=5
```

**Ordenar:**
```
&sortBy=poblacion+D
```
(+D = descendente, +A = ascendente)

**Formatos de salida:**

| Formato | Parámetro | Uso |
|---------|-----------|-----|
| GeoJSON | `application/json` | Apps web y móviles |
| GML 3 | `application/gml+xml; version=3.2` | Clientes GIS estándar |
| Shapefile | `SHAPE-ZIP` | Descarga de datos para QGIS/ArcGIS |
| CSV | `csv` | Análisis en Excel/Python |
| KML | `application/vnd.google-earth.kml+xml` | Google Earth |

> **Tip práctico:** `outputFormat=SHAPE-ZIP` descarga un ZIP con el shapefile completo. Esto permite que tus usuarios descarguen datos directamente desde GeoServer sin que tú hagas nada — el servidor genera el shapefile al vuelo.

---

## Parte 3 — Filtros CQL: Consultas dinámicas desde la URL (30 min)

### ¿Qué es CQL?

CQL (Common Query Language) es un lenguaje simple para filtrar datos directamente desde la URL del servicio. Es como un WHERE de SQL pero que se envía como parámetro HTTP. El servidor filtra los datos ANTES de enviarlos — más eficiente que filtrar en el cliente.

### CQL funciona tanto en WMS como en WFS

**En WMS (GetMap):**
```
&CQL_FILTER=departamento='La Paz'
```
→ Solo renderiza los features que cumplen el filtro

**En WFS (GetFeature):**
```
&CQL_FILTER=departamento='La Paz'
```
→ Solo devuelve los features que cumplen el filtro

### 3.1 — Filtros por atributo de texto

```
# Exacto
CQL_FILTER=nombre='La Paz'

# Contiene (LIKE con %)
CQL_FILTER=nombre LIKE '%Paz%'

# Empieza con
CQL_FILTER=nombre LIKE 'San%'

# Diferente de
CQL_FILTER=nombre <> 'Pando'

# En una lista
CQL_FILTER=nombre IN ('La Paz','Cochabamba','Santa Cruz')
```

### 3.2 — Filtros por valor numérico

```
# Mayor que
CQL_FILTER=poblacion > 1000000

# Menor o igual que
CQL_FILTER=superficie_km2 <= 50000

# Rango (entre)
CQL_FILTER=poblacion BETWEEN 100000 AND 500000

# Igual
CQL_FILTER=codigo = '02'
```

### 3.3 — Operadores lógicos: AND, OR, NOT

```
# AND — ambas condiciones deben cumplirse
CQL_FILTER=departamento='La Paz' AND poblacion > 50000

# OR — al menos una condición
CQL_FILTER=departamento='La Paz' OR departamento='Cochabamba'

# NOT — excluir
CQL_FILTER=NOT (departamento='Pando')

# Combinados con paréntesis
CQL_FILTER=(departamento='La Paz' OR departamento='Cochabamba') AND poblacion > 100000
```

### 3.4 — Filtros espaciales

```
# Dentro de un rectángulo (BBOX)
CQL_FILTER=BBOX(geom, -68.5, -17.0, -67.5, -16.0)

# Dentro de un radio (en metros) desde un punto
CQL_FILTER=DWITHIN(geom, POINT(-68.15 -16.50), 10000, meters)

# Intersecta con un polígono
CQL_FILTER=INTERSECTS(geom, POLYGON((-68.5 -17.0, -67.5 -17.0, -67.5 -16.0, -68.5 -16.0, -68.5 -17.0)))
```

> **CUIDADO con las coordenadas espaciales en CQL:** El orden es `POINT(longitud latitud)` — longitud primero. Es diferente a LatLng de Flutter.

### 3.5 — CQL con fechas

```
# Después de una fecha
CQL_FILTER=fecha_creacion > '2020-01-01'

# Rango de fechas
CQL_FILTER=fecha_creacion BETWEEN '2020-01-01' AND '2024-12-31'
```

### 3.6 — Funciones CQL

GeoServer soporta funciones dentro de CQL:

```
# Longitud de texto
CQL_FILTER=strLength(nombre) > 10

# Convertir a mayúsculas para comparar sin case
CQL_FILTER=strToUpperCase(nombre) = 'LA PAZ'

# Área del polígono (en unidades del SRS)
CQL_FILTER=area(geom) > 10000000000

# Calcular distancia (en grados si SRS es 4326)
CQL_FILTER=distance(geom, POINT(-68.15 -16.50)) < 1
```

---

## Parte 4 — Ejercicios prácticos completos (20 min)

### Ejercicio A — Mapa de departamentos del eje troncal

```
http://localhost:8080/geoserver/datos_bolivia/wms?
  service=WMS&version=1.1.1&request=GetMap
  &layers=datos_bolivia:departamentos
  &styles=estilo_departamentos
  &CQL_FILTER=nombre IN ('La Paz','Cochabamba','Santa Cruz')
  &srs=EPSG:4326
  &bbox=-69.7,-22.9,-57.4,-9.6
  &width=800&height=600
  &format=image/png&transparent=true
```

### Ejercicio B — Descargar municipios populosos en GeoJSON

```
http://localhost:8080/geoserver/datos_bolivia/wfs?
  service=WFS&version=2.0.0&request=GetFeature
  &typeName=datos_bolivia:municipios
  &outputFormat=application/json
  &CQL_FILTER=poblacion > 100000
  &sortBy=poblacion+D
  &propertyName=nombre,departamento,poblacion,geom
```

### Ejercicio C — Descargar como Shapefile

```
http://localhost:8080/geoserver/datos_bolivia/wfs?
  service=WFS&version=2.0.0&request=GetFeature
  &typeName=datos_bolivia:rios
  &outputFormat=SHAPE-ZIP
  &CQL_FILTER=cuenca='Amazónica'
```

→ Descarga un ZIP con el shapefile listo para abrir en QGIS.

### Ejercicio D — Municipios cerca de La Paz (espacial)

```
http://localhost:8080/geoserver/datos_bolivia/wfs?
  service=WFS&version=2.0.0&request=GetFeature
  &typeName=datos_bolivia:municipios
  &outputFormat=application/json
  &CQL_FILTER=DWITHIN(geom, POINT(-68.15 -16.50), 50000, meters)
  &propertyName=nombre,poblacion,geom
```

### Ejercicio E — GetFeatureInfo con info_format JSON

1. Abrir Layer Preview → OpenLayers
2. Hacer clic sobre un departamento
3. Ver la URL que se genera en la consola del navegador (F12 → Network)
4. Copiar esa URL y cambiar `info_format=text/html` por `info_format=application/json`
5. Pegar en otra pestaña → ver el JSON crudo

---

### 💻 Práctica 05

1. Prueba GetCapabilities de WMS y WFS en el navegador. Busca tus capas en el XML.
2. Construye una URL GetMap con al menos 2 capas superpuestas y un estilo personalizado.
3. Prueba GetFeatureInfo en Layer Preview. Cambia el formato a JSON y analiza la respuesta.
4. Genera la leyenda con GetLegendGraphic y personaliza con LEGEND_OPTIONS.
5. Prueba GetFeature de WFS con outputFormat=application/json. Limita a 10 registros.
6. Descarga una capa como Shapefile con outputFormat=SHAPE-ZIP.
7. Aplica al menos 5 filtros CQL diferentes:
   - Por texto exacto
   - Por rango numérico
   - Con AND/OR
   - Espacial (BBOX o DWITHIN)
   - Con IN (lista de valores)
8. Conecta tu GeoServer desde QGIS como WMS y como WFS. Verifica que los filtros CQL también funcionan desde QGIS.

### 🚀 Reto Extra — CQL dinámico desde el navegador

Crea un archivo HTML simple con un `<input>` de texto y un `<img>`. Al escribir un filtro CQL y presionar Enter, la imagen se actualiza con el mapa filtrado:

```html
<input id="filtro" placeholder="nombre='La Paz'" style="width:400px">
<br>
<img id="mapa" width="800" height="600">

<script>
document.getElementById('filtro').addEventListener('keypress', function(e) {
  if (e.key === 'Enter') {
    var cql = encodeURIComponent(this.value);
    document.getElementById('mapa').src =
      'http://localhost:8080/geoserver/datos_bolivia/wms?' +
      'service=WMS&version=1.1.1&request=GetMap' +
      '&layers=datos_bolivia:departamentos' +
      '&srs=EPSG:4326&bbox=-69.7,-22.9,-57.4,-9.6' +
      '&width=800&height=600&format=image/png' +
      '&CQL_FILTER=' + cql;
  }
});
</script>
```

> Este reto es un adelanto del Módulo 6 (Visor Web) pero en su versión más simple.

### ✅ Checklist Clase 05

- [ ] GetCapabilities WMS y WFS probados
- [ ] GetMap con múltiples capas y estilos
- [ ] GetFeatureInfo con formato JSON
- [ ] GetLegendGraphic con LEGEND_OPTIONS
- [ ] GetFeature con GeoJSON y SHAPE-ZIP
- [ ] Al menos 5 filtros CQL diferentes
- [ ] CQL espacial (BBOX o DWITHIN)
- [ ] Conexión desde QGIS como WMS y WFS

---

## 📝 Errores comunes

1. **CQL_FILTER con error de sintaxis** → Strings entre comillas simples `'La Paz'`, NO dobles. Campos sin comillas.
2. **CQL no filtra nada** → Nombre del campo incorrecto. Usar DescribeFeatureType para verificar nombres exactos.
3. **GetFeatureInfo vacío** → Capa no tiene Queryable activado. Verificar en Publishing.
4. **WFS devuelve error 400** → typeName incorrecto. Formato: `workspace:capa`. Verificar con GetCapabilities.
5. **SHAPE-ZIP descarga vacío** → Filtro CQL no coincide con ningún registro. Probar sin filtro primero.
6. **CQL espacial error** → Coordenadas en orden `POINT(lng lat)`. Geometría debe llamarse como está en PostGIS (`geom`, `the_geom`, etc.).
7. **Caracteres especiales en CQL** → Encodear la URL: espacios → `%20`, apóstrofes en nombres con tilde → usar LIKE.
8. **GetMap con múltiples capas desalineadas** → Todas deben tener el mismo SRS o GeoServer intentará reproyectar.

## 💡 Tips de producción

- **GetCapabilities es tu carta de presentación.** Si los metadatos están vacíos, tu servicio parece amateur. Si están completos, parece profesional.
- **CQL es más potente que crear capas separadas.** En vez de 9 capas (una por departamento), crea UNA capa y filtra con CQL. Menos mantenimiento.
- **SHAPE-ZIP es tu mejor amigo** para dar acceso a datos. Los usuarios descargan shapefiles directamente desde la URL sin que tú intervengas.
- **Siempre prueba CQL en el navegador primero.** Si funciona en la URL, funciona en cualquier cliente.
- **WFS + CQL en QGIS** es extremadamente potente. Puedes filtrar datos remotos sin descargar todo.
- **Para apps web, usa WMS para visualizar y WFS solo cuando necesitas datos** (popups, análisis). WMS es más liviano.