# API Web (WebMVC)

> **Applique ces spécificités uniquement si le projet est identifié comme une API Web** (présence de `spring-boot-starter-web`, controllers REST, endpoints HTTP).

## Dépendances spécifiques

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>3.0.3</version>
</dependency>
<!-- Tests controllers -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc-test</artifactId>
    <scope>test</scope>
</dependency>
```

## Structure application/

```
application/
└── web/
    ├── controller/   # @RestController
    ├── dto/          # Request/Response DTOs
    ├── mapper/       # DTO ↔ Domain
    ├── config/       # Security, CORS, etc.
    └── exception/    # @ControllerAdvice
```

## Conventions REST

| Aspect | Convention |
| --- | --- |
| Nommage endpoints | kebab-case, pluriel (`/api/v1/commandes`) |
| GET | Lecture |
| POST | Création |
| PUT | Remplacement complet |
| PATCH | Modification partielle |
| DELETE | Suppression |
| Pagination | `?page=0&size=20&sort=createdAt,desc` |

### Codes retour

| Code | Signification |
| --- | --- |
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 404 | Not Found |
| 500 | Internal Server Error |

## Controllers - Retour direct avec @ResponseStatus

```java
// ✅ Correct - Retour direct avec @ResponseStatus
@GetMapping("/{id}")
public CommandeResponse getById(@PathVariable String id) {
    return commandeService.findById(id);
}

@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public CommandeResponse create(@Valid @RequestBody CreateCommandeRequest request) {
    return commandeService.create(request);
}

@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void delete(@PathVariable String id) {
    commandeService.delete(id);
}

// ❌ Éviter - ResponseEntity verbose
@GetMapping("/{id}")
public ResponseEntity<CommandeResponse> getById(@PathVariable String id) {
    return ResponseEntity.ok(commandeService.findById(id));
}
```

## Gestion d'erreurs centralisée avec @ResponseStatus

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class CommandeNotFoundException extends RuntimeException {
    public CommandeNotFoundException(String id) {
        super("Commande not found: " + id);
    }
}

@ResponseStatus(HttpStatus.BAD_REQUEST)
public class InvalidCommandeException extends RuntimeException {
    public InvalidCommandeException(String message) {
        super(message);
    }
}
```

### @ControllerAdvice pour cas spécifiques

Utiliser `@ControllerAdvice` uniquement pour les cas nécessitant une réponse personnalisée :

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        final var errors = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .toList();
        return new ErrorResponse("VALIDATION_ERROR", String.join(", ", errors));
    }
}

public record ErrorResponse(String code, String message) {}
```

## Documentation API avec SpringDoc OpenAPI

### Configuration application.yml

```yaml
springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
    operationsSorter: method
```

### Annotations sur les Controllers

```java
@RestController
@RequestMapping("/api/v1/commandes")
@Tag(name = "Commandes", description = "Gestion des commandes")
public class CommandeController {
    private final CommandeService commandeService;

    public CommandeController(CommandeService commandeService) {
        this.commandeService = commandeService;
    }

    @Operation(summary = "Récupérer une commande", description = "Retourne une commande par son ID")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Commande trouvée"),
        @ApiResponse(responseCode = "404", description = "Commande non trouvée")
    })
    @GetMapping("/{id}")
    public CommandeResponse getById(@PathVariable String id) {
        return commandeService.findById(id);
    }

    @Operation(summary = "Créer une commande")
    @ApiResponse(responseCode = "201", description = "Commande créée")
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public CommandeResponse create(@Valid @RequestBody CreateCommandeRequest request) {
        return commandeService.create(request);
    }
}
```

### Annotations sur les DTOs

```java
@Schema(description = "Requête de création de commande")
public record CreateCommandeRequest(
    @Schema(description = "Référence produit", example = "PROD-123")
    @NotBlank String productRef,
    @Schema(description = "Quantité", example = "5", minimum = "1")
    @Min(1) int quantity
) {}
```

## Tests spécifiques API (Slice test)

```java
@WebMvcTest(CommandeController.class)
class CommandeControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @MockitoBean
    private CommandeService commandeService;

    @Test
    void shouldReturnCommandeWhenExists() throws Exception {
        final var commande = new CommandeResponse("CMD-123", "PROD-001", 5);
        when(commandeService.findById("CMD-123")).thenReturn(commande);

        mockMvc.perform(get("/api/v1/commandes/CMD-123"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value("CMD-123"));
    }

    @Test
    void shouldReturn404WhenCommandeNotFound() throws Exception {
        when(commandeService.findById("CMD-999"))
            .thenThrow(new CommandeNotFoundException("CMD-999"));

        mockMvc.perform(get("/api/v1/commandes/CMD-999"))
            .andExpect(status().isNotFound());
    }
}
```
