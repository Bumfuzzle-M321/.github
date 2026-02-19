# Bumfuzzle
Bumfuzzle ist ein Endgeräte-Monitoring-System. Es erlaubt das Deployment eines sogenannten SensorReporters auf beliebigen Geräten, wo dann Sensoren wie Temperatur, Lüftergeschwindigkeit und weitere ausgelesen werden und in einem Webclient zentral angezeigt werden können.

# Konzept
Folgende Microservices sollen erstellt werden:
## Microservices
- Backend
- SensorReporter (x-Mal, auf Endgeräten)
- Loki als Log-Aggregator
- Prometheus für Monitoring und Alerts
### Andere Docker Container
- Kafka Server
- PostgreSQL Datenbank für Backend
- Grafana
## Kommunikation der Microservices
Die Daten, welche auf den Endgeräten vom SensorReporter gesammelt werden, werden in ein Kafka-Topic geschrieben. Im Webclient kann man als User neue Geräte erfassen. Für jedes dieser Geräte erhält meinen einen eindeutigen Key, welcher sich aus der User-ID und einer generierten, eindeutigen Geräte-ID zusammensetzt. Diesen Key kann man dann als User in die Konfigurationsdatei des SensorReporters einfügen. Wenn der SensorReporter nun die Sensoren ausliest, dann schreibt er die gesammelten Daten in ein Kafka-Topic. Damit man die Daten einem User und Gerät zuordnen kann, werden die Sensordaten als JSON mit folgendem Format serialisiert: `{"<key>": "<sensor-data>"}`. Das Backend wiederum liest aus dem Kafka-Topic die Nachrichten aus, deserialisiert das JSON und schreibt die Daten in Datenbank. Durch den Key aus dem JSON können sie dem korrekten Gerät zugeordnet werde. Ist ein Client verbunden und zeigt Gerätedaten an, so werden die aktuellen Sensordaten regelmässig über eine Websocket-Verbindung geschickt und der Client kann diese live anzeigen.
 
Das Backend wiederum exposed eine REST-API, über welche unter anderem die Authentication, die Erstellung neuer Geräte und das Auslesen von historischen Sensordaten gehandhabt werden.
 
Wie schon geschrieben existiert ausserdem eine zusätzliche Websocket-Verbindung, über welche Sensordaten fortlaufend an den Client gestreamt werden.

Für das Monitoring und die Logs gibt es im SpringBoot Backend Micrometer und den Loki-Logback-Appender. Micrometer exposed einen Endpoint, von dem sich Prometheus die Monitoring-Daten holt. Der Logback-Appender wiederum, schreibt die Logs in Loki.

# Systeme und verwendete Technologien
Im Folgenden sind die Systeme aufgelistet, welche Technologien diese verwenden und was die wichtigsten Vorteile der verwendeten Technologien sind.

## Backend
Das Backend sammelt die Kafka Nachrichten der Endgeräte und speichert diese einerseits in der Datenbank und sendet sie per Websocket an den Client. Ausserdem verarbeitet es das Hinzufügen neuer Endgeräte, sowie das Auslesen dieser. Beinhnaltet auch eine Registrierung und ein Login.

- Java & Spring Boot
  - Simpel, Etabliert und im Team bekannt
- Micrometer
  - Einfache Anbindung an Prometheus für Metriken
- Loki Logback Appender
  - Einfache Anbindung an Loki für Logs
- Spring Boot Kafka
  - Benötigt, um Kafka Messages in Spring Boot auszulesen
## Logging und Monitoring
Die folgenden Dienste werden für Logging und Monitoring verwendet:
- Loki
  - Zum verarbeiten der Logs
- Prometheus
  - Zum verarbeiten der Metriken
- Grafana
  - Visualisierung der Metriken und Logs
## SensorReporter
Der SensorReporter wird auf den Endgeräten deployed und liest Sensorendaten aus und sendet diese per Kafka an das Backend.
- Ruby
  - Ruby hat viele vorgefertigte Libraries, die für unser Projekt Hilfreich sind. Macht Spass zu schreiben.
- RDKafka Ruby
  - Gem, mit dem man einfach Messages auf Kafka producen kann
## Frontend
- React
  - Simpel, Etabliert und im Team bekannt
## Datenbank
- PostgreSQL
  - PostgreSQL ist ein leistungsstarkes, Open Source, relationales Datenbanksystem, das für Zuverlässigkeit, Datenintegrität (ACID-Konformität) und Erweiterbarkeit bekannt ist
## Schaubild
Im Schaubild zu sehen, ist der grobe Aufbau und die Kommunikation der einzelnen Services. Die grünen Pfeile stellen dabei Kommunikation zwischen Containern bzw. Services dar, während die blauen für interne Kommunikation innerhalb von Services und die roten für den Fluss der Metrics und Logs stehen. Wie genau die Microservices Kommunizieren kann dem Punkt "Kommunikation der Microservices" weiter oben entnommen werden.

![Medien](https://github.com/user-attachments/assets/231a4d7c-a743-465e-b81a-e9c62eb36ba9)
## Arbeitsaufteilung
Die Arbeit wird etwa 50/50 aufgeteilt.
- Jamie:
  - Frontend
  - Backend
  - Docker Compose für Backend, DB, Kafka Server, Loki, Prometheus und Grafana
- Jannik:
  - Frontend
  - SensorReporter
  - Docker Compose für SensorReporter
    - Konfiguration der UUID
### Weggelassenes bei Krankheitsfällen
- Frontend
