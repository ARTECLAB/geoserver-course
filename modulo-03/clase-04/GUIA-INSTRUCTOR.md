# 🎓 Guía del Instructor — Clase 04

## Simbología & Estilos SLD

**Duración:** 2 horas (120 minutos)
**Módulo:** 3 de 7

---

## ⏱️ Cronograma

| Tiempo | Duración | Sección | Actividad |
|--------|----------|---------|-----------|
| 00:00 | 5 min  | Repaso | Verificar práctica Módulo 2 |
| 00:05 | 10 min | Parte 1 | ¿Qué es SLD? Analogía con QGIS |
| 00:15 | 15 min | Parte 2 | Estructura base + editor de GeoServer |
| 00:30 | 20 min | Parte 3 | Polígonos: simple → clasificación → rangos |
| 00:50 | 10 min | Descanso | |
| 01:00 | 15 min | Parte 4-5 | Líneas + Puntos |
| 01:15 | 15 min | Parte 6 | Etiquetas con halo + followLine |
| 01:30 | 10 min | Parte 7 | Escalas |
| 01:40 | 15 min | Extra | Actividad E "Mejora este mapa" o Actividad C (QGIS→SLD) |
| 01:55 | 5 min  | Cierre | Preview Módulo 4 |

---

## 🎬 Desarrollo

### Apertura (10 min)

> "Hasta ahora sus mapas se ven... genéricos. Polígonos grises, líneas negras, puntos rojos. Eso es porque GeoServer usa estilos por defecto que no saben nada de sus datos. Hoy les enseñamos a GeoServer cómo DEBEN verse sus mapas."

**Muestra el antes/después:** Primero Layer Preview con estilo por defecto. Luego la misma capa con un estilo profesional. El contraste es impactante.

> "La diferencia entre un mapa amateur y uno profesional está en la simbología. En 2 horas van a dominarla."

### Parte 2 — Estructura base (15 min)

**NO hagas que escriban el envoltorio XML.** Dales la plantilla y que solo cambien las Rules.

> "Este envoltorio XML siempre es igual. Cópienlo una vez y nunca más lo toquen. Lo que importa son las Rules de adentro — ahí es donde hacen la magia."

**Muestra el editor de GeoServer:**
1. Styles → Add a new style
2. Escribir/pegar SLD
3. Clic en "Validate" → corrige errores
4. Clic en "Preview legend" → ve la leyenda
5. Save

> "El ciclo es: editar → validar → previsualizar → guardar. Pueden iterar rápido."

### Parte 3 — Polígonos (20 min)

**Secuencia de complejidad creciente:**

1. **Polígono simple** (2 min) — Un color para todos. "El Hello World de SLD."

2. **Clasificación por atributo** (8 min) — Un color por departamento. 
   > "Esto es lo que en QGIS llaman 'Categorizado'. Aquí es un `PropertyIsEqualTo` por cada valor."

3. **Clasificación por rango** (8 min) — Colores por población.
   > "Y esto es lo que en QGIS llaman 'Graduado'. Usamos `PropertyIsGreaterThan` y `PropertyIsLessThan`."

**TUS DATOS:** departamentos de Bolivia con colores del Instituto Nacional de Estadística.

**Momento clave:** Cuando el mapa de departamentos pasa de gris a coloreado por nombre.

> "¿Ven eso? Antes era un bloque gris. Ahora cada departamento tiene su color. Y la leyenda se generó SOLA desde las reglas SLD."

### Parte 4-5 — Líneas y Puntos (15 min)

**Líneas:** Ríos con grosor por importancia. Mostrar el efecto casing para carreteras.

> "El truco de la doble línea es simple: una línea gruesa oscura de fondo y una línea delgada clara encima. Es como Google Maps hace las carreteras."

**Puntos:** Capitales con tamaño por población.

> "Los puntos grandes son ciudades grandes. Simple, intuitivo, profesional."

**Formas disponibles:** Mostrar circle, square, triangle, star. Que elijan su favorita.

### Parte 6 — Etiquetas (15 min)

> "Un mapa sin etiquetas es un mapa incompleto. Pero etiquetas mal puestas son peor que no tener ninguna."

**Muestra primero SIN halo:** El texto se pierde sobre los colores.

**Luego CON halo:** El texto es perfectamente legible.

> "¿Ven la diferencia? El halo blanco de 2 píxeles es la diferencia entre 'no se lee' y 'se lee perfecto'. Siempre pongan halo."

**Para ríos:** Mostrar `followLine` — la etiqueta sigue la curva del río.

> "Esto es lo que se ve en los mapas profesionales. El nombre del río sigue su cauce. En QGIS hay una opción similar, pero en SLD controlamos exactamente cómo se comporta."

### Parte 7 — Escalas (10 min)

> "¿Tiene sentido mostrar el nombre de cada ciudad cuando ves todo Bolivia? No. Solo cuando haces zoom. Las escalas controlan eso."

Mostrar cómo las etiquetas aparecen/desaparecen al hacer zoom.

### Actividad Extra — "Mejora este mapa" (15 min)

Si sobra tiempo (o si los estudiantes van rápido):

1. Dar un SLD genérico (todo gris, sin etiquetas)
2. 10 minutos para mejorarlo
3. Cada uno muestra su resultado en Layer Preview
4. Votan el mejor

> "Este ejercicio es divertido y consolida todo lo que aprendieron. No hay respuesta correcta — es creatividad + técnica."

**Si NO sobra tiempo:** Saltar esta actividad y asignarla como parte de la práctica.

### Cierre — Preview Módulo 4

> "Sus mapas ahora se ven profesionales. Pero hasta ahora solo los ven en Layer Preview. En la siguiente clase vamos a dominar los servicios WMS y WFS: cómo configurarlos, cómo hacer consultas dinámicas con filtros CQL, y cómo integrar todo con clientes externos."

---

## ⚠️ Errores Comunes

| Problema | Solución |
|----------|----------|
| SLD no valida | Error de sintaxis XML. Revisar cierre de tags |
| Color no se ve | Verificar formato hex con # (`#ff0000` no `ff0000`) |
| Filtro no coincide | PropertyName case-sensitive en PostGIS |
| Etiquetas superpuestas | Agregar conflictResolution=true |
| Leyenda vacía | Cada Rule necesita `<Title>` |
| Escala confusa | MinScale = zoom cercano, MaxScale = zoom lejano |
| SLD de QGIS no funciona | Limpiar namespaces y versiones incompatibles |

## 💡 Tips

1. **Antes/después** es el arma más potente de esta clase. Muestra el mapa feo → mapa bonito.
2. **No les hagas escribir el envoltorio XML.** Dales la plantilla completa y que solo editen las Rules.
3. **ColorBrewer** (colorbrewer2.org) es tu mejor amigo para elegir paletas. Muéstralo en pantalla.
4. **Ten 3-4 SLD completos** listos para copiar si algún estudiante se traba. No pierdas tiempo debuggeando XML en clase.
5. **El halo en etiquetas** es el tip más práctico de esta clase. Simple y de alto impacto visual.
6. **Si sobra mucho tiempo:** Actividad A (mapa coroplético completo) + Actividad C (exportar de QGIS a SLD)
7. **Si falta tiempo:** Saltar escalas (Parte 7) y asignar como tarea. Las partes 3-6 son más importantes.