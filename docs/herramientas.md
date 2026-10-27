---
title: Herramientas
nav_order: 8
---

# Herramientas
{: .no_toc }

Una sección por herramienta: qué hace, en qué fase se usa, cómo la usa el laboratorio y qué **no** ve. Esa última columna es la que importa para la presentación final. Todas son gratuitas y corren en la web, en el Codespace o en el runner de Actions.
{: .fs-5 .fw-300 }

<details markdown="block">
<summary>Contenido de esta página</summary>
1. TOC
{:toc}
</details>

## Resumen

| Herramienta | Fase | Versión del curso | Técnica |
|:---|:---|:---|:---|
| OWASP Cornucopia y Copi | 1 | Edición Website App | Modelado de amenazas con cartas |
| OWASP ASVS | 1 | 5.0.0, nivel 1 | Catálogo de requerimientos verificables |
| diagrams.net | 2 | Web o extensión Draw.io | DFD con fronteras de confianza |
| Semgrep CE y Playground | 3 | 1.182.0 | SAST con reglas |
| Gitleaks | 3 | v8.30.1 | Búsqueda de secretos en el historial |
| Dependabot | 3 | Servicio de GitHub | SCA: alertas y PR de actualización |
| Dependency Review | 3 y 5 | action v5 | SCA en el PR |
| OpenSSF Scorecard | 3 | action v2.5.0 | Prácticas de seguridad del repositorio |
| ZAP | 4 | 2.17.0 (`stable`) | DAST |
| pytest | 4 y 5 | 9.1.1, con requests 2.34.2 | Pruebas desde casos de abuso |
| Trivy | 5 | 0.76.1 | Escaneo de imagen y SBOM |
| cosign | 5 | v3.1.4 | Firma de la imagen con Sigstore |
| Attestations de GitHub | 5 | `attest-build-provenance` v4, `attest-sbom` v4 | Procedencia SLSA y SBOM atestado |

## OWASP Cornucopia y Copi

Cornucopia es un mazo de cartas para descubrir amenazas en equipo. Cada carta describe un ataque concreto; quien la juega dice si aplica a la app y dónde. La edición **Website App** tiene los palos validación y codificación (VE), autenticación (AT), gestión de sesiones (SM), autorización (AZ), criptografía (CR) y Cornucopia (C). Copi es la versión en línea del juego: reparte las manos por enlace.

- **En el laboratorio:** cada caso de abuso cita la carta que lo inspiró (`f1.cornucopia`).
- **No ve:** la cadena de suministro ni la configuración del despliegue; complételas a mano.
- **Enlaces:** [proyecto OWASP Cornucopia](https://owasp.org/www-project-cornucopia/) · [Copi](https://copi.owasp.org)

## OWASP ASVS 5.0

El Application Security Verification Standard es un catálogo de 345 requerimientos verificables en 17 capítulos, con tres niveles. El nivel 1 (70 requerimientos) es el mínimo para cualquier app. Cada requerimiento tiene un identificador `capítulo.sección.número`, por ejemplo `8.2.2`.

- **En el laboratorio:** cada requerimiento es un issue con su identificador (sin la V inicial) y sus criterios Dado, Cuando, Entonces.
- **Cuidado:** algunas debilidades no tienen requerimiento de nivel 1: bitácoras (todo V16), límite de peticiones (2.4.1), `HttpOnly` (3.3.4), gestión de secretos (13.3.1) y aleatoriedad de tokens (11.5.1). Ahí se permite el de nivel 2.
- **Enlaces:** [OWASP ASVS en GitHub, carpeta 5.0](https://github.com/OWASP/ASVS/tree/master/5.0) · [proyecto OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/)

## diagrams.net

Editor de diagramas gratuito. El formato `.drawio.svg` es un SVG que GitHub muestra como imagen y que lleva el diagrama editable adentro.

- **En el laboratorio:** `seguridad/fase-2/dfd.drawio.svg`. En el Codespace, la extensión Draw.io lo abre directamente.
- **No ve:** nada por sí solo. El valor está en recorrer cada flujo que cruza una frontera con STRIDE.
- **Enlaces:** [app.diagrams.net](https://app.diagrams.net) · [extensión Draw.io para VS Code](https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio)

## Semgrep CE y Semgrep Playground

Semgrep busca patrones en el código fuente con reglas escritas en YAML que se parecen al código que buscan. La edición comunitaria (CE) es gratuita. El Playground permite escribir una regla y probarla contra un fragmento en el navegador.

- **En el laboratorio:** el job `sast` corre `p/default` más las reglas propias de `seguridad/fase-3/semgrep/`, y `semgrep --test` valida cada regla contra su archivo de prueba (`f3.regla`).
- **No ve:** fallas de diseño y de lógica de negocio (un IDOR es código correcto que consulta el registro equivocado), dependencias vulnerables, configuración del servidor en ejecución.
- **Enlaces:** [documentación](https://semgrep.dev/docs/) · [escribir reglas](https://semgrep.dev/docs/writing-rules/overview) · [probar reglas](https://semgrep.dev/docs/writing-rules/testing-rules) · [Playground](https://semgrep.dev/playground)

## Gitleaks

Busca secretos (llaves, tokens, contraseñas) en archivos y en todo el historial de git, con reglas de expresiones regulares y entropía.

- **En el laboratorio:** se corre la imagen `ghcr.io/gitleaks/gitleaks:v8.30.1` con el subcomando `git`, `--redact` y `--exit-code 0`. No se usa la action porque pide licencia para organizaciones.
- **No ve:** secretos que no tienen un formato reconocible, ni secretos en la imagen de contenedor (eso lo ve Trivy).
- **Enlaces:** [repositorio de Gitleaks](https://github.com/gitleaks/gitleaks)

## Dependabot

Servicio de GitHub con dos partes: las **alertas** avisan cuando una dependencia tiene una vulnerabilidad publicada en la GitHub Advisory Database, y las **actualizaciones de versión** abren PR según `.github/dependabot.yml`.

- **En el laboratorio:** `dependabot.yml` vigila `app/` y las actions (`f3.dependabot`). Las alertas aparecen en **Security → Dependabot**.
- **No ve:** si el código vulnerable de la dependencia se usa de verdad, ni vulnerabilidades del código propio.
- **Enlaces:** [documentación de Dependabot](https://docs.github.com/es/code-security/dependabot) · [GitHub Advisory Database](https://github.com/advisories)

## Dependency Review

Action que compara las dependencias de un PR contra las de la rama base y falla si el PR agrega una dependencia con vulnerabilidad de la severidad indicada.

- **En el laboratorio:** fase 3 con `warn-only: true` (solo reporta); fase 5 sin `warn-only` y como check requerido del ruleset (bloquea).
- **No ve:** las vulnerabilidades que ya estaban antes del PR: solo mira lo que el PR cambia.
- **Enlaces:** [actions/dependency-review-action](https://github.com/actions/dependency-review-action) · [documentación](https://docs.github.com/es/code-security/supply-chain-security/understanding-your-software-supply-chain/about-dependency-review)

## OpenSSF Scorecard

Califica de 0 a 10 prácticas de seguridad del repositorio: protección de rama, actions fijadas, permisos de los tokens, revisión de código, dependencias actualizadas y otras.

- **En el laboratorio:** `lab/workflows/scorecard.yml`, copiado y fijado por hash (`f3.scorecard`). El resultado queda en **Security → Code scanning**.
- **No ve:** la seguridad de la app; mide el proceso alrededor del código.
- **Enlaces:** [scorecard.dev](https://scorecard.dev) · [ossf/scorecard-action](https://github.com/ossf/scorecard-action)

## ZAP

Escáner dinámico que nació como OWASP ZAP y hoy se publica como ZAP by Checkmarx, con licencia Apache 2.0. Ataca la app corriendo desde afuera, como un navegador o un cliente de la API.

- **Baseline** (`zap-baseline.py`): spider de un minuto y análisis pasivo. Juice Shop, PyGoat y RailsGoat.
- **API scan** (`zap-api-scan.py -f openapi`): importa la especificación OpenAPI y hace análisis activo. VAmPI.
- **En el laboratorio:** imagen `ghcr.io/zaproxy/zaproxy:stable` en el job de la fase 4, con `-I` para que solo reporte.
- **No ve:** reglas de negocio (de quién es cada recurso), lo que está detrás del inicio de sesión si no se le configura autenticación, el código y las dependencias que no se reflejan en respuestas HTTP.
- **Enlaces:** [zaproxy.org](https://www.zaproxy.org) · [baseline scan](https://www.zaproxy.org/docs/docker/baseline-scan/) · [API scan](https://www.zaproxy.org/docs/docker/api-scan/)

## pytest

Marco de pruebas de Python. Con `requests`, permite escribir pruebas que atacan la app por HTTP.

- **En el laboratorio:** `seguridad/fase-4/pruebas/`, con `@pytest.mark.caso_abuso(N)` para la trazabilidad (`f4.pruebas`) y `@pytest.mark.corregida` para el gate de la fase 5 (`f5.correcciones`). `pytest.ini` usa `--strict-markers`.
- **No ve:** lo que nadie pensó escribir. Una prueba solo encuentra lo que su caso de abuso describe.
- **Enlaces:** [documentación de pytest](https://docs.pytest.org) · [marcadores](https://docs.pytest.org/en/stable/how-to/mark.html) · [requests](https://requests.readthedocs.io)

## Trivy

Escáner de Aqua Security para imágenes de contenedor, sistemas de archivos y configuración. Encuentra vulnerabilidades en paquetes del sistema operativo y de las dependencias, secretos dentro de la imagen, y genera SBOM en CycloneDX o SPDX.

- **En el laboratorio:** gate de la fase 5 con `severity: HIGH,CRITICAL`, `exit-code: "1"`, `version: v0.76.1` (la action instala una versión anterior si no se indica) y la línea base `seguridad/fase-5/.trivyignore.yaml`; SBOM `sbom.cdx.json`.
- **Juice Shop:** el código original no trae `package-lock.json`, así que Trivy sobre `app/` no ve las dependencias npm; sobre la imagen sí.
- **No ve:** el código propio de la app ni su lógica.
- **Enlaces:** [documentación de Trivy](https://trivy.dev) · [archivo de exclusiones](https://trivy.dev/latest/docs/configuration/filtering/) · [aquasecurity/trivy-action](https://github.com/aquasecurity/trivy-action)

## cosign

Herramienta de Sigstore para firmar y verificar imágenes de contenedor. En Actions firma **sin llaves**: pide un certificado de corta duración a Fulcio con la identidad OIDC del workflow y registra la firma en el log público Rekor.

- **En el laboratorio:** `cosign sign --yes imagen@digest` en el release. `sigstore/cosign-installer` v4.1.2 instala cosign v3.0.6 si no se le pide otra versión; pase `cosign-release: v3.1.4` para la del curso. Para verificar se usa `cosign verify` con `--certificate-identity-regexp` y `--certificate-oidc-issuer` para comprobar quién firmó.
- **No ve:** si la imagen es segura. Una firma prueba origen e integridad, no ausencia de vulnerabilidades.
- **Enlaces:** [documentación de Sigstore](https://docs.sigstore.dev) · [cosign](https://github.com/sigstore/cosign)

## Attestations de GitHub

Una attestation es una declaración firmada sobre un artefacto: con qué workflow, desde qué repositorio y qué commit se construyó (procedencia SLSA), o qué componentes contiene (SBOM). GitHub las guarda y las verifica con `gh attestation verify`.

- **En el laboratorio:** `actions/attest-build-provenance` y `actions/attest-sbom` con `push-to-registry: true`. `f5.attestation` consulta la API de attestations con el digest del release.
- **No ve:** lo mismo que cosign: prueba de dónde viene la imagen, no qué tan segura es.
- **Enlaces:** [documentación de artifact attestations](https://docs.github.com/es/actions/security-for-github-actions/using-artifact-attestations) · [SLSA](https://slsa.dev)
