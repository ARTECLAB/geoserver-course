# Módulo 1 · Arquitectura & Primeros Pasos ⚙️

> **Clase 1 · 2 horas**
> Entender cómo funciona un servidor de mapas por dentro, instalar GeoServer, navegar la interfaz de administración y publicar la primera capa geoespacial.

---

## 🎯 Objetivos

- Entender la arquitectura cliente-servidor en GIS
- Conocer los estándares OGC (WMS, WFS y panorama general de WCS, WMTS, WPS, CSW)
- Instalar GeoServer en Windows y Linux
- Navegar la interfaz de administración
- Entender la estructura: Workspace → Store → Layer
- Publicar la primera capa desde un shapefile

---

## Parte 1 — ¿Qué es GeoServer y por qué importa? (15 min)

### Concepto central

GeoServer es un servidor de datos geográficos de código abierto. Su trabajo: tomar datos espaciales (shapefiles, PostGIS, GeoTIFFs) y exponerlos como servicios web estándar que cualquier aplicación puede consumir.

### ¿Quién lo usa en el mundo real?

- **Gobiernos:** catastro municipal, planificación territorial, IDE nacionales
- **Empresas:** consultoría ambiental, ingeniería civil, minería
- **ONGs:** monitoreo de deforestación, gestión de riesgos
- **Startups:** apps de delivery con zonas de cobertura, inmobiliarias con mapas

### Analogía del restaurante (usar en clase)

```
COCINA (servidor)           COMEDOR (clientes)
─────────────────           ──────────────────
Datos crudos                Apps web (Leaflet)
  ↓                         Apps móviles (Flutter)
GeoServer los procesa       QGIS
  ↓                         ArcGIS Online
Sirve como WMS/WFS          Cualquier app que hable OGC
```

**Tus datos son los ingredientes.** No los mandas crudos al cliente.
GeoServer los cocina (renderiza, filtra, estiliza) y los sirve como imagen (WMS) o como datos (WFS).

### Los estándares OGC — El idioma universal de los servidores GIS

OGC (Open Geospatial Consortium) define los estándares que permiten que cualquier servidor y cualquier cliente GIS se entiendan entre sí. GeoServer implementa los principales:

#### ⭐ Los 2 que dominarás en este curso

| Estándar | ¿Qué hace? | Analogía | Uso típico |
|----------|-----------|----------|------------|
| **WMS** | Devuelve **imágenes** renderizadas del mapa (PNG, JPEG) | Pides un plato servido → recibes una foto del plato | Visualización de mapas en web y móvil. El 90% de lo que ves en un visor de mapas |
| **WFS** | Devuelve **datos vectoriales** (GeoJSON, GML) con geometrías y atributos | Pides los ingredientes → recibes la receta completa | Descarga de datos, análisis en el cliente, edición remota de features |

#### 📚 Otros estándares OGC que GeoServer soporta

| Estándar | ¿Qué hace? | Cuándo se usa |
|----------|-----------|---------------|
| **WCS** (Web Coverage Service) | Sirve datos **raster** completos (imágenes satelitales, DEMs, ortoimágenes) | Cuando necesitas el raster original para análisis, no solo una imagen visualizada |
| **WMTS** (Web Map Tile Service) | Sirve **tiles precalculados** (imágenes cortadas en cuadrículas) | Mapas base de alto rendimiento con miles de usuarios — Google Maps usa este concepto |
| **WPS** (Web Processing Service) | Ejecuta **geoprocesos** en el servidor (buffer, intersección, reproyección) | Análisis espacial remoto sin descargar datos. Poco usado en la práctica |
| **CSW** (Catalogue Service for the Web) | Publica y busca **metadatos** de capas y servicios | Infraestructuras de Datos Espaciales (IDE) nacionales y regionales |
| **TMS** (Tile Map Service) | Similar a WMTS pero más simple. Sirve tiles por URL predecible | Alternativa ligera a WMTS. Usado por OpenStreetMap |

> **En este curso nos enfocamos en WMS y WFS** — son los dos servicios que usarás en el 95% de los proyectos reales. WCS, WMTS y los demás los mencionaremos cuando sea relevante, pero no son el foco.

---

## Parte 2 — Instalación de GeoServer 2.28.2 (25 min)

### ¿Por qué WAR + Tomcat y no el binario ZIP?

GeoServer se distribuye de dos formas:
- **Binario ZIP** — Incluye un mini-servidor (Jetty). Rápido para probar, pero NO es lo que se usa en producción.
- **WAR** — Se despliega sobre Apache Tomcat, el servidor de aplicaciones Java estándar de la industria. Es lo que usan gobiernos, empresas e instituciones.

En este curso usamos **WAR + Tomcat** desde el día 1 porque:
1. Es la forma profesional — lo que encontrarás en cualquier trabajo real
2. Cuando lleguemos al Módulo 7 (deploy en la nube), ya sabrás exactamente cómo funciona
3. Tomcat te da mejor control sobre memoria, logs, seguridad y rendimiento

### Requisitos previos

- **Java 11+** (OpenJDK recomendado)
- **Apache Tomcat 9.x** (compatible con GeoServer 2.28.2)
- Al menos 2GB RAM disponibles
- Puerto 8080 libre (Tomcat lo usa por defecto)

### Windows — Instalación paso a paso

```powershell
# ── 1. Instalar Java ──
# Descargar OpenJDK 11 desde https://adoptium.net/
# Ejecutar el instalador, marcar "Set JAVA_HOME" durante la instalación
# Verificar:
java -version

# ── 2. Instalar Apache Tomcat 9 ──
# Descargar desde https://tomcat.apache.org/download-90.cgi
# → "Binary Distributions" → Core → ZIP (64-bit)
# Descomprimir en C:\tomcat9

# Probar que Tomcat arranca:
cd C:\tomcat9\bin
startup.bat
# Abrir http://localhost:8080 → debería mostrar la página de Tomcat
# Si funciona, detenerlo: shutdown.bat

# ── 3. Desplegar GeoServer WAR ──
# Descargar GeoServer 2.28.2 WAR desde:
# https://geoserver.org/release/stable/
# → "Web Archive" → descargar geoserver.war

# Copiar el archivo geoserver.war a la carpeta webapps de Tomcat:
copy geoserver.war C:\tomcat9\webapps\

# ── 4. Iniciar Tomcat ──
cd C:\tomcat9\bin
startup.bat

# Esperar ~60 segundos (Tomcat despliega el WAR automáticamente)
# Abrir: http://localhost:8080/geoserver
```

### Linux (Ubuntu/Debian)

```bash
# ── 1. Instalar Java ──
sudo apt update
sudo apt install openjdk-11-jdk -y
java -version

# ── 2. Instalar Tomcat 9 ──
sudo apt install tomcat9 tomcat9-admin -y

# Verificar que Tomcat está corriendo:
sudo systemctl status tomcat9
# Abrir http://localhost:8080 → página de Tomcat

# ── 3. Desplegar GeoServer WAR ──
cd /tmp
wget https://sourceforge.net/projects/geoserver/files/GeoServer/2.28.2/geoserver-2.28.2-war.zip
unzip geoserver-2.28.2-war.zip
sudo cp geoserver.war /var/lib/tomcat9/webapps/

# ── 4. Esperar ~60 segundos y abrir ──
# http://localhost:8080/geoserver
```

### Credenciales por defecto

```
Usuario: admin
Contraseña: geoserver
```

⚠️ **CAMBIAR EN PRODUCCIÓN** — En el Módulo 5 configuramos seguridad real.

### Configuración de memoria para Tomcat (recomendado)

GeoServer necesita suficiente memoria. Configura Tomcat para darle al menos 1GB:

**Windows:** Editar `C:\tomcat9\bin\setenv.bat` (crear si no existe):
```bat
set CATALINA_OPTS=-Xms512m -Xmx2048m
```

**Linux:** Crear `/usr/share/tomcat9/bin/setenv.sh`:
```bash
export CATALINA_OPTS="-Xms512m -Xmx2048m"
```

Reiniciar Tomcat después de este cambio.

---

## Parte 3 — La Interfaz de Administración (15 min)

### Panel principal — Qué hay en cada sección

| Sección | Para qué sirve |
|---------|----------------|
| **Server Status** | Ver si GeoServer está funcionando, memoria, versión |
| **Workspaces** | Carpetas organizativas para agrupar capas |
| **Stores** | Conexiones a fuentes de datos (archivos, bases de datos) |
| **Layers** | Capas publicadas y disponibles para servir |
| **Styles** | Estilos SLD para la visualización de capas |
| **Layer Groups** | Agrupar varias capas en un solo servicio |
| **Layer Preview** | Vista previa para verificar que las capas se ven bien |
| **Security** | Usuarios, roles y permisos de acceso |

### La jerarquía fundamental

```
Workspace (carpeta organizativa)
  └── Store (conexión a datos)
       └── Layer (capa publicada)
            └── Style (estilo visual)
```

**Todo en GeoServer sigue esta jerarquía.** Si entiendes esto, entiendes GeoServer.

---

## Parte 4 — Tu Primera Capa Publicada (30 min)

### Paso 1: Crear un Workspace

> Menú lateral → Workspaces → Add new workspace

```
Name:      datos_bolivia
Namespace: http://arteclab.com/datos_bolivia
Default:   ✓ (marcado)
```

**¿Qué es un Workspace?** Es como una carpeta en tu sistema de archivos. Agrupa todas las capas relacionadas. Para un proyecto de catastro municipal tendrías un workspace `catastro`. Para datos nacionales, `datos_bolivia`.

### Paso 2: Crear un Store (Shapefile)

> Menú lateral → Stores → Add new Store → Shapefile

```
Workspace:  datos_bolivia
Store name: departamentos_shp
Shapefile:  /ruta/a/departamentos.shp
Charset:    UTF-8
```

**¿Qué es un Store?** Es la conexión a la fuente de datos. Le dice a GeoServer "los datos están AQUÍ". Puede ser un shapefile, una carpeta de shapefiles, una base PostGIS, un GeoTIFF, etc.

### Paso 3: Publicar la capa

Después de crear el Store, GeoServer te pregunta qué capas quieres publicar.

> Clic en "Publish" junto al nombre de la capa

**Pestaña Data:**
- Name: `departamentos`
- Title: `Departamentos de Bolivia`
- Abstract: `Límites departamentales de Bolivia — fuente INE`
- SRS: `EPSG:4326` (o el que corresponda)
- Clic en "Compute from data" para el Bounding Box

**Pestaña Publishing:**
- Default Style: elegir un estilo (polygon por defecto)

> **Save**

### Paso 4: Verificar en Layer Preview

> Menú lateral → Layer Preview → Buscar "departamentos" → OpenLayers

Si ves el mapa con los departamentos, **tu primera capa está publicada**.

---

## Parte 5 — Anatomía de una URL WMS (15 min)

Con la capa publicada, GeoServer ya está sirviendo imágenes. Prueba en el navegador:

```
http://localhost:8080/geoserver/datos_bolivia/wms?
  service=WMS
  &version=1.1.1
  &request=GetMap
  &layers=datos_bolivia:departamentos
  &styles=
  &srs=EPSG:4326
  &bbox=-69.7,-22.9,-57.4,-9.6
  &width=768
  &height=768
  &format=image/png
```

Cambia `format=image/png` por `format=application/openlayers` para ver un visor interactivo.

---

### 💻 Práctica 01 — Tu primer servidor de mapas

1. Instala GeoServer en tu máquina (Windows o Linux)
2. Verifica que arranca correctamente en `http://localhost:8080/geoserver`
3. Crea un Workspace llamado `mi_proyecto`
4. Descarga un shapefile de tu país desde Natural Earth (https://www.naturalearthdata.com/)
5. Crea un Store apuntando al shapefile
6. Publica la capa
7. Verifica en Layer Preview que se ve correctamente
8. Copia la URL WMS y pégala en el navegador — verifica que devuelve una imagen PNG

### ✅ Checklist Clase 01

- [ ] GeoServer instalado y corriendo
- [ ] Login exitoso con admin/geoserver
- [ ] Interfaz de administración navegada
- [ ] Workspace creado
- [ ] Store creado apuntando a un shapefile
- [ ] Capa publicada y visible en Layer Preview
- [ ] URL WMS probada en el navegador

---

## 📝 Errores comunes

1. **"Port 8080 already in use"** → Otro programa usa el puerto. En Windows: verificar con `netstat -ano | findstr 8080`. Cerrar el programa o cambiar el puerto de Tomcat en `conf/server.xml`
2. **Java no encontrado** → Verificar `java -version`. Reinstalar JDK y asegurar que JAVA_HOME está configurado
3. **GeoServer no aparece en localhost:8080/geoserver** → Verificar que el WAR se desplegó: debe existir la carpeta `webapps/geoserver/`. Si no, verificar logs de Tomcat en `logs/catalina.out`
4. **Tomcat arranca pero GeoServer da error** → Memoria insuficiente. Configurar `setenv.bat`/`setenv.sh` con al menos `-Xmx1024m`
5. **Shapefile no se lee** → Verificar que los 4 archivos están juntos (.shp, .shx, .dbf, .prj) y que Tomcat tiene permisos de lectura
6. **Capa sin proyección** → Falta el archivo .prj. Descargarlo o definir el SRS manualmente en GeoServer
7. **Mapa en blanco en preview** → Verificar Bounding Box. Clic en "Compute from data" y "Compute from native bounds"
8. **Caracteres raros en atributos** → Charset incorrecto. Cambiar a UTF-8 en la configuración del Store