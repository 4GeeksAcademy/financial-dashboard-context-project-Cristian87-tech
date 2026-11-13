# Reglas operativas del proyecto

Estas reglas orientan cambios y revisiones en el dashboard financiero. Lee primero la regla que corresponda a los archivos que vas a tocar; consulta más de una cuando el cambio cruce ámbitos.

| Archivo | Alcance | Cuándo aplicarlo |
|---|---|---|
| [backend-api.md](./backend-api.md) | `backend/app/**/*.py` | Cambios en modelos, endpoints o lógica de API |
| [frontend-react.md](./frontend-react.md) | `frontend/src/**/*.{ts,tsx}` | Cambios en la aplicación, componentes, tipos o cálculos |
| [testing.md](./testing.md) | `backend/tests/**`, `frontend/src/**/*.test.*` | Cambios de comportamiento o pruebas |
| [security-configuration.md](./security-configuration.md) | CORS, Docker y configuración sensible | Cambios en `backend/app/main.py`, Dockerfiles, `docker-compose.yml` o configuración de despliegue |

## Cómo usar estas reglas

1. Identifica los archivos afectados y aplica las reglas cuyos ámbitos coincidan.
2. Usa las rutas y ejemplos de este repositorio; no copies valores de ejemplo como dominios de producción.
3. Ejecuta las pruebas o comprobaciones relevantes indicadas en la regla.
4. Si el código actual contradice una regla, verifica el comportamiento antes de cambiarlo y documenta cualquier excepción necesaria.

## Relación con los documentos de análisis

`CONVENTIONS.md`, `RISKS.md`, `FINDINGS-BY-CATEGORY.md`, `INDEX-CATEGORIZED.md`, `MASTER-TABLE.md`, `QUICK-GUIDE.md`, `QUICK-START.md` y `RULES.md` conservan notas, hallazgos y propuestas del análisis. No son una lista de requisitos automáticos. Para cambios futuros, usa como guía operativa los cuatro archivos delimitados por ámbito en la tabla anterior; confirma cualquier afirmación contra el código vigente.
