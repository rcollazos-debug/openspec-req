# Delta Spec: reports — educational-project-management-platform

## ADDED Requirements

### REQ-REPORTS-001: Dashboard de cumplimiento por zona

El sistema SHALL generar un dashboard administrativo que muestre el porcentaje de cumplimiento por zona en tiempo real. El dashboard MUST calcular automáticamente el progreso basado en hitos y entregables completados, donde cada zona representa exactamente 20% del proyecto total. El sistema SHALL actualizar los porcentajes cada vez que se modifique el estado de un entregable o hito.

**Escenarios:**

**Scenario 1: Visualización de dashboard con datos actualizados**
- **Given** existen 5 zonas con hitos y entregables asignados
- **When** el Administrador accede al dashboard de cumplimiento
- **Then** el sistema SHALL mostrar el porcentaje de cumplimiento de cada zona
- **And** el sistema SHALL mostrar el porcentaje total del proyecto consolidado

**Scenario 2: Actualización automática tras completar entregable**
- **Given** una zona tiene 50% de cumplimiento
- **When** se marca como finalizado un entregable que representa 10% de esa zona
- **Then** el dashboard SHALL actualizar automáticamente el cumplimiento de la zona a 60%
- **And** el sistema SHALL recalcular el porcentaje total del proyecto

**Scenario 3: Error en cálculo de porcentajes**
- **Given** el sistema detecta inconsistencias en los cálculos de cumplimiento
- **When** se ejecuta la validación de integridad de datos
- **Then** el sistema SHALL mostrar un mensaje de error específico
- **And** el sistema SHALL registrar el error en el log de auditoría

---

### REQ-REPORTS-002: Generación de reportes descargables

El sistema SHALL permitir al Administrador y Director generar reportes de progreso en formato PDF. Los reportes MUST incluir métricas de cumplimiento por zona, historial de cambios, estado de tareas y timeline del proyecto. El sistema SHALL generar el reporte en menos de 10 segundos y MUST incluir marca de tiempo de generación.

**Escenarios:**

**Scenario 1: Generación exitosa de reporte completo**
- **Given** el usuario tiene permisos de Administrador o Director
- **When** solicita generar un reporte de progreso completo
- **Then** el sistema SHALL generar un PDF con todas las métricas de cumplimiento
- **And** el sistema SHALL incluir gráficos de progreso por zona
- **And** el reporte SHALL generarse en menos de 10 segundos

**Scenario 2: Reporte con filtros por zona específica**
- **Given** un Coordinador solicita reporte de su zona asignada
- **When** selecciona filtros para una zona específica
- **Then** el sistema SHALL generar un reporte filtrado solo para esa zona
- **And** el sistema SHALL incluir detalles de hitos y entregables de la zona

**Scenario 3: Fallo en generación de reporte por sobrecarga**
- **Given** el sistema está procesando múltiples solicitudes simultáneas
- **When** se solicita generar un reporte y el tiempo excede 10 segundos
- **Then** el sistema SHALL mostrar mensaje de error de timeout
- **And** el sistema SHALL sugerir reintentar la operación

---

### REQ-REPORTS-003: Historial completo de cambios y auditoría

El sistema SHALL mantener un registro completo de todos los cambios realizados en entregables, hitos y tareas, incluyendo quién realizó el cambio, qué se modificó y cuándo ocurrió. El historial MUST ser inmutable y SHALL estar disponible para consulta por usuarios con permisos de Administrador y Director.

**Escenarios:**

**Scenario 1: Registro automático de cambio en entregable**
- **Given** un Coordinador modifica el estado de un entregable
- **When** guarda los cambios en el sistema
- **Then** el sistema SHALL registrar automáticamente el cambio en el historial
- **And** el registro SHALL incluir usuario, timestamp, campo modificado y valores anterior/nuevo

**Scenario 2: Consulta de historial por entregable específico**
- **Given** existe historial de cambios para un entregable
- **When** un Administrador consulta el historial de ese entregable
- **Then** el sistema SHALL mostrar cronológicamente todos los cambios realizados
- **And** cada entrada SHALL mostrar usuario responsable, fecha/hora y descripción del cambio

**Scenario 3: Intento de modificación de historial de auditoría**
- **Given** un usuario intenta modificar registros del historial de auditoría
- **When** ejecuta la operación de modificación
- **Then** el sistema SHALL rechazar la operación
- **And** el sistema SHALL registrar el intento de modificación como evento de seguridad

---

### REQ-REPORTS-004: Vistas Gantt por zona y consolidada

El sistema SHALL proporcionar vistas de diagrama Gantt diferenciadas por rol: Coordinadores verán el Gantt de su zona asignada, mientras que el Director verá una vista consolidada de todas las zonas. Los diagramas MUST mostrar hitos, entregables, dependencias y progreso en tiempo real.

**Escenarios:**

**Scenario 1: Vista Gantt de Coordinador para zona específica**
- **Given** un Coordinador está autenticado y asignado a una zona
- **When** accede a la vista Gantt de su zona
- **Then** el sistema SHALL mostrar únicamente hitos y entregables de su zona asignada
- **And** el diagrama SHALL mostrar el progreso actual de cada elemento
- **And** el sistema SHALL permitir editar fechas y dependencias de su zona

**Scenario 2: Vista Gantt consolidada del Director**
- **Given** un Director está autenticado en el sistema
- **When** accede a la vista Gantt consolidada
- **Then** el sistema SHALL mostrar hitos y entregables de todas las zonas
- **And** el diagrama SHALL usar colores diferenciados por zona
- **And** el sistema SHALL mostrar el progreso global del proyecto

**Scenario 3: Actualización en tiempo real del Gantt**
- **Given** un Gantt está siendo visualizado por un usuario
- **When** otro usuario modifica el estado de un hito o entregable
- **Then** el diagrama Gantt SHALL actualizarse automáticamente sin recargar la página
- **And** el sistema SHALL resaltar visualmente los elementos que cambiaron

---

### REQ-REPORTS-005: Métricas de desempeño en pantalla de proyección

El sistema SHALL mostrar en la pantalla de proyección métricas clave del proyecto incluyendo cuenta regresiva del mes actual, porcentaje de cumplimiento por zona, y alertas de riesgos activos. La pantalla MUST actualizarse cada 10 segundos y SHALL generar impresión automática cada 5 minutos al finalizar cada mes simulado.

**Escenarios:**

**Scenario 1: Actualización automática de métricas en pantalla**
- **Given** la pantalla de proyección está activa y visible
- **When** transcurren 10 segundos desde la última actualización
- **Then** el sistema SHALL actualizar automáticamente todas las métricas mostradas
- **And** la actualización SHALL incluir cuenta regresiva, porcentajes y alertas

**Scenario 2: Impresión automática al finalizar mes**
- **Given** un mes simulado está próximo a finalizar
- **When** el contador llega a cero y finaliza el mes
- **Then** el sistema SHALL generar automáticamente una impresión del estado actual
- **And** la impresión SHALL incluir resumen de cumplimiento por zona y eventos del mes

**Scenario 3: Fallo en conexión de pantalla de proyección**
- **Given** la pantalla de proyección pierde conectividad
- **When** el sistema detecta la desconexión
- **Then** el sistema SHALL mostrar alerta de conexión perdida en el dashboard administrativo
- **And** el sistema SHALL intentar reconectar automáticamente cada 30 segundos

---

### REQ-REPORTS-006: Exportación de datos históricos

El sistema SHALL permitir la exportación de datos históricos del proyecto en formatos CSV y JSON para análisis posterior. La exportación MUST incluir datos de cumplimiento, historial de tareas, eventos de riesgo y métricas de desempeño por zona. El sistema SHALL completar la exportación en menos de 30 segundos para datasets de hasta 10,000 registros.

**Escenarios:**

**Scenario 1: Exportación completa en formato CSV**
- **Given** un Administrador solicita exportación completa de datos
- **When** selecciona formato CSV y confirma la exportación
- **Then** el sistema SHALL generar un archivo CSV con todos los datos históricos
- **And** el archivo SHALL incluir headers descriptivos para cada columna
- **And** la exportación SHALL completarse en menos de 30 segundos

**Scenario 2: Exportación filtrada por rango de fechas**
- **Given** un Director solicita datos de un período específico
- **When** define filtros de fecha inicio y fin para la exportación
- **Then** el sistema SHALL exportar únicamente datos del rango seleccionado
- **And** el sistema SHALL validar que la fecha inicio sea anterior a la fecha fin

**Scenario 3: Fallo en exportación por volumen excesivo**
- **Given** se solicita exportación de más de 10,000 registros
- **When** el sistema procesa la solicitud
- **Then** el sistema SHALL mostrar mensaje de error indicando límite excedido
- **And** el sistema SHALL sugerir aplicar filtros para reducir el volumen de datos

## MODIFIED Requirements

[No aplica - todos los requerimientos son nuevos para este dominio]

## REMOVED Requirements

[No aplica - no hay requerimientos existentes a remover]