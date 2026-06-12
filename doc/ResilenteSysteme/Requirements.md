# Requirements

## 1. Application Description

Sub-GHz Transceiver-Plattform auf Basis eines TI CC1125, gesteuert von einem
Microchip PolarFire SoC (FPGA-SoC). Die selbst entworfene MikroBus-PCB mit
CC1125 + Spannungsversorgung wird über einen SPI-Header an das PolarFire
Kit angebunden. Der MSS (Microcontroller Subsystem) initialisiert den
Transceiver, sendet/empfängt Funkpakete und kommuniziert mit dem Fabric.
Das Fabric implementiert eine einfache Status-Logik (LED-Anzeige).

**Use-Case:** Funk-Datenübertragung im Sub-GHz-Band (Demonstrator, kein
finales Produkt) — Anwendung im Rahmen der Lehrveranstaltung.

**Abgeleitete Anforderungen** siehe Abschnitt 2.

## 2. Requirements (abgeleitet)

### 2.1 Funktional
- **F1**: SPI-Kommunikation zwischen PolarFire MSS und CC1125
- **F2**: CC1125 lässt sich initialisieren und konfigurieren
  (SmartRF-Settings: Frequenz, Modulation, Datenrate)
- **F3**: 7V Boost-Converter versorgt den CC1125 (oder PA, falls vorhanden)
- **F4**: RF-Ausgang des CC1125 funktioniert (Senden eines Pakets
  nachweisbar — Spektrum/Sniffer oder Loopback)
- **F5**: Status-LED im Fabric zeigt CC1125-Zustand an
  (Init ok / SPI aktiv / TX läuft)
- **F6**: UART-Ausgabe am PC für Debug/Status

### 2.2 Nicht-funktional
- **NF1**: Code in C für MSS (SoftConsole, RISC-V) — entspricht
  Lehrinhalt SoftConsole/MSS
- **NF2**: Fabric-Design in Libero (HDL/SmartDesign) — entspricht
  Lehrinhalt Libero/Fabric
- **NF3**: PCB-Design in EDA-Tool (KiCad/Altium) — entspricht
  Lehrinhalt PCB-Design
- **NF4**: Python-Skript auf PC-Seite zum Parsen/Loggen der UART-Ausgabe
  (optional) — entspricht Lehrinhalt Python
- **NF5**: Nachvollziehbarkeit: Git-Repository mit Commits, Schematic
  und Doku committed

## 3. Interfaces

- **MSS → CC1125**: SPI (MOSI, MISO, SCK, CS) über Header-Kabel
- **MSS → PC**: UART (Log/Debug)
- **MSS → Fabric**: GPIO-Status-Signale
  (`cc1125_ready`, `spi_busy`, `tx_active`)
- **Fabric → LED**: 1 GPIO-Pin am PolarFire-Board
- **Power**: 7V-Versorgung für Boost-Converter
- **Antenne**: Sub-GHz-Antenne am SMA/PCB-Pad

## 4. Systems-Thinking-Pfad

Anwendung des Lehrkonzepts **Idea → Concept → Specification → Design →
Development → Breadboard**:

| Phase | Stand |
|---|---|
| Idea | Sub-GHz Transceiver-Demo |
| Concept | CC1125 + PolarFire SoC + MikroBus-PCB |
| Specification | dieses Dokument (Requirements.md) |
| Design | KiCad-Schematic, Libero-MSS-Config, Fabric-SmartDesign |
| Development | SoftConsole-Firmware, Libero-Build |
| Breadboard | gelötete PCB, Messungen im RF-Lab |

## 5. Project Planning

### 5.1 Work Breakdown (2er-Team)

| Aufgabe | Verantwortlich |
|---|---|
| PCB-Design (Schematic, Layout, Gerber) | Person A |
| PCB-Bestückung/Löten | Person A + B |
| Libero: MSS-Config, Pin-Assignments, Fabric-LED-Modul | Person B |
| SoftConsole: SPI-Treiber, CC1125-Init, GPIO-Steuerung | Person B |
| Test/Verifikation (SPI, RF, Lab) | Person A + B |
| Doku & Präsentation | Person A + B |
| Python-Log-Skript (optional) | Person A |

### 5.2 Grobe Meilensteine

1. Schematic/PCB fertig, Gerber exportiert
2. PCB bestückt + optisch/ elektrisch geprüft
3. Libero-Projekt: MSS + Fabric kompiliert, Bitstream flasht
4. SPI-Kommunikation steht (PARTNUM/MFR lesbar)
5. RF-Sendetest (Spektrum sichtbar)
6. LED-Fabric-Modul funktioniert
7. Doku/Präsentation fertig

## 6. Building

- PCB im SolderLab bestücken und löten
- SPI-Header-Adapter (Kabel/Wire-Wrap) zwischen PolarFire-Kit und CC1125-PCB
- Libero: SmartDesign mit Fabric-LED-FSM, MSS-Config (SPI/UART/GPIO),
  Pin-Mapping
- SoftConsole: SPI-Init, CC1125-Register-Schreib-/Lese-Routinen,
  GPIO-Setter je nach State
- Bitstream erzeugen, auf PolarFire programmieren

## 7. Testing / Verification of Results

- **T1**: SPI OK → Auslesen von PARTNUM (0x58) und MFR (0x48) liefert
  korrekte Werte → UART-Print
- **T2**: LED-Test → LED an, wenn CC1125-Init erfolgreich
- **T3**: RF-Sende-Test → Spektrum im RF-Lab Hiebel sichtbar
  (Loopback mit zweitem Knoten optional)
- **T4**: Python-Skript liest UART-Log und visualisiert/parsed Werte
- **T5**: Verifikations-Übersicht (was funktioniert / was nicht) in
  Präsentation

## 8. Using

**Demo-Ablauf:**
1. PolarFire einschalten, Bitstream läuft
2. PC-Terminal (UART) + Python-Log öffnen
3. MSS initialisiert CC1125, LED geht an
4. Initiierte SPI-Transfers am Log sichtbar, LED blinkt bei TX
5. Sende-Paket wird im RF-Lab oder per Sniffer verifiziert

## 9. Assessment-Kriterien (Selbst-Check)

Bewertet wird mit 20 Punkten / 20% Anteil:
- **Funktionalität / Umfang / Erreichung**: läuft SPI, läuft RF,
  läuft LED-Fabric?
- **Professionalität / Planung / Ausführung / Zeitplan**: Git-Historie,
  durchdachtes Design, sauberer Code, WBS
- **Verständnis der Embedded-System-Anwendung**: warum diese
  Architektur, was läuft wo (MSS vs. Fabric), warum?
- **Vorlesungsinhalte abgedeckt**: Libero (Fabric), SoftConsole (MSS,
  C), PCB-EDA, Python

## 10. Offene Punkte / Risiken

- R: HF-Layout / Reichweite (Antenne, Matching) — Mitigation: im
  RF-Lab messen, ggf. Sniffer verwenden
- R: PolarFire-Kit-Variante (MPFS250T, MPFS460, …) bestimmt MSS-Config
  und Toolchain-Version
- R: 7V-Boost liefert unter Last genug Strom? — Spec-Sheet prüfen
- R: Zweiter CC1125-Knoten für Loopback nicht garantiert verfügbar
