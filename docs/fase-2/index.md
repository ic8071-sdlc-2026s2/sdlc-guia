---
title: Fase 2 · Análisis de riesgo arquitectónico
nav_order: 4
---

# Fase 2 · Análisis de riesgo arquitectónico
{: .no_toc }

Sesión 2, segunda mitad. Lidera Arquitectura.
{: .fs-5 .fw-300 }

{: .importante }
**Objetivo.** Dibujar el diagrama de flujo de datos (DFD) de la app con sus fronteras de confianza, recorrer con STRIDE cada flujo que cruza una frontera y priorizar los riesgos en una matriz 3 por 3.
**Touchpoint de McGraw:** 2 (análisis de riesgo arquitectónico).

<details markdown="block">
<summary>Contenido de esta página</summary>
1. TOC
{:toc}
</details>

## Qué entrega

- `seguridad/fase-2/dfd.drawio.svg` editado, con al menos una frontera de confianza rotulada.
- `seguridad/fase-2/matriz-riesgos.md` con al menos 6 riesgos, cada uno con su caso de abuso, y todos los casos de la fase 1 en la matriz.

## Pasos

### 1. Abrir el DFD

Tiene dos opciones. Las dos guardan el archivo en el mismo formato `.drawio.svg`: un SVG que se ve en GitHub y que lleva el diagrama editable adentro.

- **En el Codespace.** Abra `seguridad/fase-2/dfd.drawio.svg`: la extensión Draw.io lo abre como diagrama editable. Guarde con Ctrl+S.
- **En el navegador.** Abra [app.diagrams.net](https://app.diagrams.net), **Open Existing Diagram → GitHub** o desde su equipo, edite y guarde con el mismo nombre. Si lo exporta, elija **SVG** con la opción **Include a copy of my diagram** activada.

### 2. Dibujar el DFD

Parta de la plantilla, que ya trae tres elementos, y del mapa de la app.

| Elemento | Símbolo | Ejemplos |
|:---|:---|:---|
| Entidad externa | Rectángulo | Cliente en el navegador, persona administradora, servicio de correo, cliente de la API |
| Proceso | Círculo | API REST, servidor web, autenticación, módulo de pagos |
| Almacén de datos | Dos líneas paralelas | Base SQLite, carpeta de archivos subidos, bitácora |
| Flujo de datos | Flecha con rótulo | «credenciales», «token JWT», «pedido», «archivo subido» |
| Frontera de confianza | Rectángulo punteado rotulado | «Frontera de confianza: internet / servidor» |

Rotule cada frontera con el texto **«Frontera de confianza: …»**. Un buen DFD de estas apps tiene entre 8 y 15 elementos. Más que eso suele ser ruido.

| App | Qué no puede faltar |
|:---|:---|
| Juice Shop | El frontend Angular corre en el navegador del usuario: está del lado no confiable. La API Express, la base SQLite, la colección de reseñas en MarsDB, la carga de archivos y las rutas de administración |
| PyGoat | El servidor Django, la base SQLite, el sitio de administración de Django, el registro y la autenticación, los laboratorios y el puntaje |
| VAmPI | El cliente de la API (no hay navegador), la emisión y verificación del token JWT, los endpoints de usuarios y de libros, `/createdb` y la base SQLite |
| RailsGoat | El navegador, los controladores Rails, la API en `/api`, el área de administración, la carga de archivos de beneficios y la base SQLite |

### 3. Recorrer las fronteras con STRIDE

Por cada flujo que cruza una frontera, pregunte qué amenazas de STRIDE aplican. Esta tabla, de STRIDE por elemento, dice cuáles revisar primero:

| Elemento | S | T | R | I | D | E |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|
| Entidad externa | X | | X | | | |
| Proceso | X | X | X | X | X | X |
| Almacén de datos | | X | X | X | X | |
| Flujo de datos | | X | | X | X | |

Cada amenaza que no estaba en la fase 1 puede volverse un caso de abuso nuevo. Créelo con el formulario: más casos bien trazados suman en la revisión docente.

### 4. Llenar la matriz de riesgos

En `seguridad/fase-2/matriz-riesgos.md`, una fila por riesgo:

| Columna | Qué poner |
|:---|:---|
| ID | `R-01`, `R-02`, … |
| Amenaza | Qué pasa, en una oración |
| STRIDE | La letra o la categoría |
| Caso de abuso | `#n` del issue. Puede citar varios |
| Probabilidad | 1, 2 o 3, según la escala del archivo |
| Impacto | 1, 2 o 3 |
| Riesgo | P × I, calculado: 1, 2, 3, 4, 6 o 9 |
| Tratamiento | Mitigar con `#req`, aceptar con justificación, transferir o evitar |

Después ubique cada ID en el mapa de calor del mismo archivo. Los riesgos de 6 y 9 son los candidatos a corregir en la fase 5.

## Predicción con revelación

Antes de dibujar, Arquitectura escribe en el issue **Fase 2**:

1. cuántas fronteras de confianza cree que tiene la app y dónde;
2. cuál será el riesgo más alto de la matriz.

Al terminar, compare. Es común descubrir una frontera que nadie vio, como el navegador en una SPA o un servicio que la app llama por detrás.

## Checkpoint

| Identificador | Condición | Cómo se cumple |
|:---|:---|:---|
| `f2.dfd` | El DFD fue editado e incluye al menos una frontera de confianza | `seguridad/fase-2/dfd.drawio.svg` tiene al menos dos commits en `main` (el de la plantilla y el suyo) y su contenido incluye la palabra «frontera» |
| `f2.matriz` | La matriz tiene al menos 6 riesgos con P × I correcto y caso de abuso | Filas de la primera tabla de `matriz-riesgos.md` con P e I entre 1 y 3, Riesgo igual a P × I y al menos un `#n` en «Caso de abuso» |
| `f2.cobertura` | Todos los casos de abuso aparecen en la matriz | Cada issue con etiqueta `caso-de-abuso` está citado en alguna fila válida |

## Errores comunes

- **Cambiar el nombre o el formato del DFD.** Debe seguir siendo `seguridad/fase-2/dfd.drawio.svg`. Un PNG o un `.drawio` sin SVG no cuentan.
- **Rotular en inglés** («Trust boundary»). El verificador busca «frontera».
- **Escribir la probabilidad como palabra** («Alta»). Use el número. Puede agregar la palabra después: `3 (alta)`.
- **Calcular mal el riesgo** o dejarlo vacío. El verificador recalcula P × I.
- **Poner otra tabla antes de la matriz** o cambiar los encabezados. El verificador toma la primera tabla del archivo y busca las columnas por su nombre.
- **Dejar un caso de abuso fuera** porque «no es importante». Si no es importante, va con P o I bajos, pero va.
- **Editar en una rama sin integrar.** El verificador lee `main`.

## Pistas

<details markdown="block">
<summary>Pista 1 · No sé dónde poner las fronteras</summary>
Una frontera separa zonas donde cambia el nivel de confianza: internet y su servidor, el usuario anónimo y el autenticado, la app y la base de datos, la app y un servicio externo. Si un dato cruza de una zona menos confiable a una más confiable, ahí hay que validar.
</details>

<details markdown="block">
<summary>Pista 2 · Cómo decidir la probabilidad</summary>
Pregúntese quién puede hacerlo. Cualquier visitante anónimo desde el navegador: 3. Un usuario registrado con herramientas comunes, como las de desarrollo del navegador o curl: 2. Alguien con acceso privilegiado o una cadena de varios pasos: 1.
</details>

<details markdown="block">
<summary>Pista 3 · Una fila válida</summary>

```markdown
| ID | Amenaza | STRIDE | Caso de abuso | Probabilidad | Impacto | Riesgo | Tratamiento |
|:---|:---|:---|:---|:---|:---|:---|:---|
| R-01 | Un cliente lee pedidos ajenos cambiando el identificador | I | #4 | 2 | 3 | 6 | Mitigar con #11 |
```
</details>

Anterior: [Fase 1](../fase-1/) · Siguiente: [Fase 3 · Análisis estático y cadena de suministro](../fase-3/).
