# Product

<!-- impeccable:product-schema 1 -->

## Platform

web

## Stack

TypeScript de extremo a extremo: React con Vite para el cliente, Fastify para el servidor y SQLite como base de datos. El despliegue objetivo es una Raspberry Pi 4 con 4 GB de RAM y un SSD SATA USB alimentado de 200 GB.

## Users

Un solo usuario gestiona su conocimiento y su progreso personal. Accede desde sus dispositivos mediante su red personal. El MVP no incorpora autenticación propia y confía en la política de acceso de esa red.

La experiencia es desktop-first. En móvil se exige una adaptación básica para consulta y operaciones esenciales, no paridad completa con la edición de escritorio.

## Product Purpose

El sistema convierte conocimiento personal en progreso gestionado. Permite capturar notas compuestas por bloques, organizar y relacionar su contenido, y conectar bloques concretos con metas y micro-metas medibles.

El producto tiene éxito cuando una acción realizada desde una nota y su equivalente visual generan el mismo evento estructurado, y ese evento actualiza de forma inmediata y verificable el progreso, el historial y el foco principal.

## Positioning

No es una aplicación de notas acompañada por un panel estadístico independiente. Notas, relaciones, comandos, metas e historial comparten el mismo modelo de acciones. El conocimiento puede convertirse explícitamente en evidencia o finalización, sin interpretar automáticamente el texto libre ni mantener integraciones paralelas propensas a desincronizarse.

## Operating Context

- La aplicación se sirve desde la Raspberry Pi y requiere conexión con ella; no existe modo sin conexión en el MVP.
- SQLite es la fuente de verdad para notas, bloques, relaciones, metas, revisiones y eventos.
- Los adjuntos viven dentro de una carpeta raíz administrada por la aplicación en el SSD.
- El mismo contenido de conocimiento se explora como árbol de carpetas o como grafo semántico.
- La interfaz ofrece comandos `/` con autocompletado y controles visuales que ejecutan las mismas acciones internas.
- El usuario trabaja con texto, enlaces, imágenes, código, listas, archivos, metas, micro-metas y evidencias.

## Capabilities and Constraints

- Tres contextos principales: Inicio, Conocimiento y Progreso.
- Barra lateral izquierda plegable, área principal central y panel contextual derecho.
- Editor de bloques con atajos Markdown y representación Markdown editable y sincronizada.
- Comandos ejecutables una sola vez que se transforman en elementos visuales estructurados.
- Carpetas jerárquicas, etiquetas jerárquicas y relaciones semánticas tipadas.
- Las colecciones inteligentes y las sugerencias de clasificación mediante IA quedan fuera del MVP.
- El grafo muestra notas, metas y bloques, parte de una raíz, expande un nivel por acción y representa enlaces cruzados sin duplicar nodos.
- Cuatro categorías de progreso independientes: Físico, Mental, Ocupacional y Expresivo.
- Un solo foco principal activo, perteneciente a una categoría y compuesto por una o varias metas.
- Cada meta progresa por micro-metas de igual peso. El foco promedia el porcentaje de sus metas.
- Solo una finalización explícita modifica el progreso; las referencias y evidencias no suman por sí mismas.
- El foco usa una fecha y hora límite absoluta. Al vencer solicita extender, completar o abandonar.
- La escala objetivo es de hasta 10,000 notas, 100,000 bloques y 50 GB de adjuntos.
- Una vista de grafo muestra aproximadamente hasta 250 nodos simultáneos y carga relaciones bajo demanda.
- El MVP usa un único SSD. La papelera y los snapshots protegen contra errores lógicos, pero no contra la pérdida física de la unidad.

## Evidence on Hand

El workspace no contiene una implementación, identidad visual, nombre definitivo ni recursos de marca previos. La especificación funcional confirmada por el usuario es la única evidencia de producto disponible; el trabajo futuro no debe inventar testimonios, métricas de uso ni afirmaciones de adopción.

## Product Principles

1. **Una sola acción, una sola verdad:** comandos y UI producen la misma acción de dominio y el mismo evento.
2. **Progreso explícito:** el texto libre y las relaciones genéricas nunca modifican una meta por inferencia.
3. **Contexto sin fragmentación:** carpetas, grafo, notas y progreso son proyecciones conectadas de los mismos datos.
4. **Historial honesto y reversible:** revisiones, reversiones, vencimientos y eliminaciones se conservan sin reescribir el pasado.
5. **Complejidad progresiva:** la interfaz carga un nivel de detalle a la vez y mantiene accesibles las operaciones rápidas mediante teclado y comandos.
