# Bitcoin Grafana Dashboards

Dieses Repository enthält zwei Grafana-Cloud-Dashboards zur Überwachung einer
Bitcoin-Core-Node. Die Dashboards werden als JSON-Ressourcen im aktuellen
Grafana-App-Platform-Format (`dashboard.grafana.app/v2`) versioniert und können
direkt per Grafana Git Sync aus GitHub bereitgestellt werden.

## Enthaltene Dashboards

| Dashboard | Datenquelle | Geeignet für |
| --- | --- | --- |
| [Bitcoin Node](dashboards/dashboard.json) | Prometheus | Zustand, Synchronisation, Peers, Mempool, Ressourcen und Netzwerkmetriken |
| [Bitcoin Node – Logs](dashboards/logs.json) | Loki | Live-Logs, Fehler/Warnungen, neue Blöcke und Neustarts von Bitcoin Core |

Eine ausführliche Beschreibung und Entscheidungshilfe befindet sich in der
[README des Dashboard-Ordners](dashboards/README.md).

## Voraussetzungen

- eine Grafana-Cloud-Instanz mit Admin-Rechten
- dieses GitHub-Repository und ein Branch, den Grafana lesen und – bei
  bidirektionalem Sync – beschreiben darf
- Bitcoin-Core-Metriken in Prometheus mit den im Dashboard verwendeten
  `bitcoin_*`-Metriknamen
- Prozessmetriken des Exporters (`process_*`) und für die Anzeige des freien
  Speicherplatzes `node_filesystem_avail_bytes`
- Bitcoin-Core-Logs in Loki mit dem Label `app="bitcoin-core"`
- Journald-Logs für die Neustart-Panels mit dem Label
  `job="loki.source.journal.bitcoind_journal"`

Die Datenquellen selbst werden nicht durch dieses Repository angelegt. Sie
müssen in Grafana Cloud bereits vorhanden und mit Daten befüllt sein.

## Git Sync einrichten

1. Die Änderungen dieses Repositories committen und nach GitHub pushen.
2. In Grafana Cloud **Administration → General → Provisioning** öffnen.
3. **GitHub** als Provider auswählen und den Zugriff per GitHub App oder Token
   konfigurieren.
4. Repository-URL, gewünschten Branch und `dashboards` als Sync-Pfad angeben.
5. Synchronisierung starten und anschließend unter **Dashboards** prüfen, ob
   `Bitcoin Node` und `Bitcoin Node – Logs` vorhanden sind.

Mit `dashboards` als Sync-Pfad liest Grafana nur die Dashboard-Ressourcen aus
diesem Ordner. Die Markdown-Datei wird nicht als Dashboard importiert. Git Sync
arbeitet bidirektional: Abhängig vom gewählten Workflow schreibt Grafana
Änderungen direkt zurück oder erstellt dafür einen Branch beziehungsweise Pull
Request.

## Muss vor dem Sync etwas angepasst werden?

Für den Sync selbst nicht. Beide Dateien besitzen bereits die erforderliche
CRD-Struktur mit `apiVersion`, `kind`, `metadata` und `spec`. Vor dem ersten
Sync sollten jedoch die folgenden instanzspezifischen Werte kontrolliert
werden:

- [dashboard.json](dashboards/dashboard.json) referenziert die
  Prometheus-Datenquelle `grafanacloud-prom` direkt. Existiert sie unter einem
  anderen Namen, muss der Name in den Panel-Abfragen und den Variablen `job`
  und `instance` ersetzt werden.
- Das Panel **Freier Speicherplatz** filtert auf die feste Node-Exporter-Instanz
  `yellow-duck-44516`. Dieser Wert muss zur `instance`-Bezeichnung der eigenen
  `node_filesystem_avail_bytes`-Zeitreihe passen.
- [logs.json](dashboards/logs.json) verwendet eine auswählbare Loki-Datenquelle.
  Nach dem Import sollte in der versteckten Variable `DS_LOKI` kontrolliert
  werden, dass die richtige Loki-Datenquelle ausgewählt ist.
- Falls Alloy andere Labels für Logs oder Journald vergibt, müssen die oben
  genannten Loki-Selektoren entsprechend geändert werden.

Beim Sync in dieselbe Grafana-Cloud-Instanz, aus der die Dateien exportiert
wurden, sind diese Werte in der Regel bereits korrekt.

## Repository-Struktur

```text
.
├── dashboards/
│   ├── dashboard.json
│   ├── logs.json
│   └── README.md
├── LICENSE
└── README.md
```

## Weiterführende Dokumentation

- [Grafana: Git Sync einrichten](https://grafana.com/docs/grafana/latest/as-code/observability-as-code/git-sync/git-sync-setup/)
- [Grafana: Bestehende Ressourcen zu Git Sync hinzufügen](https://grafana.com/docs/grafana/latest/as-code/observability-as-code/git-sync/export-resources/)
- [Grafana: Funktionsweise von Git Sync](https://grafana.com/docs/grafana/latest/as-code/observability-as-code/git-sync/key-concepts/)

## Lizenz

Dieses Projekt steht unter der [MIT-Lizenz](LICENSE).
