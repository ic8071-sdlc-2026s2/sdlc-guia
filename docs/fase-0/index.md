---
title: Fase 0 · Arranque
nav_order: 2
---

# Fase 0 · Arranque
{: .no_toc }

Sesión 1, segunda mitad. Todo el equipo.
{: .fs-5 .fw-300 }

{: .importante }
**Objetivo.** Dejar listo el equipo, el repositorio, el Codespace y el primer pipeline en verde.
**Touchpoint de McGraw:** no aplica. Esta fase prepara el terreno para los siete touchpoints.

<details markdown="block">
<summary>Contenido de esta página</summary>
1. TOC
{:toc}
</details>

## Antes de empezar

- Una cuenta de GitHub por persona. Si es estudiante verificado en [GitHub Education](https://github.com/education), mejor: tendrá más horas de Codespaces y acceso a Copilot para el [bonus](../bonus-ia/).
- El número de equipo y la app que le tocó en el sorteo.
- El [Lab de misiones](https://ic8071-sdlc-2026s2.github.io/sdlc-guia/lab/) abierto en la misión 0: los guía en cada paso de esta fase.

## Pasos

### 1. Crear el repositorio del equipo

1. Una persona del equipo abre la plantilla de su app (el lab tiene el botón) y pulsa **Use this template → Create a new repository**: dueño, su propia cuenta; nombre `sdlc-<app>-equipo-<n>`, por ejemplo `sdlc-vampi-equipo-3`; visibilidad **Public**.
2. En **Settings → Collaborators → Add people** invita al resto del equipo. Cada persona acepta la invitación que le llega por correo o en https://github.com/notifications.
3. En **Settings → Pages**, en «Source», eligen **GitHub Actions**.
4. Pegan el enlace del repositorio en la misión 0 del lab.

### 2. Abrir el Codespace

En la página del repositorio: **Code → Codespaces → Create codespace on main**. La primera vez tarda unos minutos: construye el contenedor e instala Semgrep, pytest y requests. El Codespace trae Docker, `gh` y la extensión de Draw.io.

{: .advertencia }
VS Code avisa cuando la app abre su puerto. Déjelo **privado**. Un puerto público expondría una app vulnerable a internet con su nombre.

Si el Codespace no abre, use el plan B: presione la tecla `.` en la página del repositorio para editar en github.dev. Ahí no puede levantar la app, pero todos los workflows corren en Actions.

### 3. Levantar la app

```bash
bash lab/app.sh levantar   # construye la imagen desde app/ y la levanta
bash lab/app.sh rapido     # usa la imagen oficial del proyecto, si existe
bash lab/app.sh humo       # verifica que responde
bash lab/app.sh logs       # bitácora del contenedor
bash lab/app.sh detener    # la detiene y borra el contenedor
```

| App | Puerto | Comando recomendado | Primer recorrido |
|:---|:---|:---|:---|
| Juice Shop | 3000 | `rapido`: usa `bkimminich/juice-shop:v20.3.0`. Construir desde `app/` compila el frontend de Angular y en 2 núcleos tarda bastante más | Tienda en línea: registro, inicio de sesión, búsqueda, carrito, pedidos, reseñas, perfil |
| PyGoat | 8000 | `levantar` (no hay imagen oficial fijada) | Regístrese en `/register` e inicie sesión en `/login/`. Es una plataforma de lecciones: concéntrese en usuarios, autenticación, puntajes y administración |
| VAmPI | 5000 | `levantar` | No tiene interfaz web: es una API. Explórela con Swagger UI en `/ui/`. El script llama `/createdb` al arrancar para crear usuarios y libros de ejemplo |
| RailsGoat | 3000 | `levantar` | Portal de empleados: inicio de sesión, pagos, vacaciones, mensajes, beneficios, área de administración |

Si el puerto ya está ocupado: `APP_PUERTO_HOST=8081 bash lab/app.sh levantar`.

### 4. Recorrer la app 10 minutos

Úsela como una persona normal. Anote en un comentario del issue **Fase 0**:

- qué hace la app y quién la usa;
- qué roles existen (anónimo, cliente, administración);
- qué datos guarda y cuáles serían graves si se filtraran.

Esas notas son la materia prima del DFD y de los casos de abuso.

### 5. Completar `EQUIPO.md`

Una fila por integrante: nombre, usuario de GitHub y el rol de cada sesión. Nadie repite rol. En el equipo de 5, la quinta persona de cada sesión toma Relatoría. Haga commit y push a `main`.

```markdown
| Nombre | Usuario de GitHub | Sesión 2 (fases 1 y 2) | Sesión 3 (fase 3) | Sesión 4 (fase 4) | Sesión 5 (fase 5) |
|:---|:---|:---|:---|:---|:---|
| María Fernández | mfernandez | Arquitectura | AppSec | DevSecOps | Equipo rojo |
```

### 6. Activar GitHub Pages

**Settings → Pages → Build and deployment → Source: GitHub Actions**. No elija «Deploy from a branch». Luego vaya a **Actions → Checkpoints → Run workflow** para publicar el primer reporte. Su página queda en `https://{{ site.org }}.github.io/<repositorio>/` y el tablero de la clase lee de ahí el archivo `estado.json`.

### 7. Confirmar el pipeline

En **Actions**, el workflow «Fase 0 · CI» construye la imagen de la app, la levanta en el runner y hace una prueba de humo. Debe terminar en verde. La primera corrida es la más lenta porque llena la caché de Docker.

## Predicción con revelación

Antes de cada herramienta, el equipo escribe qué espera que pase y después compara. En la fase 0 la predicción es del laboratorio completo:

1. Después del recorrido, cada persona escribe en el issue **Fase 0** las tres debilidades que cree que tiene la app y qué técnica cree que las va a encontrar (ARA, SAST, SCA, secretos, DAST o pruebas).
2. No discutan todavía. Guarden el comentario.
3. En la fase 5, la sección 3 de la presentación compara esa predicción con lo que realmente encontró cada técnica.

El comentario queda con fecha y hora: esa es la evidencia de que la predicción fue antes.

## Checkpoint

| Identificador | Condición | Cómo se cumple |
|:---|:---|:---|
| `f0.equipo` | `EQUIPO.md` lista al menos 4 integrantes con usuario de GitHub | El verificador lee la tabla de `EQUIPO.md`. La columna «Usuario de GitHub» debe tener el usuario tal cual, con o sin `@`, sin enlaces ni espacios. Las filas que dicen `usuario` no cuentan |
| `f0.ci` | «Fase 0 · CI» terminó en verde en la rama principal | Al menos una corrida exitosa de `fase0-ci.yml` en `main` |
| `f0.pages` | GitHub Pages está activado | El repositorio tiene Pages activado. Use la fuente «GitHub Actions» para que el reporte se publique |

Cuando las tres condiciones se cumplen, el workflow cierra el issue **Fase 0** y abre **Fase 1**. Si no pasa nada, espere a que termine «Checkpoints» o córralo a mano desde Actions.

## Errores comunes

- **No aceptar la invitación.** Sin aceptarla no ve el botón de Codespaces ni puede editar. Revise el correo o las notificaciones de GitHub; si no llegó, avise al profesor para que la reenvíe.
- **Escribir el usuario como enlace** (`[mfernandez](https://github.com/mfernandez)`) o con el nombre completo. El verificador espera solo el usuario.
- **Elegir «Deploy from a branch» en Pages.** `f0.pages` pasa, pero el reporte no se publica y el tablero muestra al equipo «sin Pages».
- **Construir Juice Shop desde `app/` en el Codespace.** Funciona, pero tarda. Use `rapido` en el Codespace; el runner de Actions sí construye desde `app/`.
- **Dejar el Codespace encendido.** Consume horas de su cuota. Se detiene solo después de 30 minutos sin uso; deténgalo usted al terminar.

## Pistas

<details markdown="block">
<summary>Pista 1 · El issue «Fase 0» no aparece</summary>
El issue lo abre el workflow «Checkpoints» la primera vez que corre. Vaya a **Actions → Checkpoints → Run workflow**. Si el workflow falla, abra la corrida y lea el paso «Evaluar checkpoints».
</details>

<details markdown="block">
<summary>Pista 2 · «Fase 0 · CI» falla</summary>
Abra la corrida y busque el primer paso en rojo. Si falla «Levantar la app y esperar a que responda», lea la bitácora del último paso: casi siempre la app tardó más que `APP_ESPERA_SEGUNDOS` o se detuvo al arrancar. Vuelva a correr el workflow: la segunda vez usa la caché.
</details>

<details markdown="block">
<summary>Pista 3 · Todo está hecho pero el tablero no muestra al equipo</summary>
Revise tres cosas: que Pages use la fuente «GitHub Actions», que el job «publicar» de «Checkpoints» haya terminado en verde y que `https://{{ site.org }}.github.io/<repositorio>/estado.json` abra en el navegador. Si abre, el tablero lo leerá en la siguiente actualización, cada 60 segundos.
</details>

Siguiente: [Fase 1 · Casos de abuso y requerimientos](../fase-1/).
