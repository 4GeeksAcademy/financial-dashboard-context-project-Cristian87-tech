# Hallazgos Concretos - Referencia Rápida

## Archivos Clave por Dominio

### Backend
| Archivo | Responsabilidad | Hallazgos |
|---------|---|---|
| `backend/app/main.py` | CORS, middleware, app init | CORS `["*"]` línea 8-12 |
| `backend/app/routes.py` | Todos los endpoints y modelos | seed=42 repetido 7 veces, tipos literales |
| `backend/tests/test_routes.py` | Tests unitarios | No hay parametrización |

### Frontend
| Archivo | Responsabilidad | Hallazgos |
|---------|---|---|
| `frontend/src/App.tsx` | Root, fetch, estado global | Fetch sin retry línea 16-21, API_BASE_URL sin validación línea 13 |
| `frontend/src/lib/financial-types.ts` | Tipos TS | Tipos DUPLICADOS de backend, sin sincronización |
| `frontend/src/lib/financial-utils.ts` | Lógica de cálculo | Ubicación correcta, lógica pura |
| `frontend/src/components/ui/` | Primitivas reutilizables | Card, Skeleton patrones correctos |
| `frontend/src/components/dashboard/` | Componentes de dominio | Props tipados correctamente |

### Config
| Archivo | Responsabilidad | Hallazgos |
|---------|---|---|
| `frontend/tsconfig.app.json` | Alias paths | `@/*` configurado correctamente línea 7-9 |
| `backend/Dockerfile` | Build image | debugpy expuesto línea 8 |
| `frontend/.env.example` | Documentación de vars | Incompleto, solo 1 var |

---

## Patrones: Qué Está Bien

### ✅ Type Hints con Literal
`backend/app/routes.py:1-9` usa `Literal[]` para enums
- Validación automática en Swagger
- TypeScript compatible

### ✅ Pydantic BaseModel Obligatorio
TODOS los esquemas (línea 15-50) heredan de `BaseModel`
- Respuesta: `response_model=list[FinancialMovement]` (línea 286)
- Genera Swagger automáticamente

### ✅ Funciones Puras sin Mutaciones
`filter_movements_by_date()`, `filter_movements()`, `ensure_chronological_order()` 
- Usan list comprehensions, no `.pop()` o `.remove()`
- Testables sin side effects

### ✅ Lógica en lib/ (Frontend)
`financial-utils.ts` contiene `computeKPIs()`, `computeMonthlyData()`, `formatCurrency()`
- Componentes solo reciben data y renderizan
- Testeable sin renderizar

### ✅ Props Tipadas con Interface
TODOS los componentes tienen `interface ComponentProps`
- `kpi-card.tsx:1-14`, `kpi-row.tsx:4`, `dashboard-header.tsx:4`
- IDE autocomplete funciona

### ✅ Importes con @/
`vite.config.ts` line 10 configura alias
- `App.tsx:1-10` usa `@/components`, `@/lib`
- No hay `../../../` imports

### ✅ Variantes con CSS Variables
`kpi-card.tsx:16-24` usa `var(--income-badge)`
- Dinámico, no hardcoded
- Fácil cambiar tema

---

## Patrones: Qué Está Mal

### ❌ CORS ["*"]
`backend/app/main.py:8` allow_origins
- Solución: `allow_origins=["http://localhost:5173", "https://yourdomain.com"]`

### ❌ Debugpy en Dockerfile
`backend/Dockerfile:8` CMD con debugpy --listen 0.0.0.0:5678
- Solución: Crear `Dockerfile.dev` con debugpy, usar `Dockerfile` sin debugpy

### ❌ Tipos Duplicados
`routes.py:1-9` vs `financial-types.ts:1-3` sin sync
- Solución: Generar tipos desde OpenAPI schema O tests de integración

### ❌ seed=42 Siempre
`routes.py:286, 310, 332, 358, 380, 399, 416`
- Solución: Parametrizar tests, variar seed en tests

### ❌ Sin Validación Input
`routes.py:375-397` permite `start_date > end_date`
- Solución: `if start_date > end_date: raise HTTPException(400)`

### ❌ Fetch sin Retry
`App.tsx:16-21` no reintentos
- Solución: Loop con exponential backoff

### ❌ API_BASE_URL sin Validación
`App.tsx:13` `?? ""`
- Solución: `if (!API_BASE_URL) throw new Error("..."`

### ❌ Period Hardcoded
`App.tsx:48` "2024 - Full Year"
- Solución: `const year = new Date().getFullYear()`

---

## Comportamientos a Preservar

| Comportamiento | Dónde | Por Qué |
|---|---|---|
| Type hints con `\|` | `routes.py:60-70` | Union types modernos |
| `response_model=` | Todos endpoints | Swagger automático |
| Funciones puras | `routes.py:92-180` | Testeable, composable |
| Props interface | Todos componentes | Type safety |
| Lógica en lib/ | `financial-utils.ts` | No en componentes |
| CSS variables | `kpi-card.tsx:16-24` | Dinámico |
| kebab-case archivos | `dashboard/` | Consistencia |
| Alias @/ | `App.tsx:1-10` | Importes limpios |

---

## Comportamientos a Evitar

| Antipatrón | Dónde NO hacerlo | Por Qué |
|---|---|---|
| CORS `["*"]` | `main.py:8` | Security risk |
| Debugpy público | `Dockerfile` | Remote access |
| Duplicar tipos | `routes.py` vs `types.ts` | Desincronización |
| seed fijo | Tests y endpoints | No valida variabilidad |
| Sin validación | Endpoints | Errores 500 vs 400 |
| Fetch sin retry | `App.tsx:16` | UX pobre |
| Estado global | `App.tsx:26-29` | Prop drilling |
| Hardcoded valores | Componentes | Difícil mantener |
| Lógica en componentes | No en `lib/` | No testeable |

---

## Checklist Operacional

### Antes de PR Backend
- [ ] Type hints en TODOS los argumentos y retornos
- [ ] `response_model=` en endpoint
- [ ] Función helper testeable (sin fetch, sin DB)
- [ ] Si Query parameter: usar `Query()` con `ge`, `le`, `enum` si aplica
- [ ] Si es date range: validar `start_date <= end_date`

### Antes de PR Frontend
- [ ] Interfaz explícita para props
- [ ] Imports con `@/`, no `../../../`
- [ ] Lógica de cálculo en `lib/`, no en JSX
- [ ] Componentes reutilizan Card, Skeleton de `ui/`
- [ ] Si endpoint, usar `VITE_API_BASE_URL`, no hardcoded

### Antes de Merge
- [ ] Tipos backend = tipos frontend (si cambió algo)
- [ ] Tests pasan (backend + frontend)
- [ ] Sin console.log, usar logging si aplica
- [ ] .env.example actualizado si hay nuevas vars

