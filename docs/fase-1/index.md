---
title: Fase 1 · Casos de abuso y requerimientos
nav_order: 3
---

# Fase 1 · Casos de abuso y requerimientos
{: .no_toc }

Sesión 2, primera mitad. Lidera AppSec.
{: .fs-5 .fw-300 }

{: .importante }
**Objetivo.** Pensar como atacante antes de mirar el código: escribir casos de abuso concretos para su app y, por cada uno, el requerimiento de ASVS 5.0 que lo impediría. Ambos quedan como issues enlazados entre sí.
**Touchpoints de McGraw:** 5 (casos de abuso) y 6 (requerimientos de seguridad).

<details markdown="block">
<summary>Contenido de esta página</summary>
1. TOC
{:toc}
</details>

## Qué entrega

- Al menos **6 casos de abuso** con el formulario «Caso de abuso», que cubran al menos **5 categorías** distintas.
- Al menos **6 requerimientos** con el formulario «Requerimiento de seguridad», tomados de ASVS 5.0.
- Cada caso enlaza sus requerimientos (`#n`) y cada requerimiento enlaza los casos que mitiga.

Las categorías son las seis letras de STRIDE más dos: **S** suplantación, **T** manipulación, **R** repudio, **I** divulgación de información, **D** denegación de servicio, **E** elevación de privilegios, **cadena de suministro** y **configuración**.

## Pasos

### 1. Leer el mapa de la app

`seguridad/fase-1/mapa-de-la-app.md` resume los componentes, los roles y los datos de su app. Complételo con las notas del recorrido de la fase 0.

### 2. Jugar una ronda de Cornucopia

1. Abra [Copi](https://copi.owasp.org), cree un juego con la edición **Website App** y agregue a cada integrante. Cada persona recibe su mano por un enlace.
2. Por turnos, cada persona lee una carta y responde dos preguntas: ¿esto le puede pasar a nuestra app?, ¿dónde exactamente? Un «sí» con un endpoint, una pantalla o un archivo concreto es un candidato a caso de abuso.
3. Anote el código de la carta: palo y valor, por ejemplo `AZ3` o `VE10`. Los palos de la edición Website App son validación y codificación (VE), autenticación (AT), gestión de sesiones (SM), autorización (AZ), criptografía (CR) y la comodín Cornucopia (C).

{: .nota }
Si Copi pide cuenta o no carga desde la red del laboratorio, descargue el mazo en PDF desde el [proyecto OWASP Cornucopia](https://owasp.org/www-project-cornucopia/) y repártanlo en pantalla. Lo que importa es el código de la carta que inspiró cada caso.

Cornucopia cubre bien S, T, I y E. Para cadena de suministro, configuración y repudio piense además en: dependencias con vulnerabilidades conocidas, secretos en el código, cabeceras y cookies, y operaciones críticas que no dejan rastro.

### 3. Escribir los casos de abuso

Use **Issues → New issue → Caso de abuso**. Escriba desde el punto de vista de quien ataca.

| Campo | Qué poner |
|:---|:---|
| Actor malicioso | Quién ataca y con qué acceso empieza: anónimo, cliente registrado, empleado |
| Objetivo del atacante | El daño que busca, en una oración |
| Categoría | Una de las ocho. Reparta: necesita al menos 5 distintas |
| Carta de Cornucopia | El código de la carta, por ejemplo `AT4` |
| Escenario | Tres líneas: `Dado …`, `Cuando …`, `Entonces …` |
| Componente afectado | Endpoint, pantalla o archivo concreto |
| CWE | El CWE que mejor describe la debilidad |
| Requerimientos que lo mitigan | Déjelo vacío por ahora; lo llena en el paso 5 |

El escenario debe tener una línea por palabra clave, sin viñetas ni negritas, y al menos dos palabras después de cada palabra clave:

```text
Dado un cliente registrado con sesión iniciada y el número de pedido de otra persona
Cuando cambia el identificador del pedido en la URL de la API
Entonces obtiene la dirección y el teléfono de entrega de esa otra persona
```

### 4. Escribir los requerimientos

Use **Issues → New issue → Requerimiento de seguridad**. Busque el requerimiento en [ASVS 5.0](https://github.com/OWASP/ASVS/tree/master/5.0/en). Prefiera el **nivel 1**: son 70 de los 345 requerimientos.

Algunas debilidades no tienen requerimiento de nivel 1. En esos casos **se permite el de nivel 2**; dígalo en el campo «Requerimiento»:

| Debilidad | Requerimiento de nivel 2 |
|:---|:---|
| Bitácoras de seguridad (repudio) | Todo el capítulo V16, por ejemplo 16.3.1 |
| Límite de peticiones | 2.4.1 |
| Cookie de sesión sin `HttpOnly` | 3.3.4 |
| Gestión de secretos | 13.3.1 |
| Tokens predecibles (aleatoriedad) | 11.5.1 |

| Campo | Qué poner |
|:---|:---|
| Identificador ASVS 5.0 | Capítulo, sección y número, **sin la V inicial**: `8.2.2`, no `V8.2.2` |
| Capítulo ASVS | El capítulo del identificador |
| Requerimiento | El texto de ASVS aterrizado a su app, verificable |
| Criterios de aceptación | `Dado`, `Cuando`, `Entonces`, donde «Entonces» es el comportamiento seguro |
| Casos de abuso que mitiga | `#3, #5` |
| Cómo se verificará | Marque lo que aplique: regla de Semgrep, prueba pytest, ZAP, gate o revisión manual |

{: .advertencia }
El verificador busca un identificador con capítulo, sección y número: acepta `6.3.1` y `V6.3.1`, como aparece en el CSV de ASVS. Un identificador incompleto, como `6.3`, no cuenta para `f1.requerimientos`.

### 5. Enlazar en los dos sentidos

1. Cree primero los casos, luego los requerimientos citando los casos.
2. Edite cada caso: en la sección **Requerimientos que lo mitigan** del cuerpo del issue, reemplace `_No response_` por los números, por ejemplo `#12, #13`. No borre el título de la sección (`### Requerimientos que lo mitigan`): el verificador lo usa para encontrar el campo.
3. GitHub muestra el enlace cruzado en la línea de tiempo de cada issue. Revise que cada caso tenga al menos un requerimiento y cada requerimiento al menos un caso.

Las etiquetas `stride:*`, `cadena-suministro`, `configuracion` y `asvs:V*` las pone el workflow «Checkpoints» a partir del formulario. No hace falta ponerlas a mano.

## Predicción con revelación

Antes del juego de Cornucopia, cada persona escribe en el issue **Fase 1**, en dos minutos y sin hablar, los tres ataques que cree más probables contra la app. Después de escribir los casos de abuso, comparen:

- ¿Qué casos salieron de una carta que nadie había anticipado?
- ¿Qué ataque anticipado no tiene carta? ¿A qué categoría pertenece?
- ¿Qué categorías quedaron vacías y por qué?

Escriban la comparación como respuesta al comentario de la predicción.

## Checkpoint

| Identificador | Condición | Cómo se cumple |
|:---|:---|:---|
| `f1.casos` | Al menos 6 casos de abuso | Issues con la etiqueta `caso-de-abuso`, que el formulario pone sola. Abiertos o cerrados, cuentan igual |
| `f1.categorias` | Los casos cubren al menos 5 categorías | Valores distintos del campo «Categoría» |
| `f1.gherkin` | Cada caso tiene escenario Dado, Cuando, Entonces con contenido | Cada línea empieza con la palabra clave y le siguen al menos dos palabras. El valor por defecto del formulario (las tres palabras solas) no pasa |
| `f1.cornucopia` | Cada caso cita su carta de Cornucopia | El campo «Carta de Cornucopia» no está vacío |
| `f1.requerimientos` | Al menos 6 requerimientos con identificador ASVS 5.0 | Issues con la etiqueta `requerimiento` cuyo campo «Identificador» contiene un número con la forma `N.N.N` |
| `f1.trazabilidad` | Cada caso enlaza al menos un requerimiento existente | El campo «Requerimientos que lo mitigan» de cada caso cita con `#n` un issue que es requerimiento |

## Errores comunes

- **Crear el issue en blanco en vez de usar el formulario.** No lleva la etiqueta ni las secciones que lee el verificador. Ciérrelo y créelo de nuevo con el formulario.
- **Escribir `V8.2.2`.** Escriba `8.2.2`.
- **Escenario con viñetas o negritas** (`- Dado …`, `**Dado** …`). Cada línea debe empezar con la palabra clave.
- **Citar en «Requerimientos que lo mitigan» el número de otro caso de abuso** en vez del requerimiento.
- **Casos genéricos** («un atacante hace inyección SQL»). Nombre el endpoint, el parámetro y el dato que se pierde. Un caso concreto se convierte en prueba en la fase 4; uno genérico no.
- **Requerimientos que no se pueden verificar** («la app debe ser segura»). Si no puede escribir el «Entonces» de los criterios, no es un requerimiento.

## Pistas

<details markdown="block">
<summary>Pista 1 · No sé qué categorías cubrir</summary>
Empiece por las nueve debilidades del núcleo del curso: casi todas las apps tienen alguna versión. Inicio de sesión sin límite de intentos (S), inyección SQL (T), operaciones críticas sin bitácora (R), recurso ajeno por identificador (I), sin límite de peticiones (D), función de administración sin verificar rol (E), secreto en el código y dependencia vulnerable (cadena de suministro), cookie de sesión sin HttpOnly y cabeceras ausentes (configuración).
</details>

<details markdown="block">
<summary>Pista 2 · Dónde buscar en el código de mi app</summary>

| App | Dónde mirar |
|:---|:---|
| Juice Shop | `app/routes/` (un archivo por endpoint de la API), `app/lib/insecurity.ts`, `app/models/`, `app/server.ts` |
| PyGoat | `app/introduction/views.py`, `app/introduction/urls.py`, `app/pygoat/settings.py`, `app/challenge/`, `app/scoring/` |
| VAmPI | `app/openapi_specs/openapi3.yml`, `app/api_views/users.py`, `app/api_views/books.py`, `app/models/`, `app/config.py` |
| RailsGoat | `app/app/controllers/`, `app/app/models/`, `app/config/routes.rb`, `app/config/initializers/` |
</details>

<details markdown="block">
<summary>Pista 3 · Puntos de partida en ASVS 5.0</summary>
Estos identificadores están verificados contra el CSV oficial de ASVS 5.0.0. Son punto de partida: lea el texto completo en la fuente y aterrícelo a su app.

| Debilidad | CWE | Requerimiento | Nivel |
|:---|:---|:---|:---|
| Inicio de sesión sin límite de intentos | CWE-307 | 6.3.1 | 1 |
| Inyección SQL | CWE-89 | 1.2.4 | 1 |
| Inyección de comandos | CWE-78 | 1.2.5 | 1 |
| XSS | CWE-79 | 3.2.2 | 1 |
| Recurso ajeno por identificador (IDOR) | CWE-639 | 8.2.2 | 1 |
| Función de administración sin verificar rol | CWE-862 | 8.2.1 | 1 |
| La API devuelve más campos de los necesarios | CWE-213 | 15.3.1 | 1 |
| Token sin verificar firma | CWE-347 | 9.1.1 | 1 |
| Regla de negocio sin validar (montos, cantidades) | CWE-20 | 2.2.1 | 1 |
| Dependencia con vulnerabilidad conocida | CWE-1395 | 15.2.1 | 1 |
| Cookie sin `Secure` | CWE-614 | 3.3.1 | 1 |
| Sin cabecera HSTS | CWE-319 | 3.4.1 | 1 |
| Cookie de sesión sin `HttpOnly` | CWE-1004 | 3.3.4 | 2 |
| Sin límite de peticiones | CWE-770 | 2.4.1 | 2 |
| Secreto escrito en el código | CWE-798 | 13.3.1 | 2 |
| Operaciones de autenticación sin bitácora | CWE-778 | 16.3.1 | 2 |
| Token o código predecible | CWE-330 | 11.5.1 | 2 |

Los de nivel 2 de esta tabla están permitidos porque su debilidad no tiene requerimiento de nivel 1. Dígalo en el campo «Requerimiento».
</details>

Anterior: [Fase 0](../fase-0/) · Siguiente: [Fase 2 · Análisis de riesgo arquitectónico](../fase-2/).
