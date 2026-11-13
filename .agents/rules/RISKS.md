# Patrones Arriesgosos - Código Específico

## 🔴 CRÍTICOS

### 1. CORS Permite Todos los Orígenes

**Archivo**: `backend/app/main.py:8-12`

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],        # ← AQUÍ: acepta cualquier origen
    allow_credentials=True,
    allow_methods=["*"],        # ← AQUÍ: todos los métodos HTTP
    allow_headers=["*"],        # ← AQUÍ: todos los headers
)
```

**Impacto real**:
- `curl -H "Origin: https://attacker.com" http://localhost:8000/api/metrics` → funciona
- Sitio malicioso puede hacer `fetch("http://localhost:8000/api/metrics")`
- Si hay cookies, se envían automáticamente

**Afecta a agentes**: No pueden asumir CORS restringido

---

### 2. Debugpy Expuesto en Dockerfile

**Archivo**: `backend/Dockerfile:8`

```dockerfile
CMD ["python", "-m", "debugpy", "--listen", "0.0.0.0:5678", "-m", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]
```

**Comportamiento concreto**:
- Contrato expuesto en puerto 5678
- `0.0.0.0` = accesible desde cualquier red
- `--listen 0.0.0.0:5678` = debugger escucha ANTES de que Uvicorn inicie
- Alguien con acceso a 5678 puede pausar ejecución, inspeccionar memoria

**Línea problema**: No hay separación dev/prod

---

## 🟡 ALTOS

### 3. Enumeraciones Duplicadas sin Sincronización

**Backend** (`backend/app/routes.py:1-9`):
```python
OperationType = Literal["income", "outcome"]
Category = Literal["suppliers", "sales", "operational", "administrative", "others"]
BusinessType = Literal["B2B", "B2C"]
```

**Frontend** (`frontend/src/lib/financial-types.ts:1-3`):
```typescript
export type OperationType = 'income' | 'outcome'
export type Category = 'suppliers' | 'sales' | 'operational' | 'administrative' | 'others'
export type BusinessType = 'B2B' | 'B2C'
```

**Comportamiento observado**:
- Si backend cambia `Category` a incluir `"marketing"`, frontend NO se entera
- Backend línea 286 `generate_mock_movements(seed=42)` puede generar categoría "marketing"
- Frontend line 7 `computeKPIs()` recibe datos con "marketing"
- Frontend no valida, envía string desconocido
- Error silencioso o gráfico roto

**Sin tests de integración que verifiquen**: `frontend recibe tipos == backend envía tipos`

---

### 4. Datos Mock Idénticos Siempre (seed=42)

**Archivo**: `backend/app/routes.py:283-287`

```python
def get_metrics(...) -> list[FinancialMovement]:
    movements = generate_mock_movements(seed=42)  # ← Siempre 42
    filtered = filter_movements(movements, ...)
    return ensure_chronological_order(filtered)
```

Todas las funciones (línea 286, 310, 332, 358, 380, 399, 416):
```python
movements = generate_mock_movements(seed=42)
```

**Comportamiento observado**:
- `generate_mock_movements(seed=42)` retorna exactamente 360 movimientos
- Orden siempre igual
- Valores siempre iguales
- `random.seed(42)` línea 102 en generator

**Impacto en tests** (`backend/tests/test_routes.py:36-44`):
```python
def test_metrics_endpoint_respects_date_filters():
    base_response = client.get("/api/metrics")
    first_date = base_response.json()[0]["create_date"]  # ← SIEMPRE "2025-01-15"
    
    response = client.get("/api/metrics", params={"start_date": first_date, ...})
    assert response.status_code == 200
```

Test pasa porque datos son reproducibles, pero no valida variabilidad.

---

### 5. Sin Validación de Edge Cases en Endpoints

**Archivo**: `backend/app/routes.py:375-397`

```python
@router.get("/api/metrics/comparison", response_model=MetricsComparison)
def get_metrics_comparison(
    start_date: date = Query(...),
    end_date: date = Query(...),
    business_type: BusinessType | None = Query(default=None),
) -> MetricsComparison:
    movements = generate_mock_movements(seed=42)
    if business_type is not None:
        movements = [item for item in movements if item.business_type == business_type]

    current_movements = filter_movements(
        movements, start_date, end_date, category=None, operation_type=None
    )
    current_net = calculate_net_value(current_movements)
```

**Caso que falla**:
- Request: `GET /api/metrics/comparison?start_date=2025-12-01&end_date=2025-01-01`
- `start_date > end_date` → NO hay validación
- `duration = end_date - start_date` → valor NEGATIVO
- `previous_start = previous_end - duration` → calcula hacia adelante en lugar de atrás
- API retorna `delta_abs` y `delta_pct` ERRADOS
- Frontend gráfico muestra números errados

**Sin HTTPException(400)**, solo calcula algo incorrecto.

---

### 6. Lógica de Alertas Demasiado Simplista

**Archivo**: `backend/app/routes.py:199-213`

```python
def detect_outcome_alerts(
    summary: list[MetricsSummaryItem],
    threshold: float,
) -> list[MetricsAlert]:
    alerts: list[MetricsAlert] = []
    historical_outcomes: list[float] = []
    for item in summary:
        if historical_outcomes:
            baseline = sum(historical_outcomes) / len(historical_outcomes)
            if baseline > 0:
                increase_ratio = (item.outcome - baseline) / baseline
                if increase_ratio > threshold:  # ← Solo compara con promedio simple
                    alerts.append(MetricsAlert(...))
        historical_outcomes.append(item.outcome)
    return alerts
```

**Comportamiento concreto**:
- Baselines = promedio SIMPLE de todos los meses históricos
- Línea 210: `(item.outcome - baseline) / baseline > threshold`
- Si historial es `[1000, 1000, 1000, 5000]`, baseline = 2000
- Mes 5 con 2500 → ratio = 0.25 (debajo de threshold 0.3) → NO alerta
- Mes 6 con 3500 → ratio = 0.75 (arriba de threshold) → SÍ alerta
- Pero no detecta: seasonalidad (Diciembre siempre alto), tendencias, volatilidad

---

## 🟠 MEDIOS

### 7. Sin Retry Logic en Fetch

**Archivo**: `frontend/src/App.tsx:16-21`

```typescript
async function fetchFinancialData(): Promise<FinancialMovement[]> {
  const response = await fetch(`${API_BASE_URL}/api/metrics`);
  if (!response.ok) {
    throw new Error(`Failed to fetch financial data: ${response.status}`);
  }
  return response.json();
}
```

**Comportamiento**:
- Si red falla 1 vez → error lanzado
- Si timeout (backend tarda >30s) → error
- Si servidor retorna 503 (temporarily unavailable) → error
- NO hay `retry`, NO hay `delay`, NO hay `exponential backoff`

Usado en línea 33-40:
```typescript
fetchFinancialData()
  .then((movements) => { ... })
  .catch(() => {
    setError("No se pudo cargar...")  // ← Usuario ve error, recarga página manualmente
  })
```

---

### 8. API_BASE_URL sin Validación

**Archivo**: `frontend/src/App.tsx:13`

```typescript
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL ?? "";
//                                                        ↑ String vacío por defecto
```

**Comportamiento**:
- Si env var NO está set → `API_BASE_URL = ""`
- Línea 18: `fetch("" + "/api/metrics")` = `fetch("/api/metrics")`
- Llamada RELATIVA al dominio actual
- En dev: `http://localhost:5173/api/metrics` → falla (no existe)
- En prod: `https://example.com/api/metrics` → podría funcionar por suerte
- Sin validación explícita, error es silencioso

---

### 9. Estado Global sin Context API

**Archivo**: `frontend/src/App.tsx:26-29`

```typescript
function App() {
  const [metrics, setMetrics] = useState<KPIMetrics | null>(null);
  const [monthlyData, setMonthlyData] = useState<MonthlyDataPoint[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
```

**Comportamiento observable**:
- 4 `useState` en componente raíz
- Línea 48-61: Pasa `metrics` a `<KPIRow metrics={metrics} />`
- `KPIRow` pasa a `<KPICard ... />`
- Si se agrega más componentes → prop drilling profundo
- Si múltiples componentes leen `loading` → no hay caché, cada uno refetch

**Sin Context o Zustand**: refactor forzado si crece a 3+ niveles.

---

## ⚪ BAJOS

### 10. Period Label Hardcoded

**Archivo**: `frontend/src/App.tsx:48`

```typescript
<DashboardHeader period="2024 - Full Year" />
```

**Comportamiento**:
- Período NUNCA actualiza
- Enero 2026 → usuario ve "2024 - Full Year" (confuso)
- No es automático

---

### 11. Sin Logging Estructurado

**Backend**: Ningún `logging.info()`, solo silencioso en caso de error
**Frontend**: Solo `console.log()` en dev

Sin logs, imposible debugear:
- Por qué request A falló
- Qué datos recibió el endpoint
- Dónde pasó el error

---

### 12. Sin Tests de Integración

**Backend** (`backend/tests/test_routes.py`): Solo tests unitarios de endpoints aislados
**Frontend** (`frontend/src/lib/financial-utils.test.ts`): Solo tests de funciones puras

Si backend cambia formato de fecha ISO:
- Tests pasan (porque en test usan strings fijos)
- Frontend falla en producción (parseFloat lo convierte)
- Nadie lo detecta hasta que usuario lo encuentra

---

### 13. .env.example Incompleto

**Archivo**: `frontend/.env.example:1`

```env
VITE_API_BASE_URL=
```

Solo documenta UNA variable. ¿Hay otras?
- No dice si es REQUERIDA
- No dice qué pasa si está vacía
- No dice formato esperado (URL completa vs relativa)

