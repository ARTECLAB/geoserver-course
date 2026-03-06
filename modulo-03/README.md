# Módulo 3 · Simbología & Estilos SLD 🎨

> **Clase 4 · 2 horas**
> Tus mapas dejan de verse genéricos. Aprendes a diseñar estilos profesionales con SLD: colores por categoría, grosores proporcionales, etiquetas inteligentes y reglas basadas en atributos.

---

## 🎯 Objetivos

- Entender la estructura y sintaxis de SLD
- Crear estilos para puntos, líneas y polígonos
- Aplicar reglas basadas en atributos (clasificación temática)
- Configurar etiquetas (labels) con posicionamiento y halo
- Usar escalas para mostrar/ocultar elementos según zoom
- Editar SLD directamente en GeoServer y ver resultados en tiempo real

---

## Parte 1 — ¿Qué es SLD y por qué importa? (10 min)

### Concepto

SLD (Styled Layer Descriptor) es el estándar OGC para definir cómo se visualizan los datos geográficos. Es un archivo XML que le dice a GeoServer: "los polígonos de esta capa pintálos de verde con borde negro" o "los puntos con población > 100,000 hazlos rojos y más grandes".

### Analogía con QGIS

En QGIS cambias la simbología haciendo clic derecho → Propiedades → Simbología. Eliges colores, grosores, clasificaciones. **SLD es eso mismo pero escrito como XML.** La ventaja: es portátil, versionable y se aplica del lado del servidor.

### ¿Dónde se edita?

En GeoServer: **Styles** → seleccionar estilo → editor de texto integrado.

GeoServer tiene un editor con:
- **Validación** — te dice si el XML tiene errores
- **Vista previa** — ves el resultado sin salir del editor
- **Leyenda generada** — crea la leyenda automáticamente desde las reglas

---

## Parte 2 — Estructura base de un SLD (15 min)

### Anatomía de un SLD

```xml
<?xml version="1.0" encoding="UTF-8"?>
<StyledLayerDescriptor version="1.0.0"
  xsi:schemaLocation="http://www.opengis.net/sld StyledLayerDescriptor.xsd"
  xmlns="http://www.opengis.net/sld"
  xmlns:ogc="http://www.opengis.net/ogc"
  xmlns:xlink="http://www.w3.org/1999/xlink"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

  <NamedLayer>
    <Name>mi_estilo</Name>
    <UserStyle>
      <Title>Mi Estilo Personalizado</Title>

      <FeatureTypeStyle>
        <Rule>
          <Name>regla_1</Name>
          <Title>Descripción de la regla</Title>

          <!-- Aquí va el simbolizador -->

        </Rule>
      </FeatureTypeStyle>

    </UserStyle>
  </NamedLayer>
</StyledLayerDescriptor>
```

### La jerarquía SLD explicada

```
StyledLayerDescriptor          ← Contenedor raíz
  └── NamedLayer               ← La capa a la que aplica
       └── UserStyle           ← El estilo definido por el usuario
            └── FeatureTypeStyle  ← Grupo de reglas
                 └── Rule         ← UNA regla de visualización
                      ├── Filter  ← Condición (opcional)
                      └── Symbolizer ← Cómo dibujar
```

> **Lo que cambia entre estilos son las Rules y los Symbolizers.** El envoltorio XML siempre es igual — cópialo y no lo toques.

---

## Parte 3 — Estilos para Polígonos (20 min)

### 3.1 — Polígono simple: Departamentos de Bolivia

```xml
<Rule>
  <Name>departamentos</Name>
  <Title>Departamentos</Title>
  <PolygonSymbolizer>
    <Fill>
      <CssParameter name="fill">#2d8659</CssParameter>
      <CssParameter name="fill-opacity">0.6</CssParameter>
    </Fill>
    <Stroke>
      <CssParameter name="stroke">#1a5c3a</CssParameter>
      <CssParameter name="stroke-width">1.5</CssParameter>
    </Stroke>
  </PolygonSymbolizer>
</Rule>
```

### 3.2 — Clasificación temática: Un color por departamento

```xml
<!-- Regla para La Paz -->
<Rule>
  <Name>la_paz</Name>
  <Title>La Paz</Title>
  <ogc:Filter>
    <ogc:PropertyIsEqualTo>
      <ogc:PropertyName>nombre</ogc:PropertyName>
      <ogc:Literal>La Paz</ogc:Literal>
    </ogc:PropertyIsEqualTo>
  </ogc:Filter>
  <PolygonSymbolizer>
    <Fill>
      <CssParameter name="fill">#3b82f6</CssParameter>
      <CssParameter name="fill-opacity">0.7</CssParameter>
    </Fill>
    <Stroke>
      <CssParameter name="stroke">#1e40af</CssParameter>
      <CssParameter name="stroke-width">1</CssParameter>
    </Stroke>
  </PolygonSymbolizer>
</Rule>

<!-- Regla para Cochabamba -->
<Rule>
  <Name>cochabamba</Name>
  <Title>Cochabamba</Title>
  <ogc:Filter>
    <ogc:PropertyIsEqualTo>
      <ogc:PropertyName>nombre</ogc:PropertyName>
      <ogc:Literal>Cochabamba</ogc:Literal>
    </ogc:PropertyIsEqualTo>
  </ogc:Filter>
  <PolygonSymbolizer>
    <Fill>
      <CssParameter name="fill">#ef4444</CssParameter>
      <CssParameter name="fill-opacity">0.7</CssParameter>
    </Fill>
    <Stroke>
      <CssParameter name="stroke">#991b1b</CssParameter>
      <CssParameter name="stroke-width">1</CssParameter>
    </Stroke>
  </PolygonSymbolizer>
</Rule>

<!-- Regla por defecto (el resto) -->
<Rule>
  <Name>otros</Name>
  <Title>Otros departamentos</Title>
  <ElseFilter/>
  <PolygonSymbolizer>
    <Fill>
      <CssParameter name="fill">#94a3b8</CssParameter>
      <CssParameter name="fill-opacity">0.5</CssParameter>
    </Fill>
    <Stroke>
      <CssParameter name="stroke">#64748b</CssParameter>
      <CssParameter name="stroke-width">0.5</CssParameter>
    </Stroke>
  </PolygonSymbolizer>
</Rule>
```

### 3.3 — Clasificación por rango numérico: Población

```xml
<!-- Población alta > 1,000,000 -->
<Rule>
  <Name>pob_alta</Name>
  <Title>Población > 1,000,000</Title>
  <ogc:Filter>
    <ogc:PropertyIsGreaterThan>
      <ogc:PropertyName>poblacion</ogc:PropertyName>
      <ogc:Literal>1000000</ogc:Literal>
    </ogc:PropertyIsGreaterThan>
  </ogc:Filter>
  <PolygonSymbolizer>
    <Fill><CssParameter name="fill">#dc2626</CssParameter></Fill>
    <Stroke><CssParameter name="stroke">#7f1d1d</CssParameter></Stroke>
  </PolygonSymbolizer>
</Rule>

<!-- Población media 200,000 - 1,000,000 -->
<Rule>
  <Name>pob_media</Name>
  <Title>Población 200,000 - 1,000,000</Title>
  <ogc:Filter>
    <ogc:And>
      <ogc:PropertyIsGreaterThanOrEqualTo>
        <ogc:PropertyName>poblacion</ogc:PropertyName>
        <ogc:Literal>200000</ogc:Literal>
      </ogc:PropertyIsGreaterThanOrEqualTo>
      <ogc:PropertyIsLessThanOrEqualTo>
        <ogc:PropertyName>poblacion</ogc:PropertyName>
        <ogc:Literal>1000000</ogc:Literal>
      </ogc:PropertyIsLessThanOrEqualTo>
    </ogc:And>
  </ogc:Filter>
  <PolygonSymbolizer>
    <Fill><CssParameter name="fill">#f59e0b</CssParameter></Fill>
    <Stroke><CssParameter name="stroke">#92400e</CssParameter></Stroke>
  </PolygonSymbolizer>
</Rule>

<!-- Población baja < 200,000 -->
<Rule>
  <Name>pob_baja</Name>
  <Title>Población < 200,000</Title>
  <ElseFilter/>
  <PolygonSymbolizer>
    <Fill><CssParameter name="fill">#22c55e</CssParameter></Fill>
    <Stroke><CssParameter name="stroke">#166534</CssParameter></Stroke>
  </PolygonSymbolizer>
</Rule>
```

---

## Parte 4 — Estilos para Líneas (15 min)

### 4.1 — Ríos de Bolivia con grosor por importancia

```xml
<!-- Ríos principales (longitud > 500 km) -->
<Rule>
  <Name>rio_principal</Name>
  <Title>Ríos principales (> 500 km)</Title>
  <ogc:Filter>
    <ogc:PropertyIsGreaterThan>
      <ogc:PropertyName>longitud_km</ogc:PropertyName>
      <ogc:Literal>500</ogc:Literal>
    </ogc:PropertyIsGreaterThan>
  </ogc:Filter>
  <LineSymbolizer>
    <Stroke>
      <CssParameter name="stroke">#3b82f6</CssParameter>
      <CssParameter name="stroke-width">3</CssParameter>
      <CssParameter name="stroke-linecap">round</CssParameter>
    </Stroke>
  </LineSymbolizer>
</Rule>

<!-- Ríos secundarios (100 - 500 km) -->
<Rule>
  <Name>rio_secundario</Name>
  <Title>Ríos secundarios (100-500 km)</Title>
  <ogc:Filter>
    <ogc:And>
      <ogc:PropertyIsGreaterThanOrEqualTo>
        <ogc:PropertyName>longitud_km</ogc:PropertyName>
        <ogc:Literal>100</ogc:Literal>
      </ogc:PropertyIsGreaterThanOrEqualTo>
      <ogc:PropertyIsLessThan>
        <ogc:PropertyName>longitud_km</ogc:PropertyName>
        <ogc:Literal>500</ogc:Literal>
      </ogc:PropertyIsLessThan>
    </ogc:And>
  </ogc:Filter>
  <LineSymbolizer>
    <Stroke>
      <CssParameter name="stroke">#60a5fa</CssParameter>
      <CssParameter name="stroke-width">1.5</CssParameter>
    </Stroke>
  </LineSymbolizer>
</Rule>

<!-- Ríos menores -->
<Rule>
  <Name>rio_menor</Name>
  <Title>Ríos menores (< 100 km)</Title>
  <ElseFilter/>
  <LineSymbolizer>
    <Stroke>
      <CssParameter name="stroke">#93c5fd</CssParameter>
      <CssParameter name="stroke-width">0.7</CssParameter>
      <CssParameter name="stroke-dasharray">4 3</CssParameter>
    </Stroke>
  </LineSymbolizer>
</Rule>
```

### 4.2 — Carreteras con doble línea (efecto casing)

```xml
<!-- Casing (línea gruesa de fondo) -->
<Rule>
  <Name>carretera_casing</Name>
  <LineSymbolizer>
    <Stroke>
      <CssParameter name="stroke">#374151</CssParameter>
      <CssParameter name="stroke-width">5</CssParameter>
      <CssParameter name="stroke-linecap">round</CssParameter>
      <CssParameter name="stroke-linejoin">round</CssParameter>
    </Stroke>
  </LineSymbolizer>
</Rule>

<!-- Relleno (línea más delgada encima) -->
<Rule>
  <Name>carretera_fill</Name>
  <LineSymbolizer>
    <Stroke>
      <CssParameter name="stroke">#fbbf24</CssParameter>
      <CssParameter name="stroke-width">3</CssParameter>
      <CssParameter name="stroke-linecap">round</CssParameter>
      <CssParameter name="stroke-linejoin">round</CssParameter>
    </Stroke>
  </LineSymbolizer>
</Rule>
```

> **Tip:** El efecto "casing" se logra con dos reglas: una línea gruesa oscura de fondo y una línea más delgada clara encima. Es como el borde de las carreteras en Google Maps.

---

## Parte 5 — Estilos para Puntos (15 min)

### 5.1 — Capitales de Bolivia con tamaño por población

```xml
<!-- Ciudades grandes (> 500,000) -->
<Rule>
  <Name>ciudad_grande</Name>
  <Title>Ciudad grande (> 500,000 hab)</Title>
  <ogc:Filter>
    <ogc:PropertyIsGreaterThan>
      <ogc:PropertyName>poblacion</ogc:PropertyName>
      <ogc:Literal>500000</ogc:Literal>
    </ogc:PropertyIsGreaterThan>
  </ogc:Filter>
  <PointSymbolizer>
    <Graphic>
      <Mark>
        <WellKnownName>circle</WellKnownName>
        <Fill><CssParameter name="fill">#dc2626</CssParameter></Fill>
        <Stroke>
          <CssParameter name="stroke">#ffffff</CssParameter>
          <CssParameter name="stroke-width">1.5</CssParameter>
        </Stroke>
      </Mark>
      <Size>16</Size>
    </Graphic>
  </PointSymbolizer>
</Rule>

<!-- Ciudades medianas (100,000 - 500,000) -->
<Rule>
  <Name>ciudad_mediana</Name>
  <Title>Ciudad mediana (100,000 - 500,000)</Title>
  <ogc:Filter>
    <ogc:And>
      <ogc:PropertyIsGreaterThanOrEqualTo>
        <ogc:PropertyName>poblacion</ogc:PropertyName>
        <ogc:Literal>100000</ogc:Literal>
      </ogc:PropertyIsGreaterThanOrEqualTo>
      <ogc:PropertyIsLessThanOrEqualTo>
        <ogc:PropertyName>poblacion</ogc:PropertyName>
        <ogc:Literal>500000</ogc:Literal>
      </ogc:PropertyIsLessThanOrEqualTo>
    </ogc:And>
  </ogc:Filter>
  <PointSymbolizer>
    <Graphic>
      <Mark>
        <WellKnownName>circle</WellKnownName>
        <Fill><CssParameter name="fill">#f59e0b</CssParameter></Fill>
        <Stroke>
          <CssParameter name="stroke">#ffffff</CssParameter>
          <CssParameter name="stroke-width">1</CssParameter>
        </Stroke>
      </Mark>
      <Size>11</Size>
    </Graphic>
  </PointSymbolizer>
</Rule>

<!-- Ciudades pequeñas -->
<Rule>
  <Name>ciudad_pequena</Name>
  <Title>Ciudad pequeña (< 100,000)</Title>
  <ElseFilter/>
  <PointSymbolizer>
    <Graphic>
      <Mark>
        <WellKnownName>circle</WellKnownName>
        <Fill><CssParameter name="fill">#22c55e</CssParameter></Fill>
        <Stroke>
          <CssParameter name="stroke">#ffffff</CssParameter>
          <CssParameter name="stroke-width">0.5</CssParameter>
        </Stroke>
      </Mark>
      <Size>7</Size>
    </Graphic>
  </PointSymbolizer>
</Rule>
```

### Formas disponibles (WellKnownName)

| Forma | Nombre | Uso típico |
|-------|--------|------------|
| ● | `circle` | Ciudades, puntos genéricos |
| ■ | `square` | Estaciones, edificios |
| ▲ | `triangle` | Alertas, picos |
| ★ | `star` | Puntos de interés, destacados |
| ✕ | `cross` | Muestras, puntos de control |
| ✕ | `x` | Marcadores, cruces |

---

## Parte 6 — Etiquetas (Labels) (15 min)

### 6.1 — Nombres de departamentos con halo

```xml
<Rule>
  <Name>etiquetas</Name>
  <TextSymbolizer>
    <Label>
      <ogc:PropertyName>nombre</ogc:PropertyName>
    </Label>
    <Font>
      <CssParameter name="font-family">Arial</CssParameter>
      <CssParameter name="font-size">12</CssParameter>
      <CssParameter name="font-weight">bold</CssParameter>
    </Font>
    <Fill>
      <CssParameter name="fill">#1e293b</CssParameter>
    </Fill>
    <!-- Halo blanco para legibilidad -->
    <Halo>
      <Radius>2</Radius>
      <Fill>
        <CssParameter name="fill">#ffffff</CssParameter>
      </Fill>
    </Halo>
    <!-- Centrar sobre el polígono -->
    <LabelPlacement>
      <PointPlacement>
        <AnchorPoint>
          <AnchorPointX>0.5</AnchorPointX>
          <AnchorPointY>0.5</AnchorPointY>
        </AnchorPoint>
      </PointPlacement>
    </LabelPlacement>
    <!-- Evitar superposición -->
    <VendorOption name="conflictResolution">true</VendorOption>
    <VendorOption name="autoWrap">80</VendorOption>
    <VendorOption name="goodnessOfFit">0.3</VendorOption>
  </TextSymbolizer>
</Rule>
```

### 6.2 — Etiquetas para ríos (siguiendo la línea)

```xml
<TextSymbolizer>
  <Label>
    <ogc:PropertyName>nombre</ogc:PropertyName>
  </Label>
  <Font>
    <CssParameter name="font-family">Arial</CssParameter>
    <CssParameter name="font-size">10</CssParameter>
    <CssParameter name="font-style">italic</CssParameter>
  </Font>
  <Fill>
    <CssParameter name="fill">#1d4ed8</CssParameter>
  </Fill>
  <Halo>
    <Radius>1.5</Radius>
    <Fill><CssParameter name="fill">#ffffff</CssParameter></Fill>
  </Halo>
  <LabelPlacement>
    <LinePlacement>
      <PerpendicularOffset>8</PerpendicularOffset>
    </LinePlacement>
  </LabelPlacement>
  <VendorOption name="followLine">true</VendorOption>
  <VendorOption name="maxAngleDelta">30</VendorOption>
  <VendorOption name="repeat">300</VendorOption>
</TextSymbolizer>
```

> **Tip:** `followLine` hace que la etiqueta siga la curva del río. `repeat` repite la etiqueta cada 300 píxeles a lo largo de la línea. `maxAngleDelta` evita que el texto se doble demasiado en curvas cerradas.

---

## Parte 7 — Escalas: Mostrar/ocultar por zoom (10 min)

```xml
<!-- Solo mostrar etiquetas cuando el zoom es cercano -->
<Rule>
  <Name>etiquetas_zoom_cercano</Name>
  <MinScaleDenominator>1</MinScaleDenominator>
  <MaxScaleDenominator>500000</MaxScaleDenominator>
  <TextSymbolizer>
    <!-- ... etiqueta ... -->
  </TextSymbolizer>
</Rule>

<!-- Línea gruesa en zoom cercano -->
<Rule>
  <MinScaleDenominator>1</MinScaleDenominator>
  <MaxScaleDenominator>1000000</MaxScaleDenominator>
  <LineSymbolizer>
    <Stroke>
      <CssParameter name="stroke-width">3</CssParameter>
    </Stroke>
  </LineSymbolizer>
</Rule>

<!-- Línea delgada en zoom lejano -->
<Rule>
  <MinScaleDenominator>1000000</MinScaleDenominator>
  <MaxScaleDenominator>50000000</MaxScaleDenominator>
  <LineSymbolizer>
    <Stroke>
      <CssParameter name="stroke-width">0.5</CssParameter>
    </Stroke>
  </LineSymbolizer>
</Rule>
```

### Referencia de escalas

| Escala aprox. | ¿Qué ves? | Denominador |
|--------------|-----------|-------------|
| País completo | Forma del país | 10,000,000+ |
| Departamento | Límites departamentales | 1,000,000 - 5,000,000 |
| Ciudad | Barrios, calles principales | 50,000 - 500,000 |
| Barrio | Calles, manzanas | 5,000 - 50,000 |
| Detalle | Edificios, postes | 1 - 5,000 |

---

## Actividades Extra — Para llenar tiempo o profundizar

### Actividad A — Estilo mapa coroplético completo (15 min)

Crear un estilo SLD completo para departamentos con:
- 3 rangos de color por superficie (pequeño/mediano/grande)
- Etiquetas con nombre del departamento
- Borde de 2px en zoom cercano, 0.5px en zoom lejano
- Leyenda que muestre los 3 rangos

### Actividad B — "Decodifica este SLD" (10 min)

Mostrar un SLD sin explicar y que los estudiantes digan qué hace:
- ¿Qué tipo de geometría estiliza?
- ¿Cuántas reglas tiene?
- ¿Qué filtros aplica?
- ¿Qué colores usa?

### Actividad C — Exportar estilo de QGIS a SLD (10 min)

1. Abrir una capa en QGIS
2. Configurar simbología con clasificación
3. Exportar como SLD: Propiedades → Simbología → Style → Save Style → SLD
4. Importar ese SLD en GeoServer

> **Tip:** El SLD que exporta QGIS funciona en GeoServer pero suele ser verboso. Muestra cómo limpiarlo.

### Actividad D — Paletas de colores profesionales (5 min)

Recursos para elegir colores:
- **ColorBrewer** (colorbrewer2.org) — Paletas diseñadas para cartografía
- **Coolors** (coolors.co) — Generador de paletas
- **Carto Colors** (carto.com/carto-colors/) — Paletas optimizadas para mapas

> **Regla de cartografía:** Máximo 7 clases en una clasificación. Más de 7 y el ojo humano no distingue los colores.

### Actividad E — Desafío en clase: "Mejora este mapa" (15 min)

Dar un SLD básico (todo gris, sin etiquetas, sin clasificación) y pedir que lo mejoren en 10 minutos. Luego cada uno muestra su resultado. Votar el mejor.

---

### 💻 Práctica 04

1. Crea un estilo SLD para polígonos con clasificación por un atributo (al menos 3 clases de color)
2. Crea un estilo SLD para líneas con grosor diferente según un atributo
3. Crea un estilo SLD para puntos con tamaño proporcional a un valor numérico
4. Agrega etiquetas a al menos una capa (con halo para legibilidad)
5. Aplica control de escala: etiquetas visibles solo en zoom cercano
6. Asigna tus estilos como Default Style de cada capa
7. Verifica en Layer Preview que todo se ve correcto
8. Verifica la leyenda generada: `http://localhost:8080/geoserver/wms?service=WMS&request=GetLegendGraphic&layer=tu_workspace:tu_capa&format=image/png`

### 🚀 Reto Extra — Estilo de carretera con casing

Crea un estilo de doble línea para carreteras: línea gruesa oscura de fondo + línea delgada amarilla encima. Que el grosor cambie según el tipo de carretera (nacional, departamental, vecinal).

### ✅ Checklist Clase 04

- [ ] Entiendo la estructura básica de un SLD
- [ ] Estilo de polígono con clasificación temática
- [ ] Estilo de línea con grosor por atributo
- [ ] Estilo de punto con tamaño proporcional
- [ ] Etiquetas con halo funcionando
- [ ] Control de escala aplicado
- [ ] Estilos asignados como Default Style
- [ ] Leyenda generada correctamente

---

## 📝 Errores comunes

1. **SLD no valida** → Error de sintaxis XML. Verificar cierre de tags. El editor de GeoServer marca la línea del error.
2. **Etiquetas no aparecen** → Verificar que el PropertyName coincide exactamente con el nombre del campo (case-sensitive en PostGIS)
3. **Colores no se ven** → Verificar formato hex: `#ff0000` (con #). Sin # no funciona.
4. **Filtro no coincide** → PropertyName debe coincidir exactamente con el nombre del atributo en el dato. Mayúsculas importan en PostGIS.
5. **Leyenda no muestra clases** → Cada Rule debe tener `<Name>` y `<Title>`. Sin Title, la leyenda queda vacía.
6. **Etiquetas superpuestas** → Agregar `<VendorOption name="conflictResolution">true</VendorOption>`
7. **Escala no funciona** → MinScale es el zoom MÁS cercano, MaxScale el MÁS lejano. Es contraintuitivo.
8. **Estilo exportado de QGIS no funciona** → QGIS agrega namespaces y versiones que GeoServer no soporta. Limpiar el XML.

## 💡 Tips de producción

- **Usa ColorBrewer** para paletas de colores. Los cartógrafos ya hicieron el trabajo duro de elegir colores que se distinguen bien.
- **Siempre pon halo en las etiquetas.** Sin halo, las letras se pierden sobre fondos de colores.
- **Las reglas se evalúan en orden.** La primera que coincide gana. Pon las más específicas primero y `ElseFilter` al final.
- **`conflictResolution` es obligatorio** para etiquetas. Sin esto, se superponen y el mapa es ilegible.
- **Máximo 5-7 clases** en una clasificación temática. Más de eso y el usuario no distingue los colores.
- **Guarda tus SLD en archivos .sld** fuera de GeoServer como backup. Si GeoServer se corrompe, no pierdes tus estilos.