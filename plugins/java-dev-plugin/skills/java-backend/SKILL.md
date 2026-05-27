---
name: java-backend
description: Développement Java/Spring Boot avec architecture DDD hexagonale (Spring Boot 4, Java 25). Détecte automatiquement le type de projet (API Web ou Worker) et charge les spécificités correspondantes.
---

# Socle Commun Java/Spring Boot

Tu es un expert Java/Spring Boot. Applique strictement les préconisations suivantes pour tout développements Springboot.

**Ressources complémentaires — Couches entrantes :**
- **API Web (WebMVC)** : Voir `references/application-web.md` pour les spécificités REST entrant
- **Worker (RabbitMQ)** : Voir `references/application-rabbit.md` pour les spécificités messaging entrant

**Ressources complémentaires — Couches sortantes :**
- **Appels HTTP sortants** : Voir `references/infrastructure-http.md` pour RestClient, TokenProvider, adaptateurs HTTP
- **Publication RabbitMQ** : Voir `references/infrastructure-rabbit.md` pour RabbitTemplate, publication de messages
- **Persistance JPA PostgreSQL** : Voir `references/infrastructure-jpa-postgre.md` pour JPA/Spring Data, Flyway, Testcontainers avec PostgreSQL

## Stack Technique

| Élément | Version/Valeur                                                         |
| --- |------------------------------------------------------------------------|
| Java | 25                                                                     |
| Spring Boot | 4.0.6                                                                  |
| Build | Maven                                                                  |
| Starters communs | starter-actuator, starter-validation, starter-restclient, starter-test |

## Architecture DDD (Hexagonale)

```
application/               # Adaptateurs entrants (voir cas spécifiques)
domain/                    # Logique métier pure
├── model/                 # Modèles métiers : classes, value classes, enum
├── service/               # Services métier (@Service autorisé)
├── repository/            # Interfaces (ports sortants)
└── exception/             # RuntimeException uniquement
infrastructure/            # Adaptateurs sortants
└── {api}/                 # Ex: directus/, salesforce/, adlinks/
    ├── client/            # RestClient + TokenProvider
    ├── repository/        # Implémentations des ports
    └── mapper/            # DTO ↔ Domain
config/                    # @Configuration globales
```

**Principes clés :**
- Le **domain** ne doit JAMAIS dépendre de l'infrastructure ou du server
- Les **repository interfaces** sont définies dans le domain, implémentées dans l'infrastructure
- Communication entre couches via des **ports** (interfaces)


## Configuration Spring

**Deux fichiers principaux :**

```
src/main/resources/
├── application.yml         # Configuration par défaut (dev local)
└── application-docker.yml  # Surcharges pour le déploiement Docker
```

> ❌ **Interdit** : Sections `---` avec `on-profile:` dans un même fichier

### Virtual Threads (obligatoire)

```yaml
spring:
  threads:
    virtual:
      enabled: true
```

## Construction d'image Docker

**Demander à l'utilisateur** quelle approche il souhaite avant de générer quoi que ce soit :

> « Pour la construction de l'image Docker, deux approches sont possibles :
> 1. **Buildpacks** (`spring-boot:build-image`) — aucun Dockerfile à maintenir, image optimisée automatiquement
> 2. **Dockerfile** — contrôle total, AOT + layering explicites, recommandé si la CI impose un Dockerfile ou si des optimisations de démarrage sont critiques
>
> Laquelle préférez-vous ? »

### Option 1 — Buildpacks

```bash
./mvnw spring-boot:build-image
```

La CI se branche directement sur cette commande Maven. Aucun Dockerfile n'est créé.

### Option 2 — Dockerfile (AOT + Layering)

Utilise l'image `bellsoft/liberica-openjre-debian:25.0.2-cds`, le layering Spring Boot et le cache AOT pour des démarrages optimisés.

```dockerfile
# Perform the extraction in a separate builder container
FROM docker.io/bellsoft/liberica-openjre-debian:25.0.2-cds AS builder
WORKDIR /builder
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=tools -jar application.jar extract --layers --destination extracted

# Runtime container
FROM docker.io/bellsoft/liberica-openjre-debian:25.0.2-cds
WORKDIR /application
COPY --from=builder /builder/extracted/dependencies/ ./
COPY --from=builder /builder/extracted/spring-boot-loader/ ./
COPY --from=builder /builder/extracted/snapshot-dependencies/ ./
COPY --from=builder /builder/extracted/application/ ./
# AOT cache training
RUN java -XX:AOTMode=record -XX:AOTConfiguration=app.aotconf -Dspring.context.exit=onRefresh -Dspring.profiles.active=aot-build -jar application.jar
RUN java -XX:AOTMode=create -XX:AOTConfiguration=app.aotconf -XX:AOTCache=app.aot -jar application.jar && rm app.aotconf

ENTRYPOINT exec java $JAVA_OPTS -Dspring.profiles.active=docker -XX:AOTCache=app.aot -jar application.jar
```

La CI se branche sur ce Dockerfile pour construire l'image.


## Prometheus — Compteurs d'erreurs API (obligatoire)

Tout appel API externe en erreur **doit** incrémenter un compteur Prometheus.

**Nommage :** `<app>_<api>_errors_total`

```java
@Component
public class AdFabConformiteAdapter implements ConformiteVisuelPort {
    private final AdFabClient client;
    private final Counter errorsCounter;

    public AdFabConformiteAdapter(AdFabClient client, MeterRegistry meterRegistry) {
        this.client = client;
        this.errorsCounter = Counter.builder("fabrication_worker_adfab_errors_total")
            .description("Nombre d'erreurs lors des appels à l'API AdFab")
            .register(meterRegistry);
    }

    @Override
    public void updateConformite(ConformiteVisuelNormalisee conformite) {
        try {
            client.updateStatut(conformite);
        } catch (Exception e) {
            errorsCounter.increment(); // ✅ .increment() pas .inc()
            throw e; // Relancer pour ne pas avaler l'erreur
        }
    }
}
```

> ❌ **Interdit** : appel API sans compteur d'erreur dans un adaptateur infrastructure
> ❌ **Interdit** : avaler l'exception après le `.increment()` sans la relancer

## Standards de Code

### Variables locales : `final var` obligatoire

```java
// ✅ Correct
final var commande = new Commande(...);
final var result = service.process(data);

// ❌ Interdit
Commande commande = new Commande(...); // Type explicite
var result = service.process(data);   // final manquant
```

**Exceptions (types explicites autorisés) :** champs de classe, paramètres, types de retour

### Principes généraux

| Principe | Détail |
| --- | --- |
| Immutabilité | records, List.of, Map.of |
| Programmation fonctionnelle | Stream API, Optional, lambdas |
| Pas de boucles | Utiliser Stream au lieu de for/while |
| Méthodes courtes | Max 20 lignes |
| Commentaires | Uniquement si logique complexe |

### Exceptions métier

- **Unchecked uniquement** (extends RuntimeException)
- Gestion centralisée (@ControllerAdvice, @RabbitListenerErrorHandler)
- Pas de try-catch dans le code métier

### Optional - API fluent obligatoire

```java
// ✅ Correct
final var commande = repository.findById(id)
    .orElseThrow(() -> new CommandeNotFoundException(id));

// ❌ Interdit
optional.get();              // Jamais
optional.isPresent() + get() // Anti-pattern
```

### Logging SLF4J

```java
private static final Logger logger = LoggerFactory.getLogger(CommandeService.class);
logger.info("Processing commande {}", commandeId); // ✅ Paramétré
logger.info("Processing " + commandeId);           // ❌ Concaténation
```

### Jackson 3 (obligatoire)

Spring Boot 4 utilise Jackson 3 par défaut :

```java
// ✅ Correct - Jackson 3
import tools.jackson.databind.ObjectMapper;
import tools.jackson.databind.JsonNode;

// ❌ Interdit - Jackson 2 (ancien package)
import com.fasterxml.jackson.databind.ObjectMapper;
```
La seule exception concerne le package 'com.fasterxml.jackson.annotation' qui lui n'a pas encore migré.


## Structure server/scheduler/ (commun)

```
server/
└── scheduler/  # @Scheduled, CommandLineRunner
```

## Tests - Principes Généraux

| Convention | Détail |
| --- | --- |
| Nommage | `shouldXxxWhenYyy()` ou `givenXxx_whenYyy_thenZzz()` |
| @DisplayName | **Pas utilisé** - nom de méthode explicite |
| Assertions | AssertJ pour toutes les assertions |
| Structure | Arrange-Act-Assert |

### Couverture cible

| Couche | Couverture minimale |
| --- | --- |
| Domain | ≥ 90% |
| Infrastructure | ≥ 80% |

### Outils par couche

| Couche | Outil | Approche |
| --- | --- |  --- |
| Domain | JUnit 5 pur | Fakes (pas Mockito), injection constructeur |
| Infrastructure - API | MockWebServer | Vérifier appels HTTP, désérialisation |
| Infrastructure - DB | Testcontainers | PostgreSQL, MongoDB, etc. |

### Tests d'architecture (ArchUnit)

```java
@AnalyzeClasses(packages = "com.example.app")
public class ArchitectureTest {
    @ArchTest
    static final ArchRule domainShouldNotDependOnAdapters =
        noClasses().that().resideInAPackage("..domain..")
            .should().dependOnClassesThat()
            .resideInAnyPackage("..infrastructure..", "..server..");

    @ArchTest
    static final ArchRule repositoriesShouldBeInterfaces =
        classes().that().resideInAPackage("..domain.repository..")
            .should().beInterfaces();

    @ArchTest
    static final ArchRule exceptionsShouldExtendRuntimeException =
        classes().that().resideInAPackage("..domain.exception..")
            .should().beAssignableTo(RuntimeException.class);
}
```
