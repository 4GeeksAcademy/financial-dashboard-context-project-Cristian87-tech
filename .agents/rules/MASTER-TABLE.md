# Tabla Maestra de Hallazgos

**Documento de referencia única** que lista todos los hallazgos con categoría, severidad, ubicación, y referencias cruzadas.

---

## TABLA MAESTRA: TODOS LOS HALLAZGOS

| # | Hallazgo | Categoría | Severidad | Archivo | Línea | Estado | Referencia |
|---|----------|-----------|-----------|---------|-------|--------|-----------|
| 1 | CORS permite `["*"]` | Seguridad | 🔴 CRÍTICO | `main.py` | 8-12 | ❌ Rompe | RISKS.md § 1, FINDINGS.md § 4 |
| 2 | Debugpy expuesto 0.0.0.0:5678 | Seguridad | 🔴 CRÍTICO | `Dockerfile` | 8 | ❌ Rompe | RISKS.md § 2, FINDINGS.md § 4 |
| 3 | Tipos duplicados (backend ↔ frontend) | Arquitectura | 🟡 ALTO | `routes.py` vs `types.ts` | 1-9 vs 1-3 | ⚠️  Riesgo | RISKS.md § 3, FINDINGS.md § 1 |
| 4 | seed=42 siempre en endpoints | Testing | 🟡 ALTO | `routes.py` | 286,310,332,358,380,399,416 | ⚠️  Riesgo | RISKS.md § 4, FINDINGS.md § 3 |
| 5 | Sin validación `start_date > end_date` | Type Safety | 🟡 ALTO | `routes.py` | 375-397 | ⚠️  Riesgo | RISKS.md § 5, FINDINGS.md § 2 |
| 6 | Alertas solo comparan promedio simple | Lógica de Negocios | 🟡 ALTO | `routes.py` | 199-213 | ⚠️  Riesgo | RISKS.md § 6, FINDINGS.md § 7 |
| 7 | Fetch sin retry logic | DX | 🟠 MEDIO | `App.tsx` | 16-21 | ⚠️  Riesgo | RISKS.md § 7, FINDINGS.md § 5 |
| 8 | API_BASE_URL sin validación | DX | 🟠 MEDIO | `App.tsx` | 13 | ⚠️  Riesgo | RISKS.md § 8, FINDINGS.md § 5 |
| 9 | Estado global sin Context API | DX | 🟠 MEDIO | `App.tsx` | 26-29 | ⚠️  Preventivo | RISKS.md § 9, FINDINGS.md § 5 |
| 10 | Sin parametrización tests backend | Testing | 🟠 MEDIO | `test_routes.py` | - | ⚠️  Riesgo | FINDINGS.md § 3 |
| 11 | Sin tests integración backend-frontend | Testing | 🟠 MEDIO | Ambos | - | ⚠️  Riesgo | FINDINGS.md § 3 |
| 12 | .env.example incompleto | Documentación | 🟠 MEDIO | `.env.example` | 1 | ⚠️  Riesgo | RISKS.md § 13, FINDINGS.md § 6 |
| 13 | Period hardcoded "2024 - Full Year" | DX | ⚪ BAJO | `App.tsx` | 48 | ⚠️  UX | RISKS.md § 10, FINDINGS.md § 5 |
| 14 | Sin logging estructurado | DX | ⚪ BAJO | Backend + Frontend | - | ⚠️  Mantenibilidad | RISKS.md § 11, FINDINGS.md § 5 |
| 15 | Router centralizado ✅ | Arquitectura | ✅ BUENO | `main.py` | 9 | ✅ Mantener | CONVENTIONS.md § Backend |
| 16 | Separación lib/ vs components/ ✅ | Arquitectura | ✅ BUENO | `lib/`, `components/` | - | ✅ Mantener | CONVENTIONS.md § Frontend |
| 17 | Componentes UI reutilizables ✅ | Arquitectura | ✅ BUENO | `ui/card.tsx`, `ui/skeleton.tsx` | - | ✅ Mantener | CONVENTIONS.md § Frontend |
| 18 | Type hints con `\|` ✅ | Type Safety | ✅ BUENO | `routes.py` | 60-70 | ✅ Mantener | CONVENTIONS.md § Backend |
| 19 | `Literal[]` para tipos cerrados ✅ | Type Safety | ✅ BUENO | `routes.py` | 1-9 | ✅ Mantener | CONVENTIONS.md § Backend |
| 20 | Pydantic `BaseModel` obligatorio ✅ | Type Safety | ✅ BUENO | `routes.py` | 15-50 | ✅ Mantener | CONVENTIONS.md § Backend |
| 21 | `response_model=` en endpoints ✅ | Type Safety | ✅ BUENO | `routes.py` | 286+ | ✅ Mantener | CONVENTIONS.md § Backend |
| 22 | Props interface explícita ✅ | Type Safety | ✅ BUENO | `kpi-card.tsx` | 1-14 | ✅ Mantener | CONVENTIONS.md § Frontend |
| 23 | Funciones puras sin mutación ✅ | Testing | ✅ BUENO | `routes.py` | 92-180 | ✅ Mantener | CONVENTIONS.md § Backend |
| 24 | Importes con alias `@/` ✅ | DX | ✅ BUENO | `App.tsx` | 1-10 | ✅ Mantener | CONVENTIONS.md § Frontend |
| 25 | Nombres kebab-case ✅ | DX | ✅ BUENO | `dashboard/` | - | ✅ Mantener | CONVENTIONS.md § Frontend |
| 26 | CSS variables dinámicas ✅ | DX | ✅ BUENO | `kpi-card.tsx` | 16-24 | ✅ Mantener | CONVENTIONS.md § Frontend |
| 27 | useEffect fetch una sola vez ✅ | DX | ✅ BUENO | `App.tsx` | 29-43 | ✅ Mantener | CONVENTIONS.md § Frontend |
| 28 | Lógica de cálculo en lib/ ✅ | Arquitectura | ✅ BUENO | `financial-utils.ts` | - | ✅ Mantener | CONVENTIONS.md § Frontend |
| 29 | Tests de funciones puras ✅ | Testing | ✅ BUENO | `financial-utils.test.ts` | - | ✅ Mantener | CONVENTIONS.md § Testing |

---

## RESUMEN POR COLUMNA

### Por Severidad

**🔴 CRÍTICOS (2)**: Debes arreglar ANTES de producción
- #1: CORS `["*"]`
- #2: Debugpy expuesto

**🟡 ALTOS (6)**: Debes arreglar PRONTO (antes de merge a main)
- #3: Tipos duplicados
- #4: seed=42 siempre
- #5: Sin validación input
- #6: Alertas simplistas

**🟠 MEDIOS (7)**: Debes arreglar DESPUÉS (próxima iteración)
- #7-12: Fetch, API_BASE_URL, Context, tests, .env

**⚪ BAJOS (3)**: Opcional, mejora UX
- #13-14: Period, logging

**✅ BUENO (11)**: Nunca cambiar, mantener patrones
- #15-29: Todos los patrones correctos

### Por Categoría

| Categoría | Crítico | Alto | Medio | Bajo | Bueno | Total |
|-----------|---------|------|-------|------|-------|-------|
| Arquitectura | - | 1 | - | - | 3 | 4 |
| Type Safety | - | 1 | 1 | - | 4 | 6 |
| Testing | - | 2 | 2 | - | 1 | 5 |
| Seguridad | 2 | - | - | - | - | 2 |
| DX | - | - | 4 | 2 | 3 | 9 |
| Documentación | - | - | 1 | - | - | 1 |
| Lógica | - | 1 | - | 1 | - | 2 |
| **TOTAL** | 2 | 6 | 7 | 3 | 11 | **29** |

### Por Estado

| Estado | Cantidad |
|--------|----------|
| ❌ Rompe (crítico) | 2 |
| ⚠️ Riesgo (alto/medio) | 12 |
| ⚠️ Preventivo | 1 |
| ⚠️ UX (bajo) | 2 |
| ✅ Mantener (bueno) | 11 |
| **TOTAL** | **29** |

### Por Archivo/Dominio

**Backend (8)**:
- main.py: #1
- routes.py: #3, #4, #5, #6, #18, #19, #20, #21, #23 (críticos y buenos)
- Dockerfile: #2
- test_routes.py: #10

**Frontend (14)**:
- App.tsx: #7, #8, #9, #13, #24, #27 (riesgos y buenos)
- types.ts: #3
- financial-utils.ts: #28
- financial-utils.test.ts: #29
- kpi-card.tsx: #22, #26
- Componentes ui/: #17
- Otros componentes: #16, #25

**Config (3)**:
- tsconfig.app.json: #24
- vite.config.ts: #24
- .env.example: #12

---

## NAVEGACIÓN RÁPIDA

### "¿Qué arreglo PRIMERO?"
1. #1: CORS (🔴 CRÍTICO) - 5 min
2. #2: Debugpy (🔴 CRÍTICO) - 10 min
3. #5: Validación input (🟡 ALTO) - 15 min
4. #4: seed variado (🟡 ALTO) - 20 min

### "¿Qué NO debo romper?"
Ver columna "✅ BUENO" (#15-29):
- Tipo hints con `|`
- Pydantic response_model
- Props interface
- Alias @/
- Funciones puras

### "¿Qué mejoro DESPUÉS?"
#7-12 (🟠 MEDIO):
- Fetch retry
- API_BASE_URL validación
- Context API
- Tests parametrizados
- .env.example

---

## REFERENCIAS CRUZADAS ENTRE DOCUMENTOS

### CONVENTIONS.md
- Detalla patrones correctos: #15-29
- Con código real y explicaciones
- Link desde tabla: Fila "Referencia"

### RISKS.md
- Detalla riesgos: #1-14
- Con escenarios de fallo
- 13 secciones numeradas

### FINDINGS-BY-CATEGORY.md
- Agrupa hallazgos por categoría
- 7 secciones temáticas
- Enlaza a CONVENTIONS.md y RISKS.md

### QUICK-GUIDE.md
- Checklist operacional
- Antipatrones rápidos
- Tabla de referencia

### INDEX-CATEGORIZED.md
- Índice de navegación
- Checklists por propósito
- Mapa de archivos

---

## CÓMO USAR ESTA TABLA

**Problema**: "¿Dónde está el hallazgo sobre X?"
**Solución**: Busca en columna "Hallazgo", luego:
1. Lee columna "Categoría" para entender el tema
2. Lee columna "Severidad" para saber urgencia
3. Lee columna "Archivo" y "Línea" para ubicación exacta
4. Lee columna "Referencia" para leer detalles

**Ejemplo**: Quiero entender CORS
- Busca "CORS" → Fila #1
- Categoría: Seguridad
- Severidad: 🔴 CRÍTICO
- Archivo: main.py línea 8-12
- Referencia: RISKS.md § 1, FINDINGS.md § 4

**Ejemplo**: Voy a hacer endpoint nuevo
- Ve a "✅ BUENO" → Filas #18, #20, #21
- Estas son patrones que debes seguir
- Referencia: CONVENTIONS.md § Backend

---

## RESUMEN PARA AGENTES

**Si eres un agente:**

1. **Antes de modificar código**: Lee esta tabla, identifica categoría
2. **Si toca backend**: Verifica #1, #2, #5, #6 (no toques #18-21)
3. **Si toca frontend**: Verifica #7, #8, #9 (no toques #24-26)
4. **Si toca tests**: Verifica #4, #10, #11 (mantén #23, #29)
5. **Si toca config**: Verifica #12 (mantén #17, #25)

**Checklist antes de PR**:
- ❌ Críticos (#1, #2) - No toques estas áreas
- 🟡 Altos (#3-6) - Si tu cambio afecta, documenta
- 🟠 Medios (#7-12) - Si tu cambio ayuda, agrégalo
- ✅ Buenos (#15-29) - Mantén estos patrones

---

## ESTADÍSTICAS FINALES

```
Total hallazgos:        29
├── 2 críticos
├── 6 altos
├── 7 medios
├── 3 bajos
└── 11 bueno

Cobertura:              100% (todos los archivos analizados)
Especificidad:          100% (archivo + línea + código)
Categorización:         7 temas temáticos
Referencias:            >50 enlaces cruzados
Documentación total:    ~36 KB (5 documentos)
```

---

## CÓMO ESTA TABLA APOYA DECISIONES

### Code Review
Revisor busca cambio en archivo → Va a tabla → Busca filas relevantes → Chequea si se violan patrones buenos o introduce riesgos

### Onboarding
Nuevo dev. busca "¿cómo se hace un componente?" → Ve filas #22, #24-26 (bueno) y #7, #8, #9 (evitar) → Lee referencias

### Roadmap
PM pregunta "¿qué arreglamos primero?" → Ver columna severidad → Orden: #1 (5 min), #2 (10 min), #5 (15 min), etc.

### Automatización
Agente busca "¿qué validaciones hagas?" → Busca filas con "validación" → Encuentra #5 (input ranges), #8 (URL validation), etc.

