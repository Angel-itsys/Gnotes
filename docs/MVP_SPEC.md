# Especificación del MVP — Sistema de progresión gestionada

La arquitectura detallada por subsistema está en [`docs/architecture/README.md`](architecture/README.md).

## 1. Objetivo

Construir una aplicación web personal en la que el conocimiento almacenado en notas pueda conectarse explícitamente con metas y micro-metas. El MVP debe demostrar una porción vertical completa: crear contenido, relacionarlo con una micro-meta, registrar su finalización y observar el resultado en Inicio, Progreso e historial sin sincronizaciones manuales entre módulos.

### Invariante central

```text
UI o comando
    → acción de dominio
    → transacción SQLite
    → evento histórico
    → proyecciones de Inicio, Conocimiento y Progreso
```

Una operación equivalente desde UI y desde comando debe producir la misma acción de dominio. Ninguna vista mantiene una copia independiente del estado.

## 2. Alcance del MVP

### Incluido

- Aplicación web desktop-first con adaptación móvil básica.
- Acceso a través de Tailscale o WireGuard.
- Navegación global: Inicio, Conocimiento y Progreso.
- Editor de notas basado en bloques.
- Representación Markdown editable y sincronizada.
- Carpetas y etiquetas jerárquicas.
- Carpeta raíz administrada para adjuntos.
- Grafo semántico de notas, metas y bloques.
- Relaciones incorporadas y tipadas.
- Metas, micro-metas, foco principal, cuenta regresiva e historial.
- Acciones equivalentes mediante UI y comandos `/`.
- Papelera, revisiones del plan y snapshots locales.
- Cronología completa y un conjunto pequeño de gráficas de progreso real.

### Fuera del MVP

- Varios usuarios o colaboración.
- Autenticación dentro de la aplicación.
- Edición o consulta sin conexión.
- Colecciones inteligentes y reglas dinámicas.
- Clasificación automática mediante IA.
- Relaciones personalizadas por el usuario.
- Fechas límite por meta o micro-meta.
- Comparación de ritmo real contra ritmo ideal.
- Puntuación global que combine las cuatro categorías.
- Recuperación ante pérdida física del único SSD.
- Paridad total de edición en móvil.

## 3. Topología de despliegue

```text
Dispositivo del usuario
    │ navegador + VPN privada
    ▼
Raspberry Pi 4 · 4 GB RAM
    ├── Fastify API
    ├── cliente React/Vite
    ├── SQLite
    └── SSD SATA USB alimentado · 200 GB
        ├── base de datos
        ├── adjuntos administrados
        ├── papelera
        └── snapshots locales
```

El servidor solo debe ser accesible desde la red privada configurada. Como no existe autenticación de aplicación, cualquier dispositivo autorizado por esa red debe tratarse como poseedor de control total.

## 4. Modelo de dominio

### 4.1 Conocimiento

| Entidad | Responsabilidad |
| --- | --- |
| `Folder` | Ubicación jerárquica de una nota; tiene como máximo una carpeta padre. |
| `Note` | Documento con título, carpeta y una secuencia ordenada de bloques. |
| `Block` | Unidad estable y clasificable: texto, encabezado, imagen, enlace, código, lista, archivo o elemento estructurado. |
| `Asset` | Archivo dentro de la raíz administrada, con ruta, tipo, tamaño y checksum. |
| `Tag` | Clasificación jerárquica; tiene como máximo una etiqueta padre. |
| `TagAssignment` | Asocia una o varias etiquetas con una nota o bloque. |
| `RelationType` | Semántica y dirección de una conexión. |
| `Relation` | Conecta dos notas, metas o bloques mediante un tipo. |

Tipos de relación incorporados:

- `relacionado-con`, bidireccional.
- `referencia`, direccional hacia el contenido citado.
- `continúa`, direccional.
- `depende-de`, direccional.
- `respalda`, direccional.
- `contradice`, direccional.
- `ejemplo-de`, direccional.
- `evidencia-para`, direccional hacia una meta o micro-meta.

Las carpetas y etiquetas impiden ciclos en sus propias jerarquías. El grafo semántico sí admite ciclos y múltiples conexiones entrantes.

### 4.2 Progreso

| Entidad | Responsabilidad |
| --- | --- |
| `Category` | Uno de cuatro valores: Físico, Mental, Ocupacional o Expresivo. |
| `Focus` | Foco principal de una categoría, con resumen y fecha límite absoluta. |
| `Goal` | Meta individual perteneciente a un foco. |
| `PlanRevision` | Versión inmutable del conjunto de micro-metas de una meta. |
| `MicroGoal` | Pieza de progreso de igual peso dentro de una revisión. |
| `CompletionEvent` | Registra finalización o reversión explícita con fecha. |
| `EvidenceLink` | Relaciona una nota, bloque o archivo con una meta o micro-meta. |
| `DomainEvent` | Historial append-only de acciones relevantes del producto. |

Solo puede existir un foco principal activo. Ese foco pertenece a una única categoría, aunque contenga varias metas.

### 4.3 Cálculo del progreso

```text
progreso_meta = micro_metas_completadas / micro_metas_totales
progreso_foco = promedio(progreso_meta de cada meta del foco)
```

- Todas las micro-metas tienen el mismo peso dentro de su meta.
- Una meta sin micro-metas muestra estado incompleto y no participa en el promedio hasta que tenga una revisión válida.
- Las relaciones `referencia`, `relacionado-con` y `evidencia-para` no modifican el porcentaje.
- Completar o revertir una micro-meta crea un evento; nunca se modifica silenciosamente el evento anterior.
- Cambiar las micro-metas de una meta activa crea una nueva `PlanRevision` después de mostrar el porcentaje anterior y el resultante.

## 5. Sistema de comandos

### 5.1 Reglas

- Los comandos comienzan con `/`, ofrecen autocompletado y pueden abrir controles visuales para elegir argumentos.
- Se ejecutan únicamente mediante confirmación explícita.
- Tras ejecutarse, se transforman en un elemento visual estructurado o en una confirmación reversible.
- Texto pegado, archivos importados y bloques de código nunca ejecutan comandos automáticamente.
- Los identificadores visibles son nombres legibles; internamente se resuelven a IDs estables.
- Cada comando invoca la misma acción que su equivalente visual.

### 5.2 Superficie inicial

```text
/note create "Título"
/note move [[Carpeta]]
/tag add [[Etiqueta]]
/rel link [[Destino]] --type relacionado-con
/goal link [[Meta]]
/micro create "Descripción"
/micro complete [[Micro-meta]]
/micro reopen [[Micro-meta]]
/evidence link [[Micro-meta]] --source current-block
/focus set [[Foco]]
/view graph [[Raíz]]
/view folders
/view expand 1
/view collapse
/file attach
```

La sintaxis definitiva puede ajustarse durante implementación, pero no puede romper las reglas anteriores ni crear un camino de escritura diferente al de la UI.

## 6. Editor y Markdown

- Una nota es una secuencia ordenada de bloques con IDs estables.
- El editor visual admite atajos Markdown.
- El modo fuente es una proyección editable sincronizada, no la fuente de verdad.
- Texto, encabezados, listas, enlaces y código pueden editarse directamente en Markdown.
- Bloques estructurados conservan una sintaxis protegida y se configuran mediante UI o comandos.

Ejemplo:

```md
# Entrenamiento

Técnica para mejorar el arco.

{{goal-link id="press-banca"}}
{{evidence id="block-7f31"}}
```

Una edición que elimine o dañe sintaxis estructurada debe mostrar un error localizado y permitir volver al último estado válido; no debe guardar parcialmente la nota.

## 7. Arquitectura de información

### 7.1 Estructura global

```text
┌────────────────┬────────────────────────────┬──────────────────┐
│ Barra plegable │ Área principal             │ Panel contextual │
│                │                            │                  │
│ Inicio         │ Inicio / Grafo / Carpetas  │ Nota, bloque,    │
│ Conocimiento   │ Editor / Progreso          │ meta o relación  │
│ Progreso       │                            │ seleccionada     │
└────────────────┴────────────────────────────┴──────────────────┘
```

### 7.2 Inicio

Orden de prioridad:

1. Foco principal con frase, progreso y cuenta regresiva.
2. Próximas micro-metas.
3. Captura rápida de una nota.

En otras vistas, el foco aparece como una franja compacta y plegable.

### 7.3 Conocimiento

La barra izquierda contiene secciones plegables para notas, carpetas, etiquetas y archivos administrados. El área central alterna entre árbol de carpetas, editor y grafo.

En el grafo:

- El usuario selecciona una raíz.
- Se carga un nivel inicial y cada expansión añade un nivel.
- Un nodo puede ser nota, meta o bloque.
- Los bloques aparecen contraídos dentro de su nota hasta que se solicitan.
- Las relaciones principales crecen hacia abajo.
- Los enlaces cruzados y ciclos se dibujan lateralmente sin duplicar nodos.
- Un clic abre una inspección rápida en el panel derecho.
- `Enter` o la acción Abrir lleva al editor sin perder el contexto del grafo.
- Los filtros pueden atenuar u ocultar nodos.

### 7.4 Progreso

- Las categorías Físico, Mental, Ocupacional y Expresivo permanecen independientes.
- Cada categoría muestra sus focos, metas e historial factual.
- No existe una puntuación total de la persona.
- El foco activo muestra cada meta y sus piezas de micro-progreso.
- Una pieza se ilumina únicamente tras una finalización explícita.

## 8. Estados y transiciones

### 8.1 Foco

```text
Borrador → Activo → Completado
                  ↘ Vencido → Extender
                            → Completar
                            → Abandonar
```

- La fecha límite pertenece solamente al foco.
- El servidor calcula la cuenta regresiva a partir de una fecha y hora absolutas.
- Reiniciar el servidor o cerrar el navegador no altera el plazo.
- `Vencido` es una condición calculada, no un fracaso automático.
- Extender conserva la fecha anterior en el historial.

### 8.2 Eliminación y recuperación

- Eliminar notas, bloques, metas o relaciones los mueve a una papelera.
- Restaurar conserva sus IDs y restablece relaciones válidas.
- El periodo de retención de la papelera queda pendiente de decisión operativa.
- Los snapshots locales permiten recuperar errores lógicos amplios, pero no la pérdida física del SSD.

## 9. Estadísticas

El MVP muestra progreso real, no predicciones de ritmo.

Incluye:

- Cronología de micro-metas completadas y reabiertas.
- Evidencias vinculadas.
- Revisiones del plan.
- Cambios de foco y fecha límite.
- Evolución histórica del porcentaje.
- Actividad por semana o mes.
- Resumen factual por categoría: metas activas, completadas y actividad reciente.

No incluye:

- Línea de ritmo ideal.
- Juicios automáticos como «adelantado» o «atrasado».
- Comparación porcentual entre categorías.
- Puntuación global personal.

## 10. Archivos y almacenamiento

- La aplicación solo navega una carpeta raíz administrada; no expone todo el sistema de archivos.
- Adjuntar un archivo lo copia a esa raíz y crea un `Asset` con ID estable.
- Mover o renombrar un archivo desde la aplicación actualiza sus referencias transaccionalmente.
- La UI debe mostrar espacio usado, espacio disponible y fallos de escritura.
- Una operación que exceda el espacio disponible debe fallar antes de crear referencias parciales.
- El límite funcional confirmado para adjuntos es 50 GB.

## 11. Escala y rendimiento funcional

- Hasta 10,000 notas.
- Hasta 100,000 bloques.
- Hasta 50 GB de adjuntos.
- Aproximadamente 250 nodos visibles por vista de grafo.
- Expansión del grafo por un nivel y bajo demanda.
- Paginación o virtualización para árboles, historiales y listas extensas.
- Las escrituras que afectan varias entidades deben ejecutarse dentro de una transacción SQLite.

Los presupuestos numéricos de latencia se establecerán después de medir un prototipo en la Raspberry Pi objetivo; la implementación no debe inventarlos en desarrollo de escritorio.

## 12. Pruebas de aceptación

### A. Paridad entre comando y UI

1. Completar una micro-meta mediante la UI.
2. Reabrirla.
3. Completarla mediante `/micro complete`.
4. Verificar que ambas finalizaciones generaron el mismo tipo de acción y estructura de evento.

### B. Conexión real entre conocimiento y progreso

1. Crear una nota y un bloque de imagen.
2. Vincular el bloque como evidencia de una micro-meta.
3. Verificar que el porcentaje no cambia.
4. Completar explícitamente la micro-meta.
5. Verificar que se ilumina una pieza y se actualizan Inicio, Progreso e historial sin recargar manualmente.

### C. Revisión del plan

1. Activar una meta con tres micro-metas y completar dos.
2. Proponer una cuarta micro-meta.
3. Verificar que la UI muestra `66.7 % → 50 %` antes de confirmar.
4. Confirmar y verificar que se conserva la revisión anterior.

### D. Persistencia del temporizador

1. Configurar una fecha límite absoluta.
2. Reiniciar la aplicación y la Raspberry Pi.
3. Verificar que la cuenta regresiva se recalcula desde la misma fecha.
4. Simular vencimiento y verificar que no se completa ni abandona automáticamente.

### E. Grafo y navegación

1. Crear un ciclo entre tres nodos y una relación adicional hacia una meta.
2. Abrir uno como raíz.
3. Verificar expansión de un nivel, enlaces laterales y ausencia de nodos duplicados.
4. Seleccionar un nodo y verificar que el panel derecho abre sin cerrar el grafo.

### F. Seguridad de ejecución

1. Pegar texto que contenga `/micro complete`.
2. Abrir una nota con ese texto dentro de un bloque de código.
3. Verificar que ninguna acción se ejecuta hasta una confirmación explícita.

### G. Eliminación y restauración

1. Eliminar una nota con relaciones y evidencias.
2. Verificar que pasa a la papelera sin alterar porcentajes históricos.
3. Restaurarla y comprobar que recupera su ID y conexiones válidas.

### H. Conectividad

1. Interrumpir la conexión con la Raspberry Pi.
2. Verificar que la aplicación informa la desconexión y bloquea escrituras.
3. Confirmar que no ofrece una edición local que pueda producir conflictos posteriores.

## 13. Decisiones operativas abiertas

Estas decisiones no deben inferirse durante la implementación:

- Tailscale o WireGuard como VPN definitiva.
- Ruta exacta de la carpeta administrada.
- Frecuencia y retención de snapshots locales.
- Periodo de retención de la papelera.
- Librería concreta para el editor de bloques.
- Librería concreta para la visualización del grafo.
- Nombre e identidad visual del producto.
- Destino externo de respaldo para una versión posterior.

## 14. Criterio de éxito del MVP

El MVP está completo cuando las pruebas A–H pasan en la Raspberry Pi objetivo y una micro-meta vinculada desde una nota puede completarse por comando o UI, actualizando de forma consistente el grafo, el foco principal, las estadísticas y el historial.
