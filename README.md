# iuris-it — verifica fonti normative italiane ed europee

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

*Plugin per Claude cowork. Una skill, una funzione: controllare che i
riferimenti normativi e giurisprudenziali italiani/europei citati in un
testo corrispondano a documenti reali e siano formalmente coerenti.*

---

## Cosa fa il plugin

Il plugin contiene **una sola skill**: `verifica-fonti`. Il nome
è volutamente conservativo: la skill **non** cerca sentenze su
database online, **non** valuta la correttezza giuridica sostanziale,
**non** aggiorna testi normativi. Fa **una cosa sola**: prende un
testo che contiene citazioni normative italiane o europee e produce
un rapporto formale (formato, plausibilità, coerenza interna,
possibili invenzioni). Per la verifica sostanziale ti indirizza ai
registri authoritative.

Riceve un testo (tipicamente l'output di un'altra skill o una bozza
dell'avvocato) e produce un rapporto di verifica delle citazioni
normative italiane ed europee. Controlla:

- **Formato** — la citazione segue uno dei pattern noti (codici italiani,
  leggi, decreti, sentenze Cassazione, Corte Costituzionale, Consiglio
  di Stato, TAR, regolamenti e direttive UE, sentenze CGUE, CEDU).
- **Plausibilità numerica** — il numero di articolo o di sentenza è
  coerente con i range noti (es. `art. 9999 c.c.` è impossibile, il
  codice civile arriva ad art. 2969).
- **Coerenza interna** — la norma citata corrisponde al contenuto
  descritto (es. `art. 1382 c.c.` come "danno extracontrattuale" è un
  refuso: la clausola penale è 1382, l'extracontrattuale è 2043).
- **Plausibilità storica** — possibili citazioni inventate (numerazioni
  che non esistevano all'epoca della sentenza).

Per ogni segnalazione indirizza ai registri authoritative: **Normattiva,
Gazzetta Ufficiale, Cassazione/Italgiure, Consiglio di Stato e TAR
(Giustizia Amministrativa), Corte Costituzionale, Garante Privacy,
AGCM, CONSOB, Banca d'Italia, EUR-Lex, InfoCuria CGUE, IATE**.

## Come si usa

L'avvocato chiede a Claude qualcosa che cita normativa o giurisprudenza
italiana. Quando Claude risponde, l'avvocato passa il testo a
`verifica-fonti`:

> *"Controlla le citazioni di questa risposta."*

oppure

> *"Passa l'output a verifica-fonti."*

Il rapporto segnala le citazioni con formato anomalo, numerazione
implausibile, coerenza contestuale debole, o possibili invenzioni — con
suggerimento di verifica sul registro autorevole. La verifica sostanziale
finale resta del professionista.

## Perché `verifica-fonti` è il default (e il resto è opt-in)

Versioni precedenti del plugin (2.x) montavano un meta-pattern attivo
di default: catalogo + skill-installer + adattamento italiano al volo
per skill US-centric, tutto sempre on. Ratifica founder 2026-05-18
(strada B), riassunta:

1. **Install runtime troppo lento per il default** — il flusso di
   installazione runtime delle skill terze, anche con security gate,
   era pesante per l'avvocato non-tech all'apertura del plugin.
2. **Non sono avvocato** — la curation editoriale di skill legali terze
   richiede competenza giuridica che resta esterna al progetto: vale
   come servizio aggiuntivo opzionale, non come gating obbligato.
3. **Claude parla già italiano nativo** — non serve "tradurre" una
   skill inglese per usarla: se l'avvocato scrive italiano a Claude,
   le skill Anthropic ufficiali producono output in italiano
   direttamente.

Il valore aggiunto residuo always-on è dove Claude non può supplire
da solo: la verifica puntuale di citazioni normative italiane (formato,
plausibilità, registri authoritative). Quello è `verifica-fonti`, ed è
il default.

Per chi vuole comunque un ecosistema curato di skill terze italiane,
la modalità avanzata (sezione successiva) tiene viva quella pista
come **opt-in attivabile via bollettino** — niente è attivo finché
l'avvocato non lo chiede esplicitamente.

Riferimento decisione: MHC-Work `_org/decision_log.md` voce
"Plugin Cowork mhc-l: ratifica strategica riduzione a verifica-fonti
only" (2026-05-18, plugin allora denominato `mhc-l`, rinominato
`iuris-it` 2026-05-19) + raffinamento founder in-session post UX test
("default invariato + bollettino opt-in").

## Cosa NON fa (in modalità default)

- **Non cura skill terze automaticamente.** In modalità default il plugin
  non propone, scarica o installa skill esterne. Per la curation di skill
  terze italiane c'è la modalità avanzata opt-in descritta sotto.
- **Non traduce skill upstream in italiano automaticamente.** Claude
  risponde italiano nativo se gli parli italiano. L'adattamento italiano
  esplicito di skill terze è anch'esso parte della modalità avanzata
  opt-in, non del default.
- **Non garantisce la correttezza sostanziale.** Una citazione formalmente
  corretta può comunque essere inapplicabile al caso concreto.
- **Non consulta i registri live.** Il controllo è basato su formato,
  plausibilità numerica, coerenza testuale e knowledge interna — i link
  ai registri authoritative indirizzano l'avvocato alla verifica, non la
  sostituiscono.

## Modalità avanzata (opt-in)

Il plugin opera a **due livelli**, e la maggior parte degli avvocati
usa solo il primo:

- **Default.** Una skill, `verifica-fonti`. Niente latenza extra,
  niente chiamate di rete, niente curation di skill terze. È il caso
  d'uso che copre l'80% dei bisogni di verifica formale delle citazioni
  normative.
- **Avanzato (opt-in).** L'avvocato che vuole esplorare un ecosistema
  di skill legal-tech italiano-curate invoca esplicitamente la skill
  **`catalogo`** (per esempio chiedendo *"apri il bollettino delle
  skill italiane"*). Da quel punto si attiva una pipeline orchestrata
  via **trigger semantici, niente setup utente richiesto**:
  - `catalogo` punta all'avvocato l'URL del bollettino curato e gli
    suggerisce il fraseggio per chiedere a Claude di aprirlo
    (l'apertura avviene via `WebFetch` standard di Claude su
    richiesta esplicita dell'avvocato, non in background dalla skill),
  - lo `skill-installer` auditeggia la skill scelta applicando
    silenziosamente i 5 controlli di sicurezza (allowlist licenze,
    tier, heuristic, license, freshness) sul `SKILL.md` letto in
    contesto via la stessa richiesta esplicita;
  - l'installazione effettiva la fa l'avvocato manualmente tramite
    Claude Desktop → Customize → Plugin → Crea plugin → URL della
    skill → Add;
  - l'`adattamento-italiano` — su richiesta esplicita successiva —
    adatta al volo il prompt della skill terza al linguaggio
    giuridico italiano se necessario.

**Come si attiva.** L'avvocato dice a Claude qualcosa come *"apri il
bollettino delle skill italiane"*, *"mostrami il catalogo"*, *"che
skill posso installare?"*. Niente di tutto questo è automatico: senza
una richiesta esplicita la pipeline avanzata resta inerte e il plugin
si comporta come il singolo-skill default.

**Trade-off onesto.** La modalità avanzata ha latenza più alta
(adattamento richiede chiamate LLM aggiuntive). È pensata per
power-user che accettano questo costo in cambio di curation
italiana. Per chi vuole solo verificare le citazioni di un atto, il
default basta e avanza.

## Doctrine pointer (3.3.0)

A partire dalla versione 3.3.0 il plugin adotta la **doctrine
pointer**: le skill avanzate (`catalogo`, `skill-installer`) **non
fanno `WebFetch` autonomous in background**. Quando serve leggere il
bollettino o il `SKILL.md` di una skill terza candidata, la skill
suggerisce all'avvocato il fraseggio per chiedere a Claude di aprire
l'URL pubblico — Claude usa il proprio `WebFetch` tool standard
(user-initiated, non skill-mediated) e il contenuto entra nel
contesto della conversazione. A quel punto le skill processano il
contenuto già letto.

In pratica:

- **Niente configurazione di rete in Claude Desktop.** Funziona
  out-of-box: l'avvocato non deve configurare nessuna egress allowlist.
- **Il gate è la richiesta esplicita dell'avvocato.** Nessuna skill
  apre URL nascostamente. L'avvocato vede sempre cosa sta per essere
  letto e decide se chiedere l'apertura.
- **L'installazione di skill terze passa dall'UI nativa di Claude
  Desktop** (Customize → Plugin → Crea plugin → URL → Add). Lo
  `skill-installer` audita il `SKILL.md` letto in contesto prima
  dell'installazione; non scrive file direttamente.

Razionale completo: test empirico founder 2026-05-19 ha mostrato che
`WebFetch` invocato esplicitamente dall'utente (*"apri https://..."*)
bypassa la sandbox proxy, mentre skill agent autonomous fetch è
soggetto a restrizioni di rete. La doctrine pointer elimina la
dipendenza da configurazione utente — non c'è più una "modalità
live" da attivare né un "fallback bundled". C'è una sola modalità.

Riferimento decisione: MHC-Work `_org/decision_log.md` voce
"Plugin Cowork mhc-l: ratifica strategica riduzione a verifica-fonti
only" (2026-05-18, plugin allora denominato `mhc-l`, rinominato
`iuris-it` 2026-05-19) + raffinamento founder in-session post UX test
("default invariato + bollettino come gateway opt-in") + doctrine
pointer 2026-05-19 (refactor 3.3.0).

## Installazione

Vedi **[DISTRIBUZIONE.md](./DISTRIBUZIONE.md)** per il flusso
passo-passo (5 click in Claude Desktop, no terminale, no account
GitHub).

## Licensing

MIT. Vedi `LICENSE`.

## Contribuire

- **PR su `verifica-fonti`** (pattern di citazione mancanti, registri
  normativi da aggiungere, plausibilità numeriche affinate) → benvenute.
- **PR su documentazione** → benvenute.
- **Issue** per segnalare false positive / false negative del controllo,
  o nuovi pattern di citazione → benvenute.

---

# iuris-it — Italian and EU legal-citation verification

*Claude cowork plugin. One skill: checks that Italian and EU legal
references cited in a text correspond to real documents and are
formally coherent. Reports format anomalies, implausible numbering,
contextual inconsistencies, and possible hallucinated citations,
directing the lawyer to authoritative registers (Normattiva, EUR-Lex,
Italgiure, Corte Costituzionale, Garante Privacy, AGCM, Banca d'Italia,
InfoCuria CGUE).*

The 2.x meta-plugin scope (skill curation, installer, on-the-fly Italian
adaptation) is no longer always-on: it has been refactored as an
**advanced opt-in** mode (2026-05-18). The default user experience is
the single `verifica-fonti` skill — fast, no LLM adapter calls, no
network. Power-users who want a curated Italian legal-tech skills
ecosystem can invoke the bollettino explicitly to open the extended
pipeline (catalogo + skill-installer + adattamento-italiano).

Pointer doctrine (3.3.0, 2026-05-19): advanced-mode skills never
perform autonomous `WebFetch` in the background. When a public URL
(the bollettino, a third-party `SKILL.md`) needs to be read, the skill
points the lawyer to the URL and the exact phrasing to ask Claude to
open it — Claude reads it via its standard user-initiated `WebFetch`
tool, the content enters context, and the skill processes it. No
network configuration required on the lawyer's side. Actual install
of third-party skills happens through Claude Desktop's native plugin
UI; the skill-installer only audits the SKILL.md and logs the result.

License: MIT.
