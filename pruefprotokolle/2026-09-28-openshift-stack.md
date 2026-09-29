# Prüfstand: CRC-/OpenShift-Einstieg, 28.09.2026

Das neue Kapitel `openshift-kennenlernen.adoc` folgt dem Compose-Einstieg.
Es erklärt CRC ohne AIO-/AD-Anbindung, den externen Testbucket und den normalen
Betrieb mit Registry-Images. Die wiederverwendbare Quelle liegt in
`datenportal-stack/deploy`; dieses Buch enthält keine Manifestkopie.

## Geprüft

- Arbeitsstand des Stack-Repositories und der Betriebsdoku abgeglichen.
- CRC-/Betreiber-Manifeste, beide Bootstrap-Overlays und die optionale lokale
  Image-Variante mit Kustomize gerendert; 15 Python-Tests erfolgreich.
- APISIX unter beliebiger UID: Weiterleitungen mit Queryparametern,
  Prefix-Rewrites, öffentliche GET-/POST-Sperren und tatsächliche HTTPS/443-
  Header zum Test-Upstream, auch bei gefälschten Forwarded-Headern.
- Veröffentlichter Editor: korrekter HTML-Basispfad und zwei lokale Assets.
  Dokumentationsimage: HTML und 42 lokale Assets mit Prefix; beide unter
  beliebiger UID, ohne Capabilities und mit `no-new-privileges`.
- Registry-Digests aller fünf Images erreichbar. Jenkins `0.1.0-2` und Sodata
  `0.1.5` nur amd64; die anderen drei Images auch ARM64.
- Produktions-JCasC und Entrypoint aus dem veröffentlichten Jenkins-Image
  gelesen. Diese passen zur Betriebsdoku; das lokale ältere Jenkins-Checkout
  verlangt noch Gruppenvariablen und wurde nicht übernommen oder geändert.
- Buch mit Asciidoctor 2.0.26 als HTML ohne Warnungen gerendert. Interne Anker
  und neue Shellblöcke geprüft; Kapitel und SVG im Browser angesehen.
- `git diff --check` erfolgreich; keine Secrets aufgenommen.

Die vollständigen Digests und detaillierten Testergebnisse stehen im
Stack-Repository unter `docs/validation.md` und `deploy/base/kustomization.yaml`.
Die Tests publizieren keine Daten und greifen nicht auf produktives AD zu.

## Grenzen

CRC war gestoppt. Es wurden keine Clusterressourcen ausgerollt. Vor einem
Registry-Start auf dem lokalen ARM64-CRC müssen passende Jenkins-/Sodata-Images
extern veröffentlicht werden. Externe S3-Testwerte und AIO-/AD-Zugänge fehlen.
Die vollständige Kette mit Seed, Erstpublikation, Lieferung, Reload,
CORS/Range, PVC-Wiederanlauf und produktiven Uploadgrenzen bleibt abzunehmen.

Nach Aktualisierung von Thoth auf `fd57999` wurde das JAR erfolgreich gebaut.
Der Biblios-Aggregator mit `--use-local-working-tree` hat anschliessend alle sechs
Komponenten einschliesslich des neuen lokalen Betriebskapitels gerendert.
Der ursprüngliche CLI-Blocker ist behoben. Ein HTTP-504 des Codeberg-Metadaten-
Endpoints wurde beim Build durch die direkte offizielle Tarball-URL derselben
interlis-lab-Version 0.1.10 umgangen. Keine Aggregator-/Thoth-Quelldateien geändert.
Das fixierte Dokumentationsimage wurde nicht neu veröffentlicht und zeigt
weiterhin seinen bisherigen Release-Stand.
