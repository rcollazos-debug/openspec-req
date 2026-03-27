# Delta Spec: quotations — calculadora-cotizaciones

## ADDED Requirements

### REQ-QUOTATIONS-001: Interfaz de entrada de datos

El sistema SHALL proporcionar una interfaz web con un campo numérico para el ingreso del valor base del producto y un botón de cálculo. El campo numérico MUST aceptar valores decimales con hasta 2 decimales de precisión. La interfaz SHALL ser responsive y funcionar correctamente en dispositivos móviles y desktop.

**Escenarios:**

**Scenario 1: Ingreso exitoso de valor válido**
- **Given** la calculadora de cotizaciones está cargada en el navegador
- **When** el usuario ingresa un valor numérico positivo (ej: 100.50) en el campo de entrada
- **Then** el campo acepta el valor sin mostrar errores de validación
- **And** el botón de calcular permanece habilitado

**Scenario 2: Interfaz responsive en dispositivo móvil**
- **Given** la calculadora está siendo accedida desde un dispositivo móvil
- **When** el usuario visualiza la interfaz
- **Then** todos los elementos se ajustan correctamente al tamaño de pantalla
- **And** el campo numérico y botón son fácilmente accesibles con el dedo

**Scenario 3: Manejo de valores decimales**
- **Given** el campo de entrada está enfocado
- **When** el usuario ingresa un valor con más de 2 decimales (ej: 100.999)
- **Then** el sistema acepta solo los primeros 2 decimales (100.99)
- **And** no muestra errores de validación

---

### REQ-QUOTATIONS-002: Validación de campo vacío

El sistema SHALL validar que el campo de valor no esté vacío al momento de ejecutar el cálculo. Cuando el campo esté vacío, el sistema MUST mostrar el mensaje exacto "Por favor ingresa un valor" y SHALL NOT proceder con el cálculo.

**Escenarios:**

**Scenario 1: Intento de cálculo con campo vacío**
- **Given** el campo de valor está vacío
- **When** el usuario hace clic en el botón de calcular
- **Then** el sistema muestra el mensaje "Por favor ingresa un valor"
- **And** no se ejecuta ninguna llamada al endpoint de cálculo
- **And** no se muestra ningún resultado

**Scenario 2: Corrección después de error de campo vacío**
- **Given** se ha mostrado el mensaje "Por favor ingresa un valor"
- **When** el usuario ingresa un valor válido y hace clic en calcular
- **Then** el mensaje de error desaparece
- **And** se procede con el cálculo normalmente

**Scenario 3: Campo con solo espacios en blanco**
- **Given** el campo contiene únicamente espacios en blanco
- **When** el usuario hace clic en el botón de calcular
- **Then** el sistema trata el campo como vacío y muestra "Por favor ingresa un valor"

---

### REQ-QUOTATIONS-003: Validación de valores negativos

El sistema SHALL validar que el valor ingresado no sea negativo. Cuando se detecte un valor negativo, el sistema MUST mostrar el mensaje exacto "No se puede ingresar valores negativos" y SHALL NOT proceder con el cálculo.

**Escenarios:**

**Scenario 1: Ingreso de valor negativo**
- **Given** el campo de entrada está enfocado
- **When** el usuario ingresa un valor negativo (ej: -50.00) y hace clic en calcular
- **Then** el sistema muestra el mensaje "No se puede ingresar valores negativos"
- **And** no se ejecuta la llamada al endpoint de cálculo
- **And** el valor negativo permanece visible en el campo

**Scenario 2: Corrección de valor negativo**
- **Given** se ha mostrado el mensaje "No se puede ingresar valores negativos"
- **When** el usuario cambia el valor a un número positivo y hace clic en calcular
- **Then** el mensaje de error desaparece
- **And** se procede con el cálculo del valor positivo

**Scenario 3: Valor cero como caso límite**
- **Given** el campo de entrada está enfocado
- **When** el usuario ingresa el valor 0 y hace clic en calcular
- **Then** el sistema acepta el valor sin mostrar errores
- **And** procede con el cálculo normalmente (resultado: 0.00)

---

### REQ-QUOTATIONS-004: Endpoint REST de cálculo

El sistema SHALL exponer un endpoint REST en la ruta `/api/calculate` que reciba el valor base mediante método POST. El endpoint MUST aplicar el factor fijo de cálculo 1.2 al valor recibido y retornar el resultado. El tiempo de respuesta SHALL ser menor a 500ms.

**Escenarios:**

**Scenario 1: Cálculo exitoso con valor entero**
- **Given** el endpoint `/api/calculate` está disponible
- **When** se envía una petición POST con valor 100
- **Then** el endpoint retorna status 200
- **And** el resultado es 120.00 (100 * 1.2)
- **And** el tiempo de respuesta es menor a 500ms

**Scenario 2: Cálculo con valor decimal**
- **Given** el endpoint `/api/calculate` está disponible
- **When** se envía una petición POST con valor 99.99
- **Then** el endpoint retorna status 200
- **And** el resultado es 119.99 (99.99 * 1.2)
- **And** mantiene la precisión decimal correcta

**Scenario 3: Manejo de valor cero**
- **Given** el endpoint `/api/calculate` está disponible
- **When** se envía una petición POST con valor 0
- **Then** el endpoint retorna status 200
- **And** el resultado es 0.00

**Scenario 4: Error de servidor indisponible**
- **Given** el backend está temporalmente indisponible
- **When** se envía una petición POST al endpoint
- **Then** el sistema retorna un error HTTP 500 o timeout
- **And** el frontend maneja el error mostrando un mensaje apropiado

---

### REQ-QUOTATIONS-005: Visualización de resultados

El sistema SHALL mostrar el resultado del cálculo en pantalla inmediatamente después de una validación exitosa. El resultado MUST mostrarse con formato numérico apropiado (2 decimales) y SHALL incluir una etiqueta descriptiva clara del valor calculado.

**Escenarios:**

**Scenario 1: Mostrar resultado después de cálculo exitoso**
- **Given** el usuario ha ingresado un valor válido (ej: 100)
- **When** hace clic en calcular y el endpoint retorna el resultado (120.00)
- **Then** el resultado se muestra inmediatamente en pantalla
- **And** el formato incluye 2 decimales (120.00)
- **And** se muestra una etiqueta descriptiva como "Cotización calculada"

**Scenario 2: Actualización de resultado con nuevo cálculo**
- **Given** ya se muestra un resultado previo en pantalla
- **When** el usuario ingresa un nuevo valor y calcula
- **Then** el resultado anterior se reemplaza por el nuevo resultado
- **And** la transición es inmediata sin parpadeos

**Scenario 3: Manejo de error en cálculo**
- **Given** el usuario ha ingresado un valor válido
- **When** el endpoint de cálculo retorna un error
- **Then** se muestra un mensaje de error apropiado
- **And** el resultado anterior (si existe) permanece visible
- **And** se proporciona opción para reintentar el cálculo

---

### REQ-QUOTATIONS-006: Factor de cálculo inmutable

El sistema SHALL aplicar un factor de cálculo fijo de 1.2 (equivalente a 20% de recargo) como regla de negocio inmutable. Este factor MUST estar implementado como una constante en el backend y SHALL NOT ser modificable durante la ejecución de la aplicación.

**Escenarios:**

**Scenario 1: Aplicación consistente del factor 1.2**
- **Given** cualquier valor base válido es enviado al endpoint
- **When** se ejecuta el cálculo
- **Then** el resultado es siempre el valor base multiplicado por exactamente 1.2
- **And** no existe variación en el factor aplicado

**Scenario 2: Verificación de inmutabilidad del factor**
- **Given** la aplicación está en ejecución
- **When** se realizan múltiples cálculos consecutivos
- **Then** todos los cálculos aplican consistentemente el factor 1.2
- **And** no hay posibilidad de modificar el factor desde la interfaz

**Scenario 3: Precisión en cálculos con decimales**
- **Given** se ingresa un valor con decimales (ej: 83.33)
- **When** se aplica el factor 1.2
- **Then** el resultado mantiene la precisión matemática correcta (99.996 → 100.00)
- **And** el redondeo se realiza a 2 decimales según estándares comerciales

## MODIFIED Requirements

[No aplica - todos los requerimientos son nuevos]

## REMOVED Requirements

[No aplica - no se eliminan requerimientos existentes]