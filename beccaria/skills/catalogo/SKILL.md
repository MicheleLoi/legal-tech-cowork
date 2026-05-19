---
name: catalogo
description: >
  Catalogo curato delle skill legal-tech per l'avvocato italiano (componente
  della modalità AVANZATA opt-in del plugin BeccarIA, non default). Fetcha
  autonomous il bollettino mensile dal VPS RegIA (bulletins.micheleloi.pro/bulletin_skills.json)
  appena l'avvocato attiva esplicitamente la skill; processa il contenuto JSON;
  presenta novità e avvisi importanti; orchestra installazione e adattamento di
  skill terze tramite richieste esplicite dell'avvocato. NON si attiva
  automaticamente: si attiva SOLO su invocazione esplicita dell'avvocato
  ("mostrami il catalogo", "apri il bollettino delle skill italiane",
  "che skill posso installare?", "ci sono novità nel bollettino?",
  "/catalogo", o equivalenti).
---

# BeccarIA — Catalogo skill legal-tech (meta-skill)

## Posizionamento: gateway modalità avanzata opt-in

Questa skill è il **punto d'ingresso del workflow avanzato opt-in** del
plugin beccaria. Il plugin opera a due livelli:

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
pointer 2026-05-19 (refactor 3.3.0: `catalogo` da fetcher a pointer)
+ revert 3.3.1 di `skill-installer` ad autonomous post-trigger +
**doctrine 4.0.0 autonomous fetch (2026-05-19 PM very late)** —
scoperta empirica: `bulletins.micheleloi.pro` è accettato dal
validator UI Claude Desktop come custom domain via Impostazioni →
allowlist egress. Il bollettino skill terze migra su VPS RegIA;
`catalogo` upgrada da pointer-pure a autonomous fetch.

---

## Doctrine evolution 3.x → 4.0.0 — autonomous fetch su VPS RegIA

**3.3.0 → 4.0.0: `catalogo` upgrade da pointer-pure a autonomous fetch.**
Versione 3.3.0/3.3.1 — questa skill non faceva `WebFetch` autonomous
del bollettino: puntava all'avvocato l'URL su GitHub raw e il
fraseggio per chiedere a Claude di aprirlo via `WebFetch`
user-initiated. Motivo: il polling ricorrente di GitHub raw in
background era bloccato dall'egress allowlist di Claude Desktop, e
configurarla era ulteriormente bloccato da un bug del validatore UI
("Solo gestori di pacchetti" rifiutava domini come `raw.githubusercontent.com`).
**Scoperta empirica 2026-05-19 sera:** dominio founder pulito
(`bulletins.micheleloi.pro`) è accettato dal validator UI Claude
Desktop senza problemi. Il blocco era specifico per GitHub raw, non
universale per domini custom.

**Conseguenze ratificate:**

1. Il bollettino skill terze è stato migrato da `legal-tech-cowork/beccaria/bollettino.json`
   (GitHub raw) a `https://bulletins.micheleloi.pro/bulletin_skills.json`
   (VPS RegIA, HTTPS Let's Encrypt, headers Content-Type + Cache-Control
   + CORS + X-Source-Code per AGPL §13).
2. `catalogo` ora fa `WebFetch` autonomous sul VPS al primo trigger
   esplicito dell'avvocato.
3. Onboarding requirement: l'avvocato aggiunge `bulletins.micheleloi.pro`
   alla allowlist egress di Claude Desktop **una volta** (single step),
   poi tutte le skill BeccarIA che usano il VPS funzionano autonomous
   (catalogo + ecosystem-scout + pattern-extractor).
4. Pointer-pure resta documentato come **fallback** per casi enterprise
   lockdown / allowlist non modificabile dall'utente.

**Skill-installer resta autonomous post-trigger esplicito (invariato).**
Quando l'avvocato sceglie esplicitamente una skill dal catalogo
("installa skill X"), `skill-installer` fa `WebFetch` autonomous del
`SKILL.md` candidato dal repo originale terzo (non dal VPS RegIA,
perché ogni skill terza vive nel proprio repo GitHub) e applica i 5
controlli automatici di sicurezza. Quel trigger esplicito è
l'autorizzazione user-initiated implicita sufficiente. La doctrine
3.3.1 di skill-installer regge senza modifiche.

In una frase: **`catalogo` ora autonomous fetch dal VPS RegIA al primo
trigger esplicito (con fallback pointer-pure se allowlist non
modificabile); `skill-installer` autonomous post-trigger esplicito su
SKILL.md di repo terzo (invariato).** Single onboarding step
(allowlist `bulletins.micheleloi.pro`) copre tutte le skill VPS-based.

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
   (che applica silenziosamente tutti i 5 check di sicurezza sul
   `SKILL.md` della skill terza — fetched autonomous dall'installer
   stesso post-trigger esplicito dell'avvocato — e poi scrive i file
   in `~/.claude/plugins/config/beccaria/installed_skills/<nome>/`).
   Dopo l'installazione, ricordo all'avvocato che può chiedere
   l'adattamento italiano come **secondo passo cosciente separato**,
   invocando esplicitamente `adattamento-italiano [nome]`. Install
   e adattamento sono due richieste distinte.
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
> aggiornata del bollettino, lo fetcho per te in chat dal VPS RegIA
> (`bulletins.micheleloi.pro`). Lo leggo non in background nascosto:
> il fetch avviene come azione esplicita di questa skill su tua
> richiesta esplicita (es. "mostrami il catalogo"). La prima volta che
> attivi la skill ti chiedo di aggiungere `bulletins.micheleloi.pro`
> alla allowlist di Claude Desktop (Impostazioni → Network egress) —
> single step, una volta sola, vale per tutte le skill BeccarIA che
> usano lo stesso VPS.
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
   `~/.claude/plugins/config/beccaria/state.json` (path standard per
   state durevole di plugin in Claude Desktop). Se non esiste, è il
   primo uso *dell'installazione del plugin* (non del progetto).
   Crealo dopo la conferma con `{"disclaimer_accepted": true,
   "accepted_on": "YYYY-MM-DD"}`. Da questa scrittura in poi, in
   qualunque cartella di lavoro l'avvocato apra cowork con beccaria
   installato, il disclaimer non si ripresenta.

2. **Fallback — stato per cartella di lavoro:** se la scrittura in
   `~/.claude/plugins/config/beccaria/state.json` fallisce (permission
   denied dalla sandbox cowork, path non scrivibile, ecc.), ripiega su
   un file `.beccaria-state.json` nella cartella di lavoro connessa con
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

## Passo 1 — Fetch autonomous del bollettino

Il bollettino vive sul VPS RegIA, servito via HTTPS:

**URL canonico produzione:**

```
https://bulletins.micheleloi.pro/bulletin_skills.json
```

Il bollettino è **popolato automaticamente** dalla routine
`bollettino-research` (founder fuori dal loop runtime — curazione
cristallizzata in una threshold policy esplicita: license OSS,
reputation minima, IT-relevance heuristic). L'updater pubblica i
JSON sul VPS via SSH/rsync.

**Cosa fai all'attivazione esplicita:**

Esegui `WebFetch` autonomous su `https://bulletins.micheleloi.pro/bulletin_skills.json`.
Parsa il JSON, processa il contenuto, procedi a Passo 2.

Schema atteso (documentato in `regia-bollettino-updater` repo, source
of truth):

```json
{
  "schema_version": "1.0.0",
  "generated_at": "2026-05-19T22:53:00Z",
  "source_count": 0,
  "skills": [
    {
      "name": "...",
      "source_repo": "owner/name",
      "source_url": "https://github.com/...",
      "tier": 1,
      "license": "Apache-2.0",
      "jurisdiction": "IT",
      "italian_adaptation_status": "ready",
      "description": "...",
      "last_seen": "2026-05-19T...",
      "notes": null
    }
  ]
}
```

(Schema definitivo coordinato con il repo updater; campi e valori esatti
dal file pydantic `src/schema/skills.py` di `regia-bollettino-updater`.)

### Fallback onboarding allowlist (prima attivazione, validator block)

Se `WebFetch` fallisce con errore di rete / validator block (succede la
prima volta che l'avvocato attiva la skill se `bulletins.micheleloi.pro`
non è ancora in allowlist Claude Desktop), il messaggio default di
Claude Desktop è "L'accesso a questo sito web è bloccato... Puoi
modificarle in Impostazioni" — **non dice cosa scrivere**. Colma il
gap con istruzioni precise:

> Non riesco a contattare il bollettino delle skill (`bulletins.micheleloi.pro`).
>
> **Per autorizzarmi (una volta sola, vale per sempre):**
> 1. Clicca su "Impostazioni" nel messaggio sopra (oppure menu → Settings → Network egress).
> 2. Aggiungi alla allowlist esattamente: **`bulletins.micheleloi.pro`** (solo hostname, senza protocollo).
> 3. Conferma.
> 4. Rifammi la richiesta — funzionerò autonomamente d'ora in avanti.
>
> Una volta che il dominio è in allowlist, anche le altre skill di
> BeccarIA che usano lo stesso VPS (`ecosystem-scout`, `pattern-extractor`)
> funzionano autonomously — single onboarding step copre tutto.
>
> **Alternativa senza modificare allowlist:** posso punterti l'URL e
> chiedimi esplicitamente *"apri https://bulletins.micheleloi.pro/bulletin_skills.json"*
> — questo bypassa il validator via user-initiated WebFetch (pattern
> pointer-pure, fallback documentato dalla doctrine 3.3.0). A quel punto
> il contenuto entra in contesto e posso processarlo come al Passo 2.

**Empirical note (2026-05-19):** `bulletins.micheleloi.pro` è
accettato dal validator UI Claude Desktop come custom domain. La
doctrine pointer-pure 3.3.0 (che assumeva blocco universale) era
basata su osservazione vera solo per GitHub raw, falsa per dominio
founder pulito.

### Cosa fare se l'avvocato rifiuta di modificare la allowlist

Rispetta la scelta. Procedi col fallback pointer-pure (punta URL,
attendi user-initiated open). Non insistere.

### Cosa fare se `skills` (una volta letto il bollettino) è vuoto

Lo stato iniziale del bollettino è vuoto perché il catalogo cresce
per accumulo automatico. Comunicalo onestamente:

> *"Il catalogo è ancora in costruzione. La routine automatica
> `bollettino-research` monitora mensilmente l'ecosistema legal-tech
> open source e pubblica sul VPS le skill che superano la threshold
> policy. Quando saranno disponibili, le vedrai qui. Nel frattempo,
> posso aiutarti su lavori legali generali — chiedimi pure."*

---

## Passo 2 — Presenta il catalogo (due pannelli)

Una volta che il contenuto del bollettino è disponibile nel contesto
(Claude lo ha appena letto su richiesta esplicita dell'avvocato),
processa il JSON e presenta in due pannelli.

### Pannello A — "Novità nel catalogo"

Mostrato quando l'avvocato dice "mostrami il catalogo", "ci sono
novità", "cosa c'è di nuovo", o all'inizio di una conversazione se è la
prima dopo un aggiornamento del bollettino (confronta `last_updated`
del bollettino con `last_seen` in `.beccaria-state.json`).

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
`.beccaria-state.json` → `installed_skills`).

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

**Doctrine: skill-installer è autonomous post-trigger esplicito
(3.3.1, invariato in 4.0.0).** Quando l'avvocato sceglie esplicitamente
una skill dal catalogo (clicca `[Installa]` o dice *"installa la skill X"*),
quel trigger esplicito autorizza implicitamente `skill-installer` a
fare `WebFetch` autonomous del `SKILL.md` candidato dal repo originale
terzo (NON dal VPS RegIA — ogni skill terza vive nel proprio repo
GitHub) per applicare i 5 controlli di sicurezza (allowlist, structural
trust, license, heuristic, freshness). L'installer scrive poi i file in
`~/.claude/plugins/config/beccaria/installed_skills/<nome>/` dopo
approvazione esplicita per-tier dell'avvocato — niente passi UI manuali.

In 4.0.0 anche `catalogo` upgrada a autonomous fetch (dal VPS RegIA
sul dominio whitelistato `bulletins.micheleloi.pro`); single onboarding
step (allowlist add) abilita tutto il workflow VPS-based.

L'avvocato vede solo una riga per-tier che riassume l'esito dei
controlli e conferma con un singolo `sì`.

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
L'installer fa autonomous `WebFetch` del `SKILL.md` candidato (è
autorizzato dal trigger esplicito dell'avvocato che ha scelto la
skill), applica i 5 controlli silenziosamente e mostra all'avvocato
una sola riga per-tier con prompt di conferma `sì/no`:

- **Tier 1** (Anthropic-official): *"Installando [nome] — plugin
  ufficiale Anthropic, licenza Apache-2.0. ... Procedo?"*
- **Tier 2** (publisher terzo, passa threshold): *"Installando
  [publisher]/[nome] — publisher terzo, passa i check tecnici
  automatici. ... Procedo?"*
- **Tier 2 WARN** (anomalia non bloccante): Tier 2 + *"Anomalia
  rilevata: [...]. Non bloccante. Procedo?"*
- **REFUSE** (license assente / hook sospetto / injection): *"Skill
  [nome] rifiutata: [motivo]. Installazione bloccata per
  sicurezza."* — terminale, nessun override.

Su `sì` esplicito, l'installer scrive i file della skill in
`~/.claude/plugins/config/beccaria/installed_skills/<nome>/` e
appende l'entry all'`install-log.yaml`. Vedi
`skills/skill-installer/SKILL.md` per il dettaglio.

### 3.2 — Due richieste separate: install poi (eventualmente) adattamento

**Per ogni skill installata, dovrai esplicitamente chiedere
l'adattamento italiano come secondo passo cosciente. L'installer non
lo fa automaticamente.**

Dopo che lo `skill-installer` conferma l'installazione, mostra a sua
volta un nudge (passive se `jurisdiction ∈ {IT, EU}`, active se
`[?]` / `other` / `none`):

> *"Skill `[nome]` installata. Se vuoi adattarla al diritto italiano
> e verificare le citazioni normative, scrivi: 'adatta `[nome]` in
> italiano' o usa `/adattamento-italiano [nome]`."*

Se l'avvocato non chiede l'adattamento, la skill resta in versione
originale (inglese) — può essere usata così, ma beccaria non proporrà
`verifica-fonti` automaticamente sui suoi output. L'avvocato può
chiedere l'adattamento in qualsiasi momento successivo.

### 3.3 — Cosa fai tu (catalogo) dopo l'installazione

Lo `skill-installer` ti restituisce l'esito (`installed` /
`cancelled` / `refused_by_security_gate`). Aggiorna
`.beccaria-state.json` di conseguenza (`installed_skills` solo se
esito `installed`). Comunica all'avvocato:

- **Installata**: ripeti il nudge sull'adattamento italiano (vedi
  3.2) + invito a rileggere il `SKILL.md` della skill installata e
  provarla su una pratica a basso rischio prima dell'uso
  professionale.
- **Cancellata dall'avvocato**: *"Installazione annullata. La skill
  `<nome>` resta non installata."*
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
   freshness — sul SKILL.md nuovo, fetched autonomous dall'installer
   stesso (post-trigger esplicito di update) — e mostra all'avvocato
   la riga per-tier per nuova conferma. Su `sì`, l'installer
   sovrascrive la skill in `installed_skills/<nome>/` con la nuova
   versione.
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
come input. Se dice no, registra il "no" in `.beccaria-state.json` →
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

1. **Primaria (globale al plugin):** `~/.claude/plugins/config/beccaria/state.json`.
   Una sola volta per installazione; si applica a tutti i progetti.
2. **Fallback (per cartella di lavoro):** `.beccaria-state.json` nella
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

1. **Fare `WebFetch` autonomous del bollettino in background.**
   Doctrine pointer (3.3.0): per il bollettino, il gate di rete è la
   richiesta esplicita dell'avvocato a Claude di aprire l'URL. Tu
   `catalogo` non apri mai l'URL del bollettino da sola — punti
   all'avvocato l'URL e il fraseggio, e aspetti che lui chieda. Test
   empirico founder 2026-05-19: skill agent autonomous fetch
   ricorrente in background (polling del bollettino) è soggetto a
   egress allowlist; user-initiated WebFetch la bypassa. Questa
   regola **non** si estende allo `skill-installer` (3.3.1 revert):
   l'installer fa autonomous fetch puntuale del `SKILL.md` candidato
   *dopo* trigger esplicito dell'avvocato — quello è un caso
   diverso (puntuale, user-initiated implicito sufficiente,
   funziona out-of-box).

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

5. **Auditare/installare una skill terza senza trigger esplicito
   dell'avvocato.** Lo `skill-installer` fa autonomous fetch del
   `SKILL.md` candidato *solo* dopo che l'avvocato ha scelto
   esplicitamente quella skill (clicca `[Installa]` o dice
   *"installa la skill X"*). Senza trigger esplicito, non
   instradare al `skill-installer` e non inventare il contenuto
   della skill terza. Il trigger esplicito è l'autorizzazione
   user-initiated implicita che legittima l'autonomous fetch
   puntuale.

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
