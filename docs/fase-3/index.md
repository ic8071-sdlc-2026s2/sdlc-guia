---
title: Fase 3 · Análisis estático y cadena de suministro
nav_order: 5
---

# Fase 3 · Análisis estático y cadena de suministro
{: .no_toc }

Sesión 3. Lidera DevSecOps; AppSec lidera la regla propia de Semgrep.
{: .fs-5 .fw-300 }

{: .importante }
**Objetivo.** Poner en el pipeline de cada pull request un análisis estático (Semgrep), una búsqueda de secretos (Gitleaks) y una revisión de dependencias (Dependabot y Dependency Review). Escribir una regla propia de Semgrep que verifique uno de sus requerimientos. Endurecer el pipeline mismo, que también es superficie de ataque.
**Touchpoint de McGraw:** 1 (revisión de código con herramientas).

En esta fase el pipeline **solo reporta**: ningún hallazgo detiene el PR. La única excepción es la prueba de su regla propia: si falla, el job falla.

<details markdown="block">
<summary>Contenido de esta página</summary>
1. TOC
{:toc}
</details>

## Qué entrega

| Archivo o ajuste | Qué hace |
|:---|:---|
| `.github/workflows/fase3-seguridad-pr.yml` | Semgrep, Gitleaks y Dependency Review en cada PR |
| `seguridad/fase-3/semgrep/<nombre>.yml` y `<nombre>.<ext>` | Regla propia y su prueba |
| `.github/dependabot.yml` | Vigila las dependencias de `app/` y las actions |
| `.github/workflows/scorecard.yml` | OpenSSF Scorecard califica las prácticas del repositorio |
| Todos los workflows | `permissions:` en el nivel superior y actions fijadas por hash |
| Ruleset de `main` | Exige pull request para integrar |

## Orden de trabajo recomendado

1. Escriban la predicción (abajo) antes de correr nada.
2. Creen una rama: `git switch -c fase-3`.
3. Hagan los pasos 1 a 5 en esa rama y abran un PR hacia `main`.
4. Mientras el PR corre, creen el ruleset (paso 6).
5. Revisen el PR en verde, que otra persona del equipo lo apruebe e intégrenlo.
6. Registren los hallazgos (paso 7).

## Predicción con revelación

Antes de correr cada herramienta, el Equipo rojo conduce la predicción y la escribe como comentario en el issue **Fase 3**, con este formato:

```markdown
### Predicción · Semgrep
Esperamos que encuentre:
1. <debilidad, archivo o endpoint, y por qué>
Esperamos que NO encuentre:
1. <debilidad de un caso de abuso que un SAST genérico no puede ver, y por qué>
Confianza: alta, media o baja
```

Hagan lo mismo para Gitleaks («¿cuántos secretos y dónde?»), Dependabot («¿qué paquete y qué severidad?») y Scorecard («¿qué puntaje de 0 a 10 y qué chequeo falla?»). Cuando la herramienta termine, respondan al comentario con la **revelación**: aciertos, fallos y sorpresas. Copien la predicción en el campo «Predicción» de cada issue de Hallazgo. La hora del comentario frente a la hora de la corrida en Actions es la evidencia de que la predicción fue antes.

## Pasos

### 1. Workflow de seguridad en PR

Copie la plantilla y complete los `TODO`:

```bash
cp lab/workflows/fase3-seguridad-pr.yml .github/workflows/
```

No cambie el nombre del archivo ni el `name:` del workflow: «Checkpoints» se dispara cuando termina un workflow con ese nombre.

| Job | Qué trae | Qué falta |
|:---|:---|:---|
| `sast` | Semgrep CE 1.182.0 con `p/default` y sus reglas propias, reporte SARIF | Opcional: subir el SARIF a Code scanning |
| `secretos` | El checkout con todo el historial | Correr la imagen `ghcr.io/gitleaks/gitleaks:v8.30.1` |
| `dependencias` | Solo corre en PR | El paso de `actions/dependency-review-action` |

**Gitleaks.** Corre como contenedor de Docker, no como action: la action oficial pide licencia para organizaciones. Use el subcomando `git`, que revisa todo el historial, sobre el repositorio montado en `/repo`. Pida reporte SARIF en `reportes/gitleaks.sarif`, `--redact` para que el secreto no se copie al reporte y `--exit-code 0` para que solo reporte.

**Dependency Review.** Compara las dependencias del PR contra las de `main` y avisa si el PR agrega una con vulnerabilidad conocida. Configúrela con `fail-on-severity: high` y `warn-only: true`. En la fase 5 quitará `warn-only`.

### 2. Regla propia de Semgrep

La regla verifica uno de **sus** requerimientos: que el código de `app/` lo cumpla o que nadie lo rompa en un PR futuro. Las instrucciones y un ejemplo completo están en `seguridad/fase-3/semgrep/README.md`.

| App | `languages:` | Archivo de prueba | Comentario de prueba |
|:---|:---|:---|:---|
| Juice Shop | `[typescript]` | `<nombre>.ts` | `// ruleid: <id>` y `// ok: <id>` |
| PyGoat | `[python]` | `<nombre>.py` | `# ruleid: <id>` y `# ok: <id>` |
| VAmPI | `[python]` | `<nombre>.py` | `# ruleid: <id>` y `# ok: <id>` |
| RailsGoat | `[ruby]` | `<nombre>.rb` | `# ruleid: <id>` y `# ok: <id>` |

1. Escriba un fragmento de código con la debilidad y otro sin ella.
2. Itere en el [Semgrep Playground](https://semgrep.dev/playground): regla a la izquierda, código a la derecha, **Run**.
3. Guarde la regla en `seguridad/fase-3/semgrep/<nombre>.yml` con `metadata.requerimiento: "#N"`, donde N es el número del issue del requerimiento.
4. Guarde el fragmento como `<nombre>.<ext>`, con `ruleid:` en la línea anterior a la que debe disparar la regla y `ok:` en la anterior a la que no.
5. Pruebe en el Codespace:

```bash
semgrep --test seguridad/fase-3/semgrep
semgrep scan app --config seguridad/fase-3/semgrep --metrics off
```

Si la regla encuentra la debilidad en `app/`, regístrela como Hallazgo con herramienta «Semgrep».

### 3. Dependabot

Cree `.github/dependabot.yml`. Use la forma `directory: "/app"`: el verificador no reconoce la lista `directories:`.

| App | Ecosistema | Directorio |
|:---|:---|:---|
| Juice Shop | `npm` | `/app` y `/app/frontend` (dos entradas) |
| PyGoat | `pip` | `/app` |
| VAmPI | `pip` | `/app` |
| RailsGoat | `bundler` | `/app` |
| Todas | `github-actions` | `/` |
| Todas, opcional | `docker` | `/app` |

```yaml
version: 2
updates:
  - package-ecosystem: "pip"        # npm, pip o bundler, según la tabla
    directory: "/app"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 3
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

{: .nota }
**Juice Shop no trae `package-lock.json`.** Sin lockfile, un escaneo de Trivy sobre el sistema de archivos de `app/` no ve las dependencias npm. En Juice Shop el SCA real sale de Dependabot, que lee `package.json`, y de Trivy sobre la imagen construida en la fase 5, que sí contiene `node_modules`. Tome eso en cuenta en su predicción.

`open-pull-requests-limit` evita que Dependabot llene el repositorio de PR: las cuatro apps tienen muchas dependencias viejas. Las alertas de Dependabot aparecen en **Security → Dependabot**; actívelas en **Settings → Advanced Security** si no lo están.

### 4. OpenSSF Scorecard

```bash
cp lab/workflows/scorecard.yml .github/workflows/
```

Fije sus actions por hash como las demás y no le agregue pasos ni variables de entorno: con `publish_results: true`, Scorecard rechaza workflows que no sigan su forma. Corre en cada push a `main`, así que la primera corrida llega cuando integre el PR. El resultado aparece en **Security → Code scanning** y en `https://scorecard.dev/viewer/?uri=github.com/{{ site.org }}/<repositorio>`.

### 5. Endurecer todos los workflows

Esto aplica a **todos** los archivos de `.github/workflows/`, incluido `fase0-ci.yml`, y a los que agregue en las fases 4 y 5. Si después agrega un workflow sin endurecer, `f3.fijadas` y `f3.permisos` vuelven a fallar.

**Permisos mínimos.** Declare `permissions:` en el nivel superior del archivo, sin sangría. Lo más seguro es `permissions: {}` arriba y dar a cada job solo lo que necesita.

| Workflow | Nivel superior | Cada job |
|:---|:---|:---|
| `fase0-ci.yml` | `permissions: {}` | `contents: read` |
| `fase3-seguridad-pr.yml` | `permissions: {}` | `contents: read`; más `security-events: write` si sube SARIF a Code scanning |
| `fase4-dast.yml` | ya trae `contents: read` | |
| `scorecard.yml` | ya trae `read-all`, que Scorecard exige | ya trae `security-events: write` e `id-token: write` |
| `checkpoints.yml` | ya está endurecido; no lo modifique | |

**Actions fijadas por hash.** Una etiqueta como `v7` se puede mover a otro commit; un hash de 40 caracteres no. En marzo de 2025 alguien movió las etiquetas de `tj-actions/changed-files` a un commit malicioso que imprimía los secretos de los workflows en sus bitácoras. Los repositorios que la usaban fijada por hash no se vieron afectados.

Cada `uses:` debe quedar así, con la versión en el comentario para las personas y para Dependabot:

```yaml
- uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
```

#### Cómo obtener el hash de una action

1. Busque la versión exacta más reciente en la página de Releases de la action, o con `gh`:

   ```bash
   gh release view --repo actions/checkout --json tagName --jq .tagName
   ```

2. Pida a la API a qué apunta esa etiqueta:

   ```bash
   gh api repos/actions/checkout/git/ref/tags/v7.0.1 --jq '.object.type + " " + .object.sha'
   ```

3. Si la respuesta dice `commit`, ese hash es el que necesita.
4. Si dice `tag`, es una **etiqueta anotada**: el hash es del objeto etiqueta, no del commit. Desreferéncielo:

   ```bash
   gh api repos/<dueño>/<action>/git/tags/<hash-del-objeto-tag> --jq '.object.type + " " + .object.sha'
   ```

   Repita hasta que el tipo sea `commit`.

Atajo equivalente con git: la línea que termina en `^{}` es el commit de una etiqueta anotada.

```bash
git ls-remote https://github.com/actions/checkout 'refs/tags/v7.0.1*'
```

Para una action dentro de un subdirectorio, como `github/codeql-action/upload-sarif`, el hash es del repositorio `github/codeql-action` y la línea queda `github/codeql-action/upload-sarif@<hash> # v4.x.y`. Los workflows `checkpoints.yml` de su repositorio ya traen tres actions fijadas que puede usar de modelo.

Dependabot, con el ecosistema `github-actions`, abre PR para actualizar estos hashes y su comentario.

### 6. Ruleset que exige pull request

Necesita permisos de administración en el repositorio; la invitación del profesor se los dio.

1. **Settings → Rules → Rulesets → New ruleset → New branch ruleset**.
2. **Ruleset name:** `main protegida`. **Enforcement status:** Active.
3. **Target branches → Add target → Include default branch**.
4. Deje marcadas **Restrict deletions** y **Block force pushes**.
5. Marque **Require a pull request before merging**, con **Required approvals: 1**.
6. **Create**.

Lo mismo desde la terminal del Codespace. `gh` sustituye `{owner}` y `{repo}` por los del repositorio actual:

```bash
cat > /tmp/ruleset.json <<'EOF'
{
  "name": "main protegida",
  "target": "branch",
  "enforcement": "active",
  "conditions": { "ref_name": { "include": ["~DEFAULT_BRANCH"], "exclude": [] } },
  "rules": [
    { "type": "deletion" },
    { "type": "non_fast_forward" },
    { "type": "pull_request", "parameters": {
        "required_approving_review_count": 1,
        "dismiss_stale_reviews_on_push": true,
        "require_code_owner_review": false,
        "require_last_push_approval": false,
        "required_review_thread_resolution": false } }
  ]
}
EOF
gh api -X POST 'repos/{owner}/{repo}/rulesets' --input /tmp/ruleset.json
```

Verifique con la misma consulta que usa el verificador:

```bash
gh api 'repos/{owner}/{repo}/rules/branches/main' --jq '.[].type'
```

La lista debe incluir `pull_request`. Desde ahora, todo cambio a `main` entra por PR y alguien más del equipo lo aprueba.

### 7. Registrar hallazgos

Descargue los reportes del PR (**Actions → la corrida → Artifacts**) o ábralos en la página del equipo, que «Checkpoints» publica en `reportes/`. Por cada hallazgo relevante para sus casos de abuso, cree un issue con el formulario **Hallazgo**: herramienta, predicción, ubicación, severidad, trazabilidad (`#caso` o `#requerimiento`) y decisión. En la fase 4 necesitará al menos 4 trazados de 3 herramientas distintas; empiece aquí.

No registre cada línea de cada reporte. Registre lo que se puede trazar a un caso de abuso o a un requerimiento, y agrupe lo repetido.

## Checkpoint

| Identificador | Condición | Cómo se cumple |
|:---|:---|:---|
| `f3.workflow` | `fase3-seguridad-pr.yml` corre Semgrep, Gitleaks y Dependency Review | El archivo, sin contar los comentarios, menciona `semgrep`, `gitleaks` y `dependency-review`. Los `TODO` comentados no cuentan |
| `f3.pr` | Ese workflow corrió en verde sobre un pull request | Al menos una corrida exitosa con evento `pull_request`. Las corridas por push a `main` no cuentan |
| `f3.regla` | Hay una regla propia de Semgrep con prueba que verifica un requerimiento | Un `.yml` en `seguridad/fase-3/semgrep/` con `requerimiento: "#N"`, donde #N es un issue de requerimiento, y un archivo con el mismo nombre base que contiene `ruleid:` |
| `f3.dependabot` | `dependabot.yml` vigila las dependencias de `app/` | `.github/dependabot.yml` contiene `directory: "/app` (o `/app/frontend`) |
| `f3.fijadas` | Todas las actions están fijadas por hash | Cada `uses:` de cada workflow tiene `@` y 40 caracteres hexadecimales, salvo las actions locales `./` |
| `f3.permisos` | Cada workflow declara `permissions` en el nivel superior | Cada archivo de `.github/workflows/` tiene una línea `permissions:` sin sangría |
| `f3.rama` | Un ruleset exige pull request para `main` | `rules/branches/main` incluye una regla de tipo `pull_request` |
| `f3.scorecard` | OpenSSF Scorecard corrió al menos una vez | Un workflow que usa `ossf/scorecard-action` tiene al menos una corrida exitosa |

## Errores comunes

- **Usar la protección de rama clásica** (Settings → Branches) en vez de un ruleset. El verificador consulta los rulesets.
- **Fijar por hash solo los workflows nuevos.** `fase0-ci.yml` también usa etiquetas.
- **Copiar el hash del objeto etiqueta** de una etiqueta anotada. Si `gh api …/git/ref/tags/…` dice `tag`, desreferencie.
- **Declarar `permissions` solo dentro de los jobs.** También hace falta en el nivel superior del archivo.
- **Usar `directories:`** en `dependabot.yml`. Use `directory:`, una entrada por carpeta.
- **Poner en `metadata.requerimiento` el número de un caso de abuso.** Debe ser un issue de requerimiento.
- **Archivo de prueba con otro nombre base** (`regla.yml` y `prueba-regla.py`). Deben llamarse igual.
- **Agregar pasos al workflow de Scorecard.** Con `publish_results: true` falla la verificación del workflow.
- **Cambiar el `name:` de un workflow.** «Checkpoints» deja de dispararse cuando ese workflow termina.
- **Aprobar su propio PR.** GitHub no lo permite: necesita a otra persona del equipo.

## Pistas

<details markdown="block">
<summary>Pista 1 · Gitleaks no encuentra el repositorio o dice «dubious ownership»</summary>
Dentro del contenedor, git desconfía de un repositorio de otro dueño. Pase a `docker run` el usuario del runner con `-u "$(id -u):$(id -g)"` y monte el repositorio con `-v "$PWD:/repo"`.
</details>

<details markdown="block">
<summary>Pista 2 · Ideas de reglas propias que verifican un requerimiento</summary>

| Requerimiento | Qué busca la regla |
|:---|:---|
| 1.2.4, consultas parametrizadas | SQL armado con interpolación: plantillas de texto en TypeScript, f-strings o `%` en Python, `#{}` dentro de `where` o `find_by_sql` en Ruby |
| 9.1.1, firma del token | Decodificar un JWT sin verificar la firma o aceptando el algoritmo `none` |
| 13.3.1, secretos | Una llave, contraseña o `SECRET_KEY` asignada como texto literal |
| 3.3.4, cookie HttpOnly | Una cookie de sesión creada con `httpOnly` o `httponly` en falso |
| 8.2.1, control por función | Una ruta de administración sin el decorador o el middleware de autorización (use `pattern-not-inside`) |

Elija una que corresponda a un requerimiento **suyo** y a un caso de abuso **suyo**.
</details>

<details markdown="block">
<summary>Pista 3 · Los dos huecos del workflow</summary>

```yaml
      - name: Buscar secretos en todo el historial
        run: |
          mkdir -p reportes
          docker run --rm -u "$(id -u):$(id -g)" -v "$PWD:/repo" \
            ghcr.io/gitleaks/gitleaks:v8.30.1 git /repo \
            --redact --exit-code 0 \
            --report-format sarif --report-path /repo/reportes/gitleaks.sarif
```

```yaml
      - uses: actions/dependency-review-action@<hash> # v5.x.y
        with:
          fail-on-severity: high
          warn-only: true
```

Para ver a Dependency Review en acción, abra un PR de prueba que agregue a `app/` una versión vieja de un paquete con una vulnerabilidad alta publicada en la [GitHub Advisory Database](https://github.com/advisories). Antes, prediga qué va a decir.
</details>

Anterior: [Fase 2](../fase-2/) · Siguiente: [Fase 4 · Análisis dinámico y pruebas de abuso](../fase-4/).
