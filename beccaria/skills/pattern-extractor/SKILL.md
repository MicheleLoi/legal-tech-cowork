---
name: pattern-extractor
description: >
  Applica pattern di lavoro all'avvocato italiano da due fonti del bollettino
  BeccarIA: (1) pattern di approccio code-grade da ecosistema legal-AI open
  source (Mike e fork, attribution AGPL obbligatoria); (2) pattern di
  metodologia legale (scaffold-not-answer, generati da AI come impalcatura,
  con nota canonica che trasferisce la responsabilità delle citazioni
  all'avvocato). Dopo l'uso, propone — opzionalmente — un form di feedback
  community via email pre-compilata (nessun upload automatico). Si attiva
  quando l'avvocato vuole un approccio strutturato per un task, oppure
  vuole contribuire feedback sul bollettino BeccarIA.
---

<!-- SPDX-License-Identifier: AGPL-3.0-only -->

# pattern-extractor — applicazione pattern bollettino + feedback community

## Due tipi di pattern, due workflow distinti

Il bollettino BeccarIA serve due famiglie di pattern, distinte da `pattern_type`:

| `pattern_type` | Fonte | Workflow | Esempio trigger |
|---|---|---|---|
| `heuristic_task` (default v1) | repo ecosistema (Mike e fork) | **attribution AGPL** + applica `prompt_template` | *"applica l'approccio di Mike al mio NDA"* |
| `legal_content_methodology` (schema v2) | curation interna RegIA (Haiku batch + founder review) | **scaffold-not-answer** + nota canonica + step come proposte | *"esame contratto trasporto"*, *"impostazione ricorso TAR"* |

Le due famiglie vivono su due endpoint separati ma sullo stesso VPS:

- `https://bulletins.micheleloi.pro/bulletin_patterns.json` — pattern heuristic_task (legacy v1, schema 1.0.0)
- `https://bulletins.micheleloi.pro/bulletin_legal_patterns.json` — pattern legal_content_methodology (LIVE dal 2026-05-28, schema 2.0, 25 pattern scaffold iniziali)

La skill instrada in base all'intento dell'avvocato. In caso di ambiguità, chiede.

## Quando attivarsi

**Trigger per `legal_content_methodology` (scaffold-not-answer):**

- Task di metodologia legale italiana: *"analisi contratto trasporto"*, *"esame ricorso amministrativo"*, *"impostazione separazione consensuale"*, *"verifica clausole vessatorie"*, *"approccio per accesso atti L.241"*, ecc.
- Avvocato chiede "come affronto un caso di [topic]" o "dammi uno scheletro per [task legale]".

**Trigger per `heuristic_task` (attribution AGPL):**

- *"Applica l'approccio di [repo_name] a [task]."*
- *"Usa il pattern [task_name] dell'ecosistema."*
- *"Fai [task] usando [strumento ecosistema]."*

**Trigger per feedback community:**

- *"Voglio contribuire feedback sul bollettino BeccarIA"*
- Dopo l'uso effettivo di un pattern (qualsiasi famiglia), la skill propone — opzionalmente — la raccolta feedback in chiusura (vedi §"Feedback community" sotto).

**Trigger via concatenazione con `ecosystem-scout`:**

Se nel turno precedente `ecosystem-scout` ha segnalato la disponibilità di un pattern per il task corrente, e l'avvocato chiede di procedere, attivati.

## Quando NON attivarsi

- Panoramica strumenti ecosistema (senza task specifico) → invoca `ecosystem-scout`.
- Installazione skill terza → invoca `skill-installer`.
- Verifica citazioni normative italiane su un atto già scritto → invoca `verifica-fonti`.
- Adattamento riferimenti US a IT/EU → invoca `adattamento-italiano`.
- Task per cui un pattern non c'è e l'avvocato lo sa già → procedi con approccio generale Claude, senza attribution fasulla.

## Workflow A — `legal_content_methodology` (scaffold-not-answer)

### A.1. Fetch del bollettino legal_patterns

Esegui `WebFetch` su:

```
https://bulletins.micheleloi.pro/bulletin_legal_patterns.json
```

Endpoint LIVE su VPS RegIA: HTTPS Let's Encrypt, `Content-Type: application/json`, `Cache-Control: max-age=3600, public`, `Access-Control-Allow-Origin: *`. Coordinato con `ecosystem-scout` (stesso sottodominio).

### A.2. Match del pattern

Best-effort matching dell'intento dell'avvocato con i campi `domain.topic_primary`, `domain.topic_sub`, `search.keywords`, `search.aree_normative_rilevanti`, `title`.

Se trovi più match plausibili (es. *"contratto trasporto"* matcha sia `legal_civile_contratti_trasporto_001` sia `legal_civile_contratti_generale_001`), elenca i candidati all'avvocato e chiedi quale preferisce.

Se non trovi nulla di pertinente:

> Nel bollettino BeccarIA non trovo un pattern di metodologia per "<task>". Posso (a) cercare un pattern code-grade nell'ecosistema esterno via `bulletin_patterns.json`, (b) procedere con un approccio generale di Claude (senza pattern), o (c) suggerirti di mandare richiesta della creazione del pattern via feedback community.

### A.3. Apertura — `nota_per_avvocato` verbatim, NON comprimibile

**PRIMA di tutto il resto**, mostra all'avvocato il campo `nota_per_avvocato` del pattern **letteralmente come è scritto nel bollettino**. Non riformulare, non comprimere, non nascondere sotto spoiler. È un atto comunicativo che trasferisce la responsabilità delle citazioni puntuali all'avvocato.

Formattazione canonica:

```
⚠️  NOTA — <campo nota_per_avvocato verbatim>

─────────────────────────────────────────────────────────────────
```

### A.4. Presentazione pattern come proposta

Dopo la nota, presenta il pattern come **scheletro proposto**, non come istruzione autoritativa. Format canonico:

```
📋 <title> (<pattern_id>)

Generato da: <generated_by.model> <generated_by.date>
<eventuale: "non ancora curato da umano" se human_validated=false>
<eventuale: "validato da founder" se founder_curated=true>
Usato finora: <empirical_use_count> volte (<empirical_validation_count> applied to real case, <empirical_rejection_count> rejected)

<reasoning_explanation verbatim>

Aree normative dove guardare:
- <aree_normative_rilevanti[0]>
- <aree_normative_rilevanti[1]>
- ...

Approccio proposto (è un punto di partenza, non l'unica via):

1. <default_approach[0].name>
   → guarda: <default_approach[0].riferimenti_da_consultare>
   <eventuale: rationale se utile per chiarire>

2. <default_approach[1].name>
   → guarda: <default_approach[1].riferimenti_da_consultare>

...

Possibili variazioni (scartane quante vuoi):
- <optional_variations[0]>
- <optional_variations[1]>
- ...

Cosa preferisci?
(a) Procedi con questo scheletro
(b) Lo adatto — ti dico come (es. salta step X, aggiungi Y)
(c) Mostra altri pattern simili
(d) Scarta tutto, fai approccio generale senza pattern
```

### A.5. Esecuzione

A seconda della risposta:

- **(a)** Procedi guidando l'avvocato step per step, mostrando per ogni step il `name`, il `rationale` e il `riferimenti_da_consultare`. Per ogni step ricorda — quando l'avvocato sta per citare articoli puntuali — di usare `/verifica-fonti` sui riferimenti specifici.
- **(b)** Raccogli l'adattamento dell'avvocato (quale step saltare, cosa aggiungere) e procedi con la versione modificata.
- **(c)** Mostra 2-3 altri pattern con `topic_primary` o `topic_sub` adiacente.
- **(d)** Procedi con approccio generale Claude, senza attribution pattern, segnalando esplicitamente: *"— modalità: approccio generale Claude, niente pattern scaffold —"*.

### A.6. Chiusura — proposta feedback community

Al termine dell'uso effettivo del pattern (l'avvocato lo ha applicato, anche solo per orientarsi), proponi la raccolta feedback (vedi §"Feedback community" sotto). Sempre opt-in esplicito, mai forzato.

## Workflow B — `heuristic_task` (attribution AGPL, invariato v1)

### B.1. Fetch del bollettino patterns legacy

Esegui `WebFetch` su:

```
${PATTERNS_BULLETIN_URL}
```

Default produzione: `https://bulletins.micheleloi.pro/bulletin_patterns.json`

### B.2. Parsing JSON (schema 1.0.0)

```json
{
  "schema_version": "1.0.0",
  "generated_at": "2026-05-19T18:00:00Z",
  "source_count": 8,
  "patterns": [
    {
      "task_name": "contract_clause_extraction",
      "description": "...",
      "prompt_template": "Sei un assistente legale. Estrai le clausole...",
      "example_input": "...",
      "example_output": "...",
      "source_repo": "willchen96/mike",
      "source_owner": "willchen96",
      "source_url": "https://github.com/willchen96/mike",
      "source_commit": "abc123",
      "source_license": "AGPL-3.0-only",
      "extraction_confidence": "high"
    }
  ]
}
```

### B.3. Attribution prefix obbligatorio

**PRIMA** di applicare il pattern, output del prefisso esattamente nel formato:

> Sto applicando un approccio derivato da **[source_repo]** ([source_owner], **[source_license]**). [Nota opzionale sul perché questo pattern è appropriato].
>
> **[pattern-extractor attiva]**

Componenti obbligatorie:

- `source_repo` (in grassetto)
- `source_owner` tra parentesi
- `source_license` (in grassetto, SPDX identifier)
- Se `source_commit` disponibile, può essere aggiunto in coda: `(...commit short_hash)`.

**Mai omettere l'attribution.** È vincolo legale (AGPL §7 / §10 — mantenimento attribuzioni) e professionale (l'avvocato deve sapere quale approccio sta usando per poterlo difendere).

### B.4. Confidence handling

- `extraction_confidence: high` → procedi senza esitazione.
- `medium` → procedi ma aggiungi nota inline alla fine: *Nota: il pattern è stato estratto con confidence media — verifica indipendentemente l'adeguatezza al tuo caso.*
- `low` → nota più forte: *⚠ Confidence bassa — il pattern è stato ricostruito euristicamente dal README del repo source. Considera quanto segue come orientamento e non come applicazione fedele del metodo originale.*

### B.5. Applicazione

Usa `prompt_template` come guida per la risposta operativa nella conversazione corrente. Il template è **testo che Claude usa nella conversazione**, NON codice da eseguire. Adattalo al task specifico (sostituisci placeholder, contestualizza al documento caricato).

Marker exit quando l'avvocato cambia topic:

> *— risposta in modalità generale —*

### B.6. Chiusura — proposta feedback community

Come per Workflow A: al termine, proponi raccolta feedback opzionale (vedi §"Feedback community").

## Feedback community (smart form via mailto, zero upload server)

### Trigger

Dopo l'uso effettivo di un pattern (qualsiasi famiglia: `legal_content_methodology` o `heuristic_task`), in chiusura — **e solo in chiusura, mai durante** — proponi all'avvocato:

> Vuoi contribuire questo feedback alla community RegIA? Aiuta a migliorare il bollettino per tutti gli avvocati italiani. Tempo: ~1 minuto. Nessun dato esce dal tuo computer senza il tuo click esplicito su "Invia email".

Se l'avvocato dice **no** o ignora, niente di più: la skill non insiste, non chiede una seconda volta nella stessa sessione.

Se l'avvocato dice **sì**, procedi con il form 4-step.

### Form 4-step

**Step 1 — Tipo di feedback (scelta multipla):**

```
Che tipo di feedback?

(1) gap_fill — Manca qualcosa che andrebbe nel pattern
(2) adaptation — Ho adattato il pattern al mio caso, ecco come
(3) correction — C'è un errore nel pattern (citazione sbagliata, step inappropriato, ecc.)
(4) general — Commento generale su utilità / esperienza
```

Salva la scelta come `<feedback_type>` (`gap_fill | adaptation | correction | general`).

**Step 2 — Cosa manca / cosa hai adattato / cosa va corretto (free-text, max 500 char):**

Adatta il prompt alla scelta del passo 1. Esempi (mostra l'esempio coerente con `<feedback_type>`):

- `gap_fill` → *"Cosa manca? (max 500 caratteri). Esempio: 'Mancano gli step per la verifica di legittimazione attiva del richiedente accesso atti'."*
- `adaptation` → *"Cosa hai adattato? (max 500 caratteri). Esempio: 'Ho aggiunto step iniziale per qualifica del rapporto vettore-consumer prima della verifica clausole'."*
- `correction` → *"Cosa va corretto? (max 500 caratteri). Esempio: 'Lo step 3 cita norme sul mandato (artt. 1701-1703 cc) ma stiamo parlando di trasporto — quelle norme sono fuori sezione'."*
- `general` → *"Il tuo commento (max 500 caratteri). Esempio: 'Pattern utile ma reasoning_explanation troppo lunga, semplificherei'."*

Salva l'input come `<feedback_body>`. Se l'avvocato eccede 500 char, segnala e chiedi di accorciare (non troncare silenziosamente).

**Step 3 — Approccio che useresti tu (opzionale, free-text, max 1000 char):**

> Vuoi proporre un approccio alternativo o un'aggiunta concreta? (opzionale, max 1000 caratteri, lascia vuoto per saltare).
>
> Esempio: *"Per i casi di trasporto consumer userei: verifica vincoli pre-contrattuali → controllo Cod. Cons. art. 48-52 (info pre-contrattuali) → poi disciplina ordinaria trasporto."*

Salva come `<lawyer_approach>` (stringa vuota se l'avvocato salta).

**Step 4 — Attribuzione (opzionale):**

> Vuoi essere accreditato (es. "Avv. Mario Rossi")? Lascia vuoto per anonimato.

Salva come `<lawyer_attribution>`. Se vuoto, usa stringa `anonymous lawyer`.

### Assemblaggio markdown verbatim

Componi il markdown deterministico nel formato (mostra all'avvocato per review pre-invio):

```markdown
---
beccaria_feedback_version: 1
pattern_id: <pattern_id usato dall'avvocato>
feedback_type: <feedback_type>
submitted_date: <YYYY-MM-DD oggi>
lawyer_attribution_optional: <lawyer_attribution se non vuota, altrimenti "anonymous lawyer">
---

## Cosa manca / cosa è stato adattato / cosa va corretto

<feedback_body verbatim>

## Approccio dell'avvocato

<lawyer_approach verbatim, oppure "non fornito" se vuoto>
```

### Generazione mailto link

Costruisci dynamicamente l'URL:

```
mailto:mhcl@micheleloi.pro?subject=[BeccarIA Feedback] <pattern_id>&body=<URL-encoded markdown>
```

Regole di encoding:

- `subject` e `body` sono URL-encoded (RFC 3986): spazi → `%20`, newline → `%0A`, `[` → `%5B`, `]` → `%5D`, `:` → `%3A`, `#` → `%23`, ecc.
- Il `pattern_id` nel subject va inserito letteralmente (è già URL-safe per costruzione: `legal_civile_contratti_trasporto_001`, `contract_clause_extraction` ecc.).
- L'intero markdown (frontmatter + corpo) va nel `body`, URL-encoded integralmente.

Esempio (gap_fill su pattern trasporto, attribuzione vuota):

```
mailto:mhcl@micheleloi.pro?subject=%5BBeccarIA%20Feedback%5D%20legal_civile_contratti_trasporto_001&body=---%0Abeccaria_feedback_version%3A%201%0Apattern_id%3A%20legal_civile_contratti_trasporto_001%0Afeedback_type%3A%20gap_fill%0Asubmitted_date%3A%202026-05-28%0Alawyer_attribution_optional%3A%20anonymous%20lawyer%0A---%0A%0A%23%23%20Cosa%20manca%20%2F%20cosa%20%C3%A8%20stato%20adattato%20%2F%20cosa%20va%20corretto%0A%0AMancano%20gli%20step%20per%20la%20verifica%20di%20legittimazione%20attiva...%0A%0A%23%23%20Approccio%20dell%27avvocato%0A%0Anon%20fornito
```

### Output finale all'avvocato

Mostra in chat (verbatim):

```
─────────────────────────────────────────────────────────────────

📨 Feedback pronto. Revisione finale prima dell'invio:

<markdown verbatim>

─────────────────────────────────────────────────────────────────

🔗 Link mailto pre-compilato (clicca o copia nel browser):

<URL mailto completo>

ℹ️  Quando clicchi il link, si apre il tuo client email con il messaggio
   già compilato. Premi Invia per condividere il feedback. Se preferisci
   non condividere, semplicemente non cliccare. Niente esce dal tuo
   computer senza il tuo click esplicito.

🔒 Nota privacy: il contenuto del messaggio sarà elaborato via AI per
   aggregazione + sintesi nel prossimo aggiornamento del bollettino.
   NON includere dati di clienti reali, nomi, partite IVA o info
   confidenziali. Se il tuo feedback dipende da elementi del caso reale,
   astrai prima di scrivere.
```

### Privacy hard constraint (vincoli strutturali)

- **NO storage server-side**: la skill **non** salva il feedback nel filesystem locale (no `feedback_history.md`, no log files, no artefatti persistenti), **non** invoca chiamate HTTP verso il backend BeccarIA, **non** crea file su disco.
- **NO logging**: il feedback esiste solo nel contesto della conversazione finché l'avvocato non invia via mailto; poi sparisce dalla skill.
- **NO auto-invio**: la skill **non** può inviare l'email autonomamente. Genera solo il link `mailto:`. L'avvocato clicca o no.
- **Doctrine "niente di sospetto esce dal server"**: zero data flow automatico verso servizi esterni. L'unico flow è il click manuale dell'avvocato sul mailto link, che apre il **suo** client email locale.

## Gestione pattern NON disponibile

Tre opzioni in ordine di preferenza:

### Opzione 1 — Suggerire pattern adiacenti

Se nel bollettino c'è un pattern per task adiacente (es. l'avvocato chiede "estrazione obbligazioni" e c'è solo "estrazione clausole"), proponi:

> Nel bollettino non trovo un pattern dedicato a "<task richiesto>". Trovo però un pattern per "<task adiacente>" — vuoi che lo applichi adattandolo al tuo caso?

Aspetta conferma esplicita.

### Opzione 2 — Approccio generale Claude con avviso esplicito

Se non c'è nessun pattern adiacente plausibile:

> Non ho trovato nell'ecosistema un pattern specifico per "<task>". Procedo con un approccio generale, senza riferimento a implementazioni esistenti. Verifica indipendentemente il risultato.
>
> *— modalità: approccio generale Claude, niente attribution ecosistema —*

**Non** simulare attribution se non c'è una fonte effettiva.

### Opzione 3 — Backend irraggiungibile

Se `WebFetch` su `https://bulletins.micheleloi.pro/...` fallisce (validator block, network, JSON corrotto):

> Non riesco a contattare il bollettino (`bulletins.micheleloi.pro`).
>
> **Per autorizzarmi (una volta sola, vale per sempre):**
> 1. Clicca su "Impostazioni" nel messaggio sopra (oppure menu → Settings → Network egress).
> 2. Aggiungi alla allowlist esattamente: **`bulletins.micheleloi.pro`** (solo hostname, senza protocollo).
> 3. Conferma.
> 4. Rifammi la richiesta — funzionerò autonomamente d'ora in avanti.
>
> Una volta che il dominio è in allowlist, anche le altre skill di BeccarIA che usano lo stesso VPS (`catalogo`, `ecosystem-scout`) funzionano autonomamente — single onboarding step copre tutto.
>
> **Alternativa senza modificare allowlist:** posso procedere con un approccio generale di Claude (segnalandolo esplicitamente, mai con attribution fasulla), oppure chiedimi esplicitamente: *"apri https://bulletins.micheleloi.pro/bulletin_legal_patterns.json"* — questo bypassa il validator via user-initiated WebFetch.

**Empirical note (2026-05-19):** `bulletins.micheleloi.pro` è accettato dal validator UI Claude Desktop come custom domain.

## MAI inventare

Hard constraint legale + professionale:

- **MAI** attribuire a Mike o fork pattern `heuristic_task` che NON sono stati effettivamente recuperati dal backend.
- **MAI** scrivere "Sto applicando un approccio derivato da [X]" se [X] non è venuto da una fetch riuscita con un match effettivo.
- **MAI** parafrasare un pattern `heuristic_task` senza attribution. L'attribuzione AGPL richiede mantenimento, non puoi "ispirarti" e omettere la fonte.
- **MAI** inventare pattern `legal_content_methodology` (scaffold). Se il bollettino non ha il pattern, lo dici. Non comporre un pattern scaffold ex novo dalla conoscenza generale di Claude senza segnalarlo come "approccio generale, non pattern bollettino".
- **MAI** alterare `nota_per_avvocato` di un pattern `legal_content_methodology` — sempre verbatim.
- **MAI** comprimere la `nota_per_avvocato` sotto spoiler / "vedi dettagli" — sempre visibile in apertura.

Se hai dubbi sull'origine di un approccio (es. backend ha risposto ma il match è incerto), opta per Opzione 2 (approccio generale senza attribution).

## Cosa NON fai

- **Non recuperi** lista degli strumenti disponibili → `ecosystem-scout`.
- **Non installi** skill terze → `skill-installer`.
- **Non modifichi** il bollettino (read-only fetch su entrambi gli endpoint).
- **Non bypassi** l'attribution prefix per `heuristic_task` nemmeno se l'avvocato dice "non ce n'è bisogno". L'attribution è vincolo, non opzione.
- **Non bypassi** la `nota_per_avvocato` per `legal_content_methodology` nemmeno se l'avvocato dice "salta la nota, vai al pattern". La nota è il contratto comunicativo del pattern scaffold.
- **Non salvi** feedback localmente, **non** chiami backend per upload feedback, **non** generi mail-send autonomo.

## Coordinamento con altre skill

| Skill | Domanda a cui risponde | Quando passo la palla |
|---|---|---|
| `ecosystem-scout` | *cosa esiste nell'ecosistema legal-AI?* | Lawyer chiede "che tool ci sono per X" |
| `pattern-extractor` (questa) | *come applico un pattern al mio caso?* + raccoglie feedback community | Lawyer chiede "fai X con metodo Y" o vuole scheletro per task legale |
| `verifica-fonti` | *questa citazione che ho scritto è corretta?* | Dopo che lawyer ha redatto atto con citazioni puntuali |
| `adattamento-italiano` | *come adatto questo pattern US a contesto IT/EU?* | Per pattern code-grade chiaramente US-centric |
| `skill-installer` | *come installo una skill terza?* | Lawyer chiede installazione |

Flusso integrato canonico (esempio):

1. Avvocato: *"Esiste un tool per redazione contratti?"* → `ecosystem-scout` risponde.
2. Avvocato: *"Applica l'approccio di [strumento] al mio contratto."* → `pattern-extractor` (Workflow B) consulta `bulletin_patterns.json`, attribution, applica.
3. Avvocato: *"Esame di un contratto di trasporto."* → `pattern-extractor` (Workflow A) consulta `bulletin_legal_patterns.json`, nota canonica, propone scheletro.
4. Avvocato cita art. 1693 cc nel suo atto: → invoca `verifica-fonti` su quella citazione specifica.
5. Chiusura: `pattern-extractor` propone feedback community → form 4-step → mailto pre-compilato.

## Tono

Forense sobrio. Per `heuristic_task`: attribution è informazione legale, non disclaimer defensivo — esponila come un fatto. Per `legal_content_methodology`: la nota canonica è un atto comunicativo che trasferisce responsabilità, non un disclaimer di copertura — leggila con il peso che ha. Il pattern applicato deve essere il più utile possibile all'avvocato; la sofisticatezza non è auto-celebrazione ma servizio.

---

*BeccarIA — pattern-extractor — AGPL-3.0-only — alimentato da `regia-bollettino-updater` (repo separato, pubblico AGPL) — due endpoint VPS RegIA: `bulletin_patterns.json` (heuristic_task, attribution AGPL) + `bulletin_legal_patterns.json` (legal_content_methodology, scaffold-not-answer) — feedback community via mailto, zero storage server-side*
