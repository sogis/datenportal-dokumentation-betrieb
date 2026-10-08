# Betriebsdokumentation Datenportal

Prozessorientierte Anleitung für APISIX, Jenkins, Datenblatt-Editor und Sodata auf
OpenShift. Einstieg: [Biblios-Buch](docs/biblios/master.adoc).

Optionaler Einstieg: **Datenportal lokal kennenlernen** führt mit Docker Compose
und vorhandenen Images vom Start bis zu einer Testpublikation. Eine Grafik und
konkrete URLs zeigen insbesondere das Zusammenspiel über APISIX.

Der zweite optionale Einstieg **Datenportal mit OpenShift/CRC kennenlernen**
verwendet fertige Images, externes S3 und die gemeinsamen Kustomize-Manifeste
aus `datenportal-stack`. CRC bleibt ohne AD/AIO nutzbar; ein getrenntes
Betreiber-Overlay ergänzt AD, Secrets und Produktionsparameter. Die gemeinsamen
Image-Versionen aus Kapitel 2 sind für AMD64 und ARM64 veröffentlicht; der
OpenShift-Stack fixiert ihre Multiarch-Digests.

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

Für Hervorhebungen verwenden wir zwei Kategorien: `[NOTE]` («Hinweis») für
ergänzende Erklärungen und `[IMPORTANT]` («Wichtig») für Voraussetzungen und
Schutzregeln. Gleiche Sachverhalte erhalten in allen Kapiteln dieselbe Kategorie.

## Prüfstand des lokalen Einstiegs

[Automatischer Stackstart vom 08.10.2026](pruefprotokolle/2026-10-08-automatischer-stackstart.md):
Frischer und erneuter Start mit lokalen Themenrepo-Quellen sowie Registry-only
ohne Schwester-Checkouts, einschliesslich Seed, Erstpublikation, Portal,
erhaltenem Bestand und Container-Sperre bei getrenntem Host-Client.

[Registry-Prüfprotokoll vom 30.09.2026](pruefprotokolle/2026-09-30-registry-stack.md):
Mit einem isolierten Dev-Stack ohne Schwester-Checkouts geprüft: versionierte
Images, GitHub-Seed, Erstpublikation, CSV-Anlieferung über Jenkins, natives
Sodata mit Suche/Detailseite/Download sowie Neustart mit erhaltenem Bestand.
Die Anleitung verwendet jetzt das native Sodata-Image `0.1.11` mit dem Explore-Fix.
Der [Prüfnachweis für 0.1.11](pruefprotokolle/2026-09-30-sodata-0.1.11.md)
ergänzt den historischen Stackdurchlauf um den isolierten Image-Regressionstest
und die CI-Prüfungen auf AMD64 und ARM64.

Das [Prüfprotokoll vom 28.09.2026](pruefprotokolle/2026-09-28-lokaler-stack.md)
bleibt als historischer Nachweis des früheren Ablaufs erhalten.

## Prüfstand des OpenShift-Einstiegs

[Leserführung CRC → Zielcluster vom 30.09.2026](pruefprotokolle/2026-09-30-crc-cluster-uebergang.md):
Erstaufbau und Wiederanlauf getrennt, eigenständigen Clusteraufbau und
Jenkins-/Git-Zugänge präzisiert. Dokumentation gerendert und visuell geprüft;
18 Stack-Tests erfolgreich. Die erneute Live-Prüfung von CA-Export und HTTPS
bleibt wegen der nicht erreichbaren CRC-API bzw. Route offen.

[Abgleich Kapitel 2/3 vom 30.09.2026](pruefprotokolle/2026-09-30-kapitel-2-3-stack.md):
gemeinsame Multiarch-Images, isolierte Jenkins-/Credential-Prüfung, Remote-Seed,
Gateway-/Asset-Tests sowie CRC mit externem S3: Erstpublikation, CSV-Lieferung,
interner Portal-Reload und Browser-SQL mit 106 Zeilen erfolgreich. Wiederanlauf
mit erhaltenem Bestand geprüft. Details und verbleibende Grenzen stehen
im Prüfprotokoll.

[Prüfprotokoll CRC/OpenShift vom 28.09.2026](pruefprotokolle/2026-09-28-openshift-stack.md):
Manifeste, Registryarchitekturen, Gateway und statische Anwendungen lokal geprüft.
Cluster-, S3- und AD-Abnahme waren zu diesem Zeitpunkt noch offen.
