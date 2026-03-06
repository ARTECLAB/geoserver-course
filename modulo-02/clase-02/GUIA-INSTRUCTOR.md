# 🎓 Guía del Instructor — Clase 02

## Publicación de Datos Vectoriales: Shapefile & Directory Store

**Duración:** 2 horas (120 minutos)
**Módulo:** 2 de 7

---

## ⏱️ Cronograma

| Tiempo | Duración | Sección | Actividad |
|--------|----------|---------|-----------|
| 00:00 | 10 min | Repaso | Verificar práctica 01, resolver dudas de instalación |
| 00:10 | 15 min | Parte 2 | Shapefile Store con configuración detallada |
| 00:25 | 20 min | Parte 3 | Directory Store — múltiples shapefiles |
| 00:45 | 15 min | Parte 4 | Metadatos profesionales |
| 01:00 | 10 min | Descanso | |
| 01:10 | 25 min | Demo | Publicar 3 capas de Bolivia EN VIVO con metadatos |
| 01:35 | 15 min | Práctica | Estudiantes publican sus propios datos |
| 01:50 | 10 min | Cierre | Preview clase 03 (PostGIS) |

---

## 🎬 Desarrollo

### Apertura (10 min)

> "En la clase pasada publicaron UNA capa con la configuración mínima. Hoy vamos a hacer las cosas como profesionales: metadatos completos, múltiples capas, y al final de la clase van a entender por qué el shapefile es limitado — lo que nos lleva a PostGIS en la siguiente clase."

**Verificar rápido:** Pedir a 2-3 estudiantes que muestren su Layer Preview funcionando.

### Parte 2 — Shapefile detallado (15 min)

**TUS DATOS:** departamentos de Bolivia.

> "Voy a publicar los departamentos de Bolivia. Pero esta vez voy a configurar TODO correctamente — como si fuera un servicio que va a entregar a un cliente."

**Muestra EN VIVO la diferencia entre:**
- Name: `departamentos` (técnico, para URLs)
- Title: `Departamentos de Bolivia` (legible, para humanos)
- Abstract: `Límites administrativos de los 9 departamentos...` (descripción)

> "¿Ven la diferencia? El nombre técnico es lo que va en la URL. El título es lo que ve el usuario en QGIS cuando agrega la capa WMS. El abstract es la documentación."

**Momento clave — Reproyección al vuelo:**

> "Mi shapefile está en UTM zona 19S (EPSG:32719) pero quiero servirlo en WGS84 (EPSG:4326). ¿Tengo que reproyectar el archivo? NO. GeoServer lo hace al vuelo. Elijo 'Reproject native to declared' y listo."

### Parte 3 — Directory Store (20 min)

> "Tengo 4 shapefiles de Bolivia en una carpeta. ¿Creo 4 stores? No. Creo UN Directory Store que apunta a la carpeta."

**Construye EN VIVO:**
1. Apuntar a la carpeta con 4 shapefiles
2. Publicar cada capa con metadatos completos
3. Mostrar que al agregar un nuevo .shp a la carpeta, aparece disponible

### Parte 4 — Metadatos (15 min)

**Muestra GetCapabilities EN VIVO:**

```
http://localhost:8080/geoserver/datos_bolivia/wms?service=WMS&version=1.1.1&request=GetCapabilities
```

> "¿Ven ese XML? Es lo que lee QGIS cuando agregan un servicio WMS. Si no ponen título, abstract ni keywords, el usuario ve nombres técnicos sin contexto. Eso es inaceptable en un proyecto profesional."

**Tip de la vida real:**

> "En mi experiencia, he visto proyectos gubernamentales rechazados porque los servicios WMS no tenían metadatos. La norma del SNIT en Bolivia exige metadatos completos. Otros países tienen normas similares. Háganlo desde ahora."

---

## ⚠️ Errores Comunes

| Problema | Solución |
|----------|----------|
| Caracteres raros (ñ, tildes) | Charset del Store = UTF-8 |
| Campos truncados (10 chars) | Limitación del shapefile, mostrar ejemplo |
| Bounding Box vacío o inválido | Siempre hacer Compute from data + Compute from native bounds |
| Directory Store no detecta archivos | Verificar que los 4 archivos (.shp .shx .dbf .prj) están completos |
| Capa reproyectada se ve deformada | Verificar que el SRS nativo es el correcto |

## 💡 Tips

1. **Abre GetCapabilities** en el navegador y busca tus capas — así ven el impacto real de los metadatos
2. **Muestra la limitación de 10 chars** del shapefile en vivo: un campo `superficie_km2` que aparece como `superfici`
3. **Haz la transición a PostGIS:** "¿Ven estas limitaciones? En la próxima clase las eliminamos todas con PostGIS."