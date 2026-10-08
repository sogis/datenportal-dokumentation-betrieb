# Prüfnachweis: Vom CRC-Test zum Clusterbetrieb

Stand: 30.09.2026. Geprüft wurden die überarbeiteten Kapitel 3 und 6 sowie
die Haupt-, CRC- und Betreiber-READMEs des Stack-Repositories. Images,
Manifeste, Skripte, Konfigurationsschlüssel und Anwendungs-APIs wurden nicht
geändert. Frühere Prüfprotokolle bleiben historische Nachweise.

## Erfolgreiche lokale Prüfungen

- Asciidoctor-HTML-Build mit `--failure-level WARN`: erfolgreich, keine
  Dokumentwarnungen. Alle 105 internen Verweise zeigen auf vorhandene,
  eindeutig vergebene IDs.
- Biblios-Build mit dem vorhandenen Thoth-All-JAR und dem aktuellen Working
  Tree: erfolgreich. Temporäre Konfiguration mit ausschliesslich dieser
  Dokumentationsquelle; kein Deployment und kein veröffentlichter Image-Build.
  Die JVM/JRuby-Hinweise auf veraltete bzw. eingeschränkte Java-Methoden sind
  Laufzeitwarnungen des Renderers, keine Dokumentfehler.
- Biblios im Browser visuell geprüft: Kapitel 3.1 bis 3.8, Unterabschnitte
  3.5.1/3.5.2 und 3.6.1, Sprungziele für Erstinitialisierung und Portalprüfung,
  Übergabetabelle und die Kategorien „Hinweis“/„Wichtig“.
- 19 Bash-Blöcke aus Kapitel 3 und den beiden Detail-READMEs mit `bash -n`
  geprüft. CA-Export und Architekturprüfung sind in Kapitel und jeweiliger
  README identisch.
- Bestehende Stack-Tests: 18 erfolgreich. Enthalten sind gerenderte
  Overlay-Verträge, Laufzeitmodus/JCasC, Credential-Verknüpfungen,
  Reload-Token und Bootstrap-Verhalten.
- `operator` und `operator-bootstrap` zusätzlich mit `oc kustomize` in einer
  temporären Kopie ohne lokale Konfiguration oder Secrets gerendert:
  jeweils 14 Ressourcen, konsistenter Zielnamespace, Sodata mit einer bzw.
  null Replikaten.
- `git diff --check` in Dokumentations- und Stack-Repository erfolgreich.

## Nachvollzogene Abläufe

Die folgenden Wege wurden anhand der Anleitung und Stack-Verträge geprüft;
dies war keine erneute Publikation oder Clusterabnahme.

| Ausgangspunkt | Vorgesehener Ablauf |
|---|---|
| CRC, neuer Testbestand | Öffentliche Manifest-404 und Lesefreigabe klären → CRC-Bootstrap → Zugänge und Seed → einmalige Initialisierung → erste Testlieferung → Portalstart und Prüfung. |
| CRC, Wiederanlauf | Konfiguration, Secrets und Bestand erhalten → CRC-Anmeldung/CA → normales Deployment bei gültigem Manifest → Seed-Prüfung → bekannte Daten prüfen; Initialisierung und Upload überspringen. |
| Zielcluster, neuer Bestand | Kapitel 4 → eigene Umgebung/Secrets/PVC → `production` mit Betreiber-JCasC/AD → Git-Zugang wählen → Kontext/Namespace/Architekturen → Operator-Bootstrap nur bei öffentlicher Manifest-404 → AD/Seed → einmalige Initialisierung → normales Operator-Overlay → Gesamtabnahme. |
| Zielcluster, vorhandener Bestand | Gleiche Umgebungsvorbereitung → gültiges Manifest und Artefakte prüfen, konkurrierende Publisher ausschliessen → direkt Operator-Overlay → AD/Seed und bekannte Daten prüfen; kein Bootstrap und keine Erstinitialisierung. |

HTTP 403, beschädigte Manifeste und Netzwerkfehler sind in keinem Ablauf ein
Nachweis für einen neuen Bestand. Eine Migration von CRC-PVC oder Testdaten
ist kein Teil dieser Anleitung. Weitere Testuploads sind ausdrücklich
optionale Schreibvorgänge.

## Live-Prüfung und offene Punkte

Die lokale Kubeconfig nennt `https://api.crc.testing:6443`; Kontext und
Namespace konnten gelesen werden. Das bestätigt keine aktive API-Verbindung.
Der Nodeabruf und der dokumentierte CA-Export scheiterten an einer abgelehnten
Verbindung zur CRC-API. Der HTTPS-Aufruf von `/gateway-health` mit dem bereits
vorhandenen CA-Zertifikat scheiterte ebenfalls an der Verbindung (curl Exit 7).

Der CA-Export brach korrekt ab und erhielt die vorhandene lokale Zertifikatsdatei
unverändert; temporäre Dateien wurden entfernt. Erfolgreicher Zertifikatsabruf,
Live-Nodearchitekturen und HTTPS-Verifikation müssen bei erreichbarem CRC noch
bestätigt werden. Ein Start oder eine Reparatur von CRC wurde nicht ausgeführt.

AD-Anmeldung, Rechte normaler Fachbenutzer und produktive Plattformabnahme
bleiben offen. Es wurden keine Deployments angewendet, keine Publikationen oder
Reloads ausgelöst und keine Bucket-Rechte geändert.
