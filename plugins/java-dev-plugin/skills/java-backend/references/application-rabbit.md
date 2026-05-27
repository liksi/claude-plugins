# Worker (RabbitMQ)

> **Applique ces spécificités uniquement si le projet est identifié comme un Worker** (présence de `spring-boot-starter-amqp`, listeners RabbitMQ, traitement de messages).

## Dépendances spécifiques

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aspectj</artifactId>
</dependency>
```

## Structure application/

```
application/
└── amqp/
    ├── listener/   # @RabbitListener (crée queues/bindings)
    ├── config/     # Connexion RabbitMQ uniquement
    └── model/      # DTOs messages
```

## RabbitMQ - Création automatique des queues

Les `@RabbitListener` sont responsables de :
- Déclarer les queues et bindings automatiquement
- Écouter les messages entrants
- Déléguer le traitement aux services du domain

```java
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "commandes.queue", durable = "true"),
    exchange = @Exchange(name = "commandes.exchange", type = ExchangeTypes.TOPIC),
    key = "commande.created"
))
public void onCommandeCreated(CommandeMessage message) {
    // Traitement du message
}
```

## Gestion d'erreurs

Utiliser `@RabbitListenerErrorHandler` pour la gestion centralisée des erreurs :

```java
@Configuration
public class RabbitErrorConfig {
    @Bean
    public RabbitListenerErrorHandler globalErrorHandler() {
        return (message, channel, throwable) -> {
            // Log et gestion de l'erreur
            throw new AmqpRejectAndDontRequeueException(throwable);
        };
    }
}
```

## Tests spécifiques Worker

```java
@Testcontainers
class CommandeListenerE2ETest {
    @Container
    static RabbitMQContainer rabbitMQ = new RabbitMQContainer("rabbitmq:3.12");

    @Test
    void shouldProcessCommandeWorkflow() {
        // Publier message RabbitMQ
        rabbitTemplate.convertAndSend("commandes.exchange", "commande.creer", message);

        // Attendre et vérifier
        await().atMost(5, SECONDS).untilAsserted(() -> {
            final var commande = repository.findByClientId(clientId);
            assertThat(commande).isNotNull();
        });
    }
}
```
