---
name: frontend-react
scope:
  - "frontend/src/**/*.ts"
  - "frontend/src/**/*.tsx"
---

# Frontend: React, TypeScript y dashboard

**Nombre:** Frontend React y TypeScript
**Alcance:** `frontend/src/**/*.ts` y `frontend/src/**/*.tsx`, con énfasis en `App.tsx`, `components/` y `lib/`.
**Justificación:** `App.tsx` coordina la carga de `/api/metrics`, los estados de carga/error y la composición del dashboard. `lib/financial-utils.ts` contiene los cálculos, y `lib/financial-types.ts` define los tipos consumidos por la interfaz.

## Guía específica del proyecto

- Mantén en `frontend/src/lib/financial-utils.ts` los cálculos financieros reutilizables, como `computeKPIs()` y `computeMonthlyData()`. Los componentes de `components/dashboard/` deben enfocarse en presentar datos mediante props.
- Conserva los tipos compartidos del frontend en `frontend/src/lib/financial-types.ts`. El backend también define el contrato en `backend/app/routes.py`; cuando cambie la forma de `FinancialMovement`, revisa ambos lados y sus pruebas.
- Deja en `App.tsx` la coordinación de la carga de datos y los estados de carga/error. Comprueba los errores HTTP antes de consumir el JSON y mantén mensajes de error visibles para el usuario.
- Lee la URL de la API desde `VITE_API_BASE_URL`, como en la configuración actual. No fijes una URL de despliegue en el código fuente.
- Usa el alias `@/` para imports entre carpetas de `src` (está configurado en `frontend/tsconfig.app.json`). Para módulos vecinos del mismo directorio, conserva imports relativos como `./financial-types`.
- Sigue los componentes de dashboard existentes: props tipadas, utilidades de Tailwind y tokens visuales del tema en vez de añadir estilos o colores aislados.
- No asumas que todo `useEffect` debe tener dependencias vacías. Declara las dependencias que correspondan a los valores usados; el efecto de carga actual en `App.tsx` usa `[]` porque se ejecuta al montar.

## Comprobación

Desde `frontend/`, ejecuta la prueba enfocada cuando cambies los cálculos:

```bash
npm test -- src/lib/financial-utils.test.ts
```

Para validar cambios de TypeScript o de la interfaz, usa los scripts existentes `npm run build` y `npm run lint`, según corresponda.
