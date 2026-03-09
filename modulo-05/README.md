# Módulo 5 · Seguridad & Control de Acceso 🔒

> **Clase 6 · 2 horas**
> Protege tu GeoServer como un profesional. Usuarios, roles, reglas de acceso por capa y workspace, restricción de servicios y buenas prácticas de seguridad.

---

## 🎯 Objetivos

- Cambiar credenciales por defecto
- Crear roles con diferentes niveles de acceso
- Crear usuarios y asignarles roles
- Configurar reglas de acceso a datos por workspace y capa
- Restringir servicios (WMS vs WFS) por rol
- Probar y validar la seguridad desde navegador y QGIS
- Aplicar buenas prácticas: mínimo privilegio, separación de workspaces, HTTPS

---

## Escenario de clase — Gobierno Departamental de La Paz

Para esta clase usamos un escenario realista:

**El Gobierno Departamental tiene un GeoServer con estos datos:**

| Workspace | Capas | Sensibilidad |
|-----------|-------|-------------|
| `datos_publicos` | departamentos, ríos, carreteras, áreas protegidas | Abierto — cualquiera puede ver |
| `catastro` | parcelas, propietarios, avalúos | Restringido — solo funcionarios |
| `planificacion` | zonificación, proyectos_viales, expansión_urbana | Interno — solo equipo técnico |

**Tipos de usuario que necesitamos:**

| Usuario | Rol | Acceso |
|---------|-----|--------|
| `admin` | ROLE_ADMIN | Todo — administra GeoServer |
| `ciudadano_web` | ROLE_VISOR_PUBLICO | Solo `datos_publicos` vía WMS. Sin WFS. |
| `tecnico_catastro` | ROLE_CATASTRO | `datos_publicos` + `catastro`. WMS + WFS. |
| `planificador` | ROLE_PLANIFICACION | `datos_publicos` + `planificacion`. WMS + WFS. |
| `director` | ROLE_DIRECTOR | Todo lectura. Sin administración. |

---

## Parte 1 — Cambiar contraseña de admin (5 min)

> Security → Users, Groups, and Roles → Users/Groups → admin → cambiar password

Regla obligatoria: mínimo 10 caracteres, combinando letras, números y símbolos.

---

## Parte 2 — Crear los roles (10 min)

> Security → Users, Groups, and Roles → Roles → Add new role

```
ROLE_VISOR_PUBLICO
ROLE_CATASTRO
ROLE_PLANIFICACION
ROLE_DIRECTOR
```

> **Tip para clase:** Dibujar en pizarra la jerarquía de roles con flechas mostrando qué puede ver cada uno. Es visual y ayuda mucho.

---

## Parte 3 — Crear usuarios (10 min)

> Security → Users, Groups, and Roles → Users/Groups → Add new user

| Usuario | Password | Roles asignados |
|---------|----------|-----------------|
| `ciudadano_web` | `visor2026!` | ROLE_VISOR_PUBLICO |
| `tecnico_catastro` | `catastro2026!` | ROLE_CATASTRO |
| `planificador` | `plan2026!` | ROLE_PLANIFICACION |
| `director` | `director2026!` | ROLE_DIRECTOR |

---

## Parte 4 — Reglas de acceso a datos (20 min)

> Security → Data → Add new rule

### Reglas detalladas:

```
# 1. Datos públicos — todos los roles pueden leer
Workspace:  datos_publicos
Layer:      *
Mode:       Read
Roles:      ROLE_VISOR_PUBLICO, ROLE_CATASTRO, ROLE_PLANIFICACION, ROLE_DIRECTOR, ROLE_ADMIN

# 2. Catastro — solo catastro y director
Workspace:  catastro
Layer:      *
Mode:       Read
Roles:      ROLE_CATASTRO, ROLE_DIRECTOR, ROLE_ADMIN

# 3. Catastro — solo catastro puede escribir
Workspace:  catastro
Layer:      *
Mode:       Write
Roles:      ROLE_CATASTRO, ROLE_ADMIN

# 4. Planificación — solo planificador y director
Workspace:  planificacion
Layer:      *
Mode:       Read
Roles:      ROLE_PLANIFICACION, ROLE_DIRECTOR, ROLE_ADMIN

# 5. Planificación — solo planificador puede escribir
Workspace:  planificacion
Layer:      *
Mode:       Write
Roles:      ROLE_PLANIFICACION, ROLE_ADMIN
```

### Capa individual restringida:

```
# 6. Avalúos — solo director y admin (ni siquiera el técnico de catastro)
Workspace:  catastro
Layer:      avaluos
Mode:       Read
Roles:      ROLE_DIRECTOR, ROLE_ADMIN
```

> **Tip:** Esta regla es más específica (workspace + capa) que la regla 2 (workspace + *), así que tiene prioridad. El técnico de catastro puede ver parcelas y propietarios pero NO avalúos.

---

## Parte 5 — Reglas de servicios (10 min)

> Security → Services

```
# WFS bloqueado para visor público
Service:  WFS
Method:   *
Roles:    ROLE_CATASTRO, ROLE_PLANIFICACION, ROLE_DIRECTOR, ROLE_ADMIN
→ ROLE_VISOR_PUBLICO NO está = no puede descargar datos

# WMS abierto para todos los roles (ya está por defecto)
```

> **Escenario real:** "El ciudadano puede VER el mapa de departamentos y ríos en la web. Pero NO puede descargar los shapefiles. Solo puede ver imágenes."

---

## Parte 6 — Pruebas completas (20 min)

### Prueba 1 — Ciudadano en ventana de incógnito

1. Login como `ciudadano_web`
2. ✅ Layer Preview: debe ver capas de `datos_publicos`
3. ❌ Layer Preview: NO debe ver capas de `catastro` ni `planificacion`
4. ❌ URL WFS: debe dar error 403 (Forbidden)

### Prueba 2 — Técnico catastro

1. Login como `tecnico_catastro`
2. ✅ Ve `datos_publicos` + `catastro` (parcelas, propietarios)
3. ❌ NO ve `catastro:avaluos` (regla más específica)
4. ❌ NO ve `planificacion`
5. ✅ Puede usar WFS para descargar parcelas

### Prueba 3 — Director

1. Login como `director`
2. ✅ Ve TODO (datos_publicos + catastro + planificacion + avaluos)
3. ❌ NO puede administrar (no tiene ROLE_ADMIN)
4. ✅ Puede usar WFS

### Prueba 4 — QGIS

1. Agregar WMS con credenciales de `ciudadano_web` → solo ve capas públicas
2. Agregar WMS con credenciales de `director` → ve todas las capas
3. Intentar WFS con `ciudadano_web` → error de acceso

---

## Actividades Extra — Para llenar tiempo

### Actividad A — Auditoría cruzada (10 min)

Pedir a los estudiantes que intercambien credenciales de usuario visor e intenten acceder a capas restringidas. Verificar que las restricciones realmente funcionan.

### Actividad B — ¿Qué pasa si...? (10 min)

Escenarios de discusión:
- "¿Qué pasa si un empleado se va de la empresa?" → Desactivar usuario (Enabled: false)
- "¿Qué pasa si necesitas dar acceso temporal?" → Crear usuario temporal, desactivar después
- "¿Qué pasa si hackean tu GeoServer?" → HTTPS, contraseñas fuertes, monitorear logs de Tomcat
- "¿Qué pasa si un usuario necesita más acceso?" → Asignarle un rol adicional

### Actividad C — Esquema de seguridad personalizado (15 min)

Cada estudiante diseña (en papel o texto) el esquema de seguridad para un proyecto de su país:
- ¿Qué workspaces necesita?
- ¿Qué roles?
- ¿Qué usuarios?
- ¿Qué reglas de datos y servicios?

Luego 2-3 comparten y el grupo da feedback.

---

### 💻 Práctica 06

1. Cambia la contraseña de admin
2. Crea al menos 3 roles: visor, editor, un rol personalizado
3. Crea al menos 3 usuarios, uno por rol
4. Configura reglas de datos: visor solo lee workspace público, editor lee y escribe todo
5. Restringe WFS para el rol visor
6. Prueba con ventana de incógnito cada usuario
7. Prueba desde QGIS con diferentes credenciales
8. (Extra) Crea una regla para una capa individual más restrictiva que la del workspace

### ✅ Checklist Clase 06

- [ ] Contraseña de admin cambiada
- [ ] Roles creados (mínimo 3)
- [ ] Usuarios con roles asignados (mínimo 3)
- [ ] Reglas de datos por workspace
- [ ] Regla de servicio WFS restringida
- [ ] Probado en ventana de incógnito
- [ ] Probado desde QGIS
- [ ] Entiendo mínimo privilegio

---

## 📝 Errores comunes

1. **Olvidar incluir ROLE_ADMIN en las reglas** → Si no incluyes ROLE_ADMIN, el admin pierde acceso a esa capa. Siempre incluye ROLE_ADMIN en todas las reglas.
2. **Regla genérica sobreescribe la específica** → Las reglas más específicas (workspace+capa) tienen prioridad sobre las genéricas (workspace+*). Pero si NO hay regla específica, aplica la genérica.
3. **Usuario no ve nada después de configurar reglas** → Verificar que el rol del usuario está en al menos una regla de datos. Sin regla = sin acceso.
4. **WFS funciona aunque lo restringas** → Verificar que la regla de servicio cubre TODAS las operaciones (Method: *), no solo GetFeature.
5. **Se bloqueó el admin** → Si accidentalmente bloqueas al admin, puedes editar los archivos de seguridad directamente en `data_dir/security/`. Busca `layers.properties` y `services.properties`.
6. **Contraseña olvidada** → En `data_dir/security/usergroup/default/users.xml` puedes ver y modificar usuarios. Las contraseñas están hasheadas pero puedes resetearlas.

## 💡 Tips de producción

- **Nunca compartas credenciales.** Cada persona tiene su propio usuario. Si alguien se va, desactivas SU usuario sin afectar a los demás.
- **Usa grupos de usuarios** si tienes muchos usuarios con los mismos roles. En vez de asignar roles individualmente, asignas el grupo.
- **Monitorea los logs de Tomcat** (`catalina.out`) para detectar intentos de acceso no autorizados.
- **Backup del directorio de datos** de GeoServer regularmente. Las configuraciones de seguridad están ahí.
- **En producción:** Configura HTTPS con un certificado Let's Encrypt (gratuito). En el Módulo 7 veremos cómo.