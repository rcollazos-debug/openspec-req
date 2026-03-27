# Design: calculadora-cotizaciones

## Overview

Se implementará una calculadora web de cotizaciones con arquitectura por capas, separando claramente el frontend Angular del backend Spring Boot. El frontend manejará la interfaz de usuario, validaciones del lado cliente y comunicación con el API REST, mientras que el backend expondrá un endpoint `/api/calculate` que aplicará el factor de cálculo fijo de 1.2 y manejará las validaciones de negocio.

La solución seguirá el patrón MVC en el backend con controladores REST, servicios de negocio y DTOs para el intercambio de datos. El frontend utilizará componentes Angular reactivos con formularios template-driven para la captura y validación de datos. La comunicación entre capas se realizará mediante HTTP/JSON, garantizando una separación limpia de responsabilidades.

El diseño prioriza la simplicidad y mantenibilidad, implementando el factor de cálculo como una constante inmutable en el backend para cumplir con la regla de negocio establecida. No se requiere persistencia de datos ni autenticación, manteniendo la aplicación como una herramienta de cálculo stateless.

## Architecture Decisions

### Decision 1: Separación Frontend-Backend con API REST
- **Context:** Se necesita una arquitectura escalable que permita evolución independiente del frontend y backend
- **Decision:** Implementar arquitectura por capas con frontend Angular y backend Spring Boot comunicándose via API REST
- **Rationale:** Permite desarrollo paralelo, testing independiente, y facilita futuras integraciones o cambios de tecnología en cualquier capa
- **Consequences:** Requiere manejo de comunicación HTTP, pero proporciona flexibilidad y mantenibilidad a largo plazo

### Decision 2: Factor de cálculo como constante inmutable
- **Context:** La regla de negocio establece que el factor 1.2 debe ser fijo e inmutable
- **Decision:** Implementar el factor como constante final en el backend, sin posibilidad de configuración
- **Rationale:** Garantiza que la regla de negocio no pueda ser modificada accidentalmente y cumple con el requerimiento de inmutabilidad
- **Consequences:** Cualquier cambio futuro del factor requerirá modificación de código y despliegue, pero asegura integridad de la regla de negocio

### Decision 3: Validación dual (frontend y backend)
- **Context:** Se requiere buena experiencia de usuario y seguridad en las validaciones
- **Decision:** Implementar validaciones tanto en frontend (UX) como en backend (seguridad)
- **Rationale:** Frontend mejora UX con feedback inmediato, backend garantiza integridad de datos independiente del cliente
- **Consequences:** Duplicación de lógica de validación, pero mayor robustez y mejor experiencia de usuario

### Decision 4: Uso de BigDecimal para precisión financiera
- **Context:** Los cálculos financieros requieren precisión decimal exacta sin errores de punto flotante
- **Decision:** Utilizar BigDecimal en Java para todos los cálculos monetarios
- **Rationale:** Evita errores de precisión típicos de float/double en cálculos financieros
- **Consequences:** Ligero overhead de performance, pero garantiza precisión exacta en cálculos comerciales

## Component Design

### QuotationCalculatorController (Backend)

**Responsabilidad:** Exponer endpoint REST para cálculo de cotizaciones y manejar validaciones de entrada
**Interfaz:**
```java
@RestController
@RequestMapping("/api")
public class QuotationCalculatorController {
    
    @PostMapping("/calculate")
    public ResponseEntity<CalculationResponse> calculate(@RequestBody CalculationRequest request);
}

// DTOs
public class CalculationRequest {
    private BigDecimal value;
}

public class CalculationResponse {
    private BigDecimal originalValue;
    private BigDecimal calculatedValue;
    private BigDecimal factor;
    private LocalDateTime timestamp;
}
```

### QuotationCalculatorService (Backend)

**Responsabilidad:** Implementar la lógica de negocio para el cálculo de cotizaciones aplicando el factor fijo
**Interfaz:**
```java
@Service
public class QuotationCalculatorService {
    private static final BigDecimal CALCULATION_FACTOR = new BigDecimal("1.2");
    
    public CalculationResult calculateQuotation(BigDecimal baseValue);
    private void validateInput(BigDecimal value);
}
```

### QuotationCalculatorComponent (Frontend)

**Responsabilidad:** Manejar la interfaz de usuario, validaciones del lado cliente y comunicación con el backend
**Interfaz:**
```typescript
@Component({
  selector: 'app-quotation-calculator',
  templateUrl: './quotation-calculator.component.html'
})
export class QuotationCalculatorComponent {
  baseValue: number | null = null;
  calculatedValue: number | null = null;
  errorMessage: string = '';
  isLoading: boolean = false;
  
  onCalculate(): void;
  private validateInput(): boolean;
  private callCalculationAPI(): void;
}
```

### QuotationCalculatorService (Frontend)

**Responsabilidad:** Manejar la comunicación HTTP con el backend para operaciones de cálculo
**Interfaz:**
```typescript
@Injectable({
  providedIn: 'root'
})
export class QuotationCalculatorService {
  private apiUrl = '/api/calculate';
  
  calculateQuotation(value: number): Observable<CalculationResponse>;
}
```

## Data Model

### CalculationRequest (Backend DTO)

| Campo | Tipo | Descripción | Restricciones |
|-------|------|-------------|---------------|
| value | BigDecimal | Valor base para el cálculo | NOT NULL, >= 0 |

### CalculationResponse (Backend DTO)

| Campo | Tipo | Descripción | Restricciones |
|-------|------|-------------|---------------|
| originalValue | BigDecimal | Valor original enviado | NOT NULL |
| calculatedValue | BigDecimal | Resultado del cálculo (originalValue * 1.2) | NOT NULL, 2 decimales |
| factor | BigDecimal | Factor aplicado (siempre 1.2) | NOT NULL, constante |
| timestamp | LocalDateTime | Momento del cálculo | NOT NULL, ISO format |

### ErrorResponse (Backend DTO)

| Campo | Tipo | Descripción | Restricciones |
|-------|------|-------------|---------------|
| error | String | Código de error | NOT NULL |
| message | String | Mensaje descriptivo del error | NOT NULL |
| timestamp | LocalDateTime | Momento del error | NOT NULL |

## Flow Diagrams

### Flujo principal de cálculo exitoso

```
Usuario → Angular Component → Angular Service → Spring Controller → Spring Service
   |            |                    |               |                    |
   |--input---->|                    |               |                    |
   |            |--validate--------->|               |                    |
   |            |                    |--HTTP POST--->|                    |
   |            |                    |               |--validate--------->|
   |            |                    |               |                    |--calculate-->
   |            |                    |               |                    |<--result-----|
   |            |                    |               |<--DTO response-----|
   |            |                    |<--HTTP 200----|                    |
   |            |<--Observable-------|               |                    |
   |<--display--|                    |               |                    |
```

### Flujo de validación con error

```
Usuario → Angular Component → Angular Service → Spring Controller
   |            |                    |               |
   |--invalid-->|                    |               |
   |            |--validate--------->|               |
   |            |                    |--HTTP POST--->|
   |            |                    |               |--validate error-->
   |            |                    |               |<--HTTP 400--------|
   |            |                    |<--error resp--|               
   |            |<--error Observable-|               |
   |<--error----|                    |               |
```

## Files to Create / Modify

| Archivo | Acción | Descripción |
|---------|--------|-------------|
| `src/main/java/com/company/quotations/controller/QuotationCalculatorController.java` | CREATE | Controlador REST que expone endpoint /api/calculate |
| `src/main/java/com/company/quotations/service/QuotationCalculatorService.java` | CREATE | Servicio de negocio con lógica de cálculo y factor fijo |
| `src/main/java/com/company/quotations/dto/CalculationRequest.java` | CREATE | DTO para peticiones de cálculo |
| `src/main/java/com/company/quotations/dto/CalculationResponse.java` | CREATE | DTO para respuestas exitosas |
| `src/main/java/com/company/quotations/dto/ErrorResponse.java` | CREATE | DTO para respuestas de error |
| `src/main/java/com/company/quotations/exception/GlobalExceptionHandler.java` | CREATE | Manejador global de excepciones |
| `src/main/java/com/company/quotations/config/CorsConfig.java` | CREATE | Configuración CORS para permitir peticiones desde Angular |
| `src/app/quotation-calculator/quotation-calculator.component.ts` | CREATE | Componente Angular principal de la calculadora |
| `src/app/quotation-calculator/quotation-calculator.component.html` | CREATE | Template HTML del componente |
| `src/app/quotation-calculator/quotation-calculator.component.css` | CREATE | Estilos CSS responsive del componente |
| `src/app/services/quotation-calculator.service.ts` | CREATE | Servicio Angular para comunicación HTTP |
| `src/app/models/calculation.model.ts` | CREATE | Interfaces TypeScript para DTOs |
| `src/app/app.module.ts` | MODIFY | Agregar imports necesarios (HttpClientModule, FormsModule) |
| `src/app/app.component.html` | MODIFY | Incluir el componente de calculadora |
| `src/test/java/com/company/quotations/controller/QuotationCalculatorControllerTest.java` | CREATE | Tests unitarios del controlador |
| `src/test/java/com/company/quotations/service/QuotationCalculatorServiceTest.java` | CREATE | Tests unitarios del servicio |
| `src/app/quotation-calculator/quotation-calculator.component.spec.ts` | CREATE | Tests unitarios del componente Angular |

## Security Considerations

**Validación de entrada:** Implementar validación estricta en el backend para prevenir inyección de datos maliciosos. Validar tipo de dato, rango numérico y formato antes del procesamiento.

**CORS Configuration:** Configurar CORS apropiadamente para permitir solo peticiones desde el dominio del frontend, evitando acceso no autorizado desde otros orígenes.

**Rate Limiting:** Aunque no se especifica en los requerimientos, considerar implementar rate limiting básico para prevenir abuso del endpoint de cálculo.

**Input Sanitization:** Sanitizar y validar todos los inputs numéricos para prevenir ataques de overflow o valores extremos que puedan causar problemas de performance.

**Error Handling:** No exponer información sensible del sistema en mensajes de error, mantener respuestas genéricas que no revelen detalles de implementación.

## Performance Considerations

**Caching de respuestas:** Implementar cache HTTP con headers apropiados (Cache-Control: no-cache) para evitar cache de resultados dinámicos.

**Optimización de BigDecimal:** Reutilizar instancias de BigDecimal para el factor constante 1.2 para evitar creación repetitiva de objetos.

**Connection pooling:** Configurar pool de conexiones HTTP en Angular para manejar múltiples peticiones concurrentes eficientemente.

**Response time:** Mantener el endpoint optimizado para cumplir con el requerimiento de respuesta < 500ms, evitando operaciones costosas innecesarias.

**Memory management:** Dado que no hay persistencia, asegurar que no se acumulen objetos en memoria entre peticiones, manteniendo el servicio stateless.

**Concurrent requests:** Diseñar el servicio para manejar múltiples peticiones simultáneas sin bloqueos, utilizando operaciones thread-safe.