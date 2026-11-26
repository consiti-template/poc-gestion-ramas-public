# PoC - Gestión de ramas, Pull Requests y promoción de versiones

## 1. Descripción del proyecto

Esta prueba de concepto (PoC) tiene como objetivo evaluar y demostrar una estrategia estandarizada de gestión de ramas, Pull Requests e integración de cambios para un monorepositorio.

La propuesta busca facilitar el desarrollo concurrente, mantener la estabilidad de los ambientes y controlar la promoción selectiva de funcionalidades entre las ramas `dev`, `test` y `main`.

Para validar el flujo de trabajo se utilizó GitHub como plataforma de gestión del repositorio, Pull Requests e Issues, simulando diferentes situaciones que pueden presentarse durante el desarrollo de un proyecto de software.

## 2. Objetivos

- Establecer una estructura de ramas que permita separar el desarrollo, las pruebas y las versiones estables.
- Definir convenciones para nombrar ramas, commits y Pull Requests.
- Controlar la integración de cambios mediante Pull Requests.
- Evaluar diferentes estrategias de merge según el tipo de integración.
- Demostrar la promoción selectiva de funcionalidades mediante `cherry-pick`.
- Simular la atención de errores detectados durante QA y su posterior backport.
- Validar la protección de la rama principal mediante revisiones y aprobaciones.

## 3. Estrategia de ramas

Se propone un modelo híbrido que combina prácticas de GitHub Flow con ramas de ambiente y ramas de release.

### 3.1 Ramas principales

| Rama | Propósito |
|---|---|
| `dev` | Integración de funcionalidades y correcciones durante el desarrollo. |
| `test` | Validación de cambios antes de su promoción a producción. |
| `main` | Contiene la versión estable del proyecto. |

### 3.2 Ramas auxiliares

| Tipo de rama | Propósito | Ejemplo |
|---|---|---|
| `feature/*` | Desarrollo de nuevas funcionalidades. | `feature/POC-002-productos` |
| `bugfix/*` | Corrección de errores detectados durante el desarrollo o QA. | `bugfix/POC-005-corregir-error` |
| `release/*` | Preparación y promoción controlada de versiones. | `release/1.0.1` |
| `backport/*` | Incorporación selectiva de correcciones a otras ramas. | `backport/POC-005-corregir-error` |

Las ramas auxiliares se crean a partir de la rama que corresponda según el flujo de trabajo y se integran mediante Pull Requests cuando aplica.

## 4. Convenciones de nomenclatura

Para mantener un historial organizado y facilitar la identificación de los cambios, se establecieron las siguientes convenciones.

### 4.1 Nombres de ramas

Se utiliza la estructura:

`<tipo>/<identificador>-<descripcion>`

Ejemplos:

- `feature/POC-002-productos`
- `bugfix/POC-005-corregir-error`
- `backport/POC-005-corregir-error`
- `release/1.0.1`

### 4.2 Mensajes de commit

Se utiliza una estructura basada en Conventional Commits:

`tipo(alcance): descripción [POC-XXX]`

Ejemplos:

- `feat(productos): agregar modulo de productos [POC-002]`
- `fix(productos): corregir precio de teclado [POC-005]`
- `chore(release): promover version 1.0.1 a test`

### 4.3 Títulos de Pull Requests

Los Pull Requests siguen una estructura similar a los commits:

`tipo(alcance): descripción [POC-XXX]`

Esto permite relacionar los cambios con sus requerimientos y facilitar su seguimiento.

### 4.4 Descripción de Pull Requests

Se estableció una plantilla común para documentar las integraciones:

1. Ticket relacionado.
2. Descripción de los cambios.
3. Issue que cierra (si aplica).
4. Pull Request relacionado (si aplica).

## 5. Flujo de trabajo

### 5.1 Desarrollo de funcionalidades

1. Obtener la última versión de la rama `dev`.
2. Crear una rama `feature/*` a partir de `dev`.
3. Realizar los cambios correspondientes.
4. Registrar los cambios mediante commits con la nomenclatura establecida.
5. Publicar la rama en GitHub.
6. Crear un Pull Request hacia `dev`.
7. Revisar los cambios y completar la integración.

### 5.2 Corrección de errores

Cuando se detecta un error durante QA:

1. Crear una rama `bugfix/*` a partir de la rama correspondiente a la versión afectada.
2. Realizar y registrar la corrección.
3. Crear un Pull Request hacia la rama de release.
4. Revisar e integrar la corrección.
5. Promover la actualización hacia `test`.
6. Realizar un backport hacia `dev` cuando corresponda, utilizando `cherry-pick -x` para conservar la trazabilidad del cambio original.

### 5.3 Preparación de versiones

Las ramas `release/*` se crean a partir de `main`.

Cuando se necesita preparar una versión con funcionalidades específicas, se utiliza `git cherry-pick -x` para incorporar únicamente los commits aprobados.

Esto permite excluir funcionalidades que todavía no están listas o que fueron rechazadas durante QA.

### 5.4 Promoción a QA

La rama de release se integra en `test` mediante un Pull Request.

La integración utiliza Merge Commit para conservar la referencia de la promoción de la versión.

Una vez integrada, se verifica que los archivos y correcciones esperados estén presentes y que las funcionalidades rechazadas no formen parte de la versión candidata.

### 5.5 Promoción a producción

Después de completar la validación en QA:

1. Crear un Pull Request desde `release/*` hacia `main`.
2. Solicitar las revisiones y aprobaciones correspondientes.
3. Verificar los cambios incluidos.
4. Integrar mediante Merge Commit.

La rama `main` conserva así el historial de las promociones realizadas.

## 6. Estrategias de integración

Se utilizan diferentes estrategias de merge según el propósito de cada Pull Request.

| Integración | Estrategia | Justificación |
|---|---|---|
| `feature/*` → `dev` | Squash and Merge | Agrupa los cambios de una funcionalidad en un solo commit. |
| `bugfix/*` → `release/*` | Squash and Merge | Mantiene la corrección como una unidad identificable. |
| `backport/*` → `dev` | Squash and Merge | Integra la corrección sin duplicar el historial original. |
| `release/*` → `test` | Merge Commit | Conserva el registro de la promoción a QA. |
| `release/*` → `main` | Merge Commit | Conserva el registro de la promoción a producción. |
| Reversión mediante Pull Request | Squash and Merge | Integra el retiro de una funcionalidad sin reescribir el historial. |

### 6.1 Uso de cherry-pick

Se utiliza el comando:

`git cherry-pick -x <commit>`

La opción `-x` agrega una referencia al commit original en el mensaje del nuevo commit.

Esto permite identificar el origen de los cambios cuando se realizan promociones selectivas o backports entre ramas con historiales diferentes.

## 7. Protección de ramas

Se configuraron reglas de protección en GitHub para controlar las integraciones.

### 7.1 Configuración aplicada en la PoC

| Regla | `dev` | `test` | `main` |
|---|---|---|---|
| Requerir Pull Request | Sí | Sí | Sí |
| Aprobaciones obligatorias | 0 | 0 | 1 |
| Descartar aprobaciones obsoletas | No configurado | No configurado | Sí |
| Bloquear force push | Sí | Sí | Sí |
| Restringir eliminación de ramas | Sí | Sí | Sí |

La rama `main` requiere al menos una aprobación de un usuario con permisos de escritura antes de permitir la integración.

### 7.2 Recomendaciones de configuración

Como mejora para una implementación real, se recomienda:

- Exigir al menos una aprobación para `dev` y `test`.
- Exigir dos aprobaciones para `main`, cuando la disponibilidad de revisores lo permita.
- Configurar verificaciones automáticas obligatorias antes de permitir merges.
- Mantener restringidos los force pushes y la eliminación de ramas protegidas.
- Revisar periódicamente las reglas de protección.

Estas recomendaciones forman parte de la propuesta y no deben confundirse con las configuraciones efectivamente aplicadas durante la PoC.

## 8. Escenarios de prueba realizados

Se ejecutaron cinco escenarios para evaluar el comportamiento de la estrategia propuesta.

### Escenario A. Desarrollo concurrente

**Objetivo:** demostrar la integración de funcionalidades desarrolladas en paralelo.

**Actividades realizadas:**

- Se crearon ramas independientes para los módulos de productos y categorías.
- Se generaron Pull Requests hacia `dev`.
- Se integraron los cambios en un orden diferente al de su desarrollo.
- Se actualizó la rama de productos mediante rebase para incorporar los cambios más recientes de `dev`.
- Se verificó que cada Pull Request incluyera únicamente sus archivos correspondientes.

**Resultado:** se logró integrar el trabajo concurrente sin perder los cambios de ninguna funcionalidad.

### Escenario B. División del trabajo

**Objetivo:** demostrar que un mismo requerimiento puede dividirse en varios Pull Requests independientes.

**Actividades realizadas:**

- Se utilizó el requerimiento POC-004 relacionado con el módulo de usuarios.
- Se dividió el trabajo en documentación de estructura y documentación del listado de usuarios.
- Se crearon dos ramas independientes.
- Se abrieron dos Pull Requests asociados al mismo requerimiento.
- Se integraron ambos cambios en `dev`.

**Resultado:** se comprobó que un requerimiento puede gestionarse mediante varios Pull Requests, facilitando la revisión y la integración de cambios independientes.

### Escenario C. Promoción selectiva a QA

**Objetivo:** demostrar que una versión puede prepararse incorporando únicamente funcionalidades seleccionadas.

**Actividades realizadas:**

- Se creó la rama `release/1.0.0` a partir de `main`.
- Se seleccionaron los commits correspondientes a POC-001, POC-002 y POC-003.
- Se incorporaron mediante `cherry-pick -x`.
- Se creó un Pull Request hacia `test`.
- Se integró mediante Merge Commit.

**Resultado:** se promovieron los módulos de usuarios, categorías y productos a QA, dejando fuera POC-004.

### Escenario D. Corrección de errores y backport

**Objetivo:** demostrar el tratamiento de una corrección detectada durante QA y su incorporación posterior a desarrollo.

**Actividades realizadas:**

- Se identificó una inconsistencia en el precio del teclado.
- Se creó el requerimiento POC-005.
- Se creó una rama `bugfix/*` desde `release/1.0.0`.
- Se corrigió el precio de `25.00` a `30.00`.
- Se integró la corrección mediante Pull Request y Squash and Merge.
- Se promovió la actualización a `test` mediante Merge Commit.
- Se creó una rama `backport/*` desde `dev`.
- Se utilizó `cherry-pick -x` para incorporar la corrección a desarrollo.
- Se integró el backport mediante Pull Request.

**Resultado:** la corrección quedó incorporada tanto en la versión de QA como en desarrollo, manteniendo la trazabilidad del cambio original.

### Escenario E. Promoción parcial a producción

**Objetivo:** demostrar la exclusión de una funcionalidad rechazada durante QA y la promoción de una versión aprobada a producción.

**Actividades realizadas:**

- Se incorporó POC-004 a `test` para simular su recepción durante QA.
- Se simuló el rechazo de la funcionalidad.
- Se creó la rama `release/1.0.1` a partir de `main`.
- Se seleccionaron los cambios aprobados correspondientes a POC-001, POC-002, POC-003 y POC-005.
- Se incorporaron mediante `cherry-pick -x`, excluyendo POC-004.
- Se creó un Pull Request para retirar POC-004 de `test`, utilizando `git revert` y Squash and Merge.
- Se promovió `release/1.0.1` hacia `test` mediante Merge Commit.
- Se verificó la presencia de los tres módulos y la corrección del precio del teclado a `30.00`.
- Se creó un Pull Request desde `release/1.0.1` hacia `main`.
- Se obtuvo la aprobación requerida de un revisor con permisos de escritura.
- Se integró mediante Merge Commit.

**Resultado:** se demostró la promoción controlada de una versión aprobada, excluyendo la funcionalidad rechazada y conservando la trazabilidad de los cambios.

## 9. Limitaciones de la prueba de concepto

La PoC se enfocó en validar el flujo de ramas e integración mediante GitHub.

Por lo tanto, presenta las siguientes limitaciones:

- Se utilizó un repositorio público de demostración con archivos de documentación como representación de funcionalidades.
- Los Issues de GitHub se utilizaron para simular la gestión de requerimientos.
- No se configuraron flujos de integración continua ni verificaciones automáticas obligatorias.
- No se realizaron despliegues reales hacia ambientes de desarrollo, QA o producción.
- La validación de los escenarios se realizó mediante historial de commits, Pull Requests y revisión de archivos modificados.
- La aprobación de Pull Requests se comprobó mediante las reglas de protección configuradas en GitHub.

## 10. Conclusiones

La prueba de concepto permitió evaluar una estrategia híbrida de gestión de ramas que combina el desarrollo mediante ramas de corta duración, la separación de ambientes y la preparación controlada de versiones.

Los cinco escenarios demostraron la viabilidad de:

- Integrar funcionalidades desarrolladas concurrentemente.
- Dividir requerimientos en cambios independientes.
- Promover selectivamente funcionalidades entre ramas.
- Gestionar correcciones detectadas durante QA y realizar backports.
- Excluir funcionalidades rechazadas y promover versiones aprobadas mediante Pull Requests.

La estrategia propuesta proporciona una base organizada para controlar el flujo de cambios, mantener la estabilidad de las ramas principales y mejorar la trazabilidad de las integraciones.

---

## 11. Documentación relacionada

La propuesta técnica, el diagnóstico y la investigación que dieron origen a esta prueba de concepto se encuentran documentados en un repositorio independiente.

Este repositorio contiene la implementación práctica de los escenarios utilizados para validar la estrategia propuesta.

## 12. Tecnologías y herramientas

- Git
- GitHub
- GitHub Pull Requests
- GitHub Issues
- Git Bash / PowerShell

## 13. Alcance

Este repositorio tiene fines demostrativos y académicos. Su contenido representa una prueba de concepto para evaluar y documentar una estrategia de gestión de ramas, integración de cambios y promoción de versiones en un monorepositorio.

