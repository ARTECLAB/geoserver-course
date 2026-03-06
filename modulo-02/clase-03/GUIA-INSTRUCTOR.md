# 🎓 Guía del Instructor — Clase 03

## Conexión a PostGIS y Grupos de Capas

**Duración:** 2 horas (120 minutos)
**Módulo:** 2 de 7

---

## ⏱️ Cronograma

| Tiempo | Duración | Sección | Actividad |
|--------|----------|---------|-----------|
| 00:00 | 5 min  | Repaso | Verificar práctica 02 |
| 00:05 | 10 min | Parte 1 | Shapefile vs PostGIS — por qué migrar |
| 00:15 | 15 min | Parte 2 | Preparar PostGIS: base + importar datos |
| 00:30 | 20 min | Parte 3 | Conectar GeoServer a PostGIS |
| 00:50 | 10 min | Descanso | |
| 01:00 | 20 min | Parte 4-5 | Publicar capas + Vistas SQL |
| 01:20 | 15 min | Parte 6 | Layer Groups |
| 01:35 | 15 min | Práctica | Estudiantes conectan su PostGIS |
| 01:50 | 10 min | Cierre | Resumen módulo, preview Módulo 3 (SLD) |

---

## 🎬 Desarrollo

### Apertura (10 min)

**Muestra la tabla de Shapefile vs PostGIS en pantalla.**

> "La clase pasada trabajamos con shapefiles. Funcionan, pero tienen limitaciones serias. Hoy conectamos GeoServer con PostGIS — la base de datos espacial más usada del mundo. Es el paso de amateur a profesional."

> "¿Cuántos de ustedes han usado PostGIS antes? ¿PostgreSQL al menos?"

Si pocos levantan la mano:
> "No se preocupen. No vamos a hacer SQL avanzado. Solo necesitan crear una base, importar datos y conectar. GeoServer hace el resto."

### Parte 2 — Preparar PostGIS (15 min)

**Construye EN VIVO:**

1. `CREATE DATABASE geoserver_curso;`
2. `CREATE EXTENSION postgis;`
3. Importar un shapefile con shp2pgsql

> "Tres comandos y tienen una base de datos espacial lista. Ahora importemos los departamentos de Bolivia."

**Si algún estudiante no tiene PostGIS:**

> "Si no lo tienen instalado, pueden seguir la demo conmigo y lo instalan como tarea. En la práctica les doy los pasos. Para las siguientes clases SÍ van a necesitar PostGIS activo."

### Parte 3 — Conectar GeoServer (20 min)

**Momento crítico: Si la conexión falla, se pierde la clase.**

> "Antes de configurar en GeoServer, vamos a probar la conexión desde psql o PgAdmin. Si funciona ahí, funciona en GeoServer. Si no funciona ahí, NO va a funcionar en GeoServer."

Hazlo paso a paso EN VIVO. Si falla:
1. ¿PostgreSQL está corriendo? → `sudo systemctl status postgresql`
2. ¿Contraseña correcta? → probar con psql
3. ¿pg_hba.conf permite conexiones? → verificar la línea de `md5`

**Cuando funciona:**

> "¿Ven la lista de tablas? GeoServer detectó todas las tablas con geometría de la base. Solo falta publicarlas."

### Parte 4-5 — Publicar + Vistas SQL (20 min)

**Publicar tabla normal primero.** Mostrar que es igual que con shapefile pero con nombres de campo completos.

> "¿Ven? `superficie_km2` aparece completo. Con shapefile se truncaba a `superfici`. Esa es la primera ventaja."

**Vista SQL — el momento wow de esta clase:**

> "Ahora viene lo potente. Quiero una capa que muestre SOLO los municipios de La Paz con más de 10,000 habitantes. Con shapefiles tendría que crear un archivo nuevo. Con PostGIS, escribo una consulta y GeoServer la ejecuta al vuelo."

Crea la vista EN VIVO. Cuando funcione en Layer Preview:

> "Esta capa NO existe como tabla. Se genera dinámicamente desde el SQL. Si mañana agrego un municipio nuevo a la tabla y tiene más de 10,000 habitantes, aparece automáticamente. Sin tocar GeoServer."

### Parte 6 — Layer Groups (15 min)

> "Último tema del módulo. ¿Quieren que sus usuarios agreguen UNA sola URL y obtengan todo el mapa de Bolivia? Para eso son los Layer Groups."

Crea un grupo con: departamentos (fondo) → ríos → carreteras → capitales (arriba).

> "El orden importa. Lo que está abajo se dibuja primero. Los capitales van arriba porque son puntos que deben verse sobre todo lo demás."

### Cierre — Preview Módulo 3

> "Tienen datos publicados, pero se ven genéricos — los polígonos son grises, las líneas negras, los puntos rojos. En la Módulo 3 les damos vida con estilos SLD: colores por departamento, grosores por tipo de río, íconos personalizados. La diferencia entre un mapa amateur y uno profesional está en la simbología."

---

## ⚠️ Errores Comunes

| Problema | Solución |
|----------|----------|
| Cannot connect to PostGIS | Verificar: ¿PG corriendo? ¿Puerto? ¿Contraseña? ¿pg_hba.conf? |
| Tabla no aparece en la lista | No tiene columna de geometría tipo `geometry` |
| Vista SQL sin geometría | Asegurar que la columna `geom` está en el SELECT |
| Vista SQL sin SRID | Especificar SRID manualmente al configurar la vista |
| Layer Group vacío | Verificar que las capas individuales funcionan primero |
| Datos PostGIS no se ven | Verificar índice espacial + Compute from data |

## 💡 Tips

1. **Ten PostGIS funcionando ANTES de la clase** y avisa a los estudiantes que lo instalen
2. **La vista SQL es lo que más impresiona** — tómate el tiempo de mostrarla bien
3. **Muestra el Layer Group en QGIS** — agregar como WMS con una sola URL
4. **Si un estudiante no tiene PostGIS:** que siga la demo contigo y lo instale como tarea para la Clase 04