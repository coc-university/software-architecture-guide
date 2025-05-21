# project-architecture-guide

In dieser Übersicht werden Entscheidungshilfen gegeben, um die passende Architektur für ein neues Projekt zu finden.  
Im fachlichen Abschnitt spielt Domain Driven Design (DDD) eine große Rolle.  
Bei den Technologien wird Java & Spring Boot beispielhaft referenziert.  

## 1) Fachliche Anforderungen ermitteln
- gemeinsam mit Fachexperten funktionale und nicht-funktionale Anforderungen (Qualitätsmerkmale) sammeln
- Geschäftsprozesse/Use-Cases grafisch modellieren 
- zb via Event Storming oder Domain Storytelling

## 2) Kontext-Übersicht erstellen
- Sub-Domänen bzw. Bounded Context ermitteln und Context-Map erstellen
  - die Frage klären, welche Bereiche gehören fachlich eng zusammen und welche nicht
  - also wo besteht eine hohe Kohäsion mit gleichzeitig geringe Kopplung zu anderen Bereichen
  - bzw wo gibt es Sprachgrenzen, also unterschiedliche Bedeutungen für denselben Begriff
- Schnitte einführen, um Kontext-Grenzen deutlich zu machen
- Kontexte kategorisieren
  - Core: Kern der Anwendung, zentrale Business-Prozesse
  - Supporting: Unterstützt den Core (Zusatz-Features)
  - Generic: kein Anwendungsbezug, aber ein notwendiges Übel (zb. Nutzerverwaltung)
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
  - a) 3-Layer (API, Business, DB)
  - b) Ports/Adapter, Hexagonal, Onion
- die Geschäftslogik sollte möglichst frei von Technologien sein (wenig Spring)
- Aufteilung von Logik und Daten
  - a) Transaction Script: 
    - Trennung von Logik und Daten/State (nicht klassisch objektorientiert)
    - alle Geschäftslogik liegt in Service-Klassen
    - DB Entitäten haben nur Daten/State (Anemic Domain Model)
  - b) Domain Model / Object Oriented Design
    - Logik und Daten/State gemeinsam in einer Klasse (Rich Domain Model)
    - Services sind sehr klein und delegieren nur weiter an die Domain Objekte

## 6) Service-Adapter nach außen entscheiden

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
- API Security 
  - zb OAuth2 Flow mit JWT (Spring Security Resource Server)
  - evtl. ein Gateway als zusätzlicher Schutz

### 6.2) Datenbank
- jeder Service hat seine eigene logische Datenbank (evtl. physisch kombiniert)
- Datenbank Technologie  
  - SQL: strukturierte Daten, komplexe Abfragen
    - Postgres, MySql, etc
    - Spring Data JPA (komplex) oder Data JDBC (einfach)
  - NoSQL: unstrukturierte Daten, einfache Abfragen, gute Skalierung
    - MongoDb, etc
    - Spring Data MongoDB
- Datenbank Interaktion
  - den aktuellen Zustand speichern oder Event Sourcing
  - lokale Transaktionen (@Transactional) für zusammenhängende Geschäftsprozesse
  - Saga Pattern: service-übergreifende Prozesse (ggf. Kompensationsoperation)
  - Transaction Outbox Pattern: garantierte Event-Zustellung über extra DB-Tabelle

## Dokumentation der Architektur
- Arc42: https://arc42.de/overview/
- Canvas: https://canvas.arc42.org
- https://github.com/feststelltaste/software-component-canvas