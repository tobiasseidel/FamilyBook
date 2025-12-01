# 📘 FalilyBook Historische Fotoanalyse & Personen-/Ortszuordnung

## 🧭 Projektvision

Dieses Projekt soll eine lokal ausgeführte Anwendung bereitstellen, die historische Familienfotos automatisch analysiert, Gesichter erkennt, Personen über Jahrzehnte hinweg zuordnet, Orte vorschlägt und am Ende hochwertig annotierte Ausgabebilder erzeugt.  
Es dient der Bewahrung von Familiengeschichte und der Strukturierung eines über Jahrzehnte gewachsenen Fotoarchivs – vollständig offline, vollständig privat.

---

## 🎯 Ziele

- Automatische Analyse einer großen Fotosammlung aus einem Netzwerkordner  
- Altersrobuste Personen­erkennung mit modernen KI-Embedding-Modellen  
- Verwaltung von Personen, Gesichtern, Clustern und Zuordnungen  
- Optionale Erkennung von Gebäuden/Umgebungen zur Geolokalisierung  
- Manuelle Bestätigung und Bearbeitung unsicherer Vorschläge  
- Komfortables Web-Frontend für Review, Zuordnung und Bearbeitung  
- Persistente Speicherung aller Metadaten, Crops und Ergebnisse (SQLite)  
- Automatischer Export annotierter Bilder mit Rahmen, Namen, Datum und Ort  

---

## 🧱 Systemarchitektur

### Backend
- **Python**, **Flask** für API + Webfrontend  
- **SQLite** zur Metadatenspeicherung  
- **InsightFace / ArcFace** für Face Detection & Embeddings  
- **Eigenes Job-System** für Hintergrundverarbeitung (lange Laufzeiten möglich)  
- Module:  
  - Scanner (Dateien erkennen)  
  - Analyzer (Gesichter, Gebäude, Embeddings)  
  - Clusterer (unsupervised)  
  - Matcher (Personenvorschläge)  
  - Preview-Generator  
  - Exporter (annotierte Bilder)  

---

### Frontend
- Flask-Templates (Bootstrap oder Tailwind)  
- Bildgalerie  
- Foto-Detailansicht mit Gesichtscrops  
- Personenverwaltung  
- Clusterverwaltung  
- Geolokalisierungs-Unterstützung (Gebäudevorschläge)  
- Exportbereich  

---

### Datenbankmodell (SQLite)

#### photos
- id  
- filepath  
- folder_year  
- folder_label  
- hash  
- import_date  

#### faces
- id  
- photo_id  
- rel_bbox    
- embedding  
- preview_path  
- person_id (nullable)  
- cluster_id (nullable)  

#### persons
- id  
- name  
- primary_embedding  
- year_of_birth (optional)  
- notes  

#### clusters
- id  
- description  
- representative_face_id  

#### jobs
- type  
- status  
- progress  
- timestamps  

---

## 🔄 Workflow

### 1. Ordnerscan
- Das Backend erhält Zugriff auf einen Netzwerkordner  
- Neue Dateien werden erkannt (Dateiname + Hash)

---

### 2. Fotoanalyse
- Gesichtserkennung (InsightFace)  
- Embeddings erstellen  
- Bounding Boxes als relative Werte speichern  
- Crops erzeugen (persistent)  
- Gebäude-/Ortsmerkmals-Erkennung optional

---

### 3. Matching
- Vergleich der Embeddings mit existierenden Personen  
- Ähnlichkeitsbasierte Vorschläge  
- Unsichere Fälle werden markiert  

---

### 4. Clustering unbekannter Gesichter
- Automatische Gruppierung neuer/unbekannter Personen  
- Benutzer kann Cluster benennen → komplette Zuordnung  

---

### 5. Manuelle Zuordnung
Über das Web-Frontend:
- Zuordnung von Gesichtern zu bestehenden Personen  
- Neue Personen anlegen  
- Korrekturen durchführen  
- Gebäude zuordnen  
- Ort/Zeit bearbeiten

---

### 6. Export (End-Output)
Automatische Erzeugung hochwertiger Ausgabe-Bilder:

Für jedes Originalfoto optional:
- Rahmen um erkannte Personen  
- Namen direkt am Rahmen  
- Titel/Ort/Jahr/Anmerkungen im Bild  
- Export in Unterordner (z. B. `/annotated/1964/11/`)  
- Optional: „Story-Karten“ (eine Seite pro Bild mit Infos)

---

## 🧩 Ordnerstruktur (vorgeschlagen)

```
project/
│── app/
│ ├── api/
│ ├── frontend/
│ ├── jobs/
│ ├── models/
│ ├── services/
│ └── utils/
│
│── data/
│ ├── db.sqlite
│ ├── previews/
│ └── annotated/
│
│── config/
│── scripts/
│── README.md
│── requirements.txt
│── run.py
```

---

## 🧠 KI-Modelle

### Gesichtserkennung
- **InsightFace FaceAnalysis**
- ArcFace-Embeddings (512-D)

### Gebäude-/Szenenerkennung (optional)
- EfficientNet oder MobileNetV3  
- Modell auf „common landmarks“ trainierbar (oder definierte Custom-Klassen)  

### Altersrobustheit  
ArcFace liefert sehr stabile Embeddings über Jahrzehnte → Empfehlung fest integriert.

---

## ⚙️ Setup / Installation

```
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Konfiguration des Foto-Quellordners in `config/settings.yaml`:

```
PHOTO_SOURCE: "/mnt/photos"
Server starten:

python run.py

```

## 🧪 Development Principles

TDD für stabilere Kernmodule (embedding, matching, clustering)

Klar getrennte Module (Scanner, Analyzer, Matcher, Exporter …)

Keine Datenverluste → niemals über Gesichter/Personen drüber schreiben

Reproduzierbarkeit: gleiche Eingaben → gleiche Ergebnisse

Lokalität: keine Cloud, keine externen Uploads, volle Datenhoheit

Erweiterbarkeit > Geschwindigkeit

Keine Vorlaufoptimierung – erstmal Funktionssicherheit

## 🛣 Roadmap
### Phase 1 – Grundgerüst

Projektstruktur aufsetzen

SQLite-Schema definieren

Scanner implementieren

Face Detection + Embeddings

Preview-Generator

### Phase 2 – Matching & UI

Matching-Logik (Thresholds)

Personenverwaltung

Foto-Detailansicht

Clusterer

### Phase 3 – Gebäude-/Ortsanalyse

Landmark-Erkennung

Ortsvorschläge pro Bild

UI für Ort-Review

### Phase 4 – Export

Annotated Image Renderer

Ausgabe als JPEG/PNG

Stapelverarbeitung

Export-Folder-Struktur

### Phase 5 – Feinschliff & Release

 Dokumentation

Settings & Konfiguration

Optional: Offline-Modell-Downloads

## 📔 Worklog (Template)
```
## YYYY-MM-DD – <Kurzer Titel>
### What was done
- …

### Problems / Open Questions
- …

### Decisions
- …

### Next Steps
- …

```
