# Reglas Propuestas del Proyecto

Cada regla cita hechos concretos del repositorio (archivo:línea, código observado).

---

## 🔴 REGLAS CRÍTICAS (Seguridad - No Negociables)

### REGLA #1: Whitelist explícito en CORS

**Regla**: Nunca uses `allow_origins=["*"]` en CORSMiddleware. Solo whitelist dominios específicos.

**Hecho del repo**: 
- `backend/app/main.py:8-12` actualmente permite `allow_origins=["*"]`
- Esto permite que cualquier sitio web acceda a la API

**Verificación**: 
```bash
grep -n 'allow_origins' backend/app/main.py
```
Debe mostrar: `allow_origins=["http://localhost:5173", "https://yourdomain.com"]`

**Severidad**: 🔴 CRÍTICO - Seguridad

---

### REGLA #2: Debugpy solo en desarrollo, nunca expuesto en 0.0.0.0

**Regla**: El debugger nunca escucha en `0.0.0.0:5678`. Usa `127.0.0.1:5678` en desarrollo. Crea `Dockerfile.dev` y `Dockerfile` separados.

**Hecho del repo**:
- `backend/Dockerfile:8` actual: `python -m debugpy --listen 0.0.0.0:5678 -m uvicorn`
- `0.0.0.0` expone el debugger públicamente a cualquier red

**Verificación**:
```bash
grep -n "debugpy" backend/Dockerfile
```
Debe NO contener `0.0.0.0:5678`

**Severidad**: 🔴 CRÍTICO - Seguridad

---

## 🟡 REGLAS DE ARQUITECTURA (No Negociables)

### REGLA #3: Type Hints con `|` en TODOS los parámetros y retornos

**Regla**: Usa `parametro: tipo | None` en lugar de `Optional[]` o tipos sin hint. TODOS los parámetros y retornos deben tener type hint.

**Hecho del repo**:
- `backend/app/routes.py:60-70` usa: `start_date: date | None = Query(default=None)`
- `backend/app/routes.py:1-9` define tipos como: `OperationType = Literal["income", "outcome"]`

**Verificación**:
```bash
grep -n "def " backend/app/routes.py | grep -v " -> " 
```
Debe retornar cero líneas (todos los def tienen `-> TipoRetorno`)

**Severidad**: 🟡 ALTO - Type Safety

---

### REGLA #4: Pydantic BaseModel obligatorio en TODOS los schemas

**Regla**: TODOS los modelos que circulan entre endpoints deben heredar de `Pydantic.BaseModel`. TODOS los endpoints deben tener `response_model=`.

**Hecho del repo**:
- `backend/app/routes.py:15-50` define 6 modelos: `FinancialMovement`, `MetricsFacets`, `MetricsSummaryItem`, `TopCategoryItem`, `MetricsComparison`, `MetricsAlert`
- `backend/app/routes.py:286` usa: `@router.get("/api/metrics", response_model=list[FinancialMovement])`

**Verificación**:
```bash
grep -n "@router" backend/app/routes.py | grep -v "response_model"
```
Debe retornar cero líneas

**Severidad**: 🟡 ALTO - Documentación automática + validación

---

### REGLA #5: Literal[] para tipos cerrados, nunca strings libres

**Regla**: Cuando un parámetro tiene valores fijos, usa `Literal["valor1", "valor2"]`, no `str`.

**Hecho del repo**:
- `backend/app/routes.py:1-9` define: `OperationType = Literal["income", "outcome"]`
- `GroupBy = Literal["day", "week", "month"]`
- `BusinessType = Literal["B2B", "B2C"]`

**Verificación**:
```bash
grep -n "Query.*str" backend/app/routes.py
```
Debería retornar cero líneas (todos deben ser Literal o tipos cerrados)

**Severidad**: 🟡 ALTO - Type Safety + Swagger

---

### REGLA #6: Funciones puras sin mutación en helpers

**Regla**: Funciones helper que procesan datos deben ser puras: retornan nueva lista/objeto, NUNCA mutanomutacian el input.

**Hecho del repo**:
- `backend/app/routes.py:92-108` usa: `filtered = [movement for movement in filtered if ...]` (crea nueva lista)
- NO usa `.pop()`, `.remove()`, o asignación a índices

**Verificación**:
```bash
grep -E "\.pop\(|\.remove\(|\.sort\(\)" backend/app/routes.py
```
Debe retornar cero líneas (en funciones helper)

**Severidad**: 🟡 ALTO - Testabilidad

---

### REGLA #7: Lógica de cálculo NUNCA en componentes React

**Regla**: Toda lógica de cálculo va en `lib/`. Los componentes React solo reciben `props` y renderizan.

**Hecho del repo**:
- `frontend/src/lib/financial-utils.ts` contiene: `computeKPIs()`, `computeMonthlyData()`, `formatCurrency()`
- `frontend/src/components/dashboard/kpi-card.tsx:45-50` solo renderiza: `<span>{value}</span>`

**Verificación**:
```bash
grep -n "const.*=" frontend/src/components/**/*.tsx | grep -v "props\|state\|const variant"
```
No debe contener cálculos complejos, solo transformación de props

**Severidad**: 🟡 ALTO - Testabilidad + Separación

---

### REGLA #8: Importes SIEMPRE con alias @/, NUNCA ../../../

**Regla**: Usa `import { X } from "@/folder/file"`, NUNCA `import { X } from "../../folder/file"`.

**Hecho del repo**:
- `frontend/tsconfig.app.json:7-9` define: `"@/*": ["./src/*"]`
- `frontend/src/App.tsx:1-10` usa: `import { DashboardHeader } from "@/components/dashboard/dashboard-header"`

**Verificación**:
```bash
grep -r "\.\./\.\./\.\." frontend/src --include="*.tsx" --include="*.ts"
```
Debe retornar cero líneas

**Severidad**: 🟡 ALTO - DX

---

### REGLA #9: Props con interface explícita, NUNCA tipado anónimo

**Regla**: TODO componente React debe tener `interface ComponentProps { ... }` antes del `export function`.

**Hecho del repo**:
- `frontend/src/components/dashboard/kpi-card.tsx:1-14` define:
  ```typescript
  interface KPICardProps {
    label: string
    value: string
    icon: LucideIcon
    variant: 'income' | 'outcome' | 'profit' | 'profitPercent'
    loading?: boolean
  }
  ```

**Verificación**:
```bash
grep -B5 "export function" frontend/src/components/**/*.tsx | grep "interface"
```
Debe encontrar interface para CADA componente

**Severidad**: 🟡 ALTO - Type Safety

---

### REGLA #10: Nombres de archivo kebab-case, imports PascalCase

**Regla**: Archivos en `components/` usan `kebab-case`: `my-component.tsx`. Import del componente usa PascalCase.

**Hecho del repo**:
- Archivos: `dashboard-header.tsx`, `kpi-card.tsx`, `income-outcome-chart.tsx`
- Import: `import { DashboardHeader } from "@/components/dashboard-header"`

**Verificación**:
```bash
ls frontend/src/components/dashboard/ | grep -E "[A-Z]"
```
Debe retornar cero líneas (sin mayúsculas en nombres de archivo)

**Severidad**: 🟡 ALTO - Consistencia

---

## 🟠 REGLAS DE VALIDACIÓN (Recomendadas para robustez)

### REGLA #11: Valida ranges en Query parameters

**Regla**: Si un endpoint recibe `start_date` y `end_date`, valida que `start_date <= end_date` antes de procesar.

**Hecho del repo**:
- `backend/app/routes.py:375-397` NO valida esto actualmente
- Request con `start_date=2025-12-01&end_date=2025-01-01` produce delta incorrecto

**Verificación**:
```bash
grep -A10 "start_date.*end_date" backend/app/routes.py | grep -i "if.*start.*end"
```
Debe contener validación explícita

**Severidad**: 🟠 MEDIO - Robustez

---

### REGLA #12: Mock data con seed VARIADO en tests, NUNCA seed=42 siempre

**Regla**: Tests que usan `generate_mock_movements()` deben variar el seed. Usa `@pytest.mark.parametrize("seed", [42, 123, 456])`.

**Hecho del repo**:
- `backend/app/routes.py:286, 310, 332, 358, 380, 399, 416` usan: `seed=42` (SIEMPRE igual)
- `backend/tests/test_routes.py:36-44` prueba solo UN caso

**Verificación**:
```bash
grep "seed=" backend/tests/test_routes.py | sort | uniq
```
Debe mostrar múltiples seeds, no solo `seed=42`

**Severidad**: 🟠 MEDIO - Testing

---

### REGLA #13: Fetch con retry exponencial backoff

**Regla**: Fetch en frontend debe tener reintentos con exponential backoff (base 2, máximo 3 intentos).

**Hecho del repo**:
- `frontend/src/App.tsx:16-21` actual NO tiene retry
- Un fallo de red = error inmediato al usuario

**Verificación**:
```bash
grep -A10 "async function fetchFinancialData" frontend/src/App.tsx | grep -i "retry\|delay\|setTimeout"
```
Debe contener lógica de retry

**Severidad**: 🟠 MEDIO - UX

---

### REGLA #14: API_BASE_URL validado al startup

**Regla**: `API_BASE_URL` debe validarse al arranque. Si está vacío, lanza error claro.

**Hecho del repo**:
- `frontend/src/App.tsx:13` actual: `const API_BASE_URL = import.meta.env.VITE_API_BASE_URL ?? ""`
- Si vacío, el error es silencioso

**Verificación**:
```bash
grep -A5 "API_BASE_URL" frontend/src/App.tsx | grep "if.*!API_BASE_URL\|throw"
```
Debe contener validación explícita

**Severidad**: 🟠 MEDIO - DX

---

### REGLA #15: Documenta TODAS las variables de env en .env.example

**Regla**: `.env.example` debe documentar CADA variable: si es requerida, formato esperado, valor por defecto (si aplica).

**Hecho del repo**:
- `frontend/.env.example:1` actual solo tiene: `VITE_API_BASE_URL=`
- No documenta si es requerida, formato, o qué pasa si vacía

**Verificación**:
```bash
head -20 frontend/.env.example
```
Debe tener comentarios explicativos para CADA variable

**Severidad**: 🟠 MEDIO - Documentación

---

### REGLA #16: Tests backend parametrizados

**Regla**: Tests que prueban comportamiento variable deben usar `@pytest.mark.parametrize`.

**Hecho del repo**:
- `backend/tests/test_routes.py` NO usa parametrización
- Cada caso de prueba debe testar múltiples valores

**Verificación**:
```bash
grep "@pytest.mark.parametrize" backend/tests/test_routes.py | wc -l
```
Debe tener al menos 3-5 uses de parametrize

**Severidad**: 🟠 MEDIO - Testing

---

### REGLA #17: Tests de integración backend-frontend

**Regla**: Debe haber tests que validen que el formato JSON del backend coincida con lo que espera el frontend.

**Hecho del repo**:
- Backend: `backend/app/routes.py:286` retorna `list[FinancialMovement]`
- Frontend: `frontend/src/App.tsx:20` recibe y parsea con `computeKPIs(movements)`
- Sin tests que validen que ambos usan mismo formato

**Verificación**:
```bash
grep -r "test.*integration\|test.*backend.*frontend" backend/tests frontend/src
```
Debe retornar al menos 1 archivo de integración

**Severidad**: 🟠 MEDIO - Testing

---

### REGLA #18: Alertas con lógica estadística, no solo promedio

**Regla**: Alertas de anomalía deben usar media móvil exponencial (EMA) + desviación estándar, NO solo promedio simple.

**Hecho del repo**:
- `backend/app/routes.py:199-213` actual solo compara: `baseline = sum() / len()`
- NO detecta seasonalidad ni tendencias

**Verificación**:
```bash
grep -A20 "def detect_outcome_alerts" backend/app/routes.py | grep -i "std\|deviation\|ema\|exponential"
```
Debe contener lógica estadística avanzada

**Severidad**: 🟠 MEDIO - Lógica de Negocios

---

## ✅ REGLAS OBLIGATORIAS (Patrones a Mantener)

### REGLA #19: Type hints con uniones modernas

**Regla**: En Python 3.13, usa `tipo1 | tipo2` en lugar de `Union[tipo1, tipo2]`.

**Hecho del repo**: `backend/app/routes.py:60-70` ya lo hace correctamente

**Severidad**: ✅ BUENO - Mantener

---

### REGLA #20: Router centralizado en routes.py

**Regla**: TODO endpoint en un solo archivo `routes.py`, importado en `main.py`.

**Hecho del repo**: `backend/app/main.py:9` hace `app.include_router(router)`

**Severidad**: ✅ BUENO - Mantener

---

### REGLA #21: CSS variables para estilos dinámicos

**Regla**: Estilos condicionales usan `var(--nombre-css)`, NUNCA hardcoded colors.

**Hecho del repo**: `frontend/src/components/dashboard/kpi-card.tsx:16-24` usa `bg-[var(--income-badge)]`

**Severidad**: ✅ BUENO - Mantener

---

### REGLA #22: useEffect fetch solo al montar componente

**Regla**: useEffect que hace fetch debe tener dependency array vacío `[]` para ejecutar UNA SOLA VEZ.

**Hecho del repo**: `frontend/src/App.tsx:29-43` tiene `}, [])`

**Severidad**: ✅ BUENO - Mantener

---

---

## 📊 RESUMEN DE REGLAS

| ID | Regla | Categoría | Severidad | Verificación |
|----|-------|-----------|-----------|--------------|
| 1 | Whitelist CORS | Seguridad | 🔴 CRÍTICO | `grep allow_origins` |
| 2 | Debugpy 127.0.0.1 | Seguridad | 🔴 CRÍTICO | `grep debugpy` |
| 3 | Type hints `\|` | Arquitectura | 🟡 ALTO | `grep "def " \| grep " -> "` |
| 4 | Pydantic + response_model | Arquitectura | 🟡 ALTO | `grep response_model` |
| 5 | Literal[] para enums | Arquitectura | 🟡 ALTO | `grep Literal` |
| 6 | Funciones puras | Arquitectura | 🟡 ALTO | `grep -E ".pop\|.remove\|.sort\(\)"` |
| 7 | Lógica en lib/ | Arquitectura | 🟡 ALTO | grep en componentes |
| 8 | Alias @/ | Arquitectura | 🟡 ALTO | `grep "\.\./\.\./\.\."` |
| 9 | Props interface | Arquitectura | 🟡 ALTO | `grep "interface.*Props"` |
| 10 | kebab-case archivos | Arquitectura | 🟡 ALTO | `ls \| grep [A-Z]` |
| 11 | Valida ranges | Validación | 🟠 MEDIO | `grep "if.*start.*end"` |
| 12 | Seed variado | Testing | 🟠 MEDIO | `grep "seed=" \| sort \| uniq` |
| 13 | Fetch con retry | DX | 🟠 MEDIO | `grep retry` |
| 14 | API_BASE_URL validado | DX | 🟠 MEDIO | `grep "if.*!API_BASE_URL"` |
| 15 | .env.example documentado | Documentación | 🟠 MEDIO | `grep "^#"` en .env.example |
| 16 | Tests parametrizados | Testing | 🟠 MEDIO | `grep "@pytest.mark.parametrize"` |
| 17 | Tests integración | Testing | 🟠 MEDIO | `grep "integration"` en tests |
| 18 | Alertas con estadística | Lógica | 🟠 MEDIO | `grep "std\|ema"` |
| 19 | Union con `\|` | Patrón | ✅ BUENO | Mantener |
| 20 | Router centralizado | Patrón | ✅ BUENO | Mantener |
| 21 | CSS variables | Patrón | ✅ BUENO | Mantener |
| 22 | useEffect `[]` | Patrón | ✅ BUENO | Mantener |

