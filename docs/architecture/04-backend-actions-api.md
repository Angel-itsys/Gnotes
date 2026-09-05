# 04 — Backend, acciones y API

## Forma del backend

Fastify aloja un monolito modular. Cada módulo expone contratos de aplicación y mantiene sus repositorios; las rutas HTTP son adaptadores, no el lugar donde viven las reglas.

```text
server/
├── transport/           rutas, serialización y errores HTTP
├── application/         dispatcher y servicios de aplicación
├── domain/              entidades, políticas e invariantes
├── modules/
│   ├── knowledge/
│   ├── taxonomy/
│   ├── relations/
│   ├── progress/
│   ├── assets/
│   └── history/
└── infrastructure/      SQLite, filesystem, reloj y logs
```

Esta estructura es conceptual; los nombres definitivos pueden cambiar sin mezclar responsabilidades.

## API de acciones

Todas las mutaciones entran por un contrato común:

```ts
type ActionEnvelope<TType extends string, TPayload> = {
  actionId: string;
  type: TType;
  payload: TPayload;
  expectedVersion?: number;
};
```

El `actionId` permite responder de forma idempotente si el navegador reintenta una solicitud cuyo resultado no recibió.

Respuesta conceptual:

```ts
type ActionResult<TResult> = {
  actionId: string;
  result: TResult;
  events: DomainEventSummary[];
  versions: Record<string, number>;
};
```

UI y comandos construyen el mismo `ActionEnvelope`. No existen endpoints exclusivos de comandos que repliquen reglas.

## Consultas

Las lecturas pueden usar endpoints orientados a cada proyección:

- Inicio: foco, cuenta regresiva, progreso y próximas micro-metas.
- Explorador: hijos de carpeta, etiquetas y archivos.
- Nota: bloques ordenados y metadatos.
- Grafo: vecindad de una raíz, tipos y profundidad.
- Progreso: categorías, focos, metas, revisiones y porcentajes.
- Historial: eventos paginados por fecha o entidad.

Una consulta nunca produce efectos laterales.

## Pipeline de una acción

1. Validar forma y tamaño de la solicitud.
2. Resolver menciones o IDs antes de entrar al dominio.
3. Comprobar idempotencia por `actionId`.
4. Cargar entidades y versiones requeridas.
5. Ejecutar políticas e invariantes de dominio.
6. Confirmar estado y evento dentro de una transacción.
7. Devolver resultado y versiones.
8. Notificar a otras sesiones si existe un canal de actualización activo.

## Errores estables

| Código de dominio | Significado |
| --- | --- |
| `VALIDATION_ERROR` | Payload o Markdown inválido. |
| `NOT_FOUND` | Entidad ausente o eliminada. |
| `AMBIGUOUS_REFERENCE` | Una mención humana coincide con varios destinos. |
| `VERSION_CONFLICT` | Estado modificado desde la última lectura. |
| `INVARIANT_VIOLATION` | La acción rompería una regla del dominio. |
| `STORAGE_UNAVAILABLE` | SSD, ruta o archivo no disponible. |
| `INSUFFICIENT_SPACE` | No hay capacidad segura para completar la escritura. |
| `ACTION_ALREADY_APPLIED` | Reintento idempotente; se devuelve el resultado conocido. |

La UI traduce estos códigos a mensajes accionables; no depende del texto interno del servidor.

## Reloj

El backend inyecta una abstracción de reloj. La fecha límite se guarda como instante absoluto y el servidor es autoridad para vencimiento e historial. Las pruebas pueden sustituir el reloj sin cambiar la hora del sistema.

## Actualización entre sesiones

El contrato no depende de una tecnología específica. El MVP puede usar un canal ligero de eventos o invalidación periódica, pero debe cumplir:

- Una acción exitosa actualiza de inmediato la pestaña que la inició.
- Otras sesiones detectan cambios sin duplicar eventos.
- La pérdida del canal de actualización no habilita edición offline.
- Reconectar obliga a consultar versiones actuales antes de escribir.

La elección concreta entre SSE, WebSocket o consulta periódica queda para el prototipo en la Raspberry Pi.

## Límites

- Tamaños máximos de payload separados de los archivos adjuntos.
- Archivos subidos mediante flujo dedicado, no embebidos en JSON.
- Paginación obligatoria en historiales y árboles grandes.
- Vecindad del grafo limitada por profundidad, tipos y cantidad máxima.
- Logs sin contenido completo de notas, comandos ni nombres de archivos sensibles por defecto.

## Seguridad

No existe sesión de usuario en el MVP. El backend debe:

- Escuchar únicamente en la interfaz o ruta de red decidida para la VPN, o quedar protegido por firewall equivalente.
- Rechazar orígenes web no configurados.
- No exponer rutas arbitrarias del host.
- Normalizar y validar toda ruta bajo la raíz administrada.
- Tratar todo contenido de notas y archivos como datos, nunca como código ejecutable.
