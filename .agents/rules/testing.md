---
name: testing
scope:
  - "backend/tests/**"
  - "frontend/src/**/*.test.*"
---

# Pruebas: validar comportamiento del dashboard

**Nombre:** Pruebas backend y frontend
**Alcance:** `backend/tests/**` y pruebas `*.test.*` bajo `frontend/src/`. También aplica al modificar comportamiento de API o cálculos, aunque el cambio esté en código de producción.
**Justificación:** `backend/tests/test_routes.py` comprueba endpoints, filtros y datos generados; `frontend/src/lib/financial-utils.test.ts` cubre agregaciones, orden cronológico y formato de valores.

## Guía específica del proyecto

- Para el backend, agrega o ajusta pruebas en `backend/tests/test_routes.py` para el comportamiento HTTP y los filtros de `/api/metrics`. Usa `TestClient` y verifica tanto el status como los datos relevantes de la respuesta.
- Para el frontend, coloca las pruebas junto a la funcionalidad bajo `frontend/src/`; sigue el patrón Vitest de `financial-utils.test.ts` para cálculos y formateadores.
- Usa datos explícitos y reproducibles. Si una prueba usa `generate_mock_movements`, pasa un seed, como hacen las pruebas backend actuales.
- Cubre límites propios de este dashboard: fechas inicial/final inclusivas, períodos sin ingresos (porcentaje de beneficio cero), movimientos desordenados y datos que cruzan el año.
- Usa `pytest.mark.parametrize` cuando varios casos ejerciten la misma regla con entradas distintas; no parametrices ni dupliques pruebas solo para alcanzar un número.
- Si cambia el contrato entre el JSON de FastAPI y `FinancialMovement`, prueba la respuesta backend y actualiza las pruebas de los tipos/cálculos frontend afectados. No afirmes que existe una suite de integración backend-frontend: actualmente las suites se ejecutan por separado.
- Ejecuta la prueba más cercana al cambio primero. Recurre a lint o build cuando el cambio afecte tipos, imports o configuración.

## Comandos del proyecto

```bash
# Run from the repository root
(cd backend && pytest tests/test_routes.py)
(cd frontend && npm test -- src/lib/financial-utils.test.ts)
(cd frontend && npm run lint)
(cd frontend && npm run build)
```

Los comandos frontend se ejecutan desde `frontend/`; no es necesario correr todas las pruebas para un cambio aislado si la prueba enfocada cubre el comportamiento modificado.
