# 11 — Calidad y pruebas

## Objetivo

Demostrar que la conexión entre conocimiento y progreso es consistente en la Raspberry Pi objetivo y que el sistema falla sin corromper estado.

## Capas de prueba

| Capa | Responsabilidad |
| --- | --- |
| Dominio | Fórmulas, estados, ciclos, revisiones e invariantes. |
| Aplicación | Coordinación de acciones, idempotencia, versiones y eventos. |
| Persistencia | Transacciones, migraciones, papelera, archivos y snapshots. |
| Contrato | Formas de acciones, consultas y errores. |
| Frontend | Estado de paneles, comandos, editor, filtros y desconexión. |
| End-to-end | Flujos completos UI/API/SQLite/archivos. |
| Raspberry Pi | Rendimiento, reinicios, espacio, reloj y operación real. |

## Propiedades críticas

1. UI y comando equivalentes producen la misma acción semántica.
2. Una acción idempotente genera como máximo un evento efectivo.
3. Estado y evento se confirman juntos o no se confirman.
4. Evidencia y relaciones no modifican progreso.
5. Solo la revisión activa participa en el cálculo.
6. Ningún ciclo de carpeta o etiqueta puede persistirse.
7. Un ciclo semántico no provoca expansión infinita.
8. Restaurar conserva identidad.
9. El tiempo del cliente no puede modificar el vencimiento del servidor.
10. Una ruta de asset nunca escapa de la raíz administrada.

## Pruebas de contrato comando/UI

Para cada acción visible en ambos caminos:

```text
interacción UI ─┐
                ├─► ActionEnvelope normalizado ─► misma aserción
comando / ──────┘
```

Comparar tipo, payload e ID de entidades. Los metadatos de origen pueden diferir, pero no el efecto.

## Flujos end-to-end obligatorios

### A. Conocimiento a progreso

1. Crear nota y bloque de imagen.
2. Importar asset.
3. Vincular bloque como evidencia.
4. Confirmar que el porcentaje no cambia.
5. Completar micro-meta.
6. Confirmar actualización de Inicio, Progreso, grafo e historial.

### B. Revisión de plan

1. Crear meta con tres micro-metas.
2. Completar dos.
3. Proponer una cuarta.
4. Mostrar `66.7 % → 50 %`.
5. Confirmar y conservar revisión anterior.

### C. Temporizador

1. Guardar deadline absoluto.
2. Reiniciar navegador, servicio y Raspberry Pi.
3. Verificar cálculo desde el mismo instante.
4. Vencer y exigir decisión explícita.

### D. Grafo

1. Crear ciclo entre tres nodos y enlace a una meta.
2. Expandir por niveles.
3. Confirmar nodos únicos, enlaces cruzados y límite.
4. Abrir inspector sin perder el lienzo.

### E. Seguridad de comandos

1. Pegar un comando.
2. Importarlo en Markdown.
3. Mostrarlo dentro de código.
4. Verificar cero acciones hasta confirmación explícita.

### F. Papelera

1. Eliminar nota con relaciones y evidencias.
2. Confirmar que historial y porcentajes no se reescriben.
3. Restaurar y recuperar IDs y conexiones válidas.

### G. Fallos de almacenamiento

1. Simular falta de espacio durante importación.
2. Simular asset movido fuera de la aplicación.
3. Simular fallo entre preparación de archivo y transacción.
4. Verificar ausencia de referencias parciales y temporales abandonados.

### H. Desconexión

1. Interrumpir API o VPN.
2. Verificar bloqueo de mutaciones.
3. Reconectar y refrescar versiones.
4. Confirmar que no se duplican acciones reintentadas.

## Escala de prueba

Fixtures de capacidad objetivo:

- 10,000 notas.
- 100,000 bloques.
- 50 GB representados mediante manifiestos y una muestra realista de archivos.
- Grafo con más conexiones de las 250 permitidas en una vista.
- Historial suficiente para exigir paginación.

Los 50 GB no deben materializarse en cada ejecución automatizada. Las pruebas de capacidad combinan datos sintéticos, archivos dispersos o muestras controladas y una validación real periódica en el SSD objetivo.

## Rendimiento

No se fijan milisegundos antes del prototipo real. El proceso es:

1. Instrumentar consulta, serialización, red y render por separado.
2. Medir en Raspberry Pi 4 con 4 GB y SSD objetivo.
3. Registrar p50, p95 y peor caso para flujos principales.
4. Establecer presupuestos basados en esas mediciones.
5. Convertirlos en puertas de regresión.

Flujos que deben medirse:

- Abrir Inicio.
- Abrir una nota grande.
- Cambiar entre editor y Markdown.
- Expandir un nivel de grafo.
- Completar una micro-meta.
- Cargar historial paginado.
- Importar archivos de tamaños representativos.

## Accesibilidad y adaptación

- Recorrido completo por teclado de navegación y acciones principales.
- Foco visible y orden lógico entre paneles.
- Estado de progreso comprensible sin depender del color.
- Cuenta regresiva compatible con tecnologías asistivas sin anuncios por segundo.
- Preferencia de movimiento reducido.
- Smoke tests en escritorio y en ancho móvil para operaciones esenciales.

## Puertas de entrega

El MVP no se considera completo hasta que:

- Pasen los flujos A–H.
- Pasen migraciones desde una base vacía y desde la versión anterior disponible.
- Se verifique recuperación desde un snapshot local.
- Se pruebe al menos una interrupción de energía o terminación forzada controlada.
- Se mida la capacidad en la Raspberry Pi objetivo.
- No existan escrituras fuera de la raíz administrada.
- Las decisiones operativas abiertas necesarias para desplegar estén resueltas y documentadas.
