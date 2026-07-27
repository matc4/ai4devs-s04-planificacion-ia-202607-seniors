# Backlog inicial — FlowSync MVP

Fuente: `docs/PRD.md` v1.0 (Q3 2026). Alcance: exclusivamente los requisitos funcionales del MVP (secciones 3.1 a 3.5). Todo lo inferido y no literal en el PRD está marcado con **(asumido)**.

## Índice de épicas

| Épica | Prefijo | Stories | Origen en el PRD |
| --- | --- | --- | --- |
| E1 · Autenticación y gestión de cuenta | AUTH | 4 | §3.1 |
| E2 · Gestión de tareas (CRUD) | TASK | 5 | §3.2 |
| E3 · Organización y filtrado | ORG | 2 | §3.3 |
| E4 · Exportación | EXP | 1 | §3.4 |
| E5 · Sincronización con Google Calendar | SYNC | 5 | §3.5 |

---

# Épica E1 · Autenticación y gestión de cuenta

---

# User Story: Registro de cuenta con email y contraseña

## ID
AUTH-01

## Historia

Como visitante sin cuenta en FlowSync

Quiero crear una cuenta con mi email y una contraseña

Para tener un espacio propio y privado donde gestionar mis tareas.

## Valor de negocio

- Es la puerta de entrada al producto: sin registro no hay activación ni medición del criterio de éxito de conexión a Google Calendar.
- Habilita el aislamiento de datos por usuario, requisito de privacidad del MVP.
- Reduce fricción inicial al no exigir verificación de email por enlace.

## Criterios de aceptación

### Escenario 1 — Registro exitoso

Dado que soy un visitante en el formulario de registro

Cuando envío un email no registrado y una contraseña de al menos 8 caracteres

Entonces se crea mi cuenta y quedo autenticado en FlowSync.

### Escenario 2 — Email ya registrado

Dado que el email que introduzco ya pertenece a una cuenta existente

Cuando envío el formulario de registro

Entonces veo un mensaje que indica que el email ya está registrado y se me ofrece ir al inicio de sesión.

### Escenario 3 — Contraseña demasiado corta

Dado que estoy en el formulario de registro

Cuando envío una contraseña de menos de 8 caracteres

Entonces veo un mensaje de validación comprensible y la cuenta no se crea.

### Escenario 4 — Email con formato inválido (asumido)

Dado que estoy en el formulario de registro

Cuando envío un valor que no tiene formato de email

Entonces veo un mensaje de validación comprensible y la cuenta no se crea.

## Reglas de negocio

- La contraseña debe tener al menos 8 caracteres.
- El email es único en el sistema e identifica la cuenta.
- No hay verificación de email por enlace en el MVP.
- Los errores de validación se muestran en lenguaje comprensible, nunca como error técnico.
- La contraseña nunca se almacena en claro (asumido).

## Dependencias

- Ninguna. Es la story raíz de la épica.

## Requerimientos técnicos

- Validación de entrada en backend con VineJS (stack fijado en PRD §5).
- Emisión de access token de `@adonisjs/auth` al completar el registro.

## Definición de terminado

- Registro funcional end-to-end en web de escritorio y móvil.
- Casos de validación (email duplicado, email inválido, contraseña corta) cubiertos por tests automatizados.
- Mensajes de error revisados en español y sin jerga técnica.
- Verificado que un usuario recién creado no accede a datos de otro usuario.

---

# User Story: Inicio de sesión

## ID
AUTH-02

## Historia

Como usuario registrado en FlowSync

Quiero iniciar sesión con mi email y contraseña

Para recuperar el acceso a mis tareas desde cualquier navegador.

## Valor de negocio

- Permite el uso recurrente del producto, condición para medir retención durante las 4 semanas del MVP.
- Sostiene la promesa de privacidad: solo el titular accede a sus tareas.

## Criterios de aceptación

### Escenario 1 — Credenciales correctas

Dado que tengo una cuenta en FlowSync

Cuando introduzco mi email y contraseña correctos

Entonces quedo autenticado y accedo a mi listado de tareas.

### Escenario 2 — Credenciales incorrectas

Dado que estoy en el formulario de inicio de sesión

Cuando introduzco una contraseña que no corresponde a mi email

Entonces veo un mensaje de credenciales inválidas y no accedo a la aplicación.

### Escenario 3 — Email no registrado

Dado que estoy en el formulario de inicio de sesión

Cuando introduzco un email que no tiene cuenta asociada

Entonces veo un mensaje de credenciales inválidas sin revelar si el email existe (asumido).

### Escenario 4 — Acceso a ruta protegida sin sesión

Dado que no tengo sesión activa

Cuando intento abrir directamente el listado de tareas

Entonces soy redirigido al inicio de sesión (asumido).

## Reglas de negocio

- La sesión se mantiene mediante tokens de acceso.
- No existe recuperación de contraseña en el MVP.
- Un usuario solo puede ver sus propios datos.

## Dependencias

- AUTH-01 (debe existir una cuenta que autenticar).

## Requerimientos técnicos

- Autenticación por access tokens de `@adonisjs/auth`.
- Protección de rutas de la SPA React según estado de sesión.

## Definición de terminado

- Login funcional en escritorio y móvil.
- Tests de credenciales válidas e inválidas en verde.
- Verificado que las rutas de datos rechazan peticiones sin token válido.

---

# User Story: Cierre de sesión

## ID
AUTH-03

## Historia

Como usuario autenticado en FlowSync

Quiero cerrar mi sesión

Para que nadie más pueda ver mis tareas desde el dispositivo que estoy usando.

## Valor de negocio

- Requisito básico de confianza y privacidad, especialmente en equipos compartidos.
- Elimina el riesgo de exposición de datos personales tras el uso.

## Criterios de aceptación

### Escenario 1 — Cierre de sesión exitoso

Dado que estoy autenticado en FlowSync

Cuando selecciono cerrar sesión

Entonces mi sesión termina y vuelvo a la pantalla de inicio de sesión.

### Escenario 2 — Token invalidado

Dado que he cerrado sesión

Cuando se reutiliza el token de acceso anterior contra la API

Entonces la petición es rechazada por falta de autenticación.

### Escenario 3 — Navegación posterior

Dado que he cerrado sesión

Cuando uso el botón "atrás" del navegador para volver al listado de tareas

Entonces no veo datos de tareas y soy redirigido al inicio de sesión (asumido).

## Reglas de negocio

- El cierre de sesión invalida el token de acceso en uso.
- Cerrar sesión no borra ni altera las tareas del usuario.

## Dependencias

- AUTH-02 (debe existir una sesión activa que cerrar).

## Requerimientos técnicos

- Revocación del access token en backend y limpieza del estado de sesión en el cliente.

## Definición de terminado

- Acción de cierre de sesión accesible desde la interfaz en escritorio y móvil.
- Test que confirma que el token revocado deja de ser válido.

---

# User Story: Pantalla de bienvenida tras el registro

## ID
AUTH-04

## Historia

Como usuario que acaba de registrarse

Quiero ver una pantalla de bienvenida que explique en una frase qué hace FlowSync y me invite a crear mi primera tarea

Para entender la propuesta de valor y empezar a usar el producto sin ayuda externa.

## Valor de negocio

- Apoya directamente el criterio de éxito del MVP de completar el flujo completo sin ayuda externa.
- Reduce el abandono inmediato tras el registro al dar un siguiente paso claro.

## Criterios de aceptación

### Escenario 1 — Bienvenida tras registro

Dado que acabo de completar el registro con éxito

Cuando entro por primera vez en la aplicación

Entonces veo la pantalla de bienvenida con la explicación en una frase y una invitación a crear mi primera tarea.

### Escenario 2 — Acción de creación desde la bienvenida

Dado que estoy en la pantalla de bienvenida

Cuando acepto la invitación a crear mi primera tarea

Entonces accedo al flujo de creación de tarea.

### Escenario 3 — No se repite en sesiones posteriores

Dado que ya he pasado por la pantalla de bienvenida

Cuando vuelvo a iniciar sesión más adelante

Entonces accedo directamente al listado de tareas y no vuelvo a ver el onboarding (asumido).

## Reglas de negocio

- El onboarding es mínimo: una frase de explicación y una invitación a la acción.
- La bienvenida no bloquea el uso de la aplicación; el usuario puede continuar sin crear la tarea (asumido).

## Dependencias

- AUTH-01 (se dispara tras el registro exitoso).
- TASK-01 (destino de la invitación a crear la primera tarea).

## Requerimientos técnicos

- Marca de onboarding ya visto asociada al usuario (asumido).

## Definición de terminado

- Pantalla implementada y responsive.
- Texto de la frase de valor validado por producto.
- Test que confirma que el onboarding aparece solo en el primer acceso.

---

# Épica E2 · Gestión de tareas (CRUD)

---

# User Story: Crear una tarea

## ID
TASK-01

## Historia

Como profesional del conocimiento usuario de FlowSync

Quiero crear una tarea indicando al menos un título, y opcionalmente descripción y fecha límite

Para registrar mis pendientes en un único sitio.

## Valor de negocio

- Es la funcionalidad núcleo del producto: sin tareas no hay valor ni sincronización que demostrar.
- La fecha límite opcional es el disparador de la propuesta diferenciadora (evento en el calendario).

## Criterios de aceptación

### Escenario 1 — Creación con solo título

Dado que estoy autenticado en FlowSync

Cuando creo una tarea indicando únicamente un título

Entonces la tarea se guarda con estado `pending` y aparece en mi listado.

### Escenario 2 — Creación con todos los campos

Dado que estoy autenticado en FlowSync

Cuando creo una tarea con título, descripción y fecha límite

Entonces la tarea se guarda con los tres valores y aparece en mi listado.

### Escenario 3 — Título vacío

Dado que estoy en el formulario de creación de tarea

Cuando intento guardar sin título

Entonces veo un mensaje de validación comprensible y la tarea no se crea.

### Escenario 4 — Aislamiento por usuario

Dado que otro usuario ha creado tareas en FlowSync

Cuando consulto mi listado de tareas

Entonces no veo ninguna tarea que no sea mía.

## Reglas de negocio

- El título es obligatorio; descripción y fecha límite son opcionales.
- Toda tarea nueva nace en estado `pending`.
- Una tarea pertenece a un único usuario; FlowSync MVP no contempla tareas compartidas.

## Dependencias

- AUTH-02 (requiere sesión activa).

## Requerimientos técnicos

- Validación de los campos de tarea con VineJS.
- Persistencia mediante Lucid ORM sobre SQLite (stack fijado en PRD §5).

## Definición de terminado

- Creación disponible en escritorio y móvil.
- Tests de creación con y sin campos opcionales, y de título vacío, en verde.
- Verificado el aislamiento de datos entre usuarios.

---

# User Story: Ver el listado de mis tareas

## ID
TASK-02

## Historia

Como usuario de FlowSync con varios pendientes

Quiero ver el listado de mis tareas ordenado de forma que lo más relevante para hoy aparezca primero

Para saber de un vistazo qué tengo que hacer.

## Valor de negocio

- Es la pantalla principal del producto y el punto de partida de cualquier otra acción.
- El orden por relevancia sustituye la revisión manual de dos herramientas, que es el problema que FlowSync resuelve.

## Criterios de aceptación

### Escenario 1 — Listado con tareas

Dado que tengo tareas creadas

Cuando abro el listado de tareas

Entonces veo mis tareas con su título, estado y fecha límite cuando la tienen.

### Escenario 2 — Orden por defecto

Dado que tengo tareas con distintas fechas límite

Cuando abro el listado sin aplicar ningún filtro

Entonces las tareas más relevantes para hoy aparecen primero, según el criterio de ordenación acordado en refinamiento.

### Escenario 3 — Rendimiento con volumen alto

Dado que tengo 200 tareas en mi cuenta

Cuando abro el listado de tareas

Entonces el listado se muestra en menos de 1 segundo.

### Escenario 4 — Uso en móvil

Dado que accedo desde el navegador de un móvil

Cuando abro el listado de tareas

Entonces la interfaz es usable y legible sin desplazamiento horizontal.

## Reglas de negocio

- El criterio exacto de ordenación se define en refinamiento; el requisito es que priorice lo relevante para "hoy".
- Solo se muestran tareas del usuario autenticado.
- Las tareas archivadas no aparecen en el listado por defecto (asumido).

## Dependencias

- TASK-01 (debe existir contenido que listar).

## Requerimientos técnicos

- Consulta paginada u optimizada para cumplir el objetivo de 1 segundo con 200 tareas (asumido).
- Interfaz React 19 + Tailwind v4 + shadcn/ui, responsive.

## Definición de terminado

- Listado funcional y responsive.
- Medición de tiempo de carga con 200 tareas documentada y por debajo de 1 segundo.
- Criterio de ordenación acordado con producto y reflejado en tests.

---

# User Story: Editar una tarea

## ID
TASK-03

## Historia

Como usuario de FlowSync

Quiero editar cualquier campo de una tarea existente

Para mantener mis pendientes actualizados cuando cambian las circunstancias.

## Valor de negocio

- Evita el patrón "borrar y recrear", que rompería la trazabilidad de la tarea y su evento asociado.
- Habilita el caso de uso más frecuente del producto: mover una fecha límite.

## Criterios de aceptación

### Escenario 1 — Edición de título y descripción

Dado que tengo una tarea creada

Cuando modifico su título y su descripción y guardo

Entonces el listado y el detalle muestran los nuevos valores.

### Escenario 2 — Edición de la fecha límite

Dado que tengo una tarea con fecha límite

Cuando cambio la fecha límite y guardo

Entonces la tarea queda persistida con la nueva fecha.

### Escenario 3 — Eliminación de la fecha límite

Dado que tengo una tarea con fecha límite

Cuando borro la fecha límite y guardo

Entonces la tarea queda sin fecha límite y sigue siendo válida.

### Escenario 4 — Título vaciado

Dado que estoy editando una tarea

Cuando borro el título y trato de guardar

Entonces veo un mensaje de validación y los cambios no se guardan.

## Reglas de negocio

- Todos los campos de la tarea son editables.
- El título sigue siendo obligatorio tras la edición.
- Un usuario solo puede editar sus propias tareas.

## Dependencias

- TASK-01 (debe existir la tarea).
- Relacionada con SYNC-03 (la propagación del cambio al calendario se cubre allí).

## Requerimientos técnicos

- Validación equivalente a la de creación mediante VineJS.

## Definición de terminado

- Edición disponible en escritorio y móvil.
- Tests de edición de cada campo y de validación de título en verde.
- Verificado que un usuario no puede editar tareas ajenas.

---

# User Story: Borrar una tarea

## ID
TASK-04

## Historia

Como usuario de FlowSync

Quiero borrar una tarea

Para quitar de mi lista lo que ya no tiene sentido mantener.

## Valor de negocio

- Mantiene la lista limpia y confiable, lo que sostiene la percepción de simplicidad del producto.
- Da al usuario control sobre sus propios datos.

## Criterios de aceptación

### Escenario 1 — Borrado exitoso

Dado que tengo una tarea en mi listado

Cuando la borro

Entonces desaparece de mi listado de forma permanente.

### Escenario 2 — Confirmación previa

Dado que solicito borrar una tarea

Cuando se me pide confirmación y cancelo

Entonces la tarea permanece intacta en mi listado (asumido).

### Escenario 3 — Aislamiento por usuario

Dado que existe una tarea de otro usuario

Cuando intento borrarla directamente contra la API

Entonces la operación es rechazada y la tarea no se borra.

## Reglas de negocio

- El borrado es una acción distinta del archivado: archivar conserva la tarea, borrar la elimina.
- Un usuario solo puede borrar sus propias tareas.

## Dependencias

- TASK-01 (debe existir la tarea).
- Relacionada con SYNC-03 (la eliminación del evento asociado se cubre allí).

## Requerimientos técnicos

- Endpoint de borrado con verificación de propiedad del recurso.

## Definición de terminado

- Borrado disponible en escritorio y móvil, con confirmación.
- Tests de borrado y de rechazo de borrado ajeno en verde.

---

# User Story: Cambiar el estado de una tarea

## ID
TASK-05

## Historia

Como usuario de FlowSync

Quiero cambiar el estado de una tarea entre pendiente, completada y archivada

Para reflejar en qué punto está cada pendiente sin tener que borrarlo.

## Valor de negocio

- Marcar como completada es la acción de mayor frecuencia y la que produce la sensación de progreso.
- El estado archivado permite retirar tareas de la vista sin perder el histórico exportable.

## Criterios de aceptación

### Escenario 1 — Marcar como completada

Dado que tengo una tarea en estado `pending`

Cuando la marco como completada

Entonces su estado pasa a `completed` y así se refleja en el listado.

### Escenario 2 — Archivar una tarea

Dado que tengo una tarea en estado `pending` o `completed`

Cuando la archivo

Entonces su estado pasa a `archived` y deja de aparecer en el listado por defecto (asumido).

### Escenario 3 — Reabrir una tarea completada

Dado que tengo una tarea en estado `completed`

Cuando la devuelvo a pendiente

Entonces su estado pasa a `pending` (asumido).

### Escenario 4 — Estado no permitido

Dado que se intenta fijar un estado distinto de `pending`, `completed` o `archived`

Cuando se envía la petición a la API

Entonces la operación es rechazada con un error de validación.

## Reglas de negocio

- Los estados válidos son exclusivamente `pending`, `completed` y `archived`.
- Cambiar de estado no altera el resto de campos de la tarea.

## Dependencias

- TASK-01 (debe existir la tarea).
- Relacionada con SYNC-03 (el efecto de completar sobre el evento se cubre allí).

## Requerimientos técnicos

- Estado modelado como valor cerrado y validado con VineJS.

## Definición de terminado

- Cambio de estado accesible con una sola acción desde el listado.
- Tests de todas las transiciones y del estado inválido en verde.

---

# Épica E3 · Organización y filtrado

---

# User Story: Filtrar tareas por estado

## ID
ORG-01

## Historia

Como usuario de FlowSync con muchas tareas activas

Quiero filtrar mis tareas por estado

Para concentrarme solo en el subconjunto que me interesa en ese momento.

## Valor de negocio

- Mantiene el producto usable en el rango real del usuario objetivo (5 a 30 tareas activas).
- Da acceso a lo completado y archivado sin ensuciar la vista principal.

## Criterios de aceptación

### Escenario 1 — Filtro por pendientes

Dado que tengo tareas en varios estados

Cuando filtro por `pending`

Entonces veo únicamente mis tareas pendientes.

### Escenario 2 — Filtro por completadas

Dado que tengo tareas en varios estados

Cuando filtro por `completed`

Entonces veo únicamente mis tareas completadas.

### Escenario 3 — Filtro por archivadas

Dado que tengo tareas archivadas

Cuando filtro por `archived`

Entonces veo únicamente mis tareas archivadas.

### Escenario 4 — Filtro sin resultados

Dado que no tengo ninguna tarea en el estado seleccionado

Cuando aplico ese filtro

Entonces veo un mensaje que indica que no hay tareas en ese estado (asumido).

## Reglas de negocio

- El filtro opera sobre los tres estados definidos.
- El filtro respeta el orden por defecto del listado.
- El filtro nunca muestra tareas de otro usuario.

## Dependencias

- TASK-02 (el filtro actúa sobre el listado).
- TASK-05 (deben existir tareas en distintos estados).

## Requerimientos técnicos

- Filtrado resuelto en la consulta de datos, no solo en el cliente, para sostener el objetivo de rendimiento (asumido).

## Definición de terminado

- Filtro visible y usable en escritorio y móvil.
- Tests por cada estado y para el caso sin resultados en verde.

---

# Job Story: Estado vacío cuando no hay tareas

## ID
ORG-02

## Historia

Cuando abro FlowSync y no tengo ninguna tarea, ya sea porque mi cuenta es nueva o porque las he archivado todas

Quiero ver un estado vacío con una invitación clara a crear la primera tarea

Para poder empezar a usar el producto sin quedarme frente a una pantalla en blanco.

## Valor de negocio

- Elimina el momento de mayor riesgo de abandono: la primera pantalla sin contenido.
- Refuerza el criterio de éxito de completar el flujo sin ayuda externa.

## Criterios de aceptación

### Escenario 1 — Cuenta nueva

Dado que acabo de registrarme y no he creado ninguna tarea

Cuando abro el listado de tareas

Entonces veo el estado vacío con la invitación a crear la primera tarea.

### Escenario 2 — Todas las tareas archivadas

Dado que todas mis tareas están archivadas

Cuando abro el listado por defecto

Entonces veo el estado vacío en lugar de una lista sin elementos.

### Escenario 3 — Acción desde el estado vacío

Dado que estoy viendo el estado vacío

Cuando acepto la invitación

Entonces accedo al flujo de creación de tarea.

### Escenario 4 — Desaparición del estado vacío

Dado que estoy viendo el estado vacío

Cuando creo mi primera tarea

Entonces el estado vacío desaparece y veo la tarea en el listado.

## Reglas de negocio

- El estado vacío del listado por defecto se muestra cuando el usuario no tiene tareas visibles en él.
- El estado vacío del listado es distinto del mensaje de "sin resultados" de un filtro concreto (asumido).

## Dependencias

- TASK-02 (se muestra dentro del listado).
- TASK-01 (destino de la invitación).

## Requerimientos técnicos

- Ninguno adicional al del listado.

## Definición de terminado

- Estado vacío implementado y responsive.
- Tests para cuenta nueva y para todas las tareas archivadas en verde.

---

# Épica E4 · Exportación

---

# User Story: Exportar mis tareas a CSV

## ID
EXP-01

## Historia

Como usuario de FlowSync

Quiero exportar mis tareas a un archivo CSV

Para poder llevarme mis datos y usarlos fuera de la aplicación.

## Valor de negocio

- Reduce la barrera de adopción al garantizar que los datos no quedan atrapados en el producto.
- Es un argumento de confianza relevante para el perfil de usuario que viene de otra herramienta.

## Criterios de aceptación

### Escenario 1 — Exportación con tareas

Dado que tengo tareas creadas

Cuando solicito la exportación

Entonces se descarga un archivo CSV que incluye, por cada tarea, título, descripción, estado y fecha límite.

### Escenario 2 — Alcance de la exportación

Dado que tengo tareas en los tres estados

Cuando solicito la exportación

Entonces el archivo incluye todas mis tareas, con independencia de su estado (asumido).

### Escenario 3 — Campos opcionales vacíos

Dado que tengo tareas sin descripción y sin fecha límite

Cuando solicito la exportación

Entonces esas columnas aparecen vacías y el archivo sigue siendo un CSV válido.

### Escenario 4 — Aislamiento por usuario

Dado que existen tareas de otros usuarios

Cuando solicito la exportación

Entonces el archivo contiene únicamente mis tareas.

## Reglas de negocio

- El CSV incluye como mínimo título, descripción, estado y fecha límite.
- Los valores con separadores, comillas o saltos de línea se escapan según el formato CSV (asumido).
- La exportación no modifica ninguna tarea.

## Dependencias

- TASK-01 (debe existir contenido que exportar).

## Requerimientos técnicos

- Generación del CSV en codificación UTF-8 y formato de fecha consistente y documentado (asumido).

## Definición de terminado

- Exportación disponible desde la interfaz en escritorio y móvil.
- Archivo verificado abriéndolo en una hoja de cálculo.
- Tests de escapado de caracteres y de campos vacíos en verde.

---

# Épica E5 · Sincronización con Google Calendar

> Nota de alcance heredada del PRD: la dirección de sincronización del MVP es FlowSync → Google Calendar. La sincronización inversa (editar el evento en Google y que se refleje en la tarea) queda fuera de estas stories a la espera del spike técnico previsto en el PRD §3.5 y §7.

---

# User Story: Conectar mi cuenta de Google

## ID
SYNC-01

## Historia

Como usuario de FlowSync que usa Google Calendar a diario

Quiero autorizar a FlowSync el acceso a mi calendario mediante mi cuenta de Google

Para que mis tareas puedan reflejarse en el calendario que ya consulto.

## Valor de negocio

- Es la puerta a la funcionalidad diferenciadora del producto y la métrica clave del MVP: al menos el 40% de los registrados deben conectar su calendario.
- Sin conexión no hay sincronización posible, por lo que bloquea toda la épica.

## Criterios de aceptación

### Escenario 1 — Conexión exitosa

Dado que estoy autenticado en FlowSync y no tengo cuenta de Google conectada

Cuando completo la autorización OAuth con Google

Entonces FlowSync muestra mi cuenta de Google como conectada.

### Escenario 2 — Autorización denegada

Dado que inicio el flujo de autorización con Google

Cuando deniego los permisos solicitados

Entonces vuelvo a FlowSync con un mensaje comprensible y sin cuenta conectada.

### Escenario 3 — Estado visible de la conexión

Dado que tengo mi cuenta de Google conectada

Cuando entro en la configuración de la conexión

Entonces veo que la conexión está activa y qué cuenta de Google está asociada (asumido).

### Escenario 4 — Privacidad de los tokens

Dado que mi cuenta de Google está conectada

Cuando se consultan los datos almacenados de mi conexión

Entonces mis tokens de Google no son accesibles por otros usuarios y no se exponen en la interfaz ni en los logs.

## Reglas de negocio

- La conexión es individual: cada usuario autoriza su propia cuenta de Google.
- Los tokens de Google se almacenan de forma segura.
- Se solicitan únicamente los permisos necesarios para leer y escribir en el calendario del usuario.

## Dependencias

- AUTH-02 (requiere sesión activa en FlowSync).
- Configuración previa del proyecto en Google Cloud Console y revisión de permisos (riesgo identificado en PRD §7).

## Requerimientos técnicos

- Flujo OAuth 2.0 contra Google Calendar API.
- Almacenamiento cifrado de los tokens y de su caducidad (asumido).

## Definición de terminado

- Flujo de conexión y de denegación probados end-to-end contra Google.
- Verificado que los tokens no aparecen en logs ni en respuestas de la API.
- Operaciones de conexión registradas en logs para diagnóstico.

---

# User Story: Reflejar mis tareas con fecha límite como eventos del calendario

## ID
SYNC-02

## Historia

Como usuario de FlowSync con la cuenta de Google conectada

Quiero que mis tareas con fecha límite aparezcan como eventos en mi Google Calendar

Para no tener que copiarlas a mano ni mirar dos sitios distintos para saber qué tengo hoy.

## Valor de negocio

- Es la propuesta de valor central del producto y la hipótesis que el MVP quiere validar.
- Elimina por completo el trabajo manual de doble gestión tarea/calendario.

## Criterios de aceptación

### Escenario 1 — Tarea con fecha límite crea evento

Dado que tengo mi cuenta de Google conectada

Cuando creo una tarea con fecha límite

Entonces aparece un evento correspondiente en mi Google Calendar con el título de la tarea.

### Escenario 2 — Tarea sin fecha límite no crea evento

Dado que tengo mi cuenta de Google conectada

Cuando creo una tarea sin fecha límite

Entonces no se crea ningún evento en mi Google Calendar.

### Escenario 3 — Sin conexión de Google no hay evento

Dado que no tengo cuenta de Google conectada

Cuando creo una tarea con fecha límite

Entonces la tarea se guarda con normalidad y no se intenta crear ningún evento.

### Escenario 4 — Zona horaria correcta

Dado que tengo mi cuenta de Google conectada

Cuando creo una tarea con fecha límite

Entonces el evento aparece en mi calendario en la fecha esperada según la regla de zona horaria acordada, sin desplazarse de día.

### Escenario 5 — Vínculo tarea-evento

Dado que una tarea ha generado un evento

Cuando consulto esa tarea en FlowSync

Entonces existe una referencia interna que la asocia de forma unívoca a su evento de calendario (asumido).

## Reglas de negocio

- Solo las tareas con fecha límite generan eventos.
- Una tarea genera como máximo un evento de calendario.
- La correspondencia entre fecha límite y hora del evento se define explícitamente antes de implementar (riesgo de zonas horarias del PRD §7).

## Dependencias

- SYNC-01 (requiere cuenta de Google conectada).
- TASK-01 (requiere la creación de tareas con fecha límite).
- Spike técnico de la API de Google recomendado en el PRD §7.

## Requerimientos técnicos

- Escritura de eventos vía Google Calendar API.
- Registro en logs de cada operación de sincronización.

## Definición de terminado

- Creación de evento verificada contra una cuenta real de Google Calendar.
- Regla de zona horaria documentada y cubierta por tests.
- Operaciones registradas en logs con información suficiente para diagnosticar fallos.

---

# User Story: Propagar los cambios de mis tareas al calendario

## ID
SYNC-03

## Historia

Como usuario de FlowSync con la cuenta de Google conectada

Quiero que al cambiar la fecha, completar o borrar una tarea su evento de calendario se actualice o se elimine

Para que mi calendario nunca muestre información desactualizada.

## Valor de negocio

- Sin propagación de cambios la sincronización pierde credibilidad y el usuario vuelve al mantenimiento manual.
- Sostiene la confianza necesaria para que el producto sustituya a la herramienta anterior.

## Criterios de aceptación

### Escenario 1 — Cambio de fecha límite

Dado que tengo una tarea con fecha límite ya reflejada como evento

Cuando cambio su fecha límite en FlowSync

Entonces el evento correspondiente se actualiza a la nueva fecha en mi Google Calendar.

### Escenario 2 — Tarea completada

Dado que tengo una tarea con evento asociado

Cuando la marco como completada en FlowSync

Entonces su evento se elimina o se marca en el calendario según la regla acordada.

### Escenario 3 — Tarea borrada

Dado que tengo una tarea con evento asociado

Cuando borro la tarea en FlowSync

Entonces su evento se elimina de mi Google Calendar.

### Escenario 4 — Fecha límite retirada

Dado que tengo una tarea con evento asociado

Cuando elimino su fecha límite en FlowSync

Entonces el evento correspondiente se elimina del calendario (asumido).

### Escenario 5 — Evento ya inexistente

Dado que el evento asociado a una tarea ya no existe en Google Calendar

Cuando actualizo esa tarea en FlowSync

Entonces la operación no rompe la aplicación y el resultado queda registrado en los logs (asumido).

## Reglas de negocio

- El tratamiento exacto de una tarea completada (eliminar el evento o marcarlo) se acuerda en refinamiento antes de implementar.
- Los cambios en campos sin reflejo en el evento no generan actualizaciones innecesarias en la API (asumido).
- La tarea es la fuente de verdad; el MVP no propaga cambios desde el calendario hacia la tarea.

## Dependencias

- SYNC-02 (debe existir el evento que actualizar).
- TASK-03, TASK-04 y TASK-05 (son los disparadores del cambio).

## Requerimientos técnicos

- Actualización y borrado de eventos vía Google Calendar API usando la referencia tarea-evento.
- Registro en logs de cada operación de sincronización.

## Definición de terminado

- Los tres disparadores (fecha, completada, borrada) verificados contra una cuenta real de Google Calendar.
- Regla de tratamiento de tareas completadas documentada.
- Tests automatizados sobre la lógica de propagación en verde.

---

# User Story: Desconectar mi cuenta de Google

## ID
SYNC-04

## Historia

Como usuario de FlowSync con la cuenta de Google conectada

Quiero desconectar mi cuenta de Google cuando quiera

Para dejar de compartir mi calendario con FlowSync sin perder mis tareas.

## Valor de negocio

- Requisito de control y confianza: saber que se puede revertir reduce la fricción para conectar en primer lugar, lo que impacta directamente en la métrica del 40%.
- Evita que el usuario tenga que borrar su cuenta para cortar el acceso al calendario.

## Criterios de aceptación

### Escenario 1 — Desconexión exitosa

Dado que tengo mi cuenta de Google conectada

Cuando la desconecto desde FlowSync

Entonces la conexión queda marcada como inactiva y FlowSync deja de sincronizar.

### Escenario 2 — Las tareas se conservan

Dado que acabo de desconectar mi cuenta de Google

Cuando abro mi listado de tareas

Entonces todas mis tareas siguen existiendo sin cambios.

### Escenario 3 — No hay nueva sincronización

Dado que he desconectado mi cuenta de Google

Cuando creo o modifico una tarea con fecha límite

Entonces no se crea ni se actualiza ningún evento en Google Calendar.

### Escenario 4 — Reconexión

Dado que he desconectado mi cuenta de Google

Cuando vuelvo a conectarla

Entonces la sincronización queda disponible de nuevo (asumido).

## Reglas de negocio

- Desconectar no borra las tareas ya creadas en FlowSync.
- Los tokens de Google dejan de ser utilizables tras la desconexión (asumido).
- El destino de los eventos ya creados en el calendario tras la desconexión se acuerda en refinamiento; el PRD solo especifica que las tareas se conservan.

## Dependencias

- SYNC-01 (debe existir una conexión activa que desconectar).

## Requerimientos técnicos

- Revocación y borrado seguro de los tokens almacenados (asumido).

## Definición de terminado

- Desconexión disponible desde la interfaz y verificada end-to-end.
- Test que confirma que tras desconectar no se emiten llamadas a la API de Google.
- Verificado que las tareas permanecen intactas.

---

# Job Story: Sincronización resiliente ante fallos de la API de Google

## ID
SYNC-05

## Historia

Cuando guardo o modifico una tarea y la API de Google no está disponible o devuelve un error

Quiero que FlowSync guarde igualmente mi tarea y reintente la sincronización más tarde

Para poder seguir trabajando sin perder datos ni tener que repetir la operación a mano.

## Valor de negocio

- Sostiene directamente el criterio de éxito del MVP: menos del 5% de operaciones de sincronización fallidas de forma no recuperable.
- Evita que un fallo externo se perciba como un fallo del producto, protegiendo la confianza en la propuesta de valor.

## Criterios de aceptación

### Escenario 1 — La tarea se guarda pese al fallo

Dado que la API de Google devuelve un error

Cuando creo o modifico una tarea con fecha límite

Entonces la tarea se guarda correctamente en FlowSync y se me informa de que la sincronización está pendiente (asumido).

### Escenario 2 — Reintento posterior exitoso

Dado que una operación de sincronización quedó pendiente por un fallo de la API

Cuando la API de Google vuelve a estar disponible

Entonces la operación se reintenta y el evento queda reflejado en el calendario.

### Escenario 3 — Límite de reintentos

Dado que una operación de sincronización falla de forma repetida

Cuando se agota la política de reintentos acordada

Entonces la operación se marca como fallida de forma no recuperable y queda registrada en los logs (asumido).

### Escenario 4 — Trazabilidad de los fallos

Dado que se ha producido un fallo de sincronización

Cuando se revisan los logs

Entonces hay información suficiente para identificar la tarea, la operación y el error devuelto por Google.

### Escenario 5 — Rate limits

Dado que la API de Google responde con un límite de peticiones excedido

Cuando FlowSync procesa el reintento

Entonces respeta la espera indicada antes de volver a intentarlo (asumido).

## Reglas de negocio

- Un fallo de sincronización nunca impide guardar la tarea en FlowSync.
- Las operaciones de sincronización se registran en logs para poder diagnosticar fallos.
- La política concreta de reintentos y su límite se acuerda en refinamiento.

## Dependencias

- SYNC-02 y SYNC-03 (son las operaciones que se reintentan).
- Spike técnico de la API de Google recomendado en el PRD §7 (rate limits y casos límite).

## Requerimientos técnicos

- Mecanismo de reintento diferido para las operaciones de sincronización (asumido).
- Logs de sincronización sin exponer tokens ni datos sensibles.

## Definición de terminado

- Comportamiento verificado simulando errores y caídas de la API de Google.
- Política de reintentos documentada.
- Logs revisados y validados como suficientes para diagnosticar un fallo real.

---

## Fuera de alcance (confirmado en el PRD §1 y §3.5)

- Equipos o tareas compartidas.
- Calendarios distintos de Google (Outlook, iCal).
- Notificaciones push o por email.
- Aplicación móvil nativa.
- Etiquetas, proyectos o subtareas.
- Recordatorios configurables propios de FlowSync.
- Sincronización inversa completa Google Calendar → FlowSync (pendiente de spike técnico).
