# Módulo 7 · GeoWebCache & Deploy en Producción ☁️

> **Clase 8 · 2 horas**
> El cierre del curso. Optimizas el rendimiento con GeoWebCache y despliegas en Google Cloud. Tu servidor queda público con datos de Bolivia.

---

## 🎯 Objetivos

- Configurar GeoWebCache para tile caching
- Ejecutar seed y truncate por capa
- Comparar rendimiento con y sin cache
- Crear VM en Google Cloud con free tier
- Instalar Java + Tomcat 9 + GeoServer 2.28.2 WAR en la nube
- Abrir puertos y acceder públicamente
- Subir datos y verificar acceso remoto
- Aplicar checklist de producción

---

## Parte 1 — GeoWebCache (40 min)

### Demostración de velocidad: con vs sin cache

**EN VIVO:** Abrir F12 → Network → Cargar Layer Preview → ver tiempos por tile.

| Métrica | Sin cache | Con cache |
|---------|-----------|-----------|
| Tiempo por tile | ~200-500 ms | ~5-20 ms |
| CPU | Alta (renderizando) | Mínima (leyendo disco) |
| Escalabilidad | 10-20 usuarios simultáneos | 100+ usuarios |

**Tu dato para clase:** Usar la capa `departamentos` de Bolivia con estilo clasificado (SLD pesado con etiquetas). Mostrar la diferencia.

### Configurar caching para datos de Bolivia

```
Capa: datos_bolivia:departamentos
  Tile caching: ✓
  Formats: image/png, image/jpeg
  Gridsets: EPSG:4326, EPSG:900913
  Metatiling: 4×4

Capa: datos_bolivia:rios_principales
  Tile caching: ✓
  Formats: image/png
  Metatiling: 4×4

Capa: datos_bolivia:capitales
  Tile caching: ✓
  Formats: image/png
  Metatiling: 2×2  (puntos no necesitan metatiles grandes)
```

### Seed del territorio boliviano

```
Zoom start: 0
Zoom stop:  10
Grid set:   EPSG:4326
Format:     image/png
Bounding box (Bolivia): -69.7, -22.9, -57.4, -9.6
```

> **Tip:** Limitar el BBOX al territorio de Bolivia evita generar tiles vacíos del resto del mundo. Reduce tiempo y espacio en disco.

### ¿Cuánto espacio ocupan los tiles?

Estimación para Bolivia (zoom 0-10):

| Zoom | Tiles aprox | Espacio |
|------|-------------|---------|
| 0-5 | ~50 | < 1 MB |
| 6-8 | ~2,000 | ~20 MB |
| 9-10 | ~15,000 | ~150 MB |
| Total | ~17,000 | ~170 MB |

Si agregas zoom 11-14 para zonas urbanas: +500 MB. Zoom 15+: +varios GB.

---

## Parte 2 — Deploy en Google Cloud (50 min)

### Preparación ANTES de la clase

1. **Crear tu propia VM** con GeoServer funcionando
2. **Subir datos de Bolivia** al servidor
3. **Verificar que funciona** desde otra red (pedir a alguien que pruebe)
4. **Tener el visor Leaflet** (Módulo 6) apuntando a la IP pública

### Demo EN VIVO paso a paso

La secuencia de la demo:

1. Abrir Google Cloud Console
2. Crear VM → e2-medium, Ubuntu 22.04, 30 GB
3. SSH desde el navegador
4. Instalar Java + Tomcat + GeoServer (copiar los comandos del README)
5. Abrir puerto 8080 en firewall
6. Acceder: `http://IP_EXTERNA:8080/geoserver`
7. Cambiar contraseña
8. Subir un shapefile con `gcloud compute scp`
9. Publicar la capa
10. Verificar desde Layer Preview

> **Momento wow final:** Abrir el visor Leaflet del Módulo 6 pero apuntando a la IP pública del servidor en la nube. Los estudiantes ven su mapa de Bolivia servido desde Google Cloud.

### Instalación completa en el servidor (comandos para copiar)

```bash
# ── Sistema ──
sudo apt update && sudo apt upgrade -y

# ── Java ──
sudo apt install openjdk-11-jdk -y

# ── Tomcat 9 ──
sudo apt install tomcat9 tomcat9-admin -y
sudo systemctl enable tomcat9

# ── Memoria ──
sudo bash -c 'cat > /usr/share/tomcat9/bin/setenv.sh << EOF
export CATALINA_OPTS="-Xms512m -Xmx2048m -XX:+UseG1GC"
EOF'
sudo chmod +x /usr/share/tomcat9/bin/setenv.sh

# ── GeoServer 2.28.2 ──
cd /tmp
wget https://sourceforge.net/projects/geoserver/files/GeoServer/2.28.2/geoserver-2.28.2-war.zip
sudo apt install unzip -y
unzip geoserver-2.28.2-war.zip
sudo cp geoserver.war /var/lib/tomcat9/webapps/

# ── CORS (para visor web) ──
# Esperar a que GeoServer se despliegue (~60 seg)
# Luego editar web.xml para agregar filtro CORS
sudo nano /var/lib/tomcat9/webapps/geoserver/WEB-INF/web.xml
# Agregar el bloque CORS antes de </web-app>

# ── Carpeta de datos ──
sudo mkdir -p /opt/geodatos
sudo chown tomcat:tomcat /opt/geodatos

# ── Reiniciar ──
sudo systemctl restart tomcat9

# ── Verificar ──
sudo systemctl status tomcat9
curl -I http://localhost:8080/geoserver/web/
```

### PostGIS en el servidor (opcional avanzado)

```bash
# Instalar PostgreSQL + PostGIS
sudo apt install postgresql postgis postgresql-14-postgis-3 -y

# Crear base de datos
sudo -u postgres createdb geoserver_prod
sudo -u postgres psql -d geoserver_prod -c "CREATE EXTENSION postgis;"

# Crear usuario para GeoServer
sudo -u postgres psql -c "CREATE USER geoserver WITH PASSWORD 'contraseña_segura';"
sudo -u postgres psql -c "GRANT ALL ON DATABASE geoserver_prod TO geoserver;"
```

### HTTPS con Nginx reverse proxy (referencia)

Para producción real necesitas HTTPS. La forma estándar:

1. Instalar Nginx como reverse proxy
2. Obtener certificado SSL con Let's Encrypt (certbot)
3. Nginx escucha en puerto 443 (HTTPS) y redirige a Tomcat en 8080

```bash
# Instalar Nginx
sudo apt install nginx -y

# Instalar certbot
sudo apt install certbot python3-certbot-nginx -y

# Obtener certificado (necesitas un dominio apuntando a tu IP)
sudo certbot --nginx -d tudominio.com
```

> **Nota:** HTTPS requiere un dominio propio. Para el curso usamos solo IP+HTTP, pero menciono HTTPS como paso siguiente para producción real.

---

## Parte 3 — Cierre del Curso (20 min)

### Repaso por módulo (pedir participación)

> "Empezaron hace 4 semanas sin saber qué era GeoServer. ¿Qué saben ahora?"

- Módulo 1: "Instalaron un servidor de mapas desde cero"
- Módulo 2: "Publicaron datos desde shapefile y PostGIS"
- Módulo 3: "Diseñaron estilos profesionales con SLD"
- Módulo 4: "Dominaron WMS, WFS y filtros CQL"
- Módulo 5: "Protegieron el servidor con seguridad real"
- Módulo 6: "Construyeron un visor web y vieron Flutter"
- Módulo 7: "Optimizaron rendimiento y desplegaron en la nube"

> "Ahora tienen un servidor de mapas en producción, accesible desde cualquier parte del mundo, con sus propios datos, estilos profesionales, seguridad configurada, y un visor web que lo consume. Eso es lo que hacen los profesionales GIS en las empresas y los gobiernos."

### Próximos pasos que pueden explorar

- **HTTPS + dominio propio** con Let's Encrypt (gratuito)
- **PostGIS en producción** para datos dinámicos y editables
- **WFS-T** para edición remota de features
- **GeoNode** como plataforma completa de IDE sobre GeoServer
- **MapProxy** como alternativa/complemento a GeoWebCache
- **Docker** para desplegar GeoServer en contenedores
- **Monitoreo** con herramientas como Grafana + Prometheus
- **Curso Flutter GIS** para construir apps móviles que consuman sus servicios

---

### 💻 Práctica 08

1. Activa GeoWebCache en al menos 2 capas
2. Ejecuta seed de zoom 0-8 para una capa
3. Compara velocidad con y sin cache (F12 → Network)
4. Crea cuenta en Google Cloud (cloud.google.com/free)
5. Crea VM con Ubuntu 22.04, 4 GB RAM
6. Instala Java + Tomcat + GeoServer en la VM
7. Abre puerto 8080 en firewall
8. Accede desde tu navegador local a la IP pública
9. Cambia contraseña de admin
10. (Extra) Sube un shapefile y publícalo en el servidor cloud

### ✅ Checklist Final del Curso

- [ ] GeoWebCache configurado y seed ejecutado
- [ ] Diferencia de velocidad verificada
- [ ] VM en Google Cloud funcionando
- [ ] GeoServer accesible desde IP pública
- [ ] Contraseña cambiada
- [ ] Entiendo el checklist de producción completo
- [ ] 🎉 ¡Curso completado!

---

## 📝 Errores comunes

1. **VM no arranca** → Verificar que la región tiene disponibilidad de máquinas e2-medium. Probar otra región.
2. **GeoServer no accesible** → Verificar: ¿Tomcat corriendo? ¿Puerto 8080 abierto en firewall? ¿IP externa correcta?
3. **Seed se queda en 0%** → Verificar Bounding Box de la capa. Sin BB, GeoWebCache no sabe qué área cachear.
4. **Disco lleno** → Seed con zoom muy alto. Truncar y hacer seed con zoom máximo más bajo.
5. **Java out of memory** → Aumentar -Xmx en setenv.sh. Con 4 GB de VM, usar -Xmx2048m máximo.
6. **Firewall no abre** → Verificar que la regla aplica a "All instances" o tiene el tag correcto de la VM.
7. **CORS error en visor web apuntando al cloud** → Configurar CORS en web.xml del GeoServer en la nube (igual que en local).

## 💡 Tips de producción

- **Siempre limita el BBOX del seed** a la zona que realmente necesitas. No generes tiles para todo el mundo.
- **Usa JPEG para capas base pesadas** (satélite, ortofoto). PNG para capas con transparencia.
- **Metatiling 4×4** es el sweet spot para la mayoría de capas.
- **Haz seed en horas de baja carga** (noche/madrugada). El proceso consume CPU.
- **Backup del data_dir** antes y después de cualquier cambio importante. Es tu seguro de vida.
- **Monitorea el disco** — los tiles acumulados pueden llenarlo. Configura alertas.
- **Para producción seria:** Usa Docker + Nginx + Let's Encrypt. Es más fácil de mantener y escalar.