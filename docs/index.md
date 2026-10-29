---
title: Inicio
nav_order: 1
permalink: /
---

# Laboratorio SDLC Seguro
{: .no_toc }

**Empiecen aquí: [Lab de misiones](https://ic8071-sdlc-2026s2.github.io/sdlc-guia/lab/).** El lab los guía paso a paso y verifica su repositorio. Esta guía tiene el detalle de cada fase.
{: .importante }

IC-8071 Seguridad del Software · Instituto Tecnológico de Costa Rica
{: .fs-5 .fw-300 }

En este laboratorio su equipo recibe una aplicación web intencionalmente vulnerable y la lleva por el ciclo de vida de desarrollo seguro completo: casos de abuso, requerimientos de seguridad, análisis de riesgo arquitectónico (ARA), análisis estático (SAST), análisis de composición (SCA), búsqueda de secretos, análisis dinámico (DAST), pruebas escritas desde los casos de abuso y un release firmado con SBOM y attestations. Cada fase sigue uno de los touchpoints de McGraw.

{: .importante }
La lección central: **ninguna técnica detecta todas las debilidades**. Las fallas de diseño aparecen en el ARA y en las pruebas escritas desde los casos de abuso; las de código, en SAST; las de dependencias, en SCA; las de configuración, en DAST. Al final usted debe poder mostrar, debilidad por debilidad, qué técnica la vio y cuál no.

<details markdown="block">
<summary>Contenido de esta página</summary>
1. TOC
{:toc}
</details>

## Calendario

Son 5 sesiones de 90 minutos. Las fechas las fija el enunciado del curso. Cada sesión empieza con 5 minutos en los que el profesor muestra la fase anterior resuelta sobre su app de demostración, **Damn Vulnerable GraphQL Application** (DVGA), que no le toca a ningún equipo.

| Sesión | Fases | Qué hace el equipo | Guía |
|:---|:---|:---|:---|
| 1 | Clase 24 y fase 0 | Clase de DevOps y DevSecOps, actividad «Diseña tu pipeline». Sorteo de apps, roles, repositorio del equipo, Codespace y primer pipeline en verde | [Fase 0](fase-0/) |
| 2 | Fases 1 y 2 | Casos de abuso con Cornucopia, requerimientos ASVS 5.0, DFD con fronteras de confianza y matriz de riesgos 3 por 3 | [Fase 1](fase-1/), [Fase 2](fase-2/) |
| 3 | Fase 3 | Semgrep con regla propia, Gitleaks, Dependabot, Dependency Review, Scorecard y endurecimiento del pipeline | [Fase 3](fase-3/) |
| 4 | Fase 4 | ZAP contra la app levantada en el runner y pruebas pytest desde los casos de abuso | [Fase 4](fase-4/) |
| 5 | Fase 5 | Correcciones, imagen firmada, SBOM, attestations y gate (35 min); presentación (30 min); intercambio rojo y azul (20 min); cierre (5 min) | [Fase 5](fase-5/) |

El ritmo es libre dentro de cada sesión. Un equipo que no termina una fase en clase puede completarla hasta el cierre del curso: cada fase se califica con el estado del repositorio en esa fecha.

## Las apps

Las cuatro son apps vulnerables mantenidas por la comunidad, con licencia MIT. El sorteo de la fase 0 decide cuál recibe su equipo. El código de `app/` llega sin cambios, fijado por versión o commit.

| App | Stack | Repositorio plantilla | Modo de ZAP | Lenguaje de la regla Semgrep | Dependencias que vigila Dependabot |
|:---|:---|:---|:---|:---|:---|
| OWASP Juice Shop v20.3.0 | Node.js, TypeScript, Express y Angular; SQLite | `sdlc-juice-shop` | baseline | TypeScript (`.ts`) | npm en `/app` y `/app/frontend` |
| OWASP PyGoat v3.0.0 | Python 3.11 y Django 4.2; SQLite | `sdlc-pygoat` | baseline | Python (`.py`) | pip en `/app` |
| VAmPI, commit f16052d | Python 3.11, Flask y Connexion; SQLite; solo API | `sdlc-vampi` | api (OpenAPI) | Python (`.py`) | pip en `/app` |
| OWASP RailsGoat, commit f5951f1 | Ruby 3.4 y Rails; SQLite | `sdlc-railsgoat` | baseline | Ruby (`.rb`) | bundler en `/app` |

{: .nota }
Las cuatro apps publican sus vulnerabilidades: Juice Shop tiene guía oficial de soluciones, VAmPI las lista en su README, PyGoat trae la carpeta `Solutions` y RailsGoat tiene una wiki. Puede consultarlas. La nota no premia descubrir: premia **trazar, automatizar y justificar**. Si usa una fuente pública, cítela en el issue correspondiente.

## Roles

Los roles rotan por sesión y nadie repite. La rotación se anota en `EQUIPO.md` en la fase 0. Quien tiene el rol lidera esa fase y responde por el resultado, pero todo el equipo trabaja en ella.

| Rol | Responde por |
|:---|:---|
| Arquitectura | DFD, fronteras de confianza, matriz de riesgos, decisión de qué se corrige y qué se acepta, línea base del gate |
| AppSec | Casos de abuso, requerimientos ASVS, regla propia de Semgrep, revisión y trazabilidad de cada hallazgo |
| DevSecOps | Workflows: permisos mínimos, actions fijadas, gates, firma y attestations |
| Equipo rojo | Ataca la app levantada, escribe las pruebas de abuso y conduce la predicción antes de cada herramienta |
| Relatoría | Solo en el equipo de 5: bitácora de decisiones de la sesión y presentación final |

## Cómo se avanza

1. Cada fase llega como un issue con las tareas y el checkpoint. El de la fase 0 lo abre el workflow **Checkpoints** con el primer push.
2. El workflow **Checkpoints** corre en cada push a `main`, en cada cambio de issues, al terminar los demás workflows y una vez al día. Revisa las condiciones de la fase, pone etiquetas a los issues según sus formularios y publica el reporte del equipo en GitHub Pages.
3. Cuando todas las condiciones de una fase se cumplen, cierra su issue y abre el de la fase siguiente.
4. El [tablero de avance](tablero/) muestra la fase actual de cada equipo, el porcentaje por fase y las condiciones pendientes. Es el que se proyecta en clase.

Hay 29 condiciones automáticas en total, con identificadores `f0.*` a `f5.*`. Cada página de fase explica cómo se cumple cada una.

## Cómo se evalúa

| Componente | Peso | Quién lo mide |
|:---|:---|:---|
| Checkpoints automáticos | 35 % | Verificador central, con su propia copia de `checkpoints.py` |
| Trazabilidad y respuestas | 35 % | Revisión docente de issues, matriz, PR, predicciones y presentación |
| Reflexión individual | 30 % | Revisión docente de `seguridad/reflexiones/<usuario>.md`, con commit propio |
| [Bonus de IA](bonus-ia/) | hasta 10 % extra | Voluntario; la nota total tiene tope de 100 % |

- **Regla acumulativa.** Cada fase se califica con el estado del repositorio en la fecha de cierre del curso. Al cierre de cada sesión el profesor guarda una foto del avance y pone la etiqueta `sesion-N` en `main`; eso queda como registro para el tablero, no como nota.
- **Commit propio.** La reflexión individual solo cuenta si el commit que la sube sale de la cuenta de GitHub de quien la escribe.
- **No toque la infraestructura.** No modifique `lab/checkpoints.py` ni `.github/workflows/checkpoints.yml`. La nota de checkpoints la calcula el verificador central, no la copia de su repositorio.

## Reglas éticas

{: .advertencia }
Las apps del laboratorio son vulnerables a propósito. Las técnicas que practica aquí son delito si se usan contra sistemas ajenos sin autorización; en Costa Rica, el Código Penal tipifica los delitos informáticos en la sección que reformó la Ley 9048 de 2012.

- Ataque **solo la app de su equipo**, dentro de su Codespace o en el runner de GitHub Actions.
- Mantenga **privado** el puerto de la app en el Codespace. No lo cambie a público y no despliegue la app en ningún servidor.
- No escanee ni ataque la red del TEC, los Codespaces de otros equipos ni servicios de terceros. Las herramientas apuntan siempre a `localhost`.
- En el intercambio rojo y azul de la sesión 5 se revisa **código**, no se ataca la app de otro equipo.
- No suba secretos reales al repositorio. Los secretos que trae la app son parte del ejercicio.
- Puede consultar soluciones públicas de las apps. Cítelas. No copie artefactos de otro equipo del curso.

## Enlaces

| Página | Para qué |
|:---|:---|
| [Herramientas](herramientas/) | Una sección corta por herramienta, con su enlace oficial |
| [Preguntas frecuentes](preguntas-frecuentes/) | Problemas comunes de Codespaces, checkpoints y workflows |
| [Bonus de IA](bonus-ia/) | Un agente con sus requerimientos como guardrails y un PR que pase el pipeline |
| [Tablero](tablero/) | Avance de los equipos, para proyectar |
