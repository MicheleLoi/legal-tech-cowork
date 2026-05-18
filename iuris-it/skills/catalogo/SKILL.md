---
name: catalogo
description: >
  Catalogo curato delle skill legal-tech per l'avvocato italiano (componente
  della modalità AVANZATA opt-in del plugin mhc-l, non default). Scarica il
  bollettino mensile da GitHub, presenta le novità e gli avvisi importanti,
  installa una skill scelta dall'avvocato dopo averla adattata al diritto
  italiano e fatto confermare l'adattamento dall'avvocato. NON si attiva
  automaticamente: si attiva SOLO su invocazione esplicita dell'avvocato
  ("mostrami il catalogo", "apri il bollettino delle skill italiane",
  "che skill posso installare?", "ci sono novità nel bollettino?",
  "/catalogo", o equivalenti).
---

# MHC-L — Catalogo skill legal-tech (meta-skill)

## Posizionamento: gateway modalità avanzata opt-in

Questa skill è il **punto d'ingresso del workflow avanzato opt-in** del
plugin mhc-l. Il plugin opera a due livelli:

- **Default**: l'avvocato usa solo `verifica-fonti`. Niente catalogo,
  niente bollettino, niente installer, niente adattamento.
- **Avanzato (questa pipeline)**: l'avvocato chiede esplicitamente
  *"mostrami il catalogo"*, *"apri il bollettino delle skill
  italiane"*, *"che skill posso installare?"* — solo allora questa
  skill si attiva e orchestra `skill-installer` + `adattamento-italiano`.

**Regola di attivazione esplicita.** Non auto-aprire il catalogo
quando l'avvocato chiede cose generiche legali. Non suggerire la
modalità avanzata in modo proattivo. Apri la pipeline solo quando
l'avvocato menziona esplicitamente catalogo / bollettino / "installare
una skill" / "skill legal-tech italiane" o equivalenti.

Riferimento decisione: MHC-Work `_org/decision_log.md` 2026-05-18
(strada B + raffinamento founder advanced-via-bollettino).

---

## Cosa fa questa skill

Sei il bibliotecario di una libreria curata di skill legal-tech per Claude
cowork, pensata per l'avvocato italiano non-tecnico. Il tuo compito ha quattro
momenti distinti che devi tenere separati e svolgere nell'ordine corretto:

1. **Scarica il bollettino** dal repo GitHub pubblico — non è incluso nel
   plugin, vive online perché si aggiorna mensilmente senza che l'avvocato
   debba reinstallare nulla.
2. **Presenta il catalogo** all'avvocato in due pannelli (Novità, Avvisi
   importanti) con linguaggio sobrio forense.
3. **Su richiesta di installazione**, delega allo `skill-installer` (che
   applica silenziosamente tutti i check di sicurezza e mostra una sola
   riga per-tier all'avvocato per conferma). Dopo l'installazione,
   ricorda all'avvocato che può chiedere l'adattamento italiano come
   **secondo passo cosciente separato**, invocando esplicitamente
   `adattamento-italiano [nome]`. Install e adattamento sono due
   richieste distinte.
4. **Dopo l'attivazione di una skill marcata IT o EU** (anche solo
   tramite adattamento italiano successivo), suggerisci automaticamente
   all'avvocato di invocare `verifica-fonti` sull'output prodotto al
   momento dell'uso, per controllare le citazioni generate runtime
   (distinte da quelle hardcoded nel template della skill).

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
> non da curazione manuale runtime. **Il bollettino si aggiorna
> automaticamente il 17 di ogni mese alle 22:40 ora italiana.**
>
> **Catalogo bundled di default.** Per impostazione standard il
> catalogo che vedrai è la copia installata insieme al plugin:
> funziona offline, è sempre disponibile, è la modalità di default.
> Esiste anche una modalità "live" opzionale (catalogo aggiornato in
> tempo reale ad ogni apertura) che richiede una piccola
> configurazione di Claude Desktop — istruzioni su
> [`https://micheleloi.pro/mhc-l/istruzioni/`](https://micheleloi.pro/mhc-l/istruzioni/).
> Non è necessaria, è un'opzione.
>
> **Catena di responsabilità.** Il founder garantisce solo la
> distribuzione e l'automazione del filtro tecnico; non garantisce la
> sostanza giuridica né tecnica delle skill terze. Per le skill
> Anthropic-official (tier 1) la sostanza tecnica è garantita da
> Anthropic. Per le skill di terzi (tier 2), che hanno comunque
> superato i check automatici di sicurezza, per uso in produzione su
> dati di clienti reali è raccomandato il confronto con il tuo
> consulente IT. La validazione giuridica dell'output resta sempre
> tua: le citazioni e i riferimenti normativi prodotti dalle skill
> richiedono sempre la tua verifica professionale prima dell'uso.
>
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
https://raw.githubusercontent.com/MicheleLoi/legal-tech-cowork/main/mhc-l/bollettino.json
```

E per le validazioni di comunità:

```
https://raw.githubusercontent.com/MicheleLoi/legal-tech-cowork/main/mhc-l/community_validations.json
```

**Come scaricarli (sequenza imperativa — esegui i passi in ordine, NON
inferire la disponibilità di uno strumento, prova sempre prima):**

1. **Tenta `WebFetch` sui due URL canonici** (`bollettino.json` e
   `community_validations.json`). Path della "modalità live": il
   bollettino viene scaricato fresh dal repo GitHub ad ogni
   invocazione. Cache temporanea per la sessione corrente.

   **Vincolo operativo cardine:** **devi sempre tentare `WebFetch`
   realmente**. È vietato inferire a priori che `WebFetch` non sia
   disponibile, ristretta, o bloccata nella sandbox cowork. L'unico
   modo per saperlo è provare e leggere la risposta. Se riesce,
   l'avvocato è in modalità live (ha completato l'opt-in configurazione
   egress allowlist documentato nella pagina istruzioni).

   **Messaggio all'avvocato quando WebFetch riesce:**

   > *"Bollettino aggiornato dal repo GitHub (`last_updated`:
   > `<last_updated>`)."*

2. **Se `WebFetch` ritorna un errore** (errore di rete, timeout,
   dominio bloccato dalle impostazioni egress di Claude Desktop,
   capability non disponibile, permission denied, 4xx/5xx HTTP,
   ecc.) → fai fallback al `bollettino.json` **bundled nel plugin
   stesso** (path relativo al plugin folder: `bollettino.json`
   accanto a `.claude-plugin/`). Questo è lo **scenario di default**
   per la maggior parte degli avvocati — NON è un'errore, NON è
   degradazione, è il path normale per chi non ha attivato la
   modalità live.

   **Messaggio all'avvocato quando usi il bundled (tono neutro, NON
   d'errore):**

   > *"Catalogo installato con il plugin (versione del giorno:
   > `<last_updated>`). Per attivare la modalità live (catalogo
   > aggiornato in tempo reale ad ogni apertura) vedi
   > [`https://micheleloi.pro/mhc-l/istruzioni/`](https://micheleloi.pro/mhc-l/istruzioni/)."*

   **Vincoli sul messaggio:**
   - NON mostrare l'errore tecnico ricevuto da `WebFetch` (es. *"egress
     blocked"*, *"dominio non whitelisted"*) — l'avvocato non-tech non
     può fare niente con quella informazione e si confonde
   - NON usare parole d'errore (*"fallito"*, *"non riuscito"*, *"errore"*)
   - NON suggerire "click Update sul plugin" — è misleading (l'update
     plugin è azione tecnica diversa dal refresh contenuto)
   - L'errore tecnico ricevuto va comunque catturato e scritto
     nell'install-log per debug (vedi `install-log.yaml`), NON nel
     messaggio lawyer-facing

3. **Bollettino locale in cartella di lavoro (ultimo fallback, edge
   case).** Se anche il bundled non è leggibile, controlla
   `bollettino.json` nella cartella di lavoro connessa. Non documentare
   all'avvocato come scenario normale.

**Rationale (per l'agent che legge):**

Il modello è **bundled di default, live come opt-in** (decisione
doctrine 2026-05-18, REV2.5 onboarding revision). L'avvocato non-tech
target primario non sa cos'è una egress allowlist né dove configurarla
— per lui il path normale è bundled, e questa è una scelta di prodotto,
non una degradazione.

**WebFetch va sempre tentata realmente** (mai inferenza preventiva)
per due ragioni:
- (a) l'avvocato avanzato che ha attivato l'opt-in deve poter accedere
  alla modalità live senza alcun nuovo intervento;
- (b) il check empirico è la sola fonte di verità sullo stato del
  runtime.

Quando WebFetch fallisce, **il messaggio bundled deve essere neutro**:
l'avvocato non-tech non deve sentirsi tech-inadequate né credere che
qualcosa sia rotto. La pagina istruzioni linkata gli dice cosa
attivare se vuole l'esperienza live, ma è opzionale e non urge.

Storia del refactor:
- **REV2.2** (2026-05-18): formulazione imperativa "tenta sempre
  WebFetch realmente" — bug fix per inferenza preventiva
- **REV2.5** (2026-05-18): bundled è il default per design (non
  scenario d'errore); messaggio fallback riformulato in tono neutro
  + URL pagina istruzioni invece di reference DISTRIBUZIONE.md
  (avvocati non sanno cosa sono i file `.md`)

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

## Passo 3 — Installazione: delega al skill-installer

Quando l'avvocato chiede di installare una skill (clicca `[Installa]` o dice
"installa la skill X"), **non scrivere file tu**. L'installazione è
delegata allo `skill-installer` (forkato da `legal-builder-hub` di
Anthropic, sotto Apache-2.0) — gestisce allowlist, fetch in subagent
read-only, structural trust check, license verification pre+post fetch,
freshness gate, install log strutturato, tutto silenziosamente.
L'avvocato vede solo una riga per-tier che riassume cosa sta installando
e conferma esplicitamente.

**Install e adattamento italiano sono due richieste separate.**
L'installazione avviene tramite `skill-installer` che applica i
controlli di sicurezza automaticamente. Dopo l'installazione l'avvocato
riceve un promemoria su come richiedere l'adattamento italiano — questo
richiede una seconda richiesta esplicita da parte sua. La REV2 cascade
refactor (2026-05-18) ha rimosso il vecchio hook che attivava
l'adattamento automaticamente per skill IT/EU: non funzionava in cowork
e lasciava fuori il caso comune di skill generiche che l'avvocato
voleva comunque in italiano.

### 3.1 — Invoca skill-installer

Passa al `skill-installer` la voce del bollettino selezionata
dall'avvocato (`bollettino_entry` completo, inclusi `repo_url`,
`skill_path`, `tier`, `reputation.license`, `founder_disclaimer`).
L'installer esegue il proprio workflow silenziosamente e mostra
all'avvocato una sola riga per-tier:

- **Tier 1** (Anthropic-official): *"Installando [nome] — plugin
  ufficiale Anthropic, licenza Apache-2.0. ..."*
- **Tier 2** (publisher terzo, passa threshold): *"Installando
  [publisher]/[nome] — publisher terzo, passa i check tecnici
  automatici. ..."*
- **Tier 2 WARN** (anomalia non bloccante): Tier 2 + *"Anomalia
  rilevata: [...]. Non bloccante. Procedi?"*
- **REFUSE** (license assente / hook sospetto / injection): *"Skill
  [nome] rifiutata: [motivo]. Installazione bloccata per sicurezza."*

Vedi `skills/skill-installer/SKILL.md` per il dettaglio.

### 3.2 — Due richieste separate: install poi (eventualmente) adattamento

**Per ogni skill installata, dovrai esplicitamente chiedere
l'adattamento italiano come secondo passo cosciente. L'installer non
lo fa automaticamente.**

Dopo che lo `skill-installer` conferma l'installazione, mostra a sua
volta un nudge:

> *"Skill `[nome]` installata in versione originale (inglese). Per
> adattarla al diritto italiano e verificare le citazioni normative,
> scrivi: 'adatta `[nome]` in italiano' o usa `/adattamento-italiano
> [nome]`."*

Se l'avvocato non chiede l'adattamento, la skill resta in versione
originale (inglese) — può essere usata così, ma MHC-L non proporrà
`verifica-fonti` automaticamente sui suoi output. L'avvocato può
chiedere l'adattamento in qualsiasi momento successivo.

### 3.3 — Cosa fai tu (catalogo) dopo l'installazione

Lo `skill-installer` ti restituisce l'esito (`installed` / `cancelled` /
`refused_by_security_gate`). Aggiorna `.mhc-l-state.json` di
conseguenza (`installed_skills` solo se esito `installed`). Comunica
all'avvocato:

- **Installata**: ripeti il nudge sull'adattamento italiano (vedi 3.2)
  + invito a rileggere i file in
  `~/.claude/plugins/config/mhc-l/installed_skills/<nome>/` e provarla
  su una pratica a basso rischio prima dell'uso professionale.
- **Cancellata dall'avvocato**: *"Installazione annullata. La skill
  `<nome>` resta non installata."*
- **Rifiutata dal gate sicurezza**: ripeti il motivo dato dallo
  `skill-installer` (es. license assente, pattern injection bloccante).
  Non bypassare.

**Mai installare bypassando lo `skill-installer`.** Il fork del layer
sicurezza Anthropic è il motivo per cui esiste questo plugin nella forma
attuale — bypassarlo significa perdere allowlist, internal trust
analysis, freshness gate, security findings.

### 3.4 — Update di una skill già installata (re-install + re-adattamento opzionale)

Quando una skill **già installata** dall'avvocato riceve un update
upstream (nuovo commit / nuovo release nel suo repo originale) e il
bollettino lo segnala, il flusso è **deliberatamente conservativo**:
re-installa via `skill-installer` (che rifà tutti i check sulla nuova
versione) e — se la skill era stata adattata in italiano in precedenza —
proponi all'avvocato di rigenerare l'adattamento da zero.

Sequenza:

1. **Re-installa la nuova versione** via `skill-installer`. L'installer
   rifà allowlist, license check, structural trust, heuristic scan,
   freshness — e mostra all'avvocato la riga per-tier per nuova
   conferma. Su `sì`, sovrascrive la skill installata con la nuova
   versione originale.
2. **Se la skill era stata adattata in italiano** (controlla
   `install-log.yaml` → `italian_adaptation_applied: true` per quel
   `skill_name`), proponi all'avvocato:
   > *"La skill `[nome]` era stata adattata in italiano. L'upgrade ha
   > sovrascritto l'adattamento con la nuova versione originale. Vuoi
   > rigenerare l'adattamento italiano sulla nuova versione? (sì / no /
   > più tardi)"*
3. Su `sì`, invoca `adattamento-italiano [nome]` come secondo passo.
   Quella skill rigenera la proposta da zero sulla nuova versione,
   pre-flight `verifica-fonti`, e — se l'avvocato approva — sovrascrive
   di nuovo la skill installata con la versione adattata. Se in passato
   l'avvocato aveva fatto edit specifici (registrati in
   `lawyer_edits_applied`), `adattamento-italiano` glieli ripropone
   come suggerimento di riapplicarli.

**Perché rigenera-da-zero l'adattamento e non un diff:** applicare un
diff a un adattamento custom prodotto da un LLM è fragile (la mappatura
adaptation-italiano ↔ skill-originale non è strutturata, gli errori di
merge silenziosi su riferimenti normativi non danno errori runtime ma
errori sostanziali). Costo: 5-15K token + 5-10 min di tempo
dell'avvocato per review. Costo accettabile data la frequenza
(mensile, non continua) e il contesto (legale, dove la correttezza
vince sull'efficienza).

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
      "tier": 1,
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

**Nota su `tier`:**
- `1` — Anthropic-official (publisher sotto `anthropics/*`). Drive la
  riga per-tier "plugin ufficiale Anthropic" mostrata dallo
  `skill-installer`.
- `2` — community vetted (publisher terzo che passa la threshold
  policy). Drive la riga per-tier "publisher terzo, passa i check
  tecnici automatici" mostrata dallo `skill-installer`.

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

1. **Installare una skill bypassando `skill-installer`.** Lo
   `skill-installer` (forkato Apache-2.0 da Anthropic) è il solo
   punto che applica allowlist, internal trust analysis, license
   verification, heuristic scan, freshness gate, install log
   strutturato. Bypassarlo = perdere il layer di sicurezza
   industrial-grade.
2. **Scrivere file di skill installata direttamente da catalogo.** Lo
   `skill-installer` è il solo che scrive in
   `~/.claude/plugins/config/mhc-l/installed_skills/`.
3. **Invocare `adattamento-italiano` automaticamente dopo
   l'installazione.** REV2 (2026-05-18) ha disaccoppiato install e
   adattamento: l'adattamento è un secondo passo cosciente che
   l'avvocato richiede esplicitamente. Il tuo compito è solo
   mostrare il nudge post-install che lo invita a farlo (vedi
   Passo 3.2).
4. **Mostrare il catalogo come fosse uno shop.** Non è una vetrina,
   è una lista skill ecosystem-monitored che ha superato la
   threshold policy. Niente "promo", "trending", "consigliati per
   te" stile raccomandazione algoritmica.
5. **Saltare il disclaimer al primo uso.** L'avvocato deve sapere
   che la responsabilità giuridica resta sua, che il founder
   garantisce solo la distribuzione, e che il bollettino è popolato
   automaticamente.
6. **Trattare il bundled come scenario degradato o d'errore.** Il
   bundled è la **modalità di default** per la maggior parte degli
   avvocati (target primario = non-tech, non ha configurato l'egress
   allowlist di Claude Desktop). Il messaggio deve essere neutro
   ("Catalogo installato con il plugin, versione: X"), mai
   allarmista. La modalità live è opt-in opzionale, non lo standard
   atteso. REV2.5 doctrine (2026-05-18).

7. **Inferire la disponibilità di `WebFetch` senza provarla.**
   Tentare `WebFetch` resta vincolo cardine (REV2.2): il check
   empirico è la sola fonte di verità sul runtime, e l'avvocato
   avanzato che ha attivato l'opt-in egress deve poter accedere
   alla modalità live senza alcun nuovo intervento. Mai inferire
   "WebFetch non funziona" senza provare.

8. **Mostrare l'errore tecnico di `WebFetch` nel messaggio
   lawyer-facing.** L'avvocato non-tech non può fare niente con
   *"egress blocked"*, *"dominio non whitelisted"*, *"403"*, ecc. —
   si confonde, si sente tech-inadequate, perde fiducia nel sistema
   ("perché mi sta mostrando un errore?"). L'errore tecnico ricevuto
   da `WebFetch` va catturato e scritto nell'`install-log.yaml` per
   debug futuro, NON mostrato in chat. Il messaggio lawyer-facing
   resta neutro (vedi bullet 6 + Passo 1 §2).

9. **Suggerire "clicca Update sul plugin" come refresh del
   bollettino.** Misleading: l'update plugin è operazione tecnica di
   manutenzione (pull commit più recente da GitHub) — non è
   "aggiorna il catalogo" nel mental model dell'avvocato. Per il
   refresh contenuto la via giusta è la modalità live (opt-in
   egress allowlist documentato sulla pagina istruzioni). REV2.5
   doctrine (2026-05-18).
