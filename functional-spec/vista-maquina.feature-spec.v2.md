---
artifact: feature-spec
schema_version: 1.0
feature: vista-maquina
version: 2

agent:
  name: functional-analyst
  version: 1.0
  mode: revision-consolidation

source:
  legacy_map: legacy-map/vista-maquina.legacy-map.md
  feature_spec_v1: functional-spec/vista-maquina.feature-spec.md
  human_review: human-review/vista-maquina.functional-review.yaml

status: APPROVED_WITH_OPEN_ITEMS
architecture_can_start: true
human_validation_required: true
remaining_pending_items: 1
pending_items:
  - id: HV-016
    item: formato visual de las fechas de mantenimiento e instalación
    blocking: false

human_decisions:
  processed: 19
  confirmed: 16
  confirmed_with_change: 2
  rejected: 0
  pending: 1

generated_at: 2026-09-28
---

# Feature Specification V2 — Consulta de máquina y registro de instalación de repuestos/insumos

## 1. Purpose

Permitir a un usuario autenticado localizar una máquina mediante su serial único y consultar en un único punto sus datos generales, la información de su contrato activo y los históricos asociados.

Permitir además registrar la instalación de un repuesto/insumo en la máquina consultada, dejando constancia de la fecha, la lectura del contador, el usuario solicitante y una nota opcional.

La consulta integral de la máquina y el registro de instalaciones se mantienen como capacidades funcionales diferenciadas. La autorización diferenciada por perfiles queda fuera del alcance de esta versión; sólo se exige autenticación.

---

## 2. Actors

- `Usuario autenticado`
  - Description: Persona con sesión iniciada que puede consultar máquinas y registrar instalaciones. Esta versión no diferencia permisos por perfil.
  - Status: CONFIRMED
  - Human source: `HV-013`

- `Usuario solicitante`
  - Description: Persona que solicitó la instalación. No interactúa necesariamente con el sistema; se registra como texto libre obligatorio informado por el usuario autenticado.
  - Status: CONFIRMED
  - Human source: `HV-011`

---

## 3. Functional capabilities

### CAP-001 — Consultar máquina por serial

- Type: PRIMARY
- Description: Localizar una máquina mediante su serial único y mostrar sus datos generales.
- Source: Feature Spec V1, `CAP-001`; Human Review `HV-001`

### CAP-002 — Asistencia en la localización del serial

- Type: SUPPORTING
- Description: Sugerir hasta 12 seriales existentes que comiencen por el texto ingresado, sin distinguir mayúsculas y minúsculas.
- Source: Feature Spec V1, `CAP-002`; Human Review `HV-017`

### CAP-003 — Consultar información contractual activa

- Type: SECONDARY
- Description: Mostrar empresa, contrato maestro y ubicación del único contrato activo de la máquina, cuando exista.
- Source: Feature Spec V1, `CAP-003`; Human Review `HV-002`, `HV-003`

### CAP-004 — Consultar histórico de lecturas de contador

- Type: SECONDARY
- Description: Mostrar exclusivamente las lecturas periódicas asociadas al contrato activo de la máquina.
- Source: Feature Spec V1, `CAP-004`; Human Review `HV-005`

### CAP-005 — Consultar histórico de mantenimientos

- Type: SECONDARY
- Description: Mostrar todos los mantenimientos de la máquina e identificar el contrato al que pertenece cada uno.
- Source: Feature Spec V1, `CAP-005`; Human Review `HV-006`

### CAP-006 — Consultar historial de repuestos/insumos instalados

- Type: SECONDARY
- Description: Mostrar todas las instalaciones de la máquina, identificar el contrato al que pertenece cada una y permitir ordenar y filtrar por repuesto.
- Source: Feature Spec V1, `CAP-006`; Human Review `HV-006`, `HV-018`

### CAP-007 — Registrar instalación de repuesto/insumo

- Type: PRIMARY
- Description: Registrar una instalación únicamente sobre una máquina existente previamente consultada, con repuesto/insumo, usuario solicitante, fecha y lectura de contador obligatorios, y nota opcional.
- Source: Feature Spec V1, `CAP-007`; Human Review `HV-008`, `HV-009`, `HV-010`, `HV-011`, `HV-012`

### CAP-008 — Acceso autenticado

- Type: SUPPORTING
- Description: Permitir la funcionalidad sólo a usuarios con sesión iniciada, sin autorización diferenciada por perfiles en esta versión.
- Source: Feature Spec V1, `CAP-008`; Human Review `HV-013`

---

## 4. Main flow

### Flujo principal A — Consulta de máquina

1. El usuario autenticado accede a la funcionalidad.
2. El usuario comienza a escribir el serial.
3. El sistema sugiere hasta 12 seriales que comienzan por el texto ingresado, sin distinguir mayúsculas y minúsculas.
4. El usuario selecciona o completa el serial y solicita la consulta.
5. El sistema valida que el serial haya sido informado y localiza la única máquina identificada por él.
6. El sistema muestra modelo, marca y zona.
7. Si existe contrato activo, el sistema muestra empresa, contrato maestro y ubicación, junto con las lecturas de contador pertenecientes exclusivamente a ese contrato.
8. El sistema muestra todos los mantenimientos y todas las instalaciones de la máquina, identificando el contrato al que pertenece cada registro.
9. El usuario puede ordenar el historial de instalaciones y filtrarlo mediante repuestos presentes en ese mismo historial.

### Flujo principal B — Registro de instalación

1. El acceso al registro se habilita únicamente después de consultar una máquina existente.
2. El sistema ofrece el catálogo de repuestos/insumos.
3. El usuario selecciona obligatoriamente un repuesto/insumo.
4. El sistema propone la fecha actual y el usuario informa una fecha no futura, sin hora.
5. El usuario informa obligatoriamente el usuario solicitante como texto libre y la lectura entera del contador; puede añadir una nota opcional de hasta 500 caracteres.
6. El sistema valida que el contador no sea negativo ni menor que las lecturas previas de instalación de la máquina.
7. El usuario confirma el registro.
8. El sistema registra la instalación, informa el éxito y actualiza el historial.
9. El sistema limpia todos los campos del formulario, incluida la selección del repuesto/insumo.

---

## 5. Inputs

### IN-001 — Serial de máquina

- Description: Identificador único de la máquina a consultar.
- Required: YES
- Constraints: Coincidencia exacta; debe informarse antes de consultar; longitud de 1 a 45 caracteres conservada desde la evidencia legacy.
- Origin: Legacy-derived in Feature Spec V1.
- Human source: `HV-001`, `HV-019`
- Status: CONFIRMED

### IN-002 — Texto parcial de serial

- Description: Texto utilizado para recibir sugerencias de seriales.
- Required: NO
- Constraints: Coincidencia por inicio del serial, sin distinguir mayúsculas y minúsculas; máximo 12 sugerencias.
- Origin: Legacy-derived in Feature Spec V1.
- Human source: `HV-017`
- Status: CONFIRMED

### IN-003 — Repuesto/insumo instalado

- Description: Elemento del catálogo instalado en la máquina.
- Required: YES
- Constraints: Debe pertenecer al catálogo de repuestos/insumos.
- Origin: Feature Spec V1, `IN-003`.
- Human source: `HV-008` (actualización relacionada necesaria con `VAL-005`)
- Status: CONFIRMED

### IN-004 — Nota

- Description: Observación libre sobre la instalación.
- Required: NO
- Constraints: Máximo 500 caracteres. El límite legacy de 300 caracteres no se conserva.
- Origin: Legacy-derived in Feature Spec V1; changed by Human Review.
- Human source: `HV-012`
- Status: CONFIRMED

### IN-005 — Usuario solicitante

- Description: Persona que solicitó la instalación.
- Required: YES
- Constraints: Texto libre, máximo 45 caracteres; no se selecciona de usuarios conocidos y no se sustituye por el usuario autenticado.
- Origin: Legacy-derived in Feature Spec V1; confirmed by Human Review.
- Human source: `HV-011`
- Status: CONFIRMED

### IN-006 — Fecha de instalación

- Description: Fecha en que se instaló el repuesto/insumo.
- Required: YES
- Constraints: No puede ser futura; el sistema propone la fecha actual; se registra como fecha sin hora. La fecha actual se determina en `America/Bogota`. El formato visual continúa pendiente.
- Origin: Feature Spec V1, `IN-006`; changed by Human Review.
- Human source: `HV-010`, `HV-016`
- Status: CONFIRMED, excepto formato visual `REQUIRES_HUMAN_VALIDATION` (`HV-016`)

### IN-007 — Lectura de contador en la instalación

- Description: Valor del contador de la máquina al momento de la instalación.
- Required: YES
- Constraints: Número entero, no negativo y no menor que las lecturas previas de instalación de la misma máquina. No se relaciona con las lecturas mensuales del contrato.
- Origin: Legacy-derived in Feature Spec V1; constraints extended by Human Review.
- Human source: `HV-007`
- Status: CONFIRMED

---

## 6. Outputs

### OUT-001 — Datos generales de la máquina

- Description: Modelo, marca y zona de la máquina consultada.
- Origin: Legacy-derived in Feature Spec V1.
- Status: CONFIRMED

### OUT-002 — Información contractual activa

- Description: Empresa, contrato maestro y ubicación del único contrato activo de la máquina. Si no existe, se informa expresamente y no se muestran datos contractuales.
- Origin: Legacy-derived in Feature Spec V1; consolidated by Human Review.
- Human source: `HV-002`, `HV-003`
- Status: CONFIRMED

### OUT-003 — Histórico de lecturas de contador

- Description: Período y valor de corte de las lecturas pertenecientes exclusivamente al contrato activo; no incluye contratos anteriores.
- Origin: Legacy-derived in Feature Spec V1; scope confirmed by Human Review.
- Human source: `HV-005`
- Status: CONFIRMED

### OUT-004 — Histórico de mantenimientos

- Description: Todos los mantenimientos de la máquina, con fecha, observación e identificación del contrato al que pertenece cada registro.
- Date rule: `America/Bogota`; formato visual pendiente.
- Origin: Legacy-derived in Feature Spec V1; contract identification introduced by Human Review.
- Human source: `HV-006`, `HV-016`
- Status: CONFIRMED, excepto formato visual de fecha `REQUIRES_HUMAN_VALIDATION` (`HV-016`)

### OUT-005 — Historial de repuestos/insumos instalados

- Description: Todas las instalaciones de la máquina, con fecha, repuesto, usuario solicitante, contador, nota e identificación del contrato al que pertenece cada registro; permite ordenar y filtrar, e informa cuando no hay registros.
- Date rule: `America/Bogota`; formato visual pendiente.
- Origin: Legacy-derived in Feature Spec V1; contract identification introduced by Human Review.
- Human source: `HV-006`, `HV-016`, `HV-018`
- Status: CONFIRMED, excepto formato visual de fecha `REQUIRES_HUMAN_VALIDATION` (`HV-016`)

### OUT-006 — Sugerencias de seriales

- Description: Hasta 12 seriales que comienzan por el texto ingresado, sin distinguir mayúsculas y minúsculas.
- Human source: `HV-017`
- Status: CONFIRMED

### OUT-007 — Confirmación de registro

- Description: Mensaje de éxito y actualización del historial tras registrar la instalación.
- Origin: Legacy-derived in Feature Spec V1.
- Status: CONFIRMED

### OUT-008 — Error controlado de registro

- Description: Mensaje con información relevante para corregir la situación y reintentar, sin información sensible ni detalles internos.
- Origin: Legacy behavior replaced by Human Review.
- Human source: `HV-014`
- Status: CONFIRMED

---

## 7. Functional requirements

### FR-001 — Sugerir seriales durante la búsqueda

El sistema debe sugerir hasta 12 seriales existentes que comiencen por el texto ingresado, sin distinguir mayúsculas y minúsculas.

- Origin: Legacy-derived in Feature Spec V1.
- Human source: `HV-017`
- Status: CONFIRMED

### FR-002 — Consultar máquina por serial único

El sistema debe permitir a un usuario autenticado consultar una máquina indicando su serial exacto y único.

- Origin: Legacy-derived in Feature Spec V1; uniqueness confirmed by Human Review.
- Human source: `HV-001`
- Status: CONFIRMED

### FR-003 — Mostrar datos generales

Tras consultar una máquina, el sistema debe mostrar su modelo, marca y zona.

- Origin: Legacy-derived in Feature Spec V1.
- Status: CONFIRMED

### FR-004 — Mostrar información del contrato activo

Tras consultar una máquina, el sistema debe mostrar empresa, contrato maestro y ubicación del único contrato activo, identificado funcionalmente por el estado 1. Si no existe contrato activo, debe aplicar `ALT-001`.

- Origin: Legacy-derived in Feature Spec V1; definition and cardinality changed by Human Review.
- Human source: `HV-002`, `HV-003`
- Status: CONFIRMED

### FR-005 — Mostrar lecturas del contrato activo

El sistema debe mostrar únicamente las lecturas de contador asociadas al contrato activo de la máquina y excluir las de contratos anteriores.

- Origin: Legacy-derived in Feature Spec V1; scope confirmed by Human Review.
- Human source: `HV-005`
- Status: CONFIRMED

### FR-006 — Mostrar todos los mantenimientos

El sistema debe mostrar todos los mantenimientos de la máquina e identificar el contrato al que pertenece cada uno. Las fechas se interpretan en `America/Bogota`.

- Origin: Legacy-derived in Feature Spec V1; contract identification introduced by Human Review.
- Human source: `HV-006`, `HV-016`
- Status: CONFIRMED, excepto formato visual de fecha `REQUIRES_HUMAN_VALIDATION` (`HV-016`)

### FR-007 — Mostrar todas las instalaciones

El sistema debe mostrar todas las instalaciones de repuestos/insumos de la máquina, identificar el contrato al que pertenece cada una y mostrar fecha, repuesto, usuario solicitante, contador y nota. Las fechas se interpretan en `America/Bogota`. Si no existen registros, debe indicarlo.

- Origin: Legacy-derived in Feature Spec V1; contract identification introduced by Human Review.
- Human source: `HV-006`, `HV-016`
- Status: CONFIRMED, excepto formato visual de fecha `REQUIRES_HUMAN_VALIDATION` (`HV-016`)

### FR-008 — Ordenar y filtrar el historial de instalaciones

El sistema debe permitir ordenar y filtrar el historial por repuesto. Las opciones del filtro deben incluir únicamente repuestos presentes en el historial de la máquina consultada.

- Origin: Legacy-derived in Feature Spec V1; filter scope changed by Human Review.
- Human source: `HV-018`
- Status: CONFIRMED

### FR-009 — Registrar una instalación

El sistema debe permitir registrar, sobre una máquina existente consultada, una instalación con repuesto/insumo, usuario solicitante, fecha y lectura de contador obligatorios, y nota opcional. Debe proponer la fecha actual, impedir fechas futuras y registrar sólo la fecha, sin hora.

- Origin: Legacy-derived in Feature Spec V1; required data and date behavior confirmed by Human Review.
- Human source: `HV-009`, `HV-010`, `HV-011`, `HV-012`
- Status: CONFIRMED

### FR-010 — Seleccionar del catálogo

El sistema debe ofrecer el catálogo de repuestos/insumos e identificar cada elemento por su descripción, con su nombre como información complementaria.

- Origin: Legacy-derived in Feature Spec V1.
- Status: CONFIRMED

### FR-011 — Confirmar, actualizar y limpiar

Tras un registro exitoso, el sistema debe informar al usuario, reflejar la instalación en el historial y limpiar todos los campos del formulario, incluida la selección de repuesto/insumo.

- Origin: Legacy-derived in Feature Spec V1; cleanup changed by Human Review.
- Human source: `HV-015`
- Status: CONFIRMED

### FR-012 — Restringir acceso por autenticación

El sistema debe impedir el acceso a usuarios sin sesión iniciada y dirigirlos al inicio de sesión. En esta versión no debe aplicar autorización diferenciada por perfiles.

- Origin: Authentication legacy-derived; role scope introduced by Human Review.
- Human source: `HV-013`
- Status: CONFIRMED

---

## 8. Business rules

### BR-001 — Unicidad del serial

El serial identifica de forma única una máquina.

- Legacy evidence: búsqueda por igualdad de serial y selección del primer resultado, registrada en Feature Spec V1.
- Human source: `HV-001`
- Status: CONFIRMED

### BR-002 — Contrato activo

El estado 1 significa contrato activo. Una máquina puede tener como máximo un contrato activo.

- Legacy evidence: selección del primer registro máquina-contrato con estado 1, registrada en Feature Spec V1.
- Human source: `HV-002`
- Status: CONFIRMED

### BR-003 — Alcance de las lecturas mensuales

Las lecturas mensuales pertenecen a una relación máquina-contrato y la consulta muestra exclusivamente las del contrato activo, nunca las de contratos anteriores.

- Legacy evidence: relación de contador con máquina-contrato, registrada en Feature Spec V1.
- Human source: `HV-005`
- Status: CONFIRMED

### BR-004 — Alcance de mantenimientos e instalaciones

Se muestran todos los mantenimientos y todas las instalaciones de la máquina, independientemente del contrato; cada registro debe identificar el contrato al que pertenece.

- Legacy evidence: asociaciones a la máquina, registradas en Feature Spec V1.
- Human source: `HV-006`
- Status: CONFIRMED

### BR-005 — Lectura de instalación

Toda instalación registra una lectura entera de contador. La lectura no puede ser negativa ni menor que las lecturas previas de instalación de la máquina y no tiene relación con las lecturas mensuales del contrato.

- Legacy evidence: obligatoriedad y entero, registrados en Feature Spec V1.
- Human source: `HV-007`
- Status: CONFIRMED

### BR-006 — Asociaciones obligatorias de la instalación

Toda instalación debe asociarse a una máquina existente previamente consultada y a un repuesto/insumo del catálogo.

- Origin: Feature Spec V1; consolidated consistently with Human Review.
- Human source: `HV-008`, `HV-009`
- Status: CONFIRMED

### BR-007 — Usuario solicitante

Toda instalación debe registrar el usuario solicitante como texto libre. No es necesario registrar adicionalmente al usuario autenticado que realiza el registro.

- Legacy evidence: solicitante como texto libre, registrado en Feature Spec V1.
- Human source: `HV-011`
- Status: CONFIRMED

### BR-008 — Autenticación sin roles diferenciados

La consulta y el registro requieren un usuario autenticado. La autorización diferenciada por perfiles no forma parte de esta versión.

- Legacy evidence: control de sesión, registrado en Feature Spec V1.
- Human source: `HV-013`
- Status: CONFIRMED

---

## 9. Validations

### VAL-001 — Lectura de contador obligatoria y no negativa

La lectura es obligatoria y debe ser mayor o igual que cero.

- Human source: `HV-007`
- Status: CONFIRMED

### VAL-002 — Lectura entera y no decreciente

La lectura debe ser un número entero y no puede ser menor que las lecturas previas de instalación de la máquina. No se compara con lecturas mensuales.

- Human source: `HV-007`
- Status: CONFIRMED

### VAL-003 — Nota opcional de hasta 500 caracteres

La nota no es obligatoria y, cuando se informa, no puede superar 500 caracteres.

- Legacy origin: límite de 300 caracteres en V1, reemplazado.
- Human source: `HV-012`
- Status: CONFIRMED

### VAL-004 — Usuario solicitante obligatorio

El usuario solicitante es texto libre obligatorio y no puede superar 45 caracteres.

- Origin: longitud legacy-derived in Feature Spec V1; obligatoriedad confirmed by Human Review.
- Human source: `HV-011` (relación necesaria para consistencia con `IN-005` y `FR-009`)
- Status: CONFIRMED

### VAL-005 — Repuesto/insumo obligatorio

Debe seleccionarse un repuesto/insumo del catálogo antes de registrar.

- Human source: `HV-008`
- Status: CONFIRMED

### VAL-006 — Máquina existente consultada

No debe permitirse el acceso al registro si no hay una máquina existente previamente consultada.

- Human source: `HV-009`
- Status: CONFIRMED

### VAL-007 — Fecha de instalación

La fecha es obligatoria, no puede ser futura, se propone inicialmente con la fecha actual de `America/Bogota` y se registra sin hora.

- Human source: `HV-010`, `HV-016`
- Status: CONFIRMED

### VAL-008 — Serial obligatorio

Debe informarse un serial antes de ejecutar la consulta. Si está vacío, se muestra un mensaje indicando que el serial es obligatorio.

- Human source: `HV-019`
- Status: CONFIRMED

---

## 10. Alternative flows

### ALT-001 — Máquina sin contrato activo

**Given** una máquina existente sin contrato activo

**When** el usuario la consulta

**Then** el sistema muestra sus datos generales, todos sus mantenimientos y todas sus instalaciones; indica que no existe contrato activo y no muestra información contractual ni lecturas de contador.

- Human source: `HV-003`
- Status: CONFIRMED

### ALT-002 — Máquina sin históricos

**Given** una máquina existente sin registros en uno o más históricos

**When** el usuario la consulta

**Then** el sistema muestra los datos disponibles e indica que no hay registros en cada histórico vacío.

- Origin: Feature Spec V1, unchanged.
- Status: CONFIRMED

### ALT-003 — Nueva consulta

**Given** una máquina consultada con un filtro aplicado

**When** el usuario consulta otra máquina

**Then** el sistema reemplaza toda la información por la nueva consulta y no conserva filtros ni datos anteriores.

- Origin: Feature Spec V1, unchanged.
- Status: CONFIRMED

### ALT-004 — Filtro sin coincidencias

**Given** una máquina con historial de instalaciones

**When** el usuario aplica una opción de filtro correspondiente a un repuesto presente en el historial y no quedan registros visibles por la combinación de filtros vigente

**Then** el sistema indica que no hay registros para el filtro aplicado.

- Human source: `HV-018`
- Status: CONFIRMED

### ALT-005 — Texto sin sugerencias

**Given** un texto parcial que no coincide con el inicio de ningún serial

**When** el usuario lo escribe

**Then** el sistema no muestra sugerencias.

- Origin: Feature Spec V1, unchanged.
- Status: CONFIRMED

---

## 11. Error scenarios

### ERR-001 — Máquina no encontrada

- Situation: el serial consultado no corresponde a ninguna máquina.
- Legacy behavior: excepción no controlada.
- Expected behavior: no mostrar datos de una máquina y mostrar un mensaje de error indicando que la máquina no existe.
- Human source: `HV-004`
- Status: CONFIRMED

### ERR-002 — Serial duplicado en datos

- Situation: más de una máquina comparte un serial, en contradicción con `BR-001`.
- Legacy behavior: selección no determinista del primer resultado.
- Consolidated decision: el serial es único y no existe desambiguación funcional; el comportamiento legacy de seleccionar arbitrariamente no se conserva. La resolución de datos que incumplan la regla queda fuera del flujo funcional normal.
- Human source: `HV-001`
- Status: CONFIRMED

### ERR-003 — Más de un contrato activo

- Situation: una máquina tiene más de un contrato con estado 1, en contradicción con `BR-002`.
- Legacy behavior: selección no determinista del primero.
- Consolidated decision: está prohibido que una máquina tenga más de un contrato activo y no se conserva la selección arbitraria legacy. No se define una regla de preferencia porque contradiría la decisión humana.
- Human source: `HV-002`
- Status: CONFIRMED

### ERR-004 — Registro sin máquina consultada

- Situation: el usuario no ha consultado una máquina existente.
- Expected behavior: no permitir el acceso al registro e indicar que debe consultarse primero una máquina existente.
- Human source: `HV-009`
- Status: CONFIRMED

### ERR-005 — Registro sin repuesto/insumo

- Situation: no se seleccionó un repuesto/insumo.
- Expected behavior: impedir el registro e indicar que el repuesto/insumo es obligatorio.
- Human source: `HV-008`
- Status: CONFIRMED

### ERR-006 — Lectura de contador inválida

- Situation: la lectura está vacía, contiene decimales, es negativa o es menor que una lectura previa de instalación de la máquina.
- Expected behavior: impedir el registro e indicar la condición incumplida. No se compara con lecturas mensuales.
- Human source: `HV-007`
- Status: CONFIRMED

### ERR-007 — Fallo al registrar

- Situation: el registro no puede completarse.
- Expected behavior: informar un error controlado con información relevante para que el usuario corrija la situación y reintente; no mostrar información sensible ni detalles internos y no dejar un registro parcial.
- Human source: `HV-014`
- Status: CONFIRMED

### ERR-008 — Acceso sin sesión

- Situation: un usuario sin sesión intenta acceder.
- Expected behavior: denegar el acceso y dirigir al inicio de sesión.
- Origin: Legacy-derived in Feature Spec V1.
- Status: CONFIRMED

---

## 12. Acceptance criteria

### AC-001 — Sugerencias de serial

**Given** existen más de 12 seriales que comienzan por "ab", con distintas combinaciones de mayúsculas y minúsculas

**When** el usuario escribe "ab"

**Then** el sistema muestra como máximo 12 seriales cuyo inicio coincide sin distinguir mayúsculas y minúsculas.

- Human source: `HV-017`

### AC-002 — Consulta de máquina existente

**Given** una máquina única con serial "ABC123", modelo "M1", marca "X" y zona "Norte"

**When** el usuario consulta "ABC123"

**Then** el sistema muestra modelo "M1", marca "X" y zona "Norte".

- Human source: `HV-001`

### AC-003 — Contrato activo

**Given** una máquina con un contrato en estado 1

**When** el usuario la consulta

**Then** el sistema trata ese contrato como activo y muestra su empresa, contrato maestro y ubicación.

- Human source: `HV-002`

### AC-004 — Lecturas sólo del contrato activo

**Given** una máquina con tres lecturas del contrato activo y dos de contratos anteriores

**When** el usuario la consulta

**Then** el sistema muestra las tres lecturas del contrato activo y ninguna de contratos anteriores.

- Human source: `HV-005`

### AC-005 — Todos los mantenimientos y su contrato

**Given** una máquina con mantenimientos asociados a contratos distintos

**When** el usuario la consulta

**Then** el sistema muestra todos los mantenimientos e identifica el contrato de cada uno, interpretando sus fechas en `America/Bogota`.

- Human source: `HV-006`, `HV-016`
- Open item: formato visual de fecha pendiente en `HV-016`.

### AC-006 — Todas las instalaciones y su contrato

**Given** una máquina con instalaciones asociadas a contratos distintos

**When** el usuario la consulta

**Then** el sistema muestra todas las instalaciones e identifica el contrato de cada una, interpretando sus fechas en `America/Bogota`.

- Human source: `HV-006`, `HV-016`
- Open item: formato visual de fecha pendiente en `HV-016`.

### AC-007 — Opciones y resultado del filtro

**Given** un historial con instalaciones de "Tóner" y "Fusor" y un catálogo que también contiene "Rodillo"

**When** el usuario abre las opciones del filtro y selecciona "Tóner"

**Then** las opciones incluyen "Tóner" y "Fusor", no incluyen "Rodillo", y el historial muestra sólo instalaciones de "Tóner".

- Human source: `HV-018`

### AC-008 — Historial vacío

**Given** una máquina sin instalaciones

**When** el usuario la consulta

**Then** el sistema indica que no hay registros en el historial.

### AC-009 — Registro exitoso

**Given** un usuario autenticado con una máquina existente consultada y un repuesto del catálogo

**When** registra una instalación con fecha no futura, usuario solicitante, lectura válida y una nota de hasta 500 caracteres

**Then** el sistema registra la instalación usando una fecha sin hora, informa el éxito, actualiza el historial y limpia todos los campos, incluido el repuesto seleccionado.

- Human source: `HV-010`, `HV-011`, `HV-012`, `HV-015`, `HV-016`
- Open item: la fecha usa `America/Bogota`; su formato visual continúa pendiente en `HV-016`.

### AC-010 — Contador obligatorio

**Given** una máquina existente consultada

**When** el usuario intenta registrar sin lectura

**Then** el sistema no registra e indica que la lectura es obligatoria.

- Human source: `HV-007`

### AC-011 — Contador válido

**Given** una máquina cuya última lectura de instalación es 15000

**When** el usuario informa una lectura decimal, negativa o menor que 15000

**Then** el sistema rechaza el valor e indica la condición incumplida, sin comparar contra lecturas mensuales.

- Human source: `HV-007`

### AC-012 — Nota y usuario solicitante

**Given** una máquina existente consultada

**When** el usuario deja vacía la nota y proporciona un solicitante de hasta 45 caracteres

**Then** el sistema acepta esos datos;

**And when** la nota supera 500 caracteres, el solicitante está vacío o supera 45 caracteres

**Then** el sistema impide el registro e indica la condición incumplida.

- Human source: `HV-011`, `HV-012`

### AC-013 — Serial inexistente

**Given** no existe una máquina con serial "ZZZ999"

**When** el usuario consulta "ZZZ999"

**Then** el sistema informa que la máquina no existe y no muestra datos de ninguna máquina.

- Human source: `HV-004`

### AC-014 — Máquina sin contrato activo

**Given** el usuario consultó una máquina con contrato activo y luego consulta otra sin contrato activo

**When** se completa la segunda consulta

**Then** el sistema muestra los datos generales, mantenimientos e instalaciones de la segunda máquina, indica que no existe contrato activo y no muestra información contractual ni lecturas de la consulta anterior.

- Human source: `HV-003`

### AC-015 — Registro no accesible sin máquina

**Given** un usuario autenticado sin una máquina existente consultada

**When** intenta acceder al registro

**Then** el sistema no permite el acceso e indica que primero debe consultar una máquina existente.

- Human source: `HV-009`

### AC-016 — Repuesto obligatorio

**Given** una máquina existente consultada

**When** el usuario intenta registrar sin seleccionar repuesto/insumo

**Then** el sistema no registra e indica que el repuesto/insumo es obligatorio.

- Human source: `HV-008`

### AC-017 — Acceso sin sesión

**Given** un usuario sin sesión iniciada

**When** intenta acceder

**Then** el sistema deniega el acceso y lo dirige al inicio de sesión.

- Human source: `HV-013`

### AC-018 — Fecha de instalación

**Given** una máquina existente consultada

**When** el usuario inicia un registro

**Then** el sistema propone la fecha actual de `America/Bogota`;

**And when** el usuario omite la fecha o informa una fecha futura

**Then** el sistema impide el registro;

**And when** registra una fecha válida

**Then** se conserva la fecha sin hora.

- Human source: `HV-010`, `HV-016`

### AC-019 — Error controlado de registro

**Given** un intento de registro que no puede completarse

**When** ocurre el fallo

**Then** el sistema no deja un registro parcial y muestra información útil para corregir y reintentar, sin datos sensibles ni detalles internos.

- Human source: `HV-014`

### AC-020 — Serial obligatorio

**Given** un usuario autenticado

**When** solicita una consulta con el serial vacío

**Then** el sistema no ejecuta la consulta e indica que el serial es obligatorio.

- Human source: `HV-019`

---

## 13. Legacy behaviors not preserved

Los siguientes comportamientos provienen de evidencia legacy y no son requisitos del sistema futuro:

- Excepción no controlada al consultar un serial inexistente; reemplazada por `HV-004`.
- Selección no determinista ante seriales duplicados; contradice `HV-001`.
- Selección no determinista ante varios contratos con estado 1; contradice `HV-002`.
- Persistencia de información contractual o lecturas de una consulta anterior; reemplazada por `HV-003`.
- Intento de registrar sin máquina o sin repuesto y dependencia de un fallo de persistencia; reemplazado por `HV-008` y `HV-009`.
- Conservación del repuesto seleccionado tras registrar; reemplazada por `HV-015`.
- Mensaje genérico de registro fallido; reemplazado por `HV-014`.
- Filtro alimentado con todo el catálogo; reemplazado por `HV-018`.
- Límite legacy de 300 caracteres para la nota; reemplazado por `HV-012`.
- Ausencia de autorización diferenciada por perfiles: no se considera defecto en esta versión, sino decisión de alcance de `HV-013`.
- Cargas completas, recargas internas, errores tipográficos y demás detalles técnicos listados en la sección 13 de la V1; no se convierten en requerimientos funcionales.

---

## 14. Human review decisions

| ID | Decision | Consolidated result | Impact processed |
|----|----------|---------------------|------------------|
| HV-001 | CONFIRMED | El serial identifica de forma única una máquina. | BR-001, FR-002, ERR-002; consistencia: IN-001, CAP-001, AC-002 |
| HV-002 | CONFIRMED_WITH_CHANGE | Estado 1 significa contrato activo y existe como máximo uno por máquina. | BR-002, FR-004, ERR-003; consistencia: CAP-003, OUT-002, AC-003 |
| HV-003 | CONFIRMED | Sin contrato activo se muestran datos generales, mantenimientos e instalaciones, pero no información contractual ni lecturas. | ALT-001, AC-014; consistencia: FR-004, OUT-002 |
| HV-004 | CONFIRMED | Un serial inexistente produce un mensaje de máquina inexistente. | ERR-001, AC-013 |
| HV-005 | CONFIRMED | Se muestran lecturas exclusivamente del contrato activo. | BR-003, FR-005, AC-004; consistencia: CAP-004, OUT-003 |
| HV-006 | CONFIRMED | Se muestran todos los mantenimientos e instalaciones y se identifica su contrato. | BR-004, FR-006, FR-007, OUT-004, OUT-005, AC-005, AC-006; consistencia: CAP-005, CAP-006 |
| HV-007 | CONFIRMED | Contador entero, obligatorio, no negativo, no decreciente respecto de lecturas previas de instalación y sin relación con lecturas mensuales. | BR-005, IN-007, VAL-001, VAL-002, ERR-006; consistencia: AC-010, AC-011 |
| HV-008 | CONFIRMED | El repuesto/insumo es obligatorio. | VAL-005, ERR-005, AC-016; consistencia: IN-003, BR-006, CAP-007 |
| HV-009 | CONFIRMED | Sin máquina existente consultada no se permite acceder al registro. | VAL-006, ERR-004, AC-015; consistencia: BR-006, CAP-007, FR-009 |
| HV-010 | CONFIRMED | Fecha obligatoria, no futura, propuesta con la fecha actual y sin hora. | IN-006, VAL-007, FR-009, AC-009; consistencia: flujo B, AC-018 |
| HV-011 | CONFIRMED | Solicitante de texto libre obligatorio; no se registra adicionalmente al usuario autenticado. | BR-007, IN-005, FR-009, AC-009; consistencia: VAL-004, AC-012 |
| HV-012 | CONFIRMED | Nota opcional con máximo de 500 caracteres. | VAL-003, IN-004, AC-012; consistencia: CAP-007, FR-009 |
| HV-013 | CONFIRMED_WITH_CHANGE | Sólo se exige autenticación; roles diferenciados fuera del alcance. | BR-008, FR-012, Actors; consistencia: CAP-008, AC-017, sección 13 |
| HV-014 | CONFIRMED | Error controlado, útil para corregir y reintentar, sin información sensible o interna. | ERR-007; consistencia: OUT-008, AC-019 |
| HV-015 | CONFIRMED | Después del éxito también se limpia el repuesto seleccionado. | FR-011, AC-009; consistencia: flujo B |
| HV-016 | PENDING | `America/Bogota` confirmada; formato visual de fechas pendiente. | FR-006, FR-007, OUT-004, OUT-005, AC-005, AC-006, AC-009; consistencia: IN-006, VAL-007, AC-018 |
| HV-017 | CONFIRMED | Sugerencias por inicio, sin distinguir mayúsculas/minúsculas, máximo 12. | FR-001, IN-002, AC-001; consistencia: CAP-002, OUT-006 |
| HV-018 | CONFIRMED | El filtro ofrece sólo repuestos presentes en el historial de la máquina. | FR-008, AC-007, ALT-004; consistencia: CAP-006 |
| HV-019 | CONFIRMED | Un serial vacío produce mensaje de obligatoriedad. | VAL-008; consistencia: IN-001, AC-020 |

---

## 15. Rejected decisions

No existen decisiones `REJECTED` en el Human Review.

---

## 16. Open questions

| ID | Pending question | Related items | Blocking |
|----|------------------|---------------|----------|
| HV-016 | Definir el formato visual de las fechas de mantenimiento e instalación. La zona horaria `America/Bogota` ya está confirmada. | FR-006, FR-007, OUT-004, OUT-005, AC-005, AC-006, AC-009 | NO |

No se trasladan como pendientes funcionales las preguntas de la V1 que quedaron resueltas por el Human Review, fueron reemplazadas por decisiones humanas o corresponden a detalles legacy/técnicos fuera del alcance funcional consolidado.

---

## 17. Traceability

### 17.1 Functional traceability

| Functional item | Legacy / V1 source | Human source | Classification |
|-----------------|--------------------|--------------|----------------|
| FR-001 | V1 FR-001; Observed behavior #2 | HV-017 | LEGACY_DERIVED + HUMAN_CONFIRMED |
| FR-002 | V1 FR-002; Observed behavior #3 | HV-001 | LEGACY_DERIVED + HUMAN_CONFIRMED |
| FR-003 | V1 FR-003; UI / Resolución EL | Review approval | LEGACY_DERIVED |
| FR-004 | V1 FR-004; Observed behavior #5 | HV-002, HV-003 | LEGACY_DERIVED + HUMAN_MODIFIED |
| FR-005 | V1 FR-005; Observed behavior #7 | HV-005 | LEGACY_DERIVED + HUMAN_CONFIRMED |
| FR-006 | V1 FR-006; Observed behavior #7, #8 | HV-006, HV-016 | LEGACY_DERIVED + HUMAN_EXTENDED + OPEN_ITEM |
| FR-007 | V1 FR-007; Observed behavior #7 | HV-006, HV-016 | LEGACY_DERIVED + HUMAN_EXTENDED + OPEN_ITEM |
| FR-008 | V1 FR-008; Observed behavior #9 | HV-018 | LEGACY_DERIVED + HUMAN_CONFIRMED |
| FR-009 | V1 FR-009; Observed behavior #11 | HV-009, HV-010, HV-011, HV-012 | LEGACY_DERIVED + HUMAN_EXTENDED |
| FR-010 | V1 FR-010; Observed behavior #1, #10 | Review approval | LEGACY_DERIVED |
| FR-011 | V1 FR-011; Observed behavior #13 | HV-015 | LEGACY_DERIVED + HUMAN_MODIFIED |
| FR-012 | V1 FR-012; Observed behavior #16 | HV-013 | LEGACY_DERIVED + HUMAN_SCOPE_DECISION |
| BR-001 | V1 BR-001; Inferred business rule #1 | HV-001 | LEGACY_CANDIDATE + HUMAN_CONFIRMED |
| BR-002 | V1 BR-002; Inferred business rule #2 | HV-002 | LEGACY_CANDIDATE + HUMAN_MODIFIED |
| BR-003 | V1 BR-003; Inferred business rule #3 | HV-005 | LEGACY_CANDIDATE + HUMAN_CONFIRMED |
| BR-004 | V1 BR-004; Observed behavior #7 | HV-006 | LEGACY_CANDIDATE + HUMAN_EXTENDED |
| BR-005 | V1 BR-005; Inferred business rule #4 | HV-007 | LEGACY_CANDIDATE + HUMAN_EXTENDED |
| BR-006 | V1 BR-006; Inferred business rule #5 | HV-008, HV-009 | LEGACY_CANDIDATE + HUMAN_CONFIRMED |
| BR-007 | V1 BR-007; Inferred business rule #6 | HV-011 | LEGACY_CANDIDATE + HUMAN_CONFIRMED |
| BR-008 | V1 BR-008; Inferred business rule #7 | HV-013 | LEGACY_CANDIDATE + HUMAN_SCOPE_DECISION |
| VAL-001–VAL-008 | V1 validations | HV-007–HV-012, HV-019 as mapped in section 14 | CONSOLIDATED_VALIDATIONS |
| ALT-001 | V1 ALT-001; Observed behavior #6 | HV-003 | LEGACY_DEFECT_REPLACED |
| ALT-004 | V1 ALT-004; Observed behavior #9 | HV-018 | LEGACY_DERIVED + HUMAN_CONFIRMED |
| ERR-001–ERR-007 | V1 error scenarios | HV-001, HV-002, HV-004, HV-007–HV-009, HV-014 | LEGACY_BEHAVIOR_CONSOLIDATED |
| AC-001–AC-020 | V1 acceptance criteria plus direct human decisions | HV mappings in each AC | VERIFIABLE_CONSOLIDATED_CRITERIA |

### 17.2 Authority distinction

- `LEGACY_DERIVED`: comportamiento funcional originado en el Legacy Map y trasladado a la Feature Spec V1.
- `HUMAN_CONFIRMED`: decisión humana que confirma una propuesta de la V1.
- `HUMAN_MODIFIED`, `HUMAN_EXTENDED` o `HUMAN_SCOPE_DECISION`: comportamiento introducido o cambiado por el Human Review; prevalece sobre la evidencia legacy.
- `OPEN_ITEM`: únicamente el formato visual de fechas de `HV-016`; no bloquea arquitectura.

---

## 18. Consolidation result

- Todas las decisiones `HV-001` a `HV-019` fueron procesadas.
- Ningún elemento confirmado permanece con estado global `REQUIRES_HUMAN_VALIDATION`.
- Los comportamientos anteriores contradichos por `HV-002`, `HV-012`, `HV-013`, `HV-014`, `HV-015` y `HV-018` fueron sustituidos.
- No existen decisiones rechazadas.
- El único pendiente es el formato visual de fechas de `HV-016`; la zona horaria `America/Bogota` está confirmada.
- El gate indica `architecture_can_start: true`; por existir un pendiente no bloqueante, el estado final es `APPROVED_WITH_OPEN_ITEMS`.
