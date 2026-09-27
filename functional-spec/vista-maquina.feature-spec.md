---
artifact: feature-spec
schema_version: 1.0
feature: vista-maquina

agent:
  name: functional-analyst
  version: 1.0

source:
  artifact: legacy-map/vista-maquina.legacy-map.md

status: DRAFT

human_validation_required: true
open_questions: 12

generated_at: 2026-09-27T12:00:00
---

# Feature Specification — Consulta de máquina y registro de instalación de repuestos/insumos

## 1. Purpose

Permitir a un usuario del sistema localizar una máquina mediante su serial y consultar en un único punto su información general (modelo, marca, zona), su información contractual vigente (empresa, contrato maestro, ubicación) y sus históricos asociados (lecturas de contador, mantenimientos y repuestos/insumos instalados).

Adicionalmente, permitir registrar la instalación de un repuesto/insumo en la máquina consultada, dejando constancia de la fecha, la lectura del contador en ese momento, quién la solicitó y una nota descriptiva.

La funcionalidad legacy agrupa en una sola pantalla dos responsabilidades diferenciadas: **consulta integral de la máquina** (solo lectura) y **registro de instalación de repuestos/insumos** (escritura). Esta especificación las separa como capacidades distintas.

---

## 2. Actors

- `Usuario autenticado`
  - Description: Persona que ha iniciado sesión en el sistema y accede a la funcionalidad. El legacy solo verifica la existencia de sesión; no aplica ningún control de perfil o rol ni para consultar ni para registrar instalaciones.
  - Status: REQUIRES_HUMAN_VALIDATION (se desconoce qué perfiles de negocio deben usar la funcionalidad y si la consulta y el registro requieren permisos distintos)

- `Usuario solicitante`
  - Description: Persona que solicitó la instalación del repuesto/insumo. En el legacy se registra como texto libre y no se relaciona con los usuarios del sistema. No interactúa directamente con el sistema en este flujo; es un dato informado por el actor que registra.
  - Status: REQUIRES_HUMAN_VALIDATION

---

## 3. Functional capabilities

### CAP-001 — Consultar máquina por serial

- Type: PRIMARY
- Description: Localizar una máquina indicando su serial y mostrar sus datos generales (modelo, marca, zona).
- Source: `Feature`, `Actions` (buscarDatosMaquina), `Observed behavior #3`

### CAP-002 — Asistencia en la localización del serial

- Type: SUPPORTING
- Description: Sugerir seriales existentes a medida que el usuario escribe, para facilitar la localización de la máquina.
- Source: `Actions` (autoComplete), `Observed behavior #2`

### CAP-003 — Consultar información contractual vigente de la máquina

- Type: SECONDARY
- Description: Mostrar la empresa, el contrato maestro y la ubicación asociados al contrato vigente de la máquina consultada.
- Source: `Actions` (buscarContrato), `Observed behavior #5`, `Inferred business rule #2`

### CAP-004 — Consultar histórico de lecturas de contador

- Type: SECONDARY
- Description: Mostrar las lecturas periódicas de contador (período año-mes y valor de corte mensual) asociadas al contrato vigente de la máquina.
- Source: `Observed behavior #7`, `Inferred business rule #3`

### CAP-005 — Consultar histórico de mantenimientos

- Type: SECONDARY
- Description: Mostrar los mantenimientos realizados a la máquina (fecha y observación).
- Source: `Observed behavior #7`, `UI / Resolución EL`

### CAP-006 — Consultar historial de repuestos/insumos instalados

- Type: SECONDARY
- Description: Mostrar los repuestos/insumos instalados en la máquina (fecha, repuesto, usuario solicitante, contador, nota), con posibilidad de ordenar y filtrar por repuesto.
- Source: `Observed behavior #7`, `Observed behavior #9`

### CAP-007 — Registrar instalación de repuesto/insumo

- Type: PRIMARY
- Description: Registrar que un repuesto/insumo del catálogo fue instalado en la máquina consultada, indicando nota, usuario solicitante, fecha de instalación y lectura de contador.
- Source: `Actions` (adicionarRepuesto), `Observed behavior #11`, `#12`, `#13`, `#14`

### CAP-008 — Acceso restringido a usuarios autenticados

- Type: SUPPORTING
- Description: La funcionalidad solo está disponible para usuarios con sesión iniciada.
- Source: `Actions` (verificarSesion), `Observed behavior #16`, `Inferred business rule #7`

---

## 4. Main flow

**Flujo principal A — Consulta de máquina**

1. El usuario autenticado accede a la funcionalidad de consulta de máquina.
2. El usuario comienza a escribir el serial de la máquina.
3. El sistema sugiere seriales existentes que coinciden con el texto ingresado.
4. El usuario selecciona o completa el serial y solicita la búsqueda.
5. El sistema localiza la máquina con ese serial.
6. El sistema muestra los datos generales de la máquina: modelo, marca y zona.
7. El sistema determina el contrato vigente de la máquina y muestra empresa, contrato maestro y ubicación.
8. El sistema muestra el histórico de lecturas de contador del contrato vigente.
9. El sistema muestra el histórico de mantenimientos de la máquina.
10. El sistema muestra el historial de repuestos/insumos instalados en la máquina; el usuario puede ordenarlo y filtrarlo por repuesto.

**Flujo principal B — Registro de instalación de repuesto/insumo** (requiere haber completado el flujo A)

1. Con una máquina consultada, el usuario elige registrar una instalación.
2. El sistema ofrece el catálogo de repuestos/insumos disponibles.
3. El usuario selecciona el repuesto/insumo instalado.
4. El usuario informa nota, usuario solicitante, fecha de instalación y lectura de contador.
5. El usuario confirma el registro.
6. El sistema valida los datos.
7. El sistema registra la instalación asociada a la máquina consultada y al repuesto/insumo seleccionado.
8. El sistema informa que la instalación se registró correctamente.
9. El sistema actualiza la información de la máquina consultada, incluyendo el historial de repuestos/insumos, y deja el formulario de registro listo para un nuevo registro.

---

## 5. Inputs

### IN-001 — Serial de máquina

- Description: Identificador de la máquina a consultar.
- Required: YES (para ejecutar la consulta)
- Constraints: Búsqueda por coincidencia exacta. Longitud entre 1 y 45 caracteres según el modelo de datos legacy. En el legacy no existe validación de campo vacío antes de buscar.
- Source: `UI / Resolución EL` (valorIngresado), `Database` (Serial `@Size(1,45)`), `Observed behavior #3`
- Status: REQUIRES_HUMAN_VALIDATION (unicidad y tratamiento de serial vacío)

### IN-002 — Texto parcial de serial (para sugerencias)

- Description: Texto que el usuario escribe para recibir sugerencias de seriales.
- Required: NO
- Constraints: En el legacy las sugerencias se basan en seriales que comienzan por el texto, sin distinguir mayúsculas/minúsculas, con un máximo de 12 sugerencias.
- Source: `Observed behavior #2`
- Status: REQUIRES_HUMAN_VALIDATION (criterio de coincidencia y límite)

### IN-003 — Repuesto/insumo instalado

- Description: Elemento del catálogo de repuestos/insumos que se instaló en la máquina.
- Required: UNKNOWN (el modelo de datos exige la asociación, pero la interfaz legacy no lo valida antes de guardar)
- Constraints: Debe pertenecer al catálogo de repuestos. En el legacy el catálogo se presenta ordenado por descripción, mostrando la descripción y, como información complementaria, el nombre.
- Source: `Validation` (optional=false; selectOneMenu no required), `Observed behavior #10`, `Inferred business rule #5`
- Status: REQUIRES_HUMAN_VALIDATION

### IN-004 — Nota

- Description: Observación libre sobre la instalación.
- Required: NO (según evidencia legacy)
- Constraints: Máximo 300 caracteres.
- Source: `Validation` (nota `@Size(max=300)`)
- Status: CANDIDATE

### IN-005 — Usuario solicitante

- Description: Persona que solicitó la instalación.
- Required: NO (según evidencia legacy)
- Constraints: Texto libre, máximo 45 caracteres. No se relaciona con el usuario autenticado.
- Source: `Validation` (usuario `@Size(max=45)`), `Observed behavior #12`, `Inferred business rule #6`
- Status: REQUIRES_HUMAN_VALIDATION

### IN-006 — Fecha de instalación

- Description: Fecha en que se instaló el repuesto/insumo.
- Required: UNKNOWN (el legacy no la valida como obligatoria)
- Constraints: El legacy la almacena con fecha y hora; no hay restricciones de rango documentadas.
- Source: `Validation` ("No hay validación en el bean de ... fecha"), `Database` (fecha TIMESTAMP)
- Status: REQUIRES_HUMAN_VALIDATION

### IN-007 — Lectura de contador en la instalación

- Description: Valor del contador de la máquina en el momento de la instalación.
- Required: YES
- Constraints: Número entero (sin decimales). No hay evidencia de restricciones de rango (negativos, relación con lecturas anteriores).
- Source: `Validation` (required, decimalPlaces=0, `@NotNull`), `Observed behavior #15`, `Inferred business rule #4`
- Status: CANDIDATE

---

## 6. Outputs

### OUT-001 — Datos generales de la máquina

- Description: Modelo, marca y zona de la máquina consultada.
- Source: `UI / Resolución EL` (modeloid, zonaid, marcaid)

### OUT-002 — Información contractual vigente

- Description: Empresa del contrato, nombre de empresa del contrato maestro y ubicación de la máquina según el contrato vigente.
- Source: `UI / Resolución EL` (contratoActual), `Observed behavior #5`

### OUT-003 — Histórico de lecturas de contador

- Description: Lista de lecturas (período año-mes y valor de corte mensual) del contrato vigente, paginada.
- Source: `UI / Resolución EL` (contadorList), `Observed behavior #7`

### OUT-004 — Histórico de mantenimientos

- Description: Lista de mantenimientos de la máquina (fecha y observación), paginada.
- Source: `UI / Resolución EL` (mantenimientoList), `Observed behavior #7`, `#8`

### OUT-005 — Historial de repuestos/insumos instalados

- Description: Lista de instalaciones de la máquina (fecha, descripción del repuesto, usuario solicitante, contador, nota), paginada, ordenable y filtrable por descripción del repuesto; muestra un mensaje cuando no hay registros.
- Source: `UI / Resolución EL` (historialRepuestosList), `Observed behavior #9`, `PrimeFaces components` (emptyMessage)

### OUT-006 — Sugerencias de seriales

- Description: Lista de seriales existentes que coinciden con el texto ingresado.
- Source: `Observed behavior #2`

### OUT-007 — Confirmación de registro de instalación

- Description: Mensaje informativo de éxito tras registrar la instalación, y actualización del historial de repuestos/insumos.
- Source: `Observed behavior #13`

### OUT-008 — Mensaje de error de registro

- Description: Mensaje de error cuando la instalación no pudo registrarse.
- Source: `Observed behavior #14`

---

## 7. Functional requirements

### FR-001 — Sugerir seriales durante la búsqueda

El sistema debe sugerir al usuario seriales de máquinas existentes que coincidan con el texto que está escribiendo, para facilitar la localización de la máquina.

- Source: `Actions` (autoComplete), `Observed behavior #2`
- Status: CANDIDATE (criterio de coincidencia y límite de sugerencias: REQUIRES_HUMAN_VALIDATION, ver HV-017)

### FR-002 — Consultar máquina por serial

El sistema debe permitir a un usuario autenticado consultar una máquina indicando su serial exacto.

- Source: `Actions` (buscarDatosMaquina), `Observed behavior #3`, `Inferred business rule #1`
- Status: CANDIDATE

### FR-003 — Mostrar datos generales de la máquina

Tras consultar una máquina, el sistema debe mostrar su modelo, marca y zona.

- Source: `UI / Resolución EL` (modeloid, marcaid, zonaid)
- Status: CANDIDATE

### FR-004 — Mostrar información contractual vigente

Tras consultar una máquina, el sistema debe mostrar la empresa, el contrato maestro y la ubicación correspondientes al contrato vigente de la máquina.

- Source: `Actions` (buscarContrato), `Observed behavior #5`, `Inferred business rule #2`
- Status: REQUIRES_HUMAN_VALIDATION (definición de "contrato vigente", ver BR-002)

### FR-005 — Consultar histórico de lecturas de contador

El sistema debe mostrar el histórico de lecturas de contador (período y valor de corte mensual) asociadas al contrato vigente de la máquina consultada.

- Source: `Observed behavior #7`, `Inferred business rule #3`
- Status: REQUIRES_HUMAN_VALIDATION (alcance: solo contrato vigente vs. todos los contratos, ver HV-005)

### FR-006 — Consultar histórico de mantenimientos

El sistema debe mostrar los mantenimientos registrados para la máquina consultada, con su fecha y observación.

- Source: `Observed behavior #7`, `#8`
- Status: CANDIDATE

### FR-007 — Consultar historial de repuestos/insumos instalados

El sistema debe mostrar las instalaciones de repuestos/insumos registradas para la máquina consultada, con fecha, repuesto, usuario solicitante, lectura de contador y nota. Si no existen registros, debe indicarlo explícitamente.

- Source: `Observed behavior #7`, `PrimeFaces components` (emptyMessage)
- Status: CANDIDATE

### FR-008 — Ordenar y filtrar el historial de repuestos/insumos

El sistema debe permitir ordenar y filtrar el historial de repuestos/insumos de la máquina por el repuesto instalado.

- Source: `Observed behavior #9`
- Status: CANDIDATE (origen de las opciones de filtro: REQUIRES_HUMAN_VALIDATION, ver HV-018)

### FR-009 — Registrar instalación de repuesto/insumo

El sistema debe permitir registrar la instalación de un repuesto/insumo del catálogo en la máquina consultada, con nota, usuario solicitante, fecha de instalación y lectura de contador.

- Source: `Actions` (adicionarRepuesto), `Observed behavior #11`
- Status: CANDIDATE

### FR-010 — Seleccionar repuesto/insumo del catálogo

Al registrar una instalación, el sistema debe ofrecer el catálogo de repuestos/insumos para su selección, identificando cada elemento por su descripción y ofreciendo su nombre como información complementaria.

- Source: `Observed behavior #1`, `#10`
- Status: CANDIDATE

### FR-011 — Confirmar el registro y actualizar la consulta

Tras registrar una instalación con éxito, el sistema debe informar al usuario, reflejar la nueva instalación en el historial de la máquina consultada y dejar el formulario de registro vacío para un nuevo registro.

- Source: `Observed behavior #13`
- Status: REQUIRES_HUMAN_VALIDATION (el legacy no limpia la selección de repuesto, ver HV-015)

### FR-012 — Restringir acceso a usuarios autenticados

El sistema debe impedir el acceso a la funcionalidad a usuarios sin sesión iniciada, dirigiéndolos al inicio de sesión.

- Source: `Actions` (verificarSesion), `Navigation`, `Observed behavior #16`, `Inferred business rule #7`
- Status: CANDIDATE (autorización por rol: REQUIRES_HUMAN_VALIDATION, ver HV-013)

---

## 8. Business rules

### BR-001 — El serial identifica a la máquina

Una máquina se identifica funcionalmente por su serial para efectos de consulta. Candidata: el serial es único entre máquinas.

- Legacy evidence: búsqueda por igualdad de serial; autocompletado por serial; Serial obligatorio (`Maquina.java:29,43-47`, `MaquinaFacade.java:43-46`). No existe restricción de unicidad visible; el legacy toma el primer resultado.
- Confidence: MEDIUM
- Status: `REQUIRES_HUMAN_VALIDATION`

### BR-002 — Contrato vigente de la máquina

La información contractual mostrada para una máquina corresponde a su contrato vigente. Candidata: una máquina tiene como máximo un contrato vigente a la vez.

- Legacy evidence: se toma el primer registro máquina-contrato con `Estado_contrato = 1`, sin orden definido (`ControladorVistaMaquina.java:167-174`). El significado del valor 1 no está documentado y la unicidad no se garantiza.
- Confidence: MEDIUM
- Status: `REQUIRES_HUMAN_VALIDATION`

### BR-003 — Las lecturas de contador pertenecen a la relación máquina-contrato

Las lecturas periódicas de contador se registran para una máquina dentro de un contrato concreto, no para la máquina de forma global; por ello la consulta muestra las del contrato vigente.

- Legacy evidence: `Contador.maquinasContratoid` (`Contador.java:69-71`); tabla alimentada desde el contrato actual (`VistaMaquina.xhtml:116-119`).
- Confidence: MEDIUM
- Status: `REQUIRES_HUMAN_VALIDATION`

### BR-004 — Mantenimientos e instalaciones pertenecen a la máquina

Los mantenimientos y las instalaciones de repuestos/insumos se asocian a la máquina, con independencia del contrato vigente; por ello se muestran completos aunque la máquina haya cambiado de contrato.

- Legacy evidence: `Maquina.mantenimientoList`, `Maquina.historialRepuestosList` (`VistaMaquina.xhtml:133,153-155`); Observed behavior #7.
- Confidence: MEDIUM
- Status: `REQUIRES_HUMAN_VALIDATION`

### BR-005 — Toda instalación registra la lectura del contador

Toda instalación de un repuesto/insumo debe registrar la lectura del contador de la máquina en ese momento.

- Legacy evidence: contador obligatorio en UI (`VistaMaquina.xhtml:230`) y obligatorio en el modelo (`HistorialRepuestos.java:58-61`).
- Confidence: MEDIUM
- Status: `REQUIRES_HUMAN_VALIDATION`

### BR-006 — Toda instalación se asocia a una máquina y a un repuesto del catálogo

No puede existir una instalación sin máquina ni sin repuesto/insumo del catálogo.

- Legacy evidence: asociaciones obligatorias en el modelo (`HistorialRepuestos.java:50-56`); la UI no lo valida antes de guardar.
- Confidence: HIGH
- Status: `REQUIRES_HUMAN_VALIDATION`

### BR-007 — Usuario solicitante de la instalación

La instalación registra quién la solicitó, no necesariamente quién la registró en el sistema.

- Legacy evidence: campo "Usuario solicitante" de texto libre, no tomado de la sesión (`VistaMaquina.xhtml:219-222`).
- Confidence: LOW
- Status: `REQUIRES_HUMAN_VALIDATION`

### BR-008 — Acceso requiere autenticación

La consulta de máquinas y el registro de instalaciones requieren usuario autenticado. No hay evidencia de reglas de autorización por perfil.

- Legacy evidence: `verificarSesion()` en la plantilla (`PlantillaController.java:19-27`, `plantilla.xhtml:13`).
- Confidence: HIGH (autenticación) / LOW (autorización)
- Status: `REQUIRES_HUMAN_VALIDATION`

---

## 9. Validations

### VAL-001 — Lectura de contador obligatoria

La lectura del contador es obligatoria al registrar la instalación de un repuesto/insumo.

- Source: `Validation` (required, `@NotNull`), `Observed behavior #15`
- Status: CANDIDATE

### VAL-002 — Lectura de contador entera

La lectura del contador debe ser un número entero, sin decimales.

- Source: `Validation` (decimalPlaces=0), `Observed behavior #15`
- Status: CANDIDATE (restricciones adicionales como no negativo o no inferior a la última lectura: REQUIRES_HUMAN_VALIDATION, ver HV-007)

### VAL-003 — Longitud máxima de la nota

La nota de la instalación no puede superar 300 caracteres.

- Source: `Validation` (nota `@Size(max=300)`)
- Status: CANDIDATE

### VAL-004 — Longitud máxima del usuario solicitante

El usuario solicitante no puede superar 45 caracteres.

- Source: `Validation` (usuario `@Size(max=45)`)
- Status: CANDIDATE

### VAL-005 — Repuesto/insumo obligatorio

Debe seleccionarse un repuesto/insumo del catálogo para registrar una instalación.

- Source: `Validation` (asociación obligatoria en el modelo; selección no obligatoria en la UI legacy), `Inferred business rule #5`
- Status: REQUIRES_HUMAN_VALIDATION

### VAL-006 — Máquina consultada obligatoria para registrar

Solo puede registrarse una instalación si hay una máquina consultada (existente) seleccionada.

- Source: `Validation` ("No hay validación en el bean de: máquina buscada antes de guardar"), `Open questions #5`
- Status: REQUIRES_HUMAN_VALIDATION

### VAL-007 — Fecha de instalación

Obligatoriedad y rango permitido de la fecha de instalación (p. ej., no futura).

- Source: `Validation` (sin validación de fecha en el legacy)
- Status: REQUIRES_HUMAN_VALIDATION

### VAL-008 — Serial informado para buscar

Debe indicarse un serial antes de ejecutar la consulta.

- Source: `Validation` (sin validación de serial en el legacy)
- Status: REQUIRES_HUMAN_VALIDATION

---

## 10. Alternative flows

### ALT-001 — Máquina sin contrato vigente

**Given**
una máquina existente que no tiene ningún contrato vigente

**When**
el usuario la consulta por su serial

**Then**
el sistema muestra los datos generales de la máquina, sus mantenimientos y su historial de repuestos/insumos, e indica que la máquina no tiene contrato vigente, sin mostrar información contractual ni lecturas de contador de otra máquina o de una consulta anterior.

- Status: REQUIRES_HUMAN_VALIDATION (el legacy conserva datos de la consulta anterior; ver sección 13)

### ALT-002 — Máquina sin históricos

**Given**
una máquina existente sin lecturas de contador, mantenimientos o instalaciones registradas

**When**
el usuario la consulta

**Then**
el sistema muestra los datos disponibles y, para cada histórico vacío, indica que no hay registros.

- Status: CANDIDATE

### ALT-003 — Nueva consulta sobre una consulta previa

**Given**
el usuario ya consultó una máquina y aplicó un filtro al historial de repuestos/insumos

**When**
consulta otra máquina

**Then**
el sistema reemplaza toda la información mostrada (datos generales, contractuales, históricos) por la de la nueva máquina y no conserva filtros ni datos de la consulta anterior.

- Status: REQUIRES_HUMAN_VALIDATION

### ALT-004 — Filtro de historial sin coincidencias

**Given**
una máquina consultada con historial de repuestos/insumos

**When**
el usuario filtra por un repuesto que no figura en el historial

**Then**
el sistema indica que no hay registros para el filtro aplicado.

- Status: CANDIDATE

### ALT-005 — Texto sin sugerencias

**Given**
un texto parcial que no coincide con ningún serial existente

**When**
el usuario lo escribe en el campo de serial

**Then**
el sistema no muestra sugerencias.

- Status: CANDIDATE

---

## 11. Error scenarios

### ERR-001 — Máquina no encontrada

- Situation: el usuario consulta un serial que no corresponde a ninguna máquina.
- Legacy behavior: se produce una excepción no controlada; el usuario no recibe un mensaje claro (comportamiento visible exacto no verificado).
- Expected behavior: REQUIRES_HUMAN_VALIDATION (propuesta candidata: informar que no existe una máquina con ese serial y no mostrar datos de ninguna máquina).

### ERR-002 — Serial asociado a más de una máquina

- Situation: existen varias máquinas con el mismo serial.
- Legacy behavior: se muestra la primera de forma no determinista.
- Expected behavior: REQUIRES_HUMAN_VALIDATION (depende de BR-001).

### ERR-003 — Varios contratos vigentes para una máquina

- Situation: la máquina tiene más de un contrato con estado vigente.
- Legacy behavior: se muestra el primero según un orden no definido.
- Expected behavior: REQUIRES_HUMAN_VALIDATION (depende de BR-002).

### ERR-004 — Registro sin máquina consultada

- Situation: el usuario intenta registrar una instalación sin haber consultado una máquina existente.
- Legacy behavior: el registro falla en persistencia y se muestra el mensaje genérico "No se pudo insertar".
- Expected behavior: REQUIRES_HUMAN_VALIDATION (propuesta candidata: impedir el registro e informar que debe consultarse primero una máquina).

### ERR-005 — Registro sin repuesto/insumo seleccionado

- Situation: el usuario confirma el registro sin seleccionar repuesto/insumo.
- Legacy behavior: no hay validación previa; el resultado depende del fallo de persistencia (mensaje genérico previsible, no verificado).
- Expected behavior: REQUIRES_HUMAN_VALIDATION (propuesta candidata: impedir el registro indicando que el repuesto/insumo es obligatorio).

### ERR-006 — Lectura de contador no informada

- Situation: el usuario confirma el registro sin informar la lectura de contador.
- Legacy behavior: el registro se impide con el mensaje de obligatoriedad por defecto del framework.
- Expected behavior: impedir el registro indicando explícitamente que la lectura del contador es obligatoria (CANDIDATE).

### ERR-007 — Fallo al registrar la instalación

- Situation: el registro no puede completarse por cualquier causa.
- Legacy behavior: mensaje de error genérico "ERROR / No se pudo insertar" sin detalle de causa.
- Expected behavior: REQUIRES_HUMAN_VALIDATION (nivel de detalle a comunicar al usuario; no debe quedar registro parcial).

### ERR-008 — Acceso sin sesión

- Situation: un usuario sin sesión intenta acceder a la funcionalidad.
- Legacy behavior: redirección a la página de inicio de sesión.
- Expected behavior: denegar el acceso y dirigir al inicio de sesión (CANDIDATE).

---

## 12. Acceptance criteria

### AC-001 — Sugerencias de serial

**Given**
un usuario autenticado
AND existen máquinas con seriales "ABC123" y "ABD456"

**When**
escribe "ab" en el campo de serial

**Then**
el sistema sugiere "ABC123" y "ABD456".

### AC-002 — Consulta de máquina existente

**Given**
un usuario autenticado
AND una máquina existente con serial "ABC123", modelo "M1", marca "X" y zona "Norte"

**When**
consulta el serial "ABC123"

**Then**
el sistema muestra modelo "M1", marca "X" y zona "Norte".

### AC-003 — Información contractual vigente

**Given**
una máquina con serial "ABC123" con un único contrato vigente de la empresa "E1", contrato maestro "CM1" y ubicación "Piso 2"

**When**
el usuario consulta el serial "ABC123"

**Then**
el sistema muestra empresa "E1", contrato maestro "CM1" y ubicación "Piso 2".

### AC-004 — Lecturas de contador del contrato vigente

**Given**
una máquina con un contrato vigente que tiene 3 lecturas de contador
AND un contrato no vigente anterior con 2 lecturas

**When**
el usuario consulta la máquina

**Then**
el sistema muestra las 3 lecturas del contrato vigente (período y valor de corte).
(Pendiente de HV-005 si deben mostrarse también las del contrato anterior.)

### AC-005 — Mantenimientos de la máquina

**Given**
una máquina con 2 mantenimientos registrados

**When**
el usuario consulta la máquina

**Then**
el sistema muestra los 2 mantenimientos con su fecha y observación.

### AC-006 — Historial de repuestos/insumos

**Given**
una máquina con 4 instalaciones registradas

**When**
el usuario consulta la máquina

**Then**
el sistema muestra las 4 instalaciones con fecha, repuesto, usuario solicitante, contador y nota.

### AC-007 — Filtrar historial por repuesto

**Given**
una máquina con instalaciones de los repuestos "Tóner" (2) y "Fusor" (1)

**When**
el usuario filtra el historial por "Tóner"

**Then**
el sistema muestra solo las 2 instalaciones de "Tóner".

### AC-008 — Historial vacío

**Given**
una máquina sin instalaciones registradas

**When**
el usuario la consulta

**Then**
el sistema indica que no hay registros en el historial de repuestos/insumos.

### AC-009 — Registro de instalación exitoso

**Given**
un usuario autenticado con la máquina "ABC123" consultada
AND el repuesto "Tóner" existe en el catálogo

**When**
registra una instalación de "Tóner" con contador 15000, fecha 2026-09-01, usuario solicitante "Juan" y nota "Cambio preventivo"

**Then**
el sistema informa que la instalación se registró correctamente
AND el historial de la máquina "ABC123" incluye la instalación de "Tóner" con contador 15000, fecha 01/09/2026, solicitante "Juan" y nota "Cambio preventivo"
AND el formulario de registro queda vacío.

### AC-010 — Contador obligatorio

**Given**
un usuario con una máquina consultada

**When**
intenta registrar una instalación sin informar el contador

**Then**
el sistema no registra la instalación e indica que el contador es obligatorio.

### AC-011 — Contador entero

**Given**
un usuario con una máquina consultada

**When**
intenta informar el contador "150,5"

**Then**
el sistema no acepta valores con decimales.

### AC-012 — Límites de nota y usuario solicitante

**Given**
un usuario con una máquina consultada

**When**
intenta registrar una nota de 301 caracteres o un usuario solicitante de 46 caracteres

**Then**
el sistema no registra la instalación e indica el límite superado.

### AC-013 — Serial inexistente (pendiente de validación)

**Given**
no existe ninguna máquina con serial "ZZZ999"

**When**
el usuario consulta "ZZZ999"

**Then**
el sistema informa que no se encontró la máquina y no muestra datos de ninguna máquina.
(Resultado esperado sujeto a HV-004.)

### AC-014 — Máquina sin contrato vigente (pendiente de validación)

**Given**
el usuario consultó antes la máquina "ABC123" con contrato vigente de la empresa "E1"
AND la máquina "XYZ789" no tiene contrato vigente

**When**
consulta "XYZ789"

**Then**
el sistema no muestra la empresa "E1" ni ningún otro dato contractual o de contadores de "ABC123", e indica que "XYZ789" no tiene contrato vigente.
(Resultado esperado sujeto a HV-003.)

### AC-015 — Registro sin máquina consultada (pendiente de validación)

**Given**
un usuario autenticado que no ha consultado ninguna máquina

**When**
intenta registrar una instalación

**Then**
el sistema no permite el registro e indica que debe consultarse primero una máquina.
(Sujeto a HV-009.)

### AC-016 — Registro sin repuesto (pendiente de validación)

**Given**
un usuario con una máquina consultada

**When**
intenta registrar una instalación sin seleccionar repuesto/insumo

**Then**
el sistema no registra la instalación e indica que el repuesto/insumo es obligatorio.
(Sujeto a HV-008.)

### AC-017 — Acceso sin sesión

**Given**
un usuario sin sesión iniciada

**When**
intenta acceder a la funcionalidad

**Then**
el sistema no muestra la funcionalidad y lo dirige al inicio de sesión.

---

## 13. Legacy behaviors not to preserve automatically

- Excepción no controlada al consultar un serial inexistente.
  - Legacy evidence: `MaquinaFacade.java:46` (`get(0)` sobre lista vacía), `ControladorVistaMaquina.java:96-99`; Open question #1
  - Classification: POSSIBLE_DEFECT
  - Decision: `DO_NOT_PROPAGATE_WITHOUT_VALIDATION`

- Selección no determinista cuando hay seriales duplicados o varios contratos con estado 1.
  - Legacy evidence: `MaquinaFacade.java:43-46`, `ControladorVistaMaquina.java:167-174`; Open questions #2, #3
  - Classification: POSSIBLE_DEFECT
  - Decision: `DO_NOT_PROPAGATE_WITHOUT_VALIDATION`

- Información contractual y de contadores de una consulta anterior que permanece visible al consultar una máquina sin contrato vigente.
  - Legacy evidence: `ControladorVistaMaquina.java:167-174`; Observed behavior #6; Open question #4
  - Classification: POSSIBLE_DEFECT
  - Decision: `DO_NOT_PROPAGATE_WITHOUT_VALIDATION`

- Permitir intentar el registro sin máquina consultada ni repuesto seleccionado, dependiendo de un fallo de persistencia para impedirlo.
  - Legacy evidence: `ControladorVistaMaquina.java:139-150`, `VistaMaquina.xhtml:207-212`; Open question #5
  - Classification: POSSIBLE_DEFECT
  - Decision: `DO_NOT_PROPAGATE_WITHOUT_VALIDATION`

- La selección de repuesto no se limpia tras un registro exitoso; posible descripción vacía en la fila recién registrada.
  - Legacy evidence: `ControladorVistaMaquina.java:142-144`; Open question #6
  - Classification: POSSIBLE_DEFECT
  - Decision: `DO_NOT_PROPAGATE_WITHOUT_VALIDATION`

- El filtro del historial no se reinicia al consultar otra máquina.
  - Legacy evidence: `ControladorVistaMaquina.java:49`, `VistaMaquina.xhtml:158`; Open question #7
  - Classification: POSSIBLE_DEFECT
  - Decision: `DO_NOT_PROPAGATE_WITHOUT_VALIDATION`

- Mensaje de error genérico sin causa al fallar el registro.
  - Legacy evidence: `ControladorVistaMaquina.java:147-148`; Observed behavior #14
  - Classification: TECHNICAL_BEHAVIOR
  - Decision: `DO_NOT_PROPAGATE_WITHOUT_VALIDATION`

- Carga completa de todas las máquinas al abrir la pantalla para las sugerencias (lista que no se refresca durante la sesión de pantalla) y recarga del catálogo de repuestos en cada evaluación.
  - Legacy evidence: `ControladorVistaMaquina.java:83-84,125-133`; Observed behavior #1, #10; Open question #8
  - Classification: TECHNICAL_BEHAVIOR
  - Decision: `DO_NOT_PROPAGATE_WITHOUT_VALIDATION`

- Opciones del filtro del historial tomadas de todo el catálogo, incluyendo repuestos sin descripción (opción vacía).
  - Legacy evidence: `ControladorVistaMaquina.java:160-165`, `Respuesto.java:43-45`; Open question #9
  - Classification: POSSIBLE_DEFECT
  - Decision: `DO_NOT_PROPAGATE_WITHOUT_VALIDATION`

- Posible desfase de fecha entre el registro de la fecha de instalación y su visualización por diferencias de zona horaria.
  - Legacy evidence: `HistorialRepuestos.java:38-40`, `VistaMaquina.xhtml:164-165,224-226`; Open question #12
  - Classification: POSSIBLE_DEFECT
  - Decision: `DO_NOT_PROPAGATE_WITHOUT_VALIDATION`

- Errores tipográficos de interfaz ("Ubicaion"), hoja de estilos ignorada, identificadores no semánticos, tipo genérico incorrecto, cast inseguro.
  - Legacy evidence: `VistaMaquina.xhtml:8-10,98,207`, `ControladorVistaMaquina.java:49,161-164`; Open questions #7, #9, #10, #11
  - Classification: TECHNICAL_BEHAVIOR
  - Decision: `DO_NOT_PROPAGATE_WITHOUT_VALIDATION`

- Ausencia de control de autorización por perfil.
  - Legacy evidence: `PlantillaController.java:22`; Open question #14
  - Classification: POSSIBLE_DEFECT
  - Decision: `DO_NOT_PROPAGATE_WITHOUT_VALIDATION`

- Hallazgos fuera de la cadena de esta funcionalidad (NamedQuery con nombre inconsistente en `buscarPorEmpresa`, `refresh()` defectuoso, generación de esquema en `persistence.xml`, typo en outcome de cierre de sesión).
  - Legacy evidence: Open questions #15, #16, #17; `Navigation` (`plantilla.xhtml:25`)
  - Classification: TECHNICAL_BEHAVIOR
  - Decision: `DO_NOT_PROPAGATE_WITHOUT_VALIDATION` (no relevantes para esta especificación)

---

## 14. Human validation required

| ID | Question | Related item | Impact |
|----|----------|--------------|--------|
| HV-001 | ¿El serial es único por máquina? Si no lo es, ¿cómo debe desambiguarse la consulta? | BR-001, ERR-002, FR-002 | HIGH |
| HV-002 | ¿Qué significa `Estado_contrato = 1` y qué otros estados existen? ¿Puede una máquina tener más de un contrato vigente simultáneamente? Si es posible, ¿cuál debe mostrarse? | BR-002, ERR-003, FR-004 | HIGH |
| HV-003 | ¿Qué debe mostrarse cuando la máquina no tiene contrato vigente? | ALT-001, AC-014 | HIGH |
| HV-004 | ¿Cuál es el comportamiento esperado al consultar un serial inexistente? | ERR-001, AC-013 | HIGH |
| HV-005 | ¿Las lecturas de contador deben mostrarse solo del contrato vigente o también de contratos anteriores de la máquina? | BR-003, FR-005, AC-004 | MEDIUM |
| HV-006 | ¿Se confirma que mantenimientos e instalaciones se muestran para la máquina completa, independientemente del contrato? | BR-004, FR-006, FR-007 | MEDIUM |
| HV-007 | ¿La lectura de contador en la instalación admite valores negativos o menores a lecturas previas? ¿Existe relación con las lecturas mensuales de contador? | BR-005, VAL-001, VAL-002 | MEDIUM |
| HV-008 | ¿La selección de repuesto/insumo es obligatoria para registrar una instalación? | VAL-005, ERR-005, AC-016 | HIGH |
| HV-009 | ¿Debe exigirse una máquina consultada y existente antes de permitir el registro? | VAL-006, ERR-004, AC-015 | HIGH |
| HV-010 | ¿La fecha de instalación es obligatoria? ¿Admite fechas futuras? ¿Debe proponerse la fecha actual? ¿Interesa registrar la hora? | VAL-007, IN-006 | MEDIUM |
| HV-011 | ¿El usuario solicitante debe seguir siendo texto libre o seleccionarse de usuarios/personas conocidas? ¿Es obligatorio? ¿Debe registrarse también el usuario autenticado que realiza el registro? | BR-007, IN-005 | MEDIUM |
| HV-012 | ¿La nota es obligatoria? ¿Es adecuado el límite de 300 caracteres? | VAL-003, IN-004 | LOW |
| HV-013 | ¿Qué perfiles pueden consultar máquinas y cuáles pueden registrar instalaciones? | BR-008, FR-012, Actors | HIGH |
| HV-014 | ¿Qué información debe recibir el usuario cuando el registro falla? | ERR-007 | MEDIUM |
| HV-015 | Tras un registro exitoso, ¿debe limpiarse también la selección de repuesto/insumo? | FR-011 | LOW |
| HV-016 | ¿En qué zona horaria y formato deben registrarse y mostrarse las fechas de mantenimiento e instalación? | FR-006, FR-007, sección 13 | MEDIUM |
| HV-017 | ¿El criterio de sugerencias de serial (empieza por, sin distinguir mayúsculas, máximo 12) es el deseado? | FR-001, IN-002 | LOW |
| HV-018 | ¿Las opciones del filtro del historial deben ser todo el catálogo o solo los repuestos presentes en el historial de la máquina? | FR-008 | LOW |
| HV-019 | ¿Qué debe ocurrir si el usuario solicita la consulta con el serial vacío? | VAL-008 | LOW |

---

## 15. Open questions

1. REQUIRES_LEGACY_ANALYSIS — Valores reales existentes de `Estado_contrato` en datos y su significado (no documentado en código).
2. REQUIRES_LEGACY_ANALYSIS — Qué perfiles tienen acceso a la opción de menú de esta funcionalidad (configurado en la tabla `Menu`, no disponible en el repositorio).
3. REQUIRES_LEGACY_ANALYSIS — Comportamiento exacto visible para el usuario al consultar un serial inexistente.
4. REQUIRES_LEGACY_ANALYSIS — Existencia de restricción de unicidad del serial a nivel de base de datos.
5. REQUIRES_LEGACY_ANALYSIS — Si la instalación recién registrada se muestra con la descripción del repuesto vacía tras el refresco.
6. REQUIRES_LEGACY_ANALYSIS — Confirmación del posible desfase de día en la fecha de instalación por zona horaria.
7. REQUIRES_LEGACY_ANALYSIS — Si existen otras funcionalidades legacy que registren, editen o eliminen instalaciones de repuestos o lecturas de contador, y su relación con esta.
8. REQUIRES_HUMAN_VALIDATION — ¿"Repuesto" e "insumo" son el mismo concepto de negocio? ¿Es relevante la categoría del repuesto para esta funcionalidad?
9. REQUIRES_HUMAN_VALIDATION — ¿Es necesario poder corregir o anular una instalación registrada? El legacy no lo ofrece en esta pantalla.
10. REQUIRES_HUMAN_VALIDATION — Significado funcional de "período año-mes" y "corte mensual" de las lecturas de contador.
11. REQUIRES_HUMAN_VALIDATION — ¿Deben mostrarse otros datos de la máquina o del contrato (descripción de la máquina, tipo de contrato) que el legacy tiene en su modelo pero no muestra?
12. REQUIRES_HUMAN_VALIDATION — ¿Cómo llega el usuario a esta funcionalidad y existe algún punto de entrada con serial preseleccionado? (en el legacy: NO ENCONTRADO en código, se asume menú).

---

## 16. Traceability

Numeración de Observed behavior e Inferred business rules según su orden de aparición en el legacy-map.

| Functional item | Legacy source | Classification |
|-----------------|---------------|----------------|
| FR-001 | Observed behavior #2; Actions autoComplete | FUNCTIONAL_REQUIREMENT |
| FR-002 | Observed behavior #3; Actions buscarDatosMaquina; Inferred business rule #1 | FUNCTIONAL_REQUIREMENT |
| FR-003 | UI / Resolución EL (modeloid, marcaid, zonaid) | FUNCTIONAL_REQUIREMENT |
| FR-004 | Observed behavior #5; Actions buscarContrato; Inferred business rule #2 | FUNCTIONAL_REQUIREMENT |
| FR-005 | Observed behavior #7; Inferred business rule #3 | FUNCTIONAL_REQUIREMENT |
| FR-006 | Observed behavior #7, #8 | FUNCTIONAL_REQUIREMENT |
| FR-007 | Observed behavior #7; PrimeFaces components (emptyMessage) | FUNCTIONAL_REQUIREMENT |
| FR-008 | Observed behavior #9 | FUNCTIONAL_REQUIREMENT |
| FR-009 | Observed behavior #11; Actions adicionarRepuesto | FUNCTIONAL_REQUIREMENT |
| FR-010 | Observed behavior #1, #10 | FUNCTIONAL_REQUIREMENT |
| FR-011 | Observed behavior #13 | FUNCTIONAL_REQUIREMENT |
| FR-012 | Observed behavior #16; Navigation; Inferred business rule #7 | FUNCTIONAL_REQUIREMENT |
| BR-001 | Inferred business rule #1; Open question #2 | BUSINESS_RULE_CANDIDATE |
| BR-002 | Inferred business rule #2; Open question #3 | BUSINESS_RULE_CANDIDATE |
| BR-003 | Inferred business rule #3 | BUSINESS_RULE_CANDIDATE |
| BR-004 | Observed behavior #7 | BUSINESS_RULE_CANDIDATE |
| BR-005 | Inferred business rule #4 | BUSINESS_RULE_CANDIDATE |
| BR-006 | Inferred business rule #5 | BUSINESS_RULE_CANDIDATE |
| BR-007 | Inferred business rule #6; Observed behavior #12 | BUSINESS_RULE_CANDIDATE |
| BR-008 | Inferred business rule #7; Open question #14 | BUSINESS_RULE_CANDIDATE |
| VAL-001 | Validation (required, @NotNull); Observed behavior #15 | VALIDATION |
| VAL-002 | Validation (decimalPlaces=0); Observed behavior #15 | VALIDATION |
| VAL-003 | Validation (nota @Size 300) | VALIDATION |
| VAL-004 | Validation (usuario @Size 45) | VALIDATION |
| VAL-005 | Validation (optional=false; combo no required) | VALIDATION |
| VAL-006 | Validation (sin validación de máquina); Open question #5 | VALIDATION |
| VAL-007 | Validation (sin validación de fecha) | VALIDATION |
| VAL-008 | Validation (sin validación de serial) | VALIDATION |
| ALT-001 | Observed behavior #6; Open question #4 | ERROR_SCENARIO |
| ALT-002 | PrimeFaces components (emptyMessage) | FUNCTIONAL_REQUIREMENT |
| ALT-003 | Open question #7 | ERROR_SCENARIO |
| ALT-004 | Observed behavior #9 | FUNCTIONAL_REQUIREMENT |
| ALT-005 | Observed behavior #2 | FUNCTIONAL_REQUIREMENT |
| ERR-001 | Observed behavior #4; Open question #1 | POSSIBLE_DEFECT / ERROR_SCENARIO |
| ERR-002 | Observed behavior #3; Open question #2 | ERROR_SCENARIO |
| ERR-003 | Observed behavior #5; Open question #3 | ERROR_SCENARIO |
| ERR-004 | Open question #5 | ERROR_SCENARIO |
| ERR-005 | Validation (combo no required); Open question #5 | ERROR_SCENARIO |
| ERR-006 | Observed behavior #15 | ERROR_SCENARIO |
| ERR-007 | Observed behavior #14 | ERROR_SCENARIO |
| ERR-008 | Observed behavior #16; Navigation | ERROR_SCENARIO |
| Sección 13 | Observed behavior #1, #4, #6, #10, #14; Open questions #1-#17 | POSSIBLE_DEFECT / TECHNICAL_BEHAVIOR |

---

## 17. Analysis trail

1. Observation: carga inicial de todas las máquinas y del catálogo de repuestos al abrir la pantalla.
   - Classification: TECHNICAL_BEHAVIOR
   - Evidence: `Observed behavior #1`
   - Decision: no se convierte en requisito; solo se conserva la necesidad de ofrecer el catálogo (FR-010). Registrado en sección 13.
2. Observation: sugerencias de serial por prefijo, sin distinguir mayúsculas, máximo 12.
   - Classification: FUNCTIONAL_REQUIREMENT
   - Evidence: `Observed behavior #2`
   - Decision: FR-001; los parámetros concretos pasan a HV-017.
3. Observation: búsqueda por serial exacto tomando el primer resultado.
   - Classification: FUNCTIONAL_REQUIREMENT + BUSINESS_RULE_CANDIDATE
   - Evidence: `Observed behavior #3`, `Inferred business rule #1`
   - Decision: FR-002; unicidad como BR-001 (ACCEPT_AS_CANDIDATE, REQUIRES_HUMAN_VALIDATION); duplicados como ERR-002.
4. Observation: excepción no controlada ante serial inexistente.
   - Classification: POSSIBLE_DEFECT
   - Evidence: `Observed behavior #4`, `Open question #1`
   - Decision: ERR-001 con comportamiento esperado pendiente (HV-004); no se propaga.
5. Observation: contrato actual = primer registro con estado 1.
   - Classification: BUSINESS_RULE_CANDIDATE
   - Evidence: `Observed behavior #5`, `Inferred business rule #2`
   - Decision: BR-002 (REQUIRES_HUMAN_VALIDATION); FR-004; ERR-003.
6. Observation: el contrato de la consulta anterior persiste si la nueva máquina no tiene contrato vigente.
   - Classification: POSSIBLE_DEFECT
   - Evidence: `Observed behavior #6`, `Open question #4`
   - Decision: ALT-001 define la necesidad; HV-003; sección 13.
7. Observation: contadores del contrato activo; mantenimientos e historial de la máquina.
   - Classification: FUNCTIONAL_REQUIREMENT + BUSINESS_RULE_CANDIDATE
   - Evidence: `Observed behavior #7`, `Inferred business rule #3`
   - Decision: FR-005, FR-006, FR-007; BR-003 y BR-004 (REQUIRES_HUMAN_VALIDATION).
8. Observation: fechas mostradas en formato dd/MM/yyyy zona America/Bogota.
   - Classification: TECHNICAL_BEHAVIOR (presentación)
   - Evidence: `Observed behavior #8`
   - Decision: no se fija como requisito; zona horaria y formato a HV-016.
9. Observation: historial ordenable y filtrable por descripción del repuesto.
   - Classification: FUNCTIONAL_REQUIREMENT
   - Evidence: `Observed behavior #9`
   - Decision: FR-008; origen de opciones de filtro a HV-018.
10. Observation: catálogo de repuestos recargado en cada evaluación; descripción como etiqueta, nombre como detalle.
    - Classification: FUNCTIONAL_REQUIREMENT (presentación del catálogo) + TECHNICAL_BEHAVIOR (recarga)
    - Evidence: `Observed behavior #10`
    - Decision: FR-010; la recarga repetida va a sección 13.
11. Observation: al guardar se crea un registro de instalación ligado a máquina y repuesto con los datos tecleados.
    - Classification: FUNCTIONAL_REQUIREMENT
    - Evidence: `Observed behavior #11`
    - Decision: FR-009.
12. Observation: usuario solicitante como texto libre, no tomado de la sesión.
    - Classification: BUSINESS_RULE_CANDIDATE
    - Evidence: `Observed behavior #12`, `Inferred business rule #6`
    - Decision: BR-007 (LOW, REQUIRES_HUMAN_VALIDATION); HV-011.
13. Observation: tras guardar se limpia el formulario (excepto repuesto), se refresca la máquina y se muestra mensaje de éxito.
    - Classification: FUNCTIONAL_REQUIREMENT + POSSIBLE_DEFECT (repuesto no reiniciado)
    - Evidence: `Observed behavior #13`, `Open question #6`
    - Decision: FR-011; HV-015.
14. Observation: cualquier excepción al guardar muestra "No se pudo insertar".
    - Classification: ERROR_SCENARIO
    - Evidence: `Observed behavior #14`
    - Decision: ERR-007; HV-014.
15. Observation: contador obligatorio y entero.
    - Classification: VALIDATION
    - Evidence: `Observed behavior #15`, `Validation`
    - Decision: VAL-001, VAL-002, ERR-006.
16. Observation: redirección a inicio de sesión si no hay sesión.
    - Classification: FUNCTIONAL_REQUIREMENT
    - Evidence: `Observed behavior #16`, `Inferred business rule #7`
    - Decision: FR-012, BR-008, ERR-008; autorización por rol a HV-013.
17. Observation: evaluación de reglas inferidas.
    - Classification: —
    - Evidence: `Inferred business rules #1-#7`
    - Decision: #1 → BR-001, #2 → BR-002, #3 → BR-003, #4 → BR-005, #5 → BR-006, #6 → BR-007, #7 → BR-008; todas ACCEPT_AS_CANDIDATE con REQUIRES_HUMAN_VALIDATION. Ninguna rechazada como técnica.
18. Observation: validaciones de longitud y asociaciones obligatorias del modelo; ausencia de validación de máquina, repuesto, fecha y serial en la interfaz.
    - Classification: VALIDATION
    - Evidence: `Validation`
    - Decision: VAL-003, VAL-004 (CANDIDATE); VAL-005 a VAL-008 (REQUIRES_HUMAN_VALIDATION).
19. Observation: hallazgos técnicos fuera de la cadena funcional (Open questions #10, #11, #15, #16, #17).
    - Classification: NOT_RELEVANT / TECHNICAL_BEHAVIOR
    - Evidence: `Open questions`
    - Decision: registrados en sección 13 sin generar requisitos.
20. Observation: el legacy-map no incluye front-matter de estado; su contenido cubre todas las secciones del schema.
    - Classification: —
    - Evidence: legacy-map completo
    - Decision: se considera suficiente para generar la especificación en estado DRAFT (no BLOCKED).
