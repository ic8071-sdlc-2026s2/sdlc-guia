---
title: Bonus de IA
nav_order: 9
---

# Bonus de IA
{: .no_toc }

Voluntario e individual. Hasta 10 % extra sobre la nota del laboratorio, con tope de 100 %. Plazo: el cierre del curso.
{: .fs-5 .fw-300 }

Un agente de IA que escribe código es un desarrollador más del equipo, con una diferencia: no leyó sus casos de abuso. En este bonus usted le da los requerimientos de seguridad de su equipo como **guardrails**, le encarga una tarea y entrega un pull request hecho por el agente que **pase el pipeline** sin debilitarlo. Lo que se evalúa no es que el agente programe bien, sino que usted lo haya acotado bien y pueda mostrar si respetó los requerimientos.

<details markdown="block">
<summary>Contenido de esta página</summary>
1. TOC
{:toc}
</details>

## Reglas

- **Agente libre**, siempre que sea gratuito para usted. Abajo hay un ejemplo paso a paso con GitHub Copilot.
- **Guardrails en el repositorio.** Los requerimientos del equipo van en un archivo de instrucciones versionado, no solo en el chat.
- **Una tarea trazada.** El issue que recibe el agente cita un caso de abuso y un requerimiento del equipo: corregir una debilidad, escribir una prueba de abuso o una regla de Semgrep.
- **El PR pasa el pipeline** que exista en ese momento: como mínimo «Fase 3 · Seguridad en PR» en verde. No vale desactivar un job, marcar una prueba como `skip` ni agregar excepciones a la línea base para que pase.
- **Revisión humana.** Usted revisa el diff completo antes de pedir la aprobación, y otra persona del equipo lo aprueba: el ruleset lo exige.
- **Sin secretos.** No le dé tokens, contraseñas ni acceso a otros repositorios.

## Qué entrega

Un archivo `seguridad/bonus-ia/<su-usuario>.md`, subido desde su propia cuenta, con:

| Sección | Contenido |
|:---|:---|
| Agente | Cuál, con qué plan y en qué modo (agente en la nube, agente en el editor, CLI) |
| Guardrails | Enlace al commit del archivo de instrucciones y qué requerimientos incluyó |
| Tarea | Enlace al issue que recibió el agente, con su caso de abuso y su requerimiento |
| PR | Enlace al PR y a la corrida en verde del pipeline |
| Bitácora | Enlace a la sesión del agente o la conversación exportada |
| Análisis | 200 a 400 palabras: qué requerimientos respetó el agente, cuál intentó violar o ignoró, qué tuvo que corregirle y qué cambiaría en las instrucciones |

## Ejemplo paso a paso con GitHub Copilot

{: .advertencia }
Los planes y nombres de Copilot cambian seguido. Lo de esta sección se revisó en octubre de 2026 en la documentación de GitHub; el profesor confirma al inicio del curso los puntos marcados como **por confirmar**.

### 1. Obtener Copilot

Los estudiantes verificados en [GitHub Education](https://github.com/education) reciben Copilot Pro sin costo. Active el beneficio y revise en **Settings → Copilot** que su plan esté activo. **Por confirmar:** el nombre y alcance del plan estudiantil vigente.

Copilot ofrece dos formas de trabajar como agente:

| Modo | Dónde corre | Quién hace los commits | Plan |
|:---|:---|:---|:---|
| Agente en la nube (Copilot cloud agent, antes «coding agent») | Un entorno de GitHub Actions; usted le asigna un issue y abre un PR | Copilot | Pro, Pro+, Business o Enterprise |
| Modo agente en el editor | Su Codespace, en el chat de VS Code | Usted, con los cambios que propone el agente | Disponible también con límites más bajos en el plan gratuito |

**Por confirmar:** si el agente en la nube está habilitado en los repositorios de la organización del curso para cuentas con Copilot Pro personal. Si no lo está, use el modo agente en el Codespace.

### 2. Escribir los guardrails

Cree `.github/copilot-instructions.md` en una rama y entre por PR, como todo cambio desde la fase 3. Copilot lo lee en el chat, en el agente del editor y en el agente en la nube. Ejemplo para un equipo de VAmPI; reemplace los requerimientos por los suyos:

```markdown
# Instrucciones para agentes en este repositorio

Este repositorio es un laboratorio de seguridad. La app de app/ es VAmPI, una API
Flask vulnerable a propósito. Su trabajo es corregir debilidades sin crear otras.

## Requerimientos de seguridad del equipo (no negociables)

- #12 · ASVS 8.2.2: cada endpoint que recibe un username o un id verifica que el
  recurso pertenezca al usuario del token, salvo que el usuario sea admin.
- #13 · ASVS 1.2.4: toda consulta a la base usa parámetros o el ORM; nunca SQL
  armado con f-strings, % o concatenación.
- #15 · ASVS 9.1.1: los JWT se verifican con firma y algoritmo fijo; nunca
  verify_signature=False ni el algoritmo none.
- #17 · ASVS 6.3.1: el login limita los intentos fallidos por usuario.

## Reglas del repositorio

- No modifique lab/checkpoints.py, .github/workflows/checkpoints.yml ni lab/app.sh.
- No desactive ni debilite workflows, jobs, pruebas o gates. No agregue
  excepciones a seguridad/fase-5/.trivyignore.yaml.
- Corregir no es cambiar vulnerable=1 a 0: es cambiar el código del endpoint.
- Toda action nueva va fijada por hash de 40 caracteres con la versión en un
  comentario, y todo workflow declara permissions en el nivel superior.
- Cada cambio de comportamiento lleva una prueba en seguridad/fase-4/pruebas/
  marcada con @pytest.mark.caso_abuso(N), que afirma el comportamiento seguro.
- Antes de terminar, corra: semgrep --test seguridad/fase-3/semgrep
```

Escriba requerimientos que se puedan comprobar en el diff. «Escriba código seguro» no es un guardrail.

### 3. Escribir el issue de la tarea

Un issue normal (no un formulario) con la tarea acotada:

```markdown
Corregir el caso de abuso #4 (un usuario lee el secreto del libro de otra persona con
GET /books/v1/{book_title}) según el requerimiento #12.

Criterios de aceptación:
Dado un usuario autenticado como name1 y el título de un libro que pertenece a name2
Cuando pide GET /books/v1/<ese título> con su propio token
Entonces la API responde 404 y no entrega el secreto del libro

- Agregue @pytest.mark.corregida a la prueba existente del caso #4.
- No cambie otros endpoints.
```

### 4a. Asignarlo al agente en la nube

1. En el issue, en la barra derecha, **Assignees → Copilot**. Puede agregar contexto en **Optional prompt** y elegir la rama base.
2. Copilot reacciona al issue con un emoji de ojos y abre un PR en borrador vinculado al issue. Solo ve el título, la descripción y los comentarios que existían al asignarlo: lo que agregue después, escríbalo en el PR.
3. Cuando Copilot empuja cambios, los workflows **no corren solos**: en el PR aparece **Approve and run workflows**. Revise el diff antes de aprobar la corrida, porque los workflows corren con el código que escribió el agente.
4. Para pedir cambios, comente en el PR mencionando a `@copilot`.
5. Al terminar, Copilot le pide revisión. Revise, pida la aprobación de alguien del equipo e integre.

Si el agente necesita herramientas instaladas (Semgrep, pytest), se le preparan en `.github/workflows/copilot-setup-steps.yml`, con un job llamado `copilot-setup-steps`. Ese archivo es un workflow más: necesita `permissions:` arriba y actions fijadas por hash, o `f3.permisos` y `f3.fijadas` fallan.

### 4b. O usar el modo agente en el Codespace

1. En el Codespace, abra el chat de Copilot y elija el modo **Agent**.
2. Cree una rama: `git switch -c bonus-ia-<usuario>`.
3. Pegue la tarea del issue. El agente lee `.github/copilot-instructions.md`, edita archivos y propone comandos; usted aprueba cada comando antes de que corra.
4. Revise el diff, haga commit y abra el PR. En la descripción escriba `Hecho con Copilot en modo agente` y enlace el issue.
5. Exporte la conversación desde el menú del chat y adjúntela o péguela en su archivo de evidencia. **Por confirmar:** el nombre exacto de la opción de exportar en la versión de VS Code del Codespace.

### 5. Documentar

Complete `seguridad/bonus-ia/<su-usuario>.md` con la tabla de la sección «Qué entrega». Lo más valioso del análisis son los momentos en que el agente se salió de los guardrails: por ejemplo, si para que una prueba pasara intentó cambiar la prueba en vez del código, o si agregó una dependencia sin fijar versión.

## Otros agentes

Cualquier agente gratuito sirve si cumple las reglas: por ejemplo, un agente de terminal dentro del Codespace con el plan gratuito de su proveedor. Ponga los mismos guardrails en el archivo que ese agente lee (muchos leen `AGENTS.md` en la raíz) y entregue la misma evidencia. Revise las condiciones de uso del proveedor y no le dé acceso a secretos.

## Cómo se califica

| Criterio | Peso dentro del bonus |
|:---|:---|
| Guardrails: requerimientos del equipo concretos, verificables y trazados | 30 % |
| PR: pasa el pipeline sin debilitarlo y cumple los requerimientos | 40 % |
| Evidencia y análisis crítico de lo que hizo el agente | 30 % |

El bonus suma hasta 10 puntos porcentuales a su nota individual del laboratorio. La nota total no pasa de 100 %.
