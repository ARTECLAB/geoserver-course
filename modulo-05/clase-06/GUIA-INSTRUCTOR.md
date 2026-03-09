# 🎓 Guía del Instructor — Clase 06

## Seguridad & Control de Acceso

**Duración:** 2 horas (120 minutos)
**Módulo:** 5 de 7

---

## ⏱️ Cronograma

| Tiempo | Duración | Sección | Actividad |
|--------|----------|---------|-----------|
| 00:00 | 5 min  | Repaso | Verificar práctica Módulo 4 (CQL) |
| 00:05 | 10 min | Intro | ¿Por qué seguridad? Escenarios reales |
| 00:15 | 5 min  | Parte 1 | Cambiar contraseña de admin |
| 00:20 | 10 min | Parte 2 | Conceptos: Usuarios → Roles → Reglas |
| 00:30 | 10 min | Parte 3 | Crear roles y usuarios EN VIVO |
| 00:40 | 20 min | Parte 4 | Reglas de acceso a datos — paso a paso |
| 01:00 | 10 min | Descanso | |
| 01:10 | 10 min | Parte 5 | Reglas de servicios (WMS vs WFS) |
| 01:20 | 20 min | Parte 6 | Pruebas completas: navegador + QGIS |
| 01:40 | 10 min | Extra | Actividad B "¿Qué pasa si...?" o Actividad C |
| 01:50 | 10 min | Cierre | Buenas prácticas, preview Módulo 6 |

---

## 🎬 Desarrollo

### Apertura (10 min)

> "Tienen un GeoServer con datos publicados, estilos bonitos y filtros CQL potentes. Pero ahora mismo, ¿quién puede acceder? CUALQUIERA. La contraseña de admin es 'geoserver' — la sabe todo el mundo. Todas las capas están abiertas. En un proyecto real, eso es una falla de seguridad grave."

**Plantear escenarios reales:**

> "Imaginen que trabajan en el gobierno departamental de La Paz. Tienen datos de catastro: parcelas, propietarios, avalúos. ¿Quieren que cualquier persona en internet vea cuánto vale la casa de su vecino? No. Necesitan controlar quién ve qué."

> "O imaginen una consultora ambiental con 5 clientes. Cada cliente pagó por su estudio. ¿Quieren que el cliente A vea los datos del cliente B? No. Cada uno solo ve lo suyo."

**TUS DATOS para demo:** Usar el escenario del Gobierno Departamental de La Paz (README) con 3 workspaces y 5 roles.

### Parte 2 — Conceptos (10 min)

**Dibuja en pizarra/pantalla:**

```
USUARIOS          ROLES              REGLAS
─────────         ──────             ──────
ciudadano_web  →  VISOR_PUBLICO   →  datos_publicos: READ ✅
                                     catastro: ❌
                                     WFS: ❌

tecnico_catastro → CATASTRO       →  datos_publicos: READ ✅
                                     catastro: READ + WRITE ✅
                                     avaluos: ❌

director        →  DIRECTOR       →  TODO: READ ✅
                                     WRITE: ❌
                                     ADMIN: ❌
```

> "Los usuarios son personas. Los roles son permisos agrupados. Las reglas dicen qué puede hacer cada rol. Si mañana contratan un nuevo técnico de catastro, solo le asignan el rol CATASTRO y automáticamente tiene los mismos permisos. No necesitan crear reglas nuevas."

### Parte 3-4 — Crear roles, usuarios y reglas EN VIVO (30 min)

**Hazlo paso a paso. Esta es la parte más operativa de la clase.**

Secuencia:
1. Cambiar contraseña admin (2 min)
2. Crear 3 roles (3 min)
3. Crear 3 usuarios con roles (5 min)
4. Crear reglas de datos — empezar por la más simple (datos_publicos → todos leen)
5. Ir agregando reglas más restrictivas (catastro → solo ciertos roles)
6. La regla de avalúos individual (demostrar prioridad de reglas específicas)

> "¿Ven? La regla de avalúos es más específica (workspace + capa individual) que la regla general de catastro (workspace + todas). GeoServer evalúa la más específica primero. El técnico de catastro puede ver parcelas pero NO avalúos."

### Parte 5 — Reglas de servicios (10 min)

> "El ciudadano puede ver el mapa de departamentos en la web. ¿Pero debe poder descargar el shapefile de departamentos? En muchos proyectos no. El WMS es público pero el WFS es restringido."

Crear la regla de servicio WFS EN VIVO.

### Parte 6 — Pruebas (20 min)

**Este es el momento más satisfactorio de la clase.** Las restricciones funcionan y los estudiantes lo ven.

1. **Ventana de incógnito** → login como ciudadano_web
   > "¿Ven? Solo ve datos_publicos. Las capas de catastro y planificación no existen para este usuario."

2. **Cambiar a tecnico_catastro**
   > "Ahora ve más capas. Pero NO ve avalúos. La regla específica le bloqueó esa capa."

3. **Probar URL WFS como ciudadano_web**
   > "Error 403 Forbidden. No tiene permiso. La regla de servicio WFS lo bloqueó."

4. **QGIS** — conectar como diferentes usuarios y comparar las capas visibles

> "Esto es exactamente lo que pasa en un sistema real. Cada usuario ve solo lo que debe ver."

### Actividad Extra — "¿Qué pasa si...?" (10 min)

Si sobra tiempo, plantear escenarios:

- "Un empleado renuncia" → Desactivar usuario, no borrar (auditoría)
- "Un hacker intenta entrar" → Contraseñas fuertes + HTTPS + monitoreo de logs
- "El director necesita acceso temporal a catastro:avaluos" → Asignarle ROLE_CATASTRO temporal
- "Necesitan dar acceso a 50 ciudadanos nuevos" → Crear rol, crear usuarios, asignar rol. O mejor: acceso anónimo para datos públicos

### Cierre (10 min)

**Resumen de buenas prácticas** (mostrar la lista del README).

**Preview Módulo 6:**

> "Su GeoServer ahora está seguro y profesional. En la siguiente clase lo conectamos con el mundo: un visor web con Leaflet que consume sus servicios WMS/WFS, y una app Flutter que muestra sus capas en un mapa móvil. Un servidor, múltiples plataformas."

---

## ⚠️ Errores Comunes

| Problema | Solución |
|----------|----------|
| Admin pierde acceso | ROLE_ADMIN no incluido en la regla. Siempre incluirlo |
| Usuario no ve nada | Su rol no está en ninguna regla de datos |
| WFS sigue funcionando para visor | Method debe ser * (todos), no solo GetFeature |
| Se bloqueó completamente | Editar `data_dir/security/layers.properties` manualmente |
| Contraseña olvidada | Editar `data_dir/security/usergroup/default/users.xml` |
| Regla genérica sobreescribe | Verificar prioridad: workspace+capa > workspace+* |

## 💡 Tips

1. **Dibuja el esquema de seguridad en pizarra** antes de crear nada en GeoServer. Roles y flechas.
2. **Prueba CADA usuario en ventana de incógnito** después de crear las reglas. Es la única forma de estar seguro.
3. **El escenario del Gobierno Departamental** es realista y permite mostrar 3 niveles de acceso diferentes.
4. **Si un estudiante se bloquea:** Los archivos de seguridad están en `data_dir/security/`. Se pueden editar con un editor de texto.
5. **Conecta con su realidad:** "¿En su trabajo quién debería ver qué? Diseñen eso." La Actividad C es perfecta para esto.
6. **Si sobra mucho tiempo:** Hacer la Actividad C (diseño de esquema personalizado) — cada uno diseña para su contexto y lo comparten.
7. **Si falta tiempo:** Saltar la restricción de servicios (Parte 5) y asignar como tarea. Las reglas de datos (Parte 4) son más importantes.