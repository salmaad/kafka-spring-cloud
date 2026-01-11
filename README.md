#Activité Pratique : Kafka, Spring Cloud Stream et Kafka Streams

Ce projet implémente une architecture pilotée par les événements (**Event-Driven Architecture**) en utilisant **Apache Kafka**. L'objectif est de démontrer comment produire, consommer et analyser des flux de données en temps réel avec l'écosystème **Spring Cloud**.

---

##Fonctionnalités
- **Infrastructure :** Cluster Kafka (Broker + Zookeeper) via Docker.
- **Micro-services Spring Boot :** Utilisation du modèle fonctionnel de Spring Cloud Stream.
- **Real-time Analytics :** Traitement de flux avec Kafka Streams (fenêtrage, agrégation).
- **Dashboard Temps Réel :** Visualisation graphique des données via Server-Sent Events (SSE).

---

##Stack Technique
- **Java 21** & **Spring Boot 3**
- **Spring Cloud Stream** (Binder Kafka)
- **Kafka Streams** (Analyse de flux)
- **Docker / Docker Compose**
- **Lombok**
- **Smoothie.js** (Visualisation frontend)

---

##Étapes de Réalisation

### 1. Infrastructure Kafka
Le projet utilise Docker Compose pour démarrer l'environnement Kafka.
```bash
docker-compose up -d
```
<img width="1917" height="966" alt="image" src="https://github.com/user-attachments/assets/34619659-e373-48a9-8be4-9aaddff97558" />

### 2. Modèle de Données
Définition d'un Record Java pour représenter les événements de visite de page :
```java
public record PageEvent(String name, String user, Date date, long duration) {}
```

###3. Production et Consommation de Messages
Producer REST : Utilisation de StreamBridge pour envoyer des événements via un endpoint HTTP.

Supplier : Une fonction qui génère automatiquement un flux de données aléatoires toutes les 100ms.

Consumer : Une fonction qui écoute les messages et les affiche dans la console.

###4. Analyse en temps réel avec Kafka Streams
Mise en œuvre d'un pipeline de traitement pour calculer le nombre de visites par page sur une fenêtre glissante de 5 secondes :

Filtrage : Exclusion des événements avec une durée de visite trop courte.

Groupage : Par nom de page (P1, P2...).

Fenêtrage : TimeWindows de 5 secondes.

Agrégation : Calcul du nombre de visites (count) stocké dans un State Store nommé count-store.
<img width="957" height="1003" alt="image" src="https://github.com/user-attachments/assets/73efe24d-999d-4a9b-942b-e0f6beaf1451" />

###5. Visualisation Frontend
Backend : Un contrôleur expose les résultats du State Store via un flux SSE (text/event-stream).

Frontend : Une page HTML utilisant Smoothie.js pour afficher les courbes de trafic en temps réel pour chaque page.

<img width="954" height="1003" alt="image" src="https://github.com/user-attachments/assets/96781719-0ac7-4496-b4b1-fda2b36a28a3" />

##Configuration Principale
Le fichier application.properties définit les bindings entre les fonctions Java et les topics Kafka :
```Properties
# Définition des fonctions Java
spring.cloud.function.definition=pageEventConsumer;pageEventSupplier;kStreamFunction

# Configuration des Topics
spring.cloud.stream.bindings.pageEventSupplier-out-0.destination=T3
spring.cloud.stream.bindings.kStreamFunction-in-0.destination=T3
spring.cloud.stream.bindings.kStreamFunction-out-0.destination=T4

# Fréquence de commit pour Kafka Streams (1s)
spring.cloud.stream.kafka.streams.binder.configuration.commit.interval.ms=1000
```

##Comment tester le projet ?
Démarrer Kafka : docker-compose up -d

Lancer l'application : ./mvnw spring-boot:run

Tester le Producer : Accédez à http://localhost:8080/publish/T3/P1

Consulter les statistiques (JSON) : http://localhost:8080/analytics

Voir le Dashboard : Ouvrez http://localhost:8080/index.html dans votre navigateur.



###Référence
Projet réalisé dans le cadre de la formation sur les architectures distribuées, basé sur la démonstration du Pr. Mohamed Youssfi.
