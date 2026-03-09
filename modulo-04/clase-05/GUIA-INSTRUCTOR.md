# 🎓 Guía del Instructor — Clase 05

## Servicios WMS, WFS & Filtros CQL

**Duración:** 2 horas (120 minutos)
**Módulo:** 4 de 7

---

## ⏱️ Cronograma

| Tiempo | Duración | Sección | Actividad |
|--------|----------|---------|-----------|
| 00:00 | 5 min  | Repaso | Verificar estilos SLD del Módulo 3 |
| 00:05 | 25 min | Parte 1 | WMS a fondo: GetMap, GetFeatureInfo, GetLegendGraphic |
| 00:30 | 20 min | Parte 2 | WFS: GetFeature, DescribeFeatureType, SHAPE-ZIP |
| 00:50 | 10 min | Descanso | |
| 01:00 | 30 min | Parte 3 | CQL: texto, numérico, AND/OR, espacial |
| 01:30 | 20 min | Parte 4 | Ejercicios prácticos (A-E) guiados |
| 01:50 | 10 min | Cierre | Reto extra HTML, preview Módulo 5 |

---

## 🎬 Desarrollo

### Apertura (5 min)

> "Tienen datos publicados con estilos bonitos. Pero hasta ahora solo los ven en Layer Preview. Hoy aprendemos a CONTROLAR los servicios: qué pedir, cómo pedir, y cómo filtrar. Después de esta clase van a poder construir URLs que hagan exactamente lo que necesitan — sin tocar GeoServer."

### Parte 1 — WMS a fondo (25 min)

**La clave de esta parte es el navegador como herramienta.**

Secuencia EN VIVO:

1. **GetCapabilities** — Pegar URL en navegador, buscar una capa con Ctrl+F
   > "Este XML es la carta del restaurante. Todo lo que tu servidor ofrece está aquí."

2. **GetMap** — Construir la URL parámetro por parámetro
   > "Vamos a construir la URL pieza por pieza. Empezamos con solo la capa y vamos agregando."

   Mostrar que cambiar `width=200` vs `width=1200` cambia la resolución.
   Mostrar que cambiar `format=image/jpeg` comprime la imagen.
   Mostrar que `transparent=true` con PNG permite superponer capas.

3. **GetFeatureInfo** — Ir a Layer Preview → OpenLayers → clic en un departamento
   > "¿Ven el popup? Eso es GetFeatureInfo. QGIS hace lo mismo cuando usas 'Identificar features' sobre una capa WMS."

   Abrir Network en F12, copiar la URL de GetFeatureInfo, cambiar a `info_format=application/json`, mostrar el JSON.

4. **GetLegendGraphic** — Mostrar la leyenda como imagen
   > "Esta imagen la genera GeoServer automáticamente desde su SLD. Si las Rules tienen buenos Titles, la leyenda se ve profesional."

   Mostrar personalización con LEGEND_OPTIONS.

### Parte 2 — WFS (20 min)

**Momento clave: La diferencia entre WMS y WFS.**

> "WMS te da una FOTO del mapa. WFS te da los DATOS reales. Miren..."

1. Mostrar GetMap → imagen PNG. "Esto es WMS. Una foto. No puedes hacer nada con los datos."
2. Mostrar GetFeature → GeoJSON. "Esto es WFS. Los datos reales con coordenadas y atributos. Puedes analizarlos, filtrarlos, graficarlos."

**El truco de SHAPE-ZIP:**

```
&outputFormat=SHAPE-ZIP
```

> "Acabo de crear un servicio de descarga de shapefiles. Cualquier usuario con esta URL puede descargar los datos como shapefile listo para QGIS. Sin que yo haga nada."

Mostrar EN VIVO: descargar el ZIP, descomprimir, abrir en QGIS.

### Parte 3 — CQL (30 min)

**Esta es la parte más interactiva de la clase.** Construye URLs EN VIVO y ve los resultados instantáneamente.

Secuencia de complejidad creciente:

1. **Texto exacto** (3 min)
   ```
   CQL_FILTER=nombre='La Paz'
   ```
   > "Solo La Paz en el mapa. Todo lo demás desapareció."

2. **Numérico** (3 min)
   ```
   CQL_FILTER=poblacion>1000000
   ```
   > "Solo departamentos con más de 1 millón de habitantes."

3. **LIKE** (3 min)
   ```
   CQL_FILTER=nombre LIKE 'San%'
   ```
   > "Todo lo que empieza con 'San': Santa Cruz, San José..."

4. **IN** (3 min)
   ```
   CQL_FILTER=nombre IN ('La Paz','Cochabamba','Santa Cruz')
   ```
   > "El eje troncal. Tres departamentos con un solo filtro."

5. **AND** (3 min)
   ```
   CQL_FILTER=departamento='La Paz' AND poblacion>50000
   ```
   > "Municipios grandes de La Paz. Combinamos dos condiciones."

6. **OR** (2 min)
   ```
   CQL_FILTER=departamento='La Paz' OR departamento='Oruro'
   ```

7. **BETWEEN** (2 min)
   ```
   CQL_FILTER=poblacion BETWEEN 100000 AND 500000
   ```

8. **Espacial — BBOX** (5 min)
   ```
   CQL_FILTER=BBOX(geom,-68.5,-17.0,-67.5,-16.0)
   ```
   > "Solo los features que están dentro de este rectángulo. Es como un clip espacial pero sin crear datos nuevos."

9. **Espacial — DWITHIN** (5 min)
   ```
   CQL_FILTER=DWITHIN(geom,POINT(-68.15 -16.50),50000,meters)
   ```
   > "Todo lo que está a menos de 50 km de La Paz. ¿Ven el potencial? Filtro por proximidad."

**Después de cada filtro, muestra el resultado en el navegador.** El feedback visual instantáneo es lo que hace que CQL "haga clic" en los estudiantes.

### Parte 4 — Ejercicios guiados (20 min)

Hacer los ejercicios A-E del README juntos. Si alguno es rápido, que lo haga solo. Si alguno se traba, hacerlo en pantalla.

**El ejercicio E (ver GetFeatureInfo en Network)** es particularmente revelador:

> "Vamos a espiar qué hace el visor de GeoServer por detrás. Abran F12 → Network → hagan clic en el mapa. ¿Ven esa petición? ESO es lo que un visor web hace cuando el usuario hace clic. En el Módulo 6 van a construir ese visor."

### Cierre (10 min)

> "Ahora saben controlar GeoServer sin tocar la interfaz de administración. Todo se hace desde la URL. CQL es su arma secreta: con una sola línea de texto filtran datos del servidor en tiempo real."

**Si sobra tiempo:** Mostrar el reto extra del HTML con CQL dinámico. Es simple pero impactante — los estudiantes escriben un filtro y el mapa se actualiza.

**Preview Módulo 5:**

> "Tienen un servidor potente. Pero ahora mismo CUALQUIERA que conozca la URL puede acceder a todo. En la siguiente clase ponemos seguridad: usuarios, roles, y restricción por capa. Solo los autorizados ven lo que deben ver."

---

## ⚠️ Errores Comunes

| Problema | Solución |
|----------|----------|
| CQL con comillas dobles | Usar comillas SIMPLES para textos: `'La Paz'` no `"La Paz"` |
| Campo no encontrado | Verificar nombre exacto con DescribeFeatureType. PostGIS es case-sensitive |
| GetFeatureInfo vacío | Capa sin Queryable activado en Publishing |
| SHAPE-ZIP vacío | CQL no coincide. Probar sin filtro primero |
| CQL espacial error | POINT(lng lat) — longitud primero. Nombre del campo de geometría correcto |
| URL demasiado larga | Encodear caracteres especiales. Espacios → %20 |
| WFS error 400 | typeName incorrecto: debe ser workspace:capa |

## 💡 Tips

1. **El navegador es tu laboratorio.** Toda esta clase se puede hacer sin salir del navegador.
2. **Construye URLs paso a paso** — agrega un parámetro, prueba, agrega otro, prueba. No escribas la URL completa de golpe.
3. **SHAPE-ZIP impresiona** — los estudiantes ven que pueden crear un servicio de descarga con una URL.
4. **CQL es adictivo** — una vez que ven los resultados instantáneos, quieren probar todo.
5. **Conecta con su realidad:** "¿Su jefe quiere ver solo los municipios de La Paz? Un filtro CQL. ¿Quiere descargarlo como shapefile? SHAPE-ZIP."
6. **Ten 5-6 URLs CQL listas** en un archivo de texto para copiar rápido si la demo se complica.
7. **Si sobra mucho tiempo:** Hacer el reto extra HTML con CQL dinámico en vivo con los estudiantes.