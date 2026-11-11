# Estructura del Proyecto - Financial Dashboard

## 📋 Resumen Ejecutivo

**Financial Dashboard** es una aplicación full-stack para gestionar y visualizar métricas financieras. Consta de:
- **Backend**: API REST con FastAPI (Python) en puerto 8000
- **Frontend**: Aplicación React + TypeScript con Vite en puerto 5173
- **Orquestación**: Docker Compose

---

## 🏗️ Arquitectura General

```
financial-dashboard/
├── backend/          # FastAPI REST API (Python)
├── frontend/         # React + TypeScript SPA (Node.js)
├── docker-compose.yml
├── README.md & README.es.md
└── AGENTS.md
```

---

## 🔌 Entry Points

### Backend Entry Point
**Archivo**: [backend/app/main.py](/workspaces/financial-dashboard-context-project-Cristian87-tech/backend/app/main.py)
- **Framework**: FastAPI
- **Punto de inicio**: Crea instancia `FastAPI(title="Financial Metrics API")`
- **Puerto**: 8000
- **Middleware**: CORS habilitado para todos los orígenes
- **Router**: Incluye rutas desde `app.routes`

**Comando de inicio**:
```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Frontend Entry Point
**Archivo**: [frontend/src/main.tsx](/workspaces/financial-dashboard-context-project-Cristian87-tech/frontend/src/main.tsx)
- **Framework**: React 19 + TypeScript
- **Build tool**: Vite
- **Puerto**: 5173 (dev), 4173 (preview)
- **Punto de montaje**: `#root` en [frontend/index.html](/workspaces/financial-dashboard-context-project-Cristian87-tech/frontend/index.html)
- **Componente raíz**: [App.tsx](/workspaces/financial-dashboard-context-project-Cristian87-tech/frontend/src/App.tsx)

**Scripts disponibles**:
```json
{
  "dev": "vite",                          // Desarrollo con hot reload
  "build": "tsc -b && vite build",       // Build de producción
  "lint": "eslint .",                    // Linting
  "preview": "vite preview",              // Vista previa del build
  "test": "vitest run",                  // Tests una sola vez
  "test:watch": "vitest",                // Tests en watch mode
  "test:coverage": "vitest run --coverage" // Cobertura de tests
}
```

---

## 🎯 Servicios y Módulos

### Backend Services

#### 1. **API REST Endpoints** ([backend/app/routes.py](/workspaces/financial-dashboard-context-project-Cristian87-tech/backend/app/routes.py))

**Modelos de datos**:
```python
FinancialMovement:
  - create_date: date
  - amount: float
  - operation_type: "income" | "outcome"
  - category: "suppliers" | "sales" | "operational" | "administrative" | "others"
  - business_type: "B2B" | "B2C"

MetricsFacets:
  - operation_types: list
  - business_types: list
  - categories: list
  - min_date, max_date: date

MetricsSummaryItem:
  - period: str
  - income: float
  - outcome: float
  - net: float

TopCategoryItem:
  - category: Category
  - operation_type: OperationType
  - total_amount: float
```

**Enumeraciones**:
- **OperationType**: `"income"` | `"outcome"`
- **Category**: `"suppliers"` | `"sales"` | `"operational"` | `"administrative"` | `"others"`
- **BusinessType**: `"B2B"` | `"B2C"`
- **GroupBy**: `"day"` | `"week"` | `"month"`

**Rutas principales** (inferidas de modelo):
- `GET /api/metrics` - Obtiene todos los movimientos financieros
- (Verificar en routes.py para endpoints completos)

#### 2. **Testing** ([backend/tests/](/workspaces/financial-dashboard-context-project-Cristian87-tech/backend/tests/))
- **conftest.py**: Configuración de fixtures pytest
- **test_routes.py**: Tests de endpoints API

### Frontend Services

#### 1. **Core App Component** ([src/App.tsx](/workspaces/financial-dashboard-context-project-Cristian87-tech/frontend/src/App.tsx))
- **Responsabilidad**: Componente raíz que orquesta la visualización
- **Funciones principales**:
  - Fetch de datos desde `/api/metrics`
  - Cálculo de KPIs y datos mensuales
  - Manejo de estados: `metrics`, `monthlyData`, `loading`, `error`
  - Renderizado de dashboard

#### 2. **Componentes Dashboard**
```
src/components/dashboard/
├── dashboard-header.tsx       # Encabezado con período
├── kpi-row.tsx               # Fila de KPIs (cards)
├── kpi-card.tsx              # Card individual de KPI
├── income-outcome-chart.tsx   # Gráfico ingresos vs egresos
└── profit-percent-chart.tsx   # Gráfico de rentabilidad
```

#### 3. **Componentes UI** (base)
```
src/components/ui/
├── card.tsx                   # Componente card reutilizable
└── skeleton.tsx               # Skeleton loader para estados de carga
```

#### 4. **Utilidades y Tipos**
```
src/lib/
├── financial-types.ts         # Tipos TypeScript de dominio
├── financial-utils.ts         # Funciones de cálculo
├── financial-utils.test.ts    # Tests de utilidades
├── mock-data.ts               # Datos mock para desarrollo
└── utils.ts                   # Utilidades generales (cn para clsx)
```

#### 5. **Configuración Frontend**
- **vite.config.ts**: Configuración de Vite con alias `@` → `src/`
- **tsconfig.json**: Configuración TypeScript
- **eslint.config.js**: Configuración de linting
- **.env.example**: Variables de entorno (ej: `VITE_API_BASE_URL`)

---

## 📡 Flujo de Datos

### Ciclo Request-Response

```
User Browser
    ↓
Frontend (React)
    ├── App.tsx: useEffect → fetchFinancialData()
    ├── fetch GET ${API_BASE_URL}/api/metrics
    └── computeKPIs() + computeMonthlyData()
    ↓
Backend API (FastAPI)
    ├── CORS Middleware (permitir todas las fuentes)
    ├── Router: GET /api/metrics
    ├── Lógica de negocio (routes.py)
    └── Response: FinancialMovement[]
    ↓
Frontend UI Render
    ├── DashboardHeader
    ├── KPIRow (con KPICards)
    ├── IncomeOutcomeChart
    └── ProfitPercentChart
```

### Variables de Entorno Frontend
```
VITE_API_BASE_URL  # URL base del backend (ej: http://localhost:8000)
```

---

## 🐳 Orquestación (Docker Compose)

**Archivo**: [docker-compose.yml](/workspaces/financial-dashboard-context-project-Cristian87-tech/docker-compose.yml)

### Servicios

#### Backend Service
- **Image**: Build desde `./backend/Dockerfile`
- **Puertos**: 
  - 8000 → FastAPI API
  - 5678 → Debugger (posible pdb/debugpy)
- **Volúmenes**: 
  - `./backend:/app` (código compartido)
  - Permite desarrollo con hot reload

#### Frontend Service
- **Image**: Build desde `./frontend/Dockerfile`
- **Puerto**: 5173 → Vite dev server
- **Volúmenes**:
  - `./frontend:/app`
  - `/app/node_modules` (evita sincronización de node_modules)
- **Dependencias**: Espera a que backend esté listo

---

## 📦 Dependencias

### Backend
**Archivo**: [backend/requirements.txt](/workspaces/financial-dashboard-context-project-Cristian87-tech/backend/requirements.txt)
```
fastapi          # Framework REST
pydantic         # Validación de datos
uvicorn          # ASGI server
# + otros (verificar requirements.txt)
```

### Frontend
**Principales** (package.json):
```json
{
  "dependencies": {
    "react": "^19.2.4",
    "react-dom": "^19.2.4",
    "recharts": "^3.8.1",      // Gráficos
    "lucide-react": "^1.8.0",  // Iconos
    "tailwind-merge": "^3.5.0",
    "clsx": "^2.1.1"           // Utilidad de className
  },
  "devDependencies": {
    "typescript": "^5.x",
    "vite": "^5.x",
    "vitest": "^latest",       // Testing
    "eslint": "^9.x"
  }
}
```

---

## 🚀 Cómo Iniciar

### Con Docker Compose
```bash
docker-compose up
# Frontend: http://localhost:5173
# Backend: http://localhost:8000
# API Docs: http://localhost:8000/docs (Swagger)
```

### Desarrollo Local

**Backend**:
```bash
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

**Frontend**:
```bash
cd frontend
npm install
npm run dev
# Abre http://localhost:5173
```

---

## 📊 Estructura de Carpetas Detallada

```
financial-dashboard-context-project-Cristian87-tech/
│
├── backend/
│   ├── Dockerfile              # Build image Python
│   ├── requirements.txt         # Dependencias Python
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py            # ⭐ Entry point FastAPI
│   │   └── routes.py          # Endpoints API + Modelos
│   └── tests/
│       ├── conftest.py        # Configuración pytest
│       └── test_routes.py     # Tests de rutas
│
├── frontend/
│   ├── Dockerfile             # Build image Node.js + Vite
│   ├── package.json           # Scripts + Dependencias
│   ├── package-lock.json
│   ├── index.html             # Punto de entrada HTML
│   ├── vite.config.ts         # Configuración Vite
│   ├── tsconfig.json          # TypeScript config
│   ├── eslint.config.js       # Linting rules
│   ├── .env.example           # Template de env vars
│   ├── .gitignore
│   │
│   ├── src/
│   │   ├── main.tsx           # ⭐ Entry point React
│   │   ├── App.tsx            # ⭐ Componente raíz
│   │   ├── index.css          # Estilos globales + Tailwind
│   │   │
│   │   ├── components/
│   │   │   ├── dashboard/
│   │   │   │   ├── dashboard-header.tsx
│   │   │   │   ├── kpi-row.tsx
│   │   │   │   ├── kpi-card.tsx
│   │   │   │   ├── income-outcome-chart.tsx
│   │   │   │   └── profit-percent-chart.tsx
│   │   │   └── ui/
│   │   │       ├── card.tsx
│   │   │       └── skeleton.tsx
│   │   │
│   │   └── lib/
│   │       ├── financial-types.ts      # Tipos de dominio
│   │       ├── financial-utils.ts      # Lógica de cálculo
│   │       ├── financial-utils.test.ts
│   │       ├── mock-data.ts
│   │       └── utils.ts                # Helpers (cn)
│   │
│   └── public/                # Archivos estáticos
│
├── docker-compose.yml         # Orquestación servicios
├── README.md                  # Documentación en inglés
├── README.es.md               # Documentación en español
├── AGENTS.md                  # Guía para agentes
└── STRUCTURE.md               # Este archivo
```

---

## 🔄 Flujo de Desarrollo

### Frontend → Backend Communication
1. **Fetch Request**: `GET /api/metrics` (sin query params por defecto)
2. **Backend Processing**: Calcula movimientos financieros
3. **JSON Response**: Array de `FinancialMovement`
4. **Frontend Calculation**:
   - `computeKPIs()`: Calcula métricas agregadas
   - `computeMonthlyData()`: Agrupa datos por mes
5. **UI Rendering**: Actualiza componentes con datos

### Error Handling
- Backend: Retorna 500 si hay error
- Frontend: Captura en `.catch()` y muestra mensaje de error
- Mensaje fallback: _"No se pudo cargar la información financiera. Revisa la API de backend."_

---

## 🧪 Testing

### Backend
```bash
cd backend
pytest tests/
```

### Frontend
```bash
cd frontend
npm run test        # Una sola ejecución
npm run test:watch  # Modo watch
npm run test:coverage
```

---

## 🔐 Seguridad

- **CORS**: Permitido para todos los orígenes (`allow_origins=["*"]`)
- **Nota**: En producción, restringir a orígenes específicos

---

## 📝 Resumen Ejecutivo de Servicios

| Servicio | Tecnología | Puerto | Responsabilidad |
|----------|-----------|--------|-----------------|
| **API Backend** | FastAPI + Uvicorn | 8000 | Endpoints REST, datos financieros |
| **Frontend App** | React + Vite | 5173 | UI Dashboard, visualizaciones |
| **Debugger** | debugpy/pdb | 5678 | Debugging backend |

---

## 🎓 Próximos Pasos

1. **Explorar endpoints**: Visita `http://localhost:8000/docs` (Swagger)
2. **Revisar tipos**: Analizar [financial-types.ts](/workspaces/financial-dashboard-context-project-Cristian87-tech/frontend/src/lib/financial-types.ts)
3. **Entender cálculos**: Revisar [financial-utils.ts](/workspaces/financial-dashboard-context-project-Cristian87-tech/frontend/src/lib/financial-utils.ts)
4. **Personalizar UI**: Modificar componentes en [components/dashboard](/workspaces/financial-dashboard-context-project-Cristian87-tech/frontend/src/components/dashboard)

---

**Última actualización**: 2026-11-11
**Proyecto**: Financial Dashboard Context Project
