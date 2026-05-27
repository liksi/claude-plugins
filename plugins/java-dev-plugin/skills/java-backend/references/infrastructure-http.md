# Infrastructure — Appels HTTP Sortants (RestClient)

> **Applique ces spécificités uniquement si le projet émet des appels HTTP vers des APIs externes** (présence d'adaptateurs dans `infrastructure.http.*` appelant des services tiers via RestClient).

## Dépendances spécifiques

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-restclient</artifactId>
</dependency>
```

## Structure infrastructure/

```
infrastructure/
└── http/
    └── {api}/                              # Ex: commande/, directus/, salesforce/
        ├── {Api}{Port}Adapter.java         # Implémentation du port domain + DTOs internes
        └── {Api}RestClientConfig.java      # Bean RestClient (@Qualifier)
```

Les DTOs de réponse HTTP sont déclarés comme **records internes** à l'adaptateur — pas de fichier séparé sauf si réutilisés par plusieurs adaptateurs.

## Configuration du RestClient

Un bean RestClient par API externe, injecté via `@Qualifier`.

```java
@Configuration
public class CommandeRestClientConfig {

    @Bean
    public RestClient commandeRestClient(@Value("${commande.base-url}") String baseUrl) {
        return RestClient.builder()
            .baseUrl(baseUrl)
            .defaultStatusHandler(HttpStatusCode::isError, (request, response) -> {
                throw new CommandeApiException(response.getStatusCode(), request.getURI());
            })
            .build();
    }
}
```

## Adaptateur — Pattern complet

```java
@Component
public class ApiCommandeOperationProvider implements OperationProvider {

    private final RestClient commandeRestClient;
    private final Counter errorsCounter;

    public ApiCommandeOperationProvider(
            @Qualifier("commandeRestClient") RestClient commandeRestClient,
            MeterRegistry meterRegistry) {
        this.commandeRestClient = commandeRestClient;
        this.errorsCounter = Counter.builder("myapp_commande_errors_total")
            .description("Erreurs lors des appels à l'API Commande")
            .register(meterRegistry);
    }

    @Override
    public Optional<Operation> getOperation(String insertionId) {
        try {
            final var insertion = commandeRestClient.get()
                .uri(uriBuilder -> uriBuilder
                    .path("/insertions/{insertionId}")
                    .build(insertionId))
                .retrieve()
                .body(InsertionResponse.class);

            if (insertion == null || insertion.operation() == null) {
                return Optional.empty();
            }

            final var iri = insertion.operation();
            final var operationId = iri.substring(iri.lastIndexOf('/') + 1);

            return Optional.ofNullable(commandeRestClient.get()
                    .uri(uriBuilder -> uriBuilder
                            .path("/operations/{operationId}")
                            .build(operationId))
                    .retrieve()
                    .body(OperationResponse.class)
                    ).map(op -> new Operation(op.id(), op.opportuniteCrmId()));
        } catch (Exception e) {
            errorsCounter.increment();
            throw e;
        }
    }

    record InsertionResponse(String id, String operation) {}

    record OperationResponse(String id,
                             @JsonProperty("opportunite_crm_id") String opportuniteCrmId) {}
}
```

> `@JsonProperty`

**Points clés :**
- `@Qualifier` pour désigner le bon RestClient quand plusieurs beans existent
- DTOs comme **records internes** — déclarés dans l'adaptateur, visibilité package
- **URI builder** pour les URLs avec variables de chemin : `uriBuilder -> uriBuilder.path(...).build(...)`
- Retourner `Optional<T>` plutôt que `null` depuis le port domain
- Compteur Prometheus obligatoire sur le bloc try/catch englobant

> ❌ **Interdit** : retourner `null` — utiliser `Optional.empty()`
> ❌ **Interdit** : appel API sans compteur d'erreur dans l'adaptateur
> ❌ **Interdit** : avaler l'exception après `.increment()`

## Gestion des erreurs HTTP

Centraliser dans le `defaultStatusHandler` du bean RestClient, pas dans chaque appel.

```java
public class CommandeApiException extends RuntimeException {
    public CommandeApiException(HttpStatusCode status, URI uri) {
        super("Commande API error %s on %s".formatted(status, uri));
    }
}
```

## Authentification (TokenProvider)

Si l'API nécessite un token, utiliser un intercepteur sur le bean RestClient :

```java
@Bean
public RestClient securedRestClient(@Value("${api.base-url}") String baseUrl,
                                    TokenProvider tokenProvider) {
    return RestClient.builder()
        .baseUrl(baseUrl)
        .requestInterceptor((request, body, execution) -> {
            request.getHeaders().setBearerAuth(tokenProvider.getToken());
            return execution.execute(request, body);
        })
        .build();
}
```

## Tests avec WireMock (`@RestClientTest`)

Utiliser `@RestClientTest` (slice test Spring Boot) + WireMock pour tester l'adaptateur avec son vrai bean RestClient.

Dependance Maven: 
Cette dependace est dans le 
```xml
<dependency>
    <groupId>org.wiremock</groupId>
    <artifactId>wiremock-standalone</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@RestClientTest(components = ApiCommandeOperationProvider.class)
@Import({CommandeRestClientConfig.class, ApiCommandeOperationProviderTest.OAuth2TestConfig.class})
class ApiCommandeOperationProviderTest {

    private static WireMockServer wireMockServer;

    @Autowired
    private ApiCommandeOperationProvider provider;

    @Autowired
    private InMemoryOAuth2AuthorizedClientService authorizedClientService;

    @BeforeAll
    static void setUp() {
        wireMockServer = new WireMockServer(options().dynamicPort());
        wireMockServer.start();
        WireMock.configureFor("localhost", wireMockServer.port());
    }

    @AfterAll
    static void tearDown() {
        wireMockServer.stop();
    }

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.security.oauth2.client.provider.commande.token-uri",
            () -> wireMockServer.baseUrl() + "/oauth/token");
        registry.add("http.commande.url", wireMockServer::baseUrl);
    }

    @BeforeEach
    void resetState() {
        wireMockServer.resetAll();
        authorizedClientService.removeAuthorizedClient("commande", "anonymousUser");
    }

    @Test
    void shouldReturnMappedOperationWhenFound() {
        stubTokenEndpoint();
        wireMockServer.stubFor(get(urlPathEqualTo("/insertions/INS-1"))
            .willReturn(okJson("""
                {"id":"INS-1","operation":"/operations/OP-42"}
                """)));
        wireMockServer.stubFor(get(urlPathEqualTo("/operations/OP-42"))
            .willReturn(okJson("""
                {"id":"OP-42","opportunite_crm_id":"CRM-99","packs":[]}
                """)));

        final var result = provider.getOperation("INS-1");

        assertThat(result).hasValueSatisfying(op -> assertThat(op.id()).isEqualTo("OP-42"));
    }

    @Test
    void shouldReturnEmptyWhenInsertionHasNoOperation() {
        stubTokenEndpoint();
        wireMockServer.stubFor(get(urlPathEqualTo("/insertions/INS-1"))
            .willReturn(okJson("""{"id":"INS-1","operation":null}""")));

        assertThat(provider.getOperation("INS-1")).isEmpty();
    }

    @Test
    void shouldCallInsertionThenOperationEndpoints() {
        stubTokenEndpoint();
        wireMockServer.stubFor(get(urlPathEqualTo("/insertions/INS-1"))
            .willReturn(okJson("""{"id":"INS-1","operation":"/operations/OP-42"}""")));
        wireMockServer.stubFor(get(urlPathEqualTo("/operations/OP-42"))
            .willReturn(okJson("""{"id":"OP-42","opportunite_crm_id":"CRM-99","packs":[]}""")));

        provider.getOperation("INS-1");

        WireMock.verify(1, getRequestedFor(urlPathEqualTo("/insertions/INS-1")));
        WireMock.verify(1, getRequestedFor(urlPathEqualTo("/operations/OP-42")));
    }

    private void stubTokenEndpoint() {
        wireMockServer.stubFor(post("/oauth/token")
            .willReturn(okJson("""
                {"access_token":"test-token","token_type":"Bearer","expires_in":3600}
                """)));
    }

    @TestConfiguration
    static class OAuth2TestConfig {

        @Bean
        ClientRegistrationRepository clientRegistrationRepository(
                @Value("${spring.security.oauth2.client.provider.commande.token-uri}") String tokenUri) {
            final var registration = ClientRegistration.withRegistrationId("commande")
                .clientId("test-client-id")
                .clientSecret("test-client-secret")
                .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
                .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_POST)
                .tokenUri(tokenUri)
                .build();
            return new InMemoryClientRegistrationRepository(registration);
        }

        @Bean
        InMemoryOAuth2AuthorizedClientService authorizedClientService(
                ClientRegistrationRepository repository) {
            return new InMemoryOAuth2AuthorizedClientService(repository);
        }
    }
}
```

**Points clés :**
- `@RestClientTest` charge uniquement le composant testé et l'auto-config RestClient — pas de contexte Spring complet
- `@Import` pour charger la config du RestClient et les beans OAuth2 de test
- WireMock démarré une seule fois par classe (`@BeforeAll`) — `resetAll()` + clear du token OAuth2 entre chaque test
- `@DynamicPropertySource` pour câbler l'URL WireMock dans les propriétés Spring
- `WireMock.verify()` pour asserter que les bons endpoints ont été appelés