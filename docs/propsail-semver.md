# Versionamiento Semántico (SemVer) en PropSail

**Documento interno – Lineamientos para todos los repositorios PropSail**  
Stack: NestJS · Prisma · React · React Native · SDK TypeScript · MariaDB/MySQL · cPanel

---

## 1. Propósito de este documento

Este documento define **cómo usamos el Versionamiento Semántico (SemVer)** en PropSail para todos los proyectos:

- Backend (NestJS + Prisma)
- Frontend Web (React)
- Mobile (React Native)
- SDK TypeScript
- Librerías internas o paquetes compartidos

El objetivo es que **cualquier persona del equipo**, al ver un número de versión, pueda entender:

- Qué tipo de cambio se hizo.
- Qué tan riesgoso es actualizar.
- Si hay o no **ruptura de compatibilidad** (breaking changes).

---

## 2. ¿Qué es SemVer?

SemVer (Semantic Versioning) es una convención de nomenclatura de versiones donde una versión se escribe como:

```text
MAJOR.MINOR.PATCH
ejemplos: 1.0.0 · 2.1.3 · 14.2.9
```

A veces también se denomina `X.Y.Z`.

### 2.1 Significado de cada número

- **PATCH (Z)**  
  Se incrementa cuando se hace una **corrección** de errores:
  - No se agregan nuevas funcionalidades.
  - No se rompen contratos públicos.
  - Ejemplos:
    - `1.0.1 → 1.0.2`
    - `2.1.0 → 2.1.1`
    - `14.2.9 → 14.2.10`

- **MINOR (Y)**  
  Se incrementa cuando se **agrega una funcionalidad nueva**, pero **sin romper compatibilidad**:
  - Se pueden añadir endpoints nuevos, parámetros opcionales, pantallas nuevas, etc.
  - El número de `PATCH` se resetea a cero.
  - Ejemplos:
    - `1.0.1 → 1.1.0`
    - `2.1.0 → 2.2.0`
    - `14.2.9 → 14.3.0`

- **MAJOR (X)**  
  Se incrementa cuando se introduce un **cambio incompatible**:
  - Se rompen contratos públicos (API, SDK, modelos, etc.).
  - Requiere ajustes en los consumidores (frontend, mobile, integraciones).
  - `MINOR` y `PATCH` se resetean a cero.
  - Ejemplos:
    - `1.0.1 → 2.0.0`
    - `2.1.0 → 3.0.0`
    - `14.2.9 → 15.0.0`

> En SemVer, “compatible” significa: **el código que funcionaba con la versión anterior sigue funcionando sin cambios**.

---

## 3. Pre-releases y etiquetas adicionales

Además de `X.Y.Z`, se pueden usar etiquetas adicionales para indicar el **estado de estabilidad**:

- `-dev`   → código en desarrollo, no apto para producción.  
  Ej: `1.0.1-dev`
- `-alpha` → primeras pruebas, puede estar incompleto o inestable.  
  Ej: `1.0.1-alpha`
- `-beta`  → más estable que alpha, pero aún en pruebas.  
  Ej: `2.1.0-beta`
- `-RC1`, `-RC2` → “Release Candidate” (candidatos a release).  
  Ej: `0.3.0-RC1`, `0.3.0-RC2`  
- **Stable** → no se indica explícitamente; versiones sin sufijo se consideran estables.  
  Ej: `1.0.1`, `2.1.0`, `14.2.9`

> Por convención, todo lo que esté **por debajo de `1.0.0`** se considera **pre-release** (el producto aún no es estable).

### 3.1 Punto de partida recomendado para PropSail

- Para servicios/librerías **en desarrollo** (sin producción):  
  `0.1.0`
- Para el **primer release en producción**:  
  `1.0.0`

---

## 4. ¿Por qué es importante SemVer en PropSail?

1. **Comunicación clara**  
   El número de versión es un mensaje al resto del equipo:
   - `1.2.3 → 1.2.4` → bugfix  
   - `1.2.3 → 1.3.0` → nueva funcionalidad compatible  
   - `1.2.3 → 2.0.0` → cuidado, hay cambios incompatibles

2. **Multi-repo y multi-cliente**  
   PropSail trabaja con:
   - Backend NestJS
   - SDK TypeScript
   - Frontend React
   - Mobile React Native  
   Un cambio en el backend o el SDK puede afectar varios repos. SemVer permite ver **rápido** el impacto.

3. **Gestores de dependencias (npm, etc.)**  
   Node/npm usan SemVer para decidir **qué versiones instalar** según las reglas de rango.  
   Esto afecta directamente cómo frontend/mobile usan el SDK, y cómo el backend depende de paquetes externos.

4. **Changelogs claros**  
   SemVer se complementa muy bien con **changelogs**, documentos donde se resume qué se cambió en cada versión.

---

## 5. Cómo se aplica SemVer en el stack de PropSail

### 5.1 Backend (NestJS + Prisma)

Repositorio ejemplo: `PropSail/propsail-backend-nest`

- Cambios **PATCH** (`X.Y.Z → X.Y.(Z+1)`):
  - Corrección de bugs sin cambiar la firma de endpoints.
  - Ajustes en validaciones sin romper clientes.
  - Cambios internos de implementación sin afectar la API pública.

- Cambios **MINOR** (`X.Y.Z → X.(Y+1).0`):
  - Nuevos endpoints que **no rompen** los existentes.
  - Nuevos parámetros **opcionales** en endpoints.
  - Nuevos campos en respuestas que no rompen consumo existente (los clientes pueden ignorarlos).

- Cambios **MAJOR** (`X.Y.Z → (X+1).0.0`):
  - Eliminar endpoints existentes.
  - Cambiar nombres de campos esperados por clientes.
  - Modificar comportamiento de la API de forma que el código actual del frontend/mobile deje de funcionar.
  - Cambios de modelo de datos que exijan actualización sí o sí del SDK y los clientes.

### 5.2 SDK TypeScript

Repositorio ejemplo: `PropSail/propsail-sdk-ts`

El SDK es **crítico** porque es el contrato que consumen frontend y mobile.

- PATCH:
  - Corrección de tipos sin cambiar las firmas.
  - Fix a un método que antes fallaba.
- MINOR:
  - Nuevos métodos o recursos (nuevos endpoints mapeados).
  - Nuevos parámetros opcionales.
- MAJOR:
  - Renombrar métodos o cambiar los parámetros obligatorios.
  - Cambiar tipos de retorno de forma incompatible.
  - Eliminar métodos.

> Regla práctica: **si algo rompe la compilación o el runtime de proyectos que usaban la versión anterior sin cambios, es un MAJOR.**

### 5.3 Frontend y Mobile

Aunque el frontend y la app mobile no son “librerías” para terceros, también es útil aplicar SemVer a las releases internas:

- PATCH:
  - Fix de bugs visuales.
  - Corrección de textos.
  - Ajustes menores que no cambian flujos.
- MINOR:
  - Nuevas pantallas o componentes.
  - Nuevos flujos que no rompen uso actual.
- MAJOR:
  - Rediseños que cambian drásticamente la navegación.
  - Eliminación de funciones que los usuarios ya usan.

---

## 6. SemVer y gestores de dependencias (npm, etc.)

Los gestores de dependencias (npm, yarn, pnpm) usan SemVer para resolver versiones.

Algunas formas habituales de especificar versiones:

### 6.1 Versión exacta

```json
"@propsail/sdk": "2.1.3"
```

- Se instala exactamente la `2.1.3`.

### 6.2 Rango de versiones

```json
"@propsail/sdk": ">=2.1.0 <2.2.0"
```

- Cualquier versión `2.1.x` sirve.  
- Nunca instalará `2.2.0` o superior.

### 6.3 Intervalo con guiones

```json
"@propsail/sdk": "2.1.0 - 2.3.0"
```

- Cualquier versión entre `2.1.0` y `2.3.0` (incluidas).

### 6.4 Comodines (`*`)

```json
"@propsail/sdk": "2.1.*"
```

- Cubre `2.1.0` hasta `2.1.n`.

```json
"@propsail/sdk": "2.*"
```

- Cubre `2.0.0` hasta `2.n`.

### 6.5 Tilde (`~`)

```json
"@propsail/sdk": "~2.1.0"
```

- `>= 2.1.0` y `< 2.2.0` (todas las `2.1.x`).

```json
"@propsail/sdk": "~2.1"
```

- `>= 2.1.0` y `< 3.0.0` (todas las versiones desde `2.1.0` hasta antes de `3.0.0`, según interpretación de la herramienta y la forma exacta usada).

*(La idea general: **la tilde suele fijar minor y permitir patches**, dependiendo del formato usado.)*

### 6.6 Caret (`^`)

```json
"@propsail/sdk": "^2.1.0"
```

- `>= 2.1.0` y `< 3.0.0`.  
- Permite todas las versiones **MINOR** y **PATCH** posteriores, pero no un cambio de **MAJOR**.

Esta es la forma más habitual para librerías que siguen SemVer correctamente, porque:

- Puedes recibir mejoras y fixes.
- No recibes cambios incompatibles.

> Para paquetes internos como `@propsail/sdk`, lo normal será usar `^X.Y.Z` y confiar en que el equipo respeta SemVer.

---

## 7. Cómo versionamos en la práctica (Git + npm)

### 7.1 Campo `version` en `package.json`

Ejemplo:

```json
{
  "name": "@propsail/backend",
  "version": "1.3.0",
  "description": "Backend API PropSail",
  "main": "dist/main.js"
}
```

- Este número debe seguir todas las reglas SemVer.
- Es la fuente de verdad para el ecosistema Node.

### 7.2 Tags en Git

Además de `package.json`, se crean **tags** en el repo para marcar releases:

```bash
git tag -a v1.3.0 -m "Release 1.3.0 - nuevas funcionalidades X, Y, Z"
git push origin v1.3.0
```

Convención:

- Prefijo `v` + versión SemVer  
  Ej: `v1.0.0`, `v2.1.3`, `v2.2.0-beta.1`

### 7.3 Uso de comandos npm

Atajo para manejar versiones:

```bash
# PATCH
npm version patch    # 1.0.0 → 1.0.1

# MINOR
npm version minor    # 1.0.1 → 1.1.0

# MAJOR
npm version major    # 1.1.3 → 2.0.0
```

Estos comandos:

- Actualizan `package.json`.
- Crean un tag automáticamente (si está configurado por defecto).
- Puedes luego hacer `git push --follow-tags`.

---

## 8. Política de versionado recomendado para PropSail

### 8.1 Punto de inicio

- **Proyectos en desarrollo (sin producción):** `0.1.0`
- **Primer release productivo:** `1.0.0`

### 8.2 Tabla guía de cambios

| Tipo de cambio                                    | Ejemplo de impacto                                           | Tipo SemVer | Ejemplo       |
|--------------------------------------------------|--------------------------------------------------------------|------------|---------------|
| Fix de bug interno                               | Corrige validación pero API igual                            | PATCH      | 1.0.3 → 1.0.4 |
| Cambio menor de texto/estilo en front            | Ajuste de UI sin cambiar flujos                              | PATCH      | 1.2.0 → 1.2.1 |
| Nuevo endpoint que no rompe nada                 | `/properties/:id/history` nuevo                              | MINOR      | 1.2.1 → 1.3.0 |
| Nuevos campos opcionales en una respuesta        | `User` ahora incluye `avatarUrl` (opcional)                  | MINOR      | 1.3.0 → 1.4.0 |
| Eliminar endpoint o campo requerido              | Se elimina `/legacy-auth` o un campo deja de existir         | MAJOR      | 1.4.2 → 2.0.0 |
| Cambiar tipo de dato de un campo existente       | `price` pasa de `string` a `number` en la API pública        | MAJOR      | 2.1.0 → 3.0.0 |
| Cambios en SDK que exigen tocar código cliente   | Rename de funciones, parámetros obligatorios nuevos, etc.    | MAJOR      | 0.9.0 → 1.0.0 |

---

## 9. Pre-releases en PropSail

Para pruebas internas o environments previos a producción, se pueden usar:

- `1.3.0-alpha.1`  
- `1.3.0-beta.1`  
- `1.3.0-rc.1`

Uso recomendado:

1. `-alpha`: prototipos, alcanza para QA interno muy temprano.
2. `-beta`: funcionalidad casi completa, pero pendiente de pulir.
3. `-rc.x`: candidato a release, se espera solo corrección de bugs menores.
4. Versión final estable: `1.3.0` (sin sufijo).

---

## 10. Buenas prácticas y checklist de equipo

### 10.1 Antes de subir una nueva versión

- [ ] ¿Identificaste si el cambio es **patch, minor o major**?
- [ ] ¿El número en `package.json` sigue SemVer?
- [ ] ¿Se creó/actualizó el **tag** correspondiente en Git?
- [ ] ¿Se actualizó el **CHANGELOG** (si aplica)?
- [ ] ¿Los proyectos que dependen de este repo saben si necesitan ajustar algo?
- [ ] ¿Se respetaron las reglas de compatibilidad (no romper minor/patch)?

### 10.2 Para Trainees

- Siempre pregunta:  
  > “¿Este cambio obliga a otros repos a cambiar código?”  
  - Si **sí** → probablemente es **MAJOR**.  
  - Si **no**, pero agrega cosas nuevas → **MINOR**.  
  - Si solo corrige algo → **PATCH**.

---

## 11. Conclusión

- SemVer es una forma **estándar y clara** de expresar el estado de un proyecto.
- En PropSail, SemVer es la base para:
  - Versionar backend, frontend, mobile y SDK.
  - Publicar releases internos de forma ordenada.
  - Actualizar dependencias sin miedo innecesario.
- Usar SemVer correctamente es parte de la **madurez técnica** del equipo y facilita el trabajo entre desarrollo, QA, operaciones y negocios.
