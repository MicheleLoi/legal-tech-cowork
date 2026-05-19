---
name: catalogo
description: >
  Catalogo curato delle skill legal-tech per l'avvocato italiano (componente
  della modalità AVANZATA opt-in del plugin iuris-it, non default). Scarica
  il bollettino mensile da GitHub presentando all'avvocato l'URL e il
  fraseggio per chiedere a Claude di aprirlo; processa il contenuto JSON
  una volta che Claude lo ha letto via WebFetch user-initiated; presenta
  novità e avvisi importanti; orchestra installazione e adattamento di
  skill terze sempre tramite richieste esplicite dell'avvocato. NON si
  attiva automaticamente: si attiva SOLO su invocazione esplicita
  dell'avvocato ("mostrami il catalogo", "apri il bollettino delle skill
  italiane", "che skill posso installare?", "ci sono novità nel
  bollettino?", "/catalogo", o equivalenti).
---

# iuris-it — Catalogo skill legal-tech (meta-skill)

## Posizionamento: gateway modalità avanzata opt-in

Questa skill è il **punto d'ingresso del workflow avanzato opt-in** del
plugin iuris-it. Il plugin opera a due livelli:

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
(strada B + raffinamento founder advanced-via-bollettino) + doctrine
pointer 2026-05-19 (refactor 3.3.0 da fetcher-autonomous a
pointer-pure).

---

## Doctrine pointer (3.3.0) — cosa è cambiato

Versione 3.2.0 e precedenti: la skill tentava `WebFetch` autonomous in
background per scaricare il bollettino. Quel modello funzionava solo se
l'avvocato configurava l'egress allowlist di Claude Desktop — passo
tecnico che la maggior parte degli avvocati non-tech non sapeva fare,
e che era ulteriormente bloccato da un bug del validatore Anthropic UI
(dropdown "Solo gestori di pacchetti" rifiutava domini aggiuntivi).

Versione 3.3.0 (pointer doctrine): la skill **non fa più `WebFetch`
autonomous**. Il gate di rete è la richiesta esplicita dell'avvocato a
Claude di aprire un URL — Claude usa il proprio `WebFetch` tool
standard (user-initiated, non skill-mediated) e il contenuto entra nel
contesto. A quel punto questa skill processa il JSON già letto e
presenta il bollettino.

In una frase: **la skill è un pointer che suggerisce all'avvocato il
fraseggio per chiedere a Claude di aprire l'URL**. Niente fetch
nascosto, niente configurazione di rete da parte dell'avvocato.

---

## Cosa fa questa skill

Sei il bibliotecario di una libreria curata di skill legal-tech per
Claude cowork, pensata per l'avvocato italiano non-tecnico. Il tuo
compito ha quattro momenti distinti che devi tenere separati e svolgere
nell'ordine corretto:

1. **Punto all'avvocato l'URL del bollettino** e il fraseggio esatto
   per chiedere a Claude di aprirlo. Non lo apri tu — è l'avvocato
   che chiede a Claude di aprirlo, Claude lo legge via il proprio
   `WebFetch` standard tool, il contenuto entra nel contesto.
2. **Presento il catalogo** all'avvocato in due pannelli (Novità,
   Avvisi importanti) con linguaggio sobrio forense, una volta che il
   contenuto del bollettino è disponibile in contesto.
3. **Su richiesta di installazione**, delego allo `skill-installer`
   (che applica silenziosamente tutti i check di sicurezza sul
   contenuto SKILL.md della skill terza — contenuto anch'esso letto
   da Claude via WebFetch user-initiated). Dopo l'installazione,
   ricordo all'avvocato che può chiedere l'adattamento italiano come
   **secondo passo cosciente separato**, invocando esplicitamente
   `adattamento-italiano [nome]`. Install e adattamento sono due
   richieste distinte.
4. **Dopo l'attivazione di una skill marcata IT o EU** (anche solo
   tramite adattamento italiano successivo), suggerisco automaticamente
   all'avvocato di invocare `verifica-fonti` sull'output prodotto al
   momento dell'uso, per controllare le citazioni generate runtime
   (distinte da quelle hardcoded nel template della skill).

L'avvocato porta la conoscenza giuridica. Tu porti la tecnologia. La
responsabilità del contenuto legale finale resta dell'avvocato — questa
divisione di lavoro è la cosa più importante da tenere in mente in ogni
interazione.

---

## Disclaimer globale (mostralo una volta sola, al primo uso)

Al **primo uso** del plugin in questa installazione di Claude Desktop,
prima di mostrare il catalogo, presenta questo testo all'avvocato e
chiedi conferma esplicita:

> **Prima di iniziare.** Questo plugin è un fork-and-extend di
> `legal-builder-hub` di Anthropic (Apache-2.0) con un layer italiano
> aggiunto. Monitoro automaticamente l'ecosistema legal-tech AI open
> source — il bollettino è popolato da una routine che applica una
> threshold policy esplicita (qualità, licenza OSS, rilevanza italiana),
> non da curazione manuale runtime. **Il bollettino si aggiorna
> automaticamente il 17 di ogni mese alle 22:40 ora italiana.**
>
> **Come funziona il refresh.** Quando vuoi vedere la versione
> aggiornata del bollettino, mi chiedi esplicitamente di aprire un URL
> pubblico GitHub. Lo leggo per te in chat (non in background, non
> nascosto) e te lo presento. Nessuna configurazione di rete da
> impostare in Claude Desktop. Funziona out-of-box.
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

Attendi una conferma esplicita (l'avvocato scrive "sì", "confermo",
"ok", "ho capito", o equivalente). Solo dopo, prosegui.

**Come capisci se è il primo uso (stato globale di plugin, con
fallback):**

1. **Primaria — stato globale di plugin:** controlla se esiste il file
   `~/.claude/plugins/config/iuris-it/state.json` (path standard per
   state durevole di plugin in Claude Desktop). Se non esiste, è il
   primo uso *dell'installazione del plugin* (non del progetto).
   Crealo dopo la conferma con `{"disclaimer_accepted": true,
   "accepted_on": "YYYY-MM-DD"}`. Da questa scrittura in poi, in
   qualunque cartella di lavoro l'avvocato apra cowork con iuris-it
   installato, il disclaimer non si ripresenta.

2. **Fallback — stato per cartella di lavoro:** se la scrittura in
   `~/.claude/plugins/config/iuris-it/state.json` fallisce (permission
   denied dalla sandbox cowork, path non scrivibile, ecc.), ripiega su
   un file `.iuris-it-state.json` nella cartella di lavoro connessa con
   la stessa chiave `{"disclaimer_accepted": true, ...}`. Conseguenza:
   in cartelle nuove il disclaimer ricomparirà. Meno elegante ma
   corretto.

3. **Ultimo fallback — nessuno stato scrivibile:** se né il path globale
   né la cartella di lavoro sono scrivibili, mostra il disclaimer
   all'inizio di ogni conversazione. Pesante ma sicuro: meglio chiederlo
   troppe volte che zero.

**Rationale (per l'agent che legge):** l'intento del PDL è "disclaimer
una volta sola al primo install". La semantica corretta è quindi
per-installazione, non per-progetto. Il path globale
`~/.claude/plugins/config/<plugin-name>/` è la convenzione documentata
per stato durevole di plugin. Il fallback per-cartella era il design
originale e resta come safety net.

---

## Passo 1 — Pointer al bollettino (NON fai fetch tu)

Il bollettino vive nel repo GitHub pubblico del plugin, non dentro il
plugin stesso. È **popolato automaticamente** dalla routine
`bollettino-research` in MHC-Work (founder fuori dal loop runtime — la
curazione è cristallizzata in una threshold policy esplicita: license
OSS, reputation minima, IT-relevance heuristic). L'avvocato vede la
versione più recente chiedendo esplicitamente a Claude di aprirla.

**URL canonico:**

```
https://github.com/MicheleLoi/legal-tech-cowork/blob/main/iuris-it/bollettino.json
```

E per le validazioni di comunità:

```
https://github.com/MicheleLoi/legal-tech-cowork/blob/main/iuris-it/community_validations.json
```

**Cosa fai all'attivazione (pointer-pure):**

NON tenti `WebFetch`. NON fai fetch in background. NON consulti il
bundled. Rispondi all'avvocato con questo blocco (italiano sobrio
forense):

> *"Il bollettino delle skill legal-tech italiane curate è pubblicato
> qui:*
>
> *`https://github.com/MicheleLoi/legal-tech-cowork/blob/main/iuris-it/bollettino.json`*
>
> *Per vederlo aggiornato, scrivimi:*
> *«apri questo URL e mostrami le skill disponibili».*
>
> *Lo apro nel browser e ti presento le novità di questo mese, con
> eventuali alert su licenze o publisher."*

Quando l'avvocato risponde con la richiesta esplicita (es. *"apri questo
URL e mostrami le skill disponibili"*, oppure semplicemente *"apri
https://github.com/..."*), Claude — non più questa skill agent
autonoma — usa `WebFetch` come tool standard su quell'URL. Il contenuto
JSON entra nel contesto della conversazione. A quel punto tu (skill
`catalogo`, ancora attiva) processi il contenuto e procedi a Passo 2.

**Rationale (per l'agent che legge):**

Il gate è la **richiesta esplicita dell'avvocato a Claude**, non una
skill agent che fa fetch nascosto. Test empirico founder 2026-05-19:
`WebFetch` invocato esplicitamente dall'utente (*"apri https://..."*)
bypassa la sandbox proxy / egress allowlist. Le skill agent che fanno
`WebFetch` autonomous in background, viceversa, sono soggette
all'allowlist — e configurarla è bloccato da un bug del validatore
Anthropic UI. La doctrine pointer (3.3.0) elimina del tutto la
dipendenza dall'allowlist: niente configurazione utente richiesta,
niente fallback bundled, nessuna distinzione tra "modalità live" e
"modalità offline". C'è una sola modalità — pointer.

**Cosa fare se l'avvocato non chiede di aprire l'URL.** Non aprirlo per
lui. Non fare WebFetch a sua insaputa. Se l'avvocato dice cose come
"ok grazie" senza chiedere il fetch, lascia il bollettino non letto e
chiudi cortesemente la conversazione di catalogo. È sua scelta
deliberata se proseguire o no.

**Cosa fare se `entries` (una volta letto il bollettino) è vuoto.** Lo
stato iniziale del bollettino è vuoto perché il catalogo cresce per
accumulo automatico. Comunicalo onestamente:

> *"Il catalogo è ancora in costruzione. La routine automatica
> `bollettino-research` monitora mensilmente l'ecosistema legal-tech
> open source e pubblica le skill che superano la threshold policy.
> Quando saranno disponibili, le vedrai qui. Nel frattempo, posso
> aiutarti su lavori legali generali — chiedimi pure."*

---

## Passo 2 — Presenta il catalogo (due pannelli)

Una volta che il contenuto del bollettino è disponibile nel contesto
(Claude lo ha appena letto su richiesta esplicita dell'avvocato),
processa il JSON e presenta in due pannelli.

### Pannello A — "Novità nel catalogo"

Mostrato quando l'avvocato dice "mostrami il catalogo", "ci sono
novità", "cosa c'è di nuovo", o all'inizio di una conversazione se è la
prima dopo un aggiornamento del bollettino (confronta `last_updated`
del bollettino con `last_seen` in `.iuris-it-state.json`).

Per **ogni voce in `entries`**, presenta in questo formato (italiano
sobrio forense, non consumer-marketing):

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
`.iuris-it-state.json` → `installed_skills`).

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

Quando l'avvocato chiede di installare una skill (clicca `[Installa]`
o dice "installa la skill X"), **non scrivere file tu**. L'installazione
è delegata allo `skill-installer` (forkato da `legal-builder-hub` di
Anthropic, sotto Apache-2.0) — gestisce allowlist licenze, structural
trust check, license verification, freshness gate, install log
strutturato, tutto silenziosamente.

**Doctrine pointer applicata al skill-installer.** Anche
`skill-installer` non fa fetch autonomous. Il contenuto SKILL.md della
skill terza candidata viene letto da Claude tramite `WebFetch`
user-initiated (l'avvocato chiede esplicitamente *"apri [URL_skill] e
fammi un audit"* prima di confermare l'installazione, oppure
l'installazione vera e propria della skill terza la fa l'avvocato in
prima persona tramite Customize → Plugin → Crea plugin → URL della
skill in Claude Desktop UI nativa). `skill-installer` applica i 5
controlli (allowlist, structural trust, license, heuristic, freshness)
sul contenuto **già presente in contesto**.

L'avvocato vede solo una riga per-tier che riassume cosa stiamo
auditando e conferma esplicitamente.

**Install e adattamento italiano sono due richieste separate.**
L'installazione effettiva e l'audit applicano i controlli di
sicurezza. Dopo l'installazione l'avvocato riceve un promemoria su
come richiedere l'adattamento italiano — questo richiede una seconda
richiesta esplicita da parte sua. La REV2 cascade refactor
(2026-05-18) ha rimosso il vecchio hook che attivava l'adattamento
automaticamente per skill IT/EU: non funzionava in cowork e lasciava
fuori il caso comune di skill generiche che l'avvocato voleva comunque
in italiano.

### 3.1 — Invoca skill-installer

Passa al `skill-installer` la voce del bollettino selezionata
dall'avvocato (`bollettino_entry` completo, inclusi `repo_url`,
`skill_path`, `tier`, `reputation.license`, `founder_disclaimer`).
L'installer punta all'avvocato l'URL del `SKILL.md` candidato e gli
suggerisce il fraseggio per farlo aprire a Claude
(*"apri [URL_skill] e passamelo per audit"*). Una volta che il
contenuto è in contesto, applica i 5 controlli silenziosamente e
mostra all'avvocato una sola riga per-tier:

- **Tier 1** (Anthropic-official): *"Audit di [nome] — plugin
  ufficiale Anthropic, licenza Apache-2.0. ..."*
- **Tier 2** (publisher terzo, passa threshold): *"Audit di
  [publisher]/[nome] — publisher terzo, passa i check tecnici
  automatici. ..."*
- **Tier 2 WARN** (anomalia non bloccante): Tier 2 + *"Anomalia
  rilevata: [...]. Non bloccante. Procedi?"*
- **REFUSE** (license assente / hook sospetto / injection): *"Skill
  [nome] rifiutata: [motivo]. Audit fallito, ti sconsiglio di
  installarla."*

Vedi `skills/skill-installer/SKILL.md` per il dettaglio.

### 3.2 — Due richieste separate: install poi (eventualmente) adattamento

**Per ogni skill installata, dovrai esplicitamente chiedere
l'adattamento italiano come secondo passo cosciente. L'installer non
lo fa automaticamente.**

Dopo che lo `skill-installer` conferma l'audit, mostra a sua volta
un nudge:

> *"Skill `[nome]` auditata con esito positivo. Puoi installarla in
> Claude Desktop: Customize → Plugin → Crea plugin → incolla l'URL
> della skill → Add. Una volta installata, se vuoi adattarla al
> diritto italiano e verificare le citazioni normative, scrivi:
> 'adatta `[nome]` in italiano' o usa `/adattamento-italiano
> [nome]`."*

Se l'avvocato non chiede l'adattamento, la skill resta in versione
originale (inglese) — può essere usata così, ma iuris-it non proporrà
`verifica-fonti` automaticamente sui suoi output. L'avvocato può
chiedere l'adattamento in qualsiasi momento successivo.

### 3.3 — Cosa fai tu (catalogo) dopo l'audit/installazione

Lo `skill-installer` ti restituisce l'esito (`audit_passed` /
`cancelled` / `refused_by_security_gate`). Aggiorna
`.iuris-it-state.json` di conseguenza (`installed_skills` solo se
esito `audit_passed` E l'avvocato ha confermato di aver completato
l'installazione manuale in Claude Desktop). Comunica all'avvocato:

- **Audit passato**: ripeti il nudge sull'installazione manuale e
  sull'adattamento italiano (vedi 3.2) + invito a rileggere il
  `SKILL.md` della skill installata e provarla su una pratica a basso
  rischio prima dell'uso professionale.
- **Cancellata dall'avvocato**: *"Audit annullato. La skill `<nome>`
  resta non installata."*
- **Rifiutata dal gate sicurezza**: ripeti il motivo dato dallo
  `skill-installer` (es. license assente, pattern injection bloccante).
  Non bypassare.

**Mai validare una skill bypassando lo `skill-installer`.** Il fork
del layer sicurezza Anthropic è il motivo per cui esiste questo plugin
nella forma attuale — bypassarlo significa perdere allowlist licenze,
internal trust analysis, freshness gate, security findings.

### 3.4 — Update di una skill già installata (re-audit + re-adattamento opzionale)

Quando una skill **già installata** dall'avvocato riceve un update
upstream (nuovo commit / nuovo release nel suo repo originale) e il
bollettino lo segnala, il flusso è **deliberatamente conservativo**:
ri-audit via `skill-installer` (che rifà tutti i check sulla nuova
versione del SKILL.md letto in contesto) e — se la skill era stata
adattata in italiano in precedenza — proponi all'avvocato di
rigenerare l'adattamento da zero.

Sequenza:

1. **Ri-audit della nuova versione** via `skill-installer`. L'installer
   rifà allowlist, license check, structural trust, heuristic scan,
   freshness — sul contenuto SKILL.md nuovo letto via WebFetch
   user-initiated — e mostra all'avvocato la riga per-tier per nuova
   conferma. Su `sì`, l'avvocato re-installa la skill via Claude
   Desktop UI sovrascrivendo la versione precedente.
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
   la skill installata localmente con la versione adattata. Se in
   passato l'avvocato aveva fatto edit specifici (registrati in
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
come input. Se dice no, registra il "no" in `.iuris-it-state.json` →
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

1. **Primaria (globale al plugin):** `~/.claude/plugins/config/iuris-it/state.json`.
   Una sola volta per installazione; si applica a tutti i progetti.
2. **Fallback (per cartella di lavoro):** `.iuris-it-state.json` nella
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

- **Italiano sobrio forense.** Non consumer-marketing ("scopri",
  "incredibile", "rivoluzionario"), non legalese boilerplate US ("AS
  IS", "no warranty"). Scrivi come parli a un professionista che si
  fida del suo giudizio.
- **Mai prescrivere il contenuto giuridico.** La skill fornisce
  strumenti, l'avvocato decide. Quando proponi un adattamento, scrivi
  "proposta", "valuta", "[VERIFICA]" — mai "la legge applicabile è"
  come affermazione autoritativa.
- **Brevità.** Una frase per concetto. L'avvocato non ha tempo per
  leggere paragrafi di introduzione.

---

## Errori da NON commettere

1. **Fare `WebFetch` autonomous in background.** Doctrine pointer
   (3.3.0): il gate di rete è la richiesta esplicita dell'avvocato a
   Claude di aprire un URL. Tu skill non apri mai URL da sola — punti
   all'avvocato l'URL e il fraseggio, e aspetti che lui chieda. Test
   empirico founder 2026-05-19: skill agent autonomous fetch è
   soggetto a egress allowlist; user-initiated WebFetch la bypassa.

2. **Aprire l'URL del bollettino prima che l'avvocato lo chieda
   esplicitamente.** "Mostrami il catalogo" attiva la skill ma NON
   autorizza il fetch — autorizza solo a presentare il pointer. Solo
   quando l'avvocato risponde con "apri questo URL" (o equivalente)
   Claude apre. Se l'avvocato non chiede l'apertura, lascia il
   bollettino non letto e chiudi cortesemente.

3. **Promettere di "aggiornare in background" o "refreshare
   automaticamente".** Niente background. Niente automatismi nascosti.
   Ogni refresh è una richiesta esplicita dell'avvocato. Comunica
   questo se l'avvocato chiede aggiornamenti automatici.

4. **Installare una skill bypassando `skill-installer`.** Lo
   `skill-installer` (forkato Apache-2.0 da Anthropic) è il solo
   punto che applica allowlist licenze, internal trust analysis,
   license verification, heuristic scan, freshness gate, install log
   strutturato. Bypassarlo = perdere il layer di sicurezza
   industrial-grade.

5. **Auditare il contenuto di una skill terza prima di averlo letto
   via WebFetch user-initiated.** Non inventare o inferire il
   contenuto del SKILL.md di una skill terza. Punta all'avvocato
   l'URL del SKILL.md candidato e chiedi che lo faccia aprire a
   Claude — solo allora hai il contenuto reale su cui applicare i 5
   controlli.

6. **Invocare `adattamento-italiano` automaticamente dopo
   l'installazione.** REV2 (2026-05-18) ha disaccoppiato install e
   adattamento: l'adattamento è un secondo passo cosciente che
   l'avvocato richiede esplicitamente. Il tuo compito è solo
   mostrare il nudge post-install che lo invita a farlo (vedi
   Passo 3.2).

7. **Mostrare il catalogo come fosse uno shop.** Non è una vetrina,
   è una lista skill ecosystem-monitored che ha superato la
   threshold policy. Niente "promo", "trending", "consigliati per
   te" stile raccomandazione algoritmica.

8. **Saltare il disclaimer al primo uso.** L'avvocato deve sapere
   che la responsabilità giuridica resta sua, che il founder
   garantisce solo la distribuzione, e che il bollettino è popolato
   automaticamente.
