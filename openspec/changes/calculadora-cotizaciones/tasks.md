# Tasks: calculadora-cotizaciones

## Progress
- Total: 24 tasks
- Completed: 0 / 24

---

## 1. Backend Infrastructure & Configuration

- [ ] **1.1** Configurar estructura base del proyecto Spring Boot
  - **Description:** Crear la estructura de paquetes base para el proyecto Spring Boot con arquitectura por capas (controller, service, dto, config, exception). Configurar application.properties con puerto y configuraciones básicas.
  - **Files:** `src/main/java/com/company/quotations/`, `src/main/resources/application.properties`
  - **Acceptance criteria:**
    - [ ] Estructura de paquetes creada siguiendo convenciones de Spring Boot
    - [ ] Application.properties configurado con puerto 8080 y configuraciones básicas
    - [ ] Proyecto compila sin errores con Maven/Gradle

- [ ] **1.2** Configurar CORS para comunicación con frontend Angular
  - **Description:** Implementar configuración CORS que permita peticiones desde el frontend Angular (puerto 4200 por defecto) con métodos POST y headers necesarios para la comunicación HTTP.
  - **Files:** `src/main/java/com/company/quotations/config/CorsConfig.java`
  - **Acceptance criteria:**
    - [ ] CORS configurado para permitir origen localhost:4200
    - [ ] Métodos POST, GET, OPTIONS habilitados
    - [ ] Headers Content-Type y Accept permitidos

- [ ] **1.3** Configurar manejo global de excepciones
  - **Description:** Implementar GlobalExceptionHandler con @ControllerAdvice para manejar excepciones de validación, errores de formato numérico y errores generales del sistema con respuestas JSON consistentes.
  - **Files:** `src/main/java/com/company/quotations/exception/GlobalExceptionHandler.java`
  - **Acceptance criteria:**
    - [ ] Maneja IllegalArgumentException retornando HTTP 400
    - [ ] Maneja NumberFormatException con mensaje apropiado
    - [ ] Respuestas de error siguen formato JSON estándar con timestamp

---

## 2. Backend Data Transfer Objects (DTOs)

- [ ] **2.1** Crear DTO para peticiones de cálculo
  - **Description:** Implementar CalculationRequest con validaciones de entrada usando Bean Validation. Debe incluir campo value de tipo BigDecimal con validaciones @NotNull y @DecimalMin.
  - **Files:** `src/main/java/com/company/quotations/dto/CalculationRequest.java`
  - **Acceptance criteria:**
    - [ ] Campo value de tipo BigDecimal con validación @NotNull
    - [ ] Validación @DecimalMin("0.0") para valores no negativos
    - [ ] Incluye getters, setters y constructor por defecto

- [ ] **2.2** Crear DTO para respuestas exitosas
  - **Description:** Implementar CalculationResponse con todos los campos requeridos: originalValue, calculatedValue, factor y timestamp. Usar BigDecimal para valores monetarios y LocalDateTime para timestamp.
  - **Files:** `src/main/java/com/company/quotations/dto/CalculationResponse.java`
  - **Acceptance criteria:**
    - [ ] Campos originalValue y calculatedValue de tipo BigDecimal
    - [ ] Campo factor de tipo BigDecimal (siempre 1.2)
    - [ ] Campo timestamp de tipo LocalDateTime con formato ISO

- [ ] **2.3** Crear DTO para respuestas de error
  - **Description:** Implementar ErrorResponse para manejo consistente de errores con campos error (código), message (descripción) y timestamp. Debe ser utilizado por el GlobalExceptionHandler.
  - **Files:** `src/main/java/com/company/quotations/dto/ErrorResponse.java`
  - **Acceptance criteria:**
    - [ ] Campo error de tipo String para código de error
    - [ ] Campo message de tipo String para descripción
    - [ ] Campo timestamp de tipo LocalDateTime
    - [ ] Constructor que acepta error y message, generando timestamp automáticamente

---

## 3. Backend Business Logic

- [ ] **3.1** Implementar servicio de cálculo de cotizaciones
  - **Description:** Crear QuotationCalculatorService con la lógica de negocio principal. Implementar factor constante 1.2 como static final, método de cálculo usando BigDecimal con precisión de 2 decimales y validaciones de entrada.
  - **Files:** `src/main/java/com/company/quotations/service/QuotationCalculatorService.java`
  - **Acceptance criteria:**
    - [ ] Constante CALCULATION_FACTOR = new BigDecimal("1.2") declarada como static final
    - [ ] Método calculateQuotation que multiplica valor por factor usando BigDecimal
    - [ ] Resultado redondeado a 2 decimales usando RoundingMode.HALF_UP
    - [ ] Validación de entrada que rechaza valores null y negativos

- [ ] **3.2** Implementar validaciones de negocio
  - **Description:** Crear métodos de validación privados en el servicio para verificar que los valores de entrada cumplan las reglas de negocio: no nulos, no negativos, formato numérico válido.
  - **Files:** `src/main/java/com/company/quotations/service/QuotationCalculatorService.java`
  - **Acceptance criteria:**
    - [ ] Método validateInput que verifica valor no null
    - [ ] Validación que rechaza valores negativos con mensaje específico
    - [ ] Lanza IllegalArgumentException con mensajes descriptivos para cada tipo de error

---

## 4. Backend REST API

- [ ] **4.1** Implementar controlador REST principal
  - **Description:** Crear QuotationCalculatorController con endpoint POST /api/calculate. Debe recibir CalculationRequest, validar entrada, llamar al servicio de cálculo y retornar CalculationResponse con manejo de errores.
  - **Files:** `src/main/java/com/company/quotations/controller/QuotationCalculatorController.java`
  - **Acceptance criteria:**
    - [ ] Endpoint POST /api/calculate mapeado correctamente
    - [ ] Acepta @RequestBody CalculationRequest con @Valid
    - [ ] Retorna ResponseEntity<CalculationResponse> con status 200 para éxito
    - [ ] Maneja errores retornando status HTTP apropiados (400 para validación)

- [ ] **4.2** Configurar headers de respuesta HTTP
  - **Description:** Configurar headers HTTP apropiados en las respuestas del controlador: Content-Type application/json, Cache-Control no-cache, y headers CORS necesarios.
  - **Files:** `src/main/java/com/company/quotations/controller/QuotationCalculatorController.java`
  - **Acceptance criteria:**
    - [ ] Header Content-Type: application/json en todas las respuestas
    - [ ] Header Cache-Control: no-cache para evitar cache de resultados
    - [ ] Respuestas incluyen timestamp en formato ISO 8601

---

## 5. Frontend Angular Structure

- [ ] **5.1** Configurar módulo principal de Angular
  - **Description:** Modificar app.module.ts para incluir las importaciones necesarias: HttpClientModule para comunicación HTTP, FormsModule para formularios template-driven, y declarar el componente de calculadora.
  - **Files:** `src/app/app.module.ts`
  - **Acceptance criteria:**
    - [ ] HttpClientModule importado y agregado a imports array
    - [ ] FormsModule importado para manejo de formularios
    - [ ] QuotationCalculatorComponent declarado en declarations array

- [ ] **5.2** Crear interfaces TypeScript para DTOs
  - **Description:** Definir interfaces TypeScript que correspondan a los DTOs del backend: CalculationRequest, CalculationResponse y ErrorResponse. Mantener consistencia de tipos con el backend.
  - **Files:** `src/app/models/calculation.model.ts`
  - **Acceptance criteria:**
    - [ ] Interface CalculationRequest con campo value de tipo number
    - [ ] Interface CalculationResponse con todos los campos del DTO backend
    - [ ] Interface ErrorResponse para manejo de errores HTTP

- [ ] **5.3** Crear servicio HTTP para comunicación con backend
  - **Description:** Implementar QuotationCalculatorService que maneje la comunicación HTTP con el endpoint /api/calculate. Debe incluir manejo de errores HTTP y transformación de respuestas.
  - **Files:** `src/app/services/quotation-calculator.service.ts`
  - **Acceptance criteria:**
    - [ ] Método calculateQuotation que envía POST a /api/calculate
    - [ ] Retorna Observable<CalculationResponse> para manejo reactivo
    - [ ] Manejo de errores HTTP con catchError y transformación apropiada

---

## 6. Frontend Component Implementation

- [ ] **6.1** Implementar componente principal de calculadora
  - **Description:** Crear QuotationCalculatorComponent con lógica de presentación, manejo de formulario, validaciones del lado cliente y comunicación con el servicio HTTP. Incluir propiedades para valor base, resultado calculado, mensajes de error y estado de carga.
  - **Files:** `src/app/quotation-calculator/quotation-calculator.component.ts`
  - **Acceptance criteria:**
    - [ ] Propiedades: baseValue, calculatedValue, errorMessage, isLoading
    - [ ] Método onCalculate() que ejecuta validaciones y llama al servicio
    - [ ] Validaciones del lado cliente para campo vacío y valores negativos
    - [ ] Manejo de estados de carga y error con feedback visual

- [ ] **6.2** Crear template HTML responsive
  - **Description:** Implementar template HTML con formulario para ingreso de valor, botón de cálculo, área de resultados y mensajes de error. Debe ser responsive y accesible en dispositivos móviles y desktop.
  - **Files:** `src/app/quotation-calculator/quotation-calculator.component.html`
  - **Acceptance criteria:**
    - [ ] Input numérico con validación HTML5 y binding bidireccional
    - [ ] Botón de cálculo con estado disabled durante carga
    - [ ] Área de resultados que muestra valor calculado con formato de 2 decimales
    - [ ] Mensajes de error con estilos distintivos y accesibilidad

- [ ] **6.3** Implementar estilos CSS responsive
  - **Description:** Crear estilos CSS que proporcionen una interfaz atractiva y responsive. Incluir estilos para formulario, botones, mensajes de error, área de resultados y adaptación a diferentes tamaños de pantalla.
  - **Files:** `src/app/quotation-calculator/quotation-calculator.component.css`
  - **Acceptance criteria:**
    - [ ] Estilos responsive con media queries para móvil y desktop
    - [ ] Estilos distintivos para estados de error, éxito y carga
    - [ ] Botones y campos de entrada con tamaño apropiado para touch en móviles
    - [ ] Paleta de colores consistente y accesible

---

## 7. Frontend Integration & Validation

- [ ] **7.1** Implementar validaciones del lado cliente
  - **Description:** Crear métodos de validación en el componente que verifiquen campo vacío, valores negativos y formato numérico antes de enviar petición al backend. Mostrar mensajes de error específicos según el tipo de validación.
  - **Files:** `src/app/quotation-calculator/quotation-calculator.component.ts`
  - **Acceptance criteria:**
    - [ ] Validación de campo vacío con mensaje "Por favor ingresa un valor"
    - [ ] Validación de valores negativos con mensaje "No se puede ingresar valores negativos"
    - [ ] Validación de formato numérico con mensaje apropiado
    - [ ] Validaciones ejecutadas antes de llamada HTTP

- [ ] **7.2** Integrar componente en aplicación principal
  - **Description:** Modificar app.component.html para incluir el componente de calculadora y proporcionar estructura básica de la aplicación con título y contenedor principal.
  - **Files:** `src/app/app.component.html`
  - **Acceptance criteria:**
    - [ ] Componente <app-quotation-calculator> incluido en template principal
    - [ ] Estructura HTML básica con título de aplicación
    - [ ] Contenedor principal con estilos responsive

---

## 8. Backend Unit Tests

- [ ] **8.1** Tests unitarios del servicio de cálculo
  - **Description:** Escribir tests unitarios completos para QuotationCalculatorService cubriendo todos los escenarios: cálculos exitosos, validaciones de entrada, manejo de errores y casos límite como valor cero.
  - **Files:** `src/test/java/com/company/quotations/service/QuotationCalculatorServiceTest.java`
  - **Acceptance criteria:**
    - [ ] Test para cálculo exitoso con valor entero (100 → 120.00)
    - [ ] Test para cálculo con decimales y redondeo correcto
    - [ ] Test para validación de valores negativos con excepción esperada
    - [ ] Test para validación de valor null con excepción apropiada
    - [ ] Cobertura mínima del 90% en el servicio

- [ ] **8.2** Tests unitarios del controlador REST
  - **Description:** Crear tests unitarios para QuotationCalculatorController usando MockMvc para probar endpoints, validaciones, códigos de respuesta HTTP y formato de respuestas JSON.
  - **Files:** `src/test/java/com/company/quotations/controller/QuotationCalculatorControllerTest.java`
  - **Acceptance criteria:**
    - [ ] Test para POST /api/calculate con valor válido retorna 200
    - [ ] Test para petición con valor negativo retorna 400
    - [ ] Test para petición con JSON malformado retorna 400
    - [ ] Test para verificar estructura de respuesta JSON
    - [ ] Mock del servicio para aislar tests del controlador

---

## 9. Frontend Unit Tests

- [ ] **9.1** Tests unitarios del componente Angular
  - **Description:** Escribir tests unitarios para QuotationCalculatorComponent cubriendo renderizado, interacciones de usuario, validaciones del lado cliente y comunicación con el servicio HTTP.
  - **Files:** `src/app/quotation-calculator/quotation-calculator.component.spec.ts`
  - **Acceptance criteria:**
    - [ ] Test para renderizado inicial del componente sin errores
    - [ ] Test para validación de campo vacío al hacer clic en calcular
    - [ ] Test para validación de valores negativos
    - [ ] Test para llamada exitosa al servicio y mostrar resultado
    - [ ] Mock del servicio HTTP para aislar tests del componente

- [ ] **9.2** Tests unitarios del servicio HTTP Angular
  - **Description:** Crear tests para QuotationCalculatorService de Angular verificando peticiones HTTP, manejo de respuestas exitosas, manejo de errores HTTP y transformación de datos.
  - **Files:** `src/app/services/quotation-calculator.service.spec.ts`
  - **Acceptance criteria:**
    - [ ] Test para petición HTTP POST exitosa con datos correctos
    - [ ] Test para manejo de error HTTP 400 del backend
    - [ ] Test para manejo de error de conexión/timeout
    - [ ] Uso de HttpClientTestingModule para mock de peticiones HTTP

---

## 10. Integration Tests

- [ ] **10.1** Tests de integración backend completo
  - **Description:** Escribir tests de integración que prueben el flujo completo desde el endpoint REST hasta el servicio de cálculo, verificando la integración entre todas las capas del backend.
  - **Files:** `src/test/java/com/company/quotations/integration/QuotationCalculatorIntegrationTest.java`
  - **Acceptance criteria:**
    - [ ] Test end-to-end con @SpringBootTest para flujo completo
    - [ ] Verificación de respuesta HTTP completa con headers correctos
    - [ ] Test de manejo de errores de validación en flujo completo
    - [ ] Verificación de tiempo de respuesta menor a 500ms