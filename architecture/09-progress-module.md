# 09 — Módulo de progreso y estadísticas

## Propósito

Convertir finalizaciones explícitas de micro-metas en una representación honesta del avance, conectada con el conocimiento que aportó contexto o evidencia.

## Estructura

```text
Categoría
└── Foco
    ├── Meta A
    │   └── Revisión activa
    │       ├── Micro-meta
    │       └── Micro-meta
    └── Meta B
        └── Revisión activa
            └── Micro-meta
```

Categorías cerradas del MVP:

- Físico.
- Mental.
- Ocupacional.
- Expresivo.

Cada categoría conserva metas e historial propios. No existe una puntuación que mezcle categorías.

## Foco principal

- Solo uno puede estar activo.
- Pertenece a una categoría.
- Contiene una o varias metas.
- Tiene una frase resumida.
- Tiene una fecha y hora límite absoluta.
- Se muestra completo en Inicio y compacto en el resto de la aplicación.

## Meta y micro-meta

- Una meta agrupa micro-metas mediante una revisión activa.
- Cada micro-meta aporta una pieza de igual valor.
- Una pieza se ilumina tras un evento explícito de finalización.
- La evidencia es opcional.
- Reabrir crea un evento compensatorio y apaga la pieza en la proyección actual.

## Fórmulas

```text
progreso_meta = micro_metas_completadas / micro_metas_totales
progreso_foco = promedio(progreso_meta de metas configuradas)
```

Ejemplo:

```text
Meta A: 2/2 = 100 %
Meta B: 0/10 = 0 %
Foco: (100 % + 0 %) / 2 = 50 %
```

No se usa el total combinado de micro-metas del foco.

## Revisión del plan

Agregar, retirar o sustituir micro-metas de una meta activa:

1. Construye una revisión propuesta.
2. Calcula progreso actual y progreso resultante.
3. Muestra el cambio antes de confirmar.
4. Crea una `PlanRevision` inmutable.
5. Registra el evento y activa la revisión nueva en una transacción.

Las micro-metas continuadas deben conservar identidad. Las retiradas permanecen visibles en historial, no en el cálculo activo.

## Evidencia y relaciones

Una nota, bloque o asset puede conectarse como:

- Referencia de contexto.
- Evidencia de una meta.
- Evidencia de una micro-meta.

Ninguna de estas relaciones completa la micro-meta. La finalización requiere una acción separada desde UI o `/micro complete`.

## Temporizador

- La fecha límite pertenece solo al foco.
- El backend conserva el instante absoluto.
- El cliente calcula la visualización usando una hora de servidor sincronizada.
- Cerrar el navegador o reiniciar la Raspberry Pi no pausa el tiempo.
- La cuenta regresiva no crea escrituras cada segundo.
- Al llegar a cero, `Vencido` se deriva durante lectura.

Decisiones al vencer:

- Extender: conserva fecha anterior y registra la nueva.
- Completar: cierra el foco explícitamente.
- Abandonar: cierra sin declarar automáticamente éxito o fracaso de sus metas.

## Superficie de Progreso

- Selector de categoría.
- Lista de focos y metas de la categoría.
- Vista detallada del foco con barras segmentadas.
- Inspector de micro-meta con descripción, estado, historial y evidencias.
- Cronología de eventos.
- Gráficas pequeñas de progreso real y actividad.

## Estadísticas permitidas

- Evolución histórica del porcentaje.
- Micro-metas completadas y reabiertas por periodo.
- Evidencias vinculadas.
- Revisiones del plan.
- Cambios de foco y deadline.
- Conteos factuales de metas activas y completadas por categoría.

No se incluyen ritmo ideal, juicio de atraso, comparación entre categorías ni puntuación vital global.

## Acciones

- Crear y editar foco en borrador.
- Activar foco.
- Extender, completar o abandonar foco vencido.
- Crear meta.
- Proponer y activar revisión.
- Crear micro-meta dentro de propuesta.
- Completar y reabrir micro-meta.
- Vincular y retirar evidencia.
- Seleccionar foco principal.

## Casos límite

| Caso | Resultado |
| --- | --- |
| Meta sin micro-metas | Mostrar «configuración incompleta» y excluir del promedio. |
| Foco sin metas configuradas | No permitir activación. |
| Completar dos veces con el mismo `actionId` | Un solo evento; devolver resultado previo. |
| Reabrir una micro-meta incompleta | Rechazar como operación inválida. |
| Evidencia eliminada | Mantener evento y marcar referencia retirada. |
| Deadline vencido sin navegador abierto | Derivar `Vencido` en la próxima lectura. |
| Cambiar reloj del cliente | No alterar timestamps ni deadline del servidor. |
