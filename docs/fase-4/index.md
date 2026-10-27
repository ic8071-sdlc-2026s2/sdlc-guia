---
title: Fase 4 · Análisis dinámico y pruebas de abuso
nav_order: 6
---

# Fase 4 · Análisis dinámico y pruebas de abuso
{: .no_toc }

Sesión 4. Lidera el Equipo rojo.
{: .fs-5 .fw-300 }

{: .importante }
**Objetivo.** Atacar la app corriendo: un escaneo dinámico con ZAP y pruebas automáticas escritas desde sus casos de abuso. Registrar y trazar lo que encontraron todas las herramientas hasta ahora, y decidir qué se corrige.
**Touchpoints de McGraw:** 3 (pruebas de penetración) y 4 (pruebas de seguridad basadas en riesgo).

El pipeline **sigue solo reportando**. Las pruebas de abuso afirman el comportamiento seguro, así que fallan mientras la app sea vulnerable: ese rojo es lo esperado.

<details markdown="block">
<summary>Contenido de esta página</summary>
1. TOC
{:toc}
</details>

## Qué entrega

- `.github/workflows/fase4-dast.yml` con el paso de ZAP completo, endurecido como los demás.
- Al menos 3 pruebas en `seguridad/fase-4/pruebas/test_*.py`, cada una marcada con `@pytest.mark.caso_abuso(N)`.
- Al menos 4 issues de Hallazgo trazados, de al menos 3 herramientas distintas, cada uno con su decisión.

## Predicción con revelación

Antes de correr ZAP, el Equipo rojo escribe en el issue **Fase 4** una tabla con sus casos de abuso:

```markdown
### Predicción · ZAP
| Caso | ¿Lo verá ZAP? | Por qué |
|:---|:---|:---|
| #4 IDOR en pedidos | No | ZAP no sabe que el pedido 7 es de otra persona |
| #9 Cookie sin HttpOnly | Sí | Revisa las cabeceras de cada respuesta |
```

Después del escaneo, agreguen una columna con lo que pasó. La pregunta de fondo: ¿qué debilidades de sus casos puede ver un escáner dinámico y cuáles solo ve una prueba que conoce la lógica del negocio?

Hagan lo mismo con las pruebas: antes de correrlas, cada autor predice si su prueba va a fallar (app vulnerable) o a pasar.

## Pasos

### 1. Completar el workflow de DAST

```bash
cp lab/workflows/fase4-dast.yml .github/workflows/
```

El workflow construye la imagen, levanta la app en el runner con `lab/app.sh`, corre ZAP y luego pytest, y sube todo como artefacto `reporte-dast`. Usted escribe el paso de ZAP. El modo depende de la app y está en `ZAP_MODO`, dentro de `lab/app.env`:

| App | `ZAP_MODO` | Script de ZAP | Objetivo |
|:---|:---|:---|:---|
| Juice Shop | `baseline` | `zap-baseline.py` | `http://localhost:$APP_PUERTO` |
| PyGoat | `baseline` | `zap-baseline.py` | `http://localhost:$APP_PUERTO` |
| RailsGoat | `baseline` | `zap-baseline.py` | `http://localhost:$APP_PUERTO` |
| VAmPI | `api` | `zap-api-scan.py` con `-f openapi` y `-O http://localhost:$APP_PUERTO` | `http://localhost:$APP_PUERTO$ZAP_OPENAPI`, la especificación OpenAPI |

- **Baseline** recorre el sitio con el spider durante un minuto y hace solo análisis pasivo: mira cabeceras, cookies, mensajes de error y contenido. No ataca.
- **API scan** importa la especificación OpenAPI, llama cada endpoint y hace análisis **activo**: sí envía ataques. Por eso se corre solo contra la app propia, en el runner o en su Codespace. El `/openapi.json` de VAmPI no declara `servers`, así que ZAP no sabe a qué dirección llamar: pásela con `-O http://localhost:$APP_PUERTO`.

Corra ZAP con la imagen `ghcr.io/zaproxy/zaproxy:stable` y `docker run`:

- `--network host`, para que el contenedor de ZAP llegue a `localhost` del runner;
- `-v "$PWD/reportes:/zap/wrk:rw"`, porque ZAP escribe sus reportes en `/zap/wrk`;
- `-r zap.html -J zap.json`, para el reporte legible y el de datos;
- `-I`, para que las alertas de nivel advertencia no detengan el job.

Recuerde fijar por hash las actions del archivo copiado y mantener `permissions:` en el nivel superior: `f3.fijadas` y `f3.permisos` revisan todos los workflows.

{: .nota }
Existen las actions oficiales `zaproxy/action-baseline` y `zaproxy/action-api-scan`. Puede usarlas en lugar de `docker run`; si lo hace, fíjelas por hash y ponga `allow_issue_writing: false`, porque por omisión abren un issue con los resultados y necesitan permiso de escritura en issues.

### 2. Correr ZAP en el Codespace

Para iterar sin esperar al runner, con la app levantada:

```bash
bash lab/app.sh levantar
mkdir -p reportes && chmod 777 reportes
source lab/app.env
docker run --rm --network container:lab-app -v "$PWD/reportes:/zap/wrk:rw" \
  ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t "http://localhost:$APP_PUERTO" -r zap.html -I
```

`--network container:lab-app` pone a ZAP en la red del contenedor de la app, así que `localhost` es la app misma y el puerto es el interno, `APP_PUERTO`, no el que usted publicó en el Codespace con `APP_PUERTO_HOST`. Abra `reportes/zap.html` en el Codespace con clic derecho → **Open Preview** o descárguelo.

| App | Nota |
|:---|:---|
| Juice Shop | Es una SPA de Angular: el spider tradicional ve poco. Pruebe además `-j`, que agrega el spider AJAX y tarda más. Prediga antes qué cambia |
| PyGoat | Casi todo está detrás del inicio de sesión. Baseline sin autenticar ve el login, el registro y lo público. Compárelo con lo que encuentran sus pruebas autenticadas |
| VAmPI | Use `zap-api-scan.py -t "http://localhost:$APP_PUERTO$ZAP_OPENAPI" -f openapi -O "http://localhost:$APP_PUERTO"`. El escaneo activo puede cambiar datos: vuelva a llamar `/createdb` después |
| RailsGoat | Igual que PyGoat: lo interesante está detrás del inicio de sesión |

### 3. Escribir las pruebas de abuso

Cada prueba sale de un caso de abuso y afirma el «Entonces» **seguro** del requerimiento que lo mitiga. Mientras la app sea vulnerable, falla.

```python
import pytest


def iniciar_sesion(sesion, app_url, usuario, clave):
    """Adáptela a su app: ver la tabla de autenticación."""
    r = sesion.post(f"{app_url}/api/login", json={"usuario": usuario, "clave": clave})
    assert r.status_code == 200, f"no se pudo iniciar sesión ({r.status_code}): la prueba no llegó al ataque"
    sesion.headers["Authorization"] = f"Bearer {r.json()['token']}"


@pytest.mark.caso_abuso(4)
def test_cliente_no_puede_leer_pedido_ajeno(app_url, sesion):
    """Dado un cliente con sesión iniciada y el identificador de un pedido de otra persona
    Cuando pide ese pedido a la API
    Entonces la API responde 403 o 404 y no entrega los datos"""
    iniciar_sesion(sesion, app_url, "cliente1", "clave1")
    r = sesion.get(f"{app_url}/api/pedidos/2")  # el pedido 2 es de otra persona
    assert r.status_code in (403, 404), f"la API entregó un pedido ajeno con {r.status_code}"
```

El ejemplo es ilustrativo: los endpoints y los campos no son los de su app. Lo que se copia es la forma:

- una prueba por caso de abuso, con el número del issue en `@pytest.mark.caso_abuso(N)`;
- la docstring con el escenario del caso;
- los pasos previos (crear usuario, iniciar sesión) verifican que funcionaron, para que la prueba falle en el ataque y no antes;
- la aserción final describe el comportamiento seguro, con un mensaje que explique qué pasó.

Use los fixtures de `conftest.py`: `app_url` (la URL de la app, de `APP_URL` o de `lab/app.env`) y `sesion` (una `requests.Session`). Si necesita usuarios propios, créelos con nombres únicos, por ejemplo `f"rojo-{uuid.uuid4().hex[:8]}"`, para que las pruebas no dependan del orden.

| App | Cómo iniciar sesión desde una prueba | Usuarios de ejemplo |
|:---|:---|:---|
| Juice Shop | `POST /rest/user/login` con JSON `email` y `password`; la respuesta trae `authentication.token`. Envíelo como `Authorization: Bearer <token>` | `app/data/static/users.yml` |
| PyGoat | Formulario de Django: `GET /login/` deja la cookie `csrftoken`; luego `POST /login/` con `username`, `password` y `csrfmiddlewaretoken` (el valor de esa cookie). Puede registrar usuarios en `/register` | Regístrelos usted |
| VAmPI | `POST /users/v1/login` con JSON `username` y `password`; la respuesta trae `auth_token`. Envíelo como `Authorization: Bearer <token>` | Los que crea `/createdb`; ver `app/models/user_model.py` |
| RailsGoat | Formulario: `POST /sessions` con `email` y `password`; la sesión queda en la cookie. Revise en `app/app/controllers/sessions_controller.rb` qué espera | `app/db/seeds.rb` |

Corra las pruebas en el Codespace antes de subirlas:

```bash
bash lab/app.sh levantar
pytest seguridad/fase-4/pruebas -v
```

Si cambió el puerto con `APP_PUERTO_HOST`, indique la URL: `APP_URL=http://localhost:8081 pytest seguridad/fase-4/pruebas -v`.

{: .importante }
**Rojo por la razón correcta.** Una prueba de abuso bien escrita falla con `AssertionError` en la última línea, con un mensaje que describe la debilidad. Si falla con `ConnectionError`, `KeyError` o en el inicio de sesión, la prueba está rota, no la app.

### 4. Registrar y trazar hallazgos

Revise los reportes de Semgrep, Gitleaks, Dependabot, ZAP y sus pruebas. Por cada hallazgo que se pueda trazar a un caso de abuso o a un requerimiento, cree un issue con el formulario **Hallazgo**:

| Campo | Qué poner |
|:---|:---|
| Herramienta | La que lo encontró. Si fue una prueba suya, «Prueba pytest» |
| Predicción | Lo que esperaban antes de correr la herramienta, copiado del comentario de la predicción |
| Descripción | Qué reportó y qué significa en su app |
| Ubicación | `archivo:línea`, endpoint o paquete y versión |
| Trazabilidad | `#caso` y `#requerimiento`. Un hallazgo sin traza no suma |
| Decisión | Corregir, aceptar el riesgo con justificación o falso positivo |

### 5. Decidir qué se corrige

Su equipo corrige en la fase 5 **solo las debilidades trazadas a sus casos de abuso, al menos 3**. El resto se clasifica con justificación: aceptar el riesgo (con el riesgo de la matriz y por qué es tolerable) o falso positivo (con la evidencia). Elija para corregir los riesgos más altos de la matriz que tengan prueba de abuso: en la fase 5 esa prueba es la que demuestra la corrección.

## Checkpoint

| Identificador | Condición | Cómo se cumple |
|:---|:---|:---|
| `f4.dast` | `fase4-dast.yml` corre ZAP y pytest y terminó en verde | El archivo menciona ZAP y pytest fuera de los comentarios y tiene al menos una corrida exitosa. Las pruebas en rojo no lo impiden: ese paso tiene `continue-on-error`. ZAP sí: sin `-I`, las advertencias detienen el job |
| `f4.pruebas` | Al menos 3 pruebas marcadas con un caso de abuso existente | En los `test_*.py` de `seguridad/fase-4/pruebas/` hay al menos 3 `@pytest.mark.caso_abuso(N)` donde N es un issue de caso de abuso |
| `f4.hallazgos` | Al menos 4 hallazgos trazados de 3 herramientas distintas | Issues de Hallazgo con al menos un `#n` en «Trazabilidad»; entre ellos, al menos 3 valores distintos de «Herramienta» |

{: .nota }
La revisión docente abre `reportes/fase4-dast/` en la página de su equipo y busca el reporte de ZAP. Un workflow en verde sin reporte de ZAP no cumple el entregable aunque pase el checkpoint.

## Errores comunes

- **Olvidar `-I`.** ZAP termina con código 2 cuando hay advertencias y el job queda en rojo.
- **En modo api, apuntar a la raíz u olvidar `-O`.** `zap-api-scan.py` necesita la URL de la especificación (`$ZAP_OPENAPI`), `-f openapi` y, en VAmPI, `-O http://localhost:$APP_PUERTO` porque la especificación no declara `servers`. Sin `-O`, ZAP no llama a ningún endpoint y el reporte sale casi vacío.
- **Marcar la prueba con el número de un requerimiento** en vez del caso de abuso.
- **Escribir la prueba al revés**: afirmar que el ataque funciona. Entonces pasa en verde con la app vulnerable y, cuando corrijan, falla. La prueba afirma el comportamiento seguro.
- **Marcadores mal escritos.** `pytest.ini` usa `--strict-markers`: `caso_abuso` y `corregida` se escriben exactamente así.
- **Pruebas que dependen de otras** (una crea el usuario, otra lo usa). Cada prueba prepara lo suyo.
- **Copiar el workflow sin fijar sus actions.** `f3.fijadas` vuelve a fallar.
- **Registrar hallazgos sin traza** o con todos de la misma herramienta.

## Pistas

<details markdown="block">
<summary>Pista 1 · ¿Qué caso convierto primero en prueba?</summary>
Los que tienen un «Entonces» observable en una respuesta HTTP: un código de estado, un campo que no debería venir, un dato ajeno. IDOR, función de administración sin rol, inicio de sesión sin límite de intentos y cabeceras ausentes son buenos primeros candidatos. El repudio (falta de bitácora) es difícil de probar desde afuera: déjelo para revisión manual.
</details>

<details markdown="block">
<summary>Pista 2 · Patrones para inicio de sesión sin bloqueo y cabeceras</summary>

```python
@pytest.mark.caso_abuso(2)
def test_inicio_de_sesion_se_bloquea_tras_intentos_fallidos(app_url, sesion):
    for _ in range(10):
        r = sesion.post(URL_LOGIN, json={"usuario": "cliente1", "clave": "incorrecta"})
    assert r.status_code in (423, 429), f"tras 10 intentos fallidos la app respondió {r.status_code}"


@pytest.mark.caso_abuso(9)
def test_respuestas_traen_cabeceras_de_seguridad(app_url, sesion):
    r = sesion.get(f"{app_url}/")
    faltan = [c for c in ("X-Content-Type-Options", "Content-Security-Policy") if c not in r.headers]
    assert not faltan, f"faltan cabeceras: {faltan}"
```

`URL_LOGIN` y los campos dependen de su app. Mantenga bajo el número de intentos: la prueba corre en cada push.
</details>

<details markdown="block">
<summary>Pista 3 · El paso de ZAP completo</summary>

```yaml
      - name: Escaneo dinámico con ZAP
        run: |
          mkdir -p reportes && chmod 777 reportes
          source lab/app.env
          if [ "$ZAP_MODO" = "api" ]; then
            docker run --rm --network host -v "$PWD/reportes:/zap/wrk:rw" ghcr.io/zaproxy/zaproxy:stable \
              zap-api-scan.py -t "http://localhost:$APP_PUERTO$ZAP_OPENAPI" -f openapi \
              -O "http://localhost:$APP_PUERTO" -r zap.html -J zap.json -I
          else
            docker run --rm --network host -v "$PWD/reportes:/zap/wrk:rw" ghcr.io/zaproxy/zaproxy:stable \
              zap-baseline.py -t "http://localhost:$APP_PUERTO" -r zap.html -J zap.json -I
          fi
```
</details>

Anterior: [Fase 3](../fase-3/) · Siguiente: [Fase 5 · Release firmado y gate](../fase-5/).
