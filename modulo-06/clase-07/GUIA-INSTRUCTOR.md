# 🎓 Guía del Instructor — Clase 07

## Visor Web & App Móvil

**Duración:** 2 horas (120 minutos)
**Módulo:** 6 de 7

---

## ⏱️ Cronograma

| Tiempo | Duración | Sección | Actividad |
|--------|----------|---------|-----------|
| 00:00 | 5 min  | Repaso | Verificar práctica Módulo 5 (seguridad) |
| 00:05 | 10 min | Intro | Arquitectura: un servidor, múltiples clientes |
| 00:15 | 10 min | Paso 1 | Visor base Leaflet + mapa OSM |
| 00:25 | 15 min | Paso 2 | Agregar capas WMS de tu GeoServer |
| 00:40 | 10 min | Paso 3 | Panel de capas con checkboxes + leyenda |
| 00:50 | 10 min | Descanso | |
| 01:00 | 15 min | Paso 4 | GetFeatureInfo con popup |
| 01:15 | 10 min | Paso 5 | Filtro CQL desde la interfaz |
| 01:25 | 15 min | Parte 2 | Demo Flutter: mismas capas en app móvil |
| 01:40 | 10 min | Extra | Actividad A (mapas base) o B (WFS en Leaflet) |
| 01:50 | 10 min | Cierre | Preview Módulo 7 (deploy) |

---

## 🎬 Desarrollo

### Apertura (10 min)

> "Todo lo que hicimos hasta ahora — publicar datos, estilos, filtros, seguridad — tenía un propósito: servir mapas a aplicaciones reales. Hoy construimos esas aplicaciones. Un visor web con Leaflet y una demo de app móvil con Flutter. Ambos consumen las mismas URLs de su GeoServer."

**Mostrar el diagrama de arquitectura:**

> "GeoServer es el motor. Es como el motor de un auto — no lo ves, pero hace todo el trabajo. El visor web es el auto sedán. La app Flutter es la camioneta. El QGIS es el camión de carga. Todos usan el mismo motor."

### Paso 1 — Visor base (10 min)

**Construye EN VIVO.**

> "Un archivo HTML, dos líneas de Leaflet desde CDN, y ya tienen un mapa interactivo. Sin instalar nada, sin compilar, sin frameworks."

```html
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

> "Esta línea descarga Leaflet desde un CDN. Son 42 kilobytes. Con eso hacen un mapa interactivo completo."

Mostrar el mapa vacío con OSM. Los estudiantes copian el código y lo abren en su navegador.

### Paso 2 — Capas WMS (15 min)

**TUS DATOS:** departamentos, ríos, capitales, áreas protegidas de Bolivia.
**DATOS ESTUDIANTES:** capas de Natural Earth que publicaron en los módulos anteriores.

> "Ahora la magia. Esta línea conecta Leaflet a SU GeoServer:"

```javascript
L.tileLayer.wms('http://localhost:8080/geoserver/mi_proyecto/wms', {
  layers: 'mi_proyecto:paises', format: 'image/png', transparent: true
}).addTo(mapa);
```

> "¿Ven sus datos sobre el mapa? El mismo servicio WMS que probaron en el navegador en el Módulo 4, ahora se ve en un visor web profesional."

**Momento wow:** Cuando ven SUS datos, con SUS estilos SLD, sobre un mapa OSM interactivo.

### Paso 3 — Panel de capas + Leyenda (10 min)

> "En QGIS tienen un panel de capas. Aquí hacen lo mismo con HTML y un poco de JavaScript."

Mostrar cómo los checkboxes controlan las capas. Mostrar la leyenda que se actualiza dinámicamente.

### Paso 4 — GetFeatureInfo (15 min)

> "¿Recuerdan GetFeatureInfo del Módulo 4? Ahora lo conectamos al clic del usuario. Clic en un departamento → popup con los atributos."

**Construir paso a paso:**
1. `mapa.on('click', function(e) {...})`
2. Construir la URL GetFeatureInfo
3. `fetch(url)` para pedir los datos
4. Parsear JSON y construir HTML del popup
5. `L.popup().setLatLng(e.latlng).setContent(html).openOn(mapa)`

> "Acaban de construir lo que hace QGIS con 'Identificar features', pero en la web. Cualquier persona con un navegador puede ver estos datos."

### Paso 5 — Filtro CQL (10 min)

> "¿Recuerdan CQL del Módulo 4? Ahora el USUARIO puede escribir el filtro desde la interfaz."

Mostrar `setParams({ CQL_FILTER: cql })` y cómo Leaflet recarga la capa.

> "Escriban `nombre='La Paz'` y presionen Filtrar... Solo La Paz en el mapa. Escriban `poblacion>500000`... Solo departamentos grandes. Todo sin tocar GeoServer."

### Parte 2 — Demo Flutter (15 min)

> "Todo lo que hicimos en Leaflet, una app Flutter lo hace con las MISMAS URLs. Miren..."

**Si tienes Flutter instalado y un teléfono a mano:**
1. Muestra la app corriendo
2. Muestra el Drawer con switches de capas
3. Destaca que la URL WMS es IDÉNTICA a la de Leaflet

**Si NO tienes Flutter:**
1. Muestra el código lado a lado con Leaflet
2. Muestra la tabla comparativa
3. Explica que la IP debe ser la local (no localhost)

> "No necesitan saber Flutter para entender esto. El punto es: publicaron datos UNA vez en GeoServer y los están consumiendo desde web, desde móvil, desde QGIS. Un dato, una fuente, múltiples presentaciones."

**Si algún estudiante está interesado en Flutter:**

> "Tenemos un curso completo de Flutter GIS donde construyen una app Android desde cero que se conecta a GeoServer, captura GPS en campo, y exporta datos a QGIS. Pregúntenme después."

### Cierre (10 min)

> "Tienen un visor web funcional consumiendo su propio GeoServer. El próximo paso es que esto sea accesible desde FUERA de su red local. En la última clase hacemos deploy en Google Cloud: su GeoServer queda accesible desde cualquier parte del mundo con una URL pública."

---

## ⚠️ CORS — Configurar ANTES de la clase

**Sin CORS, el visor web NO puede conectarse a GeoServer.** El navegador bloquea las peticiones.

### Habilitar CORS en GeoServer (Tomcat)

Editar `webapps/geoserver/WEB-INF/web.xml` y agregar ANTES de `</web-app>`:

```xml
<filter>
  <filter-name>CorsFilter</filter-name>
  <filter-class>org.apache.catalina.filters.CorsFilter</filter-class>
  <init-param>
    <param-name>cors.allowed.origins</param-name>
    <param-value>*</param-value>
  </init-param>
  <init-param>
    <param-name>cors.allowed.methods</param-name>
    <param-value>GET,POST,HEAD,OPTIONS,PUT</param-value>
  </init-param>
  <init-param>
    <param-name>cors.allowed.headers</param-name>
    <param-value>Content-Type,X-Requested-With,Accept,Origin,Access-Control-Request-Method,Access-Control-Request-Headers,Authorization</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>CorsFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

Reiniciar Tomcat. **Verifica que CORS funciona ANTES de la clase.** Si falla en clase, pierdes 15 minutos debuggeando.

> **Tip:** Envía esta configuración a los estudiantes por el grupo de WhatsApp ANTES de la clase para que lo configuren.

---

## ⚠️ Errores Comunes

| Problema | Solución |
|----------|----------|
| Capa WMS no aparece | CORS no habilitado. Verificar consola F12 |
| CORS error | Configurar filtro CORS en web.xml de GeoServer |
| GetFeatureInfo vacío | Capa sin Queryable activado |
| Flutter no conecta | Usar IP local (192.168.x.x), no localhost |
| Popup vacío | info_format debe ser application/json |
| setParams no funciona | La variable de la capa debe ser la correcta |
| Leyenda no carga | CORS bloqueando la imagen. Verificar consola |

## 💡 Tips

1. **Configura CORS antes de la clase.** Es el bloqueador #1. Sin CORS, nada funciona en el visor web.
2. **Construye el visor paso a paso EN VIVO.** Cada bloque de código agrega una funcionalidad visible.
3. **El momento wow** es cuando ven SUS datos (publicados en módulos anteriores) en un visor web profesional.
4. **Flutter es demo, no clase.** Muestra que funciona y compara con Leaflet. No intentes enseñar Flutter en 15 minutos.
5. **Si sobra tiempo:** Actividad B (WFS en Leaflet con GeoJSON) es la más impactante — ven los datos como vectores interactivos.
6. **Si falta tiempo:** Saltar Flutter (Parte 2) y enfocarse en el visor Leaflet completo. Flutter es demo, no core.