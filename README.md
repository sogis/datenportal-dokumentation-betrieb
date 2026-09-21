# Betriebsdokumentation Datenportal

Prozessorientierte Anleitung für Jenkins, Datenblatt-Editor und Sodata auf
OpenShift. Einstieg: [Biblios-Buch](docs/biblios/master.adoc).

Aufbau der Dokumentation: Voraussetzungen → Jenkins und Seed → Publikationsbestand
initialisieren → Editor und Sodata → Gesamtabnahme. Danach dienen die Kapitel
«Laufender Betrieb» und «Störungen beheben» als Nachschlagewerk.

Die Anleitung erklärt die benötigten Einstellungen, ihre Wirkung und ihren
Konfigurationsort. S3/Garage und der Bucket werden vorausgesetzt. Die bisherigen
Notizen zu ENV, Repository, Betriebsmodus, Bootstrap und Seed sind in die Kapitel
integriert; die Datenblatt-ID-Prüfung wird bei Abnahme und Fehlerdiagnose behandelt.

Erster Wurf: mit den Komponentenquellen abgeglichen, noch nicht gegen produktives
AD, OpenShift und S3 abgenommen. Platzhalter vor Ausführung ersetzen.

Optional mit Asciidoctor als HTML rendern:

```sh
asciidoctor -o /tmp/datenportal-betrieb.html docs/biblios/master.adoc
```
