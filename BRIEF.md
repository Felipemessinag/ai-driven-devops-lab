# Resumen del repositorio

> Generado a partir de una revisión completa del repo (octubre 2026).

## Qué es

Sandbox educativo en español para el **Proyecto Integrador** del programa *AI-Driven DevOps Accelerator*. Se usa como **template de GitHub**: cada participante crea su copia ("Use this template"), la abre en Codespaces y trabaja 8 semanas de labs sobre una app de ejemplo ("Orders API"), documentando su colaboración con IA en una bitácora semanal.

- ~32 archivos trackeados, rama única `main`.
- No requiere cuenta cloud: los labs de Terraform corren contra [LocalStack](https://www.localstack.cloud/).

## Estructura

| Ruta | Contenido |
|---|---|
| `app/` | App de ejemplo "Orders API": `api/` (Flask) + `worker/` (procesa la cola) + `docker-compose.yml` |
| `labs/semana1..8-*/` | Un lab por semana, cada uno con `scenario.md` y sus artefactos (pipeline roto, Terraform, manifiesto K8s, logs, esqueleto de agente) |
| `bitacora/template.md` | Plantilla de bitácora de colaboración con IA que cada estudiante copia y llena |
| `bitacora-template.md` | Copia raíz de la plantilla de bitácora (duplicado exacto) |
| `docs/ARQUITECTURA.md` | Diagrama ASCII de la arquitectura de la app de ejemplo |
| `.devcontainer/` | Devcontainer/Codespaces: Python 3.11 + Docker-in-Docker + Terraform |
| `.github/workflows/ci.yml` | CI base: pytest |

## Stack técnico

- **Python 3.11** — Flask 3.0.3 (API), worker solo stdlib, pytest 8.2.0
- **Contenedores** — imágenes `python:3.11-slim`, docker-compose (api + worker), devcontainer con docker-in-docker
- **IaC** — Terraform (`hashicorp/aws ~> 5.0`) contra LocalStack (bucket S3), lockfile commiteado
- **Kubernetes** — un Deployment deliberadamente inseguro (lab semana 8)
- **CI/CD** — GitHub Actions (pytest); sin Dependabot ni otros pipelines

## Los 8 labs

| Semana | Tema | Reto |
|---|---|---|
| 1 | Setup | Preparar el entorno (sin reto técnico) |
| 2 | Desarrollo y testing | Bug de lógica + tests fallidos |
| 3 | CI/CD | Pipeline de GitHub Actions roto |
| 4 | Cloud / IaC | Terraform con configuración riesgosa (LocalStack) |
| 5 | Agentes | Esqueleto de agente por completar |
| 6 | Observabilidad | Detectar una anomalía en logs |
| 7 | Runbooks / ChatOps | Generar un runbook para un incidente |
| 8 | DevSecOps | Revisar manifiesto K8s y Dockerfile inseguros (intencionales) |

## Dos rutas

- **Ruta A — Reto guiado:** trabajar los labs de `labs/` tal cual.
- **Ruta B — Problema propio:** usar solo `bitacora/template.md` con un caso real del propio trabajo, produciendo las mismas entradas semanales.

## Estado actual

**Sólido:** sin secretos reales ni estado de Terraform commiteado; `.env` ignorado y ausente; lockfile de Terraform commiteado (correcto); las rutas del README coinciden con los labs existentes; árbol de trabajo limpio.

**Pendientes principales** (de la revisión completa):

1. `bitacora-template.md` es un duplicado byte-idéntico de `bitacora/template.md` — riesgo de drift en un repo que a su vez es template.
2. CI mínima: sin `permissions:` (mínimo privilegio), sin caché de pip, actions ancladas por tag mayor (no SHA), sin Dependabot — las dependencias pinned (`flask==3.0.3`, abril 2024) envejecerán en silencio.
3. `labs/semana8-devsecops/Dockerfile-inseguro` no puede ejecutarse: tiene el `pip install` comentado y no hay `requirements.txt` en esa carpeta — confunde un ejercicio que enseña seguridad, no dependencias.
4. Los Dockerfiles base (`app/`) corren como root y usan el servidor Flask de desarrollo — repiten exactamente lo que la semana 8 enseña a corregir, sin señalarlo.
5. Falta LICENSE; `.gitignore` incompleto (`.DS_Store`, `*.tfplan`, `app/worker/queue.json`, `.vscode/`, `.opencode/`…).
6. `app/docker-compose.yml`: clave `version` obsoleta; el flujo API→Worker documentado en `docs/ARQUITECTURA.md` no está cableado (sin volumen compartido ni `depends_on` — el worker siempre imprime "No hay cola de órdenes todavía").

> **Nota:** los artefactos inseguros de `labs/semana8-devsecops/` y `labs/semana4-cloud-iac/` son **intencionales** (cada `scenario.md` los define como parte del ejercicio). El único riesgo operativo real es el falso `sk_live_...` del manifiesto (`labs/semana8-devsecops/manifiesto-inseguro.yaml:24`), que disparará el secret scanning / push protection de GitHub en cada push.
