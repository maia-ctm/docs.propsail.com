# Convención de Commits en PropSail (Conventional Commits)

**Documento interno – Lineamientos para todos los repositorios PropSail**  
Stack: NestJS · Prisma · React · React Native · SDK TypeScript · MariaDB/MySQL · cPanel

---

## 1. Propósito

Este documento define **cómo debemos escribir los mensajes de commit** en todos los repositorios de PropSail, usando la especificación de **Conventional Commits**.

Objetivos:

- Que todo el equipo use **el mismo formato** de commit.
- Que el historial de Git sea **legible** para humanos y máquinas.
- Poder **automatizar**:
  - generación de CHANGELOG,
  - versionado (SemVer),
  - procesos de CI/CD.

Esta convención se usa en:

- Backend (NestJS + Prisma)
- Frontend Web (React)
- Mobile (React Native)
- SDK TypeScript
- Librerías internas y herramientas de DevOps

---

## 2. Resumen rápido: Conventional Commits + SemVer

Conventional Commits propone un formato estándar para los mensajes de commit:

```text
<tipo>(<ámbito opcional>): <descripción corta>
<LINEA EN BLANCO>
[cuerpo opcional]
<LINEA EN BLANCO>
[nota(s) al pie opcional(es)]
```

Se relaciona directamente con **SemVer**:

- `fix` → cambios de tipo **PATCH**.
- `feat` → cambios de tipo **MINOR**.
- Commits con `BREAKING CHANGE` → cambios de tipo **MAJOR**, sin importar el tipo.

> Para más detalle de SemVer, ver `propsail-semver.md`.

---

## 3. Formato del mensaje de commit

Formato completo:

```text
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

- **Header** (cabecera) – **obligatorio**:
  - `type` (tipo) → qué clase de cambio es.
  - `scope` (ámbito) → qué parte del sistema toca (opcional).
  - `subject` (descripción corta) → qué se hizo.
- **Body** (cuerpo) – opcional:
  - Explica el “por qué” y el contexto.
- **Footer** (nota al pie) – opcional:
  - `BREAKING CHANGE: ...`
  - Referencias a issues, tickets, etc.

Reglas generales:

- Máximo recomendado **100 caracteres por línea**.
- Idioma: puedes usar **español o inglés**, pero sé consistente dentro del repo.
- No mezclar muchos cambios distintos en un solo commit: **pequeños y enfocados**.

---

## 4. Tipos de commit en PropSail

Usaremos los tipos recomendados por la convención (inspirados en Angular) y la especificación de Conventional Commits.

> 🧠 **Recuerda:**  
> - `feat` → se relaciona con SemVer **MINOR**  
> - `fix` → se relaciona con SemVer **PATCH**  
> - `BREAKING CHANGE` → se relaciona con SemVer **MAJOR**

### 4.1 `feat` – Nueva funcionalidad

Cuando se **añade una nueva funcionalidad**.

Ejemplos:

- `feat(backend-auth): add 2FA token endpoint`
- `feat(mobile-profile): allow users to upload avatar`

Impacto SemVer habitual: **MINOR**.

---

### 4.2 `fix` – Corrección de errores

Cuando se **corrige un bug**.

Ejemplos:

- `fix(backend-auth): handle expired refresh token properly`
- `fix(web-filters): fix price range slider issue`

Impacto SemVer habitual: **PATCH**.

---

### 4.3 `docs` – Documentación

Cambios en **documentación**, sin afectar código ejecutable.

Ejemplos:

- `docs(semver): add versioning guidelines for PropSail`
- `docs(readme): update local setup instructions`

---

### 4.4 `style` – Estilo de código (sin cambiar lógica)

Cambios de **formato y estilo**, que **no cambian el comportamiento**:

- Espacios.
- Indentación.
- Comillas simples/dobles.
- Puntos y comas, etc.

Ejemplos:

- `style(backend): format auth controller with prettier`
- `style(web-ui): normalize imports order`

---

### 4.5 `refactor` – Refactor de código

Cambios que **no añaden funcionalidad** ni **corrigen bugs**, pero mejoran el diseño interno del código.

Ejemplos:

- `refactor(sdk): extract http client to shared module`
- `refactor(backend-search): simplify query builder`

---

### 4.6 `perf` – Rendimiento

Cambios orientados a **mejorar la performance**.

Ejemplos:

- `perf(backend-search): add indexes to listing queries`
- `perf(mobile-list): virtualize property list rendering`

---

### 4.7 `test` – Tests

Añadir o corregir **tests**.

Ejemplos:

- `test(backend-auth): add unit tests for login flow`
- `test(sdk): fix flaky integration test`

---

### 4.8 `build` – Sistema de build / dependencias

Cambios que afectan al **build** o dependencias:

- Configuración de bundlers.
- Dependencias de npm.
- Scripts de build.

Ejemplos:

- `build(ci): bump node version to 22.x`
- `build(repo): add standard-version as dev dependency`

---

### 4.9 `ci` – Integración continua

Cambios en **CI/CD**:

- Workflows de GitHub Actions (o la plataforma que se use).
- Pipelines.
- Scripts de deployment.

Ejemplos:

- `ci(backend): add test step to main pipeline`
- `ci(release): integrate semantic-release for sdk`

---

### 4.10 `chore` – Tareas rutinarias

Tareas que no son feature ni bugfix, ni cambian código de negocio:

- Actualizar `.gitignore`.
- Ajustar configuración de linters.
- Limpieza de archivos.

Ejemplos:

- `chore(repo): add editorconfig file`
- `chore(backups): ignore tmp backup files`

---

### 4.11 `revert` – Revertir un commit

Cuando se revierte un commit anterior.

Formato recomendado:

```text
revert: <header del commit que se revierte>

This reverts commit <hash>.
```

Ejemplo:

```text
revert: feat(mobile-auth): add tenant login

This reverts commit 1234abcd.
```

---

## 5. Ámbitos (`scope`) en PropSail

El **scope** es opcional, pero muy recomendable.  
Sirve para indicar **qué parte** del sistema se toca.

Ejemplos de scopes útiles en PropSail:

- `backend-auth`
- `backend-properties`
- `sdk`
- `web-auth`
- `web-dashboard`
- `mobile-auth`
- `mobile-map`
- `infra`
- `devops`
- `docs`
- `security`

Ejemplos:

- `feat(backend-auth): add 2FA via email token`
- `fix(web-auth): handle invalid token redirect`
- `refactor(sdk): normalize error handling`
- `ci(devops): add lint job to pipeline`

> Si el cambio es muy transversal (afecta a todo el repo), el scope puede omitirse:  
> `style: apply formatter to all files`

---

## 6. Descripción (subject)

La **descripción corta** debe:

- Usar **imperativo y tiempo presente**:
  - ✅ “add”, “update”, “fix”
  - ❌ “added”, “fixed”
- Comenzar en **minúscula**.
- No terminar en punto (`.`).
- Ser breve y clara (< 100 caracteres).

Ejemplos correctos:

- `feat(web-auth): add reset password screen`
- `fix(backend-search): handle empty filters`
- `docs(readme): document env variables`

---

## 7. Cuerpo del commit (body)

El **cuerpo** es opcional, pero muy útil cuando:

- El cambio es complejo.
- Se necesita explicar contexto/motivación.
- Se requiere comparar comportamiento nuevo vs anterior.

Reglas:

- También en **imperativo/presente**:
  - “change”, “add”, “remove”.
- Explica el **por qué**, no solo el “qué”.

Ejemplo:

```text
fix(backend-auth): handle expired tokens correctly

previously the backend returned 500 when refresh token was expired.
now it returns 401 with a specific error code so clients can redirect
to login screen gracefully.
```

---

## 8. Notas al pie (footer) y BREAKING CHANGE

La **nota al pie** se usa para:

- Marcar cambios que **rompen compatibilidad**.
- Referenciar issues o tickets.

### 8.1 BREAKING CHANGE

Si el commit introduce un cambio incompatible (MAJOR), se debe indicar:

```text
BREAKING CHANGE: <descripción clara del cambio>
```

Ejemplo:

```text
feat(backend-auth): require email verification before login

BREAKING CHANGE: /auth/login now returns 403 for users without verified email.
clients must handle this status and show appropriate message.
```

También se puede marcar una ruptura con `!` después del tipo:

```text
feat!(backend-auth): remove legacy login endpoint
```

> **Regla:** cualquier commit con `BREAKING CHANGE` se considera MAJOR a nivel SemVer.

### 8.2 Referencias a issues

Se pueden usar referencias estándar:

```text
fix(web-map): center map on user location

fixes #123
```

o

```text
fix: correct minor typos in code

see the issue for details on the typos fixed

Refs #133
```

---

## 9. Relación con SemVer (resumen)

Recordatorio de mapeo entre Conventional Commits y SemVer:

- `fix` → **PATCH**  
  Ej: `1.2.3 → 1.2.4`
- `feat` → **MINOR**  
  Ej: `1.2.3 → 1.3.0`
- `BREAKING CHANGE` (en body o footer, o `!` en el tipo) → **MAJOR**  
  Ej: `1.2.3 → 2.0.0`

Otros tipos (`docs`, `style`, `refactor`, etc.) **no cambian** SemVer por sí solos, salvo que incluyan un `BREAKING CHANGE`.

---

## 10. Ejemplos completos en contexto PropSail

### 10.1 Feature sin ruptura

```text
feat(mobile-auth): add login screen for tenants
```

### 10.2 Bugfix con detalle

```text
fix(backend-auth): handle invalid jwt error

previously the api returned 500 when the jwt was malformed.
now it returns 401 with an explicit error code so clients
can redirect user to login.
```

### 10.3 Documentación

```text
docs(semver): add semantic versioning guidelines for propsail
```

### 10.4 Refactor

```text
refactor(sdk): extract http client to separate module
```

### 10.5 Cambio que rompe compatibilidad

```text
feat(backend-search): change listings default sorting

BREAKING CHANGE: /properties endpoint now sorts by createdAt desc
instead of price asc. clients that rely on old behavior must
pass an explicit sort parameter.
```

---

## 11. Herramientas recomendadas (JavaScript/TypeScript)

Existen varias herramientas que entienden Conventional Commits y permiten automatizar tareas. Algunas relevantes para nuestro stack:

- **commitlint**  
  Linter para mensajes de commit. Permite **validar** el mensaje antes de aceptarlo.  
  Suele integrarse con hooks tipo `husky` o `simple-git-hooks`.

- **standard-version**  
  Herramienta que:
  - Lee el historial de commits.
  - Calcula la **nueva versión** (SemVer) según `feat`, `fix`, `BREAKING CHANGE`.
  - Genera o actualiza el `CHANGELOG`.
  - Actualiza `package.json` y crea un commit de release.

- **semantic-release**  
  Va más allá:
  - Automatiza el flujo completo de releases.
  - Trabaja con ramas (`main`, `next`, `beta`, `alpha`, etc.).
  - Analiza commits, genera notas de release, crea tags y puede publicar paquetes.

- **multi-semantic-release**  
  Variante pensada para **monorepos**, para versionar varios paquetes en un mismo repo.

> Integrar estas herramientas es una tarea de DevOps/Arquitectura, pero su eficacia depende totalmente de que el **equipo respete la convención** de commits.

---

## 12. Preguntas frecuentes (FAQ)

### 12.1 ¿Qué pasa en etapas iniciales del proyecto?

Aunque el proyecto esté “verde”, es recomendable **usar Conventional Commits desde el principio**:

- El historial será mucho más claro cuando el proyecto crezca.
- Ayuda a entender decisiones y cambios con el tiempo.

### 12.2 ¿Qué hago si un commit encaja en varios tipos?

Lo ideal es **separar en varios commits**:

- Un commit por bugfix (`fix`).
- Otro commit para refactor (`refactor`).
- Otro commit para docs (`docs`).

Esto hace que el historial sea más limpio y fácil de leer.

### 12.3 ¿Esto no hace más lento el desarrollo?

Hace más lento el caos 😄.  
En la práctica:

- Obliga a pensar un poco en **qué se está cambiando**.
- Facilita la vida del propio equipo al leer el historial.
- Permite automatizar tareas de release y changelog.

### 12.4 ¿Qué pasa si uso un tipo incorrecto?

- Si aún no se ha mergeado:
  - Puedes usar `git commit --amend` o `git rebase -i` para corregir.
- Si ya se liberó:
  - Dependerá del flujo, pero en general no es dramático; simplemente esa herramienta (standard-version / semantic-release) puede interpretar mal ese cambio.

### 12.5 ¿Cómo se relaciona con SemVer?

- `fix` → PATCH  
- `feat` → MINOR  
- `BREAKING CHANGE` → MAJOR  

Esta relación permite que las herramientas **calculen versiones automáticamente**.

---

## 13. Checklist para el equipo de PropSail

Antes de hacer un commit, revisa:

- [ ] ¿Elegí el **tipo** correcto? (`feat`, `fix`, `docs`, etc.)
- [ ] ¿Puedo añadir un **scope** útil? (`backend-auth`, `sdk`, `web-auth`, etc.)
- [ ] ¿La descripción está en **imperativo**, en **minúscula** y sin punto final?
- [ ] ¿El mensaje es **claro** para alguien que no vio el código?
- [ ] Si hay un cambio incompatible, ¿agregué `BREAKING CHANGE:` en el body/footer?
- [ ] ¿Se podría dividir el cambio en varios commits más específicos?

Si la respuesta es “sí” a todo, el commit está listo para entrar al historial de PropSail 👌