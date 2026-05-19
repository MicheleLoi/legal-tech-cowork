# SPDX-License-Identifier: AGPL-3.0-only
---
name: ecosystem-scout
description: >
  Panoramica intelligente dell'ecosistema legal-AI open source (Mike e fork
  nazionali). Risponde a domande dell'avvocato su quali strumenti open source
  esistono per task legal-AI, su quali giurisdizioni, con quali licenze.
  Consulta un bollettino aggiornato dal VPS di BeccarIA e segnala
  implicazioni AGPL per studio legale.
---

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
  `pattern-extractor` (che applica un pattern alla conversazione corrente).
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

Default placeholder: `https://api.regia.it/bulletins/bulletin_ecosystem.json`

L'URL definitivo va sostituito quando il VPS è live. Se Michele ha
cambiato il sottodominio, sostituisci qui o leggi dall'env var
`ECOSYSTEM_BULLETIN_URL` quando disponibile.

### 2. Parsing del JSON

Schema atteso (documentato in `regia-bollettino-updater` repo, source
of truth):

```json
{
  "schema_version": "1.0.0",
  "generated_at": "2026-05-19T18:00:00Z",
  "source_count": 12,
  "repos": [
    {
      "name": "...",
      "owner": "...",
      "url": "...",
      "description": "...",
      "license": "AGPL-3.0-only",
      "inferred_jurisdiction": "IT",
      "inferred_capabilities": ["contract_review", "..."],
      "last_activity": "2026-05-15T10:30:00Z",
      "stars": 123,
      "fork_count": 5,
      "is_active": true,
      "notes": "annotazione manuale del founder (opzionale)"
    }
  ]
}
```

### 3. Filtra/ordina per rilevanza

Best-effort matching della domanda dell'avvocato con `inferred_capabilities`
e `inferred_jurisdiction` dei repo nel bollettino. Privilegia:

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

## Gestione errori

### WebFetch fallisce o validator blocca

Comportamento:

> Non riesco a contattare il servizio di monitoring dell'ecosistema
> (`${ECOSYSTEM_BULLETIN_URL}`). Posso comunque rispondere sulla base della
> mia conoscenza generale dell'ecosistema legal-AI open source, ma i dati
> potrebbero non essere aggiornati.
>
> Se vuoi i dati aggiornati ora, puoi chiedermi esplicitamente di aprire
> il bollettino: *"apri ${ECOSYSTEM_BULLETIN_URL}"* — questo bypassa il
> blocco del validator e mi permette di fetcharlo come azione utente.

Pattern di fallback documentato: pointer-pure (lo stesso pattern di
`catalogo` doctrine 3.3.0).

### JSON malformato o schema_version inatteso

> Il bollettino dell'ecosistema sembra corrotto o di una versione
> incompatibile (atteso schema_version >= 1.0.0, ricevuto <X>).
> Avvisa Michele e nel frattempo posso rispondere sulla base della mia
> conoscenza generale.

### Strumento non trovato nel bollettino

> Nel bollettino curato non risulta uno strumento specifico per <task /
> giurisdizione>. La mia risposta è basata sulla conoscenza generale —
> verifica indipendentemente se trovi un riferimento da seguire.

## Cosa NON fai

- **Non installi** nulla → quello è `skill-installer`.
- **Non applichi pattern** dell'ecosistema a un task → quello è
  `pattern-extractor`.
- **Non inventi** strumenti che non sono nel bollettino. Se non c'è un
  match, dichiaralo apertamente.
- **Non ometti** mai la licenza nelle risposte. La licenza è
  informazione legale rilevante per l'avvocato.
- **Non parli** mai di "fork di BeccarIA" — BeccarIA non è fork tecnico,
  è strumento separato che consuma metadati dell'ecosistema via bollettino.

## Coordinamento con le altre skill di BeccarIA

Flusso tipico integrato:

1. Avvocato: *"Esiste un tool per estrarre clausole?"*
   → `ecosystem-scout` → consulta bollettino → risponde con elenco strumenti
2. Avvocato: *"Ok, applica l'approccio di Mike al mio contratto."*
   → `pattern-extractor` → recupera pattern + attribution → applica
3. Avvocato (dopo): *"Installa la skill X."*
   → `skill-installer` → autonomous fetch post-trigger → install

Le tre skill non si sovrappongono: scout = panoramica strategica,
pattern-extractor = applicazione operativa con attribution, installer =
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
