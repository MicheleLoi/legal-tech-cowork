# BOLLETTINO_HISTORY.md

Audit trail delle esecuzioni della routine `bollettino-research`.
Ogni entry documenta: voci aggiunte, candidate scartate, SHA commit.

---

## 2026-05-17

- **2 voci aggiunte** (id: `anthropics-knowledge-work-legal`, `terminalskills-contract-review`)
- **Candidati scartati:** ~300+ esaminati, principali motivi di esclusione:
  - Nessuna license OSS valida (la categoria più larga: ~60% dei candidati)
  - Schema mancante — no `SKILL.md` né `plugin.json` riconoscibile (platform applicative complete come OpenContracts, kipeum86/contract-review-agent)
  - Filtro spam: publisher individuale anonimo <100 stars (es. mukul975/Privacy-Data-Protection-Skills 86★, frank-luongt/faos-skills-marketplace 18★)
  - Stars <50 con publisher individuale (decine di repo topic:claude-skill)
  - Formato plugin non compatibile MHC-L (CoWork-OS `cowork.plugin.json` è proprietario CoWork-OS)
  - Rilevanza IT-only esclusa: diritto coreano (reidrockhind539/korean-privacy-terms), UAE (harvey-specter-uae), India/Sri Lanka (vari repo)
  - US-only senza segnali EU/IT: Tennessee law (tn-lawmaster-ai), Delaware, FRCP
- **Commit:** bca8a96
- **Note:** Primo run della routine. Ecosistema skills legali ancora nascente; la maggior parte dei candidati sono applicazioni complete (no skill installabile) o repos individuali senza license. anthropics/knowledge-work-plugins è l'unico auto-pass reputation verificato in questo run.
