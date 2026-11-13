# Índice de Hallazgos Organizados por Categoría

## 📌 Navegación Rápida

**Por Propósito:**
- [Antes de PR](#antes-de-pr-checklist-por-categoría)
- [Code Review](#guía-de-code-review-por-categoría)
- [Onboarding](#guía-de-onboarding-por-categoría)
- [Escalabilidad](#plan-de-escalabilidad)

**Por Severidad:**
- [Críticos](#-críticos-seguridad) 🔴
- [Altos](#-altos-arquitectura--testing) 🟡
- [Medios](#-medios-dx--testing) 🟠
- [Bajos](#-bajos-ux--logging) ⚪

**Por Archivo:**
- [Backend](#backend-files)
- [Frontend](#frontend-files)
- [Config](#config-files)

---

## 🔴 CRÍTICOS (Seguridad)

### Debes ARREGLAR ANTES de producción

**1. CORS Permite Todos los Orígenes**
- 📄 Archivo: `backend/app/main.py:8-12`
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 4. SEGURIDAD](#4-seguridad)
- ⚡ Acción inmediata: Whitelist dominios permitidos
- 🔗 Referencia: RISKS.md § 1. CORS Permite Todos los Orígenes

**2. Debugpy Expuesto en Dockerfile**
- 📄 Archivo: `backend/Dockerfile:8`
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 4. SEGURIDAD](#4-seguridad)
- ⚡ Acción inmediata: Crear Dockerfile.dev separado
- 🔗 Referencia: RISKS.md § 2. Debugpy Expuesto en Dockerfile

---

## 🟡 ALTOS (Arquitectura + Testing)

### Debes ARREGLAR PRONTO (antes de merge a main)

**3. Tipos Duplicados Backend ↔ Frontend**
- 📄 Archivos: `backend/app/routes.py:1-9` ↔ `frontend/src/lib/financial-types.ts:1-3`
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 1. ARQUITECTURA](#1-arquitectura-y-separación-de-responsabilidades)
- ⚡ Acción: Tests de integración que validen formato
- 🔗 Referencia: RISKS.md § 3. Enumeraciones Duplicadas sin Sincronización

**4. Datos Mock Idénticos Siempre (seed=42)**
- 📄 Archivo: `backend/app/routes.py:286, 310, 332, 358, 380, 399, 416`
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 3. TESTING](#3-testing-y-reproducibilidad)
- ⚡ Acción: Parametrizar tests con múltiples seeds
- 🔗 Referencia: RISKS.md § 4. Datos Mock Idénticos Siempre

**5. Sin Validación Input en Endpoints**
- 📄 Archivo: `backend/app/routes.py:375-397`
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 2. TYPE SAFETY](#2-type-safety-y-validación)
- ⚡ Acción: Agregar `if start_date > end_date: raise HTTPException(400)`
- 🔗 Referencia: RISKS.md § 5. Sin Validación de Edge Cases

**6. Alertas Demasiado Simplistas**
- 📄 Archivo: `backend/app/routes.py:199-213`
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 7. LÓGICA DE NEGOCIOS](#7-lógica-de-negocios)
- ⚡ Acción: Implementar media móvil, desviación estándar, seasonalidad
- 🔗 Referencia: RISKS.md § 6. Lógica de Alertas Demasiado Simplista

**7. Sin Tests de Integración Backend-Frontend**
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 3. TESTING](#3-testing-y-reproducibilidad)
- ⚡ Acción: Crear test que valide formato de respuesta real

---

## 🟠 MEDIOS (DX + Testing)

### Debes ARREGLAR DESPUÉS (próxima iteración)

**8. Fetch sin Retry Logic**
- 📄 Archivo: `frontend/src/App.tsx:16-21`
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 5. DESARROLLO Y EXPERIENCIA](#5-desarrollo-y-experiencia-del-desarrollador-dx)
- ⚡ Acción: Agregar exponential backoff retry
- 🔗 Referencia: RISKS.md § 7. Sin Retry Logic en Fetch

**9. API_BASE_URL sin Validación**
- 📄 Archivo: `frontend/src/App.tsx:13`
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 5. DESARROLLO Y EXPERIENCIA](#5-desarrollo-y-experiencia-del-desarrollador-dx)
- ⚡ Acción: Validar al startup, lanzar error si vacío
- 🔗 Referencia: RISKS.md § 8. API_BASE_URL sin Validación

**10. Estado Global sin Context API**
- 📄 Archivo: `frontend/src/App.tsx:26-29`
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 5. DESARROLLO Y EXPERIENCIA](#5-desarrollo-y-experiencia-del-desarrollador-dx)
- ⚡ Acción: Refactor con Context API o Zustand (preventivo)
- 🔗 Referencia: RISKS.md § 9. Estado Global sin Context API

**11. .env.example Incompleto**
- 📄 Archivo: `frontend/.env.example:1`
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 5. DESARROLLO Y EXPERIENCIA](#5-desarrollo-y-experiencia-del-desarrollador-dx)
- ⚡ Acción: Documentar todas las vars y sus formatos
- 🔗 Referencia: RISKS.md § 13. .env.example Incompleto

**12. Sin Parametrización de Tests Backend**
- 📄 Archivo: `backend/tests/test_routes.py`
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 3. TESTING](#3-testing-y-reproducibilidad)
- ⚡ Acción: Usar `@pytest.mark.parametrize` en tests

---

## ⚪ BAJOS (UX + Logging)

### Opcional, Mejora DX pero No Crítico

**13. Period Label Hardcoded**
- 📄 Archivo: `frontend/src/App.tsx:48`
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 5. DESARROLLO Y EXPERIENCIA](#5-desarrollo-y-experiencia-del-desarrollador-dx)
- ⚡ Acción: Usar año actual dinámico
- 🔗 Referencia: RISKS.md § 10. Period Label Hardcoded

**14. Sin Logging Estructurado**
- 📖 Ver detalles: [FINDINGS-BY-CATEGORY.md § 5. DESARROLLO Y EXPERIENCIA](#5-desarrollo-y-experiencia-del-desarrollador-dx)
- ⚡ Acción: Agregar logging.info() en backend, pino/winston en frontend
- 🔗 Referencia: RISKS.md § 11. Sin Logging Estructurado

---

## ✅ BUENO (Mantener)

### Patrones Correctos - Nunca Cambiar

| Categoría | Hallazgo | Archivo | Línea |
|-----------|----------|---------|-------|
| **Type Safety** | Type hints con `\|` | `routes.py` | 60-70 |
| **Type Safety** | `Literal[]` para enums | `routes.py` | 1-9 |
| **Type Safety** | Pydantic `BaseModel` | `routes.py` | 15-50 |
| **Type Safety** | `response_model=` en endpoints | `routes.py` | 286+ |
| **Type Safety** | Props interface explícita | `kpi-card.tsx` | 1-14 |
| **Testing** | Funciones puras sin mutación | `routes.py` | 92-180 |
| **Arquitectura** | Lógica en `lib/`, no en componentes | `financial-utils.ts` | - |
| **DX** | Importes con alias `@/` | `App.tsx` | 1-10 |
| **DX** | Nombres kebab-case | `dashboard/` | - |
| **DX** | CSS variables dinámicas | `kpi-card.tsx` | 16-24 |

📖 Ver detalles: [CONVENTIONS.md](./CONVENTIONS.md)

---

## 📋 Antes de PR: Checklist por Categoría

### Backend PR Checklist

**Type Safety:**
- [ ] Todos los parámetros tienen type hints
- [ ] Todos los retornos tienen type hints
- [ ] Usar `Literal[]` para tipos cerrados, no strings
- [ ] Usar `|` para unions, no `Optional[]`

**Validación:**
- [ ] Si fecha range: `if start_date > end_date: raise HTTPException(400)`
- [ ] Si Query parameter: usar `Query()` con `ge`, `le`, `enum` si aplica
- [ ] Probar casos edge: límites, valores vacíos, valores inválidos

**Endpoints:**
- [ ] TODOS tienen `response_model=`
- [ ] Swagger `/docs` muestra el endpoint correctamente
- [ ] Datos mock usan `generate_mock_movements(seed=...)` con seed VARIADO

**Testing:**
- [ ] Tests usan `@pytest.mark.parametrize` con múltiples valores
- [ ] Tests no hardcodean valores que vienen de seed fijo

**Seguridad:**
- [ ] NO hay `allow_origins=["*"]`
- [ ] NO hay debugpy exponiendo 0.0.0.0:5678

### Frontend PR Checklist

**Type Safety:**
- [ ] Componente nuevo tiene `interface ComponentProps`
- [ ] TODOS los props tienen tipos explícitos
- [ ] NO hay `any`

**Imports:**
- [ ] Usa `@/` alias, no `../../../`
- [ ] Imports de lógica vienen de `lib/`, no de componente sibling

**Componentes:**
- [ ] Lógica de cálculo está en `lib/`, no en JSX
- [ ] Componente solo recibe data y renderiza
- [ ] Si hay loading: usa `<Skeleton>` de `ui/skeleton.tsx`
- [ ] Si es card: usa `<Card>` de `ui/card.tsx`

**Styling:**
- [ ] Usa CSS variables `var(--nombre)`, no colores hardcoded
- [ ] Si variantes: usa `Record<Variant, string>` como en `kpi-card.tsx:16-24`

**Nombres:**
- [ ] Archivo usa kebab-case: `my-component.tsx`
- [ ] Componente exportado es PascalCase: `export function MyComponent`

**API:**
- [ ] Fetch usa `VITE_API_BASE_URL` validado
- [ ] Si retry necesario: implementar exponential backoff
- [ ] Error handling muestra mensaje útil

### Ambos PR Checklist

**Documentación:**
- [ ] `.env.example` actualizado si hay nuevas vars de env
- [ ] Comentarios solo en código confuso, no obvio

**Integration:**
- [ ] Si cambió tipos backend: verificar frontend recibe lo esperado
- [ ] Si cambió endpoint: verificar tests pasan

---

## 🔍 Guía de Code Review por Categoría

### Al revisar, busca:

**Seguridad (🔴 crítico):**
- [ ] CORS no es `["*"]`
- [ ] debugpy no está expuesto públicamente
- [ ] Validación de input existe

**Arquitectura (🟡 alto):**
- [ ] Tipos están sincronizados o tests de integración validan
- [ ] Lógica está en `lib/` o helper, no en componente
- [ ] Router/separación de responsabilidades respetada

**Type Safety (🟡 alto):**
- [ ] Todos los parameters tienen tipos
- [ ] Props son interfaces explícitas
- [ ] Pydantic `response_model=` en endpoints

**Testing (🟡 alto + 🟠 medio):**
- [ ] Tests usan múltiples valores (no solo caso feliz)
- [ ] seed de mock data varía
- [ ] Tests de integración validan formato end-to-end

**DX (🟠 medio):**
- [ ] Importes usan `@/`, no `../../../`
- [ ] Fetch tiene retry (si aplica)
- [ ] Variables de env validadas al startup
- [ ] .env.example documentado

**UX (⚪ bajo):**
- [ ] Mensajes de error son útiles (no genéricos)
- [ ] Loading states implementados
- [ ] Valores dinámicos no hardcoded

---

## 🎓 Guía de Onboarding por Categoría

### Qué debe saber ANTES de modificar código

1. **Arquitectura** (30 min)
   - Backend: router en `routes.py`, lógica separada
   - Frontend: `components/` renderiza, `lib/` calcula
   - Leer: [CONVENTIONS.md § Separación de Responsabilidades](./CONVENTIONS.md)

2. **Type Safety** (20 min)
   - Backend usa `Literal[]` no Enum, `|` no Optional
   - Frontend props son `interface`, no tipo anónimo
   - Leer: [CONVENTIONS.md § Type Hints](./CONVENTIONS.md)

3. **Seguridad** (15 min)
   - CORS está whitelistado (no `["*"]`)
   - debugpy está en dev solo
   - Leer: [FINDINGS-BY-CATEGORY.md § 4. SEGURIDAD](#4-seguridad)

4. **Testing** (20 min)
   - Tests usan múltiples seeds, no `seed=42` siempre
   - Lógica pura está sin side effects
   - Leer: [CONVENTIONS.md § Testing](./CONVENTIONS.md)

5. **Patrones a Evitar** (15 min)
   - NO duplicar tipos sin sync
   - NO hardcodear valores
   - NO fetch sin retry
   - NO `any` en TypeScript
   - Leer: [FINDINGS-BY-CATEGORY.md § Resumen por Severidad](#resumen-por-severidad)

---

## 📈 Plan de Escalabilidad

### Si crece a 2-3 endpoints más:
- **Arquitectura**: Separar `routes.py` en `routes/movements.py`, `routes/metrics.py`
- **Estado**: Considerar Context API o Zustand (no es urgente aún)
- **Testing**: Agregar tests de integración backend-frontend

### Si crece a 5+ componentes en dashboard:
- **DX**: Refactor estado con Context API
- **Organización**: Carpetas por feature (`income/`, `outcome/`, `alerts/`)
- **Imports**: Índices centrales `index.ts` en cada carpeta

### Si hay múltiples usuarios/autenticación:
- **Seguridad**: Revisar CORS y agregar autenticación
- **Backend**: Agregar validación de usuario en endpoints
- **Frontend**: Agregar token refresh, manejo de sesión

### Si datos reales (no mock):
- **Validación**: Agregar pydantic validadores
- **Logging**: Implementar logging estructurado
- **Tests**: Agregar tests de integración con base de datos

---

## 📁 Backend Files

**Main Configuration:**
- `backend/app/main.py` (11 líneas)
  - CORS policy: línea 8-12 (⚠️ CRÍTICO)
  - Router include: línea 9 (✅ PATRÓN)
  - Referencia: [FINDINGS-BY-CATEGORY.md § 4. SEGURIDAD](#4-seguridad)

**API Endpoints & Types:**
- `backend/app/routes.py` (520 líneas)
  - Type definitions: línea 1-9 (✅ PATRÓN - duplicados con frontend ⚠️)
  - Modelos Pydantic: línea 15-50 (✅ PATRÓN)
  - Funciones puras: línea 92-180 (✅ PATRÓN)
  - Alertas lógica: línea 199-213 (⚠️ SIMPLISTA)
  - Endpoints: línea 283-420 (líneas 286, 310, 332, 358, 380, 399, 416 ⚠️ seed=42)
  - Comparación (validación): línea 375-397 (⚠️ SIN VALIDACIÓN)
  - Referencia: [FINDINGS-BY-CATEGORY.md](#findings-by-category)

**Docker:**
- `backend/Dockerfile` (12 líneas)
  - CMD con debugpy: línea 8 (🔴 CRÍTICO)
  - Referencia: [FINDINGS-BY-CATEGORY.md § 4. SEGURIDAD](#4-seguridad)

**Testing:**
- `backend/tests/test_routes.py`
  - ⚠️ Sin parametrización
  - Referencia: [FINDINGS-BY-CATEGORY.md § 3. TESTING](#3-testing-y-reproducibilidad)

---

## 📁 Frontend Files

**Root Component:**
- `frontend/src/App.tsx` (60 líneas)
  - API_BASE_URL: línea 13 (⚠️ SIN VALIDACIÓN)
  - Fetch sin retry: línea 16-21 (⚠️)
  - Estado global: línea 26-29 (⚠️)
  - useEffect fetch: línea 29-43 (✅ PATRÓN)
  - Period hardcoded: línea 48 (⚠️ BAJO)
  - Referencia: [FINDINGS-BY-CATEGORY.md § 5. DESARROLLO](#5-desarrollo-y-experiencia-del-desarrollador-dx)

**Types:**
- `frontend/src/lib/financial-types.ts` (20 líneas)
  - Tipos duplicados de backend (⚠️)
  - Referencia: [FINDINGS-BY-CATEGORY.md § 1. ARQUITECTURA](#1-arquitectura-y-separación-de-responsabilidades)

**Lógica de Cálculo:**
- `frontend/src/lib/financial-utils.ts` (60 líneas)
  - Funciones puras: `computeKPIs()`, `computeMonthlyData()` (✅ PATRÓN)
  - Referencia: [CONVENTIONS.md § Lógica de Cálculo en lib/](./CONVENTIONS.md)

**Componentes de Dominio:**
- `frontend/src/components/dashboard/kpi-card.tsx` (50 líneas)
  - Props interface: línea 1-14 (✅ PATRÓN)
  - CSS variables: línea 16-24 (✅ PATRÓN)
  - Referencia: [CONVENTIONS.md](./CONVENTIONS.md)
  
- Otros dashboard components: `dashboard-header.tsx`, `kpi-row.tsx`, `income-outcome-chart.tsx`, `profit-percent-chart.tsx`
  - Todos siguen patrón props interface
  - Referencia: [CONVENTIONS.md](./CONVENTIONS.md)

**Componentes UI Base:**
- `frontend/src/components/ui/card.tsx`
- `frontend/src/components/ui/skeleton.tsx`
- Referencia: [CONVENTIONS.md § Componentes UI Base](./CONVENTIONS.md)

**Testing:**
- `frontend/src/lib/financial-utils.test.ts`
  - Tests de funciones puras (✅)
  - ⚠️ Sin tests de componentes
  - Referencia: [FINDINGS-BY-CATEGORY.md § 3. TESTING](#3-testing-y-reproducibilidad)

---

## 📁 Config Files

**Frontend Config:**
- `frontend/tsconfig.app.json` (20 líneas)
  - Alias paths: línea 7-9 (✅ PATRÓN)
  - Referencia: [CONVENTIONS.md § Importes con Alias](./CONVENTIONS.md)

- `frontend/vite.config.ts` (30 líneas)
  - Alias @ configuration (✅)

- `frontend/.env.example`
  - ⚠️ Incompleto, solo 1 variable
  - Referencia: [FINDINGS-BY-CATEGORY.md § 6. DOCUMENTACIÓN](#6-documentación)

**Backend:**
- `backend/requirements.txt`
- `backend/.env.example` (si existe)

---

## 🔗 Referencias Cruzadas

**Documento -> Sección:**

- [CONVENTIONS.md](./CONVENTIONS.md): Patrones correctos con código real
- [RISKS.md](./RISKS.md): Patrones peligrosos con escenarios de fallo
- [QUICK-GUIDE.md](./QUICK-GUIDE.md): Checklist operacional
- [FINDINGS-BY-CATEGORY.md](./FINDINGS-BY-CATEGORY.md): Hallazgos agrupados por categoría (ERES AQUÍ)
- [INDEX-CATEGORIZED.md](./INDEX-CATEGORIZED.md): Índice de navegación (ERES AQUÍ)

---

## 📝 Última Actualización

- **Fecha**: 2026-11-13
- **Documentos**: 5 archivos markdown
- **Total hallazgos**: 14 (2 críticos, 5 altos, 4 medios, 3 bajos, 10 bueno)
- **Archivos escaneados**: 15 (backend + frontend + config)

