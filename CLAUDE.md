# docsAgents — Contexto compartido para todos los agentes

Este repo es el **workspace de salida** de un sistema multi‑agente (analista legacy, arquitectura, developers) que trabaja sobre la aplicación legacy **CMI**.

## Topología de repos (regla dura)

| Repo | Ruta | Permiso |
|------|------|---------|
| Legacy CMI | `D:\ProyectoMultiAgentes\CMI` | **SOLO LECTURA**. Ningún agente crea, edita ni borra archivos ahí. |
| docsAgents | `D:\ProyectoMultiAgentes\docsAgents` | Único destino de escritura para todos los outputs de agentes. |

## Stack del legacy CMI

- Java 7, Maven, empaquetado `war`, desplegado en WebLogic (`WEB-INF/weblogic.xml`).
- JSF 2.2 + **PrimeFaces 6.1** (tema `south-street`). URLs mapeadas a `*.cmi` (`web.xml`), no `*.xhtml`.
- Beans CDI: `@Named("...")` + `@ViewScoped` / `@SessionScoped`.
- EJB: los beans inyectan `@EJB XxxFacadeLocal`.
- JPA EclipseLink 2.5, persistence unit `integralPU`, datasource JTA `jdbc/integralpixel` (MySQL).
- Plantilla Facelets: `src/main/webapp/WEB-INF/templates/plantilla.xhtml` (controlada por `plantillaController`).
- Reportes JasperReports 6.1: `src/main/webapp/reportes/*.jrxml` (los `.jasper` son binarios compilados, no se leen).
- Sesión: el usuario autenticado vive en `sessionMap["usuario"]` (tipo `Login`), lo pone `indexController` y lo leen `menuControler` / `plantillaController`.
- Menú dinámico: `menuControler` construye el menú desde la entidad/tabla `Menu` (URLs en BD, no en código).

## Mapeo de capas legacy → schema de artefactos

El schema de `legacy-map.md` habla de Service y Repository separados. **En CMI no existen como capas separadas.**

| Capa del schema | En CMI |
|-----------------|--------|
| UI | `CMI/src/main/webapp/**/*.xhtml` (pantallas en `protegido/`) |
| ManagedBean | `CMI/src/main/java/com/cmi/controlador/*.java` |
| Service **y** Repository | `CMI/src/main/java/com/cmi/ejb/*Facade.java` (+ interfaz `*FacadeLocal`). CRUD genérico heredado de `AbstractFacade` (`create`, `edit`, `remove`, `find`, `findAll`, `findRange`, `count`). |
| Queries | `@NamedQuery` en las entidades; `createNamedQuery` / `createQuery` / `createNativeQuery` en los Facades |
| Database | `CMI/src/main/java/com/cmi/modelo/*.java` → `@Table(name = "...")`, `@Column`, relaciones `@ManyToOne` / `@OneToMany` / `@JoinColumn` |

### Trampas conocidas de nombres
- El nombre EL (`#{x}`) es el valor de `@Named`, **no** el nombre de la clase. Resolver siempre con Grep de `@Named("x")`.
  - `#{controladorSubReportes}` → clase `controladorReportesSub`
  - `#{controaldorCalendario}`, `#{controldorEditarSubContratos}`, `#{menuControler}` tienen typos intencionales en el código: respetarlos.
- Clases con typos: `CategotiaRepuestos`, `Respuesto`. No "corregirlos" en los artefactos; citar el nombre real.

## Convenciones de artefactos

- Legacy Analyst → `docsAgents/legacy-map/<feature-kebab-case>.legacy-map.md`
- Todo hallazgo cita su evidencia como `ruta/relativa/a/CMI:línea`.
- Lo que no se pudo encontrar se marca `NO ENCONTRADO` y va a *Open questions*. **Nunca se inventa.**
- Idioma de los artefactos: español; nombres de secciones del schema en inglés (tal como están definidos).
