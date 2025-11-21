# Guía de Pruebas Unitarias en PropSail  
Pruebas unitarias para backend (NestJS), frontend web y mobile (React Native)
================================================================================

> Versión: 1.0.0  
> Público objetivo: equipo de desarrollo PropSail (trainee / junior)  
> Stack: TypeScript, NestJS, Prisma, JWT, Swagger/OpenAPI, React/React Native, SWR, MMKV, Tailwind, cPanel + MySQL/MariaDB

---

## 1. Contexto: ¿por qué esta guía?

En PropSail vamos a construir un producto que debe ser **estable**, **seguro** y **fácil de mantener**.  
Con un stack moderno (NestJS + React/React Native + Prisma + MySQL/MariaDB) esto **solo es sostenible** si tenemos **pruebas automatizadas** desde el día uno.

Las **pruebas unitarias** son la base de todo:

- Nos avisan rápido cuando rompemos algo.
- Sirven como **documentación viva** del comportamiento del código.
- Permiten refactorizar sin miedo.
- Son el primer filtro de calidad antes de integrar cambios en el trunk (`main`).

Esta guía explica **qué son las pruebas unitarias** y **cómo aplicarlas de forma práctica** en el día a día del equipo PropSail.

---

## 2. Qué son las pruebas unitarias (unit tests)

Una **unidad** es la pieza más pequeña de código que tiene sentido probar de forma aislada:

- Una **función** (ej: `calculateCommission`).
- Un **método** de una clase (ej: `AuthService.validateUser`).
- Un **hook** o **composable** (ej: `useFetchProperties`).
- Un **componente pequeño** (ej: un botón, un input, un card simple).

Una **prueba unitaria**:

- Es un pequeño trozo de código que:
  1. **Prepara** los datos y el contexto (Arrange).
  2. **Ejecuta** la unidad a probar (Act).
  3. **Comprueba** el resultado con aserciones (Assert).

- Responde a preguntas del tipo:
  - “¿Qué pasa si llamo esta función con estos parámetros?”
  - “¿Qué debería devolver este servicio si el usuario no existe?”
  - “Qué debería renderizar este componente si no hay datos?”

---

## 3. Relación con otros tipos de pruebas

En PropSail seguiremos la idea de la **pirámide de testing**:

- **Base** → Pruebas **unitarias** (muchas, rápidas, baratas).
- **Capa intermedia** → Pruebas de **integración** (módulos colaborando).
- **Capa superior** → Pruebas de **extremo a extremo (E2E)** y funcionales (flujos completos).

Resumen:

- **Unitarias** → prueban **una pieza pequeña**, aislada (lo que cubre esta guía).
- **Integración** → prueban que módulos se hablan bien entre sí (servicio + repositorio + capa HTTP falsa, etc.).
- **E2E / funcionales** → prueban flujos completos como lo haría un usuario real.

---

## 4. Principios clave de las pruebas unitarias

### 4.1. Aislamiento de componentes

La unidad bajo prueba debe estar lo más **aislada posible**:

- No debería:
  - Conectarse a base de datos real.
  - Hacer llamadas HTTP reales.
  - Acceder a disco o sistemas externos.
- En su lugar usamos:
  - **Mocks** (imitar comportamiento de dependencias).
  - **Stubs** (devuelven datos prefijados).
  - **Fakes** (implementaciones simples de algo más complejo).

Ejemplo para backend:

- `UserService` depende de `UserRepository`.
- En tests unitarios **no** usamos Prisma real.
- Usamos un `UserRepository` falso en memoria o un mock de Jest.

---

### 4.2. Resultados repetibles y consistentes

Una prueba unitaria:

- Debe **dar siempre el mismo resultado** si el código no ha cambiado.
- No debe depender de:
  - Hora del sistema.
  - Estado de un servidor externo.
  - Orden de ejecución de otras pruebas.

Si una prueba “a veces pasa y a veces falla” sin cambiar código → es una mala prueba (flaky test).

---

### 4.3. Probar unidades pequeñas

Mejor muchas pruebas pequeñas que pocas pruebas enormes.

- Una prueba debería cubrir **una condición concreta**.
- Si necesitas comentar demasiado qué hace, probablemente está probando demasiado.

Ejemplo:

- ✅ Bien: `should_add_two_positive_numbers`
- ✅ Bien: `should_throw_when_email_is_invalid`
- ❌ Mal: `should_do_auth_and_save_user_and_send_email_and_log_everything`

---

### 4.4. Automatización y velocidad

- Las pruebas unitarias deben ser **rápidas** (milisegundos).
- Deben ejecutarse:
  - Localmente antes de hacer commit.
  - En la pipeline de CI en cada `push` o `pull request` al trunk.

Si los tests son lentos, el equipo **dejará de ejecutarlos**.  
Si son rápidos, se convierten en un hábito natural.

---

## 5. Unit testing en el stack PropSail

### 5.1. Backend (NestJS + Prisma + JWT)

#### 5.1.1. Qué vamos a probar

En el backend probaremos principalmente:

- **Funciones puras** (helpers, utils).
- **Servicios de dominio** (`AuthService`, `PropertyService`, etc.).
- **Pipes**, **guards** y **interceptors** (validación, auth, logging).
- **Adapters** que transforman datos entre capas.

#### 5.1.2. Qué NO se considera prueba unitaria

- Conectar a MySQL real → eso es integración.
- Levantar el servidor HTTP y hacer peticiones reales → eso es E2E.
- Probar Prisma con la base de datos → eso es integración.

#### 5.1.3. Herramientas recomendadas

Para backend NestJS usaremos:

- **Jest** como framework de testing.
- `@nestjs/testing` para crear módulos de prueba.
- `ts-jest` o configuración de Jest para TypeScript.
- `jest-mock` para crear mocks y espías.

---

#### 5.1.4. Ejemplo simple (función pura)

`src/shared/utils/currency.ts`:

```ts
export function formatCLP(amount: number): string {
  if (!Number.isFinite(amount)) {
    throw new Error('Invalid amount');
  }

  return new Intl.NumberFormat('es-CL', {
    style: 'currency',
    currency: 'CLP',
    maximumFractionDigits: 0,
  }).format(amount);
}
```

Test unitario `src/shared/utils/__tests__/currency.spec.ts`:

```ts
import { formatCLP } from '../currency';

describe('formatCLP', () => {
  it('formatea montos positivos en CLP', () => {
    const result = formatCLP(10000);
    expect(result).toBe('$10.000');
  });

  it('arroja error con montos no numéricos', () => {
    // @ts-expect-error probando un caso incorrecto
    expect(() => formatCLP('no-numero')).toThrow('Invalid amount');
  });
});
```

---

#### 5.1.5. Ejemplo de servicio NestJS con mocks

Servicio:

```ts
// src/modules/auth/auth.service.ts
import { Injectable } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersRepository } from '../users/users.repository';
import * as bcrypt from 'bcryptjs';

@Injectable()
export class AuthService {
  constructor(
    private readonly usersRepo: UsersRepository,
    private readonly jwtService: JwtService,
  ) {}

  async validateUser(email: string, password: string) {
    const user = await this.usersRepo.findByEmail(email);
    if (!user) return null;

    const isValid = await bcrypt.compare(password, user.passwordHash);
    if (!isValid) return null;

    return { id: user.id, email: user.email };
  }

  async login(email: string, password: string) {
    const user = await this.validateUser(email, password);
    if (!user) {
      throw new Error('Invalid credentials');
    }

    const payload = { sub: user.id, email: user.email };
    const accessToken = await this.jwtService.signAsync(payload);

    return { accessToken, user };
  }
}
```

Test unitario:

```ts
// src/modules/auth/__tests__/auth.service.spec.ts
import { AuthService } from '../auth.service';
import { UsersRepository } from '../../users/users.repository';
import { JwtService } from '@nestjs/jwt';

describe('AuthService', () => {
  let authService: AuthService;
  let usersRepo: jest.Mocked<UsersRepository>;
  let jwtService: jest.Mocked<JwtService>;

  beforeEach(() => {
    usersRepo = {
      findByEmail: jest.fn(),
    } as any;

    jwtService = {
      signAsync: jest.fn(),
    } as any;

    authService = new AuthService(usersRepo, jwtService);
  });

  it('devuelve null si el usuario no existe', async () => {
    usersRepo.findByEmail.mockResolvedValue(null);

    const result = await authService.validateUser('test@propsail.com', 'pass');
    expect(result).toBeNull();
  });

  it('lanza error si las credenciales son inválidas al hacer login', async () => {
    usersRepo.findByEmail.mockResolvedValue(null);

    await expect(
      authService.login('test@propsail.com', 'pass'),
    ).rejects.toThrow('Invalid credentials');
  });

  it('devuelve un token si las credenciales son válidas', async () => {
    usersRepo.findByEmail.mockResolvedValue({
      id: 'user-1',
      email: 'test@propsail.com',
      passwordHash: '$2a$10$hash-falso',
    });

    // mock de bcrypt.manual (en la vida real se hace con jest.mock('bcryptjs'))
    const bcrypt = require('bcryptjs');
    jest.spyOn(bcrypt, 'compare').mockResolvedValue(true);

    jwtService.signAsync.mockResolvedValue('token-falso');

    const result = await authService.login('test@propsail.com', 'pass');

    expect(jwtService.signAsync).toHaveBeenCalledWith({
      sub: 'user-1',
      email: 'test@propsail.com',
    });
    expect(result.accessToken).toBe('token-falso');
  });
});
```

Puntos a notar:

- No se usa DB real.
- `UsersRepository` y `JwtService` están mockeados.
- El test se enfoca en la **lógica de autenticación**, no en detalles de infraestructura.

---

### 5.2. Frontend web (React + SWR + Tailwind)

*(Si tu frontend web está en otra tech, la idea es la misma; aquí asumimos React.)*

#### 5.2.1. Qué probamos

- Componentes **pequeños**:
  - Presentacionales (cards, botones, inputs).
  - Layouts simples.
- Hooks:
  - Lógica de datos (`useProperties`, `useAuth`).
  - Lógica de UI (`useModal`, `usePagination`).
- Utilidades:
  - Formateo de fechas, montos, etc.

#### 5.2.2. Herramientas recomendadas

- **Jest** (mismo framework que backend).
- **React Testing Library**:
  - Foco en “cómo lo usa el usuario”, no en detalles internos.
- **MSW (Mock Service Worker)** (opcional, para tests de integración de datos).

Ejemplo de test básico de componente:

```tsx
// src/components/PropertyCard.tsx
import React from 'react';

type PropertyCardProps = {
  title: string;
  price: number;
};

export const PropertyCard: React.FC<PropertyCardProps> = ({ title, price }) => {
  return (
    <article>
      <h2>{title}</h2>
      <p>{price.toLocaleString('es-CL')}</p>
    </article>
  );
};
```

Test:

```tsx
// src/components/__tests__/PropertyCard.spec.tsx
import React from 'react';
import { render, screen } from '@testing-library/react';
import { PropertyCard } from '../PropertyCard';

describe('PropertyCard', () => {
  it('muestra título y precio formateado', () => {
    render(<PropertyCard title="Depto en Providencia" price={125000000} />);

    expect(
      screen.getByText('Depto en Providencia'),
    ).toBeInTheDocument();

    expect(screen.getByText('125.000.000')).toBeInTheDocument();
  });
});
```

---

### 5.3. Mobile (React Native + MMKV + SWR + Gluestack UI)

#### 5.3.1. Qué probamos

- **Hooks** de lógica de negocio (por ejemplo `useLogin`, `useUserSession`).
- **Funciones puras** (formatos, validaciones, parsers).
- Componentes que contengan poca lógica (por ejemplo, un formulario de login).

#### 5.3.2. Herramientas recomendadas

- **Jest** (otra vez, mismo framework).
- **React Native Testing Library**.
- Mocks para:
  - MMKV (storage).
  - Fetch / clientes HTTP.
  - Navegación (React Navigation).

Ejemplo simple de hook:

```ts
// src/hooks/usePasswordStrength.ts
export function getPasswordStrength(password: string): 'weak' | 'medium' | 'strong' {
  if (password.length < 6) return 'weak';
  if (password.length < 10) return 'medium';
  return 'strong';
}
```

Test:

```ts
// src/hooks/__tests__/usePasswordStrength.spec.ts
import { getPasswordStrength } from '../usePasswordStrength';

describe('getPasswordStrength', () => {
  it('devuelve weak si tiene menos de 6 caracteres', () => {
    expect(getPasswordStrength('12345')).toBe('weak');
  });

  it('devuelve medium si tiene entre 6 y 9 caracteres', () => {
    expect(getPasswordStrength('12345678')).toBe('medium');
  });

  it('devuelve strong si tiene 10 o más caracteres', () => {
    expect(getPasswordStrength('contraseñaSuperSegura')).toBe('strong');
  });
});
```

---

## 6. Buenas prácticas generales en PropSail

### 6.1. Patrón AAA (Arrange – Act – Assert)

Estructura sugerida de cada test:

```ts
it('haz_algo_cuando_pase_algo', () => {
  // Arrange (preparar contexto/datos)
  const a = 1;
  const b = 2;

  // Act (ejecutar lo que queremos probar)
  const result = sum(a, b);

  // Assert (comprobar resultado)
  expect(result).toBe(3);
});
```

---

### 6.2. Un concepto por test

- Idealmente **una aserción importante por test**.
- Pueden haber varias, pero todas deben referirse al mismo concepto.

Ejemplos de buenos nombres:

- `should_return_null_when_user_not_found`
- `should_throw_if_token_is_expired`
- `should_render_loading_state_while_fetching`

---

### 6.3. Convención de nombres y carpetas

Propuesta para todo el código de PropSail:

- Archivos de test:
  - `*.spec.ts` o `*.spec.tsx`.
- Ubicación:
  - Junto al archivo que prueban:  
    `src/modules/auth/auth.service.ts`  
    `src/modules/auth/auth.service.spec.ts`
- Nombres de describe:
  - `describe('AuthService', ...)`
  - `describe('PropertyCard', ...)`

---

### 6.4. Cobertura de código

Objetivos recomendados:

- **Backend**:
  - 80% cobertura global.
  - 90–100% en módulos críticos (auth, pagos, permisos).
- **Frontend / Mobile**:
  - 70–80% en lógica de negocio (hooks, stores).
  - Más relajado en componentes de puro diseño.

Importante: **No perseguir 100% solo por el número.**  
Lo importante es cubrir **rutas críticas y casos límite**.

---

### 6.5. Casos positivos y negativos

Siempre intenta cubrir:

- **Caso feliz** (“happy path”).
- **Errores/control de excepciones**.
- **Datos límite** (vacío, máximos, mínimos, nulos).

Ejemplo: función que valida RUT o correo:

- Happy path: formato correcto.
- Error: formato inválido.
- Límite: string vacío, `null`, `undefined`.

---

### 6.6. Pruebas y seguridad (DevSecOps)

Las pruebas unitarias ayudan a reforzar seguridad:

- Verificar que:
  - No se puede entrar con credenciales inválidas.
  - Tokens expirados o mal firmados son rechazados.
  - Los guard de NestJS bloquean acceso sin permisos.
  - Las funciones de validación sanitizan/validan inputs.

Buenas prácticas:

- Nunca incluir **secretos reales** (tokens, passwords) en tests.
- Usar valores falsos (`"TEST_SECRET"`, `"dummy-token"`).
- Probar caminos donde el sistema **rechaza** entradas peligrosas.

---

## 7. Integración con el flujo de trabajo PropSail

### 7.1. Antes de hacer commit

Regla para el equipo:

- Antes de hacer `git commit`:
  - Correr tests unitarios del módulo que tocaste.
  - Idealmente correr suite completa del proyecto.

Ejemplo de comandos típicos:

```bash
# backend
npm run test
npm run test:watch
npm run test:cov

# frontend
npm run test

# mobile
npm test
```

*(Los nombres exactos de los scripts se definirán en cada `package.json`, pero la idea es esta.)*

---

### 7.2. En la pipeline de CI/CD (GitHub Actions)

- En cada `push` o `pull request` a `main`:
  - Instalar dependencias.
  - Ejecutar `npm run lint`.
  - Ejecutar `npm test`.
- No se puede hacer merge a `main` si los tests unitarios fallan.

Esto se alinea con:

- **Trunk-Based Development**.
- **DevSecOps**.
- Entregas frecuentes y confiables.

---

## 8. ¿Cuándo no tiene sentido hacer pruebas unitarias?

Hay casos donde podemos relajar la exigencia:

- Prototipos rápidos que sabemos que se van a descartar.
- Componentes de UI extremadamente simples (puro markup).
- Código que va a ser reemplazado en muy corto plazo.

Incluso en esos casos, para **módulos de negocio y seguridad** (auth, pagos, permisos, cálculos financieros) → **siempre** deberíamos tener unit tests.

---

## 9. Checklist rápido para el equipo PropSail

### Backend (NestJS + Prisma)

- [ ] Cada servicio nuevo tiene al menos una prueba unitaria.
- [ ] Auth y permisos tienen casos de éxito y error.
- [ ] No se usan DB reales en unit tests.
- [ ] Se usan mocks para repositorios, JWT y clientes externos.

### Frontend web

- [ ] Componentes con lógica (formularios, flujos) tienen tests.
- [ ] Hooks con lógica de negocio se prueban aislados.
- [ ] No se mockea el framework innecesariamente (React, router).

### Mobile (React Native)

- [ ] Lógica de sesión y auth tiene unit tests.
- [ ] Mocks para MMKV y servicios HTTP.
- [ ] Se prueban funciones de validación y transformación de datos.

---

## 10. Resumen

- Las pruebas unitarias son la **primera línea de defensa** de calidad en PropSail.
- Nos ayudan a:
  - Detectar errores temprano.
  - Refactorizar sin miedo.
  - Documentar comportamiento.
  - Integrar cambios rápido en el trunk.
- Deben ser:
  - Rápidas.
  - Aisladas.
  - Automatizadas.
  - Claras y fáciles de entender.

> Si estás desarrollando una nueva funcionalidad y no sabes qué probar, pregúntate:  
> **“¿Qué es lo peor que podría romper esto si falla?”**  
> Eso es exactamente lo que merece una buena prueba unitaria.
