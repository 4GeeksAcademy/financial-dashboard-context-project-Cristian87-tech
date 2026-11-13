# Convenciones del Proyecto - Hallazgos Concretos

## Backend (Python 3.13 + FastAPI)

### Type Hints Obligatorios con `|` para Union
**Archivo**: `backend/app/routes.py:1-9`
```python
OperationType = Literal["income", "outcome"]
Category = Literal["suppliers", "sales", "operational", "administrative", "others"]
BusinessType = Literal["B2B", "B2C"]
GroupBy = Literal["day", "week", "month"]
```

**Comportamiento observado**:
- Usa `Literal[]` en lugar de `Enum` classes
- Union types con `|` en lugar de `Union[]` (línea 60: `date | None`)
- No hay uso de `Optional[]`

**Aplica en**:
- `backend/app/routes.py:60-70` - todos los Query parameters usan `date | None`

---

### Pydantic BaseModel con response_model
**Archivo**: `backend/app/routes.py:15-50`

Modelos:
- `FinancialMovement` (línea 15)
- `MetricsFacets` (línea 22)
- `MetricsSummaryItem` (línea 30)
- `TopCategoryItem` (línea 37)
- `MetricsComparison` (línea 44)
- `MetricsAlert` (línea 50)

**Comportamiento concreto**:
- TODOS los schemas son Pydantic `BaseModel`
- TODOS los endpoints usan `response_model=` parámetro
- Ejemplo línea 286: `@router.get("/api/metrics", response_model=list[FinancialMovement])`

**Genera automáticamente**:
- Swagger docs en `http://localhost:8000/docs`
- JSON schema validation

---

### Funciones Puras sin Mutaciones
**Archivo**: `backend/app/routes.py:92-180`

Funciones que siguen el patrón:
- `filter_movements_by_date()` (línea 92) - recibe lista, retorna nueva lista
- `filter_movements()` (línea 108) - compone llamadas a otras funciones
- `ensure_chronological_order()` (línea 123) - usa `.sort()` no mutante
- `summarize_movements()` (línea 157)

**Comportamiento concreto**:
```python
# Línea 99: NO MUTATES lista original
filtered = [
    movement for movement in filtered if movement.create_date >= start_date]
```

No hay `.pop()`, `.remove()`, o asignación a índices.

---

### Seed Fijo seed=42 en Todos los Endpoints
**Archivo**: `backend/app/routes.py`

Ocurrencias exactas:
- Línea 286: `movements = generate_mock_movements(seed=42)`
- Línea 310: `movements = generate_mock_movements(seed=42)`
- Línea 332: `movements = generate_mock_movements(seed=42)`
- Línea 358: `movements = generate_mock_movements(seed=42)`
- Línea 380: `movements = generate_mock_movements(seed=42)`
- Línea 399: `movements = generate_mock_movements(seed=42)`
- Línea 416: `movements = generate_mock_movements(seed=42)`

**Comportamiento**: Datos SIEMPRE idénticos en cada llamada.

---

### Router Incluido en main.py
**Archivo**: `backend/app/main.py:9`
```python
app.include_router(router)
```

**Resultado**:
- Todos los endpoints en `routes.py` usan prefijo `/api/`
- Ejemplo: `@router.get("/api/metrics")` (línea 286)
- Health check sin prefijo: `@router.get("/health")` (línea 281)

---

## Frontend (React + TypeScript)

### Props con Interface Explícita
**Archivo**: `frontend/src/components/dashboard/kpi-card.tsx:1-14`

```typescript
interface KPICardProps {
  label: string
  value: string
  helperText: string
  icon: LucideIcon
  variant: 'income' | 'outcome' | 'profit' | 'profitPercent'
  loading?: boolean
}

export function KPICard({ label, value, helperText, icon: Icon, variant, loading }: KPICardProps) {
```

**Comportamiento concreto**:
- TODAS las props tienen tipo explícito
- Variantes son Literal types (línea 8)
- `loading` es opcional con `?`

**Aplica en**:
- `dashboard-header.tsx` (línea 4)
- `kpi-row.tsx` (línea 4)
- `income-outcome-chart.tsx` (línea 3)
- `profit-percent-chart.tsx` (línea 3)

---

### Importes con Alias @/
**Archivo**: `frontend/tsconfig.app.json:7-9`
```json
"paths": {
  "@/*": ["./src/*"]
}
```

**En código real** (`frontend/src/App.tsx:1-10`):
```typescript
import { DashboardHeader } from "@/components/dashboard/dashboard-header"
import { KPIRow } from "@/components/dashboard/kpi-row"
import { type FinancialMovement, type KPIMetrics } from "@/lib/financial-types"
import { computeKPIs, computeMonthlyData } from "@/lib/financial-utils"
```

No hay `../../` imports en ningún archivo.

---

### Lógica de Cálculo en lib/, No en Componentes
**Ubicación separada**:

Cálculos: `frontend/src/lib/financial-utils.ts:1-60`
```typescript
export function computeKPIs(movements: FinancialMovement[]): KPIMetrics
export function computeMonthlyData(movements: FinancialMovement[]): MonthlyDataPoint[]
export function formatCurrency(value: number): string
export function formatPercent(value: number): string
```

Consumo: `frontend/src/App.tsx:20-21`
```typescript
setMetrics(computeKPIs(movements))
setMonthlyData(computeMonthlyData(movements))
```

**Componentes** NO contienen cálculos:
- `KPICard` solo renderiza props (línea 45-50)
- `IncomeOutcomeChart` solo toma `data` prop
- `Dashboard` solo orquesta y renderiza

---

### Tipos Duplicados Backend ↔ Frontend
**Backend tipos** (`backend/app/routes.py:1-9`):
```python
OperationType = Literal["income", "outcome"]
Category = Literal["suppliers", "sales", "operational", "administrative", "others"]
BusinessType = Literal["B2B", "B2C"]
```

**Frontend tipos** (`frontend/src/lib/financial-types.ts:1-3`):
```typescript
export type OperationType = 'income' | 'outcome'
export type Category = 'suppliers' | 'sales' | 'operational' | 'administrative' | 'others'
export type BusinessType = 'B2B' | 'B2C'
```

**Comportamiento**: Mantenidos por separado, sin sincronización automática.

---

### Componentes UI Base Reutilizables
**Carpeta**: `frontend/src/components/ui/`

Archivos:
- `card.tsx` (línea 1-20): `Card`, `CardContent`, `CardDescription`
- `skeleton.tsx` (línea 1-10): `Skeleton` para loading states

**Uso en componentes**:
- `kpi-card.tsx:20-30` usa `<Card>` y `<Skeleton>`
- `income-outcome-chart.tsx:30` usa `<Card>`
- `profit-percent-chart.tsx:25` usa `<Card>`

---

### CSS Variables para Estilos Condicionales
**Archivo**: `frontend/src/components/dashboard/kpi-card.tsx:16-24`

```typescript
const variantStyles: Record<KPICardProps['variant'], { badge: string; icon: string }> = {
  income: {
    badge: 'bg-[var(--income-badge)] text-[var(--income-badge-fg)]',
    icon: 'text-[var(--income-badge-fg)]',
  },
  outcome: {
    badge: 'bg-[var(--outcome-badge)] text-[var(--outcome-badge-fg)]',
    icon: 'text-[var(--outcome-badge-fg)]',
  },
  // ... más variantes
}
```

**Aplicación** (línea 42):
```typescript
<span className={cn('p-1.5 rounded-lg', styles.badge)}>
```

No hay hardcoded colores, todo usa `var(--nombre)`.

---

### Nombres de Archivos en kebab-case
**Carpeta**: `frontend/src/components/dashboard/`

Archivos:
- `dashboard-header.tsx`
- `kpi-row.tsx`
- `kpi-card.tsx`
- `income-outcome-chart.tsx`
- `profit-percent-chart.tsx`

Import en componentes: `import { KPICard } from '@/components/dashboard/kpi-card'` (PascalCase en import)

---

### useEffect con Dependency Array Vacío para Fetch
**Archivo**: `frontend/src/App.tsx:29-43`

```typescript
useEffect(() => {
  fetchFinancialData()
    .then((movements) => {
      setMetrics(computeKPIs(movements))
      setMonthlyData(computeMonthlyData(movements))
    })
    .catch(() => {
      setError("No se pudo cargar la información financiera...")
    })
    .finally(() => {
      setLoading(false)
    })
}, [])  // ← Dependency array vacío
```

Fetch ocurre UNA SOLA VEZ al montar componente.

---

## Testing

### Backend: pytest con parametrización
**Archivo**: `backend/tests/test_routes.py`

Sin parametrización (problema):
```python
def test_metrics_endpoint_respects_date_filters():
    # Línea 35-44: prueba solo UNA combinación
```

Pero tiene:
```python
def test_filter_movements_by_date_includes_range_edges():
    # Línea 17-25: prueba caso específico
```

---

### Frontend: Vitest pero SIN tests de componentes
**Archivo**: `frontend/src/lib/financial-utils.test.ts`

Existe:
- Tests de `computeKPIs()` 
- Tests de `formatCurrency()`

No existe:
- Tests de componentes (ningún archivo `.test.tsx`)
- Tests de integración (App.tsx)
- Tests de hooks

---

## Estructura de Carpetas Real

### Backend
```
backend/
├── Dockerfile        ← Exposición debugpy (línea 7)
├── requirements.txt
├── app/
│   ├── __init__.py
│   ├── main.py       ← CORS ["*"] línea 8-12
│   └── routes.py     ← Todos endpoints, modelos
└── tests/
    ├── conftest.py
    └── test_routes.py
```

### Frontend
```
frontend/
├── vite.config.ts    ← Alias @ línea 10
├── tsconfig.app.json ← Paths configurado
├── package.json
├── src/
│   ├── main.tsx      ← Mount point
│   ├── App.tsx       ← Root component, fetch línea 16-21
│   ├── index.css     ← Tailwind global
│   ├── components/
│   │   ├── dashboard/
│   │   │   ├── dashboard-header.tsx
│   │   │   ├── kpi-row.tsx
│   │   │   ├── kpi-card.tsx
│   │   │   ├── income-outcome-chart.tsx
│   │   │   └── profit-percent-chart.tsx
│   │   └── ui/
│   │       ├── card.tsx
│   │       └── skeleton.tsx
│   └── lib/
│       ├── financial-types.ts    ← Tipos duplicados
│       ├── financial-utils.ts
│       ├── financial-utils.test.ts
│       ├── mock-data.ts
│       └── utils.ts              ← cn() helper
```

---

## Comportamientos Concretos a Mantener

| Comportamiento | Archivo | Línea | Estado |
|---|---|---|---|
| Type hints con `\|` | routes.py | 60-70 | ✅ Mantener |
| `Literal[]` para enums | routes.py | 1-9 | ✅ Mantener |
| Funciones puras sin mutación | routes.py | 92-180 | ✅ Mantener |
| Pydantic `response_model=` | routes.py | 286 | ✅ Mantener |
| Props con interfaz | kpi-card.tsx | 1-14 | ✅ Mantener |
| Importes `@/` | App.tsx | 1-10 | ✅ Mantener |
| Lógica en lib/ | financial-utils.ts | - | ✅ Mantener |
| Variantes CSS | kpi-card.tsx | 16-24 | ✅ Mantener |
| Kebab-case archivos | dashboard/ | - | ✅ Mantener |

