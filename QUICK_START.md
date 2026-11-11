# Financial Dashboard - Resumen Rápido

## 🎯 ¿Qué hace?

Una **aplicación web** que visualiza métricas financieras:
- Genera datos financieros simulados (ingresos/egresos)
- Los agrupa en KPIs (clave) y gráficos mensuales
- Muestra en un dashboard visual bonito

**Datos que maneja**:
- Movimientos: fecha, monto, tipo (ingreso/egreso), categoría, tipo de negocio (B2B/B2C)
- Métricas: totales mensuales, comparativas, alertas de anomalías

---

## 🔌 ¿Cómo se conecta?

```
┌─────────────────────────────────────────────────────────┐
│                    NAVEGADOR (React)                    │
│  http://localhost:5173                                  │
│  ┌──────────────────────────────────────────────────┐   │
│  │  App.tsx → fetch("http://localhost:8000/api/metrics") │
│  └──────────────────────────────────────────────────┘   │
└────────────────────────┬────────────────────────────────┘
                         │ JSON (Array de Movimientos)
                         ↓
┌─────────────────────────────────────────────────────────┐
│              API Backend (FastAPI)                      │
│  http://localhost:8000                                  │
│  ┌──────────────────────────────────────────────────┐   │
│  │  GET /api/metrics → genera datos simulados       │   │
│  │  (otros endpoints: /facets, /summary, /alerts...) │  │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

**Flujo**:
1. Frontend hace `fetch /api/metrics`
2. Backend genera ~360 movimientos ficticios (12 meses × 30 movimientos)
3. Backend retorna array JSON
4. Frontend calcula KPIs y datos mensuales
5. React renderiza gráficos y tarjetas

---

## 🚀 ¿Cómo se ejecuta?

### Opción 1: Docker Compose (Recomendado)
```bash
docker-compose up
```
**Acceso:**
- Frontend: http://localhost:5173
- Backend API: http://localhost:8000
- Docs API: http://localhost:8000/docs

### Opción 2: Local sin Docker

**Backend** (en una terminal):
```bash
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload
# Corre en http://localhost:8000
```

**Frontend** (en otra terminal):
```bash
cd frontend
npm install
npm run dev
# Corre en http://localhost:5173
```

**Nota**: El frontend debe tener `VITE_API_BASE_URL=http://localhost:8000` en `.env` (o usa el `.env.example`)

---

## 📊 Endpoints API Principales

| Endpoint | Descripción |
|----------|-------------|
| `GET /health` | Estado del servidor |
| `GET /api/metrics` | Todos los movimientos (360 ficticios) |
| `GET /api/metrics/facets` | Filtros disponibles (categorías, tipos, fechas) |
| `GET /api/metrics/summary?groupBy=month` | Resumen por período |
| `GET /api/metrics/top-categories` | Top 5 categorías por monto |
| `GET /api/metrics/comparison?period=...` | Comparativa mes a mes |
| `GET /api/metrics/alerts` | Alertas de anomalías |
| `GET /api/metrics/b2b` | Solo movimientos B2B |
| `GET /api/metrics/b2c` | Solo movimientos B2C |

---

## 🎨 Componentes Frontend

```
App.tsx (Raíz)
 ├── DashboardHeader       (Encabezado)
 ├── KPIRow               (4 tarjetas: Ingreso, Egreso, Neto, Tasa)
 │   └── KPICard x4       (Componentes individuales)
 ├── IncomeOutcomeChart   (Gráfico de líneas: Ingresos vs Egresos)
 └── ProfitPercentChart   (Gráfico de líneas: Rentabilidad %)
```

Todos usan **Recharts** para gráficos y **Tailwind CSS** para estilos.

---

## 🧪 Tests

**Backend**:
```bash
cd backend && pytest tests/
```

**Frontend**:
```bash
cd frontend && npm run test        # Una sola vez
cd frontend && npm run test:watch  # Watch mode
```

---

## 📂 Estructura Clave

```
backend/
  └── app/
      ├── main.py          ← Inicia FastAPI
      └── routes.py        ← 20+ endpoints + lógica

frontend/
  └── src/
      ├── main.tsx         ← Inicia React
      ├── App.tsx          ← Componente principal
      ├── components/      ← Dashboard + UI
      └── lib/             ← Tipos, cálculos, datos mock
```

---

## 🔧 Variables de Entorno

**Frontend** ([.env.example](/workspaces/financial-dashboard-context-project-Cristian87-tech/frontend/.env.example)):
```
VITE_API_BASE_URL=http://localhost:8000
```

---

## ✅ Estado Actual

✅ Backend: API REST completamente funcional con 20+ endpoints  
✅ Frontend: Dashboard visual con gráficos interactivos  
✅ Docker: Orquestación lista (docker-compose.yml)  
✅ Testing: Pytest (backend) y Vitest (frontend)  
✅ Documentación: Swagger en http://localhost:8000/docs

---

## 🎓 Próximo Paso

1. **Inicia con**: `docker-compose up`
2. **Abre**: http://localhost:5173
3. **Revisa API**: http://localhost:8000/docs
4. **Lee código**: [STRUCTURE.md](/workspaces/financial-dashboard-context-project-Cristian87-tech/STRUCTURE.md) para detalles

