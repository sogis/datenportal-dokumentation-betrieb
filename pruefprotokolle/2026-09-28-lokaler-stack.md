# Prüfstand: lokaler Docker-Compose-Einstieg, 28.09.2026

Die Dokumentation ist integriert und gerendert. Der Start mit veröffentlichten
Images und die Gateway-Zugänge wurden geprüft. Die vollständige Erstpublikation
bis zur Portalansicht bleibt offen, weil der Docker-Datenträger während des Tests
nur noch rund 226 MiB freien Speicher hatte. Jenkins setzte seinen
Ausführungsknoten bei einem Grenzwert von 1 GiB deshalb offline. Das ist keine
nachgewiesene Unverträglichkeit der Images, aber auch kein erfolgreicher
Nachweis ihrer vollständigen Publikationskette.

## Verwendete Quellen

| Repository | Commit / Arbeitsstand |
|---|---|
| datenportal-dev-stack | `863aa0a09934110489702e3b67ce59a11fd0636a`, unverändert |
| datenportal-jenkins-dev | `58d9930dd94d92953861199c89ca4016d6d51b09`; JCasC unverändert; bereits vorhandene lokale Änderung an `bin/test-image-duckdb.sh` nicht verwendet |
| datenportal-themenrepo | `a354547eafdf5e833eb39edc7de06ccf6dbd361f`, unverändert |
| datenportal-pilot-daten | `67b83675b62d3dc9ce4925f606cefaae46c75ddb`, unverändert; Fixture statisch geprüft, noch nicht angeliefert |
| datenportal-dokumentation-betrieb | Lokaler Arbeitsstand einschliesslich vorher vorhandener Änderungen und neuem Einstiegskapitel |

## Image-Prüfung

Registry-Manifeste mit `docker buildx imagetools inspect` abgefragt. Die beiden
Compose-Defaults `sogis/datenportal-jenkins:0.1.0` und
`sogis/datenportal-sodata:0.1.0` liefern `not found`. Der erste isolierte
`up.sh --registry-only`-Versuch brach entsprechend beim Jenkins-Pull ab; kein
lokaler Build wurde ausgelöst. Für den zweiten Versuch wurden ausschliesslich
in der Test-`.env` die veröffentlichten Nachfolgetags gewählt.

| Image | Registry-Digest | Laufzeitprüfung |
|---|---|---|
| `sogis/datenportal-jenkins:0.1.0-2` | `sha256:59332449e6a08b9f743dd61e396372ba583ad2cb436ec6334c35b01ee9419149` | Gestartet, Login und automatisch ausgeführter Seed erfolgreich |
| `sogis/datenportal-sodata:0.1.5` | `sha256:81a3dd5873f233837ac8bea6608af3edd0262c1b8d94863bca22e8b95faa6d60` | Gepullt; erwartungsgemäss vor Erstpublikation nicht gestartet |
| `sogis/datenportal-datenblatt-editor:0.1.4` | `sha256:8336e402778423fe8a95e53f50268e81586531e6933ec24468263bd51fa99bdb` | Oberfläche und Assets über APISIX |
| `sogis/datenportal-dokumentation:latest` | `sha256:57b1df7678ae1bcb551494887fce8cf6edaaa71a3631ad82756ddcc70ca4c849` | Oberfläche und Assets über APISIX |
| `apache/apisix:3.14.1-debian` | `sha256:c228717165ecf4c0055818c9e2d9843f7b303f17a6df9e5fbaf8eafeb5007ae4` | Healthy, Routing und Sperren geprüft |
| `dxflrs/garage:v2.3.0` | `sha256:866bd13ed2038ba7e7190e840482bc27234c4afaf77be8cfa439ae088c1e4690` | Healthy, frischer Bucket initialisiert |
| `rclone/rclone:1.75.1` | `sha256:45401ad7410db1d67ffdb58e19059ad20b0d8e0285a60e38bbec55cc1019c7a5` | Healthy, öffentlicher Lesepfad erreichbar |
| `amazon/aws-cli:2.36.35` | `sha256:914ea4d4cc484eeae38810210a4b14fb54974273d00ae525646eb1940b61f7c3` | Initialisierungs- und Schlüsselprüfung erfolgreich |

## Testaufbau und Ergebnisse

- macOS mit Docker Desktop, Docker Engine 29.5.3; veröffentlichtes Jenkins-Image
  für amd64 auf dem ARM-Arbeitsplatz verwendet.
- Separate Kopie des Stack-Repositories, eigener Compose-Projektname
  `datenportal-betriebsdoku-test`, eigene Volumes, Testports 18081 und 13900.
  Die bestehende Instanz auf 8081/3900 wurde nicht verändert.
- `./scripts/up.sh --registry-only` mit den Nachfolgetags: Exitcode 0.
  Gateway, Jenkins, Editor, Dokumentation, Garage und S3-Browser healthy;
  fehlendes Manifest mit HTTP 404 erkannt, Sodata nicht gestartet.
- `/gateway-health`: HTTP 200; `/anlieferung`: HTTP 302;
  `/datenblatt-editor` und `/dokumentation`: HTTP 308.
- GET `/admin/catalog/status`, `/actuator/health`, `/apisix/admin` sowie POST
  `/admin/catalog/reload`: HTTP 403.
- Editor und Dokumentation im Browser samt geladenen Assets ohne HTTP-Fehler.
- Jenkins-Anmeldung erfolgreich. Automatischer Seed-Build 1: `SUCCESS`, Jobs
  `gretl-datenportal-agi`, `gretl-datenportal-mfk` und
  `gretl-datenportal-statistikdienst` erzeugt. Ein danach manuell angeforderter
  Seed blieb wegen des durch Speichermangel gesperrten Knotens in der Queue.
- Nebenbefund: Die Startausgabe unter der verwendeten Host-Bash enthielt einen
  fehlerhaften Sodata-Befehl mit mehrfachen `-f`. Die Anleitung enthält deshalb
  den vollständigen, aus der Compose-Konfiguration abgeleiteten Befehl
  `docker compose -f compose.yaml up -d --wait --wait-timeout 120 sodata`.
  Das Stack-Skript wurde im Rahmen dieser Dokumentationsänderung nicht geändert.
- Ausschliesslich die eigene Testinstanz samt ihren Testvolumes wurde danach
  entfernt. Neu für diesen Test bezogene Jenkins-/Sodata-Images wurden nach
  Prüfung auf anderweitige Verwendung wieder entfernt, um Docker-Speicher
  freizugeben. Keine fremden Images oder Volumes wurden bereinigt.

## Dokumentationsprüfung

- Biblios-Build über `scripts/build-local.sh` im Aggregator mit Java 25.0.3,
  vorhandenem Thoth-All-JAR und `--use-local-working-tree` erfolgreich.
- Browserprüfung: Einstieg direkt nach Überblick und vor Voraussetzungen;
  Grafik geladen, keine kaputten internen Anker, keine JavaScript-Fehler.
- SVG visuell geprüft; Bash-Blöcke syntaktisch mit `bash -n` geprüft.
- Keine Compose-, Komponenten- oder zentralen Biblios-Änderungen; keine
  Veröffentlichung vorgenommen.

## Noch abzunehmen

Mit genügend freiem Docker-Speicher denselben isolierten Registry-Ablauf
wiederholen: manueller Seed, Quiet Down, Initialisierung, öffentliche Artefakte,
CSV-Anlieferung für Altersstruktur 2025, Start von Sodata und Prüfung von Suche,
Datensatz, Downloads und «Erkunden». Zusätzlich lokalen XTF-Import/-Export im
Registry-Editor prüfen. Erst danach den entsprechenden Prüfhinweis im Kapitel
aktualisieren. Bestehende Publikationen nicht neu initialisieren.
