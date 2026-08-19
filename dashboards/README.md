# Dashboards

Die beiden Dashboards ergänzen sich: Das Metrik-Dashboard zeigt den messbaren
Zustand der Node und Entwicklungen über die Zeit; das Log-Dashboard liefert den
Textkontext für Ereignisse und Fehler. Für den laufenden Betrieb ist das
Metrik-Dashboard der erste Einstieg. Bei Auffälligkeiten hilft anschließend das
Log-Dashboard bei der Ursachenanalyse.

## Welches Dashboard brauche ich?

| Fragestellung | Dashboard |
| --- | --- |
| Ist die Node aktuell erreichbar, synchron und mit Peers verbunden? | **Bitcoin Node** |
| Wie entwickeln sich Blockhöhe, Mempool, Hashrate oder Gebühren? | **Bitcoin Node** |
| Werden CPU, RAM, Dateideskriptoren oder Datenträger knapp? | **Bitcoin Node** |
| Welche Meldung hat Bitcoin Core gerade geschrieben? | **Bitcoin Node – Logs** |
| Gab es Fehler, Warnungen, fehlgeschlagene Aktionen oder Neustarts? | **Bitcoin Node – Logs** |
| Warum hat sich eine Metrik zu einem bestimmten Zeitpunkt verändert? | Erst **Bitcoin Node**, dann für denselben Zeitraum **Bitcoin Node – Logs** |

## Bitcoin Node (`dashboard.json`)

Dieses Prometheus-Dashboard ist die Betriebsübersicht der Bitcoin-Node. Über
die Variablen **Job** und **Instanz** lässt sich die betrachtete Exporter-Serie
auswählen. Standardmäßig zeigt es die letzten 24 Stunden.

### Funktionsbereiche

- **Übersicht:** aktuelle Blockhöhe, Betriebszeit, Peer-Anzahl, RPC-Status und
  Warnungen
- **Konnektivität und Mempool:** ein- und ausgehende Verbindungen, Peer-Verlauf,
  Transaktionsanzahl, Mempool-Größe und Speichernutzung des Mempools
- **Kettenzustand:** Verlauf der Blockhöhe, Difficulty und
  Verifikationsfortschritt
- **Mining und letzter Block:** geschätzte Netzwerk-Hashrate sowie Höhe, Größe,
  Gewicht, Transaktionsanzahl, Wert und Gebühren des zuletzt bekannten Blocks
- **Gebühren:** Smart-Fee-Schätzungen für Bestätigungshorizonte von 2, 3, 5 und
  20 Blöcken
- **Datenträger und Netzwerk:** belegter Blockchain-Speicher, freier
  Dateisystemspeicher sowie gesendete und empfangene Bytes pro Sekunde
- **Prozess und Exporter:** RAM, RSS, virtueller Speicher, CPU,
  Dateideskriptoren, Prozessstart und Exporter-Laufzeitmetriken
- **Protokoll und Netzwerkzustand:** Server-/Protokollversion, gebannte Peers,
  Chain Tips, nicht ausgestrahlte Transaktionen und Mempool-Mindestgebühr

### Wann dieses Dashboard sinnvoll ist

Es eignet sich für den täglichen Health-Check, die Beobachtung einer initialen
Blockchain-Synchronisation, Kapazitätsplanung und die Erkennung schleichender
Probleme. Es beantwortet vor allem **was** sich verändert hat. Für das **warum**
sollte der betroffene Zeitraum anschließend im Log-Dashboard untersucht werden.

### Erwartete Daten

Das Dashboard fragt die Prometheus-Datenquelle `grafanacloud-prom` ab und
erwartet Metriken eines Bitcoin-Core-Exporters (`bitcoin_*`). Zusätzlich werden
Standard-Prozessmetriken (`process_*`) und eine Node-Exporter-Metrik
(`node_filesystem_avail_bytes`) verwendet.

Fast alle Bitcoin-Abfragen werden über `$job` und `$instance` gefiltert. Das
Panel **Freier Speicherplatz** ist davon unabhängig und enthält aktuell den
festen Instanzfilter `yellow-duck-44516`. Bei einem anderen Node-Exporter-Label
muss genau dieser Filter angepasst werden.

## Bitcoin Node – Logs (`logs.json`)

Dieses Loki-Dashboard konzentriert sich auf Ereignisse aus Bitcoin Core. Die
Loki-Datenquelle wird über die Variable `DS_LOKI` ausgewählt. Auch dieses
Dashboard startet mit einem Zeitraum von 24 Stunden.

### Funktionsbereiche

- **Bitcoin-Core debug.log (live):** vollständiger Log-Stream für
  `app="bitcoin-core"`
- **ERROR / WARN / FAILED:** Raten häufig relevanter Fehlerschlüsselwörter
- **UpdateTip – Events:** Rate der Meldungen zu neuen Chain-Tips und neuen
  besten Blöcken
- **Restarts:** Anzahl erkannter systemd-/Journald-Neustartereignisse in den
  letzten 24 Stunden
- **Restarts – Zeitverlauf:** stündliche Verteilung dieser Ereignisse

### Wann dieses Dashboard sinnvoll ist

Das Dashboard ist für Fehlersuche, Neustartkontrolle und die zeitliche
Einordnung auffälliger Metriken gedacht. Es ist außerdem nützlich, um zu prüfen,
ob die Node weiterhin neue Blöcke verarbeitet, wenn reine Kennzahlen nicht
genug Kontext liefern.

### Erwartete Daten

Für die Bitcoin-Core-Panels müssen Logs mit `app="bitcoin-core"` in Loki
vorhanden sein. Die beiden Restart-Panels erwarten Journald-Einträge unter
`job="loki.source.journal.bitcoind_journal"`. Andere Alloy-Komponenten- oder
Relabeling-Namen erfordern eine Anpassung dieser Selektoren.

## Empfohlener Ablauf bei Störungen

1. Im Dashboard **Bitcoin Node** den Zeitpunkt und die betroffene Metrik
   bestimmen.
2. Im Dashboard **Bitcoin Node – Logs** denselben Zeitraum einstellen.
3. Zuerst die Fehler-/Warnraten prüfen und anschließend im Live-Log nach den
   zugehörigen Meldungen suchen.
4. Bei Datenlücken zusätzlich kontrollieren, ob Prometheus und Loki noch Daten
   empfangen und ob in Grafana die richtigen Datenquellen ausgewählt sind.
