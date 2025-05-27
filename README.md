# project-architecture-guide

## Intro

In dieser Übersicht wird ein Schritt-für-Schritt Leitfaden aufgezeigt.  
Er soll dabei helfen die passende Architektur für ein neues Projekt zu finden.  
Generell haben unabhängige Kontexte und Kopplung/Kohäsion einen entscheidenden Einfluss.  
Es sind viele Konzepte von Domain Driven Design (DDD) enthalten.  
Denn die Fachlichkeit sollte in der Technik abgebildet werden.  
Bei den Technologien wird Java & Spring Boot beispielhaft referenziert.  

## 1) Fachliche Anforderungen ermitteln
- Alle relevanten Personen zusammen bringen
  - Meeting-Formate wie Event Storming oder Domain Storytelling
  - siehe auch [Link](https://www.youtube.com/watch?v=H1hzIFACDHE) bzw [Link](https://www.youtube.com/watch?v=EaKWQ1rsaqQ) 
  - analog mit Post-it's oder digital (zB Miro)
- gemeinsam mit Fachexperten die Domäne verstehen
- eine gemeinsame Sprache (Ubiquitous Language) finden
- Anforderungen sammeln
  - funktional: Fokus auf das "was" (Mehrwert für Kunde)
  - nicht-funktional: Fokus auf das "wie" (Qualitätsmerkmale), zb:
    - Performance (Antwortzeit, Durchsatz, Skalierbarkeit)
    - Zuverlässigkeit (Fehlertoleranz, Verfügbarkeit)
    - Sicherheit (Authentifizierung, Autorisierung)
    - Wartbarkeit (Modularität, Testbarkeit, Dokumentation)
    - Benutzbarkeit (Barrierefreiheit, Fehlermeldungen)
  - Beispiel: siehe [Link](https://youtu.be/nJtEvdxvfNQ?t=702)
- Geschäftsprozesse bzw Use-Cases grafisch modellieren
  - Akteure/Objekte bilden Knoten im Diagramm (später Service oder Entity im Code)
  - Aktivitäten sind Verknüpfungen/Kanten über beschriftete Pfeile (später API oder Methode)

## 2) Kontext-Übersicht erstellen
- Die Domäne in Sub-Domänen unterteilen
- Jede Sub-Domäne kann ein oder mehrere Bounded Contexts haben, siehe auch [Link](https://www.youtube.com/watch?v=yQgCmMBNle4)
- Bounded Context ermitteln und Context-Map (Landkarte) erstellen, siehe auch [Link](https://www.youtube.com/watch?v=c5H0APovhsw)
  - die Frage klären, welche Bereiche gehören fachlich eng zusammen und welche nicht
  - also wo besteht eine hohe Kohäsion (zusammengehörige Einheiten) 
  - mit gleichzeitig geringe Kopplung zu anderen Bereichen
  - jeder Kontext sollte möglichst isoliert und unabhängig sein, also wenig Abhängigkeiten haben
  - wo gibt es Sprachgrenzen, also unterschiedliche Bedeutungen für denselben Begriff
  - wie sieht die Verantwortlichkeit der Daten im Gesamtprozess aus
- Schnitte einführen, um Kontext-Grenzen deutlich zu machen
- Kontexte bzw Sub-Domänen kategorisieren
  - Core: Kern der Anwendung, zentrale Business-Prozesse (früher umsetzen, evtl. höher skalieren)
  - Supporting: Unterstützt den Core durch Zusatz-Features (wird erst später umgesetzt)
  - Generic: kein Anwendungsbezug, aber ein notwendiges Übel (kann man dazu kaufen, zb. Nutzerverwaltung)
- siehe auch Strategic Design von DDD, bzw [Link](https://www.youtube.com/watch?v=NvBsEnDgA4o) und [Link](https://www.youtube.com/watch?v=ttIRNyoLKqE)

## 3) Services definieren
- aus einem fachlichen Kontext sollen technische Bausteine entstehen
- Aufteilung
  - a) ein Service mit mehreren fachlichen Modulen (modular Monolith = Modulith)
    - über Java Packages oder via Maven-Module
    - bietet zum Projektstart ein einfaches Setup und geringe Kosten
  - b) mehrere separate Microservice, also eigenständig laufende Prozesse
    - initial evtl zu komplex für einfache Context-Map mit kleinem Team
    - kann im späteren Projektverlauf Vorteile bringen
  - siehe auch [Link](https://www.youtube.com/watch?v=6-Wu178sOEE)
- Einfluss-Faktoren für die Entscheidung:
  - fachliche Komplexität
  - Größe des Entwickler-Teams
  - Entwicklungsgeschwindigkeit (wie oft gibt es Releases/Deployments)
  - Betrieb (Skalierung, Sicherheit, Resilienz, Wartung, Kosten)
  - Datenhaltung
  - Technologie-Vielfalt 

## 4) Kontexte verknüpfen

### 4.1) Beziehungen analysieren
- alle Verknüpfungspunkte zwischen den Kontexten identifizieren
- Abhängigkeiten bzw Richtungen betrachten
  - Upstream, Downstream: liefernde und verbrauchende Kontexte
  - Conformist: Downstream muss sich anpassen, ohne Einfluss auf Upstream
  - Customer, Supplier: aktive Zusammenarbeit der Kontexte
  - Partnership: beide Seiten sind gleichberechtigt
  - Open Host Service: Kontext bietet Schnittstelle für beliebige Nutzer
  - Shared Kernel: gemeinsame Bibliothek (Lib)
  - Separate Ways: Kontexte haben nichts miteinander zu tun und bleiben unabhängig
- betrifft sowohl Services als auch Module in einem Service
- Möglichkeiten für die Koordination der Beziehungen
  - a) Orchestration (zentraler Punkt, zb Workflow-Engines)
  - b) Choreografie (verteilte Steuerung, zb über Events)
  - siehe auch [Link](https://www.informatik-aktuell.de/entwicklung/methoden/orchestrieren-oder-choreografieren-ueber-eine-streitfrage-in-microservices-architekturen.html)

### 4.2) APIs entwerfen
- API Konzept/Design
  - generell sollten Schnittstellen fachlich modelliert werden, siehe auch [Link](https://www.youtube.com/watch?v=K2eiHDtoo-A) 
  - a) CRUD: 
    - eher technisch formuliert, orientiert sich an DB-Operationen 
    - API: POST, GET,  PUT, DELETE -> DB: Create, Read, Update, Delete
    - Endpunkte werden anhand von Entitäten aufgebaut, also daten-getrieben (REST)
    - für simplen Service mit wenig Fachlogik geeignet
    - bei größeren Systemen ein Anti-Pattern
    - siehe auch [Link](https://www.youtube.com/watch?v=E9yx9w3GJk0)
  - b) CQRS: 
    - Commands: 
      - Aufträge, imperativ, POST, Seiteneffekt, evtl. nicht idempotent
      - kein PUT und DELETE wie bei REST, die Fachlichkeit entscheidet den Effekt
      - Beispiel: /command/register-book
    - Queries 
      - Abfragen, GET, idempotent
      - Query-Namen nutzen statt Entität
      - Beispiel: /query/top-selling-books
    - schreibende und lesende Aktionen werden getrennt (Responsibility Segregation)
    - fachlich sprechende Formulierungen in der Gegenwart nutzen
    - Aktionen sind wichtiger als Entitäten, also nicht daten-getrieben denken
    - nicht mehrere Aktionen in einem Request vermischen, separat halten
    - Endpunkte via Kategorien, nicht DB-Entitäten (zb. /inventory/register-book) 
    - CQRS kann auch auf die DB angewendet werden (eine DB für Write, eine für Read)
    - bzw kombinierbar mit Event Sourcing
    - siehe auch [Link](https://www.youtube.com/watch?v=cqNGAo-9pUE) bzw [Link](https://www.youtube.com/watch?v=hP-2ojGfd-Q)
  - c) Events: 
    - Benachrichtigungen austauschen über Ereignisse, die schon passiert sind
    - in der Vergangenheit formuliert (zb RejectedPayment)
    - Events können über Message-Queues verteilt werden
    - kann CQRS sinnvoll ergänzen
    - kann mit Event Sourcing kombiniert werden (DB Zustand darüber abbilden)
    - siehe auch [Link](https://www.youtube.com/watch?v=vS7sCJ1uezY)
- API Technologie
  - synchron
    - a) Plain Http + CQRS
      - Grundbausteine von Http: Url-Pfade, Http-Verben, Status-Codes, etc
      - genügt als Basis für CQRS (fachliche API)
    - a) REST: 
      - für einfache Service-to-Service Kommunikation
      - Basiert stark auf den Grundprinzipien von Http
      - nutzt technische CRUD Operationen
      - Zugriff auf Ressourcen über Url-Pfade
      - nutzt alle Http-Verben und Status-Codes
      - Paging, Sortierung, Filterung möglich
      - in der Praxis meist keine HATEOAS Links im Einsatz 
      - möglich Erweiterung: Reactive Stream (Spring Webflux, non-blocking)
    - b) gRPC: 
      - für sehr schnelle Service-to-Service Kommunikation
      - direkte Methodenaufrufe im anderen Service, keine Ressourcen
      - nutzt das binäre Format Protobuf (statt Http/Json) 
      - daher nicht direkt lesbar, schwerer zu debuggen
    - c) GraphQL: 
      - zwischen Frontend und Backend
      - nur ein Endpunkt, nur POST-Requests, immer Status-Code 200 
      - kein Over/Under-Fetching, selektieren von Properties
      - Caching ist schwieriger umsetzbar
      - Backend-Last abhängig von Frontend, potenzielles Risiko
    - siehe auch [Link](https://www.youtube.com/watch?v=NsdnGAAJfDk)
  - asynchron
    - Event-System: RabbitMQ, Kafka, etc.
    - Sonstiges: zb Webhooks, WebSockets, Server-Sent Events, etc
- API Trigger
  - a) ein einzelner Request, manuell ausgelöst
  - b) Batch-Verarbeitung (viele Requests)
  - c) Scheduling (zeitgesteuerte Requests)
  - d) event-getrieben (kein klassischer Request)
  - e) Streaming (kontinuierliche Verarbeitung eines Datenstroms)
- API Daten Modelle
  - das Daten-Schema eines anderen Services soll evtl. nicht übernommen werden
  - bedeutet es muss ein Mapping an der API erfolgen
  - so ist der Service unabhängiger gegenüber Änderungen
  - bzw Änderungen betreffen nicht direkt den Kern der Anwendung
  - in DDD: Anti Corruption Layer (ACL)
- API Security 
  - zb OAuth2 Flow mit JWT (Spring Security Resource Server)
  - evtl. ein Gateway als zusätzlicher Schutz

## 5) Datenfluss und Speicherung planen

### 5.1) Verteilte Datenhaltung koordinieren
- jeder Service hat seine eigene logische Datenbank
  - evtl. physisch kombiniert, aber kein Zugriff vom anderen Service
  - also keine Kopplung über die Datenbank, damit Zuständigkeit klar ist
- Datenaustausch
  - a) Pull-Modell:
    - Daten eines anderen Kontextes werden bei Bedarf vom Owner-Service abgefragt
    - ist einfacher und braucht weniger Speicherplatz
    - dauert aber länger und ist fehleranfällig
  - b) Push-Modell:
    - Daten eines anderen Kontextes werden im eigenen Service zusätzlich gespeichert/dupliziert
    - hier reicht unter Umständen eine Teilmenge, also nur so viel wie nötig
    - Aktualisierung der Daten per Event vom Owner-Service notwendig
    - ist komplizierter und braucht mehr Speicher
    - dafür viel schneller und man ist unabhängiger während der Verarbeitung
    - ist bei DDD üblich, denn Entitäten können in mehreren Bounded Contexts vorhanden sein
  - c) Kombination/Hybrid-Ansatz, je nach Art der Daten, bzw je nach Last im System
  - siehe auch [Link](https://www.youtube.com/watch?v=tvs-h8aCjCg)

### 5.2) Interaktion mit der Datenbank
- wie wird gespeichert
  - a) den aktuellen Zustand speichern (CRUD), keine Historie
  - b) oder Event Sourcing
    - Events speichern (nur hinten anfügen, nichts löschen)
    - Zustand per Replay ermitteln (alle Events „aufsummieren“)
    - bzw Zwischenstände (Snapshots) festhalten für bessere Performance
    - siehe auch [Link](https://www.youtube.com/watch?v=yFjzGRb8NOk) bzw [Link](https://www.youtube.com/watch?v=ss9wnixCGRY)
- bei Bedarf CQRS
  - schreibende und lesende Aktionen trennen, also zwei separate Datenbanken
  - Query-Seite ist optimiert für schnelles Lesen, evtl. denormalisiertes Modell
  - Synchronisation/Aktualisierung notwendig per Event
- Transaktionen
  - Ausführung zusammenhängender Geschäftsprozesse (alles oder nichts)
  - das System bleibt in einem gültigen Zustand
  - a) lokale Transaktionen innerhalb einer Klasse (zb @Transactional) 
  - b) Saga Pattern: service-übergreifende Prozesse (ggf. Kompensationsoperation)
  - c) Transaction Outbox Pattern: garantierte Event-Zustellung über extra DB-Tabelle, siehe auch [Link](https://www.youtube.com/watch?v=tQw99alEVHo)

### 5.2) Datenbank designen
- Datenbank Technologie  
  - a) SQL: strukturierte Daten, komplexe Abfragen
    - Postgres, MySql, etc
    - Spring Data JPA / Hibernate: komplex, mit Cache/Persistence-Context und Dirty Checking 
    - oder Data JDBC: einfacher, ohne Caching, führt SQL sofort aus, orientiert an DDD Aggregates
    - siehe auch [Link](https://www.youtube.com/watch?v=AnIouYdwxo0)
  - b) NoSQL: unstrukturierte Daten, einfache Abfragen, gute Skalierung
    - zb MongoDb via Spring Data MongoDB
    - speichern von Objekten ohne extra Entity-Klasse & Repository möglich
- Datenbank Tabellen Design
  - Tabellen normalisieren, um Redundanzen zu minimieren
  - große verschachtelte Graphen vermeiden, Konsistenzgrenzen einführen
  - wenn nötig mit IDs arbeiten statt direkt zu referenzieren
  - siehe auch DDD Tactical Design (Entity, Value Object, Aggregate)
  - und [Link](https://www.youtube.com/watch?v=xFl-QQZJFTA) bzw [Link](https://www.youtube.com/watch?v=BFXuFb40P8k)

## 6) Services intern unterteilen
- obere Ebene fachlich, danach technisch (Package by Feature, siehe auch [Link](https://www.youtube.com/watch?v=B1d95I7-zsw))
- ein Service (bzw jedes Modul davon) wird in Schichten/Ringe aufgeteilt
- jede Ebene sollte lose gekoppelt sein zur anderen (Interfaces)
- Die Beziehungen zwischen den Ebenen bzw Modulen kann zb Spring Modulith prüfen (ArcUnit)
- Klassen in Java Packages möglichst unsichtbar halten für die Außenwelt (package private)
- Varianten
  - a) Ports/Adapter, Hexagonal, Onion, Clean-Arc
    - innen liegt die fachliche Geschäftslogik, außen die technische Infrastruktur
    - von außen nach innen gerichtete Abhängigkeiten, auch wenn der Aufruf nach außen geht
    - siehe auch [Link](https://www.youtube.com/watch?v=JubdZIdLQ4M) bzw [Link](https://youtu.be/BFXuFb40P8k?t=2295)
    - Beispiel + Code, siehe [Link](https://reflectoring.io/spring-hexagonal/)
  - b) alt: Schichten (zb UI, Business, DB):
    - von oben nach unten gerichtete Abhängigkeit
    - betrachtet nicht weitere umgebende Komponenten
- die Geschäftslogik (Kern) sollte möglichst frei von Technologien sein (wenig Spring)
  - bei eingehenden Aufrufen (Inbound/Driving-Adapter) ist ein Interface optional
  - ausgehende Aufrufe (Outbound/Driven-Adapter) sollten entkoppelt sein
  - wenn das Interface (Port) im Business-Package liegt, dann dreht sich die Abhängigkeit
  - bedeutet es gibt keinen direkten Bezug zur Technologie/Implementierung
- das Framework (zb Spring) hilft bei der Umgebung
  - API (in): zb @Controller, @XxxMapping, @XxxListener
  - API (out): zb XxxTemplate, XxxClient
  - DB: zb @Repository, @Entity, @Table, @Document
- Aufteilung von Logik und Daten
  - a) Transaction Script:
    - Trennung von Logik und Daten/State (nicht klassisch objektorientiert)
    - DB Entitäten haben nur Daten/State (Anemic Domain Model)
    - alle Geschäftslogik liegt in Service-Klassen
    - geeignet für einfache Prozesse bzw CRUD-Systeme
    - kann irgendwann komplex werden (Service 1 -> Service 2 -> Service 3)
  - b) Domain Model / Object Oriented Design
    - Logik und Daten/State gemeinsam in einer Klasse (Rich Domain Model)
    - evtl. 2 Entities verwenden, ein technisches und ein fachliches Object (Mapping)
    - keine Setter nutzen, sondern stattdessen fachliche Methoden
    - Services sind sehr klein und delegieren nur weiter an die Domain Objekte
    - besser erweiterbar/verständlich in einer komplexen Umgebung
    - siehe auch [Link](https://youtu.be/VGhg6Tfxb60?t=1550)

## Architektur entwickelt sich weiter
- ein perfekter Entwurf zum Projektstart ist unrealistisch
- die Architektur muss stetig verfeinert und angepasst werden
- dabei sollten die Qualitätsmerkmale den Rahmen vorgeben, siehe auch [Link](https://www.heise.de/blog/Woran-erkennt-man-eine-gute-Softwarearchitektur-7541527.html)
- Gravierende Änderungen/Entscheidungen als ADR festhalten, siehe auch [Link](https://www.heise.de/hintergrund/Gut-dokumentiert-Architecture-Decision-Records-4664988.html)

## Technische Schulden 
- diese lassen sich nie komplett vermeiden
- Ursachen
  - fehlende Architekturplanung bzw Anpassung
  - veraltete oder unpassende Technologien
  - kurzfristige Entscheidungen aus Zeitdruck
  - fehlende Dokumentation und Wissensverlust
  - unzureichende Qualitätssicherung
- Beispiele
  - Architektur
    - enge Kopplung zwischen Komponenten
    - Vermischung von Schichten
    - keine klare Verantwortungszuordnung
    - schlechte Fehlerbehandlung/Logging/Monitoring
  - Framework & Infrastruktur
    - veraltete Bibliotheken, Frameworks, Server
    - unsichere Konfigurationen
    - keine Automatisierung (CI/CD-Pipeline)
    - schlechte Versionierung 
  - Code
    - duplizierte Abschnitte
    - nicht sprechende Namen
    - geringe Testabdeckung, bzw zu viel manuell
    - Verletzung von Clean Code-Prinzipien (SOLID, DRY, KISS)
- Umgang
  - kategorisieren nach Typ und Risiko
  - sichtbar machen über Dokumentation, zb TDR, siehe auch [Link](https://www.heise.de/blog/Technical-Debt-Records-Dokumentation-technischer-Schulden-9876115.html)
  - kommunizieren bei der Projektplanung
  - Auswirkungen/Kosten aufzeigen

## Dokumentation der Architektur
- die Doku sollte eher schlank gehalten sein 
- so können Änderungen einfach erkannt und integriert werden
- der Fokus sollte auf konstante/stabile Bereiche gelegt werden
- Elemente die sich noch häufig ändern nur auf hoher Flugebene anreißen
- Arc42: https://arc42.de/overview/
- Canvas: https://canvas.arc42.org
- https://github.com/feststelltaste/software-component-canvas

## Ablauf
![project-architecture-guide.drawio.pdf](project-architecture-guide.drawio.png)
