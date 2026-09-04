# 10 — Despliegue y operación

## Objetivo

Operar el MVP de forma predecible en una Raspberry Pi 4 con 4 GB de RAM y un único SSD SATA USB alimentado de 200 GB.

## Topología

```text
Dispositivos personales
        │
        │ Tailscale o WireGuard
        ▼
Raspberry Pi
├── proceso web/API
├── SQLite
├── raíz de adjuntos
├── papelera
├── snapshots locales
└── logs rotados
```

No se publica un puerto directamente en Internet. La elección final entre Tailscale y WireGuard permanece abierta.

## Frontera de seguridad

El MVP no tiene autenticación de aplicación. Consecuencias:

- Todo dispositivo con acceso de red al servicio tiene control total.
- La política VPN o firewall debe limitar origen y puerto.
- La interfaz no debe sugerir que existe aislamiento por usuario.
- Los logs no deben registrar contenido completo de notas ni comandos sensibles.
- Un futuro inicio de sesión requerirá una decisión explícita y migración de modelo de seguridad.

## Proceso de aplicación

El modo exacto de servicio —proceso administrado por el sistema o contenedor— permanece abierto. Cualquier opción debe proporcionar:

- Inicio automático después de reiniciar.
- Reinicio controlado tras fallo.
- Señal de apagado con cierre limpio de SQLite.
- Variables de entorno o archivo protegido para rutas y red.
- Usuario de sistema sin privilegios innecesarios.
- Endpoint local de salud.
- Logs con rotación y límite de tamaño.

## Entrega del cliente

El build estático de React/Vite puede servirse desde el mismo frente HTTP que la API. Esto conserva un solo origen y simplifica configuración de red. La decisión de usar un proxy inverso se toma al elegir la integración VPN y TLS.

## Reloj

La fecha límite absoluta depende de la hora del servidor.

- El host debe sincronizar hora antes de declarar saludable la aplicación.
- El servicio registra zona horaria de presentación por separado del instante almacenado.
- Un salto importante de reloj genera advertencia operativa.
- Cambiar zona horaria no reescribe deadlines ni eventos.

## Estructura de datos

La aplicación recibe una única ruta raíz configurable y valida que:

- SQLite y assets estén en el SSD previsto.
- Las rutas resueltas permanezcan bajo la raíz.
- Exista espacio suficiente para temporales y snapshots.
- La papelera no crezca sin visibilidad.
- El dispositivo esté montado antes de iniciar escrituras.

La ruta exacta y la política de montaje permanecen abiertas.

## Salud y observabilidad

Estado mínimo que la aplicación debe exponer sin revelar contenido:

- Proceso listo.
- SQLite accesible y migración actual.
- Raíz de archivos montada y escribible.
- Espacio total, usado y disponible.
- Hora del servidor y estado de sincronización.
- Último snapshot local exitoso.
- Último error de importación o persistencia resumido.

## Snapshots y recuperación

- Los snapshots usan una copia consistente de SQLite.
- Incluyen manifiesto de assets y versión del esquema.
- Se verifica que el snapshot pueda abrirse y que su manifiesto sea legible.
- La frecuencia y retención no se inventan hasta decisión del usuario.
- Todos los snapshots viven en el mismo SSD durante el MVP.

Un fallo físico, robo o corrupción total del SSD puede perder datos y snapshots. La UI y documentación deben expresar esta limitación sin llamar «respaldo externo» a los snapshots locales.

## Actualizaciones

Flujo recomendado:

1. Comprobar salud y espacio.
2. Crear snapshot consistente.
3. Instalar build y dependencias versionadas.
4. Ejecutar migraciones.
5. Iniciar nueva versión.
6. Ejecutar smoke tests.
7. Conservar logs y diagnóstico si falla.

La estrategia de rollback de binarios se define junto al método de despliegue; una migración de datos no debe revertirse mediante una copia de archivos no validada.

## Recuperación operativa

| Incidente | Respuesta |
| --- | --- |
| Proceso detenido | Reinicio administrado y revisión de salud. |
| SQLite no abre | Detener escrituras y ofrecer restauración de snapshot validado. |
| Asset ausente | Marcarlo como faltante; no borrar referencias automáticamente. |
| SSD desmontado | No iniciar o pasar a estado no escribible. |
| Poco espacio | Bloquear importaciones antes de fallar transacciones. |
| VPN inaccesible | Diagnóstico de red separado del estado de la aplicación. |
| Pérdida total del SSD | No recuperable en el MVP sin copia externa. |
