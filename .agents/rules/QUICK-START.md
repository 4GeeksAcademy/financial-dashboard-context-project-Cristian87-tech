# ⚡ QUICK START: REGLAS OPERACIONALES

**¿Qué es esto?** 22 reglas que todo código en este proyecto debe cumplir.

**¿Dónde está el documento completo?** [RULES.md](./RULES.md)

---

## 🚨 ANTES DE HACER PR: CHECKLISTS

### CHECKLIST SEGURIDAD (2 min)
```bash
# REGLA #1: Whitelist CORS
grep -n 'allow_origins' backend/app/main.py
# Esperado: NO ["*"], sino lista específica

# REGLA #2: Debugpy 127.0.0.1
grep -n 'debugpy' backend/Dockerfile
# Esperado: NADA en Dockerfile (solo en Dockerfile.dev)
```

**Si algo falla:** FIX AHORA antes de hacer PR.

---

### CHECKLIST ARQUITECTURA (5 min)
```bash
# REGLA #3-5: Type hints
grep ' | ' backend/app/routes.py        # Usa | para uniones
grep 'Literal\[' backend/app/routes.py  # Tipos cerrados

# REGLA #4: Pydantic
grep 'response_model=' backend/app/routes.py

# REGLA #6: Funciones puras
grep 'def calculate_' backend/app/routes.py

# REGLA #7: Lógica en lib/
ls frontend/src/lib/

# REGLA #8: Importes @/
grep -A 2 '"@/\*"' frontend/tsconfig.app.json

# REGLA #9: Props interface
grep 'interface Props' frontend/src/components/**/*.tsx

# REGLA #10: kebab-case archivos
ls frontend/src/components/dashboard/ | grep -E '\.tsx$'
# Esperado: kpi-card.tsx, portfolio-summary.tsx (no UpperCase.tsx)
```

---

### CHECKLIST DE TESTING (3 min)
```bash
# REGLA #16: Tests parametrizados
grep '@pytest.mark.parametrize' backend/tests/test_routes.py

# REGLA #17: Tests de integración
ls backend/tests/test_integration.py 2>/dev/null && echo "✓" || echo "FALTA"

# REGLA #22: useEffect con []
grep -A 2 'useEffect' frontend/src/App.tsx | grep '\[\]'
```

---

## 📋 GUÍA POR PROPÓSITO

### Voy a hacer una PR
1. Ejecuta **CHECKLIST SEGURIDAD** arriba
2. Ejecuta **CHECKLIST ARQUITECTURA** para tu código
3. Ejecuta **CHECKLIST DE TESTING** si tocas tests
4. Commit y PR

### Voy a revisar una PR
1. Busca violaciones de reglas 🔴 (críticas) y 🟡 (arquitectura)
2. Abre [RULES.md](./RULES.md) y busca por nombre del archivo
3. Comenta si falta verificación (ejemplo: "Falta response_model, REGLA #4")

### Soy nuevo en el proyecto
1. Lee [RULES.md](./RULES.md) completo
2. Enfócate en reglas 🟡 (arquitectura) y ✅ (obligatorias)
3. Pregunta si algo no está claro

### Necesito un checklist específico
- **Seguridad:** Reglas #1-2
- **Type Safety:** Reglas #3-5, #9
- **Testing:** Reglas #12, #16-17
- **DX:** Reglas #8, #13-15
- **Mantenibilidad:** Reglas #6-7, #10, #20-22

---

## 🎯 ESTADO ACTUAL DEL PROYECTO

| Categoría | Reglas | Estado | Acción |
|-----------|--------|--------|--------|
| 🔴 Críticas | #1-2 | ❌ FALLAN | Fix HOY (15 min) |
| 🟡 Arquitectura | #3-10 | ✅ PASAN | Mantener |
| 🟠 Validación | #11-18 | ❌ FALTAN | Próximas 2 semanas |
| ✅ Obligatorias | #19-22 | ✅ PASAN | Nunca cambiar |

**Cumplimiento:** 12/22 (55%)

---

## 🔍 BÚSQUEDA RÁPIDA: ARCHIVO → REGLA

```
backend/app/main.py        → #1 (CORS), #20 (Router)
backend/app/routes.py      → #3-6 (type hints, Pydantic), #11-12, #18
backend/Dockerfile         → #2 (Debugpy)
backend/tests/test_routes.py → #16-17 (parametrize, integration)
frontend/src/App.tsx       → #7, #8, #13-14, #22
frontend/src/components/   → #9-10, #21 (CSS vars)
frontend/tsconfig.app.json → #8 (@/ alias)
frontend/.env.example      → #15
```

---

## 💡 EJEMPLOS

### ❌ NO: Union antigua
```python
from typing import Union
def foo(x: Union[int, None]): ...
```

### ✅ SÍ: Union moderna
```python
def foo(x: int | None): ...
```

---

### ❌ NO: CORS abierto
```python
allow_origins=["*"]
```

### ✅ SÍ: CORS restringido
```python
allow_origins=["http://localhost:5173", "https://yourdomain.com"]
```

---

### ❌ NO: Lógica en React
```tsx
export function Dashboard() {
  const [data, setData] = useState();
  useEffect(() => {
    const profit = data.reduce((sum, d) => sum + d.profit, 0);  // ← Lógica aquí
    setData(profit);
  }, []);
  return <div>{data}</div>;
}
```

### ✅ SÍ: Lógica en lib/
```tsx
// lib/financial-utils.ts
export function calculateProfit(data) {
  return data.reduce((sum, d) => sum + d.profit, 0);
}

// components/dashboard.tsx
export function Dashboard() {
  const [data, setData] = useState();
  useEffect(() => {
    const profit = calculateProfit(data);  // ← Lógica delegada
    setProfit(profit);
  }, []);
  return <div>{profit}</div>;
}
```

---

### ❌ NO: Importes relativos
```tsx
import { KPI } from "../../../components/kpi";
```

### ✅ SÍ: Importes con @/
```tsx
import { KPI } from "@/components/kpi";
```

---

## 📞 ¿DUDAS?

- Lee la regla completa en [RULES.md](./RULES.md)
- Copia el comando bash de verificación
- Busca ejemplos de código en el repo
- Pregunta al equipo

---

**Última actualización:** 2024-11-13  
**Documentación completa:** [RULES.md](./RULES.md)
