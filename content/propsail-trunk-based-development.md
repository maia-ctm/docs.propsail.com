# Desarrollo Basado en Trunk (Trunk-Based Development) en PropSail

**Documento interno – Lineamientos para todos los repositorios de PropSail**  
Stack: NestJS · Prisma · React · React Native · SDK TypeScript · MariaDB/MySQL · cPanel · GitHub · CI/CD

---

## 1. Propósito del documento

Este documento explica cómo aplicamos **Trunk-Based Development (TBD)** en PropSail:

- Qué es el desarrollo basado en trunk.
- En qué se diferencia de GitFlow.
- Cómo se aplica **en la práctica** con nuestro stack y modelo de trabajo.
- Qué reglas deben seguir los desarrolladores (incluyendo trainees).
- Cómo se conecta con:
  - **DevSecOps**
  - **CI/CD**
  - **Versionamiento semántico (SemVer)**
  - **Convención de commits (Conventional Commits)**

Este documento es de uso obligatorio para todos los repositorios de PropSail.

---

## 2. ¿Qué es Trunk-Based Development?

**Trunk-Based Development** (TBD) es una forma de trabajar con Git donde:

- Existe una única rama **central** (el *trunk*), normalmente `main`.
- Todo el equipo integra cambios en esa rama de forma **frecuente** (idealmente diario).
- Las ramas de trabajo son:
  - **pocas**,  
  - **de muy corta duración** (horas o 1–2 días),  
  - y se eliminan después de hacer *merge*.

Objetivo principal:

> Que la rama `main` esté **siempre estable, testeada y lista para ser desplegada a producción**.

---

## 3. Comparación: GitFlow vs Trunk-Based Development

### 3.1 GitFlow (modelo clásico)

Características típicas de **GitFlow**:

- Varias ramas “principales”: `main`, `develop`, `release/*`, `hotfix/*`, `feature/*`.
- **Ramas de larga duración**:
  - `develop` puede vivir años.
  - `release/*` puede estar semanas activa.
  - `feature/*` puede durar días/semanas.
- Mucho enfoque en:
  - control fuerte de quién hace *merge*,
  - ventanas de “congelamiento de código” (*code freeze*),
  - varios pasos para llegar a producción.

Problemas frecuentes:

- **Conflictos enormes** al fusionar ramas de larga vida.
- Se duplican esfuerzos: se arregla un bug en `release/*` y luego hay que repetir el fix en `develop`.
- Los *releases* se vuelven **pesados, lentos y frágiles**.
- Aparición de frases como:  
  > “No hagan merge, estamos en *code freeze*”.

### 3.2 Trunk-Based Development

En **TBD**:

- El foco es **una sola rama principal**: `main`.
- Las ramas de trabajo:
  - son **pequeñas**,
  - tienen **poca vida** (horas o 1–2 días),
  - se usan principalmente para:
    - *Pull Requests* pequeños,
    - revisión de código,
    - correr CI.
- Se intenta que `main`:
  - **nunca esté roto**,
  - siempre tenga el código listo para un *release*.

Beneficios:

- Menos conflictos grandes al fusionar.
- *Releases* más rápidos y frecuentes.
- Todo el equipo trabaja con el código más reciente.
- Mucha sinergia con:
  - **Integración Continua (CI)**
  - **Entrega Continua (CD)**
  - **Feature flags** (para ocultar funcionalidad incompleta).

---

## 4. Principios clave de Trunk-Based Development

En PropSail, aplicamos TBD con los siguientes principios:

1. **Una sola rama principal: `main`**  
   - Es el *tronco* del árbol.
   - Todo lo importante llega ahí.
   - Siempre debe estar lista para desplegarse.

2. **Commits pequeños y frecuentes**  
   - Nada de “mega commit” con 40 archivos y 3 semanas de trabajo.
   - Pequeños pasos, cada cambio debe:
     - compilar,
     - pasar tests,
     - no romper la app.

3. **Ramas de corta duración**  
   - Usamos ramas tipo:
     - `feature/backend-auth-2fa`
     - `fix/web-map-center`
   - Duración esperada:
     - entre **unas horas y 1–2 días**.
   - Después del merge a `main` → **se elimina**.

4. **Nada de ramas de larga vida**  
   - No usamos `develop` ni `release` como ramas “permanentes”.
   - Si se crea una rama de release, es:
     - **justo a tiempo**,
     - para estabilizar una versión concreta,
     - y se elimina cuando ya no se use.

5. **`main` siempre verde**  
   - `main` debe:
     - compilar,
     - pasar los tests automáticos,
     - ser desplegable.
   - Si se rompe `main`, la prioridad es **arreglarla**.

6. **Feature flags, no ramas eternas**  
   - Nuevas funcionalidades se pueden integrar en `main`, pero:
     - ocultas detrás de **feature flags** (banderas de funcionalidad),
     - para no afectar a usuarios hasta que estén listas.

7. **Revisión continua de código**  
   - Preferimos:
     - *Pull Requests* pequeños,
     - o *pair programming* (dos personas en la misma tarea).
   - Los cambios pequeños son más fáciles de revisar y entender.

---

## 5. Cómo aplicamos TBD en PropSail (día a día)

### 5.1 Repositorios y tronco

En PropSail trabajamos con varios repositorios, por ejemplo:

- `propsail-backend` (NestJS + Prisma + MariaDB)
- `propsail-web` (React, frontend web)
- `propsail-mobile` (React Native, app móvil)
- `propsail-sdk` (SDK TypeScript para compartir lógica)
- Otros (infra, scripts, etc.)

En **todos** ellos:

- La rama principal es: `main`.
- No usamos ramas `develop` permanentes.
- Si hay un `release/*`, debe ser corto y con reglas claras (ver más abajo).

### 5.2 Flujo de trabajo recomendado por persona

Para una tarea típica (user story, bug, mejora):

1. **Actualizar la rama principal local**

   ```bash
   git checkout main
   git pull origin main
   ```

2. **Crear una rama de trabajo corta**

   Convención recomendada:

   ```bash
   git checkout -b feature/backend-auth-2fa
   # o
   git checkout -b fix/web-auth-token-renew
   ```

3. **Desarrollar en pequeños pasos**

   - Implementar una parte pequeña de la funcionalidad.
   - Asegurarse de que:
     - el proyecto compila,
     - los tests relevantes pasan.

4. **Commits con Conventional Commits**

   Ejemplos:

   ```bash
   git commit -m "feat(backend-auth): add 2fa token endpoint"
   git commit -m "fix(backend-auth): handle expired token error"
   ```

5. **Subir la rama y abrir Pull Request**

   ```bash
   git push -u origin feature/backend-auth-2fa
   ```

   - Abrir PR contra `main`.
   - PR pequeño, fácil de revisar.

6. **CI y revisión de código**

   - La pipeline de CI corre:
     - lint,
     - tests unitarios,
     - tests básicos de integración.
   - Un compañero o líder revisa los cambios.

7. **Merge rápido a `main`**

   - Si todo está OK:
     - Se hace *merge* (idealmente squash/rebase para mantener la historia limpia).
   - Luego se **elimina la rama**:

   ```bash
   git branch -d feature/backend-auth-2fa        # local
   git push origin --delete feature/backend-auth-2fa  # remoto
   ```

8. **Repetir con la siguiente tarea**

---

## 6. Feature Flags en el stack de PropSail

Las **feature flags** permiten:

- Integrar código de una funcionalidad que aún **no está lista para los usuarios**.
- Activarla/desactivarla según:
  - entorno (dev/qa/prod),
  - tipo de usuario,
  - porcentaje de tráfico.

### 6.1 Backend (NestJS + Prisma)

Opciones típicas:

- Tabla `feature_flags` en la base de datos.
- Servicio de configuración en NestJS, por ejemplo:

```ts
// pseudo-ejemplo
@Injectable()
export class FeatureFlagService {
  async isEnabled(flag: string, userId?: string): Promise<boolean> {
    // consultar BD, cache, config, etc.
    return true;
  }
}
```

Uso en un controlador:

```ts
if (!(await this.featureFlagService.isEnabled('new-login-flow'))) {
  // usar lógica antigua
} else {
  // usar nueva lógica
}
```

### 6.2 Web (React) y Mobile (React Native)

- Recibir flags desde:
  - el backend vía API,
  - o un archivo de configuración por entorno.
- Usar hooks/helpers:

```ts
const { isEnabled } = useFeatureFlags();

if (isEnabled('new-login-flow')) {
  return <NewLogin />;
}

return <LegacyLogin />;
```

Con esto:

- Se puede **integrar** la nueva lógica en `main`.
- Pero **no mostrarla** hasta que esté lista.

---

## 7. Pruebas automatizadas y CI/CD

Para que Trunk-Based Development funcione bien, necesitamos:

1. **Pruebas automatizadas**:
   - Unit tests (NestJS, React, React Native, SDK).
   - Tests de integración básicos.
2. **Pipelines de CI**:
   - Se ejecutan en cada PR y en `main`.
   - Si fallan, el cambio no se debería fusionar.
3. **Builds rápidas**:
   - Mantener pasos ligeros.
   - Cachear dependencias cuando sea posible.

Ejemplo de checklist CI para cada repo:

- Backend:
  - `npm run lint`
  - `npm run test`
  - `npm run test:e2e` (si está disponible)
- Web:
  - `npm run lint`
  - `npm run test`
  - `npm run build`
- Mobile:
  - `npm run lint`
  - `npm run test`

---

## 8. Estrategia de releases con Trunk-Based Development

### 8.1 Ideal: release desde `main`

En el escenario ideal:

- `main` siempre está **estable y listo para producir**.
- Cuando se decide liberar una versión (ej: `1.3.0`):
  - Se toma una **tag** sobre `main`:

    ```bash
    git checkout main
    git pull origin main
    git tag -a v1.3.0 -m "Release 1.3.0"
    git push origin v1.3.0
    ```

  - Se despliega esa versión usando la tag (o simplemente el último commit de `main`).

### 8.2 Rama de release (solo si es necesaria)

Si por motivos de negocio se requiere una rama de release:

- Se crea justo antes de estabilizar:

  ```bash
  git checkout main
  git pull origin main
  git checkout -b release/1.3.0
  git push -u origin release/1.3.0
  ```

Reglas importantes:

- En `release/1.3.0`:
  - **no se desarrollan nuevas features**,
  - solo se aplican **fixes**.
- Los bugs se corrigen **primero en `main`** y luego se hace **cherry-pick** hacia `release/1.3.0`:

  ```bash
  # bugfix en main
  git checkout main
  # ... fix, commit, push ...

  # cherry-pick a release
  git checkout release/1.3.0
  git cherry-pick <hash-del-fix>
  git push
  ```

- Cuando el ciclo de soporte de esa versión termina:
  - se deja de hacer cherry-pick,
  - y la rama `release/1.3.0` se puede eliminar.

> **Lo que NO hay que hacer**:  
> Corregir un bug **solo** en `release/1.3.0` y olvidar arreglarlo en `main`.  
> Eso genera regresiones en futuras versiones.

---

## 9. Buenas prácticas en Trunk-Based Development (PropSail)

### 9.1 Desarrollar en lotes pequeños

- Tareas pequeñas, PRs pequeños.
- Evitar PRs gigantes donde:
  - nadie entiende todo,
  - la revisión se vuelve lenta,
  - aumentan los bugs.

### 9.2 Mantener pocas ramas activas

- Idealmente, no más de **3 ramas activas** por repo.
- Las ramas que ya se fusionaron:
  - se eliminan en remoto y local.

### 9.3 Revisiones de código asíncronas y rápidas

- En TBD no sirve tener PRs abiertos una semana.
- Objetivo:
  - Revisar y fusionar PRs pequeños **lo antes posible**.
- Para equipos pequeños:
  - se puede combinar PRs con *pair programming*.

### 9.4 Sin congelamiento de código (code freeze)

- En TBD, el ideal es **no tener code freeze**.
- En vez de frenar el desarrollo:
  - se confía en CI,
  - en los tests,
  - y en la calidad de `main`.

---

## 10. Lo que estamos haciendo mal (antipatrones típicos)

### 10.1 Ramas de funcionalidades de larga duración

Mal patrón:

- Ramas `feature/*` que duran semanas o meses.
- Muchos desarrolladores modificando la misma rama.
- Fusiones llenas de conflictos.

Regla TBD:

- Ramas de feature: **1–2 días máximo**.
- Igual o más de un par de días → huele a problema.

### 10.2 Más de un desarrollador por rama de feature

Mal patrón:

- Una rama `feature/*` con 3–4 personas empujando código días y días.

Problemas:

- Esa rama se convierte en una especie de “mini `develop`”.
- Se rompe la idea de “rama corta, cambio focalizado”.

Regla TBD:

- Una rama de feature la lleva:
  - una persona,
  - o un par (*pair programming*).
- Si hay más gente, probablemente hay que **dividir la tarea**.

### 10.3 Arreglar bugs solo en la rama de release

Mal patrón:

- Bug se detecta en `release/1.2.0`.
- Se arregla **solo** ahí.
- No se corrige en `main`.

Consecuencia:

- En la siguiente versión (`1.3.0`), el bug reaparece.

Regla TBD:

- Siempre que se pueda:
  - reproducir bug en `main`,
  - arreglar primero en `main`,
  - luego cherry-pick a la(s) rama(s) de release que sigan vivas.

### 10.4 Congelar a todo el equipo por el release

Mal patrón:

- “Nadie haga merge, estamos en code freeze”.
- Todo el equipo se siente bloqueado.

Regla TBD:

- La mayor parte del equipo **sigue trabajando normalmente en `main`**.
- Un subconjunto pequeño:
  - se encarga de issues específicos del release,
  - gestiona la rama de release (si existe).

---

## 11. Relación con CI, CD y DevOps

### 11.1 Infraestructura de desarrollo sólida

Antes de aplicar TBD en serio, el equipo necesita:

- Repositorios claros en GitHub.
- Estaciones de trabajo preparadas (local, VM o contenedor).
- Entorno reproducible (Infraestructura como Código cuando sea posible).
- Scripts de:
  - instalación,
  - tests,
  - build.

### 11.2 Integración Continua (CI)

TBD y CI son casi inseparables:

- TBD asegura que `main` recibe cambios frecuentes.
- CI:
  - compila,
  - corre tests,
  - alerta si algo se rompe.

Requisito básico:

> Cada desarrollador debería integrar sus cambios a `main` (directo o vía PR) **al menos una vez cada 24h**.

### 11.3 Entrega Continua (CD)

Con:

- `main` siempre estable.
- Tests automatizados.
- Procesos de build confiables.

Es posible:

- Desplegar a ambientes:
  - dev,
  - qa,
  - e incluso prod,
- De forma más frecuente y segura.

TBD facilita que CD sea una **realidad**, no solo un concepto.

---

## 12. Checklist para el equipo de PropSail

Antes/durante una tarea, revisa:

- [ ] ¿Estoy trabajando en una **rama corta** basada en `main`?
- [ ] ¿Mis cambios son **pequeños y enfocados**?
- [ ] ¿Estoy usando mensajes de commit con **Conventional Commits**?
- [ ] ¿He corrido los tests antes de hacer push?
- [ ] ¿Mi PR es pequeño y fácil de revisar?
- [ ] ¿Mi cambio mantiene `main` estable y desplegable?
- [ ] Si estoy agregando una feature grande, ¿usé **feature flags** para no exponerla antes de tiempo?
- [ ] Si arreglé un bug que afecta a una release, ¿lo arreglé en `main` y luego lo cherry-pickeé a la rama de release?

Si este checklist se cumple de forma consistente, estamos aplicando Trunk-Based Development correctamente en PropSail.
