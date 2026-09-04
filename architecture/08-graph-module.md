# 08 — Módulo de grafo

## Propósito

Representar relaciones existentes entre notas, bloques, metas y micro-metas. El grafo es una proyección navegable, no un almacén independiente.

## Entrada de la proyección

Una consulta de vecindad recibe conceptualmente:

```ts
type GraphQuery = {
  root: EntityRef;
  depth: number;
  nodeTypes?: EntityKind[];
  relationTypes?: string[];
  visibility: "dim" | "hide";
  limit: number;
};
```

El servidor devuelve nodos únicos, aristas tipadas, conteos de conexiones contraídas y una indicación de truncamiento.

## Tipos de nodo

- Nota.
- Bloque.
- Meta.
- Micro-meta.

Los bloques se contraen inicialmente bajo su nota para limitar ruido. Carpetas y etiquetas actúan como navegación o filtros; no son nodos permanentes del grafo principal en el MVP.

## Disposición

```text
Raíz
├── relación principal
│   ├── siguiente nivel
│   └── siguiente nivel
└── relación principal
    └── siguiente nivel

enlaces laterales ───────────────► nodos ya visibles
```

- La raíz seleccionada inicia una disposición descendente.
- Cada acción expande un solo nivel.
- Un ID aparece una sola vez en el lienzo.
- Una segunda ruta hacia el mismo ID crea una arista cruzada.
- Un ciclo apunta al nodo existente y no inicia expansión infinita.
- El usuario puede colapsar ramas sin modificar relaciones.

La librería de visualización debe permitir controlar posiciones, aristas cruzadas, navegación por teclado y virtualización o reducción de detalle. La elección concreta permanece abierta.

## Estado de vista

Es estado no semántico del cliente:

- Raíz actual.
- Nodos expandidos.
- Selección.
- Posición y zoom.
- Filtros.
- Modo atenuar u ocultar.

Cambiar este estado no crea eventos de dominio salvo que el usuario cree, edite o elimine una relación.

## Interacciones

- Clic: seleccionar y abrir inspección derecha.
- `Enter` o Abrir: mostrar detalle/editor.
- Expandir: solicitar un nivel adicional.
- Colapsar: retirar descendencia visual no necesaria.
- Crear relación: elegir origen, destino y tipo antes de despachar acción.
- Editar tipo: sustituir mediante acción validada.
- Eliminar relación: enviar a papelera o marcar eliminación lógica según el contrato del dominio.

Comandos equivalentes:

```text
/view graph [[Raíz]]
/view expand 1
/view collapse
/view focus [[Nodo]]
/rel link [[Destino]] --type depende-de
```

## Filtros

Se puede filtrar por:

- Tipo de nodo.
- Tipo y dirección de relación.
- Carpeta o descendencia de carpeta.
- Etiqueta o descendencia de etiqueta.
- Categoría de progreso.
- Estado de meta o micro-meta.

El modo predeterminado atenúa nodos no coincidentes para conservar contexto. El modo ocultar recalcula la disposición con los nodos filtrados.

Las combinaciones guardadas como colecciones inteligentes pertenecen a una versión posterior.

## Límites de rendimiento

- Aproximadamente 250 nodos visibles simultáneamente.
- Profundidad inicial de un nivel.
- Conteos de conexiones permiten decidir si expandir.
- Las etiquetas largas y previews se reducen según zoom.
- El servidor aplica límite aunque el cliente solicite más.
- Una respuesta truncada debe indicarlo; nunca aparenta ser el grafo completo.

## Consistencia

- Crear o eliminar una relación invalida vecindades afectadas.
- Completar una micro-meta cambia su estado visual, no su identidad ni sus aristas.
- Eliminar una entidad la retira de consultas activas, pero el historial puede representar una referencia retirada.
- Restaurar recupera el mismo ID y vuelve a habilitar sus conexiones válidas.

## Estados vacíos y errores

- Nodo sin relaciones: explicar cómo crear la primera conexión.
- Filtro sin resultados: conservar raíz y ofrecer limpiar filtros.
- Límite alcanzado: mostrar nodos omitidos y sugerir enfocar una rama.
- Destino eliminado: representar arista histórica solo cuando el contexto lo requiera.
- Desconexión: mantener la composición visible como estado temporal, pero bloquear mutaciones y marcar datos como no actualizables.
