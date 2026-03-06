# 🎓 Guía del Instructor — Clase 01

## Arquitectura & Primeros Pasos

**Duración:** 2 horas (120 minutos)
**Fecha:** Martes 17 de marzo de 2026
**Módulo:** 1 de 7

---

## ⏱️ Cronograma

| Tiempo | Duración | Sección | Actividad |
|--------|----------|---------|-----------|
| 00:00 | 15 min | Parte 1 | ¿Qué es GeoServer? Arquitectura y estándares OGC |
| 00:15 | 25 min | Parte 2 | Instalación EN VIVO (Windows + mostrar Linux) |
| 00:40 | 15 min | Parte 3 | Tour por la interfaz de administración |
| 00:55 | 10 min | Descanso | |
| 01:05 | 30 min | Parte 4 | Publicar primera capa paso a paso |
| 01:35 | 15 min | Parte 5 | Anatomía URL WMS + prueba en navegador |
| 01:50 | 10 min | Cierre | Práctica, preview clase 02 |

---

## 🎬 Desarrollo

### Apertura (15 min)

> "¿Cuántos de ustedes han usado QGIS para agregar una capa WMS? Levanten la mano..."

> "Bien. Esa capa que agregan en QGIS viene de ALGÚN LADO — de un servidor. Hoy van a ser ESE servidor. Van a instalar GeoServer, publicar datos, y al final de la clase tendrán una URL que podrían pegar en QGIS de cualquier colega y verían sus datos."

**Usa la analogía del restaurante.** Dibújala en pizarra/pantalla:

> "Sus datos son los ingredientes crudos. ¿Alguien va a un restaurante y le entregan los ingredientes? No. El chef los prepara, los presenta bonito, y se los sirve. GeoServer es el chef — toma sus shapefiles, sus PostGIS, y los sirve como mapas hermosos que cualquier aplicación puede consumir."

**TUS DATOS DE EJEMPLO:** departamentos, municipios, ríos de Bolivia.
Los estudiantes usarán datos de Natural Earth (genéricos, para cualquier país).

### Parte 2 — Instalación (25 min)

**Instala EN VIVO en tu máquina.** Los estudiantes siguen en sus PCs.

> "Vamos a usar la instalación profesional: WAR sobre Tomcat. ¿Por qué no el binario ZIP? Porque en producción nadie usa el Jetty embebido. Si desde el día 1 aprenden con Tomcat, cuando lleguemos al deploy en la nube ya saben cómo funciona."

Secuencia:
1. Verificar Java: `java -version`
2. Verificar/instalar Tomcat 9
3. Descargar GeoServer 2.28.2 WAR
4. Copiar `geoserver.war` a `webapps/`
5. Iniciar Tomcat → esperar que despliegue (~60 seg)
6. Abrir `http://localhost:8080/geoserver`
7. Login con admin/geoserver

**Errores que VAN a tener:**
- Java no instalado → tener link de adoptium.net listo
- Puerto 8080 ocupado → `netstat -ano | findstr 8080` para identificar
- WAR no se despliega → verificar que está en `webapps/`, no en una subcarpeta
- Memoria insuficiente → crear `setenv.bat`/`setenv.sh`

> "Si les dice 'java not found', instalen OpenJDK 11 desde adoptium.net. Si el puerto 8080 está ocupado, primero cierren el programa que lo usa."

### Parte 3 — Interfaz (15 min)

**Tour rápido.** NO te pierdas en cada menú. Solo muestra:
1. Server Status — "¿Está vivo? Sí."
2. Workspaces — "Carpetas para organizar"
3. Stores — "Conexiones a datos"
4. Layers — "Capas publicadas"
5. Layer Preview — "Verificar que funciona"

> "Solo estos 5. El resto lo iremos descubriendo en las siguientes clases."

### Parte 4 — Primera capa (30 min)

**Este es el momento clave de la clase.** Hazlo paso a paso, despacio.

1. Crear Workspace `datos_bolivia` (tú) / los estudiantes crean `mi_proyecto`
2. Crear Store apuntando al shapefile
3. Publicar la capa
4. **Momento wow:** Ir a Layer Preview y ver el mapa

> "¿Ven eso? Hace 30 minutos no tenían un servidor de mapas. Ahora tienen uno funcionando con sus propios datos. Ese es el poder de GeoServer."

### Parte 5 — URL WMS (15 min)

**Copia la URL del Layer Preview y analízala parámetro por parámetro.**

> "Cada parámetro tiene un propósito. `layers` dice QUÉ capa. `bbox` dice DÓNDE. `width/height` dice de QUÉ TAMAÑO. `format` dice EN QUÉ FORMATO. Simple."

**Demo extra:** Pega la URL WMS en QGIS (Capa → Agregar WMS → Nuevo → pegar URL base). Los estudiantes ven sus propios datos servidos desde su propio GeoServer y consumidos en QGIS.

> "Su QGIS acaba de consumir datos de su propio servidor. En el Módulo 6 será un sitio web. En el Módulo 7 será una app en la nube."

---

## ⚠️ Errores Comunes

| Problema | Solución |
|----------|----------|
| Java not found | Instalar OpenJDK 11 de adoptium.net |
| Port 8080 in use | `netstat -ano | findstr 8080` para identificar, cerrar el programa |
| Shapefile no se lee | Verificar que .shp, .shx, .dbf, .prj están juntos |
| Mapa en blanco | Compute from data → Compute from native bounds |
| Caracteres raros | Cambiar Charset a UTF-8 en el Store |

## 💡 Tips

1. **Ten los shapefiles listos** en una carpeta accesible ANTES de la clase
2. **Natural Earth** (naturalearthdata.com) tiene datos gratuitos de todo el mundo — ideal para los estudiantes
3. **Muestra en QGIS** al final — conectar su GeoServer a QGIS es el cierre perfecto de esta clase
4. **Pide que cambien la contraseña** del admin como tarea — conexión directa con el Módulo 5