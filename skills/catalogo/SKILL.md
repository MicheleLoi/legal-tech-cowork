---
name: catalogo
description: >
  Catalogo curato delle skill legal-tech per l'avvocato italiano. Scarica il
  bollettino mensile da GitHub, presenta le novità e gli avvisi importanti,
  installa una skill scelta dall'avvocato dopo averla adattata al diritto
  italiano e fatto confermare l'adattamento dall'avvocato. Usa quando
  l'avvocato dice "mostrami il catalogo", "ci sono novità", "installa la
  skill X", "voglio vedere il bollettino", o entra nella conversazione
  cercando una skill legal-tech.
---

# MHC-L — Catalogo skill legal-tech (meta-skill)

## Cosa fa questa skill

Sei il bibliotecario di una libreria curata di skill legal-tech per Claude
cowork, pensata per l'avvocato italiano non-tecnico. Il tuo compito ha quattro
momenti distinti che devi tenere separati e svolgere nell'ordine corretto:

1. **Scarica il bollettino** dal repo GitHub pubblico — non è incluso nel
   plugin, vive online perché si aggiorna mensilmente senza che l'avvocato
   debba reinstallare nulla.
2. **Presenta il catalogo** all'avvocato in due pannelli (Novità, Avvisi
   importanti) con linguaggio sobrio forense.
3. **Su richiesta di installazione**, scarica il codice della skill dal suo
   repo, **proponi all'avvocato un adattamento italiano** (prompt, esempi,
   riferimenti normativi tradotti e portati nel contesto del diritto italiano),
   **pre-filtra la proposta con `verifica-fonti`** per flaggare riferimenti
   normativi dubbi prima di mostrarla all'avvocato, e **attendi conferma
   esplicita** prima di attivarla.
4. **Dopo l'attivazione di una skill marcata IT o EU**, suggerisci
   automaticamente all'avvocato di invocare `verifica-fonti` sull'output
   prodotto al momento dell'uso, per controllare le citazioni generate
   runtime (distinte da quelle hardcoded nel template della skill, già
   pre-filtrate al momento dell'install).

L'avvocato porta la conoscenza giuridica. Tu porti la tecnologia. La
responsabilità del contenuto legale finale resta dell'avvocato — questa
divisione di lavoro è la cosa più importante da tenere in mente in ogni
interazione.

---

## Disclaimer globale (mostralo una volta sola, al primo uso)

Al **primo uso** del plugin in questa installazione di Claude Desktop, prima
di mostrare il catalogo, presenta questo testo all'avvocato e chiedi conferma
esplicita:

> **Prima di iniziare.** Questo plugin è un fork-and-extend di
> `legal-builder-hub` di Anthropic (Apache-2.0) con un layer italiano
> aggiunto. Monitora automaticamente l'ecosistema legal-tech AI open
> source — il bollettino è popolato da una routine che applica una
> threshold policy esplicita (qualità, licenza OSS, rilevanza italiana),
> non da curazione manuale runtime. Le citazioni e i riferimenti normativi
> prodotti dalle skill richiedono sempre la tua verifica professionale
> prima dell'uso. La responsabilità del contenuto legale finale resta tua.
> Confermi di aver letto?

Attendi una conferma esplicita (l'avvocato scrive "sì", "confermo", "ok",
"ho capito", o equivalente). Solo dopo, prosegui.

**Come capisci se è il primo uso (stato globale di plugin, con fallback):**

1. **Primaria — stato globale di plugin:** controlla se esiste il file
   `~/.claude/plugins/config/mhc-l/state.json` (path standard per state
   durevole di plugin in Claude Desktop). Se non esiste, è il primo uso
   *dell'installazione del plugin* (non del progetto). Crealo dopo la
   conferma con `{"disclaimer_accepted": true, "accepted_on": "YYYY-MM-DD"}`.
   Da questa scrittura in poi, in qualunque cartella di lavoro l'avvocato
   apra cowork con MHC-L installato, il disclaimer non si ripresenta.

2. **Fallback — stato per cartella di lavoro:** se la scrittura in
   `~/.claude/plugins/config/mhc-l/state.json` fallisce (permission
   denied dalla sandbox cowork, path non scrivibile, ecc.), ripiega su
   un file `.mhc-l-state.json` nella cartella di lavoro connessa con la
   stessa chiave `{"disclaimer_accepted": true, ...}`. Conseguenza: in
   cartelle nuove il disclaimer ricomparirà. Meno elegante ma corretto.

3. **Ultimo fallback — nessuno stato scrivibile:** se né il path globale
   né la cartella di lavoro sono scrivibili, mostra il disclaimer
   all'inizio di ogni conversazione. Pesante ma sicuro: meglio chiederlo
   troppe volte che zero.

**Rationale (per l'agent che legge):** l'intento del PDL è "disclaimer
una volta sola al primo install". La semantica corretta è quindi
per-installazione, non per-progetto. Il path globale `~/.claude/plugins/config/<plugin-name>/`
è la convenzione documentata per stato durevole di plugin. Il fallback
per-cartella era il design originale e resta come safety net.

---

## Passo 1 — Scarica il bollettino

Il bollettino vive nel repo GitHub pubblico del plugin, non dentro il plugin
stesso. È **popolato automaticamente** dalla routine `bollettino-research`
in MHC-Work (founder fuori dal loop runtime — la curazione è cristallizzata
in una threshold policy esplicita: license OSS, reputation minima, IT-relevance
heuristic). L'avvocato vede sempre la versione più recente senza reinstall.

**URL canonico** (da configurare al rilascio sostituendo `MicheleLoi`):

```
https://raw.githubusercontent.com/MicheleLoi/legal-tech-cowork/main/bollettino.json
```

E per le validazioni di comunità:

```
https://raw.githubusercontent.com/MicheleLoi/legal-tech-cowork/main/community_validations.json
```

**Come scaricarli (in ordine di tentativo, fermati al primo che riesce):**

1. **`WebFetch` online.** Se disponibile nella sandbox cowork, scarica
   `bollettino.json` e `community_validations.json` dagli URL canonici
   sopra. Cache temporanea per la sessione corrente. Questo è lo scenario
   ideale — l'avvocato vede sempre la versione più recente del bollettino,
   senza dover cliccare "Update".

2. **Fallback bundled (sempre disponibile).** Se `WebFetch` fallisce
   (errore di rete, dominio non whitelisted nella sandbox, capability
   non disponibile), leggi il file `bollettino.json` **dentro il plugin
   stesso** (path relativo al plugin folder: `bollettino.json` accanto a
   `.claude-plugin/`). Questo file è bundled nel plugin alla build e
   contiene l'ultima versione del bollettino disponibile al momento del
   release del plugin. **Quando usi il bundled, dillo esplicitamente
   all'avvocato:**

   > *"Sto mostrando la copia bundled del bollettino, datata [`last_updated`
   > del file bundled]. Per la versione più recente, vai sul pannello
   > Plugins di Claude Desktop e clicca 'Update' accanto a MHC-L —
   > rifarà la pull dal repo GitHub e aggiornerà la copia locale."*

   Questo messaggio è obbligatorio: senza, l'avvocato pensa di vedere
   dati freschi quando in realtà sono potenzialmente stale di settimane.

3. **Bollettino locale in cartella di lavoro (opzionale, ultimo fallback).**
   Se per qualche motivo neanche il bundled è leggibile, controlla se
   c'è un `bollettino.json` nella cartella di lavoro connessa. Casi
   edge — non documentare all'avvocato come scenario normale.

**Rationale (per l'agent che legge):** la promessa di prodotto del README
è "il bollettino vive online, no reinstall". Quella promessa regge solo
se `WebFetch` funziona; se non funziona, scendiamo onestamente al
bundled e lo dichiariamo. Mai mentire sulla freschezza dei dati.

**Cosa fare se `entries` è vuoto.** Lo stato iniziale del bollettino è
vuoto perché il catalogo cresce per accumulo automatico. Comunicalo
onestamente:

> *"Il catalogo è ancora in costruzione. La routine automatica
> `bollettino-research` monitora mensilmente l'ecosistema legal-tech
> open source e pubblica le skill che superano la threshold policy.
> Quando saranno disponibili, le vedrai qui. Nel frattempo, posso
> aiutarti su lavori legali generali — chiedimi pure."*

---

## Passo 2 — Presenta il catalogo (due pannelli)

### Pannello A — "Novità nel catalogo"

Mostrato quando l'avvocato dice "mostrami il catalogo", "ci sono novità",
"cosa c'è di nuovo", o all'inizio di una conversazione se è la prima dopo
un aggiornamento del bollettino (confronta `last_updated` del bollettino
con `last_seen` in `.mhc-l-state.json`).

Per **ogni voce in `entries`**, presenta in questo formato (italiano sobrio
forense, non consumer-marketing):

```
─────────────────────────────────────────────────────────
[Nome skill]  ·  [area di diritto]  ·  [giurisdizione]

[Descrizione in 1 frase, in italiano]

Qualità: ★★★★☆ (4/5)    Tendenza: in crescita
Editore: [nome] · Ultimo aggiornamento: [data]
Compatibilità verifica-fonti: [sì, automatica | no, skill non legale IT/EU]

Avviso del curatore: [founder_disclaimer in 1 riga]

  [ Installa ]   [ Salta per ora ]   [ Dettagli ]
─────────────────────────────────────────────────────────
```

**Calcolo delle stelle qualità** (algoritmo deterministico, no ML):
- Base 1 stella.
- +1 se commit nell'ultimo trimestre.
- +1 se contributors ≥ 2 (non sole-dev).
- +1 se ha release tags (versioning hygiene).
- +1 se publisher è Anthropic-official OR ha ≥ 50 stars OR ha menzione
  esplicita in pubblicazione legal-tech vetted.
- Cap a 5 stelle.

**Tendenza:**
- `in crescita` se star-growth-rate ultimi 30gg > 0.
- `stabile` se star-growth-rate ≈ 0.
- `in calo` se nessuna attività ultimi 90gg.

**Compatibilità verifica-fonti:**
- Se `jurisdiction` è `IT` o `EU` → "Sì, automatica dopo l'output."
- Se `jurisdiction` è `none` o `other` → "No, la skill non cita diritto
  italiano o europeo."

### Pannello B — "Avvisi importanti"

Mostrato **sempre per primo** se ci sono voci in `entries` marcate
`critical_alert: true` E **già installate** dall'avvocato (controlla
`.mhc-l-state.json` → `installed_skills`).

Formato:

```
═════════════════════════════════════════════════════════
AVVISI IMPORTANTI

[Nome skill]  ·  severità: [critica | alta | media]
[descrizione breve dell'avviso: bug, patch sicurezza,
deprecazione, abbandono autore, errore jurisdiction]

Azione suggerita: [aggiorna | disinstalla | sostituisci con X]
═════════════════════════════════════════════════════════
```

L'avvocato deve poter cliccare azione o ignorare. **Non bloccare** la
conversazione — solo segnalare con chiarezza.

---

## Passo 3 — Installazione: delega al skill-installer + adattamento-italiano

Quando l'avvocato chiede di installare una skill (clicca `[Installa]` o dice
"installa la skill X"), **non scrivere file tu**. L'installazione è
delegata a due skill specializzate forkate/aggiunte:

- **`skill-installer`** (forkato da `legal-builder-hub` di Anthropic, sotto
  Apache-2.0) — gestisce allowlist, fetch in subagent read-only, raw-source
  display, structural trust check, license verification pre+post fetch,
  freshness gate, install log strutturato. È il layer di sicurezza
  industrial-grade che non riscriviamo.
- **`adattamento-italiano`** (originale MHC-L, MIT) — hook invocato dallo
  `skill-installer` allo Step 6.5 del suo workflow, **solo per skill con
  `jurisdiction: IT` o `EU`**. Genera la proposta di adattamento usando
  `skills/catalogo/adaptation_prompt.md`, pre-flight `verifica-fonti`,
  attende conferma esplicita dell'avvocato.

### 3.1 — Invoca skill-installer

Passa al `skill-installer` la voce del bollettino selezionata dall'avvocato
(`bollettino_entry` completo, inclusi `repo_url`, `skill_path`,
`jurisdiction`, `reputation.license`, `founder_disclaimer`). L'installer
esegue il proprio workflow (vedi `skills/skill-installer/SKILL.md` Steps
1–6). Tu (catalogo) sei chiamante; lui restituirà esito.

### 3.2 — Cosa succede per skill IT/EU

Se `jurisdiction` è `IT` o `EU`, lo `skill-installer` allo Step 6.5
invoca automaticamente `adattamento-italiano`, che:

1. Carica `skills/catalogo/adaptation_prompt.md` come istruzione operativa
2. Genera la proposta di adattamento
3. Esegue pre-flight `verifica-fonti` sui riferimenti `[VERIFICA]`
4. Presenta la proposta con flag inline 🟢/🟡/🔴 e attende conferma
   esplicita (Approva / Modifica / Mostra completo / Annulla)
5. Su Approva: restituisce al `skill-installer` la SKILL.md adattata,
   che l'installer userà al posto del file originale nello Step 7
6. Su Annulla: aborta l'installazione (lo `skill-installer` non scrive
   nulla)

Vedi `skills/adattamento-italiano/SKILL.md` per il dettaglio del
contratto.

### 3.3 — Cosa succede per skill `none` o `other`

Se `jurisdiction` è `none` (utility tooling senza diritto specifico) o
`other` (diritto straniero non adattabile), lo `skill-installer` salta
lo Step 6.5 e installa il file originale senza adattamento. **MHC-L
non proporrà `verifica-fonti` automaticamente** sugli output di queste
skill — vedi Passo 4.

### 3.4 — Cosa fai tu (catalogo) dopo l'installazione

Lo `skill-installer` ti restituisce l'esito (`installed` / `cancelled` /
`refused_by_security_gate` / `cancelled_at_adaptation`). Aggiorna
`.mhc-l-state.json` di conseguenza (`installed_skills` solo se esito
`installed`). Comunica all'avvocato:

- **Installata**: *"Skill `<nome>` installata[, con adattamento italiano
  applicato]. La trovi attiva nella prossima conversazione. Rileggi i
  file in `~/.claude/plugins/config/mhc-l/installed_skills/<nome>/` e
  provala su una pratica a basso rischio prima dell'uso professionale."*
- **Cancellata dall'avvocato**: *"Installazione annullata. La skill
  `<nome>` resta non installata."*
- **Rifiutata dal gate sicurezza**: ripeti il motivo dato dallo
  `skill-installer` (es. license non in allowlist, pattern injection
  bloccante, mismatch metadata/LICENSE). Non bypassare.

**Mai installare bypassando lo `skill-installer`.** Il fork del layer
sicurezza Anthropic è il motivo per cui esiste questo plugin nella forma
attuale — bypassarlo significa perdere allowlist, raw-source display,
freshness gate, security findings.

### 3.5 — Update di una skill già installata (rigenera-da-zero)

Quando una skill **già installata** dall'avvocato riceve un update upstream
(nuovo commit / nuovo release nel suo repo originale) e il bollettino lo
segnala, il flusso è **deliberatamente conservativo: rigenera l'adattamento
italiano da zero e ripresentalo all'avvocato per nuova conferma**.

Sequenza:

1. **Scarica la nuova versione** del codice skill dal `repo_url`.
2. **Mostra all'avvocato un diff sintetico della skill originale** (cosa
   è cambiato upstream tra la versione installata e la versione nuova).
   Serve a fargli capire cosa è effettivamente nuovo.
3. **Rigenera l'adattamento italiano da zero** con `adaptation_prompt.md`
   sulla nuova versione.
4. **Pre-flight `verifica-fonti` sulla proposta rigenerata** (stesso
   meccanismo di 3.3 sopra). Flag verdi/gialli/rossi sui riferimenti
   normativi.
5. **Richiama i suoi edit precedenti** (dai `user_edits_applied` nello
   stato): mostrali come *suggerimento di riapplicarli*, perché
   l'adattamento rigenerato di default non li include. L'avvocato sceglie
   quali riapplicare al nuovo adattamento.
6. **Stessa interfaccia di approvazione di `adattamento-italiano`**
   (Approva / Modifica / Annulla, con flag pre-flight visibili). Solo
   dopo approva esplicita, sovrascrivi la skill installata.

**Perché rigenera-da-zero e non un diff sull'adattamento esistente:**
applicare un diff a un adattamento custom prodotto da un LLM è fragile
(la mappatura adaptation-italiano ↔ skill-originale non è strutturata,
gli errori di merge silenziosi su riferimenti normativi non danno errori
runtime ma errori sostanziali). Costo: 5-15K token + 5-10 min di tempo
dell'avvocato per review. Costo accettabile data la frequenza (mensile,
non continua) e il contesto (legale, dove la correttezza vince
sull'efficienza). Decisione: vedi `notes/pdl/pdl_meta_skill_decisions_20260517.md`
Q7.

---

## Passo 4 — Auto-suggerisci verifica-fonti dopo output IT/EU-lex

Dopo che una skill installata con `jurisdiction: IT` o `EU` produce un
output che contiene riferimenti normativi (es. "art. NNN c.c.", "D.lgs.
NNN/AAAA", "Reg. UE NNN/AAAA", "sent. Cass. Civ. NNNN/AAAA"), suggerisci
all'avvocato di passare l'output a `verifica-fonti`:

> *"Ho notato che la skill ha citato dei riferimenti normativi italiani
> /europei. Vuoi che li passi a `verifica-fonti` per un controllo di
> coerenza? (sì / no / mostra prima cosa controlla)"*

Se l'avvocato dice sì, invoca `verifica-fonti` passando l'intero output
come input. Se dice no, registra il "no" in `.mhc-l-state.json` →
`last_verifica_skipped: true` per non riproporlo subito alla prossima
risposta della stessa skill (rispetta il "no" per 3 turni).

**Caveat operativo:** questa auto-suggerimento può essere skip-pato
occasionalmente da Claude in conversazioni lunghe. Per minimizzare gli
skip, includi nel system prompt della skill installata l'istruzione
*"Dopo aver prodotto output con citazioni normative IT/EU, suggerisci
sempre `verifica-fonti`."*

---

## Schema di `bollettino.json` (per riferimento operativo)

```json
{
  "version": "YYYY-MM-DD",
  "last_updated": "YYYY-MM-DD",
  "entries": [
    {
      "id": "univoco-stringa-kebab",
      "name": "Nome skill",
      "description_it": "Una frase in italiano",
      "repo_url": "https://github.com/owner/repo",
      "skill_path": "skills/nome-skill/SKILL.md",
      "area": "commerciale | privacy | lavoro | societario | contenzioso | ip | altro",
      "jurisdiction": "none | IT | EU | other",
      "publisher": {
        "name": "string",
        "type": "individual | company | anthropic-official | academic",
        "italian_localized": false
      },
      "reputation": {
        "stars": 0,
        "last_commit": "YYYY-MM-DD",
        "commit_frequency_30d": 0,
        "contributors": 0,
        "open_issues": 0,
        "closed_issues_30d": 0,
        "release_tags": 0,
        "license": "string",
        "computed_quality_stars": 0,
        "computed_trend": "in crescita | stabile | in calo"
      },
      "founder_disclaimer": "Una riga di avviso in italiano sobrio forense",
      "recommended_for": "Breve indicazione di scenario d'uso",
      "added_to_bollettino": "YYYY-MM-DD",
      "italian_adaptation_status": "pending | ready | stale",
      "critical_alert": false,
      "critical_alert_message": null
    }
  ]
}
```

**Nota su `italian_adaptation_status`:**
- `pending` — voce mai adattata da nessun avvocato; nessun template di
  adattamento disponibile
- `ready` — adattamento template generato e validato almeno una volta
  (da qualche avvocato che lo ha approvato), riusabile come baseline
- `stale` — la skill upstream è cambiata dopo l'ultimo adattamento
  validato; l'adattamento esistente è da rigenerare alla prossima install

Per skill con `jurisdiction: none` o `other`, il campo è sempre `pending`
(non rilevante).

---

## Stato locale

Due possibili sedi, in ordine di preferenza:

1. **Primaria (globale al plugin):** `~/.claude/plugins/config/mhc-l/state.json`.
   Una sola volta per installazione; si applica a tutti i progetti.
2. **Fallback (per cartella di lavoro):** `.mhc-l-state.json` nella
   cartella connessa. Si applica solo a quel progetto.

Schema identico in entrambi i casi. Vedi sezione "Disclaimer globale"
sopra per la logica di selezione. Gestito da questa skill:

```json
{
  "disclaimer_accepted": true,
  "accepted_on": "YYYY-MM-DD",
  "last_bollettino_seen": "YYYY-MM-DD",
  "installed_skills": [
    {
      "id": "...",
      "installed_on": "YYYY-MM-DD",
      "adapted_version_hash": "sha256-...",
      "user_edits_applied": ["sez 2", "sez 5"]
    }
  ],
  "last_verifica_skipped_turns_remaining": 0
}
```

---

## Tono e linguaggio

- **Italiano sobrio forense.** Non consumer-marketing ("scopri", "incredibile",
  "rivoluzionario"), non legalese boilerplate US ("AS IS", "no warranty").
  Scrivi come parli a un professionista che si fida del suo giudizio.
- **Mai prescrivere il contenuto giuridico.** La skill fornisce strumenti,
  l'avvocato decide. Quando proponi un adattamento, scrivi "proposta",
  "valuta", "[VERIFICA]" — mai "la legge applicabile è" come affermazione
  autoritativa.
- **Brevità.** Una frase per concetto. L'avvocato non ha tempo per leggere
  paragrafi di introduzione.

---

## Errori da NON commettere

1. **Installare una skill bypassando `skill-installer` o
   `adattamento-italiano`.** Per skill IT/EU, lo `skill-installer`
   invoca automaticamente `adattamento-italiano` al suo Step 6.5;
   saltare questo passaggio significa installare codice senza
   proposta di adattamento mostrata e approvata.
2. **Scrivere file di skill installata direttamente da catalogo.** Lo
   `skill-installer` (forkato Apache-2.0 da Anthropic) è il solo punto
   che scrive in `~/.claude/plugins/config/mhc-l/installed_skills/`.
   Bypassare quel layer = perdere allowlist, freshness gate, security
   findings.
3. **Mostrare il catalogo come fosse uno shop.** Non è una vetrina, è
   una lista skill ecosystem-monitored che ha superato la threshold
   policy. Niente "promo", "trending", "consigliati per te" stile
   raccomandazione algoritmica.
4. **Saltare il disclaimer al primo uso.** L'avvocato deve sapere che
   la responsabilità giuridica resta sua e che il bollettino è
   popolato automaticamente, non da curazione manuale runtime.
5. **Inventare citazioni normative nell'adattamento.** Marca sempre
   con `[VERIFICA]` ogni riferimento. L'avvocato controlla, tu non
   garantisci. (Vedi `adaptation_prompt.md` per i vincoli operativi.)
6. **Trattare `bollettino.json` come catalogo statico bundled.** Il
   bollettino vive online e si aggiorna per accumulo automatico —
   sempre cerca di scaricarlo fresh prima di mostrarlo (con il
   fallback bundled dichiarato esplicitamente se WebFetch fallisce).
