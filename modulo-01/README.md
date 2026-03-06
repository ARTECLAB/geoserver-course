# Módulo 1 · Arquitectura & Primeros Pasos ⚙️

> **Clase 1 · 2 horas**
> Entender cómo funciona un servidor de mapas por dentro, instalar GeoServer, navegar la interfaz de administración y publicar la primera capa geoespacial.

---

## 🎯 Objetivos

- Entender la arquitectura cliente-servidor en GIS
- Conocer los estándares OGC (WMS, WFS, WCS)
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

### Los 3 estándares OGC que dominarás

| Estándar | ¿Qué hace? | Analogía |
|----------|-----------|----------|
| **WMS** | Devuelve imágenes del mapa | Pides un plato servido → recibes una foto |
| **WFS** | Devuelve datos vectoriales (GeoJSON, GML) | Pides los ingredientes → recibes la receta |
| **WCS** | Devuelve datos raster | Pides la imagen satelital completa |

> **En este curso nos enfocamos en WMS y WFS** — son los más usados en producción.

---

## Parte 2 — Instalación de GeoServer (25 min)

### Requisitos previos

- **Java 11+** (OpenJDK o Oracle JDK)
- Al menos 2GB RAM disponibles
- Puerto 8080 libre

### Windows — Instalación con binario ZIP

```powershell
# 1. Verificar Java
java -version

# 2. Descargar GeoServer (versión estable)
# Ir a https://geoserver.org/release/stable/
# Descargar "Binary (OS independent)"

# 3. Descomprimir en C:\geoserver

# 4. Iniciar GeoServer
cd C:\geoserver\bin
startup.bat

# 5. Abrir en el navegador
# http://localhost:8080/geoserver
```

### Linux (Ubuntu/Debian)

```bash
# 1. Instalar Java
sudo apt update
sudo apt install openjdk-11-jdk -y

# 2. Verificar
java -version

# 3. Descargar GeoServer
cd /opt
sudo wget https://sourceforge.net/projects/geoserver/files/GeoServer/2.26.1/geoserver-2.26.1-bin.zip
sudo unzip geoserver-2.26.1-bin.zip
sudo mv geoserver-2.26.1 geoserver

# 4. Iniciar
cd /opt/geoserver/bin
sudo ./startup.sh

# 5. Abrir: http://localhost:8080/geoserver
```

### Credenciales por defecto

```
Usuario: admin
Contraseña: geoserver
```

⚠️ **CAMBIAR EN PRODUCCIÓN** — En el Módulo 5 configuramos seguridad real.

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

1. **"Port 8080 already in use"** → Otro programa usa el puerto. Cambiar en `start.ini` o cerrar el programa que usa 8080
2. **Java no encontrado** → Verificar `java -version`. Si no funciona, reinstalar JDK y agregar al PATH
3. **Shapefile no se lee** → Verificar que los 4 archivos están juntos (.shp, .shx, .dbf, .prj)
4. **Capa sin proyección** → Falta el archivo .prj. Descargarlo o definir el SRS manualmente
5. **Mapa en blanco en preview** → Verificar Bounding Box. Clic en "Compute from data" y "Compute from native bounds"
6. **Caracteres raros en atributos** → Charset incorrecto. Cambiar a UTF-8 en el Store