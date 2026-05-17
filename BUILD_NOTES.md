# BUILD_NOTES.md — note sullo stato della build e cose da verificare empiricamente

*Riferimento interno per il founder e per chi prende in mano il plugin
dopo questa prima costruzione. Niente di tutto questo va detto
all'avvocato finale.*

---

## Cosa è stato costruito in questa iterazione (fork-and-extend)

**Plugin code (in questo repo):**

- `.claude-plugin/plugin.json` — manifest
- `.claude-plugin/marketplace.json` — registry entry
- `skills/skill-installer/SKILL.md` — **forkato Apache-2.0** da
  `anthropics/claude-for-legal` legal-builder-hub @ 2026-05-17. Adattato:
  config paths rebased da `claude-for-legal/legal-builder-hub/` a
  `mhc-l/`, Step 5.5 role-routing collassato (avvocato = unica
  autorità), nuovo Step 6.5 (hook adattamento italiano per skill IT/EU).
- `skills/skill-installer/references/allowlist.md` — forkato, paths
  rebased.
- `skills/skill-installer/references/freshness.md` — forkato, cosmetic
  rebrand.
- `skills/adattamento-italiano/SKILL.md` — **originale MHC-L, MIT**.
  Hook invocato dallo skill-installer al Step 6.5 per skill `jurisdiction: IT`
  o `EU`.
- `skills/verifica-fonti/SKILL.md` — **originale MHC-L, MIT**. Estesa
  in questa iterazione con lista esplicita registri normativi italiani
  ed europei (Normattiva, Gazzetta Ufficiale, Cassazione CED, Corte
  Cost., Cons. Stato, Garante Privacy, AGCM, CONSOB, Banca d'Italia,
  EUR-Lex, InfoCuria CGUE, IATE) con URL canonici e format di citazione.
- `skills/catalogo/SKILL.md` — **originale MHC-L, MIT**. Riallineata in
  questa iterazione: Passo 3 ora delega a skill-installer + adattamento-italiano,
  no più scrittura file diretta da catalogo; disclaimer globale aggiornato
  per dichiarare framing fork-and-extend; bollettino reframed come
  "ecosystem-monitored" (non "curato founder runtime").
- `skills/catalogo/adaptation_prompt.md` — preservato (IP nostra).

**Dati (in questo repo, stato vuoto):**

- `bollettino.json` — `entries: []`, schema esteso con `italian_adaptation_status`
- `community_validations.json` — `validations: []`

**Licensing:**

- `LICENSE-ANTHROPIC` — Apache License 2.0 integrale (per parti forkate)
- `LICENSE` — MIT (per parti originali nostre), con dichiarazione
  esplicita di quali file sono coperti
- `NOTICE` — attribution Anthropic come richiesto da Apache 2.0

**Documentazione (in questo repo):**

- `README.md` — IT primary + EN secondary, riscritto per nuovo framing
  meta-plugin ecosystem-monitor + dual licensing
- `DISTRIBUZIONE.md` — riscritto come pointer al video walkthrough
  founder-recorded, con fallback testuale 5-click (path corretto:
  Customize → Plugin → "+" → Crea plugin → marketplace URL)
- `BOLLETTINO_FORMAT.md` — threshold policy esplicita aggiunta in cima,
  schema esteso con `italian_adaptation_status`, sezione "Come entra una
  voce" riscritta per riflettere routine completamente automatica
  (no più founder approval runtime)

**Fuori da questo repo, in MHC-Work:**

- `.claude/skills/bollettino-research/SKILL.md` — routine completamente
  automatica con threshold policy in cima, founder out of runtime loop
- `notes/methodology/bollettino_research_routine_20260517.md` — design
  note della routine

---

## Findings empirici cristallizzati nel decision memo 2026-05-17

### Confermato
- **`legal-builder-hub` NON è marketplace-installable su Pro standard.**
  Il marketplace ufficiale Anthropic ha solo il plugin `legal` consolidato
  (9 comandi US-centric). `claude-for-legal` repo umbrella (12+ plugin
  separati, incluso `legal-builder-hub`) NON è disponibile via marketplace
  browsing. Va installato manualmente come custom plugin.
- **License Apache-2.0** su `claude-for-legal` → fork legittimo possibile,
  con obblighi NOTICE + preservation header. Implementato in questa
  iterazione.
- **Install path su Claude Desktop Pro:**
  `Customize → Plugin → "+" → Crea plugin → Aggiungi marketplace [URL GitHub]`.
  5 click totali.
- **Friction UX confermata:** il bottone "Crea plugin" è il path
  counter-intuitivo per "aggiungere marketplace esistente". Va segnalato
  esplicitamente all'avvocato (vedi DISTRIBUZIONE.md).
- **Video walkthrough founder-recorded** è asset primario di
  distribuzione (sostituisce gli `[SCREENSHOT: ...]` marker della v1
  di DISTRIBUZIONE.md).

### Decisione architetturale ratificata
**Opzione B-fork over Opzione A-standalone.** Forkato `legal-builder-hub`
sotto Apache-2.0 e aggiunto sopra il nostro layer italiano. Rationale:
le 2 capability uniche di legal-builder-hub (security gate strutturato
+ extension API) sono il valore primario di Anthropic engineering che
non pareggeremmo in 6 mesi — vale fork + attribution. Vedi PDL
`pdl_meta_skill_vs_legal_builder_hub_20260517.md` in MHC-Work per
overlap analysis.

### Tutte le 7 Q del decision memo restano risolte
Vedi `notes/pdl/pdl_meta_skill_decisions_20260517.md` in MHC-Work.

---

## Cose NON verificate empiricamente

### 1. Format `plugin.json` su Claude Desktop reale

Il manifest è modellato sul plugin `legal` ufficiale Anthropic installato
in `~/.claude/plugins/cache/knowledge-work-plugins/legal/1.0.0/.claude-plugin/plugin.json`.
**Non verificato** che Claude Desktop accetti i campi extra (`license`,
`homepage`, `repository`).

**Test da fare:** installare il plugin via "Crea plugin → Aggiungi
marketplace", verificare che non vada in errore.

### 2. Format `marketplace.json` su Claude Desktop reale

Lo schema usato è plausibile ma non direttamente derivato da un esempio
Anthropic ufficiale.

**Test da fare:** stesso install test del manifest. Issue note GitHub
note nel PDL: `claude-code#28337`, `#40414`.

### 3. `WebFetch` dentro la sandbox cowork

La skill `catalogo` assume `WebFetch` su `raw.githubusercontent.com/...`.
Lo `skill-installer` forkato assume `WebFetch` su qualsiasi repo URL
fornito. Possibili scenari (A: libero, B: whitelist Anthropic, C:
bloccato) discussi in v1 — restano da verificare empiricamente.

### 4. Scrittura su `~/.claude/plugins/config/mhc-l/`

Lo `skill-installer` forkato scrive in
`~/.claude/plugins/config/mhc-l/installed_skills/<nome>/`,
`~/.claude/plugins/config/mhc-l/install-log.yaml`, eventualmente
`~/.claude/plugins/config/mhc-l/state.json` e `allowlist.yaml`.

**Non verificato** che la sandbox cowork permetta scrittura su questi
path. Su upstream legal-builder-hub gira come plugin standalone su
Claude Code, dove la scrittura è scontata; in cowork sandbox potrebbe
non esserlo.

### 5. Subagent read-only per Steps 2-4 dello skill-installer

Lo `skill-installer` upstream richiede un subagent read-only (Read +
WebFetch + Glob only) in restrictive mode. **Non verificato** se la
sandbox cowork espone capability per spawnare subagent con tool
restrictions. Possibile soft-degrade a "permissive mode only" se la
sandbox non supporta subagent.

### 6. Auto-suggerimento `verifica-fonti` dopo skill IT/EU

Resta open dal v1 (Q2.c PDL). Istruzione comportamentale non garantita
al 100% — Claude può saltare in conversazioni lunghe. Da testare con
2-3 tester.

### 7. Adattamento italiano: qualità sostanziale

`adaptation_prompt.md` mappa solida ma non esaustiva. **Test sostanziale
richiesto**: prendere 3-5 skill da `_tmp_cfl_survey/`, farle adattare
da Claude usando `adaptation_prompt.md` come prompt, e far review a
un avvocato italiano consultante.

### 8. Routine `bollettino-research`: end-to-end test

La routine in MHC-Work è completa ma **non è stata mai eseguita
end-to-end** in modalità auto-commit + auto-push. Da testare:
- GitHub API query (rate limit, token, parsing)
- Threshold policy filtra correttamente
- Auto-commit + auto-push funzionano (richiede `gh auth status` verde,
  remote configurato)
- Decision_log entry scritta correttamente

### 9. Skill-installer integration: hook chain catalogo → skill-installer → adattamento-italiano → skill-installer

La nuova architettura chain-of-skills è scritta ma non testata
end-to-end. Casi da verificare:
- Lo `skill-installer` riconosce di dover invocare `adattamento-italiano`
  basandosi su `bollettino_entry.jurisdiction`
- Il return value di `adattamento-italiano` viene letto correttamente
  dallo `skill-installer` e usato per sovrascrivere il SKILL.md
- Se `adattamento-italiano` ritorna `approved: false`, lo `skill-installer`
  effettivamente non scrive nulla

### 10. Skills-qa: gap da chiudere

Lo `skill-installer` forkato fa riferimento a `skills-qa` (Step 5 della
sua sequenza upstream). **Quel skill upstream non è stato forkato in
questa iterazione** — la nostra versione collassa lo step in
"heuristic scan inline" con findings surfaced. Debt: forkare anche
`skills-qa` da `_tmp_cfl_survey/legal-builder-hub/skills/skills-qa/`
in una iterazione successiva per chiudere il gap completamente. Per
ora il fork è funzionalmente self-contained ma sottopara su questo
step rispetto all'upstream.

### 11. Sottomissione al marketplace ufficiale Anthropic

Decisione differita post-2-3 cicli di bollettino mensile e feedback
tester. Confermato dal decision memo Q5.

---

## Punti dove ho fatto scelte ragionevoli non esplicitamente nel plan

### A. Come identificare commit SHA upstream

Il NOTICE e i commenti di provenance dicono "2026-05-17 snapshot"
perché `_tmp_cfl_survey/` è una copia locale del repo upstream senza
metadata `.git/` evidente. Se il founder ha lo SHA del commit da cui
ha cloncato `claude-for-legal`, aggiornare i commenti di provenance
sostituendo "2026-05-17 snapshot" con `<commit-sha>`.

### B. Gestione dello stato `italian_adaptation_status: ready`

Il plan dice "ready = adattamento template generato e validato almeno
una volta". Ho lasciato l'interpretazione operativa generica: la
promozione `pending → ready` può avvenire via `community_validations.json`
quando ≥1 avvocato approva un adattamento, o via flag esplicito founder.
Il meccanismo di promozione esatto non è ancora implementato — è uno
step della prossima iterazione (post-primo install).

### C. Skill-installer Step 6.5: posizione nel workflow

L'ho messo **dopo** Step 6 (approva install della skill originale) e
**prima** di Step 7 (install vero). Razionale: l'avvocato dà prima
l'OK al fatto di considerare l'installazione (Step 6, basato su raw
SKILL.md + trust check), poi dà un secondo OK sul contenuto adattato
(Step 6.5). Due conferme distinte perché due decisioni distinte. Non
testato empiricamente — possibile che l'UX sia pesante e vada
collassato in un'unica conferma con visualizzazione "adattato +
originale" affiancati.

### D. Skills-qa: degraded vs forked

Vedi punto 10 sopra. Ho scelto degraded (heuristic scan inline) invece
di forkare anche `skills-qa` perché il plan dice "forkare solo i pezzi
rilevanti" e `skills-qa` è una skill consistente da forkare anche lei,
fuori scope di questa iterazione. Resta debt esplicito.

---

## Checklist di pre-rilascio (test empirici)

Prima di considerare il plugin "pubblicabile per primo gruppo di avvocati
tester", servono questi test (revisionati post-fork):

- [ ] **Install via "Crea plugin → Aggiungi marketplace [URL]"** funziona
  su Claude Desktop Pro reale (5-click path).
- [ ] **Install via "Upload .zip" di fallback** funziona se l'altro fallisce.
- [ ] **`mostrami il catalogo`** attiva la skill `catalogo` e mostra
  disclaimer (nuovo framing fork-and-extend) + catalogo vuoto.
- [ ] **Disclaimer mostrato una sola volta** in sessioni successive nella
  stessa installazione del plugin (state globale + fallback per-cartella).
- [ ] **Bollettino non-vuoto** (popola manualmente 2-3 entry per test):
  - Pannello "Novità" mostra le voci correttamente formattate
  - Stelle e tendenze calcolate
  - `founder_disclaimer` per-skill visibile accanto a `[Installa]`
  - `italian_adaptation_status` visualizzato (es. badge "Adattamento:
    pending / ready / stale")
- [ ] **Click `[Installa]` su skill IT/EU**: catalogo invoca skill-installer
  → skill-installer fa allowlist + fetch + raw display + trust check →
  invoca adattamento-italiano (Step 6.5) → adattamento-italiano genera
  proposta + pre-flight verifica-fonti → mostra all'avvocato → su
  Approva, skill-installer scrive l'adattata.
- [ ] **Click `[Installa]` su skill `other` o `none`**: catalogo invoca
  skill-installer → install diretto senza Step 6.5 (no hook adattamento).
- [ ] **REFUSE verdict dello skill-installer**: nessun install prompt,
  output verbatim, stop.
- [ ] **Annulla in fase di adattamento**: skill-installer aborta, niente
  scrittura.
- [ ] **Output di skill IT/EU** triggera auto-suggest `verifica-fonti`
  (test su 5+ output per misurare skip rate).
- [ ] **`verifica-fonti` su testo con citazioni miste** produce rapporto
  corretto con citazioni dai registri authoritative italiani/europei
  (Normattiva, EUR-Lex, ecc.) formattate come da nuovo schema.
- [ ] **Routine `bollettino-research`** end-to-end: GitHub API search →
  threshold policy applicata → auto-commit + push → decision_log entry.
- [ ] **Aggiornamento bollettino via "Update"** del plugin in cowork
  pulla nuove voci correttamente.

Una volta verde su tutto sopra, il plugin è in stato "ready for first
3-5 tester avvocati" — non ancora "general availability".

---

*BUILD_NOTES.md — v2.0.0 — 2026-05-17 — Michele Loi*
