# Registry-Stack: Prüfprotokoll vom 30.09.2026

## Aufbau und Eingabestände

Geprüft wurde der lokale Einstieg der Betriebsdoku mit genau einer temporären
Kopie des Dev-Stacks, ohne Schwester-Repositories und ohne lokale Image-Builds.
Der bestehende Entwicklungsstack blieb unverändert in Betrieb.

- Dev-Stack: Working Tree auf Basis `863aa0a09934110489702e3b67ce59a11fd0636a` mit Registry-Override, Test-JCasC und aktualisiertem Startskript.
- Betriebsdoku: Working Tree auf Basis `10d4abd78ce102075e6a1ce53e38e2891afd17df` mit der überarbeiteten lokalen Anleitung.
- Themenrepo: Jenkins klonte `https://github.com/sogis/datenportal-themenrepo.git`, Branch `main`; geprüfter Commit `a0e92c99b2b39e1401106614f0c3c6a692ee64c7`.
- Docker Desktop, Linux/ARM64-Container, Docker Compose `5.1.4`.
- Eigener Compose-Projektname `datenportal-registry-abnahme`; Testports `18091` (Gateway) und `13910` (S3), um den bestehenden Stack auf `8081`/`3900` nicht zu stören.
- Neue Garage-Secrets und neue Volumes ausschliesslich für diesen Test. Keine produktiven Daten oder Zugangsdaten.
- CSV per Codeberg-Raw-Download aus `datenportal-pilot-daten`; Datenblatt per GitHub-Raw-Download aus dem Themenrepo, wie in der Anleitung.

Die Browserprüfung verwendete vorhandenes Playwright als Prüfwerkzeug auf dem
Host. Die Stack-Laufzeit und ihre Mounts benötigten keines der lokalen
Komponenten-Repositories.

## Verwendete Images

| Image | Lokal aufgelöster Registry-Digest |
| --- | --- |
| `sogis/datenportal-jenkins:0.1.0-3` | `sha256:ecb915bf07fc87b47c8abf37d5e18a89be0c49f79be59a38c1ea5a47c7e75b56` |
| `sogis/datenportal-sodata:0.1.10` | `sha256:316c8fe376c7f66f65157307fd976b1630b0e5c7a0659576ac5956183833c428` |
| `sogis/datenportal-dokumentation:0.1.18` | `sha256:21bfc655e9d51877ba56402f69dc8fcfd21869ed252eb0c0ad816593b6ee6f50` |

Sodata startete das native Binary `/opt/datenportal/app`. Das JVM-Image wurde
nicht verwendet. Weitere Images: Editor `0.1.4`, Garage `v2.3.0`, APISIX
`3.14.1-debian`, rclone `1.75.1`, AWS CLI `2.36.35`.

## Erfolgreiche Prüfungen

1. `python3 scripts/test-registry-start.py`: fünf Tests erfolgreich, einschliesslich Unterfällen für alle vier lokalen Buildoptionen, ältere/unbekannte Compose-Versionen und fehlende Jenkins-/Sodata-/Dokumentationsimages. Die echte Compose-Auflösung bestätigt versionierte Images, keine Builddefinitionen und keine Bind-Mounts ausserhalb des temporären Stackverzeichnisses.
2. `python3 scripts/test-docs-start.py`: fünf bestehende Tests erfolgreich. Lokaler Docs-Build, Image-Priorität und Fehlerbehandlung bleiben abgedeckt.
3. `bash -n scripts/up.sh`, `docker compose -f compose.yaml -f compose.registry.yaml config --quiet` und `git diff --check`: erfolgreich. `docker compose ... build` meldete erwartungsgemäss «No services to build».
4. `./scripts/up.sh --registry-only`: erfolgreich. Ohne Manifest bleibt Sodata gestoppt; der ausgegebene spätere Startbefehl enthält beide Compose-Dateien mit korrekten `-f`-Argumenten.
5. Gateway: Health HTTP 200, Anlieferung HTTP 302, Editor- und Dokumentations-Slash-Weiterleitungen HTTP 308. Öffentliche Admin-/Actuator-Pfade liefern HTTP 403, auch POST auf den Reload-Endpunkt. Editor, Dokumentation und ihre lokalen Script-/Stylesheet-Assets sind erreichbar.
6. Jenkins-Anmeldung mit dem lokalen Testkonto und automatisch ausgeführter Seed erfolgreich. Organisationsjobs wurden aus dem GitHub-Themenrepo erzeugt; kein lokaler Themenrepo-Mount und kein Git-Rückschreiben.
7. Erstinitialisierung bei nachgewiesenem HTTP 404 und aktiviertem Quiet Down: eigene beschreibbare Kopie des Seeder-Checkouts im Jenkins-Volume, `initializePublication` erfolgreich. Manifest enthält Datenblätter und DuckDB; `catalog: null` ist in diesem initialen Stand zulässig.
8. Nach Aufheben von Quiet Down: CSV im Browser über das APISIX/Jenkins-Formular für `statistikdienst` / `ch.so.bevoelkerung.altersstruktur`, Ausgabe `2025`, angeliefert. Metadatenupload leer, Reload ausgeschaltet. Job 1 erfolgreich; `report.json` bestätigt `publication: accepted` und `opendata.status: accepted`. Das Manifest enthält eine neue Release-ID und einen PublishedCatalog.
9. Native Sodata mit beiden Compose-Dateien gestartet: interner Healthcheck `UP`. Im Browser nach «Altersstruktur» gesucht, Ausgabe 2025 aufgeklappt und Detailseite geöffnet; CSV-Download erfolgreich. XLSX- und Parquet-Links werden angezeigt; deren Dateiinhalte wurden hier nicht separat geprüft.
10. `docker compose ... down` ohne Volume-Löschung, danach erneut `./scripts/up.sh --registry-only`: erfolgreich, Sodata startet automatisch. Manifest, Jenkins-Buildhistorie, `.env` und Garage-Konfiguration bleiben identisch; Suche und CSV-Download weiterhin erreichbar. Keine erneute Initialisierung.
11. Betriebsdoku mit Asciidoctor.js gerendert, ohne Warnungen; Abschnitt «Testumgebung vorbereiten» im Browser visuell geprüft.

## Bekannte Einschränkung des nativen Sodata-Images

«Daten erkunden» liefert mit `sogis/datenportal-sodata:0.1.10` reproduzierbar
HTTP 500, auch nach dem Neustart. Betroffene Testadresse:
`/series/ch.so.bevoelkerung.altersstruktur/issues/current/explore`.

Das Sodata-Log meldet `com.oracle.svm.core.jdk.UnsupportedFeatureError`:
`Record components not available for record class ch.so.agi.datenportal.explore.ExploreContextDto`.
Die Serialisierung in `ExploreContextService.toEmbeddableJson` benötigt
Reflection-Metadaten, die im nativen Image fehlen. Eine Änderung der Compose-
Konfiguration behebt diese Build-Einschränkung nicht. Für die Abnahme von
«Erkunden» ist ein korrigiertes, veröffentlichtes natives Sodata-Image nötig.
Kein Wechsel auf JVM und kein lokaler Build als Ersatz.

Die erfolgreiche Abnahme umfasst damit Registry-Start, Publikation, Suche,
Detailseite, CSV-Download und Wiederanlauf; sie ist keine vollständige Abnahme
der Erkunden-Funktion. AMD64 und produktives AD/OpenShift wurden in diesem
Durchlauf nicht geprüft. Historische Prüfprotokolle wurden nicht verändert.

Nach der Abnahme wurden ausschliesslich die Container, das Netzwerk und die
neu angelegten Volumes des Testprojekts `datenportal-registry-abnahme` entfernt.
Der bestehende Dev-Stack und seine Volumes blieben erhalten; alle sieben
laufenden Dienste waren danach weiterhin gesund.
