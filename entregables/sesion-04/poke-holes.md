# Poke-holes — SYNC-02 "Reflejar mis tareas con fecha límite como eventos del calendario"

Story sometida a crítica: **SYNC-02** (la de AC más completos a primera vista).
Método: pedir a la IA que identifique edge cases, supuestos implícitos, escenarios faltantes y dependencias/riesgos no mencionados, sin reescribir la story.

---

## Los 5 hallazgos que llevo a la sesión

### 1. Las tareas que ya existían cuando conecto Google quedan huérfanas

La story solo cubre el disparador "**creo** una tarea con fecha límite". Pero el flujo real que el propio PRD diseña es el contrario: onboarding → crear primera tarea (AUTH-04, TASK-01) → conectar Google (SYNC-01). Es decir, el usuario típico **ya tiene tareas con fecha límite en el momento de conectar**, y ninguna de ellas genera evento con los AC actuales.

Impacta directo en la métrica de éxito del 40% de conexiones: el usuario conecta, no ve nada en su calendario, y concluye que no funciona. No hay escenario de backfill, ni decisión sobre su alcance (¿todas las tareas? ¿solo futuras? ¿solo `pending`?), ni sobre qué pasa con hasta 200 tareas golpeando la API de golpe (rate limits del §7).

### 2. El Escenario 4 no es verificable: delega en una regla que todavía no existe

"*el evento aparece en la fecha esperada **según la regla de zona horaria acordada***" es un AC que remite a un acuerdo pendiente. No se puede escribir un test contra eso — viola la **T de INVEST** de forma encubierta, y el "(asumido)" que puse en otras partes aquí no aparece porque el hueco está disfrazado de criterio.

Debajo hay tres decisiones sin tomar, ninguna explícita en el PRD:
- **La fecha límite no tiene hora.** ¿El evento es de todo el día o tiene hora y duración? El PRD §7 avisa del riesgo pero no lo resuelve, y el glosario define "fecha límite" sin componente horaria.
- **No existe zona horaria del usuario.** El §3.1 no incluye ningún campo de perfil. Hay tres fuentes candidatas —zona del navegador, zona del calendario de Google, zona fija del servidor— y elegir mal desplaza eventos un día completo, que es exactamente el fallo que el escenario dice prevenir.
- Un usuario que viaja cambia la zona del navegador pero no la de su calendario. Sin fuente de verdad declarada, el mismo dato produce dos resultados distintos.

### 3. Contradicción entre SYNC-02 y SYNC-03: el título se escribe pero nunca se actualiza

SYNC-02 establece que el evento lleva **el título de la tarea**. SYNC-03 propaga cambios de fecha, completado, borrado y retirada de fecha — **pero no el cambio de título**, y encima incluye la regla "los cambios en campos sin reflejo en el evento no generan actualizaciones innecesarias".

El título sí tiene reflejo en el evento. Resultado: renombrar una tarea deja el calendario mostrando el nombre viejo de forma permanente. Es el tipo de desincronización silenciosa que destruye la confianza en el producto, y ninguna de las dos stories lo cubre.

Mismo hueco para el estado **`archived`**: SYNC-03 contempla completar y borrar, pero archivar una tarea con evento asociado no está en ninguna parte, pese a que TASK-05 lo trata como forma habitual de retirar una tarea de la vista.

### 4. La regla de "un evento como máximo" no tiene ningún escenario que la verifique

Está declarada como regla de negocio y ahí se queda. Los caminos reales hacia el evento duplicado son varios y ninguno está cubierto:
- doble envío del formulario o doble clic;
- un reintento de SYNC-05 sobre una operación que **sí** tuvo éxito en Google pero cuya respuesta se perdió (la escritura no es idempotente por defecto);
- dos pestañas abiertas creando la misma tarea.

Duplicar eventos en el calendario de alguien es el fallo más visible y más difícil de perdonar de todo el MVP, y el backlog lo trata como una nota al margen.

### 5. Falta el estado "conectado pero sin permiso válido"

Los AC cubren dos extremos —conectado (E1) y no conectado (E3)— y SYNC-05 cubre "la API falla temporalmente". Queda fuera el caso intermedio, que en producción es frecuente:

- el usuario revocó el acceso desde su cuenta de Google (fuera de FlowSync, sin pasar por SYNC-04);
- el refresh token caducó o fue invalidado;
- el usuario concedió los scopes **parcialmente** — Google lo permite, y SYNC-01 solo contempla denegar del todo. Con permiso de solo lectura, toda escritura falla de forma permanente.

Sin distinguir *fallo transitorio* de *autorización muerta*, la política de reintentos de SYNC-05 reintentará indefinidamente algo que nunca va a funcionar, inflando artificialmente la métrica de "operaciones fallidas no recuperables" (<5%) que el MVP usa como criterio de éxito. Además no hay ningún camino en la UI para pedir reconexión.

---

## Anexo — hallazgos secundarios (reales pero de menor severidad)

**Supuestos implícitos no marcados:**

- **A qué calendario se escribe.** Se asume el calendario primario. Un knowledge worker suele tener varios (trabajo/personal/compartidos) y no hay selección ni escenario. Escribir tareas en el calendario donde están sus reuniones puede percibirse como "me ha ensuciado la agenda" — riesgo de adopción, no técnico.
- **Qué más contiene el evento.** El AC solo verifica el título. Nada dice si la descripción de la tarea viaja al evento, ni si hay un enlace de vuelta a FlowSync. Sin deep-link el usuario ve el evento pero no puede actuar sobre él, lo que erosiona el "no mirar dos sitios" del propio *para*.
- **Cuándo "aparece" el evento.** El AC no acota latencia. SYNC-05 implica procesamiento asíncrono (la tarea se guarda aunque la sync falle), pero SYNC-02 está redactada como si fuera inmediato. Las dos stories asumen modelos distintos, y el estado intermedio "sincronización pendiente" solo existe marcado como (asumido) en SYNC-05.
- **Fechas límite en el pasado.** Crear una tarea vencida genera un evento en el pasado. Google lo acepta; producto no ha decidido si tiene sentido.
- **Tareas creadas con fecha y sin conexión.** El E3 dice que no se intenta crear evento. No dice si esa tarea queda marcada como "pendiente de sincronizar" para cuando el usuario conecte (enlaza con el hallazgo 1).

**Dependencias y riesgos no declarados en la story:**

- **Cuota y rate limits de la Google Calendar API** aparecen solo dentro del spike genérico del §7, no como restricción de esta story, pese a que el volumen del §4 (hasta 200 tareas) y el backfill del hallazgo 1 los tocan de lleno.
- **Cómo se testea en CI.** La DoD exige "verificada contra una cuenta real de Google Calendar", que no es automatizable de forma estable. Falta decidir entre cuenta sandbox, mock o contract test; sin esa decisión, o los tests son frágiles o directamente no existen.
- **Aislamiento entre usuarios en la propia sincronización.** El §4 exige que los datos de un usuario no sean accesibles por otro, pero ningún AC verifica que la tarea del usuario A no pueda escribirse en el calendario de B — precisamente donde un procesamiento diferido con credenciales cruzadas podría fallar.
- **Eventos recurrentes y de todo el día**, citados como caso límite en el §7, no aparecen en ningún AC ni regla de negocio de la story.
- **Google Cloud Console**: el setup del proyecto y la revisión de permisos son prerrequisito real de poder demostrar esta story, y solo figuran como dependencia de SYNC-01.
