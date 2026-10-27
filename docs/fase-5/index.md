---
title: Fase 5 · Release firmado y gate
nav_order: 7
---

# Fase 5 · Release firmado y gate
{: .no_toc }

Sesión 5, primeros 35 minutos; se completa hasta el cierre del curso. Lidera DevSecOps; Arquitectura lidera la línea base.
{: .fs-5 .fw-300 }

{: .importante }
**Objetivo.** Corregir las debilidades trazadas que su equipo decidió corregir y publicar un release que solo sale si pasa el gate: imagen de contenedor construida en Actions, escaneada, con SBOM, firmada y con attestations de procedencia verificables.
**Touchpoint de McGraw:** 7 (operaciones de seguridad).

Aquí el pipeline **bloquea**. El release no sale si Trivy encuentra vulnerabilidades altas o críticas fuera de la línea base justificada, ni si falla una prueba marcada `corregida`. Dependency Review bloquea los PR que agregan dependencias con severidad alta.

<details markdown="block">
<summary>Contenido de esta página</summary>
1. TOC
{:toc}
</details>

## Qué entrega

| Entregable | Dónde |
|:---|:---|
| Al menos 3 debilidades trazadas corregidas, cada una con su PR, su prueba `corregida` y su issue cerrado | `app/`, `seguridad/fase-4/pruebas/`, issues |
| Workflow de release completo | `.github/workflows/fase5-release.yml` |
| Línea base justificada | `seguridad/fase-5/.trivyignore.yaml` |
| Dependency Review en modo bloqueo | `.github/workflows/fase3-seguridad-pr.yml`, sin `warn-only` |
| GitHub Release con el SBOM y el digest de la imagen | Pestaña Releases |
| Presentación de 5 a 7 minutos | `seguridad/fase-5/presentacion.md` |
| Reflexión de cada integrante, con commit propio | `seguridad/reflexiones/<usuario>.md` |

## Predicción con revelación

Antes de la primera corrida del release, DevSecOps escribe en el issue **Fase 5**:

1. ¿Cuántas vulnerabilidades altas y críticas tendrá la imagen? ¿En qué capa: el sistema operativo base o las dependencias de la app?
2. ¿Pasa el gate a la primera?
3. Si un PR agrega una dependencia con una vulnerabilidad alta, ¿qué lo detiene y en qué momento?

Después de la corrida, respondan con la revelación. La comparación entre la cantidad de vulnerabilidades predicha y la real suele ser la diapositiva más elocuente de la presentación.

## Pasos

### 1. Corregir al menos 3 debilidades

Corrija solo las debilidades trazadas a sus casos de abuso que marcó «Corregir» en la fase 4. Por cada una:

1. Cree una rama: `git switch -c corregir-<caso>`.
2. Cambie el código en `app/`. Corrija la causa, no el síntoma: si el caso es un IDOR, la API verifica que el recurso sea de quien lo pide; no basta con esconder el botón.
3. Agregue `@pytest.mark.corregida` a la prueba de ese caso y córrala en el Codespace con la app reconstruida (`bash lab/app.sh levantar`). Debe pasar en verde.
4. Abra el PR con `Closes #<hallazgo>` en la descripción. Al integrarlo, GitHub cierra el issue del hallazgo.
5. Revise que el campo **Decisión** del issue diga «Corregir». Si la decisión cambió, edite el cuerpo del issue.

```python
@pytest.mark.corregida
@pytest.mark.caso_abuso(4)
def test_cliente_no_puede_leer_pedido_ajeno(app_url, sesion):
    ...
```

{: .nota }
En VAmPI, corregir no es arrancar con `vulnerable=0`: eso apaga todas las debilidades a la vez y no demuestra nada. Corregir es **eliminar la rama vulnerable** (`if vuln: …`) del endpoint que trazó. La imagen del release sigue corriendo con `vulnerable=1`, y aun así la prueba `corregida` debe pasar. Cambiar `vulnerable=0` no cuenta como corrección.

### 2. Escribir el workflow de release

```bash
cp lab/workflows/fase5-release.yml .github/workflows/
```

La plantilla trae los permisos del job y la lista de 10 pasos como comentarios. Usted escribe cada paso. Use las actions de la tabla, fijadas por hash como explica la sección «Cómo obtener el hash de una action» de la [fase 3](../fase-3/).

| Paso | Action o comando | Claves |
|:---|:---|:---|
| 1. Iniciar sesión en ghcr.io | `docker/login-action` | `registry: ghcr.io`, el usuario del actor y el `GITHUB_TOKEN` como contraseña |
| 2. Nombre y etiquetas | `docker/metadata-action` | `images: ghcr.io/<repositorio>`; la etiqueta sale del tag `v*` |
| 3. Construir y publicar | `docker/build-push-action` | `context: app`, `push: true` y un `id` para leer `outputs.digest` |
| 4. Gate de vulnerabilidades | `aquasecurity/trivy-action` | `severity: HIGH,CRITICAL`, `exit-code: "1"`, `trivyignores: seguridad/fase-5/.trivyignore.yaml` |
| 5. SBOM | `aquasecurity/trivy-action` | `format: cyclonedx`, `output: sbom.cdx.json` |
| 6. Gate de correcciones | `lab/app.sh iniciar` y pytest | `pytest -m corregida` contra la imagen publicada |
| 7. Firma | `sigstore/cosign-installer` y `cosign sign --yes` | Siempre por digest, nunca por etiqueta |
| 8. Attestations | `actions/attest-build-provenance` y `actions/attest-sbom` | `subject-name`, `subject-digest`, `push-to-registry: true` |
| 9. Verificación | `gh attestation verify` y `cosign verify` | El workflow comprueba lo que acaba de publicar |
| 10. Release | `gh release create` | Adjunte `sbom.cdx.json` y escriba en las notas `Imagen: ghcr.io/…@sha256:…` |

Detalles que suelen trabar:

- **Nombre en minúsculas.** ghcr.io exige minúsculas y el nombre de la organización puede tener mayúsculas. `metadata-action` las convierte en sus etiquetas; para armar `imagen@digest` en sus propios pasos, use la expansión de bash `${GITHUB_REPOSITORY,,}`.
- **Versión de Trivy.** `trivy-action` v0.36.0 instala por omisión una versión anterior de Trivy (v0.70.0). Para usar la del curso, agregue `version: v0.76.1`. En la segunda llamada (SBOM) puede agregar `skip-setup-trivy: true`.
- **Versión de cosign.** `sigstore/cosign-installer` v4.1.2 instala por omisión cosign v3.0.6. Para usar la v3.1.4 del curso, agregue `cosign-release: v3.1.4`.
- **Trivy también busca secretos** dentro de la imagen. Si el gate se detiene por un secreto, decida: corregirlo o justificarlo en la sección `secrets:` de la línea base.
- **Python para el gate de correcciones.** Use `actions/setup-python` como en la fase 4 antes de instalar los requisitos de las pruebas.
- **El release solo se crea desde un tag.** El workflow también tiene `workflow_dispatch`; ponga `if: startsWith(github.ref, 'refs/tags/v')` en el paso del release.
- **El digest en las notas.** El verificador toma el primer `sha256:` de 64 caracteres que aparece en las notas del release más reciente. Que ese primero sea el de la imagen.

{: .nota }
En este diseño la imagen se publica antes del escaneo, así que una imagen vulnerable puede quedar en el registro aunque el gate falle; lo que no sale es la firma, la attestation ni el release. Un diseño más estricto construye con `load: true`, escanea la imagen local y publica solo si pasa. Si lo implementa, cuéntelo en la presentación.

### 3. Construir la línea base

La línea base es la lista de vulnerabilidades altas y críticas que el equipo acepta, cada una con su justificación. Las cuatro apps traen decenas: sin línea base, el gate nunca pasa.

{: .importante }
**Primero la imagen base.** Si la imagen trae cientos de vulnerabilidades del sistema operativo, la salida correcta no es llenar la línea base: es cambiar la imagen base en `app/Dockerfile` por una variante `slim` o distroless, que trae menos paquetes. Caso medido para este curso: RailsGoat sobre la imagen completa `ruby:3.4.1` da 2586 vulnerabilidades altas y críticas. Cambiar la base es una corrección de cadena de suministro: regístrela como hallazgo de Trivy con decisión «Corregir» y pruebe que la app sigue levantando.

1. Construya la imagen en el Codespace y escanéela:

   ```bash
   bash lab/app.sh construir
   docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.76.1 \
     image --severity HIGH,CRITICAL lab-app:local
   ```

2. Para cada vulnerabilidad decida: **actualizar** la dependencia (es una corrección más), **aceptar** con justificación o descartar como **falso positivo** con evidencia.
3. Agrupe por paquete: un issue de Hallazgo con herramienta «Trivy» por paquete, con su decisión y su justificación.
4. Escriba cada excepción en `seguridad/fase-5/.trivyignore.yaml` con su `statement`, que cita el hallazgo:

   ```yaml
   vulnerabilities:
     - id: CVE-2024-00000
       statement: "Paquete de desarrollo que no se carga en tiempo de ejecución. Hallazgo #31."
       expired_at: 2026-12-31
   ```

`expired_at` hace que la excepción venza y el gate vuelva a bloquear: una línea base sin vencimiento se olvida. `ignore-unfixed: true` en el gate (ignorar lo que todavía no tiene parche publicado) es aceptable si lo justifica en la presentación: nadie puede actualizar a una versión que no existe, pero el riesgo sigue ahí.

### 4. Activar el bloqueo de Dependency Review

1. En `.github/workflows/fase3-seguridad-pr.yml`, quite `warn-only: true`.
2. En el ruleset de `main`, marque **Require status checks to pass** y agregue el check **Dependencias nuevas con Dependency Review**. GitHub solo ofrece checks que ya corrieron alguna vez en el repositorio.
3. Para la demo, abra un PR que agregue una dependencia con vulnerabilidad alta y deje el PR abierto: es la evidencia de «un PR que el gate bloqueó».

### 5. Publicar el release

```bash
git switch main && git pull
git tag v1.0.0
git push origin v1.0.0
```

Si el gate falla, corrija y publique `v1.0.1`. No mueva una etiqueta ya publicada: es el mismo problema que las actions sin fijar.

### 6. Verificar la attestation y la firma

Desde el Codespace, con el digest de las notas del release. Si la imagen es privada, primero `docker login ghcr.io`; para que otras personas la verifiquen, haga público el paquete en **Packages → el paquete → Package settings → Change visibility**.

```bash
IMAGEN=ghcr.io/<organizacion>/<repositorio>      # en minúsculas
DIGEST=sha256:<64 caracteres de las notas del release>

# Procedencia (SLSA) emitida por actions/attest-build-provenance
gh attestation verify "oci://$IMAGEN@$DIGEST" -R <organizacion>/<repositorio>

# SBOM atestado por actions/attest-sbom
gh attestation verify "oci://$IMAGEN@$DIGEST" -R <organizacion>/<repositorio> \
  --predicate-type https://cyclonedx.org/bom

# Lo mismo que consulta el verificador
gh api "repos/<organizacion>/<repositorio>/attestations/$DIGEST" --jq '.attestations | length'

# Firma de cosign: quién firmó (el workflow, desde un tag) y quién emitió la identidad (Actions)
cosign verify "$IMAGEN@$DIGEST" \
  --certificate-identity-regexp '^https://github\.com/<organizacion>/<repositorio>/\.github/workflows/fase5-release\.yml@refs/tags/v' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com
```

El Codespace no trae cosign. Descargue el binario `cosign-linux-amd64` de la versión v3.1.4 desde la [página de releases de cosign](https://github.com/sigstore/cosign/releases) y verifique su checksum. Use la misma versión mayor que firmó: cosign v3 guarda la firma en un formato que versiones viejas no leen.

Pregúntese qué demuestra cada verificación: la attestation dice **dónde y cómo** se construyó la imagen (qué repositorio, qué workflow, qué commit); la firma dice **quién** la firmó. Ninguna de las dos dice que la imagen sea segura: eso lo dice el gate.

### 7. Presentación y reflexiones

- Complete `seguridad/fase-5/presentacion.md` y borre cada comentario `<!-- COMPLETAR … -->`. La sección 3 es la tabla de qué técnica encontró qué: es la lección central del laboratorio.
- Cada integrante copia `seguridad/reflexiones/plantilla.md` a `seguridad/reflexiones/<su-usuario>.md`, la completa y la sube **desde su propia cuenta**. El nombre del archivo debe ser su usuario de GitHub tal como aparece en `EQUIPO.md`.

## Checkpoint

| Identificador | Condición | Cómo se cumple |
|:---|:---|:---|
| `f5.release` | `fase5-release.yml` escanea, genera SBOM, firma, atesta y terminó en verde | El archivo, sin comentarios, menciona `trivy`, `sbom`, `cosign` y `attest-build-provenance`, y tiene al menos una corrida exitosa |
| `f5.sbom` | Un GitHub Release publica el SBOM | Un release tiene un archivo adjunto cuyo nombre contiene `sbom` |
| `f5.attestation` | El digest del release tiene attestations verificables | El primer `sha256:…` de las notas del release más reciente tiene attestations en la API de GitHub |
| `f5.correcciones` | Al menos 3 debilidades corregidas | Al menos 3 `@pytest.mark.corregida` en las pruebas y al menos 3 issues de Hallazgo cerrados cuya Decisión empieza con «Corregir» |
| `f5.lineabase` | Cada excepción de la línea base tiene justificación | En `.trivyignore.yaml`, la cantidad de `- id:` es igual a la de `statement:` con texto |
| `f5.presentacion` | La presentación está completa | `presentacion.md` tiene al menos dos commits y ya no contiene la palabra `COMPLETAR` |

Las reflexiones no son un checkpoint automático: las califica el profesor. El verificador sí revisa que el autor del commit coincida con el nombre del archivo.

## Errores comunes

- **Cerrar el issue de hallazgo a mano sin la Decisión «Corregir»** o marcar `corregida` una prueba que todavía falla. El gate de correcciones lo detecta.
- **Firmar por etiqueta** (`imagen:v1.0.0`) en vez de por digest. La etiqueta puede moverse; la firma debe ir sobre el contenido.
- **Nombre de imagen con mayúsculas.**
- **Release sin el SBOM adjunto** o con el SBOM con otro nombre. El nombre del archivo debe contener `sbom`.
- **Otro `sha256:` antes del de la imagen** en las notas del release, por ejemplo el checksum del SBOM.
- **Excepciones sin `statement`** o con un texto vacío que no justifica nada. La revisión docente lee cada una.
- **Olvidar fijar por hash las actions del release** o agregar el workflow sin `permissions:` arriba: `f3.fijadas` y `f3.permisos` vuelven a fallar.
- **Subir la reflexión de otra persona.** No cuenta para nadie.
- **Dejar `<!-- COMPLETAR … -->`** en la presentación.

## Pistas

<details markdown="block">
<summary>Pista 1 · Pasos 1 a 3 del workflow</summary>

{% raw %}
```yaml
      - uses: docker/login-action@<hash> # v4.x.y
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - id: meta
        uses: docker/metadata-action@<hash> # v6.x.y
        with:
          images: ghcr.io/${{ github.repository }}
          tags: type=semver,pattern={{version}}

      - id: build
        uses: docker/build-push-action@<hash> # v7.x.y
        with:
          context: app
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

      - id: imagen
        run: echo "ref=ghcr.io/${GITHUB_REPOSITORY,,}" >> "$GITHUB_OUTPUT"
```
{% endraw %}

{% raw %}Desde aquí, la imagen es `${{ steps.imagen.outputs.ref }}@${{ steps.build.outputs.digest }}`.{% endraw %}
</details>

<details markdown="block">
<summary>Pista 2 · Gate, firma y attestations</summary>

{% raw %}
```yaml
      - uses: aquasecurity/trivy-action@<hash> # v0.36.0
        with:
          image-ref: ${{ steps.imagen.outputs.ref }}@${{ steps.build.outputs.digest }}
          version: v0.76.1
          severity: HIGH,CRITICAL
          exit-code: "1"
          trivyignores: seguridad/fase-5/.trivyignore.yaml

      - run: cosign sign --yes "${{ steps.imagen.outputs.ref }}@${{ steps.build.outputs.digest }}"

      - uses: actions/attest-build-provenance@<hash> # v4.x.y
        with:
          subject-name: ${{ steps.imagen.outputs.ref }}
          subject-digest: ${{ steps.build.outputs.digest }}
          push-to-registry: true
```
{% endraw %}

Falta el `cosign-installer` antes de firmar, el SBOM y su attestation (`actions/attest-sbom` con `sbom-path`), el gate de correcciones, la verificación y el release.
</details>

<details markdown="block">
<summary>Pista 3 · Generar el esqueleto de la línea base</summary>

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.76.1 \
  image --severity HIGH,CRITICAL --format json --quiet lab-app:local > trivy.json
python3 - <<'PY'
import json
vistos = set()
for resultado in json.load(open("trivy.json")).get("Results", []):
    for v in resultado.get("Vulnerabilities") or []:
        if v["VulnerabilityID"] not in vistos:
            vistos.add(v["VulnerabilityID"])
            print(f'  - id: {v["VulnerabilityID"]}  # {v["PkgName"]} {v["InstalledVersion"]}, {v["Severity"]}')
PY
```

La salida trae solo los `- id:`. El `statement` de cada uno lo escribe el equipo después de decidir; mientras falte, `f5.lineabase` no pasa.
</details>

Anterior: [Fase 4](../fase-4/) · Ver también: [Herramientas](../herramientas/).
