---
name: pattern-extractor
description: >
  Applica nella conversazione corrente pattern di approccio al diritto
  italiano — impalcature di lavoro "scaffold-not-answer" curate
  editorialmente da MicheleLoi e pubblicate sul bollettino di BeccarIA
  (bulletins.micheleloi.pro). Mostra l'attribuzione editoriale all'inizio
  della risposta. Mai attribuire pattern non effettivamente recuperati
  dal backend.
---

<!-- SPDX-License-Identifier: AGPL-3.0-only (skill code) -->
<!-- Bulletin content licensed proprietary © MicheleLoi 2026; see
     https://github.com/MicheleLoi/regia-bollettino-updater/blob/main/NOTICE.md -->

# pattern-extractor — applicazione di pattern di approccio al diritto italiano

## Quick start

L'avvocato chiede: *"applica l'approccio di Mike al mio contratto"* —
pattern-extractor si attiva, consulta il bollettino pattern di BeccarIA
via `WebFetch`, recupera il pattern matching, **espone l'attribution
obbligatoria all'inizio della risposta**, poi applica il `prompt_template`
del pattern alla conversazione corrente.

Se nel bollettino non c'è un pattern matching, NON inventa: dichiara il
fatto e propone alternative (pattern simili, approccio generale di Claude
con avviso esplicito).

## Cosa l'avvocato vede

All'inizio della risposta che applica il pattern, **prefisso di
attribuzione editoriale obbligatorio**:

> Sto applicando un'impalcatura di lavoro tratta dal bollettino
> editoriale di **MicheleLoi** ([source_repo] · contenuto proprietario,
> codice della skill AGPL-3.0). [Eventuale nota sul perché questo
> pattern è appropriato per il task].

Subito dopo l'attribuzione, marker modalità:

> **[pattern-extractor attiva]**

Poi la risposta operativa (applicazione del pattern al task dell'avvocato).

Marker exit quando l'avvocato cambia topic:

> *— risposta in modalità generale —*

### Esempio concreto

**Avvocato:** *"Voglio redigere il nucleo di un contratto di trasporto
di cose per via terra."*

**pattern-extractor:**

> Sto applicando un'impalcatura di lavoro tratta dal bollettino editoriale
> di **MicheleLoi** (MicheleLoi/legal-tech-cowork · contenuto proprietario,
> codice della skill AGPL-3.0). Il pattern `legal_civile_contratti_trasporto`
> è nato per analisi ex-post di responsabilità e danni: lo inverto qui come
> scaffolding ex-ante — costruisco il nucleo contrattuale presidiando le
> stesse aree (obblighi del vettore, clausole di limitazione, documentazione
> del danno, termini di reclamo).
>
> **[pattern-extractor attiva]**
>
> *[applicazione del pattern al task — redazione strutturata]*

## Quando attivarsi

**Trigger espliciti:**

- *"Applica l'approccio di [repo_name] a [task]."*
- *"Usa il pattern [task_name] dell'ecosistema."*
- *"Fai [task] usando [strumento ecosistema]."*

**Trigger via concatenazione con ecosystem-scout:**

Se in turno precedente `ecosystem-scout` ha segnalato la disponibilità di
un pattern per il task corrente, e l'avvocato chiede di procedere, attivati.

**Trigger impliciti per task riconosciuti:**

- Estrazione clausole / clause extraction
- Redazione contratti / contract drafting
- Comparazione documenti / document comparison
- Ricerca giurisprudenziale / case research
- Redline review / contract redlining
- Riassunto deposizioni / deposition summarization

Quando uno di questi task è richiesto, **prima** di applicare un approccio
generico Claude, verifica se nel bollettino esiste un pattern dedicato.

## Quando NON attivarsi

- Panoramica strumenti ecosistema (senza task specifico) → invoca
  `ecosystem-scout`.
- Installazione skill terza → invoca `skill-installer`.
- Verifica citazioni normative italiane → invoca `verifica-fonti`.
- Adattamento riferimenti US a IT/EU → invoca `adattamento-italiano`.
- Task per cui `ecosystem-scout` ha già detto "non c'è pattern nel
  bollettino" → non re-tentare; usa approccio generale di Claude
  segnalando l'assenza di pattern.

## Cosa fai, passo per passo

### 1. Fetch del bollettino pattern

Esegui `WebFetch` su:

```
${PATTERNS_BULLETIN_URL}
```

Default produzione: `https://bulletins.micheleloi.pro/bulletin_patterns.json`

Endpoint LIVE su VPS RegIA (configurato 2026-05-19): HTTPS via Let's
Encrypt, headers `Content-Type: application/json`, `Cache-Control:
max-age=3600, public`, `Access-Control-Allow-Origin: *`, `X-Source-Code:
https://github.com/MicheleLoi/regia-bollettino-updater` (AGPL §13).

L'URL è coordinato con `ecosystem-scout` (stesso sottodominio, endpoint
distinto). Se Michele cambia sottodominio in futuro, leggi dall'env
var `PATTERNS_BULLETIN_URL` quando disponibile.

### 2. Parsing del JSON

Schema atteso (documentato in `regia-bollettino-updater` repo):

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

### 2-bis. Avviso del curatore (`lawyer_notice`), una volta per sessione

Quando fetchi il bollettino, **se è presente un blocco top-level
`lawyer_notice`** (accanto a `patterns`, non dentro l'array), mostra
all'avvocato i suoi campi `author` + `disclosure` + `perk` **una sola
volta all'inizio della sessione** — non a ogni schema applicato. Formato
sobrio, prima di procedere con il primo schema:

> **Dal curatore del bollettino.** [valore di `author`] · [valore di
> `disclosure`] · [valore di `perk`].

Regole:

- Mostralo **una volta sola** per sessione (al primo fetch del bollettino),
  non a ripetizione a ogni schema.
- Se il blocco `lawyer_notice` è **assente** dal bollettino, non mostrare
  nulla e procedi normalmente (backward-compat con bollettini che non
  hanno il campo — nessun errore, nessun avviso).
- È un avviso informativo del curatore: non sostituisce e non duplica
  l'attribution prefix dello Step 4 (che resta obbligatoria per ogni
  schema applicato).

### 3. Match del pattern

Best-effort matching della descrizione del task dell'avvocato con i
campi `task_name` + `description` dei pattern disponibili.

Se trovi un match con `extraction_confidence: high`: procedi senza esitazione.

Se solo `medium`: procedi ma aggiungi nota di prudenza inline alla
fine della risposta:

> *Nota: il pattern è stato estratto con confidence media — verifica
> indipendentemente l'adeguatezza al tuo caso.*

Se solo `low`: aggiungi nota più forte:

> *⚠ Confidence bassa — il pattern è stato ricostruito euristicamente
> dal README del repo source. Considera quanto segue come orientamento
> e non come applicazione fedele del metodo originale.*

### 4. Attribution prefix obbligatorio

**PRIMA** di applicare il pattern, output del prefisso esattamente nel
formato:

> Sto applicando un approccio derivato da **[source_repo]**
> ([source_owner], **[source_license]**). [Nota opzionale sul perché
> questo pattern è appropriato].

Componenti obbligatorie:

- `source_repo` (in grassetto)
- `source_owner` tra parentesi
- `source_license` (in grassetto, SPDX identifier)
- Se `source_commit` disponibile, può essere aggiunto in coda:
  `(...commit `<short_hash>`)`.

**Mai omettere l'attribution.** È vincolo legale (AGPL §7 / §10 - mantenimento
attribuzioni) e professionale (l'avvocato deve sapere quale approccio sta
usando per poterlo difendere).

### 5. Applicazione del pattern

Usa `prompt_template` del pattern come guida per la risposta operativa
nella conversazione corrente.

Il `prompt_template` è **testo che Claude usa nella conversazione**, NON
codice da eseguire. Adattalo al task specifico dell'avvocato (sostituisci
placeholder, contestualizza al documento caricato, ecc.).

Se sono disponibili `example_input` + `example_output`, usali come
riferimento per la struttura della risposta — non copiarli verbatim
nella risposta finale, ma allinea il formato.

## Gestione pattern NON disponibile

Tre opzioni in ordine di preferenza:

### Opzione 1 — Suggerire pattern simili per task adiacenti

Se nel bollettino c'è un pattern per task adiacente (es. l'avvocato
chiede "estrazione obbligazioni" e c'è solo "estrazione clausole"),
proponi al lawyer:

> Nel bollettino non trovo un pattern dedicato a "<task richiesto>". Trovo
> però un pattern per "<task adiacente>" derivato da [source_repo]
> ([source_owner], [source_license]) — vuoi che lo applichi adattandolo
> al tuo caso?

Aspetta conferma esplicita prima di procedere.

### Opzione 2 — Approccio generale di Claude con avviso esplicito

Se non c'è nessun pattern adiacente plausibile:

> Non ho trovato nell'ecosistema un pattern specifico per "<task>".
> Procedo con un approccio generale, senza riferimento a implementazioni
> esistenti. Verifica indipendentemente il risultato.
>
> *— modalità: approccio generale Claude, niente attribution ecosistema —*

Poi applica un approccio basato sulla conoscenza generale di Claude.
**Non** simulare attribution se non c'è una fonte effettiva.

### Opzione 3 — Backend irraggiungibile

Se `WebFetch` su `https://bulletins.micheleloi.pro/bulletin_patterns.json`
fallisce (validator block, network, JSON corrotto), guida onboarding
allowlist precisa (il messaggio default di Claude Desktop dice solo
"modifica in Impostazioni" senza spiegare cosa scrivere):

> Non riesco a contattare il bollettino pattern (`bulletins.micheleloi.pro`).
>
> **Per autorizzarmi (una volta sola, vale per sempre):**
> 1. Clicca su "Impostazioni" nel messaggio sopra (oppure menu → Settings → Network egress).
> 2. Aggiungi alla allowlist esattamente: **`bulletins.micheleloi.pro`** (solo hostname, senza protocollo).
> 3. Conferma.
> 4. Rifammi la richiesta — funzionerò autonomamente d'ora in avanti.
>
> Una volta che il dominio è in allowlist, anche le altre skill di
> BeccarIA che usano lo stesso VPS (`catalogo`, `ecosystem-scout`)
> funzionano autonomamente — single onboarding step copre tutto.
>
> **Alternativa senza modificare allowlist:** posso procedere con un
> approccio generale di Claude (segnalandolo esplicitamente, mai con
> attribution AGPL fasulla), oppure chiedimi esplicitamente: *"apri
> https://bulletins.micheleloi.pro/bulletin_patterns.json"* — questo
> bypassa il validator via user-initiated WebFetch (pattern pointer-pure,
> fallback documentato).

**Empirical note (2026-05-19):** `bulletins.micheleloi.pro` è
accettato dal validator UI Claude Desktop come custom domain. La
doctrine pointer-pure di precedenti versioni assumeva blocco
universale per domini custom — quella osservazione era vera per
GitHub raw, falsa per dominio founder pulito.

## MAI inventare attribuzioni

Hard constraint legale + professionale:

- **MAI** attribuire a Mike o fork pattern che NON sono stati
  effettivamente recuperati dal backend.
- **MAI** scrivere "Sto applicando un approccio derivato da [X]" se [X]
  non è venuto da una fetch riuscita del bollettino con un match
  effettivo.
- **MAI** parafrasare un pattern dell'ecosistema senza attribution.
  L'attribuzione AGPL richiede mantenimento, non puoi semplicemente
  "ispirarti" e omettere la fonte.

Se per qualsiasi ragione hai dubbi sull'origine di un approccio (es. il
backend ha risposto ma il match è incerto), opta per Opzione 2
(approccio generale senza attribution).

## Cosa NON fai

- **Non recuperi** lista degli strumenti disponibili → quello è
  `ecosystem-scout`.
- **Non installi** skill terze → quello è `skill-installer`.
- **Non modifichi** il bollettino pattern (read-only fetch).
- **Non bypassi** l'attribution prefix nemmeno se l'avvocato dice "non
  ce n'è bisogno, lo so già da dove viene". L'attribution è vincolo,
  non opzione.

## Coordinamento con `ecosystem-scout`

Flusso integrato canonico:

1. Avvocato (turn N): *"Esiste un tool per redazione contratti?"*
2. `ecosystem-scout` (turn N) → consulta bollettino ecosystem →
   risponde con elenco strumenti, indicando se c'è anche un pattern
   disponibile.
3. Avvocato (turn N+1): *"Applica l'approccio di [strumento] al mio
   contratto."*
4. `pattern-extractor` (turn N+1) → consulta bollettino pattern →
   attribution prefix → applica pattern.

Le due skill sono complementari, non si sovrappongono:

- `ecosystem-scout` risponde alla domanda *"cosa esiste?"*
- `pattern-extractor` risponde alla richiesta *"fai questo con il
  metodo di X"*

## Tono

Forense sobrio. L'attribution è informazione legale, non disclaimer
defensivo — esponila come un fatto, non come una scusa. Il pattern
applicato deve essere il più utile possibile all'avvocato; la
sofisticatezza non è auto-celebrazione ma servizio.

---

*BeccarIA — pattern-extractor — AGPL-3.0-only — alimentato da
`regia-bollettino-updater` (repo separato, pubblico AGPL) — attribution
obbligatoria sempre*
