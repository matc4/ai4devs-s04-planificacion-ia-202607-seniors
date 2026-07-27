Eres un Product Owner senior con experiencia en aplicaciones SaaS), necesito crear user 
stories desde PRD (docs/PRD.md) de FlowSync, la cual es una aplicación para gestionar de tareas personales para profesionales del conocimiento, las tareas se pueden Calendarizar en Google Calendar.

Cada historia de usuario debe seguir estrictamente este formato exacto: 
Como [USER ROLE] quiero [WHAT] para [WHY].
Cada historia de trabajo (job story) debe seguir estrictamente este formato: Cuando [SITUATION], quiero [MOTIVATION] para poder [EXPECTED OUTCOME].
Cada historia de usuario y de trabajo debe cumplir con los criterios INVEST de la siguiente manera:
* Independiente: una funcionalidad o necesidad por historia.
* Negociable: abierta a discusión, evitar soluciones demasiado detalladas.
* Valiosa: beneficia claramente al usuario o al negocio.
* Estimable: alcance adecuado para estimar el esfuerzo.
* Pequeña: completable dentro de un sprint.
* Comprobable (Testable): que implique resultados verificables (no se requieren criterios de aceptación).
Usar un lenguaje claro y conciso, adecuado para equipos de desarrollo ágil.
Producir las historias en el formato exacto que se indica a continuación, sin numeración, viñetas, comentarios ni líneas en blanco.
Elegir entre historia de usuario o historia de trabajo para cada necesidad, según corresponda.
No desviarse de estos formatos ni agregar nada más.

Template de ejemplo de una user story:

Escribe todas las User Stories utilizando exactamente el siguiente formato.

# User Story: <Título>

## ID
<Prefijo>-<Número>

## Historia

Como <tipo de usuario>

Quiero <objetivo>

Para <beneficio>

## Valor de negocio

- ...
- ...

## Criterios de aceptación

### Escenario 1

Dado ...

Cuando ...

Entonces ...

### Escenario 2

...

## Reglas de negocio

- ...
- ...

## Dependencias

- ...

## Requerimientos técnicos

- ...

## Definición de terminado

- ...
- ...


Solo se deben crear las user stories derivadas del PRD, no inventar ningun feature adicional.
Tampoco se debe estimar tiempos ni proponer ninguna arquitectura.
Los criterios de aceptación deben estar escritos en el formato Given/When/Then, cada 
historia con 3-5 criterios verificables. 
Agrupar las stories por epicas, según tengan sentido.
En caso de que asumas algo, marca lo inferido con una marca (asumido)