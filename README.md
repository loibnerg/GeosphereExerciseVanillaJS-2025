## Projekt: „Klimadaten-Explorer für Geosphere Austria (klima-v2-1h)“

**Ziel**
Die Studierenden entwickeln eine kleine Web-Applikation (HTML + CSS + Vanilla-JS), die stündliche Klimadaten (tl, ff, rr, so\_h) der Stationen **16413 (Graz Straßgang)** und/oder **54 (Leibnitz-Wagna)** aus der Geosphere-Austria-API abruft, tabellarisch darstellt – und optional auch grafisch visualisiert.
Gearbeitet wird strikt nach dem **GitHub-Flow**: jede User Story ➔ eigener Branch, Pull Request, Code-Review, Merge in *main*.

---

### API-Steckbrief

| Aspekt         | Wert                                                                                    |
| -------------- | --------------------------------------------------------------------------------------- |
| Basis-Endpoint | `https://dataset.api.hub.geosphere.at/v1/station/historical/klima-v2-1h`                |
| Stationen      | `16413,54`                                                                           |
| Parameter      | `tl` (Temp 2 m, °C), `ff` (Wind m/s), `rr` (Niederschlag mm), `so_h` (Sonnenschein h)   |
| Zeitformat     | ISO 8601, z. B. `2024-01-01T00:00:00Z`                                                  |
| Beispiel-Query | https://dataset.api.hub.geosphere.at/v1/station/historical/klima-v2-1h?station_ids=16413&parameters=tl&start=2025-01-01T00:00:00Z&end=2025-01-01T23:00:00Z |

Hier das Frontend von Geosphere zum Probieren: https://dataset.api.hub.geosphere.at/app/frontend/station/historical/klima-v2-1h

---

## Backlog – 5 User Stories (+ 1 Bonus)

> **Hinweis**
> • Alle Stories sind *vertical slices*: Backend-Fetch → UI.
> • Acceptance Criteria (AC) sind zu testen und die Tests zu dokumentieren (als Textdatei).
> • Naming-Konvention für Branches: `US<n>-<stichwort>`.

---

### **US 1 – „Erste Daten sichtbar machen“**

> *Als Nutzer* möchte ich für **Station 54** den Parameter **tl** im festen Zeitraum **01.01.2024 – 01.01.2025** sehen, damit ich einen ersten Temperatur-Überblick erhalte.

**AC**

1. Statischer HTML-Button „Daten laden“ oder automatische Initialisierung.
2. Fetch → JSON → Tabelle (Datum | tl).
3. Werte in °C, auf 1 Dezimalstelle gerundet.
4. Simple Fehlerbehandlung (Console Log reicht).

---

### **US 2 – „Stationswahl ermöglichen“**

> *Als Nutzer* möchte ich zwischen den Stationen **Graz Straßgang (16413)** und **Leibnitz-Wagna (54)** umschalten können, um Messungen vergleichen zu können.

**AC**

1. Dropdown mit den beiden Stationsnamen.
2. Auswahl triggert neuen Fetch; Tabelle aktualisiert sich.
3. Datum & tl bleiben wie in US 1.

---

### **US 3 – „Zeitraum frei wählen“**

> *Als Nutzer* möchte ich **Start-** und **Enddatum** über zwei Date-Picker eingeben, um beliebige Zeitbereiche auszuwerten.

**AC**

1. Zwei `<input type="date">` Felder (min/max = API-Grenzen).
2. Validierung: Start ≤ Ende, Zeitspanne ≤ 10 Tage (Performance-Schutz).
3. Tabelle zeigt tl für den gewählten Range.
4. Stationswahl aus US 2 bleibt funktionsfähig.

---

### **US 4 – „Parameter auswählen“**

> *Als Nutzer* möchte ich über **Checkboxen** bestimmen, welche Messgrößen (tl, ff, rr, so\_h) abgefragt werden, um nur relevante Daten zu sehen.

**AC**

1. Vier Checkboxen, tl vorselektiert.
2. Tabelle zeigt jeweils eine Spalte pro aktivem Parameter, inklusive Einheiten.
3. Neustart des Fetch, wenn Auswahl geändert wird.
4. Leere Werte sauber kennzeichnen (z. B. „–“).

---

### **US 5 – „Tabelle nutzbar machen“**

> *Als Nutzer* möchte ich die Tabelle **sortieren** (auf Header-Klick).

**AC**

1. Sortieren auf jede Spaltenüberschrift (asc/desc-Toggle).
2. Responsives Layout ⇒ auf Smartphone horizontal scrollbar.

---

### **BONUS-US 6 – „Daten grafisch erleben“**

> *Als User* möchte ich eine **interaktive Grafik** der ausgewählten Parameter sehen, um Trends schneller zu erkennen.

**Empfohlene Library:** **Chart.js v4** (leichtgewichtig, vanilla-freundlich).

**AC**

1. Button „Diagramm anzeigen“ öffnet Canvas mit Linien- oder Balkendiagramm.
2. Mehrere Parameter ⇒ unterschiedliche Linien (Legende, Tooltip).
3. Aktualisiert sich bei Filter-/Datums-Änderung (Re-Render).

---


### Quick-Start-Snippets (optional für README)

```html
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <title>Dynamischer Geosphere API Fetch</title>
</head>
<body>
  <button id="fetchData">Daten abrufen</button>

  <script>
    document.getElementById('fetchData').addEventListener('click', async () => {
      // === Dynamische Parameter-Definition ===
      const baseUrl = 'https://dataset.api.hub.geosphere.at/v1/station/historical/klima-v2-1h';  // Stündliche Klimadaten

      const parameters = ['tl', 'ff', 'rr', 'so_h'];             // Parameter: Temperatur, Wind, Niederschlag, Sonne
      const stationIds = ['16413', '54'];                        // Zwei Beispielstationen
      const startDate = '2025-01-01T00:00';                      // ISO Datumsformat
      const endDate   = '2025-01-01T23:00';
      const outputFormat = 'geojson';                            // muss geojson sein!

      // === URL dynamisch zusammenbauen ===
      let url = `${baseUrl}?`;

      // Parameter mehrfach anhängen
      parameters.forEach(param => {
        url += `parameters=${param}&`;
      });

      stationIds.forEach(id => {
        url += `station_ids=${id}&`;
      });

      // Weitere feste Parameter anhängen
      url += `start=${encodeURIComponent(startDate)}&`;  // Sonderzeichen umwandeln
      url += `end=${encodeURIComponent(endDate)}&`;
      url += `output_format=${outputFormat}`;

      console.log('Request URL:', url);

      // === Daten abrufen ===
      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error(`HTTP-Fehler: ${response.status}`);
        const data = await response.json();

        console.log('GeoJSON-Daten:', data);

        // Beispiel: Daten extrahieren
        //
        // data.features enthält ein Objekt pro Station
        data.features.forEach(feature => {
          const p = feature.properties.parameters;

          // p ist kurz für parameters; p.tl.data ist das Array für Temperaturwerte; p.tl.unit ist die zugehörige Einheit
          // feature.properties.station ist die Stations Nummer
          // data.timestamps ist das Array mit allen Zeitstempeln

          console.log(`Zeit: ${data.timestamps[0]}, Station: ${feature.properties.station}, Temp: ${p.tl.data[0]}${p.tl.unit}, Wind: ${p.ff.data[0]}${p.ff.unit}, Regen: ${p.rr.data[0]}${p.rr.unit}, Sonne: ${p.so_h.data[0]}${p.so_h.unit}`);
        });

      } catch (error) {
        console.error('Fehler beim Abrufen:', error);
      }
    });
  </script>
</body>
</html>

```

---

Viel Erfolg beim Coden!


