# Legacy Map — Vista de Máquina (consulta por serial e instalación de repuestos/insumos)

> Pantalla analizada: `src/main/webapp/protegido/VistaMaquina.xhtml` · Fecha: 2026-09-27 · Agente: legacy-analyst

## Feature
Vista de Máquina: consulta de una máquina por serial y registro de instalación de repuestos/insumos.
El usuario busca una máquina por serial (con autocompletado) y ve los datos del contrato activo (empresa, contrato maestro, ubicación), modelo, zona y marca. La vista tiene pestañas con el histórico de contadores del contrato activo, los mantenimientos y el historial de repuestos. Desde la pestaña "Instalar" puede registrar un nuevo repuesto/insumo instalado en la máquina (nota, usuario solicitante, fecha, contador).

## UI
- `src/main/webapp/protegido/VistaMaquina.xhtml` — template: `./../WEB-INF/templates/plantilla.xhtml` (`VistaMaquina.xhtml:12`), rellena `ui:define name="content"` (`VistaMaquina.xhtml:13`); includes: N/A — no hay `ui:include`.
  - Stylesheet declarado fuera de la composición: `library="css" name="../resoruces/css/index.css"` (`VistaMaquina.xhtml:9`). Al estar fuera de `ui:composition`, Facelets lo ignora (ver Open questions).
  - La plantilla ejecuta `<f:event type="preRenderView" listener="#{plantillaController.verificarSesion()}">` (`src/main/webapp/WEB-INF/templates/plantilla.xhtml:13`) y pinta el menú `#{menuControler.modelo}` (`plantilla.xhtml:23`).

## ManagedBean
- `ControladorVistaMaquina` (`#{controladorVistaMaquina}`, scope `@ViewScoped` (javax.faces.view.ViewScoped)) — `src/main/java/com/cmi/controlador/ControladorVistaMaquina.java:21-23`
  - `@EJB HistorialRepuestosFacadeLocal historialEJB` (`:30-31`), `@EJB MaquinaFacadeLocal maquinasEJB` (`:35-36`), `@EJB RespuestoFacadeLocal repuestoEJB` (`:42-43`).
  - `@PostConstruct init()` (`:77-85`): crea `historial = new HistorialRepuestos()`, `repuesto = new Respuesto()`, `maquinaBuscada = new Maquina()`, carga `listaMaquinas = maquinasEJB.findAll()` y `repuestos = repuestoEJB.findAll()`.
- Bean auxiliar de plantilla: `plantillaController` (`#{plantillaController}`, `@ViewScoped`) — `src/main/java/com/cmi/controlador/PlantillaController.java:16-18` (la clase se llama `plantillaController` en minúscula).

### Resolución de expresiones EL de la pantalla
| EL | Resuelve a | Evidencia |
|----|-----------|-----------|
| `#{controladorVistaMaquina.valorIngresado}` | get/set `valorIngresado` (String) | `VistaMaquina.xhtml:22` → `ControladorVistaMaquina.java:101-107` |
| `#{controladorVistaMaquina.autoComplete}` (completeMethod) | `autoComplete(String)` | `VistaMaquina.xhtml:24` → `ControladorVistaMaquina.java:87-94` |
| `#{controladorVistaMaquina.buscarDatosMaquina()}` | `buscarDatosMaquina()` | `VistaMaquina.xhtml:29` → `ControladorVistaMaquina.java:96-99` |
| `#{controladorVistaMaquina.contratoActual.contratoid.empresa}` | `MaquinaContrato.contratoid` → `Contrato.empresa` | `VistaMaquina.xhtml:43-47` → `ControladorVistaMaquina.java:53`, `MaquinaContrato.java:50`, `Contrato.java:45` |
| `#{controladorVistaMaquina.contratoActual.contratoid.contratoMaestroid.nombreEmpresa}` | `Contrato.contratoMaestroid` → `Contratomaestro.nombreEmpresa` | `VistaMaquina.xhtml:54-59` → `Contrato.java:83`, `Contratomaestro.java:39` |
| `#{controladorVistaMaquina.maquinaBuscada.modeloid.descripcion}` | `Maquina.modeloid` → `Modelo.descripcion` | `VistaMaquina.xhtml:69-72` → `Maquina.java:61`, `Modelo.java:38` |
| `#{controladorVistaMaquina.maquinaBuscada.zonaid.descripcion}` | `Maquina.zonaid` → `Zona.descripcion` | `VistaMaquina.xhtml:79-81` → `Maquina.java:65`, `Zona.java:39` |
| `#{controladorVistaMaquina.maquinaBuscada.marcaid.descripcion}` | `Maquina.marcaid` → `Marca.descripcion` | `VistaMaquina.xhtml:91-93` → `Maquina.java:57`, `Marca.java:37` |
| `#{controladorVistaMaquina.contratoActual.ubicacion}` | `MaquinaContrato.ubicacion` | `VistaMaquina.xhtml:102-104` → `MaquinaContrato.java:46` |
| `#{controladorVistaMaquina.contratoActual.contadorList}` | `MaquinaContrato.contadorList` (`@OneToMany` lazy) | `VistaMaquina.xhtml:116-119` → `MaquinaContrato.java:60-61` |
| `#{contador.anomes}`, `#{contador.cortemensual}` | `Contador.anomes` (col `ANOMES`), `Contador.cortemensual` (col `Corte_mensual`) | `VistaMaquina.xhtml:121,124` → `Contador.java:62-63,42-43` |
| `#{controladorVistaMaquina.maquinaBuscada.mantenimientoList}` | `Maquina.mantenimientoList` | `VistaMaquina.xhtml:133` → `Maquina.java:52-53` |
| `#{mantenimiento.fechaMantenimiento}`, `#{mantenimiento.observacion}` | `Mantenimiento.fechaMantenimiento`, `.observacion` | `VistaMaquina.xhtml:135,140` → `Mantenimiento.java:40,44` |
| `#{controladorVistaMaquina.maquinaBuscada.historialRepuestosList}` | `Maquina.historialRepuestosList` | `VistaMaquina.xhtml:153-155` → `Maquina.java:67-68` |
| `#{controladorVistaMaquina.repuestosFiltrados}` (filteredValue) | get/set `repuestosFiltrados` (declarado `List<Respuesto>`) | `VistaMaquina.xhtml:158` → `ControladorVistaMaquina.java:49,69-75` |
| `#{repuesto.fecha}`, `#{repuesto.usuario}`, `#{repuesto.contador}`, `#{repuesto.nota}` (var de la tabla = `HistorialRepuestos`) | campos de `HistorialRepuestos` | `VistaMaquina.xhtml:160,187,192,197` → `HistorialRepuestos.java:40,48,61,44` |
| `#{repuesto.respuestoidRespuesto.descripcion}` (value, sortBy, filterBy) | `HistorialRepuestos.respuestoidRespuesto` → `Respuesto.descripcion` | `VistaMaquina.xhtml:169-172,181-183` → `HistorialRepuestos.java:56`, `Respuesto.java:45` |
| `#{controladorVistaMaquina.nombreRepuestos}` | `getNombreRepuestos()` (getter con lógica) | `VistaMaquina.xhtml:178` → `ControladorVistaMaquina.java:160-165` |
| `#{controladorVistaMaquina.repuesto.idRespuesto}` | `Respuesto.idRespuesto` del campo `repuesto` | `VistaMaquina.xhtml:208-210` → `ControladorVistaMaquina.java:45,117-123`, `Respuesto.java:35` |
| `#{controladorVistaMaquina.listadoRepuestos}` | `getListadoRepuestos()` (getter con lógica + acceso a BD) | `VistaMaquina.xhtml:213-214` → `ControladorVistaMaquina.java:125-133` |
| `#{controladorVistaMaquina.historial.nota}` / `.usuario` / `.fecha` / `.contador` | campos del `HistorialRepuestos historial` | `VistaMaquina.xhtml:217-229` → `ControladorVistaMaquina.java:28,152-158`, `HistorialRepuestos.java:40-61` |
| `#{controladorVistaMaquina.adicionarRepuesto()}` | `adicionarRepuesto()` | `VistaMaquina.xhtml:233-234` → `ControladorVistaMaquina.java:139-150` |
| `#{null}` (itemValue) | literal nulo (noSelectionOption) | `VistaMaquina.xhtml:176` |

## Actions
- `init()` (`@PostConstruct`) — disparado al crear el bean de vista — inicializa objetos vacíos y precarga todas las máquinas y todos los repuestos — `ControladorVistaMaquina.java:77-85`
- `autoComplete(String valor)` — disparado por `p:autoComplete completeMethod` en `VistaMaquina.xhtml:22-26` — filtra en memoria `listaMaquinas` por serial que empiece por el texto (case-insensitive) — `ControladorVistaMaquina.java:87-94`
- `buscarDatosMaquina()` — disparado por `p:commandButton actionListener` ("BUSCAR") en `VistaMaquina.xhtml:29` (update `datosMaquina pestana`) — busca la máquina por serial exacto y llama a `buscarContrato()` — `ControladorVistaMaquina.java:96-99`
- `buscarContrato()` — invocado internamente por `buscarDatosMaquina()` (no desde la UI) — recorre `maquinaBuscada.maquinaContratoList` y toma el primer contrato con `estadocontrato == 1` como `contratoActual` — `ControladorVistaMaquina.java:167-174`
- `getListadoRepuestos()` — disparado por `f:selectItems value` en `VistaMaquina.xhtml:213-214` — consulta todos los repuestos en cada evaluación y construye `SelectItem(id, descripcion, nombre)` — `ControladorVistaMaquina.java:125-133`
- `getNombreRepuestos()` — disparado por `f:selectItems value` del filtro de la columna "Descripcion" en `VistaMaquina.xhtml:178` — devuelve lista de descripciones de `repuestos` (cargados en `init`) — `ControladorVistaMaquina.java:160-165`
- `adicionarRepuesto()` — disparado por `p:commandButton actionListener` ("Guardar") en `VistaMaquina.xhtml:232-236` (update `mensaje @form`) — asocia máquina buscada y repuesto seleccionado al historial, lo persiste, reinicia el historial, recarga la máquina y muestra mensaje — `ControladorVistaMaquina.java:139-150`
- `plantillaController.verificarSesion()` — disparado por `f:event preRenderView` en `WEB-INF/templates/plantilla.xhtml:13` — redirige a `/index.cmi` si no hay `usuario` en sesión — `PlantillaController.java:19-27`

## Services
- `MaquinaFacade.findAll()` — heredada de `AbstractFacade` — `src/main/java/com/cmi/ejb/AbstractFacade.java:36-40`
- `MaquinaFacade.buscarPorSerial(String)` — propia — `src/main/java/com/cmi/ejb/MaquinaFacade.java:42-47` (declarada en `MaquinaFacadeLocal.java:34`)
- `RespuestoFacade.findAll()` — propia (sobrescribe la de `AbstractFacade`) — `src/main/java/com/cmi/ejb/RespuestoFacade.java:25-29`
- `HistorialRepuestosFacade.create(HistorialRepuestos)` — heredada de `AbstractFacade` — `src/main/java/com/cmi/ejb/AbstractFacade.java:20-22`

## Repository
- `MaquinaFacade.findAll()` → `createQuery(CriteriaQuery)` equivalente a `SELECT m FROM Maquina m` (Criteria genérico) — `src/main/java/com/cmi/ejb/AbstractFacade.java:36-40`
- `MaquinaFacade.buscarPorSerial(String)` → NamedQuery `Maquina.findBySerial`, `getResultList().get(0)` — `src/main/java/com/cmi/ejb/MaquinaFacade.java:43-46`
  - JPQL: `SELECT m FROM Maquina m WHERE m.serial = :serial` (`src/main/java/com/cmi/modelo/Maquina.java:29`)
- `RespuestoFacade.findAll()` → NamedQuery `Respuesto.findAll` — `src/main/java/com/cmi/ejb/RespuestoFacade.java:26-28`
  - JPQL: `SELECT r FROM Respuesto r order by r.descripcion` (`src/main/java/com/cmi/modelo/Respuesto.java:27`)
- `HistorialRepuestosFacade.create(HistorialRepuestos)` → `em.persist(entity)` — `src/main/java/com/cmi/ejb/AbstractFacade.java:20-22`; EntityManager `integralPU` (`HistorialRepuestosFacade.java:20-21`)
- Navegación lazy de relaciones (sin query explícita, JPA genera SELECT por FK al acceder desde la vista):
  - `Maquina.maquinaContratoList` (`Maquina.java:49-50`) — leída en `buscarContrato()` (`ControladorVistaMaquina.java:168`)
  - `MaquinaContrato.contadorList` (`MaquinaContrato.java:60-61`), `Maquina.mantenimientoList` (`Maquina.java:52-53`), `Maquina.historialRepuestosList` (`Maquina.java:67-68`) — leídas desde los `p:dataTable`
  - `@ManyToOne` (EAGER por defecto): `Maquina.marcaid/modeloid/zonaid`, `MaquinaContrato.contratoid`, `Contrato.contratoMaestroid`, `HistorialRepuestos.respuestoidRespuesto`

> En CMI, Services y Repository son el mismo `*Facade` EJB: Services describe la operación de negocio; Repository el acceso a datos que ejecuta.

## Database
- Tabla `maquina` ← entidad `Maquina` (`src/main/java/com/cmi/modelo/Maquina.java:25-30`, `@Cacheable(false)`) — columnas relevantes: `Maquinas_id` (PK, IDENTITY, `:36`), `Serial` (`@NotNull @Size(1,45)`, `:43-47`), `Descripcion` (`:39-41`) — relaciones: `Marca_id`→`marca.idmarca` (`:55-57`), `Modelo_id`→`modelo.idModelo` (`:59-61`), `Zona_id`→`zona.idzona` (`:63-65`); `@OneToMany` a `MaquinaContrato` (mappedBy `maquinasid`, `:49`), `Mantenimiento` (mappedBy `serialId`, `:52`), `HistorialRepuestos` (mappedBy `maquinaMaquinasid`, `:67`), todas `cascade = ALL`.
- Tabla `maquina_contrato` ← entidad `MaquinaContrato` (`src/main/java/com/cmi/modelo/MaquinaContrato.java:24-28`) — columnas relevantes: `maquina_contrato_Id` (PK, `:39`), `Estado_contrato` (int `@NotNull`, `:29-32`), `Ubicacion` (`@NotNull @Size(1,45)`, `:42-46`), `Descripcion` (`:63-67`) — relaciones: `Contrato_id`→`contrato.Contrato_id` (`:48-50`), `Idtipo_contrato`→`tipo_contrato` (`:52-54`), `Maquinas_id`→`maquina.Maquinas_id` (`:56-58`); `@OneToMany` a `Contador` (mappedBy `maquinasContratoid`, `:60-61`).
- Tabla `contrato` ← entidad `Contrato` (`src/main/java/com/cmi/modelo/Contrato.java:29`) — columnas relevantes: `Contrato_id` (PK, `:38`), `Empresa` (`@NotNull @Size(1,45)`, `:42-45`) — relaciones: `ContratoMaestro_id`→`contratomaestro.ContratoMaestro_id` (`:81-83`).
- Tabla `contratomaestro` ← entidad `Contratomaestro` (`src/main/java/com/cmi/modelo/Contratomaestro.java:23`) — columnas relevantes: `ContratoMaestro_id` (PK, `:32-33`), `NombreEmpresa` (`@NotNull @Size(1,45)`, `:36-39`).
- Tabla `contador` ← entidad `Contador` (`src/main/java/com/cmi/modelo/Contador.java:23`) — columnas relevantes: `ANOMES` (Integer, `:62-63`), `Corte_mensual` (long `@NotNull`, `:41-43`) — relaciones: `Maquinas_Contrato_id`→`maquina_contrato.maquina_contrato_Id` (`:69-71`), `Soporte_id`→`soporte` (`:65-66`).
- Tabla `mantenimiento` ← entidad `Mantenimiento` (`src/main/java/com/cmi/modelo/Mantenimiento.java:24`) — columnas relevantes: `fecha_mantenimiento` (DATE `@NotNull`, `:37-40`), `observacion` (`@Size(max=200)`, `:42-44`) — relaciones: `serial_id`→`maquina.Maquinas_id` (`:46-48`).
- Tabla `historial_repuestos` ← entidad `HistorialRepuestos` (`src/main/java/com/cmi/modelo/HistorialRepuestos.java:24-29`, `@Cacheable(false)`) — columnas relevantes: `IdRepuesto` (PK IDENTITY, `:32-36`), `fecha` (TIMESTAMP, `:38-40`), `nota` (`@Size(max=300)`, `:42-44`), `usuario` (`@Size(max=45)`, `:46-48`), `contador` (long `@NotNull`, `:58-61`) — relaciones: `maquina_Maquinas_id`→`maquina.Maquinas_id` (`:50-52`, optional=false), `respuesto_idRespuesto`→`respuesto.idRespuesto` (`:54-56`, optional=false). **Única tabla escrita por la pantalla (INSERT).**
- Tabla `respuesto` ← entidad `Respuesto` (`src/main/java/com/cmi/modelo/Respuesto.java:24-28`) — columnas relevantes: `idRespuesto` (PK, `:34-35`), `Nombre` (`@NotNull`, `:37-41`), `Descripcion` (`@Size(max=45)`, `:43-45`) — relaciones: `categotia_repuestos_idcategotia_repuestos`→`CategotiaRepuestos` (`:47-49`).
- Tablas `modelo` (`Modelo.java:24`, col `Descripcion` `:37-38`), `zona` (`Zona.java:23`, col `descripcion` `:38-39`), `marca` (`Marca.java:21`, col `descripcion` `:36-37`) — solo lectura para mostrar descripción.
- Persistencia: `persistence.xml` con `eclipselink.cache.shared.default=false` y `shared-cache-mode NONE` (`src/main/resources/META-INF/persistence.xml:28,31`); además `javax.persistence.schema-generation.database.action=create` (`persistence.xml:27`).

## Validation
- `Contador` obligatorio en la pestaña Instalar — origen: xhtml `required="true"` en `p:inputNumber` (sin `requiredMessage`, se usa el mensaje por defecto de JSF) — `VistaMaquina.xhtml:228-231`
- `Contador` sin decimales — origen: xhtml `decimalPlaces="0"` (restricción de formato de entrada) — `VistaMaquina.xhtml:229`
- `HistorialRepuestos.contador` `@NotNull` (primitivo `long`) — origen: Bean Validation — `HistorialRepuestos.java:58-61`
- `HistorialRepuestos.nota` `@Size(max = 300)` — origen: Bean Validation — `HistorialRepuestos.java:42-44`
- `HistorialRepuestos.usuario` `@Size(max = 45)` — origen: Bean Validation — `HistorialRepuestos.java:46-48`
- `HistorialRepuestos.maquinaMaquinasid` / `respuestoidRespuesto` `@ManyToOne(optional = false)` — origen: mapeo JPA (no hay validación previa en el bean) — `HistorialRepuestos.java:50-56`
- No hay validación en el bean de: máquina buscada antes de guardar, repuesto seleccionado, fecha, ni serial encontrado — origen: ausencia de `if` en `ControladorVistaMaquina.java:96-99,139-150`
- Insumo: la opción `--Seleccione---` tiene `itemValue=""` y el `selectOneMenu` no es `required` — `VistaMaquina.xhtml:207-212`
- Errores de persistencia: capturados de forma genérica por `catch (Exception e)` → mensaje FATAL "No se pudo insertar" — `ControladorVistaMaquina.java:147-148`

## Navigation
- `VistaMaquina.xhtml` → (misma vista) — mecanismo: ajax `actionListener` sin outcome (BUSCAR y Guardar solo actualizan componentes) — `VistaMaquina.xhtml:29,232-236`
- Cualquier vista con plantilla → `/index.cmi` si no hay sesión — mecanismo: `ExternalContext.redirect` en `preRenderView` — `PlantillaController.java:21-24`, `plantilla.xhtml:13`
- Menú superior → otras pantallas — mecanismo: menú BD (`#{menuControler.modelo}`, tabla `Menu`) — `plantilla.xhtml:23`
- Botón "Cerrar Sesion" (plantilla) → `/index?faces-redicrect=true` — mecanismo: outcome (con typo `redicrect`) — `plantilla.xhtml:25`
- Llegada a `VistaMaquina.cmi`: no se encontró ninguna referencia en código fuente (ni `h:link`, ni outcome); por convención de CMI la URL provendría de la tabla `Menu` vía `menuControler` — NO ENCONTRADO en código (ver Open questions).

## PrimeFaces components
- `p:messages` (`id="mensaje"`, `autoUpdate="true"`, detail+summary) — mostrar mensajes FacesMessage — `VistaMaquina.xhtml:14`
- `p:separator` — separadores visuales — `VistaMaquina.xhtml:15,34`
- `p:panelGrid` (`principalBuscar`) + `p:row`/`p:column` — cabecera de búsqueda — `VistaMaquina.xhtml:17-33`
- `p:outputLabel` — etiquetas y valores de solo lectura — `VistaMaquina.xhtml:20,39-105,206-227`
- `p:autoComplete` (`maxResults="12"`) — entrada del serial con sugerencias — `VistaMaquina.xhtml:22-26`
- `p:commandButton` "BUSCAR" — ejecutar búsqueda (ajax, update `datosMaquina pestana`) — `VistaMaquina.xhtml:29`
- `p:fieldset` "Datos de la maquina buscada" + `p:panelGrid id="datosMaquina"` — ficha de datos de la máquina/contrato — `VistaMaquina.xhtml:35-109`
- `p:tabView id="pestana"` con `p:tab` "Contadores", "Mantenimientos", "Historial", "Instalar" — organización del detalle — `VistaMaquina.xhtml:110-242`
- `p:dataTable` contadores (rows 10, paginator) — `VistaMaquina.xhtml:113-126`
- `p:dataTable` mantenimientos (rows 10, paginator, fecha con `f:convertDateTime dd/MM/yyyy America/Bogota`) — `VistaMaquina.xhtml:130-144`
- `p:dataTable` historial repuestos (`widgetVar="Repuestos"`, rows 5, `rowsPerPageTemplate 5,10,15`, `emptyMessage`, `filteredValue`, sort/filter por descripción) — `VistaMaquina.xhtml:147-201`
- `p:selectOneMenu` en `f:facet name="filter"` con `onchange="PF('Repuestos').filter()"` — filtro por descripción del repuesto — `VistaMaquina.xhtml:173-180`
- `p:selectOneMenu id="mesCorte"` — selección del insumo a instalar (id heredado/no semántico) — `VistaMaquina.xhtml:207-215`
- `p:inputTextarea` — nota — `VistaMaquina.xhtml:217-218`
- `p:inputText` (placeholder "usuario") — usuario solicitante — `VistaMaquina.xhtml:220-222`
- `p:calendar id="idfechaRepuesto"` — fecha de instalación — `VistaMaquina.xhtml:224-226`
- `p:inputNumber id="idContadorRepuesto"` (`decimalPlaces="0"`, `required`) — contador — `VistaMaquina.xhtml:228-231`
- `p:commandButton` "Guardar" — registrar instalación (update `mensaje @form`) — `VistaMaquina.xhtml:232-236`

## Observed behavior
- Al iniciar la vista se cargan en memoria todas las máquinas (`findAll`) y todos los repuestos ordenados por descripción.
  - Tipo: otro (carga inicial)
  - Evidence: `src/main/java/com/cmi/controlador/ControladorVistaMaquina.java:83-84`, `Respuesto.java:27`
- El autocompletado sugiere seriales cuyo valor comienza por el texto ingresado, ignorando mayúsculas/minúsculas, sobre la lista precargada (no consulta BD); la UI limita a 12 resultados.
  - Tipo: filtro
  - Evidence: `ControladorVistaMaquina.java:87-94`, `VistaMaquina.xhtml:25`
- La búsqueda usa igualdad exacta de serial (`m.serial = :serial`) y toma el primer resultado de la lista si hay varios.
  - Tipo: filtro
  - Evidence: `MaquinaFacade.java:43-46`, `Maquina.java:29`
- Si el serial no existe, `getResultList().get(0)` lanza `IndexOutOfBoundsException`; el bean no lo captura (no hay try/catch en `buscarDatosMaquina`).
  - Tipo: otro (manejo de error)
  - Evidence: `MaquinaFacade.java:46`, `ControladorVistaMaquina.java:96-99`
- El "contrato actual" mostrado es el primer registro de `maquina_contrato` de la máquina con `Estado_contrato == 1`, según el orden en que venga la colección (sin `@OrderBy`).
  - Tipo: condición
  - Evidence: `ControladorVistaMaquina.java:167-174`, `Maquina.java:49-50`
- Si la máquina no tiene ningún contrato con estado 1, `contratoActual` no se reinicia: conserva el valor de una búsqueda anterior (o `null` en la primera).
  - Tipo: condición
  - Evidence: `ControladorVistaMaquina.java:167-174` (no hay asignación fuera del `if`)
- La pestaña "Contadores" muestra los contadores del contrato activo (no de todos los contratos de la máquina); "Mantenimientos" e "Historial" muestran los de la máquina (independientes del contrato).
  - Tipo: filtro
  - Evidence: `VistaMaquina.xhtml:116-119,133,153-155`
- Las fechas de mantenimiento e historial se muestran en formato `dd/MM/yyyy` zona `America/Bogota`.
  - Tipo: otro (formato)
  - Evidence: `VistaMaquina.xhtml:136,161-165`
- El historial de repuestos se puede ordenar y filtrar por descripción del repuesto; las opciones del filtro salen de las descripciones de todos los repuestos del catálogo.
  - Tipo: filtro
  - Evidence: `VistaMaquina.xhtml:168-180`, `ControladorVistaMaquina.java:160-165`
- El combo de insumos se recarga de BD en cada evaluación del getter; muestra `descripcion` como etiqueta, `idRespuesto` como valor y `nombre` como descripción (tooltip).
  - Tipo: otro
  - Evidence: `ControladorVistaMaquina.java:125-133`
- Al guardar, se crea un registro en `historial_repuestos` vinculado a la máquina buscada y al repuesto seleccionado (objeto `Respuesto` transitorio con solo `idRespuesto` poblado), con nota, usuario, fecha y contador tecleados por el usuario.
  - Tipo: cambio de estado
  - Evidence: `ControladorVistaMaquina.java:139-143`, `VistaMaquina.xhtml:208-229`
- El campo "Usuario solicitante" es texto libre; no se toma del usuario en sesión.
  - Tipo: otro
  - Evidence: `VistaMaquina.xhtml:219-222`, `ControladorVistaMaquina.java:139-150` (no lee `sessionMap`)
- Tras guardar con éxito: se reinicia `historial`, se vuelve a buscar la máquina por `valorIngresado` (refresca tablas) y se muestra INFO "AVISO" / "Se Inserto correctamente el repuesto". El campo `repuesto` no se reinicia.
  - Tipo: mensaje
  - Evidence: `ControladorVistaMaquina.java:144-146`
- Ante cualquier excepción al guardar se muestra FATAL "ERROR" / "No se pudo insertar" (sin detalle de causa).
  - Tipo: mensaje
  - Evidence: `ControladorVistaMaquina.java:147-148`
- El contador es obligatorio y entero en la UI.
  - Tipo: validación
  - Evidence: `VistaMaquina.xhtml:228-231`
- Todas las vistas con plantilla redirigen a `/index.cmi` si no existe `usuario` en sesión.
  - Tipo: condición
  - Evidence: `PlantillaController.java:19-27`, `plantilla.xhtml:13`

## Inferred business rules
- Una máquina se identifica funcionalmente por su serial para consulta.
  - Derived from: búsqueda por `m.serial = :serial` y autocompletado por serial; `Serial` `@NotNull`.
  - Evidence: `Maquina.java:29,43-47`, `MaquinaFacade.java:43-46`
  - Confidence: MEDIUM (no hay restricción de unicidad visible en la entidad; el código toma el primero)
  - Status: `REQUIRES_FUNCTIONAL_VALIDATION`
- Una máquina tiene como máximo un contrato vigente a la vez, identificado por `Estado_contrato = 1` (activo), y es el que se muestra en la ficha.
  - Derived from: `buscarContrato()` toma el primer `MaquinaContrato` con estado 1; la NamedQuery `MaquinaContrato.buscarPorEmpresa` también filtra `estadocontrato=1`.
  - Evidence: `ControladorVistaMaquina.java:167-174`, `MaquinaContrato.java:27`
  - Confidence: MEDIUM (el significado del valor 1 no está documentado; la unicidad no se garantiza en código)
  - Status: `REQUIRES_FUNCTIONAL_VALIDATION`
- Los contadores (lecturas mensuales) pertenecen a la relación máquina-contrato, no a la máquina; por eso solo se muestran los del contrato activo.
  - Derived from: `Contador.maquinasContratoid` y tabla alimentada desde `contratoActual.contadorList`.
  - Evidence: `Contador.java:69-71`, `VistaMaquina.xhtml:116-119`
  - Confidence: MEDIUM
  - Status: `REQUIRES_FUNCTIONAL_VALIDATION`
- Toda instalación de un repuesto/insumo en una máquina debe registrar la lectura del contador en ese momento.
  - Derived from: `required="true"` en el contador y `contador` `@NotNull` en `historial_repuestos`.
  - Evidence: `VistaMaquina.xhtml:230`, `HistorialRepuestos.java:58-61`
  - Confidence: MEDIUM
  - Status: `REQUIRES_FUNCTIONAL_VALIDATION`
- Toda instalación debe estar asociada a una máquina y a un repuesto del catálogo.
  - Derived from: `@ManyToOne(optional = false)` en ambas FKs de `HistorialRepuestos`.
  - Evidence: `HistorialRepuestos.java:50-56`
  - Confidence: HIGH (restricción de modelo), aunque la UI no lo valida antes de guardar
  - Status: `REQUIRES_FUNCTIONAL_VALIDATION`
- El historial de repuestos registra quién solicitó la instalación (usuario solicitante), no necesariamente quién la registró en el sistema.
  - Derived from: etiqueta "Usuario solicitante" con texto libre; no se usa el usuario de sesión.
  - Evidence: `VistaMaquina.xhtml:219-222`
  - Confidence: LOW
  - Status: `REQUIRES_FUNCTIONAL_VALIDATION`
- La consulta de máquinas y el registro de instalaciones requieren usuario autenticado.
  - Derived from: `verificarSesion()` en `preRenderView` de la plantilla.
  - Evidence: `PlantillaController.java:19-27`, `plantilla.xhtml:13`
  - Confidence: HIGH (mecanismo), aunque no hay control de rol/perfil en esta pantalla
  - Status: `REQUIRES_FUNCTIONAL_VALIDATION`

## Dependencies
- `FacesContext` — `addMessage` para mensajes de éxito/error — `ControladorVistaMaquina.java:146,148`
- `sessionMap["usuario"]` (tipo `Login`) — verificado por la plantilla, no usado por el bean de la pantalla — `PlantillaController.java:21`
- Bean `plantillaController` — control de sesión vía plantilla — `plantilla.xhtml:13`
- Bean `menuControler` — menú superior y cierre de sesión (fuera de la cadena principal, no analizado en profundidad) — `plantilla.xhtml:23,25`
- EJBs `MaquinaFacadeLocal`, `RespuestoFacadeLocal`, `HistorialRepuestosFacadeLocal` — `ControladorVistaMaquina.java:30-43`
- Persistence unit `integralPU` — `MaquinaFacade.java:19`, `RespuestoFacade.java:17`, `HistorialRepuestosFacade.java:20`
- `javax.faces.model.SelectItem` — combos — `ControladorVistaMaquina.java:17,129`
- Reportes `.jrxml`: N/A — la pantalla no genera reportes.
- JS propio: N/A — solo `PF('Repuestos').filter()` de PrimeFaces — `VistaMaquina.xhtml:174`

## Open questions
1. **Serial inexistente**: `buscarPorSerial` hace `getResultList().get(0)` sin comprobar vacío (`MaquinaFacade.java:46`); `buscarDatosMaquina` no captura la excepción (`ControladorVistaMaquina.java:96-99`). ¿Qué ve el usuario? (probablemente error ajax/sin mensaje). Posible bug.
2. **Serial duplicado**: no hay restricción de unicidad visible en `Maquina.serial` (`Maquina.java:43-47`); si hay duplicados se muestra el primero de forma no determinista. ¿Es el serial único en BD?
3. **Semántica de `Estado_contrato = 1`**: no hay constante ni enum que lo documente (`ControladorVistaMaquina.java:169`). ¿Qué otros valores existen? ¿Puede haber más de un contrato con estado 1 por máquina?
4. **`contratoActual` obsoleto**: si la máquina buscada no tiene contrato activo, se sigue mostrando el contrato (empresa, maestro, ubicación, contadores) de la búsqueda anterior (`ControladorVistaMaquina.java:167-174`). Posible bug.
5. **Guardar sin máquina buscada**: `maquinaBuscada` se inicializa como `new Maquina()` transitoria (`:82`); guardar sin buscar produciría un error de persistencia capturado como "No se pudo insertar". Tampoco se valida que se haya elegido insumo (`itemValue=""`, no `required`, `VistaMaquina.xhtml:207-212`).
6. **`Respuesto` transitorio en la relación**: `historial.setRespuestoidRespuesto(this.repuesto)` usa un `Respuesto` con solo `idRespuesto` (`ControladorVistaMaquina.java:142`), sin `find`. El `persist` probablemente funciona por FK, pero no se ha verificado el comportamiento de EclipseLink; además `repuesto` no se reinicia tras guardar (`:144`), y tras el refresco la fila del historial recién insertada podría mostrar descripción vacía si se resuelve con ese objeto transitorio.
7. **`repuestosFiltrados` con tipo incorrecto**: está declarado `List<Respuesto>` (`ControladorVistaMaquina.java:49`) pero la tabla filtra elementos `HistorialRepuestos` (`VistaMaquina.xhtml:151-158`). Funciona por borrado de genéricos, pero el tipo es engañoso. Además `filteredValue` no se reinicia al buscar otra máquina.
8. **Rendimiento**: `getListadoRepuestos()` consulta BD en cada evaluación del getter (`ControladorVistaMaquina.java:126`); `init()` carga todas las máquinas en memoria para el autocompletado (`:83`), y la lista no se refresca durante la vida de la vista.
9. **Filtro por descripción**: el filtro usa `descripcion` de `Respuesto`, que es opcional (`@Size(max=45)`, sin `@NotNull`, `Respuesto.java:43-45`); repuestos sin descripción aparecen como opción nula. `getNombreRepuestos()` hace cast `(ArrayList<String>)` de un `List` (`:161-164`), válido solo porque se instancia como `ArrayList`.
10. **Stylesheet ignorado y con typo**: `h:outputStylesheet name="../resoruces/css/index.css"` está fuera de `ui:composition` (`VistaMaquina.xhtml:8-10`), así que Facelets lo descarta; además la ruta tiene el typo `resoruces`.
11. **Typos en UI**: etiqueta "Ubicaion" (`VistaMaquina.xhtml:98`); id `mesCorte` para el combo de insumos (`:207`), aparentemente copiado de otra pantalla.
12. **Zona horaria**: la fecha de instalación se guarda como TIMESTAMP (`HistorialRepuestos.java:38-40`) desde un `p:calendar` sin `timeZone` (`VistaMaquina.xhtml:224-226`), pero se muestra con `timeZone="America/Bogota"` (`:164-165`); posible desfase de día. No verificado.
13. **Acceso a la pantalla**: NO ENCONTRADO en código fuente ningún enlace a `VistaMaquina.cmi`; se asume que la URL está en la tabla `Menu` (datos en BD, no disponibles en el repo).
14. **Sin control de autorización por rol**: solo se verifica que exista sesión (`PlantillaController.java:22`); no hay chequeo de perfil para registrar instalaciones. ¿Es intencional?
15. **Fuera de la cadena, observado**: `MaquinaFacade.buscarPorEmpresa` usa la NamedQuery `"Maquina.maquinasPorEmpresa"` (`MaquinaFacade.java:32`), pero la definida es `"Maquinas.maquinasPorEmpresa"` (`Maquina.java:29`) y no usa `:contratoid`. No afecta a esta pantalla; posible bug en otras.
16. **`persistence.xml`** tiene `javax.persistence.schema-generation.database.action=create` (`persistence.xml:27`). No afecta al flujo, pero conviene confirmar su efecto en el entorno productivo.
17. **`HistorialRepuestosFacade.refresh()`** llama `em.refresh(HistorialRepuestos.class)` (pasa una clase, no una entidad; `HistorialRepuestosFacade.java:32-38`). No se usa en esta pantalla; posible código muerto/defectuoso.

## Evidence trail
1. Thought: punto de partida es la pantalla → Action: `Read src/main/webapp/protegido/VistaMaquina.xhtml` → Observation: template `plantilla.xhtml`, un único bean `controladorVistaMaquina`, 2 acciones (BUSCAR, Guardar), autoComplete, 4 pestañas con 3 dataTables y formulario de instalación.
2. Thought: resolver el nombre EL → Action: `Grep @Named\(.*controladorVistaMaquina en src/main/java` → Observation: `ControladorVistaMaquina.java:21`.
3. Thought: analizar el bean → Action: `Read ControladorVistaMaquina.java` → Observation: `@ViewScoped`, 3 EJBs, `init`, `autoComplete`, `buscarDatosMaquina`, `buscarContrato` (estado 1), getters con lógica `getListadoRepuestos`/`getNombreRepuestos`, `adicionarRepuesto` con try/catch y mensajes.
4. Thought: localizar Facades y entidades → Action: `Glob ejb/{Maquina,HistorialRepuestos,Respuesto,Abstract}Facade*.java` y `Glob modelo/*.java` → Observation: existen todos; entidad de contrato maestro se llama `Contratomaestro`.
5. Thought: trazar Facades → Action: `Read MaquinaFacade.java`, `HistorialRepuestosFacade.java`, `RespuestoFacade.java`, `AbstractFacade.java` → Observation: `buscarPorSerial` usa `Maquina.findBySerial` + `get(0)`; `RespuestoFacade.findAll` sobrescrito con `Respuesto.findAll`; `create` y `findAll` de máquina heredados; `buscarPorEmpresa` referencia NamedQuery con nombre distinto al definido.
6. Thought: transcribir JPQL y mapeos → Action: `Read Maquina.java`, `HistorialRepuestos.java`, `Respuesto.java`, `MaquinaContrato.java` → Observation: JPQL de `Maquina.findBySerial` y `Respuesto.findAll`; tablas `maquina`, `historial_repuestos`, `respuesto`, `maquina_contrato`; FKs y Bean Validation.
7. Thought: resolver entidades de la cadena de navegación EL → Action: `Grep @Table|@Column|@JoinColumn|... en {Contrato,Contratomaestro,Contador,Mantenimiento,Modelo,Zona,Marca}.java` → Observation: tablas y columnas `Empresa`, `NombreEmpresa`, `ANOMES`, `Corte_mensual`, `fecha_mantenimiento`, `observacion`, descripciones de modelo/zona/marca.
8. Thought: cómo se llega a la pantalla → Action: `Grep VistaMaquina en src` → Observation: solo aparece en el propio xhtml y el bean; no hay enlaces en código → menú BD (asumido).
9. Thought: confirmar firma de interfaz → Action: `Read MaquinaFacadeLocal.java` → Observation: `buscarPorSerial(String)` declarado (`:34`).
10. Thought: dependencias de la plantilla (sesión) → Action: `Read PlantillaController.java`, `Grep plantillaController|menuControler|f:event en plantilla.xhtml` → Observation: `preRenderView` → `verificarSesion()` redirige a `/index.cmi` si falta `usuario`; menú `#{menuControler.modelo}`; outcome de logout con typo `faces-redicrect`.
11. Thought: mapeo de URL → Action: `Grep url-pattern en web.xml` → Observation: `*.cmi` (`web.xml:18`).
12. Thought: comportamiento de caché relevante para el refresco tras guardar → Action: `Grep cache|property en persistence.xml` → Observation: caché compartida desactivada (`:28,31`), `schema-generation create` (`:27`).
