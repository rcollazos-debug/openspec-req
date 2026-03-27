# Proposal: calculadora-cotizaciones

## Intent

El área comercial actualmente realiza cálculos de cotizaciones de forma manual, aplicando un factor de recargo del 20% (factor 1.2) a los valores base de productos. Este proceso manual es propenso a errores humanos, consume tiempo valioso del equipo comercial y genera inconsistencias en las cotizaciones entregadas a clientes.

La falta de una herramienta automatizada impacta directamente en la eficiencia operativa del área comercial, aumenta el riesgo de errores en cotizaciones y puede resultar en pérdida de oportunidades de negocio por demoras en la respuesta a clientes. Además, la inconsistencia en los cálculos puede afectar la credibilidad y profesionalismo de la empresa.

Se requiere implementar una calculadora web de cotizaciones que automatice este proceso, garantizando cálculos precisos y consistentes, reduciendo el tiempo de respuesta al cliente y liberando al equipo comercial para actividades de mayor valor agregado.

## Scope In

- Interfaz web responsive con campo numérico para ingreso del valor base del producto
- Botón de cálculo que ejecute la operación de cotización
- Validación de campo vacío con mensaje "Por favor ingresa un valor"
- Validación de valores negativos con mensaje "No se puede ingresar valores negativos"
- Endpoint REST `/api/calculate` que reciba valor base y retorne cotización calculada
- Aplicación automática del factor fijo 1.2 (20% de recargo) en el backend
- Visualización inmediata del resultado calculado en la interfaz
- Frontend desarrollado en Angular (última versión)
- Backend desarrollado en Java con Spring Boot
- Arquitectura por capas sin persistencia de datos

## Scope Out

- Autenticación y autorización de usuarios
- Almacenamiento persistente de cotizaciones o historial
- Integración con sistemas externos (ERP, CRM, etc.)
- Configuración dinámica del factor de cálculo
- Múltiples tipos de productos o categorías
- Exportación de cotizaciones a PDF o Excel
- Notificaciones por email o SMS
- Auditoría de operaciones realizadas
- Gestión de usuarios y perfiles
- Cálculos con múltiples factores o descuentos

## Approach

Se implementará una arquitectura por capas con separación clara entre frontend y backend. El frontend Angular manejará la presentación y validaciones básicas del lado cliente, mientras que el backend Spring Boot expondrá un API REST para el cálculo de cotizaciones.

La validación se realizará en ambas capas: validaciones de UX en el frontend para mejorar la experiencia del usuario, y validaciones de negocio en el backend para garantizar la integridad de los datos. El factor de cálculo 1.2 se implementará como una constante inmutable en el backend, asegurando que no pueda ser modificado accidentalmente.

Se seguirá el patrón MVC en el backend con controladores REST, servicios de negocio y DTOs para el intercambio de datos. El frontend utilizará componentes Angular reactivos con formularios template-driven para la captura y validación de datos.

## Success Criteria

- [ ] El usuario puede ingresar un valor numérico en el campo de entrada
- [ ] El sistema muestra "Por favor ingresa un valor" cuando el campo está vacío al intentar calcular
- [ ] El sistema muestra "No se puede ingresar valores negativos" cuando se ingresa un valor menor a cero
- [ ] El endpoint `/api/calculate` responde correctamente con el valor multiplicado por 1.2
- [ ] El resultado del cálculo se muestra en pantalla inmediatamente después de la validación exitosa
- [ ] La aplicación es responsive y funciona correctamente en dispositivos móviles y desktop
- [ ] El tiempo de respuesta del cálculo es menor a 500ms
- [ ] La interfaz es intuitiva y no requiere capacitación para su uso
- [ ] El sistema maneja correctamente valores decimales con hasta 2 decimales de precisión

## Risks & Mitigations

| Riesgo | Impacto | Mitigación |
|--------|---------|------------|
| Errores de precisión en cálculos con decimales | Alto | Implementar BigDecimal en Java y validar precisión en pruebas unitarias |
| Validaciones inconsistentes entre frontend y backend | Medio | Definir contratos claros de validación y implementar pruebas de integración |
| Sobrecarga del servidor por múltiples requests simultáneos | Medio | Implementar throttling en el frontend y optimizar el endpoint para alta concurrencia |
| Cambios futuros en el factor de cálculo | Bajo | Documentar claramente que el factor 1.2 es inmutable según reglas de negocio |