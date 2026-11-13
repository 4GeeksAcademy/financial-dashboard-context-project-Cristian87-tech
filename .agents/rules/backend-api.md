---
name: backend-api
scope:
  - "backend/app/**/*.py"
---

# Backend API: contratos y lógica

**Nombre:** Backend API: contratos y lógica
**Alcance:** `backend/app/**/*.py`, especialmente `backend/app/routes.py` y `backend/app/main.py`.
**Justificación:** `routes.py` concentra el `APIRouter`, los modelos Pydantic, los tipos de dominio y las funciones que filtran movimientos. `main.py` crea la aplicación FastAPI y monta ese router.

## Guía específica del proyecto

- Registra los endpoints y sus modelos de dominio en `backend/app/routes.py`; monta el router en `backend/app/main.py` mediante `app.include_router(router)`.
- Define los contratos JSON con modelos de Pydantic y declara `response_model` en los endpoints para que FastAPI valide y documente las respuestas.
- Representa conjuntos cerrados del dominio con los alias `Literal` ya usados: `OperationType`, `Category`, `BusinessType` y `GroupBy`. Usa anotaciones de unión con `|`, siguiendo el estilo existente.
- Mantén los filtros de movimientos en helpers reutilizables. No mutues los datos recibidos; conserva el comportamiento inclusivo actual de `start_date` y `end_date`.
- Si un endpoint recibe ambas fechas, rechaza explícitamente un rango invertido antes de procesar los datos y devuelve un error HTTP apropiado.
- Los datos simulados se generan mediante `generate_mock_movements(seed=...)`. En pruebas, pasa un seed explícito para que los resultados sean reproducibles.
- Cuando cambies un campo o filtro de la API, actualiza el modelo/respuesta afectado y las pruebas de `backend/tests/test_routes.py`.

## Comprobación

Desde `backend/`, ejecuta la prueba enfocada:

```bash
pytest tests/test_routes.py
```

Consulta `backend/app/routes.py` y `backend/app/main.py` antes de asumir que una ruta o un contrato sigue igual.
