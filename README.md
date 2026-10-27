# sdlc-guia

Guía, tablero de avance y verificador central del Laboratorio SDLC Seguro de IC-8071 Seguridad del Software (Instituto Tecnológico de Costa Rica). Repositorio público.

| Pieza | Dónde | Qué es |
|:---|:---|:---|
| Guía web | `docs/` | Sitio Jekyll para GitHub Pages: inicio, fases 0 a 5, herramientas, bonus de IA y preguntas frecuentes |
| Tablero | `docs/tablero/index.html` | Página autocontenida para proyectar en clase. Lee `equipos.json` y el `estado.json` de cada equipo |
| Verificador central | `verificador/checkpoints.py` | Copia exacta de `lab/checkpoints.py` de la plantilla. Calcula la nota de checkpoints sin depender de archivos que los equipos puedan modificar |
| Equipos | `equipos.json` | Lista de equipos, apps, repositorios y páginas |
| Workflow | `.github/workflows/verificador.yml` | Corre el verificador a diario y a pedido, y guarda `docs/tablero/central.json` |

## Publicar el sitio

1. Cambie `org:` y `url:` en `docs/_config.yml`, y el enlace del tablero en `nav_external_links`, por el nombre de la organización. `profesor/scripts/publicar-plantillas.sh` lo hace solo.
2. **Settings → Pages → Build and deployment → Source: Deploy from a branch**, rama `main`, carpeta `/docs`.
3. El sitio queda en `https://<organizacion>.github.io/sdlc-guia/`. Los issues de fase de los equipos enlazan a `…/sdlc-guia/fase-N/`.

El tema es [Just the Docs](https://just-the-docs.com) v0.10.1, cargado con `remote_theme`, que GitHub Pages soporta. Las páginas que muestran expresiones de Actions (`${{ … }}`) las envuelven en `{% raw %}` para que Liquid no las interprete; mantenga esa convención si agrega ejemplos de workflows.

## equipos.json

Una entrada por equipo. El verificador usa `equipo` y `repo`; el tablero usa además `app` y `pages`.

```json
[
  {
    "equipo": "Equipo 1",
    "app": "OWASP Juice Shop",
    "repo": "organizacion/sdlc-juice-shop-equipo-1",
    "pages": "https://organizacion.github.io/sdlc-juice-shop-equipo-1/"
  }
]
```

- `repo` es `<organizacion>/<prefijo de la tarea>-<nombre del equipo>`, como lo crea GitHub Classroom.
- `pages` es opcional; si falta, el tablero la arma con el dueño en minúsculas y el nombre del repositorio.
- El tablero lee `docs/tablero/equipos.json`, que es una copia de la raíz: el workflow del verificador y `cerrar-sesion.sh` la actualizan. Si edita a mano, copie el archivo a las dos rutas.

## Verificador central

```bash
GITHUB_TOKEN=$(gh auth token) python3 verificador/checkpoints.py --lote equipos.json --salida docs/tablero/central.json
```

El workflow `verificador.yml` hace lo mismo a las 06:00 de Costa Rica y con **Run workflow**. Antes de evaluar comprueba que cada repositorio de `equipos.json` se pueda leer: si uno no es visible, se detiene en vez de calificarlo con 0 %.

- Sin secretos usa el `GITHUB_TOKEN`, que basta para leer repositorios públicos, con un límite de 1000 consultas por hora. Cada equipo consume del orden de decenas de consultas.
- Si los repositorios dejan de ser públicos o el límite no alcanza, cree un token de grano fino de solo lectura sobre los repositorios de equipo (Metadata, Contents, Issues, Actions y Administration en lectura) y guárdelo como secreto `VERIFICADOR_TOKEN`.
- Al cambiar `lab/checkpoints.py` en la plantilla, copie el archivo aquí y compruebe que son idénticos: `cmp ../sdlc-plantilla/base/lab/checkpoints.py verificador/checkpoints.py`.

## Tablero

`https://<organizacion>.github.io/sdlc-guia/tablero/`

- Cada 60 segundos pide `estado.json` a la página de cada equipo, sin caché. Si la página no responde, muestra el último resultado del verificador central (`central.json`) y lo marca así. Si tampoco hay, muestra al equipo como «sin Pages todavía».
- **Historial** muestra las fotos `docs/tablero/historial/sesion-N.json`, que guarda `profesor/scripts/cerrar-sesion.sh N` al cerrar cada sesión.
- Atajos: `H` historial, `P` todas las pendientes, `T` tema, `R` actualizar.
- Parámetros: `?demo=1` muestra datos de demostración generados en el navegador; `?tema=claro` u `?tema=oscuro` fija el tema; `?equipos=` y `?central=` cambian las rutas de los JSON.

El `estado.json` de cada equipo lo publica su propio pipeline, así que un equipo podría alterarlo. El tablero es para seguir la clase; la nota sale del verificador central.

Para revisarlo en su computadora:

```bash
cd docs/tablero && python3 -m http.server 8000
# http://localhost:8000/?demo=1
```

## Estructura

```
.github/workflows/verificador.yml   verificador central diario
equipos.json                        equipos del semestre
verificador/checkpoints.py          copia exacta del verificador de la plantilla
docs/_config.yml                    configuración de Jekyll y del tema
docs/index.md                       inicio
docs/fase-0/ … docs/fase-5/         una página por fase
docs/herramientas.md
docs/bonus-ia.md
docs/preguntas-frecuentes.md
docs/tablero/index.html             tablero
docs/tablero/equipos.json           copia de equipos.json para el tablero
docs/tablero/central.json           salida del verificador central (la crea el workflow)
docs/tablero/historial/             fotos al cierre de cada sesión
```
