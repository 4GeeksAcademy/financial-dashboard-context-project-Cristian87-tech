# Hallazgos Organizados por Categoría

## 1. ARQUITECTURA Y SEPARACIÓN DE RESPONSABILIDADES

### ✅ CONVENCIONES (Qué está bien)

#### Backend: Router Centralizado
- **Archivo**: `backend/app/main.py:9`
- **Patrón**: `app.include_router(router)` importa todos los endpoints
- **Resultado**: Todos los endpoints en `routes.py` usan prefijo `/api/`
- **Mantener**: Sí

#### Frontend: Separación lib/ vs components/
- **Archivo**: `frontend/src/lib/financial-utils.ts` (cálculos), `frontend/src/components/` (renderizado)
- **Patrón**:
  - Cálculos: `computeKPIs()`, `computeMonthlyData()`, `formatCurrency()` en `lib/`
  - Componentes solo reciben `data` y renderizan, sin lógica
- **Ejemplo**: 
  ```typescript
  // ✅ CORRECTO: App.tsx línea 34
  setMetrics(computeKPIs(movements))
  
  // ✅ CORRECTO: KPICard solo renderiza props línea 45-50
  <span>{value}</span>
  ```
- **Mantener**: Sí - es crítico para testabilidad

#### Frontend: Componentes UI Reutilizables
- **Archivo**: `frontend/src/components/ui/card.tsx`, `ui/skeleton.tsx`
- **Patrón**: Card, Skeleton son primitivas, reutilizadas en dashboard/
- **Uso**:
  - `kpi-card.tsx:20-30` usa `<Card>` y `<Skeleton>`
  - `income-outcome-chart.tsx:30` usa `<Card>`
- **Mantener**: Sí

---

### ⚠️ RIESGOS ALTOS (Qué falla)

#### Tipos Duplicados sin Sincronización
- **Backend**: `backend/app/routes.py:1-9`
  ```python
  OperationType = Literal["income", "outcome"]
  Category = Literal["suppliers", "sales", "operational", "administrative", "others"]
  BusinessType = Literal["B2B", "B2C"]
  ```
- **Frontend**: `frontend/src/lib/financial-types.ts:1-3`
  ```typescript
  export type OperationType = 'income' | 'outcome'
  export type Category = 'suppliers' | 'sales' | 'operational' | 'administrative' | 'others'
  export type BusinessType = 'B2B' | 'B2C'
  ```
- **Problema**: Si backend agrega `"marketing"` a Category, frontend no se entera
- **Escenario fallo**: Backend envía `{"category": "marketing"}`, frontend recibe desconocido, error silencioso
- **Severidad**: 🟡 ALTO
- **Solución**: Generar tipos desde OpenAPI schema O tests de integración

---

## 2. TYPE SAFETY Y VALIDACIÓN

### ✅ CONVENCIONES (Qué está bien)

#### Backend: Type Hints con `|` (Union Modernos)
- **Archivo**: `backend/app/routes.py:60-70`
- **Patrón**:
  ```python
  start_date: date | None = Query(default=None)
  end_date: date | None = Query(default=None)
  operation_type: OperationType | None = Query(default=None)
  ```
- **Beneficio**: Sintaxis moderna Python 3.13, explícita, sin Optional[]
- **Mantener**: Sí - es el estándar del proyecto

#### Backend: Literal[] para Tipos Cerrados
- **Archivo**: `backend/app/routes.py:1-9`
- **Patrón**:
  ```python
  OperationType = Literal["income", "outcome"]
  GroupBy = Literal["day", "week", "month"]
  ```
- **Beneficio**: Swagger genera dropdown automático, TypeScript compatible
- **Mantener**: Sí

#### Backend: Pydantic BaseModel Obligatorio
- **Archivo**: `backend/app/routes.py:15-50` (modelos)
- **Patrón**: TODOS los schemas heredan de `BaseModel`:
  - `FinancialMovement` (línea 15)
  - `MetricsFacets` (línea 22)
  - `MetricsSummaryItem` (línea 30)
  - etc.
- **En endpoints**: `response_model=list[FinancialMovement]` (línea 286)
- **Beneficio**: Validación automática, Swagger schema generado
- **Mantener**: Sí - es crítico para documentación

#### Frontend: Props con Interface Explícita
- **Archivo**: `frontend/src/components/dashboard/kpi-card.tsx:1-14`
- **Patrón**:
  ```typescript
  interface KPICardProps {
    label: string
    value: string
    helperText: string
    icon: LucideIcon
    variant: 'income' | 'outcome' | 'profit' | 'profitPercent'
    loading?: boolean
  }
  ```
- **Aplicado en**: 
  - `dashboard-header.tsx:4`
  - `kpi-row.tsx:4`
  - `income-outcome-chart.tsx:3`
  - `profit-percent-chart.tsx:3`
- **Beneficio**: Type safety, IDE autocomplete, documentación automática
- **Mantener**: Sí

---

### ⚠️ RIESGOS MEDIOS (Qué falla)

#### Sin Validación Input en Endpoints
- **Archivo**: `backend/app/routes.py:375-397`
- **Código problemático**:
  ```python
  @router.get("/api/metrics/comparison", response_model=MetricsComparison)
  def get_metrics_comparison(
      start_date: date = Query(...),
      end_date: date = Query(...),
      business_type: BusinessType | None = Query(default=None),
  ):
      # NO hay validación: start_date > end_date no está permitido
      current_movements = filter_movements(
          movements, start_date, end_date, ...
      )
  ```
- **Escenario fallo**: Request con `start_date=2025-12-01&end_date=2025-01-01`
  - Calcula `duration = end_date - start_date` → negativo
  - `previous_start = previous_end - duration` → calcula mal
  - API retorna `delta_abs` y `delta_pct` errados
- **Severidad**: 🟡 ALTO
- **Solución**: Usar `Query()` con validadores:
  ```python
  start_date: date = Query(..., le=date.today())
  end_date: date = Query(..., le=date.today())
  # Agregar validación: if start_date > end_date: raise HTTPException(400)
  ```

---

## 3. TESTING Y REPRODUCIBILIDAD

### ⚠️ RIESGOS ALTOS (Qué falla)

#### Datos Mock Idénticos Siempre (seed=42)
- **Archivo**: `backend/app/routes.py:283-287` y líneas 310, 332, 358, 380, 399, 416
- **Código**:
  ```python
  def get_metrics(...) -> list[FinancialMovement]:
      movements = generate_mock_movements(seed=42)  # ← SIEMPRE 42
      filtered = filter_movements(movements, ...)
      return ensure_chronological_order(filtered)
  ```
- **Comportamiento**:
  - `generate_mock_movements(seed=42)` retorna EXACTAMENTE 360 movimientos
  - Orden siempre igual
  - Valores siempre iguales
  - Seed seteado línea 102: `random.seed(42)`
- **Problema en tests** (`backend/tests/test_routes.py:36-44`):
  ```python
  def test_metrics_endpoint_respects_date_filters():
      base_response = client.get("/api/metrics")
      first_date = base_response.json()[0]["create_date"]  # ← SIEMPRE "2025-01-15"
      response = client.get("/api/metrics", params={"start_date": first_date, ...})
      assert response.status_code == 200
  ```
  - Test pasa porque datos reproducibles, pero NO valida variabilidad
- **Severidad**: 🟡 ALTO
- **Solución**: Parametrizar tests, variar seed:
  ```python
  @pytest.mark.parametrize("seed", [42, 123, 456])
  def test_metrics_with_varying_seeds(seed):
      movements = generate_mock_movements(seed=seed)
      assert len(movements) > 0
  ```

#### Sin Tests de Integración Backend-Frontend
- **Backend**: `backend/tests/test_routes.py` solo tests unitarios aislados
- **Frontend**: `frontend/src/lib/financial-utils.test.ts` solo funciones puras
- **Problema**: Si backend cambia formato JSON, frontend no lo detecta hasta producción
  - Backend cambia: `"create_date": "2025-01-15"` → `"createdAt": "2025-01-15"`
  - Tests pasan porque backend test usa string fijo
  - Frontend envía GET, recibe campo desconocido, error silencioso
- **Severidad**: 🟡 ALTO
- **Solución**: Tests de integración que validen formato de respuesta real

#### Sin Parametrización de Tests Backend
- **Archivo**: `backend/tests/test_routes.py`
- **Problema**: Tests unitarios no usan `@pytest.mark.parametrize`
- **Ejemplo**: `test_filter_movements_by_date_includes_range_edges()` prueba 1 caso
- **Solución**: Agregar parámetros:
  ```python
  @pytest.mark.parametrize("category", ["suppliers", "sales", "operational"])
  def test_filter_by_category(category):
      ...
  ```

### ✅ CONVENCIONES (Qué está bien)

#### Funciones Puras sin Mutaciones
- **Archivo**: `backend/app/routes.py:92-180`
- **Funciones**:
  - `filter_movements_by_date()` (línea 92)
  - `filter_movements()` (línea 108)
  - `ensure_chronological_order()` (línea 123)
  - `summarize_movements()` (línea 157)
- **Patrón**:
  ```python
  # Línea 99: NO MUTATES original
  filtered = [
      movement for movement in filtered 
      if movement.create_date >= start_date
  ]
  ```
- **Beneficio**: Testeable sin setup/teardown, composable
- **Mantener**: Sí

---

## 4. SEGURIDAD

### 🔴 RIESGOS CRÍTICOS (Qué rompe todo)

#### CORS Permite Todos los Orígenes
- **Archivo**: `backend/app/main.py:8-12`
- **Código**:
  ```python
  app.add_middleware(
      CORSMiddleware,
      allow_origins=["*"],        # ← CRÍTICO: cualquier origen
      allow_credentials=True,
      allow_methods=["*"],        # ← CRÍTICO: todos los métodos
      allow_headers=["*"],        # ← CRÍTICO: todos los headers
  )
  ```
- **Impacto real**:
  - `curl -H "Origin: https://attacker.com" http://localhost:8000/api/metrics` → funciona
  - Sitio malicioso ejecuta: `fetch("http://localhost:8000/api/metrics")` → accede a datos
  - Si hay cookies, se envían automáticamente
- **Severidad**: 🔴 CRÍTICO
- **Solución**:
  ```python
  allow_origins=["http://localhost:5173", "https://yourdomain.com"]
  ```

#### Debugpy Expuesto en Dockerfile
- **Archivo**: `backend/Dockerfile:8`
- **Código**:
  ```dockerfile
  CMD ["python", "-m", "debugpy", "--listen", "0.0.0.0:5678", "-m", "uvicorn", ...]
  ```
- **Comportamiento concreto**:
  - Debugger escucha en puerto 5678
  - `0.0.0.0` = accesible desde cualquier red (local + internet)
  - Alguien con acceso a 5678 puede:
    - Pausar ejecución
    - Inspeccionar memoria
    - Modificar variables en tiempo real
- **Severidad**: 🔴 CRÍTICO
- **Problema adicional**: No hay separación dev/prod
- **Solución**: Crear `Dockerfile.dev` y `Dockerfile` separados:
  ```dockerfile
  # Dockerfile (producción)
  CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
  
  # Dockerfile.dev (desarrollo)
  CMD ["python", "-m", "debugpy", "--listen", "127.0.0.1:5678", "-m", "uvicorn", ...]
  ```

---

## 5. DESARROLLO Y EXPERIENCIA DEL DESARROLLADOR (DX)

### ✅ CONVENCIONES (Qué está bien)

#### Importes Limpio con Alias @/
- **Archivo**: `frontend/tsconfig.app.json:7-9`
- **Configuración**:
  ```json
  "paths": {
    "@/*": ["./src/*"]
  }
  ```
- **En código** (`frontend/src/App.tsx:1-10`):
  ```typescript
  import { DashboardHeader } from "@/components/dashboard/dashboard-header"
  import { KPIRow } from "@/components/dashboard/kpi-row"
  import { computeKPIs } from "@/lib/financial-utils"
  ```
- **Aplicado en**: TODOS los archivos TypeScript
- **Beneficio**: Importes limpios, no `../../../`, IDE sabe resolver
- **Mantener**: Sí

#### Nombres de Archivos en kebab-case
- **Archivo**: `frontend/src/components/dashboard/`
- **Nombres**:
  - `dashboard-header.tsx`
  - `kpi-row.tsx`
  - `kpi-card.tsx`
  - `income-outcome-chart.tsx`
  - `profit-percent-chart.tsx`
- **Import**: `import { KPICard } from '@/components/dashboard/kpi-card'` (PascalCase en import)
- **Beneficio**: Consistencia, fácil encontrar archivo
- **Mantener**: Sí

#### Estilos Dinámicos con CSS Variables
- **Archivo**: `frontend/src/components/dashboard/kpi-card.tsx:16-24`
- **Patrón**:
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
  }
  ```
- **Aplicación** (línea 42):
  ```typescript
  <span className={cn('p-1.5 rounded-lg', variantStyles[variant].badge)}>
  ```
- **Beneficio**: Dinámico, permite cambiar tema sin recompilación
- **Mantener**: Sí

---

### ⚠️ RIESGOS MEDIOS (Qué degrada DX)

#### Fetch sin Retry Logic
- **Archivo**: `frontend/src/App.tsx:16-21`
- **Código**:
  ```typescript
  async function fetchFinancialData(): Promise<FinancialMovement[]> {
    const response = await fetch(`${API_BASE_URL}/api/metrics`);
    if (!response.ok) {
      throw new Error(`Failed to fetch financial data: ${response.status}`);
    }
    return response.json();
  }
  ```
- **Comportamiento**:
  - Network hiccup → error inmediato
  - Backend timeout > 30s → error inmediato
  - Backend 503 (temporarily unavailable) → error inmediato
- **En línea 33-40**:
  ```typescript
  .catch(() => {
    setError("No se pudo cargar...")  // ← Usuario debe recargar página
  })
  ```
- **Severidad**: 🟠 MEDIO
- **Solución**: Agregar retry exponential backoff:
  ```typescript
  async function fetchWithRetry(url, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
      try {
        const response = await fetch(url);
        if (response.ok) return response;
      } catch (e) {
        if (i === maxRetries - 1) throw e;
        await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000));
      }
    }
  }
  ```

#### API_BASE_URL sin Validación
- **Archivo**: `frontend/src/App.tsx:13`
- **Código**:
  ```typescript
  const API_BASE_URL = import.meta.env.VITE_API_BASE_URL ?? "";
  ```
- **Problema**:
  - Si env var NO está set → `API_BASE_URL = ""`
  - Línea 18: `fetch("" + "/api/metrics")` = `fetch("/api/metrics")`
  - En dev: relativa falla (no existe ruta local)
  - En prod: podría funcionar por suerte si backend en mismo dominio
- **Severidad**: 🟠 MEDIO
- **Solución**:
  ```typescript
  const API_BASE_URL = import.meta.env.VITE_API_BASE_URL;
  if (!API_BASE_URL) {
    throw new Error('VITE_API_BASE_URL is not set');
  }
  ```

#### Estado Global sin Context API
- **Archivo**: `frontend/src/App.tsx:26-29`
- **Código**:
  ```typescript
  function App() {
    const [metrics, setMetrics] = useState<KPIMetrics | null>(null);
    const [monthlyData, setMonthlyData] = useState<MonthlyDataPoint[]>([]);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState<string | null>(null);
  ```
- **Problema**:
  - 4 `useState` en raíz
  - Pasa `metrics` → `KPIRow` → `KPICard` (prop drilling)
  - Si crece a 5+ componentes → refactor urgente
- **Severidad**: 🟠 MEDIO (no es crisis ahora, pero preventivo)
- **Solución**:
  ```typescript
  const DashboardContext = React.createContext<DashboardContextType | null>(null);
  <DashboardContext.Provider value={{ metrics, loading, error }}>
    <Dashboard />
  </DashboardContext.Provider>
  ```

#### .env.example Incompleto
- **Archivo**: `frontend/.env.example:1`
- **Contenido**:
  ```env
  VITE_API_BASE_URL=
  ```
- **Problema**: No documenta:
  - Es REQUERIDA o opcional
  - Formato esperado (URL completa, relativa, etc.)
  - Qué pasa si está vacía
- **Severidad**: 🟠 MEDIO
- **Solución**:
  ```env
  # Backend API base URL (required)
  # Format: http://localhost:8000 or https://api.yourdomain.com
  VITE_API_BASE_URL=http://localhost:8000
  ```

---

### ⚠️ RIESGOS BAJOS (Qué confunde)

#### Period Label Hardcoded
- **Archivo**: `frontend/src/App.tsx:48`
- **Código**:
  ```typescript
  <DashboardHeader period="2024 - Full Year" />
  ```
- **Problema**: Año nunca se actualiza
  - Enero 2026 → usuario ve "2024 - Full Year" (confuso)
- **Severidad**: ⚪ BAJO (UX, no funcional)
- **Solución**:
  ```typescript
  const currentYear = new Date().getFullYear();
  const period = `${currentYear} - Full Year`;
  <DashboardHeader period={period} />
  ```

#### Sin Logging Estructurado
- **Backend**: Ningún `logging.info()`, solo silencioso en error
- **Frontend**: Solo `console.log()` en dev
- **Problema**: Imposible debugear:
  - Por qué request A falló
  - Qué datos recibió endpoint
  - Dónde pasó el error
- **Severidad**: ⚪ BAJO (funciona, pero difícil mantener)
- **Solución**:
  ```python
  # Backend
  import logging
  logger = logging.getLogger(__name__)
  logger.info(f"Filtering movements: start={start_date}, end={end_date}")
  ```

---

## 6. DOCUMENTACIÓN

### ⚠️ RIESGOS BAJOS (Qué complica)

#### .env.example Incompleto
- **Ya documentado en sección DX**
- **Severidad**: ⚪ BAJO

---

## 7. LÓGICA DE NEGOCIOS

### ⚠️ RIESGOS ALTOS (Qué falla)

#### Alertas Demasiado Simplistas
- **Archivo**: `backend/app/routes.py:199-213`
- **Código**:
  ```python
  def detect_outcome_alerts(summary: list[MetricsSummaryItem], threshold: float):
      alerts: list[MetricsAlert] = []
      historical_outcomes: list[float] = []
      for item in summary:
          if historical_outcomes:
              baseline = sum(historical_outcomes) / len(historical_outcomes)
              if baseline > 0:
                  increase_ratio = (item.outcome - baseline) / baseline
                  if increase_ratio > threshold:  # ← Solo promedio simple
                      alerts.append(MetricsAlert(...))
          historical_outcomes.append(item.outcome)
      return alerts
  ```
- **Problema**:
  - Baseline = promedio SIMPLE de todos los meses
  - Si historial es `[1000, 1000, 1000, 5000]`, baseline = 2000
  - Mes 5 con 2500 → ratio = 0.25 (debajo threshold 0.3) → no alerta
  - Mes 6 con 3500 → ratio = 0.75 (arriba) → sí alerta
  - NO detecta: seasonalidad (Dic siempre alto), tendencias, volatilidad
- **Severidad**: 🟡 ALTO (lógica incompleta)
- **Solución**: Implementar modelo estadístico:
  - Media móvil exponencial (EMA) en lugar de promedio simple
  - Desviación estándar para detectar outliers
  - Comparar con mismo mes del año anterior (seasonalidad)

---

## RESUMEN POR SEVERIDAD

### 🔴 CRÍTICOS (Debes arreglar ANTES de producción)
1. **CORS `["*"]`** - `main.py:8-12` - Seguridad
2. **Debugpy expuesto** - `Dockerfile:8` - Seguridad

### 🟡 ALTOS (Debes arreglar PRONTO)
3. **Tipos duplicados** - `routes.py` vs `types.ts` - Arquitectura
4. **seed=42 siempre** - `routes.py:286+` - Testing
5. **Sin validación input** - `routes.py:375-397` - Type safety
6. **Alertas simplistas** - `routes.py:199-213` - Lógica de negocios

### 🟠 MEDIOS (Debes arreglar DESPUÉS)
7. **Fetch sin retry** - `App.tsx:16-21` - DX
8. **API_BASE_URL sin validación** - `App.tsx:13` - DX
9. **Estado global** - `App.tsx:26-29` - DX
10. **Sin tests integración** - Backend-Frontend - Testing
11. **.env.example incompleto** - `frontend/.env.example` - Documentación

### ⚪ BAJOS (Opcional, mejora UX)
12. **Period hardcoded** - `App.tsx:48` - UX
13. **Sin logging** - Ambos - Mantenibilidad

---

## TABLA DE REFERENCIA RÁPIDA POR ARCHIVO

| Archivo | Categoría | Hallazgo | Severidad |
|---------|-----------|----------|-----------|
| `main.py:8-12` | Seguridad | CORS `["*"]` | 🔴 CRÍTICO |
| `Dockerfile:8` | Seguridad | debugpy 0.0.0.0:5678 | 🔴 CRÍTICO |
| `routes.py:1-9` | Arquitectura | Tipos duplicados | 🟡 ALTO |
| `routes.py:60-70` | Type Safety | Type hints `\|` | ✅ BUENO |
| `routes.py:92-180` | Testing | Funciones puras | ✅ BUENO |
| `routes.py:199-213` | Lógica | Alertas simplistas | 🟡 ALTO |
| `routes.py:286+` | Testing | seed=42 siempre | 🟡 ALTO |
| `routes.py:375-397` | Type Safety | Sin validación | 🟡 ALTO |
| `types.ts:1-3` | Arquitectura | Tipos duplicados | 🟡 ALTO |
| `App.tsx:13` | DX | API_BASE_URL sin validación | 🟠 MEDIO |
| `App.tsx:16-21` | DX | Fetch sin retry | 🟠 MEDIO |
| `App.tsx:26-29` | DX | Estado global | 🟠 MEDIO |
| `App.tsx:48` | DX | Period hardcoded | ⚪ BAJO |
| `kpi-card.tsx:1-14` | Type Safety | Props interface | ✅ BUENO |
| `kpi-card.tsx:16-24` | DX | CSS variables | ✅ BUENO |
| `tsconfig.app.json:7-9` | DX | Alias @/ | ✅ BUENO |
| `dashboard/` | DX | kebab-case | ✅ BUENO |
| `.env.example` | Documentación | Incompleto | 🟠 MEDIO |

