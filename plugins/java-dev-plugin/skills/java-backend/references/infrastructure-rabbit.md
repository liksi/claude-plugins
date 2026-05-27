# Infrastructure — Publication RabbitMQ Sortante

> **Applique ces spécificités uniquement si le projet publie des messages RabbitMQ** (présence de `spring-boot-starter-amqp` et d'un `RabbitTemplate` pour l'envoi de messages).

## Documentation

Chaque message publié devra faire l objet d'une documentation async api
Pour faire cela, utiliser [springwolf](https://github.com/springwolf/springwolf)

## Dépendances spécifiques

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<!-- async api doc -->
<dependency>
    <groupId>io.github.springwolf</groupId>
    <artifactId>springwolf-ui</artifactId>
    <version>${springwolf.version}</version>
</dependency>

<!-- Tests -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>rabbitmq</artifactId>
    <scope>test</scope>
</dependency>
```

## Structure infrastructure/

```
infrastructure/
└── amqp/
    ├── publisher/
    │   └── {Domain}Publisher.java  # Implémentation du port domain
    ├── config/
    │   └── RabbitConfig.java       # Sérialisation JSON uniquement
    └── model/
        └── {Domain}Message.java    # DTO message (record)
```

## Configuration RabbitMQ

Configurer uniquement la sérialisation JSON et les exchange allant être utilisés — les queues seront déclarées par les listeners

```java
@Configuration
public class RabbitConfig {

    @Bean
    public MessageConverter jackson3JsonMessageConverter(ObjectMapper objectMapper) {
        return new JacksonJsonMessageConverter(objectMapper);
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory,
                                         MessageConverter messageConverter) {
        final var template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(messageConverter);
        return template;
    }
}
```

> ❌ **Interdit** : déclarer des queues dans la config du publisher — c'est la responsabilité des listeners

## Publisher — Implémentation du port domain

```java
@Component
public class CommandePublisher implements CommandePublisherPort {

    private final RabbitTemplate rabbitTemplate;
    private final Counter errorsCounter;

    public CommandePublisher(RabbitTemplate rabbitTemplate, MeterRegistry meterRegistry) {
        this.rabbitTemplate = rabbitTemplate;
        this.errorsCounter = Counter.builder("myapp_rabbit_publish_errors_total")
            .description("Erreurs lors de la publication de messages RabbitMQ")
            .register(meterRegistry);
    }

    @Override
    public void publishCommandeCreee(Commande commande) {
        try {
            final var message = new CommandeCreeeMessage(commande.id(), commande.reference());
            rabbitTemplate.convertAndSend("commandes.exchange", "commande.creee", message);
        } catch (Exception e) {
            errorsCounter.increment();
            throw e;
        }
    }
}
```

> ❌ **Interdit** : publication sans compteur d'erreur
> ❌ **Interdit** : avaler l'exception après `.increment()`

## Messages (DTOs)

Les messages sont des records immutables — pas d'annotations Jackson sauf si nécessaire.

```java
public record CommandeCreeeMessage(String commandeId, String reference) {}
```

> ❌ **Interdit** : réutiliser les objets du domain comme messages — toujours mapper vers un DTO dédié

## Tests avec Testcontainers

```java
@SpringBootTest
@Testcontainers
class CommandePublisherTest {

    @Container
    @ServiceConnection
    static RabbitMQContainer rabbitMQ = new RabbitMQContainer("rabbitmq:3.12-management");

    @Autowired
    private CommandePublisher publisher;

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    void shouldPublishCommandeCreeeMessage() {
        final var commande = new Commande("CMD-123", "REF-001");

        publisher.publishCommandeCreee(commande);

        final var received = (CommandeCreeeMessage) rabbitTemplate.receiveAndConvert(
            "commandes.queue", 3000);
        assertThat(received).isNotNull();
        assertThat(received.commandeId()).isEqualTo("CMD-123");
    }

    @Test
    void shouldIncrementErrorCounterOnPublishFailure() {
        // Simuler une connexion coupée en arrêtant le container
        rabbitMQ.stop();

        assertThatThrownBy(() -> publisher.publishCommandeCreee(new Commande("CMD-123", "REF-001")))
            .isInstanceOf(Exception.class);

        final var counter = meterRegistry.find("myapp_rabbit_publish_errors_total").counter();
        assertThat(counter.count()).isEqualTo(1.0);
    }
}
```