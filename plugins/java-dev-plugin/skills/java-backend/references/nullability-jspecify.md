# Null-safety avec JSpecify

> Voir aussi la doc officielle : https://docs.spring.io/spring-framework/reference/core/null-safety.html

## Principe

Non-null par défaut. Chaque package racine (`domain`, `application`, `infrastructure`, `config`) déclare `@NullMarked` dans son `package-info.java` :

```java
@NullMarked
package com.example.app.domain;

import org.jspecify.annotations.NullMarked;
```

Seuls les usages explicitement annotés `@Nullable` peuvent être `null`.

## Placement des annotations

Les annotations JSpecify ciblent l'usage du type (`@Target(TYPE_USE)`) : elles se placent immédiatement avant le type, sur la même ligne.

```java
// ✅ Correct
private @Nullable String fileEncoding;
public @Nullable Operation getOperation(String insertionId) { ... }

// ❌ Interdit
@Nullable private String fileEncoding;
@Nullable public Operation getOperation(String insertionId) { ... }
```

| Cas | Exemple |
| --- | --- |
| Champ | `private @Nullable String fileEncoding;` |
| Retour | `public @Nullable Operation getOperation(...)` |
| Générique | `List<@Nullable String>` (éléments nullables) |
| Tableau (éléments nullables) | `@Nullable Object[] array` |
| Tableau (tableau nullable) | `Object @Nullable [] array` |
| Type imbriqué | `Cache.@Nullable ValueWrapper` |

## Dépendance Maven

```xml
<dependency>
    <groupId>org.jspecify</groupId>
    <artifactId>jspecify</artifactId>
</dependency>
```

La version est gérée par le BOM Spring Boot 4.

## Configuration ErrorProne / NullAway (obligatoire)

```xml
<properties>
    <error-prone.version>2.42.0</error-prone.version>
    <nullaway.version>0.12.10</nullaway.version>
</properties>
```

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <compilerArgs>
            <arg>-XDcompilePolicy=simple</arg>
            <arg>--should-stop=ifError=FLOW</arg>
            <arg>-XDaddTypeAnnotationsToSymbol=true</arg>
            <arg>-Xplugin:ErrorProne
                -XepOpt:NullAway:JSpecifyMode=true
                -XepOpt:NullAway:OnlyNullMarked=true
                -XepOpt:NullAway:CustomContractAnnotations=org.springframework.lang.Contract
                -XepDisableAllChecks
                -Xep:NullAway:ERROR
                -XepExcludedPaths:.*/(src/test/java|target/spring-aot)/.*
                -XepDisableWarningsInGeneratedCode
            </arg>
        </compilerArgs>
        <annotationProcessorPaths>
            <path>
                <groupId>com.google.errorprone</groupId>
                <artifactId>error_prone_core</artifactId>
                <version>${error-prone.version}</version>
            </path>
            <path>
                <groupId>com.uber.nullaway</groupId>
                <artifactId>nullaway</artifactId>
                <version>${nullaway.version}</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

> `JSpecifyMode=true` exige JDK 22+ — compatible avec Java 25. `OnlyNullMarked=true` limite les vérifications aux packages `@NullMarked`.

## `Optional` vs `@Nullable`

| Cas | Convention |
| --- | --- |
| Retour de port domain / service / mapper | `@Nullable T` |
| Paramètre, champ, composant de record | `@Nullable T` (jamais `Optional`) |
| Retour de `JpaRepository` Spring Data (`findById`, ...) | `Optional<T>` toléré (imposé par le framework) |
| Chaînage fluent sur un résultat Spring Data | `Optional` fluent (`.map()`, `.orElseThrow()`) |

## Suppressions légitimes

```java
// Champ initialisé tardivement (ex. InitializingBean)
@SuppressWarnings("NullAway.Init")
private @Nullable String lazyField;

// Limitation de l'analyse de flux dans une lambda
@SuppressWarnings("NullAway") // Lambda
someStream.forEach(item -> { ... });

// Appel réflexif dont le retour non-null est garanti
@SuppressWarnings("NullAway") // Reflection
Object result = Class.forName("...").newInstance();
```

## Interdits

> ❌ **Interdit** : `org.springframework.lang.Nullable`, `@NonNullApi`, `@NonNullFields` (dépréciées en Spring 7, remplacées par JSpecify)
> ❌ **Interdit** : `jakarta.annotation.Nullable`, `javax.annotation.*` (JSR-305)
> ❌ **Interdit** : `Optional` en paramètre, en champ ou en composant de record
> ❌ **Interdit** : `@SuppressWarnings("NullAway")` sans commentaire justifiant la raison