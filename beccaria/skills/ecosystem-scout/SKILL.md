---
name: ecosystem-scout
description: >
  Panoramica intelligente dell'ecosistema legal-AI open source (Mike e fork
  nazionali). Risponde a domande dell'avvocato su quali strumenti open source
  esistono per task legal-AI, su quali giurisdizioni, con quali licenze.
  Consulta un bollettino aggiornato dal VPS di BeccarIA e segnala
  implicazioni AGPL per studio legale.
---

<!-- SPDX-License-Identifier: AGPL-3.0-only -->

# ecosystem-scout — panoramica intelligente dell'ecosistema legal-AI open source

## Quick start

L'avvocato chiede: *"esiste un tool open source per estrarre clausole?"* —
ecosystem-scout si attiva, consulta il bollettino curato di BeccarIA via
`WebFetch`, e risponde con una lista di strumenti rilevanti, **sempre
citando la licenza di ogni strumento suggerito**. Se la licenza è AGPL,
aggiunge una nota sulle implicazioni per uno studio legale (chi integra
codice AGPL in workflow proprietari ha obblighi di pubblicazione del
source).

Se il bollettino non è raggiungibile, dichiara il fallimento apertamente
invece di inventare strumenti, e suggerisce all'avvocato un fetch
user-initiated dell'URL (pattern fallback pointer-pure).

## Cosa l'avvocato vede

Risposta strutturata in tabella o lista, con per ciascuno strumento:

- Nome + owner
- Descrizione breve
- Licenza (esplicita; flag `⚠ AGPL-3.0` con nota se l'avvocato è in studio
  che integra codice in workflow proprietario)
- Giurisdizione inferita (`IT`, `EU`, `US`, `CH`, etc., o `unknown`)
- Capabilities principali (`contract_review`, `pseudonymization`,
  `clause_extraction`, ecc.)
- Stato di attività (`attivo` se aggiornato di recente, `dormiente`
  altrimenti)
- URL del repo

Marker modalità all'inizio di ogni turno in cui ecosystem-scout è attiva:

> **[ecosystem-scout attiva]**

Marker exit quando l'avvocato cambia topic e si esce dalla modalità:

> *— risposta in modalità generale —*

## Quando attivarsi

**Trigger espliciti:**

- *"Che tool legal-AI esistono?"*
- *"Esiste un fork italiano di Mike?"*
- *"Che strumenti open source ci sono per [task]?"*
- *"Quali alternative ho per [giurisdizione]?"*

**Trigger impliciti** (riconosci anche queste formulazioni):

- *"Esiste qualcosa che fa estrazione clausole?"*
- *"Ci sono fork per il diritto svizzero?"*
- *"Vorrei sapere cosa fa Emilie."*
- *"Panoramica strumenti open source legali."*

## Quando NON attivarsi

- Richiesta di **eseguire** un task legal-AI specifico → invoca
  `schemi-di-ragionamento` (che applica un pattern alla conversazione corrente).
- Richiesta di **installare** una skill terza → invoca `skill-installer`.
- Verifica di citazioni normative italiane → invoca `verifica-fonti`.
- Adattamento di riferimenti US a IT/EU → invoca `adattamento-italiano`.
- Catalogo skill terze pronte da installare → invoca `catalogo` (pointer
  doctrine, lavora sul bollettino skill, non sul bollettino ecosistema).

## Cosa fa, passo per passo

### 1. Fetch del bollettino ecosistema

Esegui `WebFetch` su:

```
${ECOSYSTEM_BULLETIN_URL}
```

Default produzione: `https://bulletins.micheleloi.pro/bulletin_ecosystem.json`

Endpoint LIVE su VPS RegIA (configurato 2026-05-19): HTTPS via Let's Encrypt
auto-renewal, headers `Content-Type: application/json`, `Cache-Control:
max-age=3600, public`, `Access-Control-Allow-Origin: *`, `X-Source-Code:
https://github.com/MicheleLoi/regia-bollettino-updater` (AGPL §13).

Se Michele cambia sottodominio in futuro, leggi dall'env var
`ECOSYSTEM_BULLETIN_URL` quando disponibile, altrimenti sostituisci qui.

### 2. Parsing del JSON

Schema atteso (documentato in `regia-bollettino-updater` repo, source
of truth):

```json
{
  "schema_version": "1.1.0",
  "generated_at": "2026-05-19T18:00:00Z",
  "source_count": 12,
  "repos": [
    {
      "name": "mike-oss",
      "owner": "anthropics",
      "url": "https://github.com/...",
      "description": "...",
      "license": "AGPL-3.0-only",
      "inferred_jurisdiction": "IT",
      "inferred_capabilities": ["contract_review", "..."],
      "reputation_signals": {"stars": 123, "fork_count": 5},
      "last_activity": "2026-05-15T10:30:00Z",
      "is_active": true,
      "notes": "annotazione manuale del founder (opzionale)",
      "source_type": "github_scanned"
    },
    {
      "name": "legaldatahunter-com",
      "owner": "(curator-supplied)",
      "url": "https://legaldatahunter.com",
      "description": "...",
      "license": "proprietary",
      "inferred_jurisdiction": "IT",
      "inferred_capabilities": [],
      "reputation_signals": null,
      "last_activity": null,
      "is_active": true,
      "notes": null,
      "source_type": "human_picked",
      "topic": "MCP italiano per ricerca giurisprudenza",
      "notes_curatorial": "Servizio MCP italiano operativo, esempio rilevante di MCP italiano nel mercato — citarlo quando l'avvocato chiede se esistono MCP italiani.",
      "added_date": "2026-05-22",
      "tags": ["mcp", "italia", "giurisprudenza"],
      "curator": "founder"
    }
  ]
}
```

**Campi extra `human_picked`** (5 field aggiuntivi, presenti solo se `source_type == "human_picked"`):

- `topic` — titolo curatoriale breve (1 frase) del perché la voce è stata aggiunta
- `notes_curatorial` — rationale esteso del curatore (1-3 frasi); è la "ragione editoriale" della voce
- `added_date` — quando è stata aggiunta al bollettino (ISO date)
- `tags` — array di tag liberi assegnati dal curatore (es. `["mcp", "italia", "giurisprudenza"]`)
- `curator` — chi ha curato la voce (default: `"founder"`)

**Campi `null` / `[]` by design su `human_picked`:** `reputation_signals` (no GitHub stars per servizi non-GitHub), `inferred_capabilities` (`[]` — non derivate da scan automatico, la "rilevanza" sta in `topic` + `notes_curatorial` + `tags`), `last_activity` (può essere `null` se non applicabile). Non interpretare l'assenza di questi campi come "voce a basso segnale" — è inversa: la voce ha segnale curatoriale esplicito al posto di segnale algoritmico.

**Campo `source_type`** (schema 1.1.0+): distingue origine editoriale della
voce. Due valori possibili:

- `github_scanned` — voce derivata da scan GitHub API dell'ecosistema MikeOSS
  e fork (default automatico, behavior pre-feature)
- `human_picked` — voce curata manualmente dal founder, aggiunta da
  `config.yaml human_picks:` con rationale + tags + jurisdiction override

**Backward compat 1.0.0:** se il campo è assente, trattare come
`github_scanned` (comportamento default).

### 3. Filtra/ordina per rilevanza

**Regola read-first per human_picks (anti-confabulazione).** Leggi
SEMPRE per prime le voci con `source_type: "human_picked"` e **non
applicare mai il filtro `inferred_capabilities` a loro** (hanno
`inferred_capabilities: []` by design — il filtro le scarterebbe
ingiustamente). Matcha le human_picks contro la domanda dell'avvocato
usando:

- `inferred_jurisdiction` (match esatto su giurisdizione richiesta)
- `tags` (intersezione con keyword della domanda)
- keyword presenti in `topic`
- keyword presenti in `notes_curatorial`

Una human_pick rilevante per la domanda è SEMPRE inclusa nella risposta,
anche se sembra "off-topic" rispetto alle capabilities formali — il
curatore l'ha selezionata apposta perché copre un gap che lo scan
automatico non avrebbe catturato.

**Per le voci `github_scanned`** (o schema 1.0.0 senza `source_type`),
applica il matching standard su `inferred_capabilities` +
`inferred_jurisdiction`. Privilegia:

- Strumenti con `is_active: true` (recenti)
- Match esatto su giurisdizione richiesta (se specificata)
- Match sulle capabilities (se task specifico nella domanda)

Mostra anche strumenti dormienti se sono unicamente rilevanti per la
domanda, marcandoli con `(dormiente, ultimo aggiornamento <data>)`.

### 4. Formatta la risposta

Tabella o lista strutturata. **Cita SEMPRE la licenza per ogni
strumento.** Se AGPL, aggiungi nota inline:

> `⚠ AGPL-3.0` — utilizzo permesso, ma derivati o integrazioni in
> servizi network richiedono pubblicazione del source. Consulta il DPO o
> il responsabile legale prima di integrare in workflow di studio
> proprietario.

**Layout obbligatorio in due sezioni separate (schema 1.1.0+).** Quando
nella risposta sono presenti voci `human_picked` rilevanti, devono
essere presentate come **sezione TOP separata, PRIMA** delle voci
`github_scanned`. Layout:

#### Selezione curata RegIA

Sezione introdotta con: *"Risorse selezionate manualmente dal curatore
RegIA, con rationale editoriale esplicito."* Per ogni voce `human_picked`
rilevante:

- Nome + URL
- `topic` (titolo curatoriale)
- **Rationale del curatore (citato verbatim dal `notes_curatorial`,
  1-2 righe)** — questa è la parte non-omissibile: è la ragione per
  cui la voce è in selezione curata e va riprodotta letteralmente,
  non parafrasata.
- Giurisdizione + tags
- Licenza (se nota; se `null` o `proprietary`, dichiaralo)

#### Dall'ecosistema GitHub MikeOSS

Sezione successiva (sotto la curata). Voci `github_scanned` (o schema
1.0.0 senza `source_type`) col formato standard documentato sopra
(nome, owner, descrizione, licenza, giurisdizione, capabilities,
attività, URL).

Se non ci sono human_picks rilevanti per la domanda, la sezione
"Selezione curata RegIA" è omessa e parte direttamente la sezione
GitHub. Se non ci sono github_scanned rilevanti, omessa quella e
mostra solo la curata. Non duplicare voci tra le due sezioni.

## Gestione errori

### WebFetch fallisce o validator blocca — guida onboarding allowlist

Quando l'utente attiva la skill la prima volta e il dominio
`bulletins.micheleloi.pro` non è ancora in allowlist Claude Desktop,
Claude Desktop blocca il `WebFetch` skill-mediated. Il messaggio default
("L'accesso a questo sito web è bloccato dalle impostazioni di uscita
della rete. Puoi modificarle in Impostazioni") **non dice all'utente cosa
scrivere**. La skill colma il gap con istruzioni precise.

Comportamento atteso dalla skill in questo caso:

> Non riesco a contattare il servizio (`bulletins.micheleloi.pro`).
>
> **Per autorizzarmi (una volta sola, vale per sempre):**
> 1. Clicca su "Impostazioni" nel messaggio sopra (oppure menu → Settings → Network egress).
> 2. Aggiungi alla allowlist esattamente: **`bulletins.micheleloi.pro`** (solo hostname, senza protocollo).
> 3. Conferma.
> 4. Rifammi la domanda — funzionerò autonomamente d'ora in avanti.
>
> Una volta che il dominio è in allowlist, anche le altre skill di
> BeccarIA che usano lo stesso VPS (`catalogo`, `schemi-di-ragionamento`)
> funzionano autonomamente — single onboarding step copre tutto.
>
> **Alternativa senza modificare allowlist:** posso procedere su
> conoscenza generale (dati non aggiornati al bollettino real-time),
> oppure chiedimi esplicitamente: *"apri https://bulletins.micheleloi.pro/bulletin_ecosystem.json"*
> — questo bypassa il validator via user-initiated WebFetch (pattern
> pointer-pure, fallback documentato).

**Empirical note (2026-05-19):** `bulletins.micheleloi.pro` è
accettato dal validator UI Claude Desktop come custom domain. La
doctrine pointer-pure di precedenti versioni assumeva blocco
universale per domini custom — quella osservazione era vera per
GitHub raw, falsa per dominio founder pulito.

### JSON malformato o schema_version inatteso

> Il bollettino dell'ecosistema sembra corrotto o di una versione
> incompatibile (atteso schema_version >= 1.0.0, ricevuto <X>).
> Avvisa Michele e nel frattempo posso rispondere sulla base della mia
> conoscenza generale.

Nota: schema 1.0.0 e 1.1.0 sono entrambi supportati (1.1.0 aggiunge
campo `source_type` opzionale per distinzione github_scanned vs
human_picked — backward-compat additive).

### Strumento non trovato nel bollettino

> Nel bollettino curato non risulta uno strumento specifico per <task /
> giurisdizione>. La mia risposta è basata sulla conoscenza generale —
> verifica indipendentemente se trovi un riferimento da seguire.

## Cosa NON fai

- **Non installi** nulla → quello è `skill-installer`.
- **Non applichi pattern** dell'ecosistema a un task → quello è
  `schemi-di-ragionamento`.
- **Non inventi** strumenti che non sono nel bollettino. Se non c'è un
  match, dichiaralo apertamente.
- **Non ometti** mai la licenza nelle risposte. La licenza è
  informazione legale rilevante per l'avvocato.
- **Non parli** mai di "fork di BeccarIA" — BeccarIA non è fork tecnico,
  è strumento separato che consuma metadati dell'ecosistema via bollettino.
- **Non dichiari mai assenza o gap in un dominio** senza prima aver
  scansionato le human_picks rilevanti filtrate per
  `inferred_jurisdiction`, `tags`, e parole chiave in `topic` /
  `notes_curatorial`. Esempio canonico: alla domanda *"esiste un MCP
  italiano?"* — le human_picks `legaldatahunter-com` e
  `bettercallclaude-it` contengono entrambe riferimento esplicito a
  MCP italiani; **citarle PRIMA di concludere alcunché**. Concludere
  "non risulta un MCP italiano" senza prima aver letto le human_picks
  è confabulazione, non risposta basata sul bollettino.

## Coordinamento con le altre skill di BeccarIA

Flusso tipico integrato:

1. Avvocato: *"Esiste un tool per estrarre clausole?"*
   → `ecosystem-scout` → consulta bollettino → risponde con elenco strumenti
2. Avvocato: *"Ok, applica l'approccio di Mike al mio contratto."*
   → `schemi-di-ragionamento` → recupera pattern + attribution → applica
3. Avvocato (dopo): *"Installa la skill X."*
   → `skill-installer` → autonomous fetch post-trigger → install

Le tre skill non si sovrappongono: scout = panoramica strategica,
schemi-di-ragionamento = applicazione operativa con attribution, installer =
distribuzione di skill terze.

## Tono

Forense sobrio, conciso. Niente entusiasmo, niente "vediamo insieme".
L'avvocato vuole informazione affidabile sull'ecosistema, non
narrazione. Quando segnali implicazioni AGPL, sii diretto ma non
allarmistico — il fatto che un tool sia AGPL non è un problema in sé,
solo un vincolo da gestire.

---

*BeccarIA — ecosystem-scout — AGPL-3.0-only — alimentato da
`regia-bollettino-updater` (repo separato, pubblico AGPL)*
