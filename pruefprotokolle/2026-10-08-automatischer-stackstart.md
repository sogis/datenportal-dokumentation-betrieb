# Automatischer lokaler Stackstart — 8. Oktober 2026

Geprüft wurde der gemeinsame Start über `datenportal-dev-stack/scripts/up.sh`
mit veröffentlichten Images. Jeder Durchlauf verwendete einen temporären
Stack-Checkout, einen eigenen Compose-Projektnamen, freie Ports und eigene
Volumes. Der bestehende lokale Stack und dessen Publikationsbestand wurden
nicht für diese Starttests verwendet. Nur die eigenen Testvolumes wurden
anschliessend entfernt; Diagnoseverzeichnisse bleiben lokal erhalten.

## Tatsächliche Containerdurchläufe

| Variante | Themenquelle | Erststand | Seed erster/zweiter Start |
|---|---|---|---|
| `working-tree` | Read-only `../datenportal-themenrepo`, lokaler Stand `ac0e1d0` mit den uncommitteten Dokumentationskorrekturen dieser Aufgabe | `3f8fc216-b50c-4681-b7fb-f0c1006d9b20` | 2 / 3 |
| `--registry-only` | Jenkins-Checkout aus `https://github.com/sogis/datenportal-themenrepo.git`, Branch `main`; keine Schwester-Checkouts im Testverzeichnis | `3b5530f5-80a9-458d-869b-2af8061511a3` | 3 / 4 |

Verwendet wurden Jenkins `sogis/datenportal-jenkins:0.1.0-3`, natives Sodata
`sogis/datenportal-sodata:0.1.11`, Editor `0.1.4`, Dokumentation `0.1.18` und
die im Stack festgelegten Infrastrukturimages. Die Tests bauten keine lokalen
Jenkins-/Portalimages und verwendeten keinen Jenkins-Quellcheckout. Die lokale
Themenquelle erhielt keine fachlichen Änderungen.

Beide Varianten bestanden:

- Compose-Konfiguration, Build-Aufruf, automatischer Start und `ps -a`.
- Erfolgreicher Seed, eigener beschreibbarer Initialisierungscheckout und
  vorhandener Root-Task `initializePublication` mit S3 an, Git und Reload aus.
- Gültige `current.json`, Datenblattsammlung, DuckDB und öffentlich lesbares RDF.
  Ohne CSV-Lieferung gilt `catalog: null`; das Portal ist dennoch gestartet.
- Gesundheitsprüfungen und interner token-geschützter Portalstatus.
- Zweiter Start mit neuer erfolgreicher Seed-Buildnummer, identischem Manifest
  und identischen Hashes von `.env` und Garage-Konfiguration.
- Fehlenden Verweis im eigenen bereits publizierten Testbucket geprüft:
  Erstaufbau wird verweigert; nach Wiederherstellung bleibt der Release erhalten.
- Eigene Containerprüfung: Host-Client eines mit `flock` gesperrten Testlaufs
  beendet, konkurrierenden Lockzugriff abgewiesen und späteren Abschlussnachweis
  gelesen. Dabei fand keine weitere Publikation statt.
- Bestehender Smoke-Test: S3-Schreiben, Browser-Leserechte und Schreibsperre,
  Websitefreigabe, CORS/Range, Editor, Dokumentation, Portal/DuckDB, Jenkins und
  öffentliche Adminsperren. Temporäre Testobjekte wurden aufgeräumt.

Reproduzierbarer Aufruf aus dem Dev-Stack:

```bash
python3 scripts/test-bootstrap-live.py
```

Die vom Test ausgegebenen temporären Verzeichnisse enthalten `startup.log`
und unter dem jeweiligen Checkout `.runtime/<compose-projekt>/publication.log`
sowie den Bootstrap-Prüfstand. Keine Secretwerte sind Teil dieses Protokolls.

## Automatisierte Fehlerprüfungen mit Fixtures

18 Bootstrap-/Initialisierungstests, sechs Registry-Starttests und fünf
Docs-Starttests bestanden. Die simulierten Fälle umfassen HTTP-/Seed-Fehler,
ungültige Datenblätter vor Manifestumschaltung, beschädigte Manifeste, fehlende
DuckDB, externe oder deaktivierte Publikationsziele, bestehendes Quiet Down,
unbestätigte Aufträge, unklare Teilpublikation und parallele Bootstrap-Aufrufe.
Dies sind keine tatsächlich provozierten Ausfälle der veröffentlichten Dienste.

## Integrator, Portal und Dokumentation

- Integrator: 117 Unit-/Funktionstests sowie 12 echte Integrationstests ohne
  übersprungene Tests; JAR und Spotless-Prüfung erfolgreich. GRETL kam aus dem
  festgelegten Jenkins-Image, mit `GRADLE_JAVA_HOME_17=/no-host-java17`.
- Neue Java-Fixtures prüfen Delegation, lesende Wiederverwendung, separaten
  Stack-Timeout, erhaltene Fehlerlogs und den Stopp vor Bootstrap bei einem
  kalten Start bis zum Runtime-Abgleich. Dies ist keine neue menschliche
  fachliche Lieferfreigabe oder INT-/PROD-Abnahme.
- Portal: `./gradlew clean check` erfolgreich, einschliesslich 152
  Frontendtests, Typprüfung, Java- und Playwright-Prüfungen.
- Lokale Dokumentationsquellen mit Thoth als Working-Trees gebaut und als
  `datenportal-dokumentation:local` paketiert. Containertest für Seiten, Assets,
  Suche, Weiterleitungen, 404, Prefix und Health erfolgreich.

Historische Prüfprotokolle bleiben unverändert. Die vorhandene Änderung in
`../datenportal-jenkins-dev/bin/test-image-duckdb.sh` und die untracked
`../datenportal-sodata/ibx.png` wurden erhalten. Weitere Komponentenquellen und
die fachlichen Inhalte des Themenrepos wurden nicht geändert.
