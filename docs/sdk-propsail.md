# SDK PropSail (TypeScript)
Guía para backend, frontend web y mobile (React Native)

> **Propósito:** definir cómo trabajaremos con un **SDK TypeScript común** para consumir la API de PropSail desde **backend (NestJS), frontend web y app mobile**, de forma tipada, consistente y segura.

---

## 1. ¿Qué es el SDK PropSail?

El **SDK PropSail** es una librería TypeScript/JavaScript (paquete npm) que:

- Se instala en:
  - Frontend web (React / Next.js, etc.).
  - App mobile (React Native).
  - Scripts o servicios Node.js que necesiten hablar con la API.
- Expone **funciones tipadas** por dominio de negocio (auth, usuarios, propiedades, etc.).
- Se **genera automáticamente** a partir del **OpenAPI/Swagger** del backend NestJS.
- Centraliza:
  - `baseURL` de la API.
  - Manejo de **JWT de acceso / refresh**.
  - Manejo de errores HTTP.
  - Tipos compartidos (`User`, `Property`, `Booking`, etc.).

**No es** otro backend, ni una capa mágica. Es simplemente:

> Un cliente HTTP tipado y compartido para hablar con la API de PropSail.

---

## 2. Repositorio y paquete

### 2.1. Nombre del repositorio

En la organización GitHub `PropSail`:

- Repos recomendados:
  - `propsail-backend-api` → NestJS + Prisma.
  - `propsail-web-frontend` → Web.
  - `propsail-mobile-app` → React Native.
  - `propsail-sdk-ts` → **este SDK**.

### 2.2. Nombre del paquete npm

Paquete privado (o público si algún día se abre la API):

```text
@propsail/sdk
```

### 2.3. Versionamiento

- Se versiona con **SemVer**, alineado al documento de versionamiento semántico:
  - `MAJOR.MINOR.PATCH` → `1.3.2`, `2.0.0`, etc.
- Relación sugerida con cambios de API:
  - `PATCH`: cambios internos del SDK sin cambios de contrato público (ej. bug de tipado).
  - `MINOR`: nuevas rutas o campos **compatibles hacia atrás**.
  - `MAJOR`: cambios que rompen contrato (rename de campos, rutas, etc.).

---

## 3. Alcance del SDK

### 3.1. Qué sí hace

- Expone **clientes por dominio**:
  - `auth`: login, logout, refresh token, 2FA, recuperación de contraseña.
  - `users`: CRUD de usuarios, roles.
  - `properties`: propiedades, fotos, filtros de búsqueda.
  - `bookings` u otros dominios que agreguemos en el backend.
- Gestiona:
  - `baseURL` configurable (dev, QA, prod).
  - Cabeceras comunes (`Authorization`, `Content-Type`, etc.).
  - Errores HTTP comunes (401, 403, 404, 500).
- Provee **tipos TypeScript** generados desde OpenAPI:
  - `User`, `Property`, `Address`, `LoginRequest`, `LoginResponse`, etc.
- Pensado para usarse en:
  - Apps React (web).
  - React Native (mobile).
  - Node.js (scripts / integraciones).

### 3.2. Qué NO hace

- No almacena credenciales sensibles (usuario/contraseña) de forma persistente.
- No implementa UI (formularios, pantallas, modales).
- No decide dónde se guarda el token:
  - Web → cookie httpOnly / memory / localStorage (según decisión de seguridad del proyecto).
  - Mobile → MMKV (como se definió en el stack).
- No define lógica de negocio de alto nivel (por ejemplo reglas complejas de pricing):
  - Eso pertenece al backend.

---

## 4. Arquitectura interna del SDK

### 4.1. Capas principales

1. **Cliente HTTP base**
   - Implementado sobre `fetch` o `axios` (a acordar, propuesta inicial: `axios`).
   - Responsable de:
     - Enviar requests.
     - Añadir headers comunes.
     - Parsear respuestas JSON.
     - Unificar manejo de errores.

2. **Código generado desde OpenAPI**
   - Carpetas generadas automáticamente a partir del JSON de Swagger del backend NestJS.
   - No se edita a mano (solo se regenera).

3. **Capa de conveniencia escrita a mano**
   - Envuelve el código generado en una API más amigable:
     - Métodos más simples.
     - Abstracción de detalles de ruta, query params, etc.
   - Ejemplo:
     - Código generado: `AuthController_login`…
     - Capa manual: `sdk.auth.login({ email, password })`.

4. **Tipos compartidos**
   - Tipos e interfaces reexportados para ser reutilizados en frontend y mobile:
     - `User`, `Property`, `Paginated<T>`, etc.

### 4.2. Estructura de carpetas sugerida

```text
propsail-sdk-ts/
├── src/
│   ├── generated/            # código generado desde OpenAPI (NO tocar)
│   ├── http/
│   │   ├── httpClient.ts     # axios/fetch configurado
│   │   └── errors.ts         # tipos de errores comunes
│   ├── domains/
│   │   ├── auth.ts           # funciones de conveniencia para autenticación
│   │   ├── users.ts
│   │   ├── properties.ts
│   │   └── index.ts
│   ├── config.ts             # setBaseUrl, setAccessToken, etc.
│   ├── index.ts              # punto de entrada del SDK
│   └── types.ts              # tipos comunes reexportados
├── package.json
├── tsconfig.json
└── README.md
```

---

## 5. Generación desde OpenAPI / Swagger

### 5.1. Requisito del backend

El backend NestJS debe exponer el documento OpenAPI en un endpoint estable, por ejemplo:

```text
https://api.propsail.com/docs-json
# o
https://api.dorodrigo.website/api/docs-json
```

> El nombre final del endpoint se definirá en el proyecto backend.  
> **Importante:** debe ser accesible para el pipeline de CI/CD que genera el SDK.

### 5.2. Herramienta de generación sugerida

Se puede usar cualquier herramienta estándar. Ejemplos:

- `openapi-typescript-codegen`
- `openapi-generator-cli`
- `orval`

Ejemplo con `openapi-typescript-codegen` en `package.json` del SDK:

```jsonc
{
  "scripts": {
    "generate": "openapi --input https://api.propsail.com/docs-json --output src/generated --client axios",
    "build": "tsup src/index.ts --dts --format cjs,esm",
    "test": "vitest"
  },
  "devDependencies": {
    "openapi-typescript-codegen": "^0.29.0",
    "tsup": "^8.0.0",
    "typescript": "^5.0.0",
    "vitest": "^2.0.0",
    "axios": "^1.7.0"
  }
}
```

> Versión exacta de librerías se definirá cuando se cree el repo; esto es solo un ejemplo de referencia.

### 5.3. Flujo de trabajo para actualizar SDK

1. **Se modifica la API en el backend** (`propsail-backend-api`):
   - Se actualizan controladores, DTOs, etc.
   - Se actualiza el esquema Swagger si es necesario.

2. Se genera/actualiza el **OpenAPI JSON** del backend.

3. En el repo del SDK (`propsail-sdk-ts`):
   - Se ejecuta:
     ```bash
     npm run generate
     npm run test
     npm run build
     ```
   - Se revisan cambios generados (PR review).

4. Se actualiza versión en `package.json` según SemVer:
   - `feat` compatible → `MINOR`.
   - `fix` → `PATCH`.
   - breaking change → `MAJOR`.

5. Se publica el paquete:
   - A GitHub Packages o npm privado:
     ```bash
     npm publish --access public # o adecuado al registro
     ```

6. Frontend y mobile:
   - Actualizan versión del SDK:
     ```bash
     npm install @propsail/sdk@^1.3.0
     ```

---

## 6. Configuración del SDK en proyectos

### 6.1. Inicialización básica

```ts
// apiClient.ts
import { createClient } from '@propsail/sdk';

export const apiClient = createClient({
  baseUrl: process.env.EXPO_PUBLIC_API_URL ?? 'https://api.propsail.com',
  getAccessToken: async () => {
    // Ejemplo:
    // - Web: leer de cookie / localStorage
    // - Mobile: leer desde MMKV
    return tokenStore.getAccessToken();
  },
  onUnauthorized: () => {
    // acción centralizada:
    // - limpiar sesión
    // - redirigir a login
  },
});
```

El método `createClient` es la factoría principal del SDK y devuelve un objeto `client` con dominios:

```ts
const { auth, users, properties } = apiClient;
```

### 6.2. Uso típico en frontend web (React)

```ts
// hooks/useProperties.ts
import { useEffect, useState } from 'react';
import { apiClient } from '../apiClient';
import type { Property } from '@propsail/sdk';

export function useProperties() {
  const [data, setData] = useState<Property[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let mounted = true;

    apiClient.properties
      .list({ page: 1, pageSize: 20 })
      .then((res) => {
        if (mounted) setData(res.items);
      })
      .finally(() => {
        if (mounted) setLoading(false);
      });

    return () => {
      mounted = false;
    };
  }, []);

  return { data, loading };
}
```

### 6.3. Uso típico en React Native

```ts
// src/hooks/useLogin.ts
import { useState } from 'react';
import { apiClient } from '../apiClient';

export function useLogin() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function login(email: string, password: string) {
    setLoading(true);
    setError(null);
    try {
      const response = await apiClient.auth.login({ email, password });
      // Aquí puedes guardar tokens, manejar 2FA, etc.
      return response;
    } catch (err: any) {
      setError(apiClient.errors.toMessage(err)); // helper del SDK
      throw err;
    } finally {
      setLoading(false);
    }
  }

  return { login, loading, error };
}
```

---

## 7. Manejo de autenticación y tokens

### 7.1. Responsabilidades

- **Backend**:
  - Genera tokens de acceso y refresh tokens (JWT).
  - Define expiraciones y políticas de seguridad.

- **SDK**:
  - Añade el token de acceso en la cabecera `Authorization: Bearer <token>`.
  - Permite configurar `getAccessToken` y `onUnauthorized`.
  - Puede incluir helpers para refrescar tokens (si la API lo permite).

- **Aplicaciones (web / mobile)**:
  - Deciden dónde y cómo persistir los tokens:
    - Web: cookie httpOnly, localStorage (con cuidado), memory, etc.
    - Mobile: MMKV (como se definió en el stack de PropSail).

### 7.2. Ejemplo de reintento de refresh (opcional)

Si el proyecto define endpoint de refresh:

- Backend: `POST /auth/refresh`
- SDK: método `auth.refreshToken()`

Se puede configurar:

```ts
export const apiClient = createClient({
  baseUrl: process.env.EXPO_PUBLIC_API_URL!,
  getAccessToken: tokenStore.getAccessToken,
  onUnauthorized: async (originalRequest) => {
    const refreshToken = tokenStore.getRefreshToken();
    if (!refreshToken) {
      tokenStore.clear();
      return null;
    }

    const newTokens = await apiClient.auth.refreshToken({ refreshToken });
    tokenStore.save(newTokens);
    return newTokens.accessToken;
  },
});
```

> Detalles exactos dependerán del contrato que exponga el backend.

---

## 8. Manejo de errores

El SDK debe exponer una forma uniforme de manejar errores:

### 8.1. Tipos de errores

- `ApiError`:
  - Tiene `statusCode`, `message`, `details`.
- `NetworkError`:
  - Problemas de conexión, timeouts.
- `UnexpectedError`:
  - Cualquier otra cosa no controlada.

### 8.2. Helper de mensaje amigable

```ts
try {
  await apiClient.properties.list({ page: 1 });
} catch (err) {
  const msg = apiClient.errors.toMessage(err);
  showToast(msg);
}
```

`toMessage` podría mapear:
- 400 → mensaje de validación.
- 401 → “Tu sesión expiró, por favor vuelve a iniciar sesión”.
- 500 → “Tuvimos un problema, inténtalo más tarde”.

---

## 9. Buenas prácticas de uso en el equipo PropSail

### 9.1. No duplicar clientes HTTP

- En frontend y mobile:
  - **Siempre** usar `@propsail/sdk`.
  - No crear nuevos `fetch`/`axios` directos a la API, salvo casos MUY justificados.

### 9.2. Tipos compartidos

- Siempre que se pueda:
  - Importar tipos del SDK:
    ```ts
    import type { Property, User } from '@propsail/sdk';
    ```
  - No recrear interfaces duplicadas en cada proyecto.

### 9.3. Convención de nombres

- Métodos del SDK:
  - Verbos claros y consistentes:
    - `list`, `get`, `create`, `update`, `delete`, `search`.
- Ejemplos:
  - `properties.list(params)`
  - `properties.getById(id)`
  - `users.updateProfile(payload)`

### 9.4. Relación con DevSecOps

El SDK ayuda a DevSecOps de PropSail porque:

- Centraliza:
  - Manejo de endpoints.
  - Cabeceras y timeouts.
  - Lógica de autenticación.
- Facilita:
  - Log de requests/responses en entornos de QA.
  - Aplicar políticas de seguridad consistentes.

Se integra de forma natural con:

- **Conventional Commits** (para versionado automático).
- **SemVer** (para releases claros).
- Pipelines de CI/CD que:
  - corren tests del SDK,
  - generan changelog,
  - publican nueva versión.

---

## 10. Flujo recomendado para el equipo cuando cambia la API

1. **Cambios en backend (`propsail-backend-api`)**
   - Se ajusta NestJS + Prisma.
   - Se actualizan DTOs, validators, etc.
   - Swagger refleja el nuevo contrato.

2. **Pull Request en backend**
   - Revisión de código.
   - Tests verdes.

3. **Actualizar SDK (`propsail-sdk-ts`)**
   - `npm run generate`
   - Revisar cambios generados (no tocar a mano).
   - Ajustar capa manual (`src/domains/*`), si es necesario.
   - Actualizar versión en `package.json`.
   - `npm run test && npm run build`.

4. **Publicar SDK**
   - `npm publish` (o pipeline automático).

5. **Actualizar consumidores**
   - Frontend web:
     ```bash
     npm install @propsail/sdk@latest
     ```
   - Mobile:
     ```bash
     npm install @propsail/sdk@latest
     ```
   - Ejecutar tests y validar que todo funciona.

---

## 11. Roadmap inicial del SDK PropSail

1. Crear repositorio `propsail-sdk-ts` en la organización PropSail.
2. Definir exactamente:
   - Endpoint de Swagger del backend.
   - Herramienta de generación OpenAPI.
   - Cliente HTTP base (axios o fetch).
3. Implementar:
   - `createClient` básico.
   - Soporte para:
     - `auth.login`
     - `auth.logout`
     - `auth.refreshToken` (si aplica).
     - `properties.list`
4. Integrar el SDK en:
   - `propsail-web-frontend`.
   - `propsail-mobile-app`.
5. Alinear pipeline de CI/CD:
   - Tests.
   - Lint.
   - Publicación en registro de paquetes.

---

## 12. Resumen para el equipo PropSail

- El **SDK PropSail** es nuestro **cliente común** para hablar con la API desde web, mobile y scripts.
- Se basa en:
  - **OpenAPI/Swagger** del backend.
  - **TypeScript** + **axios/fetch**.
  - Buenas prácticas de **SemVer** + **Conventional Commits** + **Trunk-Based Development**.
- Objetivo:
  - Menos bugs.
  - Más velocidad.
  - Un contrato claro y tipado entre backend y frontend/mobile.

> A partir de ahora, cualquier nuevo consumo de API en frontend o mobile **debe pasar por `@propsail/sdk`**, salvo acuerdo explícito en contrario documentado en la arquitectura del proyecto.
