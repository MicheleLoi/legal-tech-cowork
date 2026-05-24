# BeccarIA — Skill Claude per l'avvocato italiano nell'ecosistema legal-AI open source

[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)
[![Plus: MIT](https://img.shields.io/badge/Plus-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Plus: Apache-2.0](https://img.shields.io/badge/Plus-Apache--2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)

*Plugin per Claude cowork. Sei skill per l'avvocato italiano. Posizionato
come abilitatore dentro l'ecosistema legal-AI open source italiano: rende
usabili gli strumenti dell'ecosistema (Mike e fork nazionali) senza
sostituirli, e aggiunge la verifica formale delle citazioni normative
italiane ed europee che l'ecosistema non copre nativamente.*

> **Posizione strategica.** BeccarIA è uno dei tre prodotti del brand
> ombrello **RegIA**, insieme a **MHC** (governance framework cross-domain)
> e **Recode IT** (pseudonimizzazione web). I tre moduli sono plug-and-play:
> ciascuno standalone o combinabili a stack. BeccarIA è il modulo dedicato
> all'avvocato italiano che vuole usare seriamente l'ecosistema legal-AI
> open source da Claude.

> **Identità.**  Il nome onora **Cesare Beccaria**, il padre del diritto penale moderno
> italiano, e segna la promozione del plugin a prodotto autonomo (non più
> sotto-componente di MHC-L). Il rename arriva con il bump major a
> v4.0.0.

---

## Le sei skill, in una riga ciascuna

1. **`verifica-fonti`** *(default)* — controlla che le citazioni normative
   e giurisprudenziali italiane ed europee in un testo siano formalmente
   corrette, plausibili e coerenti. È la skill always-on, copre l'80% dei
   bisogni dell'avvocato.

2. **`catalogo`** *(modalità avanzata, pointer)* — gateway al bollettino
   curato di skill legal-tech italiane terze. Pointer-only: suggerisce
   all'avvocato l'URL del bollettino e il fraseggio per chiederne
   l'apertura a Claude (niente fetch ricorrente in background).

3. **`skill-installer`** *(modalità avanzata, autonomous post-trigger)* —
   dopo che l'avvocato sceglie esplicitamente una skill terza, fa
   `WebFetch` autonomous puntuale del `SKILL.md` candidato, applica
   silenziosamente cinque controlli di sicurezza (allowlist, tier, license,
   trust, freshness), e scrive i file in `installed_skills/`.

4. **`adattamento-italiano`** *(modalità avanzata)* — adatta al volo prompt
   di skill terze inglesi a linguaggio giuridico italiano, su richiesta
   esplicita dell'avvocato.

5. **`ecosystem-scout`** *(nuova in 4.0.0)* — panoramica intelligente
   dell'ecosistema legal-AI open source (Mike, fork nazionali, altri tool).
   Risponde a domande del tipo *"esiste un tool open source per estrarre
   clausole?"* citando licenza, giurisdizione e attività di ogni
   strumento suggerito. Alimentato dal bollettino ecosystem del VPS
   BeccarIA.

6. **`pattern-extractor`** *(nuova in 4.0.0)* — applica pattern di
   prompt/approccio derivati dall'ecosistema AGPL alla conversazione
   corrente, con **attribution AGPL obbligatoria** all'inizio della
   risposta. Mai inventa attribuzioni: se nel bollettino non c'è un
   pattern per il task, lo dichiara e propone alternative.

---

## Cosa cambia in v4.0.0

- Identità autonoma sotto **RegIA** (non più sotto-componente MHC-L).
- Due skill nuove (`ecosystem-scout`, `pattern-extractor`) che integrano
  BeccarIA nell'ecosistema legal-AI open source italiano via bollettini
  JSON curati pubblicati dal VPS RegIA.
- Licenza per i nuovi componenti BeccarIA: **GNU AGPL-3.0**, per coerenza
  con l'ecosistema (Mike e fork sono AGPL). Vedi sezione "Licenza" sotto.
- Doctrine **fetch autonomous post-trigger esplicito su VPS BeccarIA**
  (`bulletins.micheleloi.pro`) per tutte le skill modalità avanzata
  (`catalogo`, `skill-installer`, `ecosystem-scout`, `pattern-extractor`).
  Single onboarding step: aggiungere `bulletins.micheleloi.pro`
  all'allowlist egress di Claude Desktop. Fallback pointer-pure
  documentato in ogni `SKILL.md` per casi di policy di rete restrittiva
  (allowlist non modificabile, enterprise lockdown).

## Come si usa in pratica

**Default workflow.** L'avvocato chiede a Claude qualcosa che cita
normativa o giurisprudenza italiana. Quando Claude risponde, l'avvocato
passa il testo a `verifica-fonti`:

> *"Controlla le citazioni di questa risposta."*

Il rapporto segnala formato anomalo, numerazione implausibile, coerenza
contestuale debole, possibili invenzioni — con suggerimento di verifica
sul registro autorevole.

**Workflow ecosistema.** L'avvocato chiede:

> *"Esiste un tool open source per redazione contratti?"*

`ecosystem-scout` si attiva, consulta il bollettino, risponde con un
elenco di strumenti rilevanti (nome, owner, licenza esplicita, giurisdizione
inferita, attività). Se uno strumento è AGPL, segnala le implicazioni
per studio legale.

> *"Ok, applica l'approccio di [strumento] al mio NDA."*

`pattern-extractor` si attiva, consulta il bollettino pattern, espone
l'**attribution prefix obbligatorio**:

> Sto applicando un approccio derivato da **[source_repo]** ([owner],
> **AGPL-3.0**). [Nota sul perché questo pattern è appropriato].

Poi applica il pattern al task. Mai senza attribution.

## Cosa NON fa (in modalità default)

- **Non cura skill terze automaticamente.** In modalità default il plugin
  espone solo `verifica-fonti`. Per la curation di skill terze italiane
  c'è la modalità avanzata opt-in (`catalogo` + `skill-installer` +
  `adattamento-italiano`).
- **Non traduce skill upstream in italiano automaticamente.** Claude
  risponde italiano nativo se gli parli italiano. L'adattamento esplicito
  è opt-in via `adattamento-italiano`.
- **Non garantisce la correttezza sostanziale.** Una citazione formalmente
  corretta può comunque essere inapplicabile al caso concreto.
- **Non consulta i registri normativi live.** Il controllo è basato su
  formato, plausibilità numerica, coerenza testuale e knowledge
  interna — i link ai registri authoritative indirizzano l'avvocato alla
  verifica, non la sostituiscono.
- **Non inventa attribuzioni AGPL.** `pattern-extractor` espone
  l'attribution solo se il pattern è stato effettivamente recuperato dal
  backend; altrimenti dichiara apertamente l'assenza e procede con
  approccio generale di Claude senza attribuire all'ecosistema.

## Modalità avanzata (opt-in, sei skill, niente setup di rete)

Il plugin opera a **due livelli**, e la maggior parte degli avvocati usa
solo il primo:

- **Default.** Una skill, `verifica-fonti`. Niente latenza extra, niente
  chiamate di rete, niente curation di skill terze.
- **Avanzato (opt-in).** L'avvocato che vuole esplorare l'ecosistema
  italiano-curato + l'ecosistema globale AGPL invoca esplicitamente una
  delle skill ecosistema: `catalogo` (per skill terze pronte da
  installare), `ecosystem-scout` (per panoramica strumenti open source),
  `pattern-extractor` (per applicare pattern dell'ecosistema con
  attribution).

**Come si attiva.** L'avvocato dice a Claude:

- *"Apri il bollettino delle skill italiane"* → `catalogo`
- *"Che tool open source esistono per [task]?"* → `ecosystem-scout`
- *"Applica l'approccio di [strumento] al mio contratto"* →
  `pattern-extractor`
- *"Installa skill X"* (dopo `catalogo`) → `skill-installer`

Niente di tutto questo è automatico: senza una richiesta esplicita la
pipeline avanzata resta inerte e il plugin si comporta come single-skill
default.

**Trade-off onesto.** La modalità avanzata ha latenza più alta
(adattamento + fetch bollettini richiede chiamate aggiuntive). È pensata
per power-user che accettano questo costo in cambio di curation italiana
+ accesso strutturato all'ecosistema globale. Per chi vuole solo
verificare le citazioni di un atto, il default basta e avanza.

## Sicurezza + privacy fetch

Il plugin opera senza configurazione di rete preventiva in Claude Desktop:

- **`catalogo` (skill terze) — pointer.** Il plugin suggerisce l'URL del
  bollettino e il fraseggio per chiederne l'apertura a Claude.
  Tu controlli quando e cosa.
- **`ecosystem-scout`, `pattern-extractor`, `skill-installer` —
  fetch puntuale solo dopo un tuo trigger esplicito** (richiesta di
  installazione, domanda sull'ecosistema, richiesta di applicare un
  pattern). Niente polling background. Niente fetch senza una tua azione
  cosciente.

Fallback pointer-pure documentato in ogni `SKILL.md` per casi di policy
di rete restrittiva (allowlist non modificabile, enterprise lockdown):
la skill ricalibra a suggerire il fraseggio per chiedere a Claude di
aprire l'URL come azione utente (`WebFetch` user-initiated, bypassa il
validator skill-mediated).

## Installazione

Due click guidati dentro Claude Code Desktop:

```text
/plugin marketplace add MicheleLoi/legal-tech-cowork
/plugin install beccaria@legal-tech-cowork
```

- **Documentazione canonica Anthropic in italiano:** [Trova e installa plugin](https://code.claude.com/docs/it/discover-plugins).
- **Guida step-by-step pensata per l'avvocato non-tecnico** (5 click, screenshot, troubleshooting, percorso alternativo zip locale): **[DISTRIBUZIONE.md](./DISTRIBUZIONE.md)**.
- **Vuoi provare `verifica-fonti` con OpenAI Codex invece che con Claude?** Percorso sperimentale per avvocati: **[CODEX_INSTALL.md](./CODEX_INSTALL.md)**.

Prima volta con Claude Code Desktop? Vedi [code.claude.com/docs/it/desktop](https://code.claude.com/docs/it/desktop).

## Licenza

BeccarIA usa un modello **multi-license** con tre componenti:

1. **AGPL-3.0** — le due skill nuove di v4.0.0 che partecipano
   nell'ecosistema legal-AI open source AGPL:
   `beccaria/skills/ecosystem-scout/` e
   `beccaria/skills/pattern-extractor/`. Vedi `LICENSE-AGPL`.

2. **MIT** — gli altri componenti BeccarIA originali (
   `beccaria/skills/verifica-fonti/`,
   `beccaria/skills/adattamento-italiano/`,
   `beccaria/skills/catalogo/adaptation_prompt.md`,
   `bollettino.json`, documentazione di progetto). Vedi `LICENSE`.

3. **Apache-2.0** — i file forkati da `anthropics/claude-for-legal`
   (l'upstream legal-builder-hub plugin). Vedi `LICENSE-ANTHROPIC` per il
   testo Apache e `NOTICE` per l'attribuzione.

Ogni file source è la fonte authoritative della propria licenza: cerca
l'header `SPDX-License-Identifier:` (quando presente) o la mappatura
per-cartella documentata in `LICENSE-AGPL`.

### Perché AGPL per le due skill nuove

Le skill `ecosystem-scout` e `pattern-extractor` servono pattern e
metadati derivati da repo AGPL (Mike e fork). Per coerenza con
l'ecosistema, sono distribuite sotto la stessa licenza che governa
l'ecosistema. La parte di BeccarIA che non interagisce con codice/dati
AGPL (verifica-fonti, adattamento-italiano, catalogo adaptation prompt)
resta MIT come prima.

## Contribuire

- **PR su `verifica-fonti`** (pattern di citazione mancanti, registri
  normativi da aggiungere, plausibilità numeriche affinate) → benvenute.
- **PR su `ecosystem-scout` o `pattern-extractor`** (formattazione output,
  schema mapping, gestione errori) → benvenute, contributi cadono sotto
  AGPL-3.0 per coerenza.
- **PR su documentazione** → benvenute.
- **Issue** per segnalare false positive / false negative del controllo,
  o nuovi pattern di citazione → benvenute.

Prima del primo PR sostanziale non-founder accettato verrà configurato
un CLA via `cla-assistant.io` (preserva l'opzione di dual licensing
futuro per chi vorrà estensioni commerciali).

---

# BeccarIA — Italian and EU legal-citation verification (English summary)

*Claude cowork plugin. Six skills for the Italian lawyer. Positioned as
an enabler within the Italian legal-AI open-source ecosystem: makes
ecosystem tools (Mike and national forks) usable for the Italian lawyer
via Claude, without replacing them, and adds formal verification of
Italian and EU legal citations that the ecosystem does not natively
cover.*

The plugin is one of three products under the **RegIA** umbrella brand,
alongside **MHC** (cross-domain governance framework) and **Recode IT**
(web-based pseudonymization). Plug-and-play modules: each standalone or
combined.

**Default workflow:** the lawyer calls `verifica-fonti` to verify Italian
and EU legal references in a text. The 2.x meta-plugin scope (skill
curation, installer, on-the-fly Italian adaptation) is no longer
always-on: it has been refactored as an **advanced opt-in** mode
(2026-05-18). The default user experience is the single
`verifica-fonti` skill — fast, no LLM adapter calls, no network.

**Ecosystem skills (new in v4.0.0):** `ecosystem-scout` provides an
intelligent overview of the legal-AI open-source ecosystem (Mike, forks)
via a curated JSON bulletin published by the RegIA VPS. `pattern-extractor`
applies patterns from the ecosystem to the current conversation, with
**mandatory AGPL attribution prefix** at the start of every applied
response. Never invents attributions: if the bulletin has no matching
pattern, declares it openly.

**Fetch posture:** all advanced-mode skills (`catalogo`,
`skill-installer`, `ecosystem-scout`, `pattern-extractor`) perform
autonomous `WebFetch` on the BeccarIA VPS (`bulletins.micheleloi.pro`)
**only after an explicit lawyer trigger** (open catalog, install
request, ecosystem question, pattern application). No background
polling, no network call without a conscious lawyer action. Single
onboarding step: the lawyer adds `bulletins.micheleloi.pro` to Claude
Desktop's egress allowlist (Settings → Network egress) — covers all
VPS-based skills simultaneously. Documented pointer-pure fallback in
every `SKILL.md` for restrictive network policies (non-configurable
allowlist, enterprise lockdown): the skill recalibrates to suggest the
user-initiated `WebFetch` phrasing instead.

**Licensing:** multi-license repository — AGPL-3.0 for new ecosystem
skills, MIT for other BeccarIA original parts, Apache-2.0 for components
forked from `anthropics/claude-for-legal`.
