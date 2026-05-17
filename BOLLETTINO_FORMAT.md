# Formato `bollettino.json`

*Riferimento per chi vuole capire la struttura del catalogo o (raramente)
proporre una voce manuale via PR. La via primaria di ingresso è la
**routine di ricerca automatica** eseguita mensilmente in MHC-Work
(`bollettino-research`), che applica la **threshold policy** sotto e
auto-committa + auto-pusha — founder fuori dal runtime loop.*

---

## Threshold policy (cristallizzazione del giudizio editoriale)

Una voce candidate passa al bollettino **solo se TUTTE** le condizioni
sotto sono soddisfatte. La policy è la versione operativa del giudizio
del founder: una volta scritta, la routine la applica meccanicamente
senza richiedere conferma runtime.

```
License OSS obbligatoria:
  Apache-2.0 | MIT | BSD-3-Clause | BSD-2-Clause | MPL-2.0
  (no proprietari, no "no license", no GPL viral incompatibile con MIT)

Reputation minima (almeno UNA delle seguenti):
  - Publisher Anthropic-official (auto-pass)
  - Stars ≥ 50 AND last commit ≤ 180 giorni
  - Publisher in whitelist esplicita
  - Contributors ≥ 2 (no sole-dev senza segnali aggiuntivi)

Filtro spam:
  NOT publisher individuale anonimo sotto 100 stars

Schema riconoscibile:
  SKILL.md o plugin.json valido (no README-only marketing)

Last commit:
  ≤ 180 giorni (escluse skill abandonware)

Italian relevance heuristic:
  Esclusione automatica di skill US-only di nicchia (es. immigration USCIS,
  state bar prep US, FRCP-only litigation) — vedi `is_italian_relevant()`
  in bollettino-research routine. Pattern: nome+description contiene
  riferimenti US-specifici hard (es. "Delaware", "FRCP", "USCIS") senza
  riferimenti EU/IT → escludi a monte.
```

E auto-mark `critical_alert: true` per voci esistenti che perdono i
requisiti (repo cancellato, license tolta, abbandono 18+ mesi, publisher
scomparso).

**Sincronia con la routine.** Questa policy esiste in due posti che devono
restare sincronizzati: qui (canonical, public-facing) e nella routine
`MHC-Work/.claude/skills/bollettino-research/SKILL.md` (operativa). Se
divergono è un bug: il giudizio del founder si esprime modificando
**entrambi** in un solo commit.

---

## Schema

```json
{
  "version": "YYYY-MM-DD",
  "last_updated": "YYYY-MM-DD",
  "entries": [ /* lista di voci, schema sotto */ ]
}
```

## Schema di una voce (`entries[N]`)

```json
{
  "id": "kebab-case-univoco",
  "name": "Nome skill",
  "description_it": "Una frase in italiano, sobria, descrive cosa fa.",

  "repo_url": "https://github.com/owner/repo",
  "skill_path": "skills/nome-skill/SKILL.md",

  "area": "commerciale | privacy | lavoro | societario | contenzioso | ip | regolatorio | altro",
  "jurisdiction": "none | IT | EU | other",

  "publisher": {
    "name": "Nome editore (es. Anthropic, Studio Rossi & Associati, ecc.)",
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
    "license": "MIT | Apache-2.0 | GPL-3.0 | other",
    "computed_quality_stars": 0,
    "computed_trend": "in crescita | stabile | in calo"
  },

  "founder_disclaimer": "Una riga in italiano sobrio forense (es. 'Solo prompt in inglese, l'adattamento italiano viene proposto al momento dell'installazione.')",
  "recommended_for": "Breve indicazione di scenario (es. 'Triage rapido di NDA standard tra parti italiane.')",

  "added_to_bollettino": "YYYY-MM-DD",

  "italian_adaptation_status": "pending | ready | stale",

  "critical_alert": false,
  "critical_alert_message": null,
  "critical_alert_severity": null
}
```

## Campi: spiegazione

### `id`
Identificatore univoco, kebab-case. Usato per dedup, lookup, riferimenti
incrociati con `community_validations.json`.

### `name`
Nome visualizzato all'avvocato. Tipicamente il nome originale della skill
(non tradotto in italiano).

### `description_it`
Descrizione **in italiano**, una frase, sobria. Non riassume l'intera
SKILL.md — riassume cosa l'avvocato ottiene se la installa.

### `repo_url` + `skill_path`
Dove vive il codice originale. `repo_url` punta alla root del repo
GitHub; `skill_path` è il percorso relativo dal repo alla `SKILL.md`.
MHC-L compone l'URL raw GitHub per scaricare.

### `area`
Categoria operativa. Una sola; se ne servono due, scegliere la primaria.

### `jurisdiction` ⚠ campo critico
- `none` → skill che non opera su diritto specifico (es. utility tooling)
- `IT` → skill esplicitamente italiana (rara, per ora)
- `EU` → skill su diritto UE (GDPR, AI Act, Reg. macchine, ecc.)
- `other` → skill su diritto straniero (US, UK, ecc.) non adattabile

**Solo `IT` e `EU` triggerano l'auto-suggerimento di `verifica-fonti`.**
Errori di tagging qui producono falsi positivi (verifica suggerita su
output non-IT/EU) o falsi negativi (citazioni IT non controllate). La
routine di ricerca include uno step di review umana del founder su
questo campo specifico.

### `publisher.type`
- `anthropic-official` → repo sotto org `anthropics/*`
- `company` → org riconoscibile (studio legale, vendor, accademia)
- `individual` → singolo sviluppatore
- `academic` → università / ricerca

Influisce sul calcolo delle stelle qualità (vedi sotto).

### `publisher.italian_localized`
`true` se la skill è già nata in italiano (assenza di adattamento
necessario). Raro.

### `reputation.*` (raccolti via GitHub API)
- `stars` — count delle stars
- `last_commit` — data ultimo commit sul branch default
- `commit_frequency_30d` — numero commit ultimi 30 giorni
- `contributors` — count contributori
- `open_issues` — issues aperte
- `closed_issues_30d` — issues chiuse ultimi 30 giorni
- `release_tags` — count tag rilascio
- `license` — license SPDX identifier
- `computed_quality_stars` — risultato algoritmo deterministico (vedi sotto)
- `computed_trend` — risultato analisi growth-rate (vedi sotto)

### `founder_disclaimer` ⚠ campo obbligatorio
Una riga di avviso scritta in italiano sobrio forense. Visibile
all'avvocato accanto al pulsante `[Installa]` nel catalogo. Esempi:

- *"Solo prompt in inglese, l'adattamento italiano viene proposto al
  momento dell'installazione."*
- *"Ultimo aggiornamento 2024, possibile abandonware."*
- *"Skill di ottima qualità ma centrata su Delaware corporate; adattamento
  italiano richiede attenzione."*
- *"Autore individuale, non vetted da organizzazioni note. Valuta prima
  di affidarti in produzione."*

**Vietato:** linguaggio legal-style boilerplate ("AS IS", "no warranty",
"the user assumes all responsibility"). Vietato anche linguaggio
consumer-marketing ("ottima!", "consigliata!", "5 stelle imperdibili").

### `recommended_for`
Una indicazione breve di scenario d'uso. Non un'attestazione di idoneità —
un suggerimento di contesto.

### `italian_adaptation_status`
Tracking dello stato dell'adattamento italiano per questa voce
(rilevante solo se `jurisdiction` è `IT` o `EU`):

- `pending` — voce mai adattata da nessun avvocato (nessun template di
  adattamento riusabile disponibile). Stato di default per voci nuove.
- `ready` — adattamento template generato e validato almeno una volta
  da un avvocato che lo ha approvato; può essere riusato come baseline
  per i prossimi install. (Promozione automatica via `community_validations.json`
  o flag esplicito founder.)
- `stale` — la skill upstream è cambiata dopo l'ultimo adattamento
  validato (cambio commit SHA significativo). Il template è da
  rigenerare alla prossima install.

Per skill con `jurisdiction: none` o `other`, il campo resta `pending`
e non è rilevante (non si adatta).

### `critical_alert` + sotto-campi
Quando `true`, la voce compare nel **Pannello B "Avvisi importanti"** del
catalogo (se l'avvocato ha installato la skill). Usato per patch
sicurezza, bug critici, deprecazioni, abbandono autore, errori scoperti
nel tagging `jurisdiction`.

- `critical_alert_severity`: `critica | alta | media`
- `critical_alert_message`: una frase che dice cosa è successo e cosa
  fare

---

## Algoritmo `computed_quality_stars`

Deterministico, no ML, 1-5 stelle:

- **Base:** 1 stella.
- **+1** se `last_commit` è entro 90 giorni dall'ora corrente.
- **+1** se `contributors` ≥ 2.
- **+1** se `release_tags` ≥ 1.
- **+1** se almeno una di:
  - `publisher.type == "anthropic-official"`
  - `stars >= 50`
  - menzione vetted in pubblicazione legal-tech riconosciuta (campo
    futuro `verified_mention: true`)
- **Cap a 5.**

## Algoritmo `computed_trend`

Su finestra 30 giorni:

- **`in crescita`** se `stars_delta_30d > 0` E `commit_frequency_30d > 0`
- **`in calo`** se `commit_frequency_90d == 0` E `last_commit` > 180gg fa
- **`stabile`** altrimenti

---

## Validazioni di comunità (`community_validations.json`)

File separato. Lo schema:

```json
{
  "version": "YYYY-MM-DD",
  "last_updated": "YYYY-MM-DD",
  "validations": [
    {
      "skill_id": "id-skill-dal-bollettino",
      "validator_id": "anon-hash-fingerprint",
      "validator_visible_name": null,
      "validation_date": "YYYY-MM-DD",
      "context": "Breve descrizione del caso d'uso (es. 'NDA bilaterale per progetto SaaS').",
      "citations_confirmed": [
        "art. 1382 c.c.",
        "D.lgs. 196/2003"
      ],
      "notes": "Eventuali osservazioni dell'avvocato che si è preso il tempo di scriverle (opzionale)."
    }
  ]
}
```

Una skill diventa `community_validated` (badge visibile nel catalogo)
quando ha ≥ 3 validazioni indipendenti (diverso `validator_id`).

Le validazioni arrivano via PR automatica generata dal bot del founder a
partire dal pool opt-in degli avvocati. L'avvocato non vede mai GitHub.

---

## Come entra una voce nel bollettino

**Via primaria — routine automatica mensile (ecosystem-monitor):**

1. La routine `bollettino-research` viene invocata automaticamente
   come **remote agent (routine claude.ai/code)** ogni **17 del mese
   alle 22:40 ora italiana** (cron `40 20 17 * *` UTC; in estate
   = 22:40 CEST, in inverno = 21:40 CET). Routine ID:
   `trig_01XKWAFBLAC4JcLibcf451J7` — log esecuzioni a
   `claude.ai/code/routines/`. Founder può anche invocarla manualmente
   in qualunque momento via Claude Code in MHC-Work.
2. Cerca su GitHub API **broadly** nell'ecosistema legal-tech AI open
   source (topic `claude-skill`, `legal-tech`, etc.; ecosystem keywords
   post-Mike Bommarito; org `anthropics/*` + publisher whitelist se
   popolata).
3. Raccoglie reputation indicators via GitHub API per ogni candidate.
4. **Applica la threshold policy meccanicamente** (vedi sezione in cima
   a questo file). Le voci che passano vanno in `entries`; le scartate
   sono loggate nella decision_log entry.
5. Auto-commit + auto-push del bollettino aggiornato. **Nessuna
   approvazione runtime founder.**
6. Scrive entry sintetica in `MHC-Work/_org/decision_log.md` per audit
   trail.

**Via secondaria — issue su GitHub (avvocato consultante):**

Un avvocato che vuole proporre una skill apre una issue sul repo
`legal-tech-cowork` con il link. La routine `bollettino-research` in
modalità ad-hoc (single-skill review) valuta quella sola voce con la
**stessa identica threshold policy** e auto-committa se passa. Nessuna
deviazione dalla policy per richieste mirate — la policy è il punto
fermo.

---

## Esempio di voce (storico, non in `entries` iniziale)

```json
{
  "id": "anthropic-cfl-nda-review",
  "name": "claude-for-legal nda-review",
  "description_it": "Triage rapido di NDA standard con classificazione GREEN/YELLOW/RED secondo playbook configurato dallo studio.",
  "repo_url": "https://github.com/anthropics/claude-for-legal",
  "skill_path": "commercial-legal/skills/nda-review/SKILL.md",
  "area": "commerciale",
  "jurisdiction": "other",
  "publisher": {
    "name": "Anthropic",
    "type": "anthropic-official",
    "italian_localized": false
  },
  "reputation": {
    "stars": 1247,
    "last_commit": "2026-04-22",
    "commit_frequency_30d": 18,
    "contributors": 12,
    "open_issues": 23,
    "closed_issues_30d": 41,
    "release_tags": 4,
    "license": "MIT",
    "computed_quality_stars": 5,
    "computed_trend": "in crescita"
  },
  "founder_disclaimer": "Playbook US-centric (Delaware, FRCP); l'adattamento italiano richiede attenzione su clausola penale (art. 1382 c.c.) e foro.",
  "recommended_for": "Studi che vogliono partire da una checklist NDA rodata e adattarla al diritto italiano sotto guida.",
  "added_to_bollettino": "2026-05-17",
  "italian_adaptation_status": "pending",
  "critical_alert": false
}
```

*Nota: questo esempio è solo documentale. NON va in `entries` finché la
routine `bollettino-research` non l'ha processato e la threshold policy
non l'ha promosso (in questo caso fittizio passerebbe: license MIT,
Anthropic-official, 1247 stars, schema valido, commit recente — anche
se `jurisdiction: other`, la policy accetta anche `other` se le altre
condizioni passano, perché l'avvocato può comunque installarla in
versione originale senza adattamento).*

---

*BOLLETTINO_FORMAT.md — v1.0.0 — 2026-05-17*
