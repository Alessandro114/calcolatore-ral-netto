# Calcolatore RAL → Netto

**Product Builder Task — Jet HR**

Prototipo funzionante di un calcolatore che, data una Retribuzione Annua Lorda (RAL), restituisce il netto annuale e mensile percepito dal dipendente, con il dettaglio di tutte le voci trattenute.

**[→ Demo live su GitHub Pages](https://alessandro114.github.io/jet-hr-ral-calculator/)**

---

## Sommario

- [Come funziona](#come-funziona)
- [Pipeline di calcolo](#pipeline-di-calcolo)
- [Dettaglio di ogni componente](#dettaglio-di-ogni-componente)
  - [1. Contributi INPS dipendente](#1-contributi-inps-dipendente)
  - [2. Fringe benefit](#2-fringe-benefit)
  - [3. Fondo pensione complementare](#3-fondo-pensione-complementare)
  - [4. IRPEF 2025](#4-irpef-2025)
  - [5. Detrazioni lavoro dipendente](#5-detrazioni-lavoro-dipendente)
  - [6. Detrazioni familiari a carico](#6-detrazioni-familiari-a-carico)
  - [7. Bonus cuneo fiscale 2025](#7-bonus-cuneo-fiscale-2025)
  - [8. Trattamento integrativo](#8-trattamento-integrativo)
  - [9. Addizionale regionale IRPEF](#9-addizionale-regionale-irpef)
  - [10. Addizionale comunale IRPEF](#10-addizionale-comunale-irpef)
  - [11. TFR — Trattamento di Fine Rapporto](#11-tfr--trattamento-di-fine-rapporto)
  - [12. Costo azienda](#12-costo-azienda)
- [Input disponibili](#input-disponibili)
- [Output generati](#output-generati)
- [Semplificazioni e limiti noti](#semplificazioni-e-limiti-noti)
- [Struttura del progetto](#struttura-del-progetto)
- [Come eseguire](#come-eseguire)

---

## Come funziona

Il calcolatore è una **singola pagina HTML** senza dipendenze esterne (zero framework, zero build step). Tutta la logica fiscale è implementata in JavaScript vanilla con funzioni isolate e documentate.

L'utente inserisce i parametri → clicca "Calcola" → ottiene:
- **Card di sintesi** con netto annuale, mensile, trattenute e costo azienda
- **Barra visuale** con la composizione della RAL (netto vs. imposte)
- **Tabella breakdown** con ogni voce di calcolo
- **Dettaglio IRPEF** per scaglione
- **Dettaglio addizionale regionale** per scaglione
- **Sezione TFR** con accantonamento e tassazione separata stimata
- **Dettaglio cuneo fiscale** (quando applicabile)
- **Lista delle ipotesi** adottate

---

## Pipeline di calcolo

```
RAL (Retribuzione Annua Lorda)
  + Fringe benefit tassabili (se superano la soglia di esenzione)
  − Contributo a fondo pensione (deducibile, max €5.164,57)
  − Contributi INPS dipendente
  ─────────────────────────────────────────
  = REDDITO COMPLESSIVO (imponibile fiscale)
  ─────────────────────────────────────────
  − IRPEF lorda (3 scaglioni progressivi)
  + Detrazioni lavoro dipendente (art. 13 TUIR)
  + Detrazioni familiari a carico (coniuge, figli ≥ 21, altri)
  + Ulteriore detrazione cuneo fiscale (redditi €20K-€40K)
  ─────────────────────────────────────────
  = IRPEF NETTA
  ─────────────────────────────────────────
  − Addizionale regionale IRPEF
  − Addizionale comunale IRPEF
  + Indennità cuneo fiscale (redditi ≤ €20K, non tassabile)
  + Trattamento integrativo (se spettante)
  ─────────────────────────────────────────
  = NETTO ANNUALE
  ÷ mensilità (13 o 14)
  = NETTO MENSILE

  Separatamente: TFR accantonato (non transita in busta paga)
```

---

## Dettaglio di ogni componente

### 1. Contributi INPS dipendente

I contributi previdenziali a carico del lavoratore finanziano la pensione futura.

| Settore | Aliquota | Note |
|---------|----------|------|
| **Privato** | 9,19% fino a €55.448; 10,19% oltre | La soglia (prima fascia pensionabile) è aggiornata annualmente dall'INPS. L'1% aggiuntivo è previsto dall'art. 3-ter L. 438/1992 |
| **Pubblico** | 8,80% flat | Aliquota unificata gestione ex-INPDAP (CTPS/CPDEL). Non prevede soglia |

**Cosa non è incluso:** il contributo INPS a carico del datore (~24-32%) viene conteggiato solo nella stima del costo azienda.

**Riferimenti normativi:** L. 335/1995 (riforma Dini), L. 438/1992 (contributo aggiuntivo 1%), Circ. INPS annuali.

---

### 2. Fringe benefit

I fringe benefit sono compensi in natura (auto aziendale, buoni pasto, telefono, alloggio) che il datore eroga al dipendente.

| Condizione | Soglia esenzione 2025-2027 |
|------------|---------------------------|
| Dipendente **senza** figli a carico | **€1.000** |
| Dipendente **con** figli a carico | **€2.000** |

**Regola chiave:** se il valore complessivo dei fringe benefit **supera** la soglia, l'**intero importo** diventa reddito imponibile (non solo l'eccedenza). È un meccanismo a franchigia, non a deduzione.

**Nel calcolatore:** l'utente inserisce il valore totale dei fringe benefit annui. Se sopra soglia, l'importo si somma all'imponibile fiscale.

**Riferimenti:** Art. 51, c.3 TUIR; L. 207/2024 (Legge di Bilancio 2025), art. 1 c. 390-391.

---

### 3. Fondo pensione complementare

I contributi versati a forme di previdenza complementare (fondi negoziali, aperti, PIP) sono **deducibili dal reddito complessivo** fino a **€5.164,57/anno**.

Questo riduce direttamente l'imponibile IRPEF, generando un risparmio fiscale pari al contributo × aliquota marginale IRPEF del contribuente.

**Esempio:** un dipendente con RAL €35.000 che versa €2.000 al fondo pensione risparmia circa €2.000 × 35% = €700 di IRPEF.

**Cosa non è incluso nel calcolatore:** il contributo del datore al fondo pensione (anch'esso deducibile entro lo stesso tetto).

**Riferimenti:** D.Lgs. 252/2005, art. 8 c. 4.

---

### 4. IRPEF 2025

L'Imposta sul Reddito delle Persone Fisiche è calcolata per scaglioni progressivi. Dal 2024 (D.Lgs. 216/2023), confermati strutturalmente nel 2025, gli scaglioni sono 3:

| Scaglione | Aliquota | Su reddito fino a | Imposta cumulata |
|-----------|----------|-------------------|------------------|
| 1° | **23%** | €28.000 | €6.440 |
| 2° | **35%** | €50.000 | €6.440 + €7.700 = €14.140 |
| 3° | **43%** | oltre | €14.140 + 43% sull'eccedenza |

L'IRPEF è **progressiva per scaglioni**: ogni euro aggiuntivo è tassato all'aliquota dello scaglione in cui ricade, non all'aliquota più alta sull'intero reddito.

**Base imponibile IRPEF** = RAL + fringe tassabili − INPS dipendente − fondo pensione deducibile.

**Riferimenti:** Art. 11 TUIR; D.Lgs. 216/2023; L. 207/2024.

---

### 5. Detrazioni lavoro dipendente

Le detrazioni riducono l'IRPEF lorda e sono inversamente proporzionali al reddito (chi guadagna meno, paga meno imposte).

| Reddito complessivo | Detrazione |
|---------------------|-----------|
| ≤ €15.000 | €1.955 |
| €15.001 – €28.000 | €1.910 + €1.190 × (28.000 − reddito) / 13.000 |
| €28.001 – €50.000 | €1.910 × (50.000 − reddito) / 22.000 |
| > €50.000 | €0 |

**Bonus €65:** per redditi tra €25.001 e €35.000 si aggiungono €65 alla detrazione calcolata (D.Lgs. 216/2023).

**Vincolo:** la detrazione non può superare l'IRPEF lorda (non genera credito d'imposta).

**Riferimenti:** Art. 13 TUIR; D.Lgs. 216/2023 art. 1 c. 2.

---

### 6. Detrazioni familiari a carico

Dopo l'introduzione dell'Assegno Unico (marzo 2022), le detrazioni in busta paga riguardano solo:

#### Coniuge a carico (reddito ≤ €2.840,51)

| Reddito del contribuente | Detrazione |
|--------------------------|-----------|
| ≤ €15.000 | 800 − 110 × (reddito / 15.000) |
| €15.001 – €40.000 | €690 (con variazioni tra €29K-€35,2K: v. tabella nel codice) |
| €40.001 – €80.000 | 690 × (80.000 − reddito) / 40.000 |
| > €80.000 | €0 |

#### Figli a carico ≥ 21 anni

- **€950** per ciascun figlio ≥ 21 con reddito ≤ €2.840,51 (o ≤ €4.000 se under 24)
- Phase-out: la detrazione si riduce proporzionalmente: × max(0, (95.000 − reddito) / 95.000)
- Per figli **< 21 anni**: si percepisce l'**Assegno Unico** (erogato dall'INPS, fuori dalla busta paga)

#### Altri familiari a carico

- **€750** per ciascuno (genitori, fratelli, ecc. conviventi con reddito ≤ €2.840,51)
- Phase-out: × max(0, (80.000 − reddito) / 80.000)

**Riferimenti:** Art. 12 TUIR; D.Lgs. 230/2021 (Assegno Unico).

---

### 7. Bonus cuneo fiscale 2025

La Legge di Bilancio 2025 (L. 207/2024) ha reso **strutturale** la riduzione del cuneo contributivo, trasformandola in un meccanismo fiscale a due componenti:

#### A) Indennità (redditi ≤ €20.000) — non tassabile

Per i redditi più bassi, si eroga un'indennità calcolata come percentuale del reddito da lavoro dipendente:

| Reddito | Percentuale |
|---------|-------------|
| ≤ €8.500 | 7,1% |
| €8.501 – €15.000 | 5,3% |
| €15.001 – €20.000 | 4,8% |

Questa indennità **non concorre alla formazione del reddito** (non è tassata) e si aggiunge direttamente al netto.

#### B) Ulteriore detrazione IRPEF (redditi €20.001 – €40.000)

| Reddito | Detrazione |
|---------|-----------|
| €20.001 – €32.000 | €1.000 fissi |
| €32.001 – €40.000 | €1.000 × (40.000 − reddito) / 8.000 |
| > €40.000 | €0 |

Questa detrazione riduce l'IRPEF netta (si somma alle detrazioni lavoro dipendente).

**Cosa ha sostituito:** il taglio contributivo del 6-7% sulle buste paga del 2023-2024 (che era temporaneo).

**Riferimenti:** L. 207/2024, art. 1 cc. 4-9.

---

### 8. Trattamento integrativo

Il "trattamento integrativo" (ex Bonus Renzi, ex Bonus 80€) è un credito IRPEF di **€1.200/anno (€100/mese)**.

Spetta a condizione che:
- Il reddito complessivo sia **≤ €15.000**
- L'IRPEF lorda sia **superiore** alle detrazioni da lavoro dipendente (altrimenti non c'è IRPEF da cui "scontare" il bonus)

**Interazione col cuneo fiscale 2025:** per redditi ≤ €15.000, l'indennità del cuneo fiscale (componente A) è generalmente più vantaggiosa del trattamento integrativo. Nel calcolatore, quando è attiva l'indennità cuneo, il trattamento integrativo non viene erogato per evitare il doppio beneficio.

**Riferimenti:** Art. 1 D.L. 3/2020 (conv. L. 21/2020).

---

### 9. Addizionale regionale IRPEF

Ogni Regione/Provincia Autonoma applica un'aliquota sull'imponibile IRPEF. Alcune hanno aliquota unica, altre usano scaglioni progressivi.

Il calcolatore include **tutte le 21 regioni** (incluse le Province Autonome di Trento e Bolzano) con le aliquote aggiornate al 2025:

| Regione | Tipo | Range aliquote |
|---------|------|----------------|
| Abruzzo | 3 scaglioni | 1,67% – 3,33% |
| Basilicata | Unica | 1,23% |
| Calabria | Unica | 1,73% |
| Campania | 4 scaglioni | 1,73% – 3,33% |
| Emilia-Romagna | 4 scaglioni | 1,33% – 3,33% |
| Friuli Venezia Giulia | 2 scaglioni | 0,70% – 1,23% |
| Lazio | 2 scaglioni | 1,73% – 3,33% |
| Liguria | 3 scaglioni | 1,23% – 3,23% |
| Lombardia | 4 scaglioni | 1,23% – 1,73% |
| Marche | 4 scaglioni | 1,23% – 1,73% |
| Molise | 3 scaglioni | 1,73% – 3,33% |
| Piemonte | 4 scaglioni | 1,62% – 3,33% |
| Puglia | 4 scaglioni | 1,33% – 1,85% |
| Sardegna | Unica | 1,23% |
| Sicilia | Unica | 1,23% |
| Toscana | 4 scaglioni | 1,42% – 3,33% |
| Trentino-AA (Bolzano) | 2 scaglioni | 1,23% – 1,73% |
| Trentino-AA (Trento) | 3 scaglioni | 0% – 1,73% |
| Umbria | 3 scaglioni | 1,23% – 3,33% |
| Valle d'Aosta | Unica | 1,23% |
| Veneto | Unica | 1,23% |

**Nota:** Trento ha una no-tax area fino a €27.000 (aliquota 0%).

**Fonti:** Delibere regionali anno d'imposta 2024/2025; QuantoPrendo.io, Money.it.

---

### 10. Addizionale comunale IRPEF

Ogni Comune italiano stabilisce la propria aliquota (0% – 0,9% max) sull'imponibile IRPEF.

Il calcolatore include i **principali comuni** per ogni regione con aliquote preimpostate, più la possibilità di inserire un'aliquota personalizzata per comuni non in lista.

**Esempi:** Milano 0,8% · Roma 0,9% · Napoli 0,9% · Firenze 0,2% · Bologna 0,8% · Bolzano 0,1%.

**Fonti:** Delibere comunali; Portale del Federalismo Fiscale (MEF).

---

### 11. TFR — Trattamento di Fine Rapporto

Il TFR è una quota della retribuzione che il datore **accantona** annualmente e che viene erogata alla cessazione del rapporto di lavoro. **Non transita in busta paga** (non riduce il netto mensile), ma è un componente importante del reddito differito.

| Voce | Formula |
|------|---------|
| Accantonamento lordo | RAL / 13,5 (~7,41% della RAL) |
| Contributo INPS fondo garanzia | − 0,50% della RAL |
| **Accantonamento netto** | **~6,91% della RAL** |

**Destinazione del TFR:**
- Lasciato in azienda (per aziende < 50 dipendenti)
- Versato al Fondo di Tesoreria INPS (per aziende ≥ 50 dipendenti)
- Conferito a un fondo pensione complementare (scelta del lavoratore)

**Tassazione:** il TFR è soggetto a **tassazione separata** (non si cumula con il reddito annuo). L'aliquota applicata è la media delle aliquote IRPEF degli ultimi 5 anni di lavoro. Nel calcolatore, stimiamo questa aliquota usando l'aliquota media IRPEF dell'anno corrente.

**Riferimenti:** Art. 2120 Codice Civile; art. 17 TUIR.

---

### 12. Costo azienda

Il "costo azienda" è il costo totale che il datore di lavoro sostiene per il dipendente. È sempre significativamente superiore alla RAL.

| Componente | Privato | Pubblico |
|-----------|---------|----------|
| RAL | 100% | 100% |
| INPS datore | ~31% | ~24,2% |
| TFR accantonamento | ~7,4% | ~7,4% |
| INAIL | ~0,4% | — |
| **Totale stimato** | **~138-140% della RAL** | **~131-132% della RAL** |

**Nota:** la stima è semplificata. Il costo reale varia per settore, CCNL, classe di rischio INAIL, presenza di welfare aziendale, etc.

---

## Input disponibili

| Input | Descrizione | Default |
|-------|-------------|---------|
| RAL | Retribuzione Annua Lorda | €30.000 |
| Mensilità | 13 (tredicesima) o 14 (+ quattordicesima) | 13 |
| Settore | Privato o Pubblico | Privato |
| Regione | 21 opzioni (tutte le regioni italiane) | Lombardia |
| Comune | Principali città per regione + aliquota custom | Milano |
| Coniuge a carico | Checkbox | No |
| Figli ≥ 21 a carico | Numero | 0 |
| Altri familiari a carico | Numero | 0 |
| Figli < 21 a carico | Checkbox (innalza soglia fringe a €2.000) | No |
| Fringe benefit | Importo annuo in € | €0 |
| Fondo pensione | Contributo annuo in € | €0 |

---

## Output generati

1. **Netto annuale** e **netto mensile** (÷ mensilità)
2. **Totale trattenute** con percentuale sulla RAL
3. **TFR accantonato** annuo (lordo e netto stimato)
4. **Costo azienda** stimato
5. **Barra visuale** con la composizione della RAL
6. **Tabella breakdown** voce per voce con segni +/−
7. **Dettaglio IRPEF** per scaglione (imponibile, aliquota, imposta)
8. **Dettaglio addizionale regionale** per scaglione
9. **Dettaglio TFR** (accantonamento, contributo INPS, tassazione)
10. **Dettaglio cuneo fiscale** (tipo, percentuale, importo)
11. **Lista completa delle ipotesi** adottate

---

## Semplificazioni e limiti noti

Questo è un **prototipo** che copre i casi più comuni. Ecco cosa è stato semplificato e perché:

| Semplificazione | Motivazione |
|-----------------|-------------|
| **Assegno Unico non incluso** | L'AU per figli < 21 è erogato dall'INPS separatamente, non transita in busta paga. Includerlo richiederebbe ISEE come input |
| **Detrazioni per oneri non incluse** | Spese mediche, interessi mutuo, ristrutturazioni, ecc. sono soggettive e non prevedibili senza il 730 |
| **Addizionale comunale semplificata** | Alcuni comuni hanno aliquote progressive (non flat). Abbiamo usato i valori noti per i capoluoghi principali |
| **TFR con aliquota media stimata** | La tassazione separata reale usa la media IRPEF degli ultimi 5 anni. Qui usiamo l'aliquota media dell'anno corrente |
| **Interazione cuneo/trattamento** | Il trattamento integrativo e l'indennità cuneo possono interagire in modo complesso per redditi intorno a €15K. Abbiamo semplificato: se il cuneo è attivo, il trattamento non si cumula |
| **CCNL non differenziato** | Diversi CCNL possono prevedere contribuzioni aggiuntive (es. CIGS per aziende > 15 dipendenti: +0,30%). Usiamo l'aliquota base |
| **Massimale contributivo non applicato** | Per i nuovi iscritti post-1996 con RAL > ~€120K c'è un tetto alla contribuzione INPS. Non implementato |
| **Conguaglio 730 non incluso** | Il calcolatore simula il netto "a regime", senza recuperi/addebiti da dichiarazione dei redditi |

---

## Struttura del progetto

```
jet-hr-ral-calculator/
├── index.html    # Tutto il calcolatore (HTML + CSS + JS)
└── README.md     # Questo file
```

Scelta deliberata: **zero dipendenze**. Il file `index.html` è self-contained e apribile direttamente nel browser. Non servono Node.js, npm, build tool, o server. Questo rende il prototipo immediatamente verificabile e deployabile su GitHub Pages senza configurazione.

---

## Come eseguire

### Locale
```bash
# Basta aprire il file nel browser
open index.html
# oppure
python3 -m http.server 8080  # e visitare http://localhost:8080
```

### Online
Il progetto è hostato su GitHub Pages: **[→ Demo live](https://alessandro114.github.io/jet-hr-ral-calculator/)**

---

## Fonti normative principali

- **TUIR** — D.P.R. 917/1986 (Testo Unico Imposte sui Redditi)
- **D.Lgs. 216/2023** — Riforma IRPEF a 3 scaglioni
- **L. 207/2024** — Legge di Bilancio 2025 (cuneo fiscale strutturale, fringe benefit 2025-2027)
- **D.Lgs. 252/2005** — Previdenza complementare
- **D.Lgs. 230/2021** — Assegno Unico Universale
- **D.L. 3/2020** — Trattamento integrativo
- **Art. 2120 c.c.** — Trattamento di Fine Rapporto
- **Circolari INPS** — Aliquote contributive annuali
- **Delibere regionali e comunali** — Addizionali IRPEF
