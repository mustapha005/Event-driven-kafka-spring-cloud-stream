# Event-driven Kafka avec Spring Cloud Stream

Projet maintenu par **Mustapha Aarab** — [mostaphaaarabe663@gmail.com](mailto:mostaphaaarabe663@gmail.com).

Cette application publie des événements de visite de pages, les traite avec Kafka Streams et affiche les compteurs dans un graphique actualisé en temps réel. Les packages Java utilisent `net.aarab.springkafka`.

## Technologies

- Java 21, Spring Boot 3.4.2 et Spring Cloud 2024.0.0.
- Spring Cloud Stream avec les binders Kafka et Kafka Streams.
- Kafka et ZooKeeper via les images Confluent 7.3.0 définies dans `compose.yml`.
- Server-Sent Events et Smoothie Charts pour le graphique.

## Flux de données

| Composant | Entrée | Sortie / rôle |
| --- | --- | --- |
| `pageEventConsumer` | Topic `T1` | Affiche les événements reçus dans la console |
| `pageEventSupplier` | Génération périodique | Publie des visites de `P1` et `P2` dans `T6` |
| `kStream` | Topic `T6` | Compte les événements par page sur des fenêtres de cinq secondes ; écrit dans `T7` et le store `count-store` |
| `/publish` | Paramètres HTTP `name` et `topic` | Publie un événement avec `StreamBridge` |
| `/analytics` | Store `count-store` | Émet les compteurs chaque seconde au format SSE |

Les identifiants `U1`, `U2`, `P1` et `P2` sont des données de démonstration.

## Démarrage

Prérequis : JDK 21 avec `JAVA_HOME` configuré, Docker Compose et accès à Internet pour les dépendances et images.

Depuis la racine du projet :

```powershell
docker compose up -d
./mvnw.cmd spring-boot:run
```

Attendre que Kafka soit prêt avant de lancer l'application. Sous Linux ou macOS, utiliser `./mvnw` à la place de `./mvnw.cmd`. Avec Maven installé, `mvn spring-boot:run` convient également.

- Tableau de bord : <http://localhost:8080/>
- Publier une visite pour le graphique : <http://localhost:8080/publish?name=P1&topic=T6>
- Publier vers le consommateur de console : <http://localhost:8080/publish?name=P2&topic=T1>
- Flux de compteurs : <http://localhost:8080/analytics>

Le store Kafka Streams doit être initialisé pour que `/analytics` puisse répondre. Le graphique charge sa bibliothèque JavaScript depuis un CDN et nécessite donc une connexion Internet.

## Compilation et test

```powershell
./mvnw.cmd clean verify
```

Le test fourni charge le contexte Spring complet et nécessite un broker Kafka accessible sur `localhost:9092`. Pour compiler sans lancer le test :

```powershell
./mvnw.cmd clean package -DskipTests
```

Les commandes de consultation des topics sont regroupées dans `commands.txt`.

Pour arrêter les conteneurs de ce projet :

```powershell
docker compose down
```

Cette configuration sert à une démonstration locale : elle utilise un seul broker, des communications PLAINTEXT et aucun mécanisme d'authentification.
