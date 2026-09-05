# 06 — Shell frontend

## Objetivo

Proporcionar una superficie estable para operar Inicio, Conocimiento y Progreso sin perder contexto. La interfaz usa una composición de tres zonas en escritorio y una navegación por capas en móvil.

## Estructura de escritorio

```text
┌──────────────────┬────────────────────────────┬───────────────────┐
│ Barra izquierda │ Área principal             │ Panel contextual  │
│ plegable         │                            │ derecho           │
│                  │                            │                   │
│ Inicio           │ Inicio / Carpetas / Grafo  │ Vista previa,     │
│ Conocimiento     │ Editor / Progreso          │ propiedades o     │
│ Progreso         │                            │ editor lateral    │
└──────────────────┴────────────────────────────┴───────────────────┘
```

- La barra izquierda puede plegarse y redimensionarse dentro de límites útiles.
- El área central pertenece al contexto activo.
- El panel derecho abre información contextual sin reemplazar el centro.
- La franja compacta del foco principal permanece disponible fuera de Inicio y puede plegarse.

## Navegación global

| Contexto | Trabajo principal |
| --- | --- |
| Inicio | Comprender el foco actual, el tiempo restante y la siguiente acción. |
| Conocimiento | Crear, editar, organizar, buscar por estructura y recorrer relaciones. |
| Progreso | Gestionar categorías, focos, metas, micro-metas e historial. |

Los cambios de contexto deben reflejarse en la URL o en un estado navegable equivalente para soportar recarga, historial atrás/adelante y enlaces internos.

## Inicio

Prioridad de contenido:

1. Frase del foco principal.
2. Cuenta regresiva absoluta.
3. Barra agregada y barras de metas.
4. Próximas micro-metas.
5. Captura rápida de una nota.

Inicio no se convierte en un tablero genérico de tarjetas. Solo reúne información necesaria para orientar la siguiente acción.

## Estado del cliente

Separar tres clases de estado:

| Clase | Ejemplos | Autoridad |
| --- | --- | --- |
| Servidor | Notas, bloques, relaciones, metas, eventos. | SQLite mediante API. |
| Vista | Nodo raíz, filtros, paneles abiertos, selección. | Cliente; persistencia opcional y no semántica. |
| Edición temporal | Comando incompleto, selección, borrador no confirmado. | Cliente hasta guardar. |

El cliente no mantiene una segunda copia autoritativa del progreso. Tras una acción, actualiza o invalida únicamente las consultas afectadas usando el resultado del servidor.

## Selección y panel contextual

- Un clic selecciona un nodo o elemento y abre una inspección rápida.
- `Enter` o la acción Abrir lleva a la edición detallada.
- `Esc` devuelve foco al contexto anterior.
- La selección se conserva al plegar el panel cuando sea posible.
- Un elemento eliminado o inaccesible muestra un estado explícito, no un panel vacío.

## Adaptación móvil

El MVP ofrece adaptación básica:

- La navegación izquierda se convierte en panel superpuesto.
- El contexto central ocupa la pantalla.
- El inspector derecho se abre como pantalla o panel deslizante.
- Inicio, consulta de notas, progreso y finalización de micro-metas son accesibles.
- La edición compleja del grafo y el modo Markdown no requieren paridad completa.

## Teclado y accesibilidad funcional

- Todas las acciones principales tienen ruta por teclado.
- La paleta de comandos no depende exclusivamente del puntero.
- El foco visual debe permanecer visible al cambiar de panel.
- Las piezas de progreso no comunican estado solo mediante color; muestran texto, icono o patrón adicional.
- La cuenta regresiva no debe anunciar cada segundo a tecnologías asistivas; se expone una descripción estable y actualizaciones razonables.
- Las animaciones de expansión respetan preferencias de movimiento reducido.

## Desconexión

Cuando se pierde la Raspberry Pi:

- Aparece un estado global de desconexión.
- Las consultas no disponibles muestran error y opción de reintento.
- Las mutaciones y confirmaciones de comandos se bloquean.
- No se presenta contenido temporal como guardado.
- Al reconectar, se refrescan versiones antes de habilitar escrituras.

## Fronteras de componentes

- `AppShell`: paneles y navegación.
- `FocusRibbon`: resumen compacto del foco.
- `CommandPalette`: entrada y resolución de comandos.
- `ContextPanel`: inspección y edición lateral.
- Superficies independientes para Inicio, Conocimiento, Grafo y Progreso.

Los nombres son orientativos; la separación de responsabilidades es obligatoria.
