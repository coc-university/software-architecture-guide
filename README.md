# project-architecture-guide

In dieser Übersicht werden Entscheidungshilfen gegeben, um die passende Architektur für ein neues Projekt zu finden.  
Im fachlichen Abschnitt spielt Domain Driven Design (DDD) eine große Rolle.  
Bei den Technologien wird Java & Spring Boot beispielhaft referenziert.  

## 1) Fachliche Anforderungen ermitteln
- gemeinsam mit Fachexperten funktionale und nicht-funktionale Anforderungen (Qualitätsmerkmale) sammeln
- Geschäftsprozesse/Use-Cases grafisch modellieren, zb via Post-it oder Miro 
- Über Meeting-Formate wie Event Storming oder Domain Storytelling

## 2) Kontext-Übersicht erstellen
- Sub-Domänen bzw. Bounded Context ermitteln und Context-Map erstellen
  - die Frage klären, welche Bereiche gehören fachlich eng zusammen und welche nicht
  - also wo besteht eine hohe Kohäsion mit gleichzeitig geringe Kopplung zu anderen Bereichen
  - bzw wo gibt es Sprachgrenzen, also unterschiedliche Bedeutungen für denselben Begriff
- Schnitte einführen, um Kontext-Grenzen deutlich zu machen
- Kontexte kategorisieren
  - Core: Kern der Anwendung, zentrale Business-Prozesse (früher umsetzen, evtl. höher skalieren)
  - Supporting: Unterstützt den Core durch Zusatz-Features (wird erst später umgesetzt)
  - Generic: kein Anwendungsbezug, aber ein notwendiges Übel (kann man dazu kaufen, zb. Nutzerverwaltung)
- siehe auch Strategic Design von DDD

## 3) Geschäftsprozesse über Kontext-Grenzen modellieren
- Klärung wie fachliche Prozesse abgebildet werden sollen
  - Orchestrieren vs Choreografieren
  - BPMN/Workflow-Engines 
- wie arbeiten die Kontexte zusammen
  - Abhängigkeiten zwischen den Kontexten (Upstream, Downstream)
  - Kommunikation von einem zum anderen Kontext
    - direkter Aufruf über API
    - asynchron, event-getrieben

## 4) Services definieren
- aus einem fachlichen Kontext sollen technische Bausteine entstehen
- Aufteilung
  - ein Service mit mehreren fachlichen Modulen (modular Monolith = Modulith)
    - a) über Java Packages
    - b) via Maven-Module
  - mehrere separate Services, also eigenständig laufende Prozesse (Microservice)
- Einfluss-Faktoren für die Entscheidung:
  - fachliche Komplexität
  - Größe des Entwickler-Teams
  - Betrieb (Skalierung, Sicherheit, Resilienz, Wartung, Kosten)
  - Datenhaltung
  - Technologie-Vielfalt 

## 5) Service unterteilen
- ein Service wird zur besseren Übersicht in Schichten aufgeteilt
- jede Schicht sollte lose gekoppelt sein zur anderen (Interfaces, Spring Modulith)
  - a) 3-Layer (API, Business, DB): von oben nach unten gerichtet
  - b) Ports/Adapter, Hexagonal, Onion: von außen nach innen gerichtet
- die Geschäftslogik sollte möglichst frei von Technologien sein (wenig Spring)
- Aufteilung von Logik und Daten
  - a) Transaction Script: 
    - Trennung von Logik und Daten/State (nicht klassisch objektorientiert)
    - alle Geschäftslogik liegt in Service-Klassen
    - DB Entitäten haben nur Daten/State (Anemic Domain Model)
  - b) Domain Model / Object Oriented Design
    - Logik und Daten/State gemeinsam in einer Klasse (Rich Domain Model)
    - Services sind sehr klein und delegieren nur weiter an die Domain Objekte

## 6) Kopplungen vom Service nach außen

### 6.1) API
- API Design 
  - a) CRUD: eher technisch formuliert, orientiert an DB (für simplen Service)
  - b) CQRS: Command & Queries, fachlich sprechende Aktionen
  - c) Events: Benachrichtigung trifft ein oder wird versendet
- API Technologie
  - a) REST: für einfache Service-to-Service Kommunikation
  - b) gRPC: falls sehr performante Aufrufe nötig sind
  - c) GraphQL: zwischen Frontend und Backend (selektieren von Properties)
  - d) Events: RabbitMQ, Kafka, etc.
  - e) Reactive Stream (Spring Webflux)
- API Security 
  - zb OAuth2 Flow mit JWT (Spring Security Resource Server)
  - evtl. ein Gateway als zusätzlicher Schutz

### 6.2) Datenbank
- jeder Service hat seine eigene logische Datenbank (evtl. physisch kombiniert)
- Datenhaltung service-übergreifend
  - a) Pull-Modell: 
    - Daten eines anderen Kontextes werden bei Bedarf vom Owner-Service abgefragt
    - ist einfacher und braucht weniger Speicherplatz
    - dauert aber länger und ist fehleranfällig
  - b) Push-Modell: 
    - Daten eines anderen Kontextes werden im eigenen Service zusätzlich gespeichert/dupliziert 
    - und per Event (Message-Queue) vom Owner-Service aktualisiert
    - ist komplizierter und braucht mehr Speicher 
    - dafür viel schneller und man ist unabhängiger während der Verarbeitung
- Datenbank Technologie  
  - a) SQL: strukturierte Daten, komplexe Abfragen
    - Postgres, MySql, etc
    - Spring Data JPA (komplex) oder Data JDBC (einfach)
  - b) NoSQL: unstrukturierte Daten, einfache Abfragen, gute Skalierung
    - MongoDb, etc
    - Spring Data MongoDB
- Datenbank Tabellen Design
  - siehe auch DDD Tactical Design (Entity, Value Object, Aggregate)
  - große verschachtelte Graphen vermeiden, Konsistenzgrenzen einführen
  - wenn nötig mit IDs arbeiten statt direkt zu referenzieren
- Datenbank Interaktion
  - den aktuellen Zustand speichern oder Event Sourcing
  - lokale Transaktionen (@Transactional) für zusammenhängende Geschäftsprozesse
  - Saga Pattern: service-übergreifende Prozesse (ggf. Kompensationsoperation)
  - Transaction Outbox Pattern: garantierte Event-Zustellung über extra DB-Tabelle

### 6.3) Weitere mögliche Kopplungen
- Gemeinsame Bibliothek (Lib), die von mehreren Services/Teams genutzt wird
- in DDD: Shared Kernel

## Dokumentation der Architektur
- Arc42: https://arc42.de/overview/
- Canvas: https://canvas.arc42.org
- https://github.com/feststelltaste/software-component-canvas