# Delta Spec: api — calculadora-cotizaciones

## ADDED Requirements

### REQ-API-001: Endpoint de cálculo de cotizaciones

El sistema SHALL exponer un endpoint REST en `/api/calculate` que reciba un valor numérico base y retorne el resultado multiplicado por el factor fijo 1.2. El endpoint MUST validar que el valor recibido sea un número válido y no negativo antes de realizar el cálculo.

**Escenarios:**

**Scenario 1: Cálculo exitoso con valor válido**
- **Given** el endpoint `/api/calculate` está disponible
- **When** se envía una petición POST con un valor numérico positivo (ejemplo: 100)
- **Then** el sistema SHALL retornar HTTP 200 con el resultado calculado (120.0)
- **And** el tiempo de respuesta MUST ser menor a 500ms

**Scenario 2: Validación de valor negativo**
- **Given** el endpoint `/api/calculate` está disponible
- **When** se envía una petición POST con un valor negativo (ejemplo: -50)
- **Then** el sistema SHALL retornar HTTP 400 con mensaje "No se puede ingresar valores negativos"

**Scenario 3: Validación de valor no numérico**
- **Given** el endpoint `/api/calculate` está disponible
- **When** se envía una petición POST con un valor no numérico (ejemplo: "abc")
- **Then** el sistema SHALL retornar HTTP 400 con mensaje de error de formato

---

### REQ-API-002: Aplicación del factor de cálculo fijo

El sistema SHALL aplicar un factor de multiplicación fijo de 1.2 a todos los valores recibidos para el cálculo de cotizaciones. Este factor MUST ser inmutable y no configurable, representando un recargo del 20% sobre el valor base.

**Escenarios:**

**Scenario 1: Aplicación correcta del factor 1.2**
- **Given** un valor base de 100.0 es recibido
- **When** el sistema procesa el cálculo de cotización
- **Then** el resultado SHALL ser exactamente 120.0
- **And** la precisión MUST mantenerse hasta 2 decimales

**Scenario 2: Cálculo con valores decimales**
- **Given** un valor base de 99.99 es recibido
- **When** el sistema procesa el cálculo de cotización
- **Then** el resultado SHALL ser 119.99 (redondeado a 2 decimales)

**Scenario 3: Inmutabilidad del factor**
- **Given** el sistema está en funcionamiento
- **When** se intenta modificar el factor de cálculo
- **Then** el factor SHALL permanecer como 1.2 sin posibilidad de cambio

---

### REQ-API-003: Manejo de precisión decimal

El sistema SHALL manejar valores decimales con precisión de hasta 2 decimales en los cálculos de cotización. El resultado MUST ser redondeado correctamente usando el método de redondeo HALF_UP para garantizar consistencia en los cálculos financieros.

**Escenarios:**

**Scenario 1: Redondeo correcto de decimales**
- **Given** un valor base de 10.555 es recibido
- **When** el sistema calcula la cotización (10.555 * 1.2 = 12.666)
- **Then** el resultado SHALL ser 12.67 (redondeado a 2 decimales)

**Scenario 2: Preservación de decimales exactos**
- **Given** un valor base de 25.50 es recibido
- **When** el sistema calcula la cotización
- **Then** el resultado SHALL ser exactamente 30.60

**Scenario 3: Manejo de valores enteros**
- **Given** un valor base entero de 50 es recibido
- **When** el sistema calcula la cotización
- **Then** el resultado SHALL ser 60.00 (con formato de 2 decimales)

---

### REQ-API-004: Estructura de respuesta JSON

El sistema SHALL retornar las respuestas del endpoint en formato JSON con una estructura consistente que incluya el valor original, el resultado calculado y metadatos relevantes. Las respuestas de error MUST seguir un formato estándar con código de error y mensaje descriptivo.

**Escenarios:**

**Scenario 1: Respuesta exitosa con estructura completa**
- **Given** una petición válida con valor 100
- **When** el cálculo se procesa exitosamente
- **Then** la respuesta SHALL contener: `{"originalValue": 100.0, "calculatedValue": 120.0, "factor": 1.2, "timestamp": "2024-01-01T10:00:00Z"}`
- **And** el Content-Type MUST ser "application/json"

**Scenario 2: Respuesta de error con formato estándar**
- **Given** una petición con valor inválido
- **When** la validación falla
- **Then** la respuesta SHALL contener: `{"error": "INVALID_VALUE", "message": "No se puede ingresar valores negativos", "timestamp": "2024-01-01T10:00:00Z"}`

**Scenario 3: Headers de respuesta obligatorios**
- **Given** cualquier petición al endpoint
- **When** se genera la respuesta
- **Then** los headers SHALL incluir "Content-Type: application/json" y "Cache-Control: no-cache"

---

### REQ-API-005: Validación de entrada de datos

El sistema SHALL validar todos los datos de entrada antes de procesar el cálculo, rechazando peticiones con valores nulos, vacíos, no numéricos o negativos. Las validaciones MUST ejecutarse en el orden: existencia, formato numérico, y valor no negativo.

**Escenarios:**

**Scenario 1: Validación de valor nulo**
- **Given** el endpoint recibe una petición
- **When** el campo valor es null o no está presente
- **Then** el sistema SHALL retornar HTTP 400 con mensaje "Por favor ingresa un valor"

**Scenario 2: Validación de valor vacío**
- **Given** el endpoint recibe una petición
- **When** el campo valor es una cadena vacía ""
- **Then** el sistema SHALL retornar HTTP 400 con mensaje "Por favor ingresa un valor"

**Scenario 3: Validación de formato numérico**
- **Given** el endpoint recibe una petición
- **When** el campo valor contiene caracteres no numéricos
- **Then** el sistema SHALL retornar HTTP 400 con mensaje "El valor debe ser un número válido"

---

### REQ-API-006: Manejo de concurrencia y rendimiento

El sistema SHALL soportar múltiples peticiones simultáneas al endpoint de cálculo sin degradación del rendimiento. El tiempo de respuesta MUST mantenerse bajo 500ms incluso con 100 peticiones concurrentes.

**Escenarios:**

**Scenario 1: Procesamiento de peticiones concurrentes**
- **Given** 50 peticiones simultáneas son enviadas al endpoint
- **When** el sistema procesa todas las peticiones
- **Then** todas las respuestas SHALL ser correctas y consistentes
- **And** el tiempo promedio de respuesta MUST ser menor a 500ms

**Scenario 2: Manejo de carga alta**
- **Given** 100 peticiones por segundo son enviadas durante 1 minuto
- **When** el sistema procesa la carga
- **Then** la tasa de éxito SHALL ser mayor al 99%
- **And** no SHALL ocurrir errores de timeout

**Scenario 3: Aislamiento de cálculos**
- **Given** múltiples peticiones con diferentes valores se procesan simultáneamente
- **When** los cálculos se ejecutan en paralelo
- **Then** cada resultado SHALL ser independiente y correcto
- **And** no SHALL ocurrir interferencia entre cálculos

## MODIFIED Requirements

[No aplica - nuevos requerimientos]

## REMOVED Requirements

[No aplica - nuevos requerimientos]