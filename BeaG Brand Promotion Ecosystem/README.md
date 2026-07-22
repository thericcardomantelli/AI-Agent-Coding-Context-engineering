# Studio Creativo — Campagna BeaG Collection

Sistema multi-agente costruito su **Claude Code** che trasforma un brief di brand (brandbook + logo) in una presentazione di storytelling visivo pronta da mostrare, senza intervento manuale sul design.

Il progetto è stato usato per generare la campagna di lancio di **BeaG collection**, brand di sportswear giovane e inclusivo fondato da Beatrice Giussani, incentrato sull'*adaptive fashion*: capi esteticamente identici nella linea tradizionale e in quella facilitata, così che nessuno si senta escluso.

## Note preliminari

1. A caso dello stato attuale dell'AI, abbiamo riscontrato delle difficoltà nella creazione delle immagini richieste. Abbiamo dovuto fare delle inegrazioni per cercare di arrivare ad un risultato accettabile.
2. I commandi non sono stati accolti o capiti correttamente e piu volte abbiamo dovuto generare dei prompt aggiuntivi. Per esempio la generazione dello slideshow per l'output finale del progetto.
3. Durante il progetto abbiamo imparato le limitazioni dell'AI e capito dove ci vuole un input umano per migliorare il risultato.
4. Magari fornendo un documento apposito sul dna del brand, si potrebbe indirizzare l'agente direttamente sul argomento ed avere dei risultati migliori. Abbiamo fornito il lookbook ma senza identificare cosa intendevamo come il dna del brand.

## Cosa produce

Dato in input il brandbook (`brandbook.md`) e il logo (`logo.jpeg`), lo studio genera:

- **`slideshow.html`** — il moodboard/brainstorming finale in stile Lotus Blossom: idea centrale, 6 rami di espansione, palette colori cliccabile, coppia tipografica, texture e materiali, nota di art direction, e 22 prompt Midjourney pronti all'uso (uno per slide, collassati in pannelli discreti).
- **`processo_creativo.html`** — documento separato che conserva tutto il materiale scartato lungo il percorso (idee, direzioni, moodboard non scelti), per rendere trasparente il processo.

Entrambi sono file HTML autonomi (CSS e JS inline, nessuna dipendenza esterna, zero CDN), apribili direttamente nel browser.

## Architettura: chi fa cosa

Un coordinatore (Claude, il "direttore creativo") delega il lavoro a 4 agenti specializzati, definiti in `.claude/agents/`:

| Agente | Ruolo | Tool |
|---|---|---|
| **ideatore** | Genera 5-10 idee divergenti, raggruppate in 3 concept. Qualità prima di quantità, nessuna auto-censura. | `read`, `write` |
| **esploratore** | Ricerca trend e referenze visive coerenti con le idee dell'ideatore, produce 3 direzioni di ispirazione distinte. | `WebSearch`, `WebFetch`, `Read`, `Write` |
| **curatore-visivo** | Traduce la direzione scelta nello slideshow finale (Lotus Blossom + moodboard) e nel documento di processo. | `Read`, `Write`, `WebSearch` |
| **critico** | Ha sempre l'ultima parola. Valuta ogni output su coerenza col brand DNA, originalità, fattibilità e forza visiva (voti 0-10), impone UNA direzione, scrive i prompt Midjourney finali. Nessun output esce senza il suo via libera. | `Read` |

Le regole di comportamento comuni a tutto lo studio (lingua italiana, tono diretto, niente gergo tecnico, niente emoticon) sono definite in `CLAUDE.md`.

## Come funziona il workflow

Il comando `/studio` (definito in `.claude/commands/studio.md`) orchestra la catena in 5 fasi, riportando l'esito di ognuna prima di passare alla successiva:

1. **Ideazione** — l'ideatore genera i concept divergenti a partire dal brief.
2. **Esplorazione** — l'esploratore cerca referenze e trend sul web, li abbina alle idee.
3. **Prima critica** — il critico valuta i concept proposti e impone LA direzione da seguire, con correzioni concrete.
4. **Curatela visiva** — il curatore-visivo trasforma la direzione approvata in `slideshow.html`.
5. **Critica finale** — il critico riesamina lo slideshow. Se boccia, si torna al punto 4 con le correzioni; se approva, il progetto è chiuso.

Se il brief iniziale è vago, il coordinatore fa al massimo 3 domande prima di partire.

## Come eseguirlo

### Prerequisiti
- [Claude Code](https://docs.claude.com/en/docs/claude-code) installato e configurato.
- I file di input del brand presenti in `.claude/visual design/`:
  - `brandbook.md` (identità, tono, storia, prodotti)
  - `logo.jpeg`

### Passi

1. **Apri il progetto in Claude Code** dalla cartella `studio_creativo/` (deve contenere `CLAUDE.md` alla radice e la cartella `.claude/`).
2. **Verifica l'input**: assicurati che `brandbook.md` e `logo.jpeg` siano aggiornati nella cartella `.claude/visual design/`. Le linee guida vincolanti sui prompt immagine sono in `.claude/agents/prompt.md`: in caso di conflitto con il brandbook, prevalgono queste ultime.
3. **Lancia il comando** nella chat di Claude Code:
   ```
   /studio
   ```
4. **Rispondi alle eventuali domande** iniziali su pubblico, tono e vincoli, se il coordinatore le pone (brief vago).
5. **Segui l'avanzamento delle 5 fasi**: il coordinatore riporta in breve l'esito di ideazione, esplorazione, prima critica, curatela visiva e critica finale, delegando ogni volta all'agente giusto tramite il tool Task.
6. **Se il critico boccia la curatela** in fase 5, il coordinatore torna automaticamente al punto 4 con le correzioni indicate: nessuna azione manuale richiesta.
7. **A fine esecuzione**, il coordinatore apre `slideshow.html` nel browser e riassume in tre righe la direzione scelta.
8. **Consulta `processo_creativo.html`** per rivedere idee e direzioni scartate, con la motivazione dello scarto.

### Rieseguire o iterare
Per una nuova campagna o un nuovo brand, basta sostituire `brandbook.md` e `logo.jpeg` in `.claude/visual design/` e rilanciare `/studio`. Gli agenti e le regole restano invariati.

## Struttura del repository

```
studio_creativo/
├── CLAUDE.md                              # regole generali dello studio
├── .claude/
│   ├── commands/studio.md                 # orchestrazione del workflow (/studio)
│   ├── agents/
│   │   ├── ideatore.md
│   │   ├── esploratore.md
│   │   ├── curatore-visivo.md
│   │   ├── critico.md
│   │   └── prompt.md                      # linee guida vincolanti per i prompt immagine
│   └── visual design/
│       ├── brandbook.md                   # input: identità del brand BeaG
│       └── logo.jpeg                      # input: logo del brand
├── creative_brief_prompt.md                # brief di partenza
├── ideazione.md                            # output grezzo dell'ideatore
├── esplorazione.md                         # output grezzo dell'esploratore
├── critica_1.md                            # prima critica e direzione imposta
├── critica_finale.md                       # critica finale e approvazione
├── slideshow.html                          # OUTPUT FINALE: moodboard/presentazione brand
└── processo_creativo.html                  # OUTPUT: documento del processo e degli scarti
```

## Risultato: la direzione scelta

Il claim approvato per BeaG collection è **"il gesto che basta"**, con l'accento cromatico cobalto (`#2C3A8C`) come firma visiva ricorrente su tutto il documento e attenzione specifica alla rappresentazione corretta delle mani e del corpo nelle immagini generate, coerentemente con il posizionamento adaptive fashion del brand.
