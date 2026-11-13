---
name: security-configuration
scope:
  - "backend/app/main.py"
  - "backend/Dockerfile"
  - "docker-compose.yml"
  - "frontend/.env.example"
---

# Seguridad y configuración de despliegue

**Nombre:** Seguridad de CORS y configuración de contenedores
**Alcance:** Middleware CORS, imágenes y servicios Docker, variables de entorno y configuración de despliegue.
**Justificación:** `backend/app/main.py` permite actualmente todos los orígenes mediante `allow_origins=["*"]` y habilita credenciales. `backend/Dockerfile` incluye `debugpy`, publica el puerto `5678` y lo enlaza a `0.0.0.0`; son configuraciones que requieren una decisión explícita antes de usarse en producción.

## Guía específica del proyecto

- Configura `CORS_ORIGINS` como una lista separada por comas de orígenes frontend concretos. El valor por defecto es `http://localhost:5173` para desarrollo local; define los orígenes de producción en el entorno de despliegue.
- No agregues `allow_origins=["*"]` como solución genérica. Si el cambio afecta `allow_credentials`, comprueba que la lista de orígenes sea compatible con las necesidades de autenticación del entorno.
- Trata `debugpy` y el puerto `5678` como herramientas exclusivas de desarrollo. No los incluyas ni los publiques en una configuración de producción; separa la configuración de desarrollo si se necesita el debugger.
- No confundas el bind de Uvicorn para que la API sea accesible dentro del contenedor con el bind del debugger: el Dockerfile actual inicia Uvicorn en `0.0.0.0:8000`, mientras que el riesgo es exponer el puerto de depuración `5678`.
- Antes de modificar puertos o servicios, revisa conjuntamente `backend/Dockerfile` y `docker-compose.yml` para conservar la comunicación entre contenedores y el flujo local.
- Si añades una variable de entorno, documenta su nombre y propósito en el ejemplo de configuración correspondiente. Nunca guardes valores secretos en el repositorio.

## Comprobación

Antes de publicar una configuración, revisa:

```bash
grep -nE 'allow_origins|allow_credentials' backend/app/main.py
grep -nE 'debugpy|EXPOSE' backend/Dockerfile
```

Confirma además los puertos publicados y las variables configuradas en `docker-compose.yml` y en la configuración real del entorno de destino.
