# 🎓 Guía del Instructor — Clase 08

## GeoWebCache & Deploy en Producción — ÚLTIMA CLASE

**Duración:** 2 horas (120 minutos)
**Módulo:** 7 de 7

---

## ⏱️ Cronograma

| Tiempo | Duración | Sección | Actividad |
|--------|----------|---------|-----------|
| 00:00 | 5 min  | Repaso | Verificar visor Leaflet del Módulo 6 |
| 00:05 | 10 min | Parte 1a | ¿Qué es GeoWebCache? Concepto de tiles |
| 00:15 | 15 min | Parte 1b | Configurar caching + seed EN VIVO |
| 00:30 | 10 min | Parte 1c | Comparar velocidad: con vs sin cache (F12 Network) |
| 00:40 | 10 min | Descanso | |
| 00:50 | 35 min | Parte 2 | Deploy Google Cloud EN VIVO paso a paso |
| 01:25 | 10 min | Parte 2b | Subir datos + verificar acceso público |
| 01:35 | 5 min  | Demo | Visor Leaflet apuntando al servidor cloud |
| 01:40 | 20 min | Cierre | Repaso del curso completo + próximos pasos |

---

## 🎬 Desarrollo

### Apertura (5 min)

> "Última clase. Hoy cerramos el ciclo completo. Primero: su servidor puede ser lento si tiene muchos usuarios — lo arreglamos con GeoWebCache. Segundo: su servidor solo funciona en su PC — lo llevamos a la nube de Google donde cualquier persona del planeta puede acceder."

### Parte 1a — Concepto de tiles y cache (10 min)

**Analogía del restaurante:**

> "Imaginen un restaurante donde el plato más pedido es la salteña. ¿Preparan cada salteña desde cero cuando la piden? No. Las preparan en cantidad ANTES de que lleguen los clientes y las tienen listas en el mostrador. Eso es GeoWebCache: prepara los tiles más pedidos y los tiene listos."

**Mostrar la pirámide de tiles en pizarra:**

```
Zoom 0: 1 tile (mundo)
Zoom 1: 4 tiles
Zoom 2: 16 tiles
...cada nivel × 4
```

> "Google Maps funciona así. Millones de tiles precalculados. Cuando mueves el mapa, tu navegador pide solo los tiles visibles. Si ya están en cache, se sirven instantáneamente."

### Parte 1b — Configurar + seed (15 min)

**EN VIVO:** Activar caching en la capa departamentos de Bolivia. Ejecutar seed de zoom 0-8.

> "Mientras el seed corre, vean el progreso. Está generando ~500 tiles para Bolivia. Cada uno es una imagen PNG de 256×256 que ya no necesita recalcular."

### Parte 1c — Comparar velocidad (10 min)

**Momento más impactante de la Parte 1.**

1. Abrir F12 → Network en Chrome
2. Cargar Layer Preview → ver tiempos: ~200-400ms por tile
3. Ahora activar el cache → recargar → ver tiempos: ~5-15ms por tile

> "¿Ven la diferencia? De 300 milisegundos a 10 milisegundos. ×30 más rápido. Con 100 usuarios simultáneos, esta diferencia es la que separa un servidor que responde de uno que se cae."

### Parte 2 — Deploy (35 min)

**Esta es la demo principal. Prepárala bien.**

> "Todo lo que hicimos hasta ahora vive en su PC. Si apagan la computadora, nadie más puede ver sus mapas. Ahora vamos a poner esto en un servidor de Google que funciona 24/7."

**TEN TU PROPIA VM YA CREADA como backup.** Si algo falla en la demo en vivo, muestras tu VM funcionando.

**Secuencia EN VIVO:**

1. Abrir console.cloud.google.com
2. Crear proyecto → Crear VM → Explicar cada opción
3. Clic en SSH → terminal en el navegador
4. Copiar los comandos del README uno por uno
5. Cada 3-4 comandos, explicar qué hace

> "Este comando instala Java. Este instala Tomcat. Este descarga GeoServer. Este lo copia a la carpeta correcta. Reiniciamos Tomcat y... vamos a ver..."

6. Crear regla de firewall para puerto 8080
7. Abrir `http://IP_EXTERNA:8080/geoserver`

> "¿Lo ven? GeoServer corriendo en la nube de Google, accesible con una IP pública. Cualquiera del mundo con esta URL puede ver su servidor."

**Si los estudiantes quieren hacerlo ellos:** Que lo hagan como tarea. En clase muéstralo tú y asegúrate de que el concepto queda claro. Crear una VM por cada estudiante en vivo consumiría demasiado tiempo.

### Demo final — Visor público (5 min)

**El cierre visual del curso completo:**

1. Abrir el `visor.html` del Módulo 6
2. Cambiar la URL de `localhost` a la IP pública del servidor cloud
3. Abrir en el navegador
4. Las capas de Bolivia se cargan desde Google Cloud

> "Datos publicados en GeoServer, estilos diseñados con SLD, seguridad configurada, rendimiento optimizado con cache, desplegado en la nube, y un visor web consumiéndolo. Todo lo que aprendieron en 4 semanas, funcionando en producción."

### Cierre del curso (20 min)

**Repaso participativo** — pregunta módulo por módulo.

**Próximos pasos** — lista del README. Destacar:
- HTTPS como paso obligatorio para producción real
- Docker como alternativa moderna de deploy
- Curso de Flutter GIS para apps móviles

**Feedback:**
> "Quiero su feedback honesto. ¿Qué les sirvió más? ¿Qué mejorarían? ¿Qué les faltó?"

**Cierre emocional:**
> "Hace 4 semanas no sabían qué era GeoServer. Ahora tienen un servidor de mapas en la nube accesible desde internet, con datos publicados profesionalmente, estilos propios, seguridad configurada y un visor web. Eso no es trivial — es lo que hacen los profesionales en la industria. Felicidades."

---

## ⚠️ Preparación ANTES de la clase

**Esto es CRÍTICO para esta clase:**

1. **TU VM debe estar lista** con GeoServer funcionando, datos de Bolivia cargados, estilos aplicados. Si algo falla en la demo en vivo, muestras tu VM.
2. **Verificar que tu VM es accesible** desde otra red (no solo tu WiFi). Pedir a alguien que pruebe.
3. **Tener los comandos** en un archivo de texto listo para copiar. No escribas en vivo en la terminal SSH.
4. **Tener el visor Leaflet** con la URL apuntando a tu IP pública.
5. **Verificar crédito de Google Cloud** — que no se haya agotado.

---

## ⚠️ Errores Comunes

| Problema | Solución |
|----------|----------|
| VM no arranca | Probar otra región. e2-medium no disponible en todas |
| GeoServer no accesible | ¿Tomcat corriendo? ¿Puerto 8080 abierto? ¿IP correcta? |
| Seed en 0% | BBOX de la capa no configurado. Compute from data |
| Disco lleno | Zoom del seed muy alto. Truncar y bajar zoom máximo |
| Out of memory | Xmx muy alto para la VM. Con 4 GB VM, usar Xmx2048m máximo |
| CORS en cloud | Configurar CORS en web.xml del GeoServer EN LA NUBE |
| Firewall no funciona | Verificar que aplica a la VM correcta |

## 💡 Tips

1. **Ten tu VM lista como backup.** La demo en vivo puede fallar por mil razones. Tu backup es tu seguro.
2. **Los comandos de instalación** deben estar en un archivo para copiar/pegar. No los escribas a mano en SSH.
3. **La comparación de velocidad** (con vs sin cache) es el momento más revelador de la Parte 1. Tómate el tiempo.
4. **El visor Leaflet apuntando al cloud** es el cierre visual perfecto. Todo el curso converge en ese momento.
5. **Si sobra tiempo:** Instalar PostGIS en el servidor cloud y conectar GeoServer. O configurar CORS y probar el visor.
6. **Si falta tiempo:** Saltar el seed (mostrar solo la config) y enfocarse en el deploy. El deploy es más importante que el cache.
7. **Certificados/HTTPS:** Mencionarlo como siguiente paso pero NO intentar configurar en clase. Requiere dominio y toma tiempo.