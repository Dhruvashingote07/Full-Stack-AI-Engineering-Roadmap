# Part 4: Spring Framework Ecosystem

> **Prerequisites**: Java 17+, basic knowledge of SQL and web concepts.
> **Duration**: 4–6 weeks (2–3 hours/day)
> **Goal**: Build production-ready Spring Boot applications with REST APIs, JPA persistence, security, messaging, and microservices.

---

## Chapter 1: Spring Core & IoC

### Introduction

The Spring Framework's foundation is the **Inversion of Control (IoC)** container. Instead of objects creating their own dependencies, the container _injects_ them. This decouples object creation from business logic and makes code testable, modular, and maintainable.

### Why It Matters

- Eliminates boilerplate `new` calls and manual wiring
- Encourages programming to interfaces
- Enables easy swapping of implementations (mock vs real)
- Forms the basis for every Spring project (Boot, Cloud, Data, Security)

### Theory

#### Inversion of Control (IoC) & Dependency Injection (DI)

**IoC** means the framework controls the flow — your code is called by the framework, not the other way around. **DI** is the mechanism: the container provides (injects) dependencies into a class.

Three injection styles:

| Style | How | Use Case |
|-------|-----|----------|
| Constructor | Parameters in constructor | **Preferred** — immutable, testable, required deps |
| Setter | `@Autowired` on setter | Optional dependencies, circular deps workaround |
| Field | `@Autowired` on field directly | Quick prototyping — **avoid in production** |

```java
// CONSTRUCTOR INJECTION (preferred)
@Component
public class OrderService {
    private final PaymentGateway paymentGateway;
    private final InventoryClient inventoryClient;

    public OrderService(PaymentGateway paymentGateway, InventoryClient inventoryClient) {
        this.paymentGateway = paymentGateway;
        this.inventoryClient = inventoryClient;
    }
}
```

#### ApplicationContext — The IoC Container

| Implementation | Description |
|----------------|-------------|
| `AnnotationConfigApplicationContext` | Java-based config (modern) |
| `ClassPathXmlApplicationContext` | XML-based config (legacy) |
| `GenericWebApplicationContext` | Web-aware (used by Spring Boot internally) |

```java
// Programmatic usage (rare in Boot — auto-configured)
try (var ctx = new AnnotationConfigApplicationContext(AppConfig.class)) {
    OrderService service = ctx.getBean(OrderService.class);
    service.placeOrder();
}
```

#### Bean Lifecycle

```
Spring Container started
    ↓
Instantiate bean
    ↓
Populate properties
    ↓
Set bean name (BeanNameAware)
    ↓
Set bean factory (BeanFactoryAware)
    ↓
Pre-initialization (BeanPostProcessor#postProcessBeforeInitialization)
    ↓
@PostConstruct / InitializingBean#afterPropertiesSet / init-method
    ↓
Post-initialization (BeanPostProcessor#postProcessAfterInitialization)
    ↓
→ Bean is READY for use ←
    ↓
Container shutdown
    ↓
@PreDestroy / DisposableBean#destroy / destroy-method
```

```java
@Component
public class DatabaseConnection {
    @Value("${db.url}")
    private String url;

    @PostConstruct
    public void init() {
        System.out.println("Connecting to: " + url);
        // open connection pool
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Closing connections...");
    }
}
```

#### Bean Scopes

| Scope | Description | Typical Use |
|-------|-------------|-------------|
| `singleton` | One instance per container (default) | Stateless services, DAOs |
| `prototype` | New instance every injection | Stateful beans, expensive creation |
| `request` | One instance per HTTP request | Web context only |
| `session` | One instance per HTTP session | User-preferences, shopping cart |
| `application` | One instance per ServletContext | Shared web app state |

> **Warning**: Injecting a `prototype` scoped bean into a `singleton` bean causes scope loss unless you use `@Lookup` or `ObjectFactory<T>`.

```java
@Component
@Scope("prototype")
public class ShoppingCart {
    private List<OrderItem> items = new ArrayList<>();

    public void addItem(OrderItem item) { items.add(item); }
}

@Component
public class CheckoutService {
    @Autowired
    private ObjectFactory<ShoppingCart> cartFactory;

    public void processCheckout() {
        ShoppingCart cart = cartFactory.getObject(); // fresh instance every time
    }
}
```

#### Stereotype Annotations & Wiring

| Annotation | Purpose |
|------------|---------|
| `@Component` | Generic Spring-managed bean |
| `@Service` | Service layer (business logic) |
| `@Repository` | DAO/Repository layer (adds exception translation) |
| `@Controller` | Web controller (MVC / REST) |
| `@Autowired` | Injects a bean by type |
| `@Qualifier` | Narrow injection to a specific bean name |
| `@Primary` | Default bean when multiple candidates exist |
| `@Value` | Injects property values / SpEL expressions |
| `@PropertySource` | Loads `.properties` file into environment |

```java
@Service
public class InventoryService {
    private final InventoryRepository repository;

    @Value("${inventory.low-stock-threshold:10}")
    private int lowStockThreshold;

    public InventoryService(InventoryRepository repository) {
        this.repository = repository;
    }

    public boolean isLowStock(String sku) {
        return repository.getStock(sku) < lowStockThreshold;
    }
}
```

```java
@Configuration
@PropertySource("classpath:custom.properties")
public class AppConfig {
    @Value("${app.name}")
    private String appName;
}
```

#### Profiles (`@Profile`)

Activate groups of beans per environment.

```java
@Service
@Profile("dev")
public class DevEmailService implements EmailService {
    public void send(String to, String body) {
        System.out.println("[DEV] Email to " + to + ": " + body);
    }
}

@Service
@Profile("prod")
public class ProdEmailService implements EmailService {
    @Autowired private AwsSesClient client;
    public void send(String to, String body) {
        client.send(new SendEmailRequest(to, body));
    }
}
```

```properties
# application.properties
spring.profiles.active=dev
```

> 💡 **Pro Tip:** Always use constructor injection — it makes dependencies explicit, enables `final` fields for immutability, and simplifies testing (no reflection needed in test setup). Field injection with `@Autowired` hides dependencies and makes it impossible to create objects in a valid state without Spring.

### Common Mistakes

1. **Field injection in production** — breaks testability; use constructor injection
2. **Circular dependencies** — redesign: extract shared logic or use `@Lazy`
3. **Prototype scope into singleton** — scope is lost; use `ObjectFactory` or `@Lookup`
4. **Forgetting to declare `@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)`** for request/session scopes in a singleton environment

### Best Practices

- Favor constructor injection over setter/field
- Use `final` fields for immutability
- Keep `@PostConstruct` logic simple (connection pools, cache warmup, validation)
- Name beans explicitly: `@Component("myBean")` when multiple of same type
- Use `@Profile("!test")` to exclude beans from test profile
- Prefer Java config over XML in modern projects

### Interview Questions

1. What's the difference between `BeanFactory` and `ApplicationContext`?
2. How does `@Autowired` resolve dependencies when multiple beans match?
3. Explain bean scopes with real-world examples.
4. How would you inject a prototype bean into a singleton?
5. What is the bean lifecycle? Name 3 hooks.
6. Difference between `@Component`, `@Service`, `@Repository`, `@Controller`?
7. How do profiles work in Spring?

### Practical Exercises

1. Create a `UserService` with constructor injection of `UserRepository`
2. Use `@Value` to inject DB credentials from `application.properties`
3. Define a `@Configuration` class that creates two `DataSource` beans; use `@Primary` and `@Qualifier`
4. Create a bean with `@PostConstruct` that logs "Application started" and `@PreDestroy` that closes resources
5. Define two profile-specific beans and test activating each profile

### Mini Project: Configuration Management System

Build a small Java project (without Spring Boot) using `AnnotationConfigApplicationContext`:
- `DatabaseConnectionPool` with `@PostConstruct` / `@PreDestroy`
- `ConfigLoader` reading `@Value("${app.timeout}")` from a `.properties` file
- Profile-based `NotificationSender` (ConsoleNotification for dev, EmailNotification for prod)
- `ObjectFactory` to handle prototype-scoped `TransactionContext`

---

## Chapter 2: Spring Boot

### Introduction

Spring Boot eliminates the boilerplate of Spring configuration. It provides **auto-configuration**, **embedded servers**, **production-ready features**, and a **convention-over-configuration** approach.

### Why It Matters

- Zero XML, minimal annotations
- Embedded Tomcat/Jetty/Undertow — no WAR deployment needed
- Production features (Actuator, metrics, health checks) out of the box
- Huge ecosystem of starters for every integration

### Theory

#### `@SpringBootApplication`

A combination of three annotations:

```java
@SpringBootApplication
public class PaymentApplication {
    public static void main(String[] args) {
        SpringApplication.run(PaymentApplication.class, args);
    }
}
```

| Annotation | Purpose |
|------------|---------|
| `@SpringBootConfiguration` | Marks this as a configuration class |
| `@EnableAutoConfiguration` | Enables auto-configuration |
| `@ComponentScan` | Scans package and sub-packages for beans |

#### Auto-Configuration & `spring.factories`

Spring Boot scans `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (previously `spring.factories`). When a class like `DataSource` is on the classpath, Boot auto-configures a `DataSource` bean.

```java
// Custom auto-configuration (advanced)
@AutoConfiguration
@ConditionalOnClass(DataSource.class)
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean
    public DataSource dataSource(DataSourceProperties props) {
        return DataSourceBuilder.create()
                .url(props.getUrl())
                .username(props.getUsername())
                .password(props.getPassword())
                .build();
    }
}
```

#### Starters

Starter POMs bundle dependencies. Convention: `spring-boot-starter-*`.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

#### Configuration: properties vs YAML

```properties
# application.properties
server.port=8080
spring.datasource.url=jdbc:postgresql://localhost:5432/payments
spring.datasource.username=app_user
spring.datasource.password=secret
```

```yaml
# application.yml — preferred for hierarchy
server:
  port: 8080

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/payments
    username: app_user
    password: secret
```

#### Embedded Servers

| Server | Default? | Benefit |
|--------|----------|---------|
| Tomcat | Yes | Mature, widely used |
| Jetty | With exclusion | Better for long-lived connections (WebSocket) |
| Undertow | With exclusion | Lightweight, higher throughput |

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

#### DevTools

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

- **Automatic restart**: detects classpath changes → restarts app quickly
- **LiveReload**: triggers browser refresh
- **Disable caching**: `Thymeleaf`, `Mustache` templates reload without restart

> **Warning**: DevTools should be `optional` or `development` scope—never packaged in production.

> ✅ **Best Practice:** Use `spring-boot-starter-actuator` in every service. Expose `health`, `info`, and `metrics` endpoints at minimum. Configure `info` to display build version, git commit, and deployment timestamp — invaluable for debugging production issues.

#### Actuator

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,env,beans
  endpoint:
    health:
      show-details: always
```

| Endpoint | Path | Description |
|----------|------|-------------|
| Health | `/actuator/health` | App health + component status |
| Info | `/actuator/info` | Custom app info (version, name) |
| Metrics | `/actuator/metrics` | JVM, thread, memory metrics |
| Env | `/actuator/env` | Environment properties |
| Beans | `/actuator/beans` | All Spring-managed beans |
| Mappings | `/actuator/mappings` | Request mappings |

```java
@Component
public class CustomHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        boolean dbUp = checkDatabase();
        return dbUp
            ? Health.up().withDetail("database", "reachable").build()
            : Health.down().withDetail("database", "unreachable").build();
    }
}
```

#### ConfigurationProperties (Type-safe configuration)

```java
@ConfigurationProperties(prefix = "app.payment")
@Component
public class PaymentProperties {
    private int retryCount;
    private Duration timeout;
    private List<String> supportedCurrencies;
    private Map<String, String> gatewayEndpoints;

    // getters and setters
}
```

```yaml
# application.yml
app:
  payment:
    retry-count: 3
    timeout: 30s
    supported-currencies:
      - USD
      - EUR
      - INR
    gateway-endpoints:
      primary: https://gateway.primary.com
      fallback: https://gateway.backup.com
```

#### Logging (Logback)

```xml
<!-- src/main/resources/logback-spring.xml -->
<configuration>
    <springProperty name="appName" source="spring.application.name"/>

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} ${appName} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/${appName}.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/${appName}.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

#### Externalized Configuration Priority

1. Devtools global settings (`~/.spring-boot-devtools.properties`)
2. `@TestPropertySource` on tests
3. Command-line arguments (`--server.port=9090`)
4. `SPRING_APPLICATION_JSON` environment variable
5. `ServletConfig` init parameters
6. `JNDI` attributes
7. System properties (`-Dserver.port=9090`)
8. OS environment variables (`SERVER_PORT=9090`)
9. `application-{profile}.properties` / `.yml`
10. `application.properties` / `.yml`
11. `@PropertySource` on configuration classes
12. Defaults (e.g., `server.port=8080`)

### Common Mistakes

1. **Omitting `@ConfigurationProperties` prefix** — beans not bound properly
2. **Actuator endpoints exposed in production** — always restrict with `management.endpoints.web.exposure.include=health,info`
3. **Properties in code instead of external config** — violates 12-factor app principles
4. **DevTools in production classpath** — can cause classloader issues
5. **Logging everything at DEBUG** — performance degradation

### Best Practices

- Use YAML over properties for readability
- Group related properties into `@ConfigurationProperties` classes
- Use profile-specific files `application-dev.yml`, `application-prod.yml`
- Set `spring.application.name` for easy identification in logs
- Always include `spring-boot-starter-actuator` for production readiness
- Configure log rotation to prevent disk exhaustion
- Use `@Data` (Lombok) on `@ConfigurationProperties` to reduce boilerplate

### Interview Questions

1. How does `@EnableAutoConfiguration` work internally?
2. Explain how to override auto-configuration.
3. What is a starter? How would you create a custom starter?
4. How does DevTools automatic restart differ from JRebel?
5. List 5 Actuator endpoints and their purpose.
6. What is the order of external configuration precedence?
7. How do you configure different logging levels per package?

### Practical Exercises

1. Create a Spring Boot app with three `application-{profile}.yml` files (dev, staging, prod)
2. Expose a custom health indicator that checks an external API
3. Create a `@ConfigurationProperties` class for email settings (host, port, credentials)
4. Enable the `/actuator/env` endpoint but restrict it to admin role
5. Configure Logback to write JSON logs for production

### Mini Project: Service Health Dashboard

Build a Spring Boot app that:
- Has 3 custom `HealthIndicator` beans (DatabaseHealth, CacheHealth, ExternalApiHealth)
- Exposes `/actuator/health` with detailed status
- Reads configuration from `application.yml` via `@ConfigurationProperties`
- Uses profile-specific configs for dev (HSQLDB) and prod (PostgreSQL)
- Logs a structured JSON entry on every health check request

---

## Chapter 3: REST API Development

### Introduction

REST (Representational State Transfer) is the dominant architectural style for web APIs. Spring Boot provides first-class support for building RESTful services with minimal boilerplate.

### Why It Matters

- Standardized, stateless communication between services and clients
- Spring's REST stack handles serialization, validation, error handling, documentation
- Well-designed REST APIs enable frontend apps, mobile apps, and third-party integrations

### Theory

#### `@RestController` & Request Mapping

```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @GetMapping
    public ResponseEntity<List<OrderResponse>> getAllOrders(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size) {
        return ResponseEntity.ok(orderService.findAll(page, size));
    }

    @GetMapping("/{id}")
    public ResponseEntity<OrderResponse> getOrder(@PathVariable Long id) {
        return ResponseEntity.ok(orderService.findById(id));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public OrderResponse createOrder(@Valid @RequestBody CreateOrderRequest request) {
        return orderService.create(request);
    }

    @PutMapping("/{id}")
    public OrderResponse updateOrder(@PathVariable Long id, @Valid @RequestBody UpdateOrderRequest request) {
        return orderService.update(id, request);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteOrder(@PathVariable Long id) {
        orderService.delete(id);
    }
}
```

| Annotation | HTTP Method | Typical Status |
|------------|-------------|----------------|
| `@GetMapping` | GET | 200 OK |
| `@PostMapping` | POST | 201 Created |
| `@PutMapping` | PUT | 200 OK |
| `@PatchMapping` | PATCH | 200 OK |
| `@DeleteMapping` | DELETE | 204 No Content |

#### Request Parameters & Path Variables

```java
// Path variable
@GetMapping("/orders/{orderId}/items/{itemId}")
public OrderItemResponse getOrderItem(
        @PathVariable Long orderId,
        @PathVariable Long itemId) { }

// Query parameters
@GetMapping("/search")
public List<OrderResponse> search(
        @RequestParam(required = false) String status,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "createdAt") String sortBy) { }
```

#### Request Body & Validation

```java
// Request DTO with Bean Validation
public record CreateOrderRequest(
    @NotBlank(message = "Customer email is required")
    @Email
    String customerEmail,

    @NotNull
    @Positive
    BigDecimal totalAmount,

    @NotEmpty
    List<@Valid OrderItemRequest> items,

    @Pattern(regexp = "^[A-Z]{2}$", message = "Currency must be ISO 2-letter code")
    String currency
) {}

// Custom validator
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = OrderAmountValidator.class)
public @interface ValidOrderAmount {
    String message() default "Total must equal sum of item amounts";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class OrderAmountValidator implements ConstraintValidator<ValidOrderAmount, CreateOrderRequest> {
    @Override
    public boolean isValid(CreateOrderRequest request, ConstraintValidatorContext context) {
        BigDecimal calculated = request.items().stream()
                .map(i -> i.price().multiply(BigDecimal.valueOf(i.quantity())))
                .reduce(BigDecimal.ZERO, BigDecimal::add);
        return calculated.compareTo(request.totalAmount()) == 0;
    }
}
```

#### Response Entity & Status Codes

```java
@GetMapping("/{id}")
public ResponseEntity<OrderResponse> getOrder(@PathVariable Long id) {
    return orderService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
}

@PostMapping
public ResponseEntity<OrderResponse> createOrder(@Valid @RequestBody CreateOrderRequest request) {
    OrderResponse created = orderService.create(request);
    URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.id())
            .toUri();
    return ResponseEntity.created(location).body(created);
}
```

#### Exception Handling

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ProblemDetail handleNotFound(ResourceNotFoundException ex) {
        ProblemDetail pd = ProblemDetail.forStatus(HttpStatus.NOT_FOUND);
        pd.setTitle("Resource Not Found");
        pd.setDetail(ex.getMessage());
        pd.setProperty("timestamp", Instant.now());
        return pd;
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail handleValidation(MethodArgumentNotValidException ex) {
        ProblemDetail pd = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        pd.setTitle("Validation Failed");
        pd.setDetail("One or more fields are invalid");

        Map<String, List<String>> errors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .collect(Collectors.groupingBy(
                        FieldError::getField,
                        Collectors.mapping(FieldError::getDefaultMessage, Collectors.toList())
                ));
        pd.setProperty("errors", errors);
        return pd;
    }

    @ExceptionHandler(Exception.class)
    public ProblemDetail handleGeneric(Exception ex) {
        ProblemDetail pd = ProblemDetail.forStatus(HttpStatus.INTERNAL_SERVER_ERROR);
        pd.setTitle("Internal Server Error");
        pd.setDetail("An unexpected error occurred");
        return pd;
    }
}
```

> **Tip**: Spring Boot 3+ uses `ProblemDetail` (RFC 7807) for standardized error responses. In older versions, use `ResponseEntity<Map<String, Object>>`.

> ✅ **Best Practice:** Always validate inputs at the controller boundary using `@Valid` and Bean Validation annotations (JSR-380). Never trust client data — validate types, ranges, formats, and business rules before passing to the service layer. This prevents invalid data from propagating through your system.

### Common Mistakes

1. **Returning entities directly** — always use DTOs to decouple API from persistence
2. **Missing validation** — always annotate request bodies with `@Valid`
3. **Exposing internal IDs** — use opaque identifiers for public APIs
4. **No global exception handler** — every endpoint should produce consistent errors
5. **Ignoring pagination** — never return unbounded lists
6. **Versioning through URL but maintaining single codebase** — leads to spaghetti

### Best Practices

- Use **DTOs/records** for request/response, never leak JPA entities
- Apply `@Valid` on all `@RequestBody` parameters
- Use `ProblemDetail` (RFC 7807) for error responses in Spring Boot 3+
- Always return `ResponseEntity` with appropriate status codes
- Document APIs with OpenAPI/swagger annotations
- Implement pagination and sorting from the start
- Version your API, even if initially only one version

### Interview Questions

1. Difference between `@Controller` and `@RestController`?
2. How would you handle validation errors globally?
3. Explain `ResponseEntity` vs `@ResponseBody` vs `@ResponseStatus`.
4. What is HATEOAS and when would you use it?
5. How does `PagedModel` work?
6. Compare API versioning strategies.
7. How does `springdoc-openapi` generate documentation?

### Practical Exercises

1. Build a CRUD REST API for `Product` entity (GET, POST, PUT, DELETE)
2. Add custom validation: `@ValidProductCode` checks code is alphanumeric + unique
3. Implement global exception handling with `ProblemDetail`
4. Add pagination and sorting to the list endpoint
5. Document the API using `@Operation` and `@ApiResponse` annotations

### Mini Project: E-Commerce Product API

Build a complete REST API:
- `Product` CRUD with DTOs, validation, pagination, sorting
- `Category` endpoint with HATEOAS links
- Search endpoint (`/api/v1/products/search?q=term&minPrice=10&maxPrice=100`)
- Proper error handling using `ProblemDetail`
- OpenAPI documentation with swagger-ui
- API versioning via URI path (`/api/v1/`)

---

## Chapter 4: Spring Data JPA & Hibernate

### Introduction

**Object-Relational Mapping (ORM)** bridges the gap between Java objects and relational database tables. **JPA** (Jakarta Persistence API) is the specification; **Hibernate** is the most popular implementation. Spring Data JPA adds a repository abstraction that eliminates boilerplate DAO code.

### Why It Matters

- Eliminates 90% of JDBC boilerplate
- Built-in transaction management
- Automatic query generation from method names
- Lazy loading, caching, dirty checking
- Database-agnostic (swap from H2 to PostgreSQL with a config change)

### Theory

#### ORM Concepts

```
┌──────────────────────┐         ┌─────────────────────┐
│    Java Object       │         │   Database Table    │
│ ┌──────────────────┐ │  ORM    │ ┌──────────────────┐│
│ │ Order order      │ │ ─────→  │ │ orders           ││
│ │ - Long id        │ │         │ │ id (BIGINT)      ││
│ │ - String email   │ │         │ │ customer_email   ││
│ │ - BigDecimal tot │ │         │ │ total_amount     ││
│ │ - LocalDate date │ │         │ │ created_date     ││
│ └──────────────────┘ │         │ └──────────────────┘│
└──────────────────────┘         └─────────────────────┘
```

| JPA Concept | Description |
|-------------|-------------|
| Entity | Java class mapped to a table |
| Persistence Context | In-memory cache of managed entities (1st-level cache) |
| Entity Manager | Interface to interact with persistence context |
| JPQL | Object-oriented query language (similar to SQL but on entities) |
| Criteria API | Type-safe programmatic queries |
| Repository | Interface with CRUD + query methods (Spring Data) |

#### Entity Mapping

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "customer_email", nullable = false, length = 255)
    private String customerEmail;

    @Column(name = "total_amount", precision = 12, scale = 2)
    private BigDecimal totalAmount;

    @Enumerated(EnumType.STRING)
    @Column(name = "status", length = 20)
    private OrderStatus status;

    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @Version
    private Long version; // optimistic locking

    // constructors, getters, setters
}
```

| Annotation | Purpose |
|------------|---------|
| `@Entity` | Marks class as JPA entity |
| `@Table` | Maps to a specific table name |
| `@Id` | Primary key |
| `@GeneratedValue` | Key generation strategy (IDENTITY, SEQUENCE, UUID) |
| `@Column` | Column mapping (name, nullable, length, unique) |
| `@Enumerated` | Enum storage (ORDINAL vs STRING — always use STRING) |
| `@Version` | Optimistic locking version field |
| `@Transient` | Field not persisted |
| `@Lob` | Large object (CLOB/BLOB) |

#### Relationships

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id", nullable = false)
    private Customer customer;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();

    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "payment_id")
    private Payment payment;

    @ManyToMany
    @JoinTable(
        name = "order_tags",
        joinColumns = @JoinColumn(name = "order_id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id")
    )
    private Set<Tag> tags = new HashSet<>();
}
```

#### Fetch Types

| Fetch Type | Behavior | When to Use |
|------------|----------|-------------|
| `FetchType.LAZY` | Loaded on first access | **Default** for `@OneToMany`, `@ManyToMany` |
| `FetchType.EAGER` | Loaded immediately with parent | Default for `@ManyToOne`, `@OneToOne` |

> **Warning**: `n+1` queries occur when you iterate over a lazily-loaded collection. Use `JOIN FETCH` or `@EntityGraph` to solve this.

```java
// Solve n+1 with JOIN FETCH
@Query("SELECT o FROM Order o JOIN FETCH o.items WHERE o.id = :id")
Optional<Order> findByIdWithItems(@Param("id") Long id);

// Using @EntityGraph
@EntityGraph(attributePaths = {"items", "payment"})
@Query("SELECT o FROM Order o WHERE o.id = :id")
Optional<Order> findByIdWithDetails(@Param("id") Long id);
```

#### Cascade Types

| Cascade Type | Description |
|--------------|-------------|
| `PERSIST` | Save child when parent is saved |
| `MERGE` | Update child when parent is merged |
| `REMOVE` | Delete child when parent is deleted |
| `REFRESH` | Refresh child when parent is refreshed |
| `DETACH` | Detach child when parent is detached |
| `ALL` | All of the above |
| `ORPHAN_REMOVAL` | Remove child when removed from collection |

```java
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
private List<OrderItem> items;

// Helper methods
public void addItem(OrderItem item) {
    items.add(item);
    item.setOrder(this);
}

public void removeItem(OrderItem item) {
    items.remove(item);
    item.setOrder(null);
}
```

#### Inheritance Strategies

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "payment_type", discriminatorType = DiscriminatorType.STRING)
public abstract class Payment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private BigDecimal amount;

    @Column(nullable = false)
    private String currency;
}

@Entity
@DiscriminatorValue("CREDIT_CARD")
public class CreditCardPayment extends Payment {
    private String cardLastFour;
    private String cardHolderName;
}

@Entity
@DiscriminatorValue("PAYPAL")
public class PayPalPayment extends Payment {
    private String paypalEmail;
    private String transactionId;
}
```

| Strategy | Table Structure | Pros | Cons |
|----------|----------------|------|------|
| `SINGLE_TABLE` | One table with discriminator column | Fast queries, no joins | Nullable columns, violates 3NF |
| `TABLE_PER_CLASS` | One table per concrete class | No nullable columns | Unions for polymorphic queries |
| `JOINED` | Shared table + child tables | Normalized schema | Multiple joins per query |

#### JPQL

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // JPQL — operates on entity names and fields
    @Query("""
        SELECT o FROM Order o
        WHERE o.customer.email = :email
        AND o.status = :status
        ORDER BY o.createdAt DESC
        """)
    List<Order> findByCustomerEmailAndStatus(
            @Param("email") String email,
            @Param("status") OrderStatus status);

    // Aggregate queries
    @Query("SELECT COUNT(o) FROM Order o WHERE o.createdAt BETWEEN :start AND :end")
    long countOrdersBetween(@Param("start") LocalDateTime start, @Param("end") LocalDateTime end);

    // Update queries
    @Modifying
    @Query("UPDATE Order o SET o.status = :status WHERE o.id = :id")
    int updateOrderStatus(@Param("id") Long id, @Param("status") OrderStatus status);
}
```

#### Criteria API (Type-safe, dynamic queries)

```java
@Service
public class OrderSearchService {
    @PersistenceContext
    private EntityManager entityManager;

    public List<Order> searchOrders(OrderSearchCriteria criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Order> query = cb.createQuery(Order.class);
        Root<Order> root = query.from(Order.class);

        List<Predicate> predicates = new ArrayList<>();

        if (criteria.email() != null) {
            predicates.add(cb.equal(root.get("customer").get("email"), criteria.email()));
        }
        if (criteria.minAmount() != null) {
            predicates.add(cb.greaterThanOrEqualTo(root.get("totalAmount"), criteria.minAmount()));
        }
        if (criteria.status() != null) {
            predicates.add(cb.equal(root.get("status"), criteria.status()));
        }
        if (criteria.fromDate() != null) {
            predicates.add(cb.greaterThanOrEqualTo(root.get("createdAt"), criteria.fromDate()));
        }

        query.where(cb.and(predicates.toArray(new Predicate[0])));
        query.orderBy(cb.desc(root.get("createdAt")));

        return entityManager.createQuery(query).getResultList();
    }
}
```

#### Spring Data JPA Repositories

| Interface | Methods Provided |
|-----------|------------------|
| `Repository<T, ID>` | Marker interface |
| `CrudRepository<T, ID>` | `save`, `findById`, `findAll`, `count`, `delete`, `existsById` |
| `PagingAndSortingRepository<T, ID>` | `findAll(Pageable)`, `findAll(Sort)` |
| `JpaRepository<T, ID>` | All above + `flush`, `getById`, `findAll(Sort, Pageable)` |

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // Query methods from method name
    List<Order> findByCustomerEmail(String email);
    Page<Order> findByStatus(OrderStatus status, Pageable pageable);

    // Custom JPQL
    @Query("SELECT o FROM Order o WHERE o.totalAmount > :amount")
    List<Order> findExpensiveOrders(@Param("amount") BigDecimal threshold);

    // Native SQL
    @Query(value = "SELECT * FROM orders WHERE EXTRACT(MONTH FROM created_at) = :month",
           nativeQuery = true)
    List<Order> findByMonth(@Param("month") int month);

    // Projections
    <T> List<T> findByCustomerEmail(String email, Class<T> type);
}
```

#### Projections

```java
// Interface-based projection (closed)
public interface OrderSummary {
    Long getId();
    String getCustomerEmail();
    BigDecimal getTotalAmount();
    OrderStatus getStatus();
}

// Interface-based projection (open — uses SpEL)
public interface OrderWithAge {
    Long getId();
    String getCustomerEmail();
    @Value("#{T(java.time.Period).between(target.createdAt.toLocalDate(), T(java.time.LocalDate).now()).getDays()}")
    Integer getAgeInDays();
}

// Record-based projection (Spring Data 3+)
public record OrderRecord(Long id, String customerEmail, BigDecimal totalAmount) { }
```

```java
// Usage
List<OrderSummary> summaries = orderRepository.findByCustomerEmail("test@test.com", OrderSummary.class);
```

#### Auditing

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
public class Order {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;
}
```

```java
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class JpaConfig {
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.ofNullable(SecurityContextHolder.getContext())
                .map(ctx -> ctx.getAuthentication())
                .map(auth -> auth.getName())
                .or(() -> Optional.of("system"));
    }
}
```

#### Transaction Management

```java
@Service
@Transactional(readOnly = true)
public class OrderService {

    private final OrderRepository orderRepository;
    private final InventoryClient inventoryClient;

    public OrderService(OrderRepository orderRepository, InventoryClient inventoryClient) {
        this.orderRepository = orderRepository;
        this.inventoryClient = inventoryClient;
    }

    @Transactional
    public OrderResponse placeOrder(CreateOrderRequest request) {
        // operations that must be atomic
        Order order = new Order(request);
        order = orderRepository.save(order);
        inventoryClient.deductStock(request.items());
        return OrderResponse.from(order);
    }

    @Transactional(rollbackFor = InsufficientStockException.class, noRollbackFor = {BusinessWarningException.class})
    public OrderResponse processPayment(Long orderId, PaymentRequest payment) {
        // custom rollback rules
    }

    @Transactional(timeout = 10)
    public void batchProcess(List<Long> orderIds) {
        // fails if not completed in 10 seconds
    }
}
```

#### Isolation Levels

| Level | Dirty Read | Non-repeatable Read | Phantom Read |
|-------|-----------|-------------------|-------------|
| `READ_UNCOMMITTED` | Yes | Yes | Yes |
| `READ_COMMITTED` | No | Yes | Yes |
| `REPEATABLE_READ` | No | No | Yes |
| `SERIALIZABLE` | No | No | No |

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public Order getOrderWithLock(Long orderId) {
    return orderRepository.findById(orderId).orElseThrow();
}
```

#### Propagation Levels

| Propagation | Behavior |
|-------------|----------|
| `REQUIRED` | Joins existing transaction or creates new (default) |
| `REQUIRES_NEW` | Suspends existing, creates new |
| `SUPPORTS` | Joins if exists, runs non-transactional otherwise |
| `NOT_SUPPORTED` | Suspends existing, runs non-transactional |
| `MANDATORY` | Joins existing or throws exception |
| `NEVER` | Runs non-transactional or throws if existing |
| `NESTED` | Savepoint within existing transaction (JDBC only) |

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void auditLog(String action) {
    // Always runs in its own transaction, independent of parent
}
```

### Common Mistakes

1. **EAGER fetching** — leads to unnecessary joins and performance issues; prefer LAZY
2. **n+1 queries** — always use `JOIN FETCH` or `@EntityGraph`
3. **Modifying entities outside a transaction** — lazy loading fails with `LazyInitializationException`
4. **Circular references in toString/hashCode** — leads to stack overflow; use `@ToString.Exclude` (Lombok)
5. **`save()` on every property change** — entities in managed state auto-sync with DB on flush
6. **Using entity classes as DTOs** — always project; entities carry persistence context overhead
7. **Not defining `equals`/`hashCode` correctly** — use business key, not ID

### Best Practices

- Use `FetchType.LAZY` everywhere except where proven necessary
- Always use `@EntityGraph` or `JOIN FETCH` for queries that need associations
- Use DTO projections (`interface` or `record`) for read-only queries
- Prefer `SEQUENCE` over `IDENTITY` for `@GeneratedValue` (Hibernate optimizes batching with sequences)
- Use `@DynamicUpdate` for partial updates on large entities
- Enable SQL logging in dev: `spring.jpa.show-sql=true`, `spring.jpa.properties.hibernate.format_sql=true`
- Add `@Version` for optimistic locking in concurrent systems
- Keep transactions small — don't hold them across network calls

### Interview Questions

1. Difference between `EntityManager` `getReference()` and `find()`?
2. How does the first-level cache work?
3. Explain the n+1 query problem and how to solve it.
4. What is the difference between `JPQL` and native queries?
5. How does Spring Data JPA derive queries from method names?
6. Explain transaction propagation with `REQUIRED` vs `REQUIRES_NEW`.
7. What is `PersistenceContext` and how is it scoped?

### Practical Exercises

1. Design entities for `Blog` with `Post`, `Comment`, `Author`, `Tag` (all relationships)
2. Write JPQL queries: find all posts by author, find recent comments, find tagged posts
3. Create `@EntityGraph` to solve n+1 on `Post.comments`
4. Implement a soft-delete mechanism using `@SQLRestriction`
5. Use `@Query` with `Pageable` to build a paginated search

### Mini Project: Blog Platform Data Layer

Build the persistence layer for a blog platform:
- Entities: `Author`, `Post`, `Comment`, `Tag`, `Category`
- All relationship types (`@OneToMany`, `@ManyToMany`, `@OneToOne`)
- Custom `@Query` for: full-text search, posts by tag, popular authors
- Projections: `PostSummary` (title, author name, comment count), `AuthorWithPostCount`
- Auditing (`@CreatedDate`, `@LastModifiedDate`, `@CreatedBy`)
- Transactional service with propagation for `publishPost` and `sendNotification`

---

## Chapter 5: Spring Security

### Introduction

Spring Security provides comprehensive authentication, authorization, and protection against common attacks. From traditional form-based login to stateless JWT and OAuth2, Spring Security covers the full spectrum.

### Why It Matters

- Industry-standard security framework for Java
- Protection against CSRF, CORS, session fixation, clickjacking
- Integrates with any authentication mechanism (LDAP, OAuth2, SAML, custom)
- Declarative security via annotations (`@PreAuthorize`, `@Secured`)

### Theory

#### Authentication vs Authorization

| Concept | Description | Spring Component |
|---------|-------------|-----------------|
| Authentication | Who are you? | `AuthenticationProvider`, `UserDetailsService` |
| Authorization | What can you do? | `AccessDecisionManager`, `@PreAuthorize` |

#### SecurityFilterChain

Spring Security works through a chain of servlet filters.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable()) // disable for stateless APIs
            .cors(Customizer.withDefaults())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**", "/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/v1/products/**").permitAll()
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/v1/orders/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .authenticationProvider(authenticationProvider())
            .addFilterBefore(jwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12); // strength factor 12
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config)
            throws Exception {
        return config.getAuthenticationManager();
    }
}
```

#### UserDetailsService

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {
    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return userRepository.findByEmail(username)
                .map(this::toUserDetails)
                .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
    }

    private UserDetails toUserDetails(User user) {
        return User.builder()
                .username(user.getEmail())
                .password(user.getPassword())
                .authorities(user.getRoles().stream()
                        .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
                        .toList())
                .accountExpired(!user.isAccountNonExpired())
                .accountLocked(!user.isAccountNonLocked())
                .credentialsExpired(!user.isCredentialsNonExpired())
                .disabled(!user.isEnabled())
                .build();
    }
}
```

#### PasswordEncoder

| Encoder | Strength | Recommendation |
|---------|----------|----------------|
| `BCryptPasswordEncoder` | Configurable (4–31) | **Preferred** — adaptive, salt included |
| `Argon2PasswordEncoder` | High | Best for new systems (memory-hard) |
| `SCryptPasswordEncoder` | High | Memory-hard but less tested in Spring |
| `Pbkdf2PasswordEncoder` | Medium | FIPS-140 compliant |
| `NoOpPasswordEncoder` | None | **Never use in production** |

```java
// BCrypt — default recommendation
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12);
}

// Argon2 — memory-hard, stronger
@Bean
public PasswordEncoder passwordEncoder() {
    return Argon2PasswordEncoder.defaultsForSpringSecurity_v5_8();
}
```

#### JWT (Stateless Authentication)

```java
@Component
public class JwtService {
    private static final SecretKey SECRET_KEY = Keys.hmacShaKeyFor(
            "your-256-bit-secret-your-256-bit-secret-your-256-bit-secret".getBytes());
    private static final long EXPIRATION = 1000 * 60 * 60; // 1 hour

    public String generateToken(UserDetails userDetails) {
        return Jwts.builder()
                .subject(userDetails.getUsername())
                .issuedAt(new Date())
                .expiration(new Date(System.currentTimeMillis() + EXPIRATION))
                .claim("roles", userDetails.getAuthorities().stream()
                        .map(GrantedAuthority::getAuthority)
                        .toList())
                .signWith(SECRET_KEY)
                .compact();
    }

    public String extractUsername(String token) {
        return extractClaims(token).getSubject();
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    private Claims extractClaims(String token) {
        return Jwts.parser()
                .verifyWith(SECRET_KEY)
                .build()
                .parseSignedClaims(token)
                .getPayload();
    }

    private boolean isTokenExpired(String token) {
        return extractClaims(token).getExpiration().before(new Date());
    }
}
```

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    private final JwtService jwtService;
    private final CustomUserDetailsService userDetailsService;

    public JwtAuthenticationFilter(JwtService jwtService, CustomUserDetailsService userDetailsService) {
        this.jwtService = jwtService;
        this.userDetailsService = userDetailsService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        String authHeader = request.getHeader("Authorization");
        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            String token = authHeader.substring(7);
            String username = jwtService.extractUsername(token);

            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                if (jwtService.isTokenValid(token, userDetails)) {
                    UsernamePasswordAuthenticationToken authToken =
                            new UsernamePasswordAuthenticationToken(
                                    userDetails, null, userDetails.getAuthorities());
                    authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    SecurityContextHolder.getContext().setAuthentication(authToken);
                }
            }
        }
        filterChain.doFilter(request, response);
    }
}
```

#### OAuth2

```yaml
# application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope: openid,profile,email
          github:
            client-id: ${GITHUB_CLIENT_ID}
            client-secret: ${GITHUB_CLIENT_SECRET}
```

```java
@Configuration
@EnableWebSecurity
public class OAuth2SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/oauth2/success", true)
                .failureUrl("/login?error=true")
                .userInfoEndpoint(userInfo -> userInfo
                    .userService(oauth2UserService())
                )
            )
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/login", "/oauth2/**").permitAll()
                .anyRequest().authenticated()
            );
        return http.build();
    }

    private OAuth2UserService<OAuth2UserRequest, OAuth2User> oauth2UserService() {
        return userRequest -> {
            String registrationId = userRequest.getClientRegistration().getRegistrationId();
            OAuth2User oauth2User = new DefaultOAuth2UserService().loadUser(userRequest);

            // Map OAuth2 attributes to local user
            String email = oauth2User.getAttribute("email");
            String name = oauth2User.getAttribute("name");

            return new DefaultOAuth2User(
                    List.of(new SimpleGrantedAuthority("ROLE_USER")),
                    oauth2User.getAttributes(),
                    "email"
            );
        };
    }
}
```

#### Method Security

```java
@Configuration
@EnableMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class MethodSecurityConfig {
    // This enables @PreAuthorize, @PostAuthorize, @Secured, @RolesAllowed
}
```

```java
@RestController
@RequestMapping("/api/v1/admin")
public class AdminController {

    @GetMapping("/users")
    @PreAuthorize("hasRole('ADMIN')")
    public List<UserResponse> getAllUsers() { }

    @GetMapping("/users/{id}")
    @PreAuthorize("hasPermission(#id, 'User', 'read')")
    public UserResponse getUser(@PathVariable Long id) { }

    @PostMapping("/users")
    @PreAuthorize("hasAuthority('USER_CREATE')")
    public UserResponse createUser(@Valid @RequestBody CreateUserRequest request) { }

    @DeleteMapping("/users/{id}")
    @Secured("ROLE_SUPER_ADMIN")
    public void deleteUser(@PathVariable Long id) { }

    @GetMapping("/reports")
    @PostAuthorize("returnObject.owner == authentication.name")
    public ReportResponse getReport(@RequestParam Long reportId) { }
}
```

#### CSRF Protection

> **Warning**: For stateless REST APIs (JWT), disable CSRF. For stateful form-based apps with sessions, **keep CSRF enabled**.

```java
// For REST APIs (stateless, no cookies for auth)
http.csrf(csrf -> csrf.disable());

// For stateful apps (form login)
http.csrf(csrf -> csrf
    .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
);
```

#### CORS Configuration

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/v1/**")
                .allowedOrigins("https://app.example.com", "http://localhost:3000")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
    }
}

// Or via SecurityFilterChain
http.cors(cors -> cors.configurationSource(corsConfigurationSource()));

@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("https://app.example.com", "http://localhost:3000"));
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS"));
    config.setAllowedHeaders(List.of("*"));
    config.setAllowCredentials(true);
    config.setMaxAge(3600L);

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/api/v1/**", config);
    return source;
}
```

#### SecurityContextHolder

```java
@Service
public class SecurityService {

    public String getCurrentUserEmail() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (auth == null || !auth.isAuthenticated()) {
            throw new SecurityException("No authenticated user");
        }
        return auth.getName();
    }

    public boolean hasRole(String role) {
        return SecurityContextHolder.getContext().getAuthentication()
                .getAuthorities().stream()
                .anyMatch(granted -> granted.getAuthority().equals("ROLE_" + role));
    }

    public User getCurrentUserEntity() {
        String email = getCurrentUserEmail();
        return userRepository.findByEmail(email)
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));
    }
}
```

### Common Mistakes

1. **Storing passwords in plaintext** — always encode with `BCryptPasswordEncoder` or `Argon2PasswordEncoder`
2. **Forgetting to mark `PasswordEncoder` as a `@Bean`** — leads to `NoSuchBeanDefinitionException`
3. **Leaving CSRF enabled for REST APIs** — blocks POST/PUT/DELETE requests from clients
4. **Hardcoding secrets (JWT secret, client secrets)** — use environment variables or Vault
5. **Not validating token expiration** — expired tokens should return 401, not 500
6. **Exposing stack traces in error responses** — use `ProblemDetail` with generic messages
7. **Using `hasAuthority("ROLE_ADMIN")` instead of `hasRole("ADMIN")`** — `hasRole` prepends `ROLE_`

### Best Practices

- Use `BCryptPasswordEncoder` with strength ≥ 10 (or Argon2 for new systems)
- For stateless JWT: disable CSRF, set session to `STATELESS`
- Always add JWT filter **before** `UsernamePasswordAuthenticationFilter`
- Use method security (`@PreAuthorize`) for fine-grained authorization
- Use environment variables for all secrets (never hardcode)
- Implement token refresh mechanism (refresh token with longer expiry)
- Add rate limiting (Buckets4j, Resilience4j) to login endpoints
- Log authentication failures (not credential details)

### Interview Questions

1. How does `SecurityFilterChain` work? What filters are in the chain?
2. Difference between `hasRole` and `hasAuthority`?
3. How does JWT authentication work end-to-end?
4. What is `SecurityContextHolder` and how is it propagated?
5. How does OAuth2 authorization code flow work?
6. Explain CSRF: what is it, when to disable, and how does Spring protect against it?
7. How does `@PreAuthorize` work with Spring AOP?

### Practical Exercises

1. Create a security config with JWT auth, public endpoints, and admin-only endpoints
2. Implement `JwtService` with token generation, validation, and refresh tokens
3. Configure OAuth2 login with Google and GitHub
4. Add method-level security with `@PreAuthorize` and `@PostAuthorize`
5. Build a permission evaluator for custom `hasPermission` expressions

### Mini Project: Secure E-Commerce API

Add security to the e-commerce API:
- JWT-based stateless authentication
- Registration + login endpoints (`/api/v1/auth/**`)
- Role-based access: `CUSTOMER` can view/place orders, `ADMIN` can manage products
- Method security: `@PreAuthorize("hasRole('ADMIN')")` on admin endpoints
- OAuth2 login via Google (optional profile enrichment)
- CORS configured for frontend at `http://localhost:3000`
- Password policy enforced at registration (min length, complexity)

---

## Chapter 6: Microservices

### Introduction

Microservices decompose a monolith into independently deployable services. Each service owns its data, scales independently, and communicates over the network. Spring Cloud provides the ecosystem for building cloud-native microservices.

### Why It Matters

- Independent scaling (scale only the services that need it)
- Technology heterogeneity (different services can use different stacks)
- Smaller, focused teams own each service
- Faster deployment cycles (change one service, not the whole app)
- Fault isolation (a bug in one service doesn't bring down everything)

### Theory

#### Monolith vs Microservices

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| Deployment | One artifact | Multiple artifacts |
| Scaling | Scale entire app | Scale per service |
| Data | Single database | Database per service |
| Team structure | Feature teams | Service teams |
| Communication | In-memory method calls | Network calls (REST/gRPC/messaging) |
| Testing | End-to-end | Service-level contracts |
| Complexity | Code complexity | Infrastructure complexity |

#### Service Decomposition — Domain-Driven Design (DDD)

```java
// Bounded Context: Order Management
// Service: order-service
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController { /* order CRUD */ }

// Bounded Context: Inventory
// Service: inventory-service
@RestController
@RequestMapping("/api/v1/inventory")
public class InventoryController { /* stock management */ }

// Bounded Context: Payment
// Service: payment-service
@RestController
@RequestMapping("/api/v1/payments")
public class PaymentController { /* payment processing */ }
```

#### Inter-service Communication: REST (Synchronous)

```java
// order-service calls payment-service over REST
@Service
public class PaymentClient {
    private final RestTemplate restTemplate;

    public PaymentClient(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public PaymentResponse processPayment(PaymentRequest request) {
        return restTemplate.postForObject(
                "http://payment-service/api/v1/payments/process",
                request,
                PaymentResponse.class);
    }
}
```

> **Warning**: Synchronous calls create temporal coupling. If `payment-service` is down, `order-service` fails. Use circuit breakers and fallbacks.

#### Service Discovery (Eureka)

```yaml
# application.yml — eureka server
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
```

```yaml
# application.yml — eureka client (order-service)
spring:
  application:
    name: order-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

```java
// Use service name instead of host:port
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}

// Now this works — Eureka resolves "payment-service" to actual instances
restTemplate.postForObject("http://payment-service/api/v1/payments/process", request, Response.class);
```

#### API Gateway (Spring Cloud Gateway)

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/v1/orders/**
          filters:
            - StripPrefix=1
            - name: CircuitBreaker
              args:
                name: orderServiceCB
                fallbackUri: forward:/fallback/orders

        - id: payment-service
          uri: lb://payment-service
          predicates:
            - Path=/api/v1/payments/**
          filters:
            - StripPrefix=1

        - id: product-service
          uri: lb://product-service
          predicates:
            - Path=/api/v1/products/**
          filters:
            - StripPrefix=1

      default-filters:
        - name: RequestRateLimiter
          args:
            redis-rate-limiter:
              replenishRate: 10
              burstCapacity: 20
```

```java
@SpringBootApplication
@EnableDiscoveryClient
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

#### Config Server (Spring Cloud Config)

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

```yaml
# application.yml — config server
server:
  port: 8888

spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/company/config-repo
          search-paths: '{application}'
          default-label: main
```

```yaml
# bootstrap.yml — config client (order-service)
spring:
  application:
    name: order-service
  cloud:
    config:
      uri: http://config-server:8888
      fail-fast: true
      retry:
        initial-interval: 1000
        max-attempts: 5
```

#### Circuit Breaker (Resilience4j)

```yaml
resilience4j:
  circuitbreaker:
    configs:
      default:
        sliding-window-size: 10
        minimum-number-of-calls: 5
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10s
        permitted-number-of-calls-in-half-open-state: 3
    instances:
      paymentService:
        base-config: default

  retry:
    configs:
      default:
        max-attempts: 3
        wait-duration: 500ms
        retry-exceptions:
          - org.springframework.web.client.HttpServerErrorException

  bulkhead:
    instances:
      paymentService:
        max-concurrent-calls: 10
        max-wait-duration: 100ms

  timelimiter:
    instances:
      paymentService:
        timeout-duration: 5s
```

```java
@Service
public class OrderService {
    private final PaymentClient paymentClient;

    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentService")
    @TimeLimiter(name = "paymentService")
    @Bulkhead(name = "paymentService")
    public CompletableFuture<OrderResponse> processOrder(OrderRequest request) {
        return CompletableFuture.supplyAsync(() -> {
            PaymentResponse payment = paymentClient.processPayment(
                    new PaymentRequest(request.totalAmount(), request.currency()));
            return OrderResponse.from(request, payment);
        });
    }

    public CompletableFuture<OrderResponse> paymentFallback(
            OrderRequest request, Throwable t) {
        log.warn("Payment service unavailable, queuing order for retry", t);
        return CompletableFuture.completedFuture(
                OrderResponse.pending(request.id(), "Payment pending"));
    }
}
```

#### Distributed Tracing (Micrometer Tracing + Zipkin)

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
# application.yml — each microservice
spring:
  application:
    name: order-service

management:
  tracing:
    sampling:
      probability: 1.0  # 100% sampling (reduce in production)

  zipkin:
    tracing:
      endpoint: http://zipkin:9411/api/v2/spans
```

#### Event-Driven Architecture (Spring Cloud Stream)

```java
// Event: OrderPlacedEvent
public record OrderPlacedEvent(
    Long orderId,
    String customerEmail,
    BigDecimal totalAmount,
    String currency,
    Instant timestamp
) {}

// Producer: order-service
@Component
public class OrderEventPublisher {
    private final StreamBridge streamBridge;

    public OrderEventPublisher(StreamBridge streamBridge) {
        this.streamBridge = streamBridge;
    }

    public void publishOrderPlaced(OrderPlacedEvent event) {
        streamBridge.send("order-placed-out-0", event);
    }
}

// Consumer: notification-service
@Component
public class OrderEventConsumer {

    @Bean
    public Consumer<OrderPlacedEvent> handleOrderPlaced() {
        return event -> {
            log.info("Sending confirmation email for order {} to {}",
                    event.orderId(), event.customerEmail());
            emailService.sendConfirmation(event.customerEmail(), event.orderId());
        };
    }
}
```

```yaml
# application.yml — order-service (producer)
spring:
  cloud:
    stream:
      bindings:
        order-placed-out-0:
          destination: orders.events
          content-type: application/json

# application.yml — notification-service (consumer)
spring:
  cloud:
    stream:
      bindings:
        handleOrderPlaced-in-0:
          destination: orders.events
          group: notification-group
          content-type: application/json
```

### Common Mistakes

1. **Shared database across services** — violates microservices principles; each service owns its data
2. **Synchronous call chains** — services calling services calling services → cascading failures
3. **No circuit breaker** — failure in one service propagates to all
4. **No distributed tracing** — impossible to debug a request spanning 5 services
5. **Over-splitting** — microservices that are too small (e.g., UserNameService + UserEmailService)
6. **Under-splitting** — one service that should have been decomposed
7. **No API gateway** — every client must know about every service

### Best Practices

- **Database per service** — no direct database access from other services
- **Use events** for cross-service communication when eventual consistency is acceptable
- **Always add circuit breakers** for synchronous calls
- **Distributed tracing is mandatory** — you can't debug without it
- **Dedicated API Gateway** handles auth, rate limiting, routing
- **Health checks** for every service (liveness + readiness)
- **Contract testing** (Pact) between services
- **Infrastructure as Code** (Terraform, Pulumi) to reproduce environments
- **Observability triad**: logging (ELK/Loki) + metrics (Prometheus) + tracing (Zipkin/Jaeger)

### Interview Questions

1. How do you decompose a monolith into microservices?
2. Explain the database-per-service pattern and its challenges.
3. How does Spring Cloud Gateway work?
4. Compare synchronous vs asynchronous inter-service communication.
5. What is the circuit breaker pattern? How does Resilience4j implement it?
6. How does distributed tracing work with Micrometer and Zipkin?
7. What is the Saga pattern? When would you use choreography vs orchestration?

### Practical Exercises

1. Create a Eureka server and register two services (order-service, payment-service)
2. Implement an API Gateway with Spring Cloud Gateway routing to both services
3. Add circuit breaker with Resilience4j to REST calls between services
4. Set up distributed tracing with Zipkin (docker-compose)
5. Publish an event when an order is placed and consume it in notification-service

### Mini Project: E-Commerce Microservices

Build a microservice system with:
- **API Gateway** (Spring Cloud Gateway) — routes, rate limiting, auth
- **Order Service** — order CRUD, publishes `OrderPlacedEvent`
- **Payment Service** — payment processing, consumes events
- **Notification Service** — sends emails on events
- **Eureka** — service discovery
- **Config Server** — centralized config from Git
- **Resilience4j** — circuit breaker on payment calls
- **Micrometer + Zipkin** — distributed tracing across all services
- **Docker-compose** — runs all services + Zipkin + RabbitMQ

---

## Chapter 7: Messaging

### Introduction

Messaging enables **asynchronous, decoupled communication** between components. Spring provides first-class support for RabbitMQ (AMQP) and Kafka (event streaming). Messaging is the backbone of event-driven architectures.

### Why It Matters

- Decouples producers from consumers (they don't need to know each other)
- Buffers against traffic spikes (messages accumulate, consumers catch up)
- Enables eventual consistency without blocking
- Reliable delivery with retry and dead-letter mechanisms

### Theory

#### RabbitMQ with Spring AMQP

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    virtual-host: /
    listener:
      simple:
        retry:
          enabled: true
          initial-interval: 1000
          max-attempts: 3
          multiplier: 2
        default-requeue-rejected: false
```

##### Configuration

```java
@Configuration
public class RabbitMQConfig {

    public static final String ORDER_EXCHANGE = "order.exchange";
    public static final String ORDER_QUEUE = "order.queue";
    public static final String ORDER_ROUTING_KEY = "order.created";
    public static final String DLQ = "order.dlq";

    @Bean
    public TopicExchange orderExchange() {
        return new TopicExchange(ORDER_EXCHANGE);
    }

    @Bean
    public Queue orderQueue() {
        return QueueBuilder.durable(ORDER_QUEUE)
                .withArgument("x-dead-letter-exchange", "")
                .withArgument("x-dead-letter-routing-key", DLQ)
                .build();
    }

    @Bean
    public Queue deadLetterQueue() {
        return QueueBuilder.durable(DLQ).build();
    }

    @Bean
    public Binding orderBinding() {
        return BindingBuilder.bind(orderQueue())
                .to(orderExchange())
                .with(ORDER_ROUTING_KEY);
    }

    @Bean
    public MessageConverter messageConverter() {
        Jackson2JsonMessageConverter converter = new Jackson2JsonMessageConverter();
        converter.setDefaultCharset(StandardCharsets.UTF_8);
        return converter;
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory,
                                         MessageConverter messageConverter) {
        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(messageConverter);
        template.setRetryTemplate(new RetryTemplate());
        return template;
    }
}
```

##### Producer

```java
@Service
public class OrderEventPublisher {
    private final RabbitTemplate rabbitTemplate;

    public OrderEventPublisher(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }

    public void publishOrderCreated(OrderCreatedEvent event) {
        rabbitTemplate.convertAndSend(
                RabbitMQConfig.ORDER_EXCHANGE,
                RabbitMQConfig.ORDER_ROUTING_KEY,
                event);
        log.info("Published order event: {}", event.orderId());
    }
}
```

##### Consumer

```java
@Component
public class OrderEventConsumer {

    @RabbitListener(queues = RabbitMQConfig.ORDER_QUEUE)
    public void handleOrderCreated(OrderCreatedEvent event) {
        log.info("Processing order: {}", event.orderId());
        // business logic
    }

    @RabbitListener(queues = RabbitMQConfig.DLQ)
    public void handleDeadLetter(OrderCreatedEvent event) {
        log.error("Order {} moved to DLQ after retries exhausted", event.orderId());
        // manual intervention / alert
    }
}
```

#### Kafka with Spring

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      properties:
        enable.idempotence: true
        acks: all
        retries: 10
    consumer:
      group-id: order-service-group
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "*"
        spring.json.use.type.headers: false
        isolation.level: read_committed
```

##### Configuration

```java
@Configuration
public class KafkaConfig {

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Object> kafkaListenerContainerFactory(
            ConsumerFactory<String, Object> consumerFactory) {
        ConcurrentKafkaListenerContainerFactory<String, Object> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory);
        factory.setCommonErrorHandler(new DefaultErrorHandler(
                new FixedBackOff(1000L, 3))); // retry 3 times
        return factory;
    }

    @Bean
    public KafkaTemplate<String, Object> kafkaTemplate(
            ProducerFactory<String, Object> producerFactory) {
        return new KafkaTemplate<>(producerFactory);
    }
}
```

##### Producer

```java
@Service
public class OrderEventPublisher {
    private final KafkaTemplate<String, Object> kafkaTemplate;
    private static final String TOPIC = "order.events";

    public OrderEventPublisher(KafkaTemplate<String, Object> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void publishOrderCreated(OrderCreatedEvent event) {
        CompletableFuture<SendResult<String, Object>> future =
                kafkaTemplate.send(TOPIC, event.orderId().toString(), event);
        future.whenComplete((result, ex) -> {
            if (ex == null) {
                log.info("Order event published to topic {} at offset {}",
                        TOPIC, result.getRecordMetadata().offset());
            } else {
                log.error("Failed to publish order event", ex);
            }
        });
    }
}
```

##### Consumer

```java
@Component
public class OrderEventConsumer {

    @KafkaListener(topics = "order.events", groupId = "notification-group")
    public void handleOrderCreated(
            @Payload OrderCreatedEvent event,
            @Header(KafkaHeaders.RECEIVED_PARTITION) int partition,
            @Header(KafkaHeaders.OFFSET) long offset) {
        log.info("Received order event from partition {} offset {}: {}",
                partition, offset, event.orderId());
        // send email, update inventory, etc.
    }

    @KafkaListener(
            topics = "order.events",
            groupId = "order-events-dlt",
            containerFactory = "kafkaListenerContainerFactory")
    @RetryableTopic(
            attempts = "4",
            backoff = @Backoff(delay = 1000, multiplier = 2),
            dltTopicSuffix = "-dlt")
    public void handleWithRetry(OrderCreatedEvent event) {
        // will be retried up to 4 times, then sent to order.events-dlt
    }

    @DltHandler
    public void handleDlt(OrderCreatedEvent event, @Header("kafka_dead_letter_topic") String topic) {
        log.warn("Order {} moved to DLT after retries", event.orderId());
    }
}
```

#### Retry & Dead Letter

| Feature | RabbitMQ | Kafka |
|---------|----------|-------|
| Retry | `spring.rabbitmq.listener.simple.retry.*` | `@RetryableTopic` or `DefaultErrorHandler` |
| Dead Letter | DLQ via `x-dead-letter-exchange` | DLT via `@RetryableTopic(dltTopicSuffix)` |
| Backoff | Fixed, exponential | `@Backoff` annotation |
| Re-queue | `default-requeue-rejected: false` sends to DLQ | Configurable via error handler |

#### Serialization

```java
// JSON serialization (default with Jackson)
@Bean
public MessageConverter messageConverter() {
    Jackson2JsonMessageConverter converter = new Jackson2JsonMessageConverter();
    converter.setDefaultCharset(StandardCharsets.UTF_8);
    return converter;
}

// Avro serialization (Apache Avro + Schema Registry)
// 1. Define .avsc schema
{
  "type": "record",
  "name": "OrderCreated",
  "namespace": "com.company.events",
  "fields": [
    {"name": "orderId", "type": "long"},
    {"name": "customerEmail", "type": "string"},
    {"name": "totalAmount", "type": "double"},
    {"name": "timestamp", "type": "long"}
  ]
}

// 2. Use AvroSerializer
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
        "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("schema.registry.url", "http://schema-registry:8081");
```

### Common Mistakes

1. **No retry configuration** — message lost on first failure
2. **No dead-letter queue** — failed messages block the queue (RabbitMQ) or log silently (Kafka)
3. **Re-queueing without limit** — infinite retry loop
4. **Deserialization errors** — wrong schema, breaking changes, missing `spring.json.trusted.packages`
5. **Synchronous processing in listeners** — blocks the consumer thread pool
6. **Not setting `auto-offset-reset` correctly** — new consumers miss or replay old messages

> 💡 **Pro Tip:** Always design your messaging schemas with backward compatibility in mind. Add new fields as optional, never remove fields, and use schema registries (Confluent Schema Registry for Avro, or JSON Schema) to enforce compatibility on produce and consume. A breaking schema change can silently drop messages in production.

### Best Practices

- Always configure retry with exponential backoff
- Always configure dead-letter queue (RabbitMQ) or DLT (Kafka)
- Use JSON for simplicity, Avro/Protobuf for schema evolution in production
- Set `acks: all` for Kafka producers (strongest durability)
- Enable idempotent producers in Kafka to prevent duplicates
- Use `@RetryableTopic` in Kafka for automated DLQ handling
- Monitor consumer lag (Kafka) or queue depth (RabbitMQ) in production
- Test message schema evolution with backward/forward compatibility

### Interview Questions

1. Compare RabbitMQ vs Kafka — when to use which?
2. How does dead-letter queue work in RabbitMQ?
3. Explain Kafka consumer groups and partition assignment.
4. What is idempotent producer in Kafka?
5. How do you handle message deserialization errors?
6. What is the difference between `@RabbitListener` and `@KafkaListener`?
7. How does Spring AMQP's `Jackson2JsonMessageConverter` work?

### Practical Exercises

1. Set up RabbitMQ topology: exchange, queue, DLQ, binding; produce and consume messages
2. Configure retry with exponential backoff for a consumer
3. Set up Kafka topic with `@KafkaListener` and `@RetryableTopic`
4. Implement Avro serialization with schema registry
5. Build a message producer that sends 10,000 messages and measure throughput

### Mini Project: Event-Driven Order Processing

Build an event-driven system:
- **Order Service** publishes `OrderPlacedEvent` (JSON)
- **Inventory Service** consumes and updates stock
- **Notification Service** consumes and sends email confirmation
- **Dead-letter handling**: after 3 retries, messages go to DLQ
- **Kafka version**: uses `@RetryableTopic` with DLT; Avro serialization
- **RabbitMQ version**: DLQ with `x-dead-letter-exchange`
- **Monitoring**: expose RabbitMQ queue depth / Kafka consumer lag as Actuator metrics

---

## Chapter 8: Deployment

### Introduction

Deploying a Spring Boot application requires packaging, containerization, orchestration, and health management. From a simple Docker container to a full Kubernetes deployment, Spring Boot's production-ready features make it straightforward.

### Why It Matters

- Containers ensure consistency across environments
- Kubernetes provides self-healing, scaling, and rolling updates
- Health checks enable automated recovery
- ConfigMaps and Secrets externalize configuration
- Helm simplifies complex deployments

### Theory

#### Dockerfile for Spring Boot

```dockerfile
# ---- Build stage ----
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
RUN ./mvnw dependency:go-offline -B
COPY src src
RUN ./mvnw package -DskipTests

# ---- Runtime stage ----
FROM eclipse-temurin:21-jre-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
USER appuser
ENTRYPOINT ["java", "-jar", "app.jar"]

# Alternative: multi-stage with layered JAR for better caching
# Use spring-boot-jarmode-layertools to extract layers
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
RUN ./mvnw dependency:go-offline -B
COPY src src
RUN ./mvnw package -DskipTests
RUN java -Djarmode=layertools -jar target/*.jar extract --destination extracted

FROM eclipse-temurin:21-jre-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --from=build --chown=appuser:appgroup app/extracted/dependencies/ ./
COPY --from=build --chown=appuser:appgroup app/extracted/spring-boot-loader/ ./
COPY --from=build --chown=appuser:appgroup app/extracted/snapshot-dependencies/ ./
COPY --from=build --chown=appuser:appgroup app/extracted/application/ ./
USER appuser
ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]
```

#### docker-compose (App + DB + Cache + Message Broker)

```yaml
version: "3.9"

services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: paymentdb
      POSTGRES_USER: app
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d paymentdb"]
      interval: 10s
      timeout: 5s
      retries: 5
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s

  rabbitmq:
    image: rabbitmq:4-management-alpine
    environment:
      RABBITMQ_DEFAULT_USER: app
      RABBITMQ_DEFAULT_PASS: ${RABBIT_PASSWORD}
    ports:
      - "5672:5672"
      - "15672:15672"
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "check_port_connectivity"]
      interval: 15s
      timeout: 5s
      retries: 5

  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/paymentdb
      SPRING_DATASOURCE_USERNAME: app
      SPRING_DATASOURCE_PASSWORD: ${DB_PASSWORD}
      SPRING_REDIS_HOST: redis
      SPRING_RABBITMQ_HOST: rabbitmq
      SPRING_RABBITMQ_USERNAME: app
      SPRING_RABBITMQ_PASSWORD: ${RABBIT_PASSWORD}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy

volumes:
  pgdata:
```

#### Kubernetes Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  labels:
    app: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
        - name: order-service
          image: registry.company.com/order-service:1.0.0
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: k8s
            - name: SPRING_DATASOURCE_URL
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: datasource-url
            - name: SPRING_DATASOURCE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: password
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "1Gi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 3
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 20
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 2
          startupProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
            failureThreshold: 30
```

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  datasource-url: jdbc:postgresql://postgres-cluster:5432/paymentdb
  spring.application.name: order-service
```

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: c3VwZXJzZWNyZXQ=  # base64 "supersecret"
  username: YXBw                   # base64 "app"
```

#### Health Checks — Liveness, Readiness, Startup

Spring Boot Actuator provides three health endpoints:

```yaml
# application-kubernetes.yml
management:
  endpoint:
    health:
      show-details: always
      group:
        liveness:
          include: livenessState,ping
        readiness:
          include: readinessState,db,redis,rabbitmq
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  health:
    readinessstate:
      enabled: true
    livenessstate:
      enabled: true
```

| Probe | Endpoint | Purpose |
|-------|----------|---------|
| Liveness | `/actuator/health/liveness` | Is the app alive? Restart if fails |
| Readiness | `/actuator/health/readiness` | Is the app ready to serve traffic? |
| Startup | `/actuator/health/readiness` | Has the app finished startup? |

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    @Autowired
    private DataSource dataSource;

    @Override
    public Health health() {
        try (Connection conn = dataSource.getConnection()) {
            if (conn.isValid(2)) {
                return Health.up().withDetail("database", "reachable").build();
            }
            return Health.down().withDetail("database", "unreachable").build();
        } catch (Exception e) {
            return Health.down(e).build();
        }
    }
}
```

#### Helm Chart

Helm packages Kubernetes YAML into reusable charts.

```
order-service-chart/
├── Chart.yaml          # metadata (name, version, description)
├── values.yaml         # default configuration values
├── values-dev.yaml     # dev overrides
├── values-prod.yaml    # prod overrides
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── configmap.yaml
    ├── secret.yaml
    ├── ingress.yaml
    ├── hpa.yaml      # horizontal pod autoscaler
    └── _helpers.tpl  # reusable templates
```

```yaml
# Chart.yaml
apiVersion: v2
name: order-service
description: Order Service for Payment Platform
version: 1.0.0
appVersion: 1.0.0
```

```yaml
# values.yaml
replicaCount: 3

image:
  repository: registry.company.com/order-service
  tag: latest
  pullPolicy: Always

service:
  port: 80
  targetPort: 8080

config:
  datasourceUrl: jdbc:postgresql://postgres:5432/paymentdb
  appName: order-service

resources:
  requests:
    memory: 512Mi
    cpu: 250m
  limits:
    memory: 1Gi
    cpu: 500m

probes:
  liveness:
    path: /actuator/health/liveness
    initialDelaySeconds: 30
  readiness:
    path: /actuator/health/readiness
    initialDelaySeconds: 20

ingress:
  enabled: true
  host: orders.company.com
  tls:
    enabled: true
    secretName: order-service-tls
```

```yaml
# templates/deployment.yaml — uses values.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "order-service.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "order-service.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "order-service.name" . }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.targetPort }}
          env:
            - name: SPRING_DATASOURCE_URL
              value: {{ .Values.config.datasourceUrl | quote }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          livenessProbe:
            httpGet:
              path: {{ .Values.probes.liveness.path }}
              port: {{ .Values.service.targetPort }}
            initialDelaySeconds: {{ .Values.probes.liveness.initialDelaySeconds }}
          readinessProbe:
            httpGet:
              path: {{ .Values.probes.readiness.path }}
              port: {{ .Values.service.targetPort }}
            initialDelaySeconds: {{ .Values.probes.readiness.initialDelaySeconds }}
```

```shell
# Helm commands
helm create order-service-chart
helm lint order-service-chart/
helm template order-service order-service-chart/  # dry-run render

# Install
helm install order-service order-service-chart/ -f values-dev.yaml --namespace dev

# Upgrade
helm upgrade order-service order-service-chart/ -f values-prod.yaml --namespace prod

# Rollback
helm rollback order-service 2 --namespace prod
```

### Common Mistakes

1. **No health checks** — Kubernetes can't detect if the app is alive
2. **Not configuring liveness vs readiness correctly** — liveness restarts, readiness removes from load balancer
3. **Hardcoded configs in Docker image** — use environment variables or ConfigMaps
4. **Running as root in container** — security risk; always create a non-root user
5. **No resource limits** — one container can starve the node
6. **Using `latest` tag** — unpredictable; always pin a version
7. **No `.dockerignore`** — bloated images from `target/`, `.git/`, etc.

> ✅ **Best Practice:** Use multi-stage Docker builds to produce minimal production images. The first stage compiles the application; the second stage contains only the JRE and the built JAR. This reduces image size from ~800MB to ~200MB, improves security by excluding build tools, and speeds up deployments.

### Best Practices

- Use **multi-stage Docker builds** to minimize image size
- Pin image tags to specific versions (never `latest`)
- Create non-root user in Docker; never run as root
- Configure all three probes: startup, liveness, readiness
- Use **Helm** for repeatable, parameterized deployments
- Externalize all environment-specific config via ConfigMaps and Secrets
- Enable Actuator health endpoints for Kubernetes probes
- Add **Horizontal Pod Autoscaler** for production workloads
- Use **Pod Disruption Budgets** to ensure availability during maintenance
- Set up **network policies** to restrict pod-to-pod communication

### Interview Questions

1. What's the difference between liveness and readiness probes?
2. How does a multi-stage Docker build reduce image size?
3. Explain how `ConfigMap` and `Secret` differ in Kubernetes.
4. What is Helm and why use it over raw YAML?
5. How do you handle database migrations in a Kubernetes deployment?
6. What is a HorizontalPodAutoscaler and how does it work?
7. How would you debug a Spring Boot app crashing in Kubernetes?

### Practical Exercises

1. Create a multi-stage Dockerfile for a Spring Boot app (target < 150MB)
2. Write docker-compose with app + PostgreSQL + Redis + health check dependencies
3. Deploy the app to Kubernetes with Deployment, Service, ConfigMap, Secret
4. Configure liveness and readiness probes pointing to Actuator endpoints
5. Create a Helm chart for the service with values for dev and prod

### Mini Project: Full Deployment Pipeline

Deploy the complete e-commerce system:
- **Dockerfiles** for all services (order, payment, notification, gateway)
- **Docker-compose** with app + PostgreSQL + Redis + RabbitMQ + Zipkin
- **Kubernetes manifests**: Deployment (3 replicas), Service, ConfigMap, Secret, Ingress
- **Helm chart**: parameterized values for dev/staging/prod
- **Health checks**: liveness + readiness probes using Spring Boot Actuator
- **Resource limits**: CPU/memory requests and limits on every container
- **Database migration**: Flyway or Liquibase as init container

---

## Revision Notes

### Core Spring Annotations Quick Reference

| Annotation | Package | Purpose |
|------------|---------|---------|
| `@Component` | `org.springframework.stereotype` | Generic bean |
| `@Service` | `org.springframework.stereotype` | Service layer bean |
| `@Repository` | `org.springframework.stereotype` | DAO/repository bean |
| `@Controller` | `org.springframework.stereotype` | MVC controller |
| `@RestController` | `org.springframework.web.bind.annotation` | REST controller |
| `@Autowired` | `org.springframework.beans.factory.annotation` | Inject bean by type |
| `@Qualifier` | `org.springframework.beans.factory.annotation` | Narrow by bean name |
| `@Value` | `org.springframework.beans.factory.annotation` | Inject property value |
| `@Profile` | `org.springframework.context.annotation` | Conditional bean per profile |
| `@Scope` | `org.springframework.context.annotation` | Bean scope |
| `@PostConstruct` | `jakarta.annotation` | Init callback |
| `@PreDestroy` | `jakarta.annotation` | Destroy callback |
| `@Configuration` | `org.springframework.context.annotation` | Java config class |
| `@Bean` | `org.springframework.context.annotation` | Bean declaration in config |
| `@PropertySource` | `org.springframework.context.annotation` | Load properties file |
| `@Transactional` | `org.springframework.transaction.annotation` | Declarative transaction |
| `@SpringBootApplication` | `org.springframework.boot.autoconfigure` | Boot entry point |

### Spring Boot Properties Cheat Sheet

```properties
# Server
server.port=8080
server.servlet.context-path=/api

# Datasource
spring.datasource.url=jdbc:postgresql://localhost:5432/db
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.hikari.maximum-pool-size=10

# JPA / Hibernate
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.format_sql=true

# Logging
logging.level.com.company=DEBUG
logging.level.org.springframework.web=INFO
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n

# Actuator
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=when-authorized

# Security
spring.security.oauth2.client.registration.google.client-id=${GOOGLE_CLIENT_ID}

# Kafka
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=app-group

# RabbitMQ
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
```

### Common Dependencies (Maven)

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.4.1</version>
</parent>

<dependencies>
    <!-- Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- Validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- Actuator -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <!-- JWT -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.12.6</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.12.6</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.12.6</version>
        <scope>runtime</scope>
    </dependency>

    <!-- OpenAPI -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>2.6.0</version>
    </dependency>

    <!-- H2 (test/dev) -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- PostgreSQL -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- DevTools -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

### Key Architectural Decisions Checklist

| Decision | Recommendation | Rationale |
|----------|---------------|-----------|
| Injection style | Constructor injection | Immutability, testability, null safety |
| Bean scope | Singleton for services | Stateless design |
| JPA fetch type | LAZY everywhere | Performance; use `JOIN FETCH` when needed |
| Key generation | `SEQUENCE` with allocationSize=50 | Hibernate batch insert optimization |
| Password hashing | BCrypt (strength 12) or Argon2 | Industry standard |
| API versioning | URI path (`/api/v1/`) | Simple, cache-friendly, easy to route |
| Error format | RFC 7807 ProblemDetail | Standardized, machine-readable |
| Inter-service comm | Async events for business logic; sync only for queries | Loose coupling, resilience |
| Circuit breaker | Resilience4j | Lightweight, Spring Boot native |
| Distributed tracing | Micrometer + Zipkin | Required for microservices |
| Container base image | Distroless or Alpine-based JRE | Minimal attack surface |
| K8s probes | All three (startup, readiness, liveness) | Proper lifecycle management |
| Config externalization | ConfigMap + Secret (K8s) or Config Server | No hardcoded values |

---

> **Next**: Part 5 — Cloud-Native & DevOps (Kubernetes, Terraform, CI/CD, Monitoring, Observability)
