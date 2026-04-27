# Technical Approach
**RFP MC-2026-0417 — Meridian Components Inventory Dashboard**

---

## Methodology

Our engagement follows an audit-first, test-first approach. We do not rewrite what works; we understand what exists before touching anything. Each requirement is addressed incrementally, with verification at every step. Assumptions are documented upfront and aligned with Meridian before work begins.

---

## R1 — Reports Module Remediation

We will begin with a full audit of the Reports module before writing a single line of fix. Every defect will be documented — not just the eight referenced in the RFP, but any additional issues surfaced during the audit. This gives Meridian a complete picture of the module's state and a baseline for sign-off.

Fixes will be verified against real combinations of the four available filters (Time Period, Warehouse, Category, Order Status), not just isolated cases. Internationalization gaps will be catalogued and addressed as part of this phase, with particular attention to the Tokyo warehouse context.

**Assumption:** The complete defect list will be aligned with Meridian's operations team before remediation begins. If Meridian holds a tracked issue log, we request it at project kickoff; otherwise, our audit output becomes the agreed baseline.

---

## R3 — Automated Browser Testing *(delivered early)*

We are deliberately sequencing R3 earlier than its RFP order. Automated test coverage is the prerequisite that unblocks IT's approval for all subsequent deployments — including the fixes from R1 and the new capability in R2. Delivering it late would bottleneck the entire engagement.

Tests will be written using Playwright, which is already configured in the project repository. Coverage will target the critical user flows across the Reports module (post-R1) and the new Restocking view (post-R2). Edge cases — empty states, filter combinations with no results, budget ceiling boundary conditions — will be explicitly included.

**Assumption:** The definition of "critical flows" will be confirmed with Meridian IT before test authoring begins. We will provide a proposed flow list for their review and approval.

---

## R2 — Restocking Recommendations

We will deliver a new Restocking view as a first-class module in the existing Vue application, consistent with the current navigation and visual patterns. The view will accept three inputs from the operator: current stock levels (drawn from the existing `/api/inventory` endpoint), demand forecast (from `/api/demand`), and a budget ceiling entered directly in the UI.

The output will be a prioritized list of recommended purchase orders — item, quantity, estimated cost, and justification — ranked by operational urgency.

For the reorder logic: if Meridian provides existing thresholds or algorithms, we will implement them exactly. If not, we will propose a model based on remaining days of coverage against supplier lead time, present it for review, and proceed on written approval. The model will be documented so Meridian's team can maintain and adjust it after handoff.

**Assumption:** Stock and demand data sufficient to drive recommendations are available via the existing API endpoints. Any gaps discovered during development will be surfaced immediately for Meridian's input.

---

## R4 — Architecture Documentation

We will produce a current-state architecture overview as a self-contained HTML document, suitable for distribution to Meridian IT without requiring any tooling or special software to view. The document will cover:

- System components and their roles (Vue frontend, FastAPI backend, JSON data layer)
- Data flow from UI filters through the API client, backend processing, and back to computed properties
- API surface: endpoints, parameters, and response shapes
- Known limitations and technical debt inherited from the previous vendor

The document is written for IT generalists, not developers — diagrams over source references, plain language over jargon.

---

## D1 — UI Modernization *(desired)*

We will refresh the visual design of the dashboard to align with contemporary standards for data-dense operational interfaces. Scope and reference point are contingent on Meridian providing a brand guide or style reference. Without one, we will propose a direction for approval before implementation.

---

## D2 — Internationalization *(desired)*

The existing codebase has partial i18n infrastructure in place. We will extend it to cover remaining modules, with Tokyo warehouse staff as the primary audience. Translation strings will be delivered in a format Meridian's team can maintain without developer involvement.

---

## D3 — Dark Mode *(desired)*

The codebase uses CSS custom properties (design tokens) for its color system, which makes a theme toggle a natural extension rather than a full redesign. We will implement an operator-selectable light/dark theme persisted across sessions. This directly addresses the low-light environment requirement for warehouse floor stations.

---

## Explicit Assumptions

| # | Assumption |
|---|---|
| A1 | The complete defect list for R1 will be provided by Meridian or validated against our audit output before remediation begins |
| A2 | "Critical flows" for R3 will be confirmed with Meridian IT before test authoring begins |
| A3 | For R2, in the absence of a Meridian-supplied reorder algorithm, the vendor will propose a days-of-coverage model for written approval |
| A4 | D1 scope is contingent on Meridian providing a brand guide or style reference |

---

*Submitted by: [Your Firm Name]*
*Contact: g.viscoli@accenture.com*
*Date: April 27, 2026*

---
---

# Approccio Tecnico
**RFP MC-2026-0417 — Meridian Components Inventory Dashboard**

---

## Metodologia

Il nostro ingaggio segue un approccio audit-first, test-first. Non riscriviamo ciò che funziona; comprendiamo ciò che esiste prima di toccare qualsiasi cosa. Ogni requisito viene affrontato in modo incrementale, con verifica a ogni fase. Le assunzioni sono documentate in anticipo e allineate con Meridian prima dell'avvio dei lavori.

---

## R1 — Remediation del modulo Reports

Inizieremo con un audit completo del modulo Reports prima di scrivere una singola riga di correzione. Ogni difetto sarà documentato — non solo gli otto citati nella RFP, ma qualsiasi problema aggiuntivo emerso durante l'audit. Questo fornisce a Meridian un quadro completo dello stato del modulo e una baseline per il sign-off.

Le correzioni saranno verificate su combinazioni reali dei quattro filtri disponibili (Periodo, Magazzino, Categoria, Stato Ordine), non solo su casi isolati. I gap di internazionalizzazione saranno catalogati e affrontati in questa fase, con particolare attenzione al contesto del magazzino di Tokyo.

**Assunzione:** La lista completa dei difetti sarà allineata con il team operativo di Meridian prima dell'avvio della remediation. Se Meridian dispone di un registro tracciato, ne chiediamo la condivisione al kickoff del progetto; altrimenti, l'output del nostro audit diventa la baseline concordata.

---

## R3 — Test automatizzati *(consegnati in anticipo)*

Sequenziamo deliberatamente R3 prima rispetto all'ordine della RFP. La copertura di test automatizzati è il prerequisito che sblocca l'approvazione IT per tutti i deployment successivi — incluse le correzioni di R1 e la nuova funzionalità di R2. Consegnarla tardi creerebbe un collo di bottiglia sull'intero ingaggio.

I test saranno scritti con Playwright, già configurato nel repository del progetto. La copertura riguarderà i flussi utente critici del modulo Reports (dopo R1) e della nuova vista Restocking (dopo R2). I casi limite — stati vuoti, combinazioni di filtri senza risultati, condizioni al limite del budget ceiling — saranno inclusi esplicitamente.

**Assunzione:** La definizione di "flussi critici" sarà confermata con il team IT di Meridian prima dell'authoring dei test. Forniremo una lista proposta di flussi per la loro revisione e approvazione.

---

## R2 — Suggerimenti di riordino

Consegneremo una nuova vista Restocking come modulo di primo livello nell'applicazione Vue esistente, coerente con i pattern di navigazione e visivi attuali. La vista accetterà tre input dall'operatore: livelli di stock correnti (dall'endpoint `/api/inventory` esistente), previsione della domanda (da `/api/demand`) e un tetto di budget inserito direttamente nell'interfaccia.

L'output sarà una lista prioritizzata di ordini d'acquisto raccomandati — articolo, quantità, costo stimato e motivazione — ordinata per urgenza operativa.

Per la logica di riordino: se Meridian fornisce soglie o algoritmi esistenti, li implementeremo fedelmente. In caso contrario, proporremo un modello basato sui giorni di copertura residui rispetto al lead time del fornitore, lo presenteremo per revisione e procederemo previa approvazione scritta. Il modello sarà documentato in modo che il team di Meridian possa mantenerlo e modificarlo dopo l'handoff.

**Assunzione:** I dati di stock e domanda sufficienti per guidare le raccomandazioni sono disponibili tramite gli endpoint API esistenti. Eventuali gap scoperti durante lo sviluppo saranno segnalati immediatamente per il riscontro di Meridian.

---

## R4 — Documentazione architetturale

Produrremo una panoramica dello stato attuale dell'architettura come documento HTML autonomo, distribuibile al team IT di Meridian senza richiedere strumenti speciali. Il documento coprirà:

- Componenti del sistema e loro ruoli (frontend Vue, backend FastAPI, data layer JSON)
- Flusso dei dati dai filtri UI attraverso l'API client, l'elaborazione backend e il ritorno alle computed properties
- Superficie API: endpoint, parametri e struttura delle risposte
- Limitazioni note e debito tecnico ereditato dal precedente fornitore

Il documento è scritto per generalisti IT, non per sviluppatori — diagrammi piuttosto che riferimenti al codice sorgente, linguaggio semplice piuttosto che gergo tecnico.

---

## D1 — Modernizzazione UI *(desiderato)*

Aggiorneremo il design visivo della dashboard per allinearlo agli standard contemporanei per le interfacce operative data-dense. Lo scope e il punto di riferimento dipendono dalla disponibilità da parte di Meridian di una brand guide o di un riferimento stilistico. In assenza di questi, proporremo una direzione per approvazione prima dell'implementazione.

---

## D2 — Internazionalizzazione *(desiderato)*

Il codebase esistente dispone di un'infrastruttura i18n parziale. La estenderemo per coprire i moduli rimanenti, con il personale del magazzino di Tokyo come destinatario principale. Le stringhe di traduzione saranno consegnate in un formato che il team di Meridian può mantenere senza coinvolgere sviluppatori.

---

## D3 — Dark mode *(desiderato)*

Il codebase utilizza CSS custom properties (design token) per il sistema dei colori, il che rende un toggle di tema un'estensione naturale piuttosto che un redesign completo. Implementeremo un tema chiaro/scuro selezionabile dall'operatore e persistito tra le sessioni. Questo risponde direttamente al requisito degli ambienti a bassa luminosità per le postazioni di piano magazzino.

---

## Assunzioni esplicite

| # | Assunzione |
|---|---|
| A1 | La lista completa dei difetti per R1 sarà fornita da Meridian o validata rispetto al nostro audit prima dell'avvio della remediation |
| A2 | I "flussi critici" per R3 saranno confermati con il team IT di Meridian prima dell'authoring dei test |
| A3 | Per R2, in assenza di un algoritmo Meridian, il vendor proporrà un modello basato sui giorni di copertura per approvazione scritta |
| A4 | Lo scope di D1 è condizionato alla disponibilità di una brand guide o riferimento stilistico da parte di Meridian |

---

*Presentato da: [Nome della vostra società]*
*Contatto: g.viscoli@accenture.com*
*Data: 27 aprile 2026*
