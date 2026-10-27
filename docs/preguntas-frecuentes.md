---
title: Preguntas frecuentes
nav_order: 10
---

# Preguntas frecuentes
{: .no_toc }

<details markdown="block">
<summary>Contenido de esta página</summary>
1. TOC
{:toc}
</details>

## Organización y evaluación

### ¿Qué pasa si no terminamos una fase en clase?

Siga después. Cada fase se califica con el estado del repositorio en la fecha de cierre del curso. Las etiquetas `sesion-N` que el profesor pone al cierre de cada sesión solo registran el avance para el tablero.

### ¿Por qué el tablero muestra un porcentaje distinto al de nuestra página?

El tablero lee primero el `estado.json` de la página de su equipo, que se actualiza con cada corrida de «Checkpoints». Si su página no responde, usa el último resultado del verificador central, que corre una vez al día. La nota la calcula el verificador central con su propia copia de `checkpoints.py`.

### ¿Podemos modificar `lab/checkpoints.py` para que pase?

No. El verificador central no usa su copia, así que no cambiaría la nota, y modificar la infraestructura del laboratorio se trata como falta en la revisión docente. Si cree que un checkpoint está mal, avísele al profesor con un issue.

### ¿Podemos usar las soluciones publicadas de nuestra app?

Sí, citándolas en el issue o en la reflexión. Las cuatro apps publican sus vulnerabilidades; la nota premia trazar, automatizar y justificar, no descubrir. Lo que no puede hacer es copiar artefactos de otro equipo del curso.

### ¿Qué hace la quinta persona del equipo de 5?

Toma Relatoría en cada sesión, rotando como los demás roles: lleva la bitácora de decisiones y prepara la presentación final.

### Mi reflexión no aparece como mía

El verificador compara el nombre del archivo con el autor de los commits que lo tocaron. Revise tres cosas:

- el archivo se llama exactamente como su usuario de GitHub: `seguridad/reflexiones/mfernandez.md`;
- lo subió usted, desde su cuenta;
- el correo con que hizo el commit está asociado a su cuenta de GitHub. En el Codespace eso ya viene configurado. En su computadora, revise `git config user.email` y **Settings → Emails** en GitHub.

## Codespaces

### El Codespace no abre o tarda mucho

La primera vez construye el contenedor e instala las herramientas; puede tardar varios minutos. Si falla, borre el Codespace y cree otro. Si sigue fallando, use el plan B: tecla `.` en la página del repositorio para editar en github.dev, y deje que Actions construya y pruebe la app.

### ¿Cuántas horas de Codespaces tengo?

Depende de cómo el profesor activó Codespaces en la organización y de su plan personal; los estudiantes verificados en GitHub Education tienen una cuota mayor. Detenga el Codespace al terminar (**Codespaces → Stop**): se detiene solo tras 30 minutos sin uso, pero esos minutos cuentan.

### VS Code me ofrece hacer público el puerto de la app

No lo haga. La app es vulnerable a propósito. Use el puerto privado: solo usted lo ve, con su sesión de GitHub.

### `bash lab/app.sh levantar` dice que el puerto está ocupado

Hay otro contenedor usando el puerto. Deténgalo con `bash lab/app.sh detener` o cambie el puerto publicado: `APP_PUERTO_HOST=8081 bash lab/app.sh levantar`. Las pruebas leen la URL de `APP_URL`: `APP_URL=http://localhost:8081 pytest seguridad/fase-4/pruebas -v`.

### La app se detiene sola al arrancar

Vea la bitácora con `bash lab/app.sh logs`. En un Codespace de 2 núcleos, Juice Shop construida desde `app/` es la más pesada: use `bash lab/app.sh rapido`.

## Checkpoints e issues

### ¿Cómo sé qué me falta?

Tres lugares muestran lo mismo: el resumen de la última corrida de **Actions → Checkpoints**, la página de su equipo (`https://{{ site.org }}.github.io/<repositorio>/`) y el tablero. Cada condición trae un detalle, por ejemplo `4 casos` o `faltan: [7, 9]`.

### Hice todo y el issue de la fase no se cierra

«Checkpoints» corre en cada push a `main`, en cada cambio de issues y al terminar los demás workflows. Si nada de eso pasó desde su último cambio, córralo a mano: **Actions → Checkpoints → Run workflow**. Recuerde que el verificador lee la rama `main`: un cambio en una rama sin integrar no cuenta.

### Borré un issue por error

Los issues borrados desaparecen del verificador. Créelo de nuevo con el formulario y actualice los `#n` que lo citaban. En adelante, cierre en vez de borrar: los issues cerrados siguen contando.

### Mi requerimiento no cuenta para `f1.requerimientos`

Revise que el campo «Identificador ASVS 5.0» tenga capítulo, sección y número (`6.3.1` o `V6.3.1`) y que el issue se haya creado con el formulario «Requerimiento de seguridad», que pone la etiqueta `requerimiento`.

### No hay requerimiento de nivel 1 para mi caso

Pasa con las bitácoras de seguridad (todo el capítulo V16), el límite de peticiones (2.4.1), la cookie `HttpOnly` (3.3.4), la gestión de secretos (13.3.1) y la aleatoriedad de tokens (11.5.1). En esos casos se permite el de nivel 2; dígalo en el campo «Requerimiento».

### Edité el formulario de un issue y el verificador ya no lo lee

El verificador busca cada campo por el título de su sección (`### Categoría`, `### Trazabilidad …`). Al editar el cuerpo, cambie solo el contenido debajo de cada título.

## Workflows

### «Resource not accessible by integration»

Al job le falta un permiso. Desde la fase 3 cada job declara sus `permissions:`; agregue el mínimo que necesita ese paso, por ejemplo `security-events: write` para subir SARIF o `pull-requests: write` para que Dependency Review comente en el PR.

### «Unable to resolve action» después de fijar por hash

El hash no existe en ese repositorio. Lo más común es haber copiado el hash del objeto etiqueta de una etiqueta anotada. Vea [cómo obtener el hash](../fase-3/) en la fase 3.

### Scorecard falla con un error de verificación del workflow

Con `publish_results: true`, Scorecard exige que su workflow tenga la forma de la plantilla: solo actions aprobadas, permisos de lectura en el nivel superior y sin variables de entorno globales. Quite lo que haya agregado y deje solo los hashes.

### Dependabot abrió muchos PR

Es normal: las cuatro apps tienen dependencias viejas. Baje `open-pull-requests-limit` y cierre los que no vaya a integrar con un comentario. No los integre sin revisar: actualizar una dependencia puede romper la app.

### Las pruebas pasan en el Codespace y fallan en el runner

Revise que no dependan de datos que usted creó a mano en su Codespace, del orden de ejecución o de tiempos de espera cortos. En el runner la app arranca limpia en cada corrida.

### El gate de Trivy no pasa nunca

Es lo esperado sin línea base: las cuatro apps traen decenas de vulnerabilidades en sus dependencias. Lea la [fase 5](../fase-5/): actualice lo que pueda, justifique lo demás en `seguridad/fase-5/.trivyignore.yaml` y cite el hallazgo en cada excepción.

## Pages y tablero

### `estado.json` de mi página da 404

Revise que Pages use la fuente «GitHub Actions» y que el job «publicar» de «Checkpoints» haya terminado en verde. La primera publicación puede tardar unos minutos.

### La página de mi equipo muestra datos viejos

GitHub Pages guarda en caché las páginas unos minutos. Recargue sin caché (Ctrl+Shift+R) o espere. El tablero pide los datos sin caché.
