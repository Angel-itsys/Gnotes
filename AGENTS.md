# Contrato de ejecución autónoma

Este archivo es obligatorio para el coordinador y todos los agentes de implementación.
La fuente de verdad del backlog es Notion, data source
`3cf5320c-26ad-80bf-8c85-000b83b1d597`. La arquitectura y los criterios de
aceptación de cada página de Notion delimitan el alcance de cada tarea.

## Roles y worktrees

| Rol | Dificultades permitidas | Worktree | Rama |
| --- | --- | --- | --- |
| Coordinador | No implementa producto | `/home/angel/Documentos/proyect1` | `main` |
| Implementador Hard/Standard | `hard`, `standard` | `/home/angel/Documentos/proyect1-hard-standard` | `agent/hard-standard` |
| Implementador Imposible | `imposible` | `/home/angel/Documentos/proyect1-imposible` | `agent/imposible` |

- Un implementador modifica exclusivamente su propio worktree y su propia rama.
- `main` solo lo modifica el coordinador al integrar una entrega aceptada.
- No usar `git reset --hard`, `git checkout --`, force-push, ni borrar trabajo de
  otro agente.
- Nunca versionar `.opencode/`, `node_modules/`, artefactos de build, secretos,
  tokens, rutas personales o resultados efímeros.

## Selección y ciclo de una tarea

1. Consultar Notion y elegir solo una tarea asignada al rol cuyo `estado` sea
   `Backlog` y cuyas dependencias estén todas en `Hecha`.
2. Leer la página completa, las fuentes de arquitectura indicadas y este archivo.
3. Cambiar únicamente esa tarea a `En curso`; no reclamar dos tareas a la vez.
4. Implementar sin ampliar alcance. Si aparece una decisión de propietario, de
   hardware o de producto no resuelta, detenerse, documentar evidencia y marcar
   `Bloqueada`; no inventarla.
5. Ejecutar todas las pruebas obligatorias de la página más las pruebas
   relevantes del código tocado.
6. Entregar al coordinador un HANDOFF estructurado. No marcar `Hecha` ni liberar
   dependencias desde un worktree de implementación.

Una tarea debe estar diseñada como una unidad cerrable. Si mezcla una entrega de
software con una validación física futura (por ejemplo, Raspberry Pi/SSD), el
coordinador debe separarla en una tarea de implementación y una de validación
operativa antes de reclamarla. No se usa un commit parcial como sustituto de una
tarea cerrada.

## Política estricta de commits

El **implementador que hizo el cambio** crea el commit en su rama; el
coordinador integra, pero no escribe ni reescribe el código de esa entrega.

Un commit solo se permite cuando se cumplen simultáneamente estas condiciones:

- La función o unidad vertical declarada en la tarea está terminada.
- Sus criterios de aceptación alcanzables están satisfechos y las pruebas pasan.
- `git diff --check` está limpio.
- La revisión propia de `git diff main...HEAD` no revela archivos ajenos ni
  cambios de alcance.
- El árbol no contiene archivos ignorados o secretos que puedan entrar al commit.

Prohibido hacer commits `WIP`, de "avance", de formateo aislado o para guardar
trabajo incompleto. Tampoco se enmienda un commit ya declarado como entrega. Si
la funcionalidad no está terminada, el agente deja un HANDOFF `BLOQUEADA` con
evidencia y no crea commit. Si una tarea contiene dos funciones independientemente
integrables, se divide primero en Notion o se documentan dos subentregas con sus
propios criterios y hashes.

Formato del mensaje:

```text
[ID-DE-TAREA] Resumen funcional en imperativo
```

## Puerta de integración

El coordinador acepta una entrega solo si recibe:

```text
HANDOFF
- tarea: <ID>
- estado: LISTA | BLOQUEADA
- commit: <hash completo o ninguno>
- alcance implementado: <lista breve>
- pruebas ejecutadas: <comando y resultado>
- criterios pendientes / bloqueos: <lista o ninguno>
- riesgos de integración: <lista o ninguno>
```

Para una entrega `LISTA`, el coordinador vuelve a ejecutar la puerta de calidad,
integra por avance rápido cuando sea posible (para preservar el commit del
implementador), actualiza Notion a `En revisión`, y después de la verificación
en `main` la marca `Hecha` con el hash final. Solo entonces habilita las tareas
dependientes. Si el remoto está autenticado, hace `git push origin main` después
de cada integración aprobada.

No se integra una entrega `BLOQUEADA`, una entrega sin hash, ni una entrega cuyo
criterio de aceptación pendiente cambie la semántica de producto.

## Paralelismo y orden

- Hard/Standard e Imposible pueden trabajar en paralelo únicamente si sus
  dependencias ya están en `Hecha` y no cambian la misma frontera o los mismos
  archivos.
- Dentro de cada worktree hay una sola tarea activa. El siguiente agente parte
  de `main` integrado, nunca de cambios no integrados del otro rol.
- El coordinador aplica el modelo y la variante indicados en Notion para cada
  despacho. No se reserva un modelo costoso para una tarea `standard` si Notion
  asigna uno más económico.
- Las validaciones físicas de Pi/SSD, red y energía se ejecutan solo en el
  hardware objetivo. Una medición de escritorio no se presenta como evidencia
  de Raspberry Pi.

## Seguridad y operación

- No añadir autenticación, modo offline, IA, microservicios ni respaldo externo
  si la tarea no lo autoriza expresamente.
- No ejecutar formateos, montajes, borrados, cambios de red/VPN o cortes de
  energía en hardware real sin una tarea operativa explícita y la autorización
  correspondiente.
- No pedir ni registrar tokens en Notion, Git o el repositorio. La autenticación
  de GitHub se configura una vez fuera de los commits.

## Instrucción de despacho

Todo agente nuevo recibe además este texto:

```text
Lee AGENTS.md y la página de Notion de tu tarea antes de modificar archivos.
Trabaja solo en tu worktree. Reclama una única tarea elegible para tu dificultad.
No crees un commit hasta terminar la unidad funcional y pasar su puerta de calidad.
Si te bloqueas, no inventes una solución: actualiza el estado y entrega HANDOFF
BLOQUEADA sin commit. Al terminar, entrega el HANDOFF exacto; no fusiones ramas,
no marques Hecha y no liberes dependencias: el coordinador lo hará.
```
