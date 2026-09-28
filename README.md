# Betriebsdokumentation Datenportal

Prozessorientierte Anleitung für APISIX, Jenkins, Datenblatt-Editor und Sodata auf
OpenShift. Einstieg: [Biblios-Buch](docs/biblios/master.adoc).

Optionaler Einstieg: **Datenportal lokal kennenlernen** führt mit Docker Compose
und vorhandenen Images vom Start bis zu einer Testpublikation. Eine Grafik und
konkrete URLs zeigen insbesondere das Zusammenspiel über APISIX.

Produktiver Aufbau der Dokumentation: Voraussetzungen → APISIX und öffentliche Zugänge → Jenkins und Seed → Publikationsbestand
initialisieren → Editor und Sodata → Gesamtabnahme. Danach dienen die Kapitel
«Laufender Betrieb» und «Störungen beheben» als Nachschlagewerk.

Die Anleitung erklärt die benötigten Einstellungen, ihre Wirkung und ihren
Konfigurationsort. Im produktiven Ablauf werden S3/Garage und der Bucket vorausgesetzt;
der optionale lokale Durchlauf richtet Garage und einen Testbucket ein. Die bisherigen
Notizen zu ENV, Repository, Betriebsmodus, Bootstrap und Seed sind in die Kapitel
integriert; die Datenblatt-ID-Prüfung wird bei Abnahme und Fehlerdiagnose behandelt.

Erster Wurf: mit den Komponentenquellen abgeglichen, noch nicht gegen produktives
AD, OpenShift und S3 abgenommen. Platzhalter vor Ausführung ersetzen.

Optional mit Asciidoctor als HTML rendern:

```sh
asciidoctor -o /tmp/datenportal-betrieb.html docs/biblios/master.adoc
```

## Prüfstand des lokalen Einstiegs

[Prüfprotokoll vom 28.09.2026](pruefprotokolle/2026-09-28-lokaler-stack.md):
Registry-Start mit expliziten veröffentlichten Image-Tags, Gateway-Zugänge und
Seed geprüft. Die vollständige Testpublikation bleibt wegen zu wenig freiem
Docker-Speicher in der Testumgebung noch abzunehmen. Das Protokoll nennt die
verwendeten Quellstände, Image-Digests und offenen Prüfungen.
