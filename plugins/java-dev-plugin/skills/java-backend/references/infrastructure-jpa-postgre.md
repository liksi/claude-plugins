# Infrastructure — Persistance (Base de Données)

> **Applique ces spécificités uniquement si le projet persiste des données avec spring jpa  (présence de `spring-boot-starter-data-jpa` ou équivalent dans le pom.xml).

## Dépendances spécifiques

### PostgreSQL / MySQL (JPA)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
<!-- Tests -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
```


## Structure infrastructure/

```
infrastructure/
└── database/
    ├── entity/
    │   └── {Domain}Entity.java     # Entité JPA / Document Mongo
    ├── repository/
    │   ├── {Domain}JpaRepository.java   # Interface Spring Data
    │   └── {Domain}RepositoryAdapter.java  # Implémentation du port domain
    └── mapper/
        └── {Domain}EntityMapper.java   # Entity ↔ Domain
```

## Entités JPA

Les entités JPA sont **séparées des objets domain**. Elles vivent uniquement dans l'infrastructure.

```java
@Entity
@Table(name = "commandes")
public class CommandeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private String id;

    @Column(nullable = false)
    private String reference;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private StatutCommande statut;

    @CreationTimestamp
    private Instant createdAt;

    // Constructeur no-arg requis par JPA
    protected CommandeEntity() {}

    public CommandeEntity(String reference, StatutCommande statut) {
        this.reference = reference;
        this.statut = statut;
    }

    // Getters uniquement — pas de setters
}
```

> ❌ **Interdit** : réutiliser les records domain comme entités JPA
> ❌ **Interdit** : setters sur les entités — utiliser le constructeur

## Repository Spring Data

```java
public interface CommandeJpaRepository extends JpaRepository<CommandeEntity, String> {

    List<CommandeEntity> findByStatut(StatutCommande statut);

    @Query("SELECT c FROM CommandeEntity c WHERE c.reference LIKE :prefix%")
    List<CommandeEntity> findByReferencePrefix(@Param("prefix") String prefix);
}
```

## Adaptateur — Implémentation du port domain

```java
@Component
public class CommandeRepositoryAdapter implements CommandeRepositoryPort {

    private final CommandeJpaRepository jpaRepository;
    private final CommandeEntityMapper mapper;

    public CommandeRepositoryAdapter(CommandeJpaRepository jpaRepository,
                                     CommandeEntityMapper mapper) {
        this.jpaRepository = jpaRepository;
        this.mapper = mapper;
    }

    @Override
    public Commande save(Commande commande) {
        final var entity = mapper.toEntity(commande);
        final var saved = jpaRepository.save(entity);
        return mapper.toDomain(saved);
    }

    @Override
    public Optional<Commande> findById(String id) {
        return jpaRepository.findById(id).map(mapper::toDomain);
    }

    @Override
    public List<Commande> findByStatut(StatutCommande statut) {
        return jpaRepository.findByStatut(statut).stream()
            .map(mapper::toDomain)
            .toList();
    }
}
```

> Pas de compteur Prometheus ici — les erreurs de base de données déclenchent des exceptions Spring qui remontent naturellement.

## Mapper Entity ↔ Domain

```java
@Component
public class CommandeEntityMapper {

    public Commande toDomain(CommandeEntity entity) {
        return new Commande(entity.getId(), entity.getReference(), entity.getStatut());
    }

    public CommandeEntity toEntity(Commande commande) {
        return new CommandeEntity(commande.reference(), commande.statut());
    }
}
```

## Migrations Flyway

```
src/main/resources/db/migration/
├── V1__create_commandes_table.sql
└── V2__add_index_on_statut.sql
```

```sql
-- V1__create_commandes_table.sql
CREATE TABLE commandes (
    id         VARCHAR(36)  PRIMARY KEY,
    reference  VARCHAR(255) NOT NULL,
    statut     VARCHAR(50)  NOT NULL,
    created_at TIMESTAMP    NOT NULL DEFAULT NOW()
);
```

> ❌ **Interdit** : modifier une migration existante — toujours créer une nouvelle version

## Tests avec Testcontainers (@DataJpaTest)

```java
@DataJpaTest
@Testcontainers
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class CommandeRepositoryAdapterTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");

    @DynamicPropertySource
    static void postgresProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private CommandeJpaRepository jpaRepository;

    private CommandeRepositoryAdapter adapter;

    @BeforeEach
    void setUp() {
        adapter = new CommandeRepositoryAdapter(jpaRepository, new CommandeEntityMapper());
    }

    @Test
    void shouldSaveAndRetrieveCommande() {
        final var commande = new Commande(null, "REF-001", StatutCommande.EN_COURS);

        final var saved = adapter.save(commande);

        assertThat(saved.id()).isNotNull();
        final var found = adapter.findById(saved.id());
        assertThat(found).isPresent();
        assertThat(found.get().reference()).isEqualTo("REF-001");
    }

    @Test
    void shouldReturnEmptyWhenCommandeNotFound() {
        final var result = adapter.findById("unknown-id");

        assertThat(result).isEmpty();
    }

    @Test
    void shouldFindCommandesByStatut() {
        adapter.save(new Commande(null, "REF-001", StatutCommande.EN_COURS));
        adapter.save(new Commande(null, "REF-002", StatutCommande.TERMINE));

        final var results = adapter.findByStatut(StatutCommande.EN_COURS);

        assertThat(results).hasSize(1);
        assertThat(results.getFirst().reference()).isEqualTo("REF-001");
    }
}
```