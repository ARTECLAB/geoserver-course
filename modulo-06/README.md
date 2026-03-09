# Módulo 6 · Visor Web & App Móvil 🌐📱

> **Clase 7 · 2 horas**
> GeoServer se conecta con el mundo real. Un visor web Leaflet con tus datos de Bolivia y una demo de integración Flutter. Un servidor, múltiples plataformas.

---

## 🎯 Objetivos

- Construir un visor web completo con Leaflet consumiendo WMS/WFS de GeoServer
- Implementar selector de capas, popups con GetFeatureInfo y filtros CQL
- Mostrar la leyenda WMS en el visor
- Demostrar cómo Flutter consume los mismos servicios
- Entender la arquitectura cliente-servidor aplicada a GIS

---

## Parte 1 — Visor Web Leaflet con datos de Bolivia (60 min)

### 1.1 — Visor base (5 min)

```html
<!DOCTYPE html>
<html>
<head>
  <title>Visor GeoServer — Bolivia</title>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width">
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: Arial, sans-serif; }
    #mapa { width: 100vw; height: 100vh; }

    .panel {
      position: absolute; top: 10px; right: 10px; z-index: 1000;
      background: white; padding: 15px; border-radius: 8px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.2); max-width: 280px;
      max-height: 90vh; overflow-y: auto;
    }
    .panel h3 { margin: 0 0 10px; font-size: 14px; color: #333; }
    .panel label { display: block; margin: 6px 0; font-size: 13px; cursor: pointer; }
    .panel input[type=checkbox] { margin-right: 6px; }
    .panel hr { margin: 10px 0; border: none; border-top: 1px solid #eee; }
    .panel img { max-width: 100%; margin: 5px 0; }

    .filtro-panel {
      position: absolute; top: 10px; left: 60px; z-index: 1000;
      background: white; padding: 10px 14px; border-radius: 8px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.2);
    }
    .filtro-panel input { width: 280px; padding: 6px 10px; border: 1px solid #ccc; border-radius: 4px; }
    .filtro-panel button { padding: 6px 12px; margin-left: 4px; cursor: pointer; }
  </style>
</head>
<body>

  <!-- Panel de filtro CQL -->
  <div class="filtro-panel">
    <input id="filtro" placeholder="CQL: nombre='La Paz'">
    <button onclick="aplicarFiltro()">Filtrar</button>
    <button onclick="limpiarFiltro()">✕</button>
  </div>

  <!-- Panel de capas + leyenda -->
  <div class="panel">
    <h3>🗺️ Capas</h3>
    <label><input type="checkbox" id="chk_deptos" checked onchange="toggleCapa('deptos')"> Departamentos</label>
    <label><input type="checkbox" id="chk_rios" onchange="toggleCapa('rios')"> Ríos principales</label>
    <label><input type="checkbox" id="chk_capitales" onchange="toggleCapa('capitales')"> Capitales</label>
    <label><input type="checkbox" id="chk_areas" onchange="toggleCapa('areas')"> Áreas protegidas</label>
    <hr>
    <h3>📋 Leyenda</h3>
    <div id="leyenda"></div>
  </div>

  <div id="mapa"></div>

  <script>
    // ── Configuración ──
    var GEOSERVER = 'http://localhost:8080/geoserver/datos_bolivia/wms';
    var WS = 'datos_bolivia';

    // ── Mapa centrado en Bolivia ──
    var mapa = L.map('mapa').setView([-16.5, -64.5], 6);

    // Mapa base
    L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; OpenStreetMap'
    }).addTo(mapa);

    // ── Capas WMS ──
    var capas = {};

    capas.deptos = L.tileLayer.wms(GEOSERVER, {
      layers: WS + ':departamentos',
      format: 'image/png', transparent: true, version: '1.1.1'
    }).addTo(mapa);

    capas.rios = L.tileLayer.wms(GEOSERVER, {
      layers: WS + ':rios_principales',
      format: 'image/png', transparent: true, version: '1.1.1'
    });

    capas.capitales = L.tileLayer.wms(GEOSERVER, {
      layers: WS + ':capitales',
      format: 'image/png', transparent: true, version: '1.1.1'
    });

    capas.areas = L.tileLayer.wms(GEOSERVER, {
      layers: WS + ':areas_protegidas',
      format: 'image/png', transparent: true, version: '1.1.1'
    });

    // ── Toggle de capas ──
    function toggleCapa(nombre) {
      var chk = document.getElementById('chk_' + nombre);
      if (chk.checked) {
        capas[nombre].addTo(mapa);
      } else {
        mapa.removeLayer(capas[nombre]);
      }
      actualizarLeyenda();
    }

    // ── Leyenda dinámica ──
    function actualizarLeyenda() {
      var html = '';
      var nombres = { deptos: 'departamentos', rios: 'rios_principales', capitales: 'capitales', areas: 'areas_protegidas' };
      for (var key in capas) {
        if (mapa.hasLayer(capas[key])) {
          html += '<img src="' + GEOSERVER + '?service=WMS&version=1.1.1'
            + '&request=GetLegendGraphic&layer=' + WS + ':' + nombres[key]
            + '&format=image/png&LEGEND_OPTIONS=fontName:Arial;fontSize:11"'
            + ' alt="' + key + '"><br>';
        }
      }
      document.getElementById('leyenda').innerHTML = html || '<em style="color:#999;font-size:12px;">Activa una capa para ver su leyenda</em>';
    }
    actualizarLeyenda();

    // ── GetFeatureInfo al hacer clic ──
    mapa.on('click', function(e) {
      var bounds = mapa.getBounds();
      var size = mapa.getSize();
      var point = mapa.latLngToContainerPoint(e.latlng);

      // Consultar la primera capa activa
      var capaActiva = null;
      var nombreCapa = null;
      var prioridad = ['capitales', 'rios', 'areas', 'deptos'];
      var nombresReales = { deptos: 'departamentos', rios: 'rios_principales', capitales: 'capitales', areas: 'areas_protegidas' };

      for (var i = 0; i < prioridad.length; i++) {
        if (mapa.hasLayer(capas[prioridad[i]])) {
          capaActiva = prioridad[i];
          nombreCapa = nombresReales[prioridad[i]];
          break;
        }
      }
      if (!capaActiva) return;

      var url = GEOSERVER + '?SERVICE=WMS&VERSION=1.1.1&REQUEST=GetFeatureInfo'
        + '&LAYERS=' + WS + ':' + nombreCapa
        + '&QUERY_LAYERS=' + WS + ':' + nombreCapa
        + '&INFO_FORMAT=application/json'
        + '&SRS=EPSG:4326'
        + '&WIDTH=' + size.x + '&HEIGHT=' + size.y
        + '&BBOX=' + bounds.toBBoxString()
        + '&X=' + Math.round(point.x) + '&Y=' + Math.round(point.y)
        + '&FEATURE_COUNT=1';

      fetch(url)
        .then(function(r) { return r.json(); })
        .then(function(data) {
          if (data.features && data.features.length > 0) {
            var props = data.features[0].properties;
            var html = '<div style="max-width:250px;">';
            html += '<b style="font-size:14px;color:#0d9488;">' + nombreCapa + '</b><hr style="margin:5px 0;">';
            for (var k in props) {
              if (k !== 'bbox' && k !== 'geom' && k !== 'the_geom') {
                html += '<b>' + k + ':</b> ' + props[k] + '<br>';
              }
            }
            html += '</div>';
            L.popup().setLatLng(e.latlng).setContent(html).openOn(mapa);
          }
        })
        .catch(function(err) { console.error('GetFeatureInfo error:', err); });
    });

    // ── Filtro CQL ──
    function aplicarFiltro() {
      var cql = document.getElementById('filtro').value;
      if (!cql) return;
      capas.deptos.setParams({ CQL_FILTER: cql });
    }

    function limpiarFiltro() {
      document.getElementById('filtro').value = '';
      capas.deptos.setParams({ CQL_FILTER: null });
    }
  </script>

</body>
</html>
```

> **Tip:** Este visor completo tiene ~130 líneas y funcionalidad de nivel profesional: capas WMS con toggle, leyenda dinámica, GetFeatureInfo con popup, filtro CQL, y panel lateral. Construirlo EN VIVO demuestra el poder de GeoServer + Leaflet.

---

## Parte 2 — Demo Flutter con capas WMS (20 min)

### El concepto — No enseñamos Flutter

Los estudiantes de este curso NO saben Flutter. El objetivo es **demostrar** que el mismo GeoServer sirve a apps móviles. Muestras el código, lo ejecutas si tienes un teléfono a mano, y explicas que las URLs son las mismas.

### Código Flutter completo para demo

```dart
import 'package:flutter/material.dart';
import 'package:flutter_map/flutter_map.dart';
import 'package:latlong2/latlong.dart';

void main() => runApp(const MaterialApp(home: VisorGeoServer()));

class VisorGeoServer extends StatefulWidget {
  const VisorGeoServer({super.key});
  @override
  State<VisorGeoServer> createState() => _VisorGeoServerState();
}

class _VisorGeoServerState extends State<VisorGeoServer> {
  // IMPORTANTE: Reemplaza con la IP de tu PC en la red local
  // (localhost no funciona en Android — necesita la IP real)
  final String geoserverUrl =
      'http://192.168.1.100:8080/geoserver/datos_bolivia/wms?';

  bool _mostrarDeptos = true;
  bool _mostrarRios = false;
  bool _mostrarCapitales = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('GeoServer en Flutter')),
      drawer: Drawer(
        child: ListView(
          padding: const EdgeInsets.all(16),
          children: [
            const Text('Capas WMS', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            SwitchListTile(
              title: const Text('Departamentos'),
              value: _mostrarDeptos,
              onChanged: (v) => setState(() => _mostrarDeptos = v),
            ),
            SwitchListTile(
              title: const Text('Ríos'),
              value: _mostrarRios,
              onChanged: (v) => setState(() => _mostrarRios = v),
            ),
            SwitchListTile(
              title: const Text('Capitales'),
              value: _mostrarCapitales,
              onChanged: (v) => setState(() => _mostrarCapitales = v),
            ),
          ],
        ),
      ),
      body: FlutterMap(
        options: MapOptions(
          initialCenter: const LatLng(-16.5, -64.5),
          initialZoom: 6,
        ),
        children: [
          TileLayer(
            urlTemplate: 'https://tile.openstreetmap.org/{z}/{x}/{y}.png',
          ),
          if (_mostrarDeptos)
            TileLayer(
              wmsOptions: WMSTileLayerOptions(
                baseUrl: geoserverUrl,
                layers: ['datos_bolivia:departamentos'],
                transparent: true, format: 'image/png',
              ),
            ),
          if (_mostrarRios)
            TileLayer(
              wmsOptions: WMSTileLayerOptions(
                baseUrl: geoserverUrl,
                layers: ['datos_bolivia:rios_principales'],
                transparent: true, format: 'image/png',
              ),
            ),
          if (_mostrarCapitales)
            TileLayer(
              wmsOptions: WMSTileLayerOptions(
                baseUrl: geoserverUrl,
                layers: ['datos_bolivia:capitales'],
                transparent: true, format: 'image/png',
              ),
            ),
        ],
      ),
    );
  }
}
```

### ⚠️ Nota sobre localhost vs IP local

Cuando la app Flutter corre en un teléfono Android, `localhost` se refiere al teléfono, NO a tu PC. Para que Flutter se conecte a tu GeoServer local:

1. Averigua la IP de tu PC en la red local: `ipconfig` (Windows) o `ip addr` (Linux)
2. Usa esa IP en la URL: `http://192.168.1.100:8080/geoserver/...`
3. Verifica que tu PC y teléfono están en la misma red WiFi
4. Verifica que el firewall de Windows no bloquea el puerto 8080

---

## Parte 3 — Actividades Extra (si sobra tiempo)

### Actividad A — Múltiples mapas base (10 min)

Agregar selector de mapa base al visor Leaflet:

```javascript
var osm = L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png');
var satelite = L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}');
var oscuro = L.tileLayer('https://basemaps.cartocdn.com/dark_all/{z}/{x}/{y}@2x.png');

var baseMaps = {
  "Calles": osm,
  "Satélite": satelite,
  "Oscuro": oscuro
};

L.control.layers(baseMaps, overlays).addTo(mapa);
osm.addTo(mapa);
```

### Actividad B — WFS en Leaflet con GeoJSON (15 min)

En vez de WMS (imágenes), cargar datos como vectores con WFS:

```javascript
fetch('http://localhost:8080/geoserver/datos_bolivia/wfs?'
  + 'service=WFS&version=2.0.0&request=GetFeature'
  + '&typeName=datos_bolivia:capitales'
  + '&outputFormat=application/json')
  .then(function(r) { return r.json(); })
  .then(function(data) {
    L.geoJSON(data, {
      pointToLayer: function(feature, latlng) {
        return L.circleMarker(latlng, {
          radius: 6, fillColor: '#dc2626',
          color: '#fff', weight: 2, fillOpacity: 0.8
        });
      },
      onEachFeature: function(feature, layer) {
        layer.bindPopup('<b>' + feature.properties.nombre + '</b>');
      }
    }).addTo(mapa);
  });
```

> **¿WMS o WFS en Leaflet?** WMS: el servidor renderiza → recibís imágenes. WFS: recibís datos GeoJSON → Leaflet los renderiza y los puedes estilizar, filtrar y clickear en el cliente. Para pocas features (<1000), WFS es más interactivo. Para muchas, WMS es más eficiente.

### Actividad C — Exportar vista del mapa (5 min)

Agregar botón que abre la URL GetMap con el BBOX actual — genera una imagen PNG del mapa visible:

```javascript
function exportarMapa() {
  var bounds = mapa.getBounds();
  var url = GEOSERVER + '?service=WMS&version=1.1.1&request=GetMap'
    + '&layers=' + WS + ':departamentos,' + WS + ':rios_principales,' + WS + ':capitales'
    + '&srs=EPSG:4326&format=image/png'
    + '&width=1200&height=900'
    + '&bbox=' + bounds.toBBoxString();
  window.open(url, '_blank');
}
```

---

### 💻 Práctica 07

1. Crea el visor web completo con Leaflet: mapa base + al menos 3 capas WMS de tu GeoServer
2. Panel de capas con checkboxes que activan/desactivan capas
3. Leyenda dinámica que se actualiza al cambiar capas
4. GetFeatureInfo con popup al hacer clic
5. Input de filtro CQL funcional
6. (Extra) Agrega selector de mapas base (Calles, Satélite, Oscuro)
7. (Extra) Carga una capa como WFS GeoJSON con L.geoJSON

### ✅ Checklist Clase 07

- [ ] Visor web Leaflet funcional
- [ ] Capas WMS de mi GeoServer visibles
- [ ] Selector de capas con toggle
- [ ] GetFeatureInfo con popup
- [ ] Filtro CQL desde la interfaz
- [ ] Leyenda dinámica
- [ ] Entiendo cómo Flutter usa las mismas URLs

---

## 📝 Errores comunes

1. **Capa WMS no aparece** → Verificar URL del GeoServer, nombre workspace:capa, CORS habilitado
2. **CORS error en el navegador** → GeoServer necesita CORS habilitado. Agregar filtro CORS en `web.xml` del webapp de GeoServer
3. **GetFeatureInfo no devuelve datos** → Capa no tiene Queryable activado. Verificar en Publishing.
4. **Popup vacío** → Verificar que info_format=application/json y que la respuesta tiene features
5. **Flutter no conecta a localhost** → Usar la IP local de la PC (192.168.x.x), no localhost
6. **Leyenda no carga** → Verificar URL de GetLegendGraphic. El estilo debe tener Rules con Title.
7. **setParams no actualiza** → Verificar que la variable de la capa es correcta y que CQL_FILTER es un string

## 💡 Tips de producción

- **CORS es obligatorio** para visores web. Sin CORS, el navegador bloquea las peticiones a GeoServer. Configúralo ANTES de la clase.
- **Leaflet es simple pero potente.** Con ~100 líneas de código tienes un visor profesional. No necesitas frameworks pesados.
- **WMS para visualizar, WFS para interactuar.** Si solo necesitas ver el mapa → WMS. Si necesitas hacer algo con los datos en el cliente → WFS.
- **La leyenda dinámica** es un detalle que distingue visores amateur de profesionales. Que cambie según las capas activas.
- **El código Flutter es para demostración.** Si algún estudiante quiere profundizar, dirigirlo al curso de Flutter GIS.