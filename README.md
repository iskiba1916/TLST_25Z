# 🌐 Sieć Transportowa w Finlandii

> **Projekt z przedmiotu Teleinformatyczne Sieci Transportowe (TLST)**  
> Wydział Elektroniki i Technik Informacyjnych, Politechnika Warszawska

---

## 👥 Autorzy

| Imię i Nazwisko |
|---|
| Jakub Surma |
| Albert Kołsut |
| Igor Skiba |
| Radosław Rytel-Kuc |

**Data:** 5 stycznia 2026 &nbsp;·&nbsp; 

---

## 📋 Opis projektu

Projekt obejmuje kompleksowe zaprojektowanie krajowej sieci transportowej dla Finlandii — od analizy potrzeb transmisyjnych, przez dobór topologii i urządzeń, aż po wymagania formalno-prawne. Sieć oparta jest na technologiach **DWDM/OTN** oraz **IP/MPLS**, z wykorzystaniem platformy **Nokia 1830 PSS** i routerów **Nokia 7750 SR**.

---

## 🗺️ Topologia sieci

Zaprojektowano hierarchiczną architekturę dwupierścieniową z magistralą północną:

```
                        [ Ivalo ]
                            |
                       [ Rovaniemi ]
                            |
Helsinki ── Turku ── Tampere ── Oulu
    |                   |
  Lahti            Jyväskylä
    |                   |
Helsinki ◄──────── Pihtipudas
```

### Główny Pierścień Południowy (Core Ring)

| Węzeł | Mieszkańcy | Odległość |
|---|---|---|
| Helsinki (Core) | 660 000 | — |
| Turku | 206 000 | 165 km |
| Tampere | 255 000 | 160 km |
| Lahti | 121 000 | 130 km |
| Helsinki | — | 105 km |

### Magistrala Północna (Tampere → Oulu)

| Węzeł | Typ | Odległość |
|---|---|---|
| Tampere | Core | — |
| Jyväskylä | Wzmacniacz EDFA | 150 km |
| Pihtipudas | Wzmacniacz EDFA | 130 km |
| Liminka | Wzmacniacz EDFA | 140 km |
| Oulu | Regional Core | 30 km |

---

## 📡 Potrzeby transmisyjne

Prognozowany wzrost zapotrzebowania na transmisję wynika z jednego z najwyższych na świecie wskaźników zużycia danych mobilnych w Finlandii:

| Parametr | Obecnie | Perspektywa długoterminowa |
|---|---|---|
| Zużycie danych / użytkownik / miesiąc | ~kilkanaście GB | ~1 500 GB |
| Przepływność / mieszkaniec (avg) | ~0,05 Mb/s | ~4,6 Mb/s |
| Obciążenie magistrali (Helsinki area) | kilka Gb/s | **rzędu Tb/s** |

---

## ⚙️ Wybrane urządzenia

### Warstwa optyczna DWDM/OTN

**Nokia 1830 PSS-32** (węzły Core: Helsinki, Turku, Tampere)
- Architektura przełączania **CDC-F** (Colorless, Directionless, Contentionless, Flexgrid)
- Transpondery koherentne **PSE-V Super Coherent** — od 100G do 800G na jednej fali
- Dynamiczna adaptacja modulacji (Probabilistic Constellation Shaping)
- Wzmacniacze EDFA + hybrydowe wzmacniacze Ramanowskie

**Nokia 1830 PSS-8** — węzły mniejsze (Lahti) i lokalizacje wzmacniakowe

### Warstwa IP/MPLS (L3)

**Nokia 7750 SR-14s**
- Chipset sieciowy **FP5** z hardwarowym szyfrowaniem ANYsec/MACsec
- Interfejsy 400GE (QSFP-DD) i 800GE
- System operacyjny **SR OS** z obsługą Segment Routing (SR-MPLS)
- Protokoły: IS-IS (routing wewnętrzny), BGP (styki zagraniczne), MPLS-TE

### Infrastruktura towarzysząca

| Komponent | Producent / Model | Zastosowanie |
|---|---|---|
| ODF | Huber+Suhner LISA | Przełącznice o wysokiej gęstości (złącza LC/APC Grade B) |
| Zasilanie DC | Vertiv NetSure 7100 | Siłownia –48V DC, sprawność >96,5%, autonomia 8h |

---

## 🔌 Kable światłowodowe (TFKable)

| Relacja | Model kabla | Włókna |
|---|---|---|
| Helsinki Core – Lahti (szkieletowa) | Z-XOTKtsd 144J | G.652.D |
| Helsinki Core – C-Lion1 Landing | Z-XOTKtsd 144J | G.652.D |
| Ring Miejski Helsinki | Z-XOTKtsd 72J | G.652.D |
| Dojście do węzła (Last Mile) | Z-XOTKtsd 48J | G.657.A2 |
| Wprowadzenie do serwerowni | ZW-NOTKtsd 48J/96J | LSOH |

---

## 🌍 Styki z sieciami zewnętrznymi

Łączna zaprojektowana przepustowość styków: **24 Tb/s**

| Kierunek | Lokalizacja styku | Technologia | Przepustowość |
|---|---|---|---|
| **Południe** (Europa) | Helsinki – C-Lion1 | Alien Wavelength, 400GBASE-LR8 | dominująca |
| **Zachód** (Szwecja) | Turku (podmorski) + Tornio (lądowy) | Koherentne 200 Gb/s | — |
| **Północ** (Norwegia) | Utsjoki / Kilpisjärvi | Światłowód lądowy | 2×100 Gb/s |
| **Wschód** (Rosja) | Vaalimaa | Dedykowane VRF, ścisłe filtrowanie | tranzytowy |

Wszystkie styki zabezpieczone **eBGP** z politykami AS-Path Prepending, **RPKI** (ochrona przed BGP Hijacking) oraz QoS DSCP marking.

---

## ⏱️ Synchronizacja sieci

Hierarchiczny system zgodny z **ITU-T G.8275.1** i **G.8262**:

- **PRTC (Primary Reference Time Clock)** — Helsinki: odbiornik GPS/Galileo/GLONASS + router Nokia 7750 SR jako **Grandmaster Clock**
- **Georedundancja** — zapasowy zegar w Oulu (>500 km od Helsinek)
- **Holdover** — oscylatory cezowe/rubidowe (ePRTC), stabilność <1,5 µs przez min. 14 dni
- **SyncE** (warstwa fizyczna) + **PTP IEEE 1588v2** (warstwa pakietowa)
- **SSM** (Synchronization Status Messages) — ochrona przed pętlami synchronizacyjnymi
- Styki zagraniczne: **Sync Reference Inhibit** — izolacja domeny fińskiej od zegarów obcych sieci

---

## ⚖️ Wymagania formalno-prawne (Finlandia)

| Obszar | Wymaganie |
|---|---|
| Rejestracja | Notyfikacja w **Traficom** (Fińska Agencja Transportu i Komunikacji) |
| Pas drogowy | Pozwolenie **Sijoituslupa** — organy **ELY-keskus** (drogi krajowe) lub urząd miasta |
| Grunty prywatne | Umowa cywilna lub przymusowa służebność (**Rasitetoimitus**) przez Maanmittauslaitos |
| Wody i środowisko | Pozwolenie wodne **Vesilupa** (AVI), ocena oddziaływania na obszary Natura 2000 |
| Bezpieczeństwo | Dyrektywa **NIS2**, Lawful Interception dla Supo, dane topologii wyłącznie w EOG |
| Współbudowanie | Obowiązek zgłoszenia w portalu **Verkkotietopiste.fi** i udostępnienia wykopu innym operatorom |

---

## 📚 Literatura i źródła

- ITU-T G.709 — *Interfaces for the Optical Transport Network*
- ITU-T G.8275.1 — *Precision time protocol telecom profile for phase/time synchronization*
- ITU-T G.8262 — *Timing characteristics of a synchronous Ethernet equipment slave clock*
- Nokia 1830 PSS Product Documentation
- Nokia 7750 SR Product Documentation
- Huber+Suhner LISA System Catalogue
- Vertiv NetSure 7100 Series Datasheet
- TFKable — Katalog kabli światłowodowych zewnętrznych
- Laki sähköisen viestinnän palveluista 917/2014 (Finlandia)
- Yhteisrakentamislaki 276/2016 (Finlandia)

---

<p align="center">
  <sub>Politechnika Warszawska · Wydział Elektroniki i Technik Informacyjnych · TLST 2025/2026</sub>
</p>
