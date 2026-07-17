# BOLLETTINO_HISTORY.md

Audit trail delle esecuzioni della routine `bollettino-research`.
Ogni entry documenta: voci aggiunte, candidate scartate, SHA commit.

---

## 2026-07-17

- **1 voce aggiunta** (id: `apiotrowski-commercial-legal-pl`, tier: 2)
- **Candidati esaminati:** ~350+ su 7 query di ricerca
  - topic:claude-skill × 100 item (su 1.543 totali)
  - topic:claude-cowork-plugin × 12
  - topic:legal-tech × 50 (su 508 totali)
  - "legal AI agent" × 50
  - "lawtech open source" × 50
  - "contract review AI" × 50
  - org:anthropics topic:skill × 0 risultati utili
- **Candidato promosso:**
  - `apiotrowski-afk/commercial-legal-pl` — 132★, Apache-2.0, SKILL.md confermato, last_commit 2026-07-17, release v0.2.1 (20 giugno 2026). Skill per contrattualistica B2B IT polacca + moduli GDPR-RODO. Reputazione: stars ≥ 50 AND last commit ≤ 180 giorni (pass). Spam filter: 132 stars > 100 (pass). Rilevanza IT: EU signal via GDPR-RODO (pass, no marker US-only). Schema: SKILL.md root + .claude-plugin/ (pass). jurisdiction=other (diritto polacco primario, GDPR secondario EU). tier=2 (publisher individuale, non anthropics/).
- **Principali motivi di esclusione per gli altri candidati:**
  - License AGPL-3.0 (viral, fuori lista): `fedec65/bettercallclaude` (33★, Swiss legal), `zeweihan/aiworkdeck` (61★, IDE legale)
  - License assente/NOASSERTION: `vikashjeyaraman/opencouncil-contract-inspector` (151★), `Vaquill-AI/awesome-legaltech` (162★, awesome-list + no license), `jacktotem/anonimizzatore-pdf` (italiano ma NOASSERTION)
  - Rilevanza IT esclusa (US-only): `ahacker-1/cre-acquisition-orchestrator` (92★ Apache-2.0, US CRE senza segnali EU/IT)
  - Diritto non-IT/EU: Wolfgangrush family (14+ repo diritto indiano, MIT, 1-2★), `madfam-org/tezca` (diritto messicano, AGPL), `erjigit7/myizam` (Kyrgyzstan), `jabezpauls/startup-law-skill` (diritto indiano)
  - Stars < 50 + publisher individuale (filtro spam): `JohnDeer-ai/veqtor-mcp` (2★), `eliasreiche/legal-ops-germany` (5★), `kjgdgch65g/nl-rag-qdrant-legal` (6★), `lowtidebuild/contract-review-agent` (39★), `rohasnagpal/AI-Blueprint` (38★)
  - Schema mancante (applicazioni complete, non skill): `zeweihan/aiworkdeck`, `sebastianfoerste/contract-review-eval-harness`, decine di repo AI contract review webapp
  - Massa repo ncreighton/* (WordPress plugin, 0★, no license, topic stuffing)
  - Prossimi alla soglia (da rivalutare): `fedec65/bettercallclaude` (33★ AGPL, license barrier unico blocco — se l'autore aggiungerà MIT/Apache), `lowtidebuild/contract-review-agent` (39★ Apache-2.0, schema da verificare)
- **Commit:** 3c710a9

---

## 2026-06-17

- **0 voci aggiunte** (nessun candidato ha superato la threshold policy in questo run)
- **2 voci esistenti aggiornate** (reputation refresh: stars, last_commit, computed_trend)
  - `anthropics-knowledge-work-legal`: stars 12.267 → 21.026, last_commit 2026-06-17
  - `terminalskills-contract-review`: stars 51 → 76, last_commit 2026-06-11, computed_trend stabile → in crescita
- **Candidati esaminati:** ~350+ distribuiti su 7 query di ricerca (topic:claude-skill × 100 items, topic:claude-cowork-plugin × 11, topic:legal-tech × 50, "legal AI agent" × 14, "lawtech" × 15, "contract review AI" × 20, anthropics/*)
- **Principali motivi di esclusione:**
  - Licenza assente o non ammissibile (NO LICENSE / NOASSERTION / GPL / AGPL / proprietaria): ~70% dei candidati
  - Stars < 50 + publisher individuale, senza segnali reputazionali aggiuntivi (filtro spam attivato)
  - Schema mancante: nessun `SKILL.md` né `plugin.json` installabile (applicazioni complete, non skill)
  - Rilevanza IT esclusa: diritto indiano (Wolfgangrush family: 14 repo), diritto cinese (NEU-ZHA, Lawyer-ray/FachuanHybridSystem 189★ NOASSERTION), coreano (sungjunlee/beopsuny-skill), russo (zarubinphil/themis), cileno (hurricxne/skill-auditoria-ley-21719), UK-only (uk-agents/uk-legal-plugins 7★), messicano (madfam-org/tezca)
  - Contenuto US-specific senza segnali EU/IT: sure-scale/doc-haus (49★ NOASSERTION), ahacker-1/cre-acquisition-orchestrator (76★ Apache-2.0, US CRE)
  - Publisher anthropics/* senza skill legali: anthropics/skills (licenze proprietarie per le skill principali), anthropics/defending-code-reference-harness (code security, non legal), anthropics/launch-your-agent (3★, startup tool)
  - Close call REFUSE: open-agreements/open-agreements (36★ Apache-2.0, SKILL.md multipli confermati, org; REFUSE per stars < 50 e apparent sole developer stevenobiajulu — non supera reputation minimum), zoharbabin/due-diligence-agents (47★ Apache-2.0; REFUSE per assenza di SKILL.md installabile, applicazione Python non skill)
- **Commit:** 58ca1c0
- **Note:** Secondo run della routine. Ecosistema ancora in espansione ma prevalenza di repos spam (topic stuffing con 0 stars) e repos di diritto non-IT/EU. Segnale positivo: crescita organica dei due entry esistenti (knowledge-work-plugins +8.759 stars in 30 giorni; TerminalSkills/skills +25 stars). Candidato più prossimo alla soglia: open-agreements/open-agreements — da rivalutare il prossimo run se raggiunge 50★ o aggiunte contributors.

---

## 2026-05-17

- **2 voci aggiunte** (id: `anthropics-knowledge-work-legal`, `terminalskills-contract-review`)
- **Candidati scartati:** ~300+ esaminati, principali motivi di esclusione:
  - Nessuna license OSS valida (la categoria più larga: ~60% dei candidati)
  - Schema mancante — no `SKILL.md` né `plugin.json` riconoscibile (platform applicative complete come OpenContracts, kipeum86/contract-review-agent)
  - Filtro spam: publisher individuale anonimo <100 stars (es. mukul975/Privacy-Data-Protection-Skills 86★, frank-luongt/faos-skills-marketplace 18★)
  - Stars <50 con publisher individuale (decine di repo topic:claude-skill)
  - Formato plugin non compatibile iuris-it (CoWork-OS `cowork.plugin.json` è proprietario CoWork-OS)
  - Rilevanza IT-only esclusa: diritto coreano (reidrockhind539/korean-privacy-terms), UAE (harvey-specter-uae), India/Sri Lanka (vari repo)
  - US-only senza segnali EU/IT: Tennessee law (tn-lawmaster-ai), Delaware, FRCP
- **Commit:** bca8a96
- **Note:** Primo run della routine. Ecosistema skills legali ancora nascente; la maggior parte dei candidati sono applicazioni complete (no skill installabile) o repos individuali senza license. anthropics/knowledge-work-plugins è l'unico auto-pass reputation verificato in questo run.
