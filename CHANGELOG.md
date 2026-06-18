# Changelog

Tutte le modifiche notabili a questo progetto sono documentate qui.

## v4.3.0 — 2026-06-18 (beta)

### `verifica-fonti` — integrazione Simpliciter MCP (secondo connettore banca dati)

La skill `verifica-fonti` ora riconosce **Simpliciter MCP** come secondo connettore
opzionale a banca dati giuridica di terzi, accanto a BuddaLaw, **innestato nello
stesso schema** (consenso esplicito al primo uso, ricerca a cascata, fallback
automatico ai registri web, provenance esplicita) — non un meccanismo parallelo.

- **Simpliciter** è un connettore **remoto** (endpoint `https://simpliciter.ai/mcp`,
  OAuth nel browser, abbonamento attivo richiesto): strumenti di sola lettura su
  normativa + giurisprudenza italiana, recupero di fonti da riferimento esatto con
  link verificabili. **BuddaLaw** resta forte sulla giurisprudenza di legittimità e
  merito. Chi non ha alcun connettore non vede cambiamenti: la skill usa i registri
  web come prima.
- **Stesse garanzie del primo connettore**: consenso esplicito al primo uso della
  sessione (default = non usare), memoria di sessione senza persistenza su disco,
  suggerimento di pseudonimizzare con Recode IT i dati di causa sensibili, fallback
  ai registri web **non opzionale**.
- **Nomi dei tool MCP di Simpliciter**: placeholder (`mcp__simpliciter__<da-confermare>`)
  finché non confermati al primo collegamento del connettore — nessun nome inventato.

### Coerenza documentazione

`README.md` aggiornato per riflettere il comportamento reale: la skill consulta i
registri pubblici live (WebFetch) e può usare connettori MCP di terzi opzionali e
consent-gated; nessun connettore è incluso o richiesto dal plugin.

## v4.2.1 — 2026-06-04 (beta)

### `verifica-fonti` — integrazione BuddaLaw MCP

La skill `verifica-fonti` ora supporta il connettore BuddaLaw MCP come **fonte
giurisprudenziale italiana opzionale**, con due regole:

1. **Solo se l'avvocato ce l'ha già**. Se nella sessione è disponibile il
   connettore BuddaLaw e l'avvocato lo ha autorizzato in allowlist, la skill
   può interrogarlo. Chi non ha BuddaLaw non vede cambiamenti: la skill
   continua a usare gli stessi registri web di prima (Italgiure CED via Chrome
   MCP, Normattiva, EUR-Lex, ecc.).
2. **Consenso esplicito al primo uso della sessione**. Anche se l'avvocato ha
   BuddaLaw configurato, la skill avvisa la prima volta della sessione che sta
   per instradare la query verso il connettore e chiede conferma. Niente uso
   silenzioso. Su rifiuto: la skill prosegue con i registri web come prima.

### Integrazione tecnica

L'abbonamento BuddaLaw resta dell'avvocato. BeccarIA non ha alcun rapporto
commerciale con BuddaLaw: è una segnalazione tecnica di compatibilità via
protocollo MCP. Se BuddaLaw non trova risultati o restituisce errore/timeout,
la skill esegue automaticamente fallback ai registri web standard senza
chiedere conferma.

### Stato di maturità

Siamo in **beta**. Il pezzo "la skill usa BuddaLaw + fallback web" è stato
validato da un tester (Giuseppe Girlando) con esito positivo su flusso
reale. Il pezzo "la skill chiede consenso al primo uso" (il consent-gate) è
stato scritto e dovrebbe comportarsi come descritto, ma non è ancora stato
osservato all'opera su un avvocato che parte da zero. Se la skill chiede male,
chiede dove non dovrebbe, o non chiede dove dovrebbe — scrivetecelo (via
issue su GitHub o nel gruppo Giuristi AI).
