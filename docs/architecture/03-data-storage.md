# 03 — Persistencia y almacenamiento

## Responsabilidades

SQLite conserva datos estructurados; el SSD conserva archivos binarios. La base registra metadatos y referencias, pero no incorpora adjuntos grandes como BLOB.

```text
SSD administrado
├── app/                 implementación desplegada
├── data/
│   ├── app.sqlite       estado e historial
│   ├── assets/          adjuntos administrados
│   ├── trash/           archivos retirados recuperables
│   └── snapshots/       copias lógicas locales
└── logs/                logs con rotación
```

Las rutas son ilustrativas. La ubicación definitiva permanece abierta y debe configurarse, no codificarse.

## Grupos de tablas

| Grupo | Contenido |
| --- | --- |
| Conocimiento | `folders`, `notes`, `blocks`, `tags`, asignaciones. |
| Relaciones | tipos incorporados y relaciones polimórficas. |
| Progreso | categorías, focos, metas, revisiones y micro-metas. |
| Actividad | finalizaciones, evidencias y eventos de dominio. |
| Archivos | metadatos, checksums, rutas y estado de disponibilidad. |
| Sistema | migraciones, claves de idempotencia y configuración no secreta. |

El esquema SQL exacto se define durante implementación. Las fronteras anteriores sí son obligatorias.

## Modelo de consistencia

- SQLite es la única fuente de verdad estructurada.
- Cada acción de escritura abre una transacción corta.
- El estado actual y su evento histórico se confirman juntos.
- Los archivos se preparan antes de confirmar su referencia en SQLite.
- Si falla el movimiento final del archivo, la transacción se revierte y el temporal se limpia.
- No se copian archivos de SQLite en caliente como mecanismo de snapshot; se usa una operación consistente de backup.

### Ejemplo: completar una micro-meta

```text
BEGIN
  validar revisión activa y versión esperada
  insertar MicroGoalCompleted
  actualizar proyección/estado actual necesario
  insertar DomainEvent
COMMIT
```

Si cualquier paso falla, ni el porcentaje ni el historial cambian.

## Concurrencia

El producto es de un usuario, pero puede abrirse en varias pestañas o dispositivos. La arquitectura debe asumir lecturas concurrentes y escrituras ocasionales.

- Versiones optimistas evitan sobrescribir cambios invisibles.
- Las acciones incluyen una clave idempotente para tolerar reintentos de red.
- Las transacciones de escritura no permanecen abiertas mientras el usuario interactúa.
- El modo de journal y los parámetros SQLite se validan con pruebas de apagado y carga en la Raspberry Pi antes de fijarse.

## Revisiones e historial

- `PlanRevision` es inmutable una vez activada.
- `DomainEvent` es append-only.
- El historial no sustituye las tablas de estado actual; no se implementa event sourcing completo.
- Las reversiones crean eventos compensatorios.
- El timestamp del servidor es la referencia temporal; el cliente puede enviar contexto, pero no imponer la hora registrada.

## Papelera

- La eliminación lógica conserva IDs y relaciones.
- Los archivos eliminados se mueven a `trash/` mediante una operación coordinada con SQLite.
- Restaurar devuelve el archivo a una ruta válida y reactiva sus referencias.
- La retención aún no está decidida; no se purga automáticamente hasta que exista una política confirmada.

## Snapshots

El MVP solo dispone de un SSD. Los snapshots locales sirven para volver a un estado lógico anterior, pero comparten el mismo dominio de fallo físico.

Un snapshot debe incluir:

- Copia consistente de SQLite.
- Manifiesto de adjuntos con checksum y tamaño.
- Versión del esquema y de la aplicación.
- Fecha del servidor y resultado de verificación.

La frecuencia y retención quedan abiertas. La UI debe distinguir «snapshot local» de «respaldo externo» para no prometer recuperación inexistente.

## Migraciones

- Toda versión desplegable lleva migraciones monotónicas y versionadas.
- Antes de migrar se crea un snapshot consistente.
- Una migración fallida impide iniciar la nueva versión y conserva el diagnóstico.
- Los cambios incompatibles en Markdown estructurado requieren una migración de proyección, no edición silenciosa del contenido.

## Capacidad

- 10,000 notas.
- 100,000 bloques.
- 50 GB de adjuntos.
- El servidor comprueba espacio libre antes de importar.
- La UI muestra uso, capacidad y estado de escritura.
- Alcanzar el límite no debe producir bloques o relaciones huérfanos.

## Índices mínimos por intención

La implementación debe permitir, sin escaneo total:

- Ordenar bloques de una nota.
- Navegar hijos de carpeta y etiqueta.
- Consultar relaciones entrantes y salientes por tipo.
- Obtener revisión activa y finalizaciones de una meta.
- Paginar eventos por entidad y fecha.
- Resolver assets por ID y checksum.

La búsqueda de texto completo no se considera confirmada en el MVP y no debe introducirse como requisito implícito.
