# MHC-L — meta-plugin di adattamento italiano per skill legal-tech AI open source

[![License: MIT (parti originali)](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![License: Apache 2.0 (parti forkate)](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

*Plugin per Claude cowork. Monitora automaticamente l'ecosistema
legal-tech AI open source, propone adattamenti italiani sotto conferma
dell'avvocato, verifica le citazioni normative italiane ed europee
contro registri authoritative.*

---

## Cosa è

MHC-L è un **fork-and-extend** di `legal-builder-hub`
([anthropics/claude-for-legal](https://github.com/anthropics/claude-for-legal),
Apache-2.0) con tre layer aggiunti originali nostri (MIT):

1. **Layer italiano (`adattamento-italiano`)** — skill invocabile
   esplicitamente dall'avvocato per adattare una skill già installata
   al diritto italiano. Genera una proposta di traduzione + mappatura
   dei riferimenti normativi all'equivalente italiano/europeo (usando
   `adaptation_prompt.md`), pre-flighta le citazioni con
   `verifica-fonti`, e mostra il diff per Approva / Modifica /
   Annulla. Si attiva (a) su richiesta diretta dell'avvocato
   ("adatta `[X]` in italiano"), oppure (b) come risposta "sì" al
   prompt che lo `skill-installer` mostra dopo aver installato una
   skill non classificata come italiana/europea (REV2.1, 2026-05-18).
   Mai automaticamente, mai come hook silenzioso.

2. **Verifica fonti italiane ed europee (`verifica-fonti`)** — controllo
   di coerenza, plausibilità e formato delle citazioni normative IT/EU.
   Cita registri authoritative (Normattiva, Gazzetta Ufficiale,
   Cassazione CED, Corte Cost., Cons. Stato, Garante Privacy, AGCM,
   CONSOB, Banca d'Italia, EUR-Lex, InfoCuria CGUE, IATE) con URL
   canonici. Pre-flight in fase di adattamento, auto-suggerita
   sull'output runtime di skill IT/EU.

3. **Bollettino ecosystem-monitored** — catalogo curato in modo
   automatico da una routine (`bollettino-research`, in MHC-Work) che
   monitora mensilmente GitHub broadly nell'ecosistema legal-tech AI
   open source (post-Mike Bommarito e oltre), applica una **threshold
   policy esplicita** (license OSS, reputation minima, IT-relevance
   heuristic) e auto-committa + auto-pusha le voci che passano.
   Founder fuori dal runtime loop — la threshold policy è la
   cristallizzazione del giudizio editoriale.

L'avvocato installa **un solo plugin** (`mhc-l`) che contiene tutto.

## Cosa NON è

- **NON** è un'italianizzazione del plugin `legal` marketplace Anthropic
  (quello consolidato a 9 comandi US-centric). MHC-L lavora a un livello
  diverso: meta-plugin che gestisce skill di terze parti, non un singolo
  plugin di skill predefinite.
- **NON** è un servizio di consulenza legale. Le skill propongono
  strumenti; l'avvocato porta il giudizio e mantiene la responsabilità
  professionale di tutto ciò che esce in output.
- **NON** sostituisce la verifica giurisprudenziale autorevole
  (Italgiure, De Jure, EUR-Lex live). `verifica-fonti` controlla formato
  e plausibilità contro knowledge interna, indirizza ai registri
  authoritative; non sostituisce la consultazione live di chi è abilitato.
- **NON** automatizza decisioni giuridiche. Tutto passa da conferme
  esplicite dell'avvocato — il modello è dual control, non automazione
  cieca.

## Come funziona (modello operativo)

1. **Bollettino bundled all'install, opzione live opzionale.** Il
   file `bollettino.json` in questo repo è il catalogo, aggiornato
   mensilmente dalla routine `bollettino-research` in MHC-Work. La
   modalità di default consegna all'avvocato la copia bundled con il
   plugin (sempre disponibile, no setup tecnico). La modalità live
   (fetch fresh ad ogni apertura) richiede un'azione facoltativa di
   configurazione dell'utente — istruzioni:
   [`https://micheleloi.pro/mhc-l/istruzioni/`](https://micheleloi.pro/mhc-l/istruzioni/).
2. **Skill discovery automatica.** La routine cerca su GitHub broadly
   skill legal-tech AI open source, applica la threshold policy
   esplicita (vedi `BOLLETTINO_FORMAT.md` § "Threshold policy") e
   pubblica le voci qualificate.
3. **Installazione con security gate.** Quando l'avvocato chiede di
   installare, il pipeline `skill-installer` forkato (Apache-2.0
   Anthropic) gestisce allowlist, fetch in subagent read-only,
   raw-source display, structural trust check, license verification
   pre+post fetch, freshness gate. Layer di sicurezza industrial-grade.
4. **Adattamento italiano come secondo passo cosciente
   dell'avvocato.** Dopo che lo `skill-installer` ha completato
   l'install, l'avvocato può chiedere esplicitamente *"adatta `[X]`
   in italiano"* (o usare `/adattamento-italiano [X]`). Per le skill
   non classificate come italiane/europee, lo `skill-installer`
   propone direttamente la scelta con un prompt sì/no (REV2.1,
   2026-05-18). In entrambi i casi `adattamento-italiano` genera la
   proposta, pre-flighta le citazioni normative con `verifica-fonti`,
   e mostra il diff all'avvocato per Approva / Modifica / Annulla.
   Hook automatici silenziosi sono stati rimossi per design.
5. **Verifica fonti post-uso.** Dopo l'output di skill marcate IT o EU,
   MHC-L suggerisce di passare il testo a `verifica-fonti` per un
   controllo di coerenza delle citazioni runtime.

## Installazione (5 click)

Vedi **[DISTRIBUZIONE.md](./DISTRIBUZIONE.md)** per la guida + link al
video walkthrough. In sintesi:

1. Claude Desktop → tab **Cowork**
2. Sidebar **Customize → Plugin → "+"**
3. **"Crea plugin"** (sì, "Crea plugin" — è il path UX
   counter-intuitivo per "aggiungi marketplace esistente")
4. Incolla URL: `https://github.com/MicheleLoi/legal-tech-cowork` → Add
5. Accanto a "mhc-l", **"Add plugin"**

## Stato del catalogo

Il catalogo cresce per accumulo automatico. La routine
`bollettino-research` in MHC-Work monitora mensilmente l'ecosistema
legal-tech open source e pubblica le voci che superano la threshold
policy. Vedi `BOLLETTINO_FORMAT.md` per lo schema, la threshold
policy e il processo di ingresso.

## Per chi è

Avvocato italiano che usa Claude per il lavoro professionale e vuole:

- **Risparmiare tempo** sull'esplorazione manuale di skill legal-tech
  esistenti senza dover monitorare lui stesso GitHub e l'ecosistema;
- **Non rinunciare al diritto italiano** quando le skill sono nate in
  contesto US-centric (la maggior parte sull'ecosistema);
- **Mantenere il controllo finale** su ogni passaggio (niente
  installazioni silenti, niente citazioni non verificabili, raw SKILL.md
  sempre mostrato prima dell'install);
- **Beneficiare del security gate Anthropic** (forkato Apache-2.0) senza
  dover reinstallare separatamente `claude-for-legal` (che non è
  marketplace-installable su Pro standard).

## Licensing — dual

Questo plugin è un **fork-and-extend** di codice Anthropic open source.
Le parti del codice sono coperte da due licenze distinte:

- **Apache-2.0** (parti forkate da `anthropics/claude-for-legal`
  legal-builder-hub): `skills/skill-installer/SKILL.md`,
  `skills/skill-installer/references/allowlist.md`,
  `skills/skill-installer/references/freshness.md`. Questi file
  conservano in cima un commento Apache 2.0 + annotazione di
  provenance ("Forked from anthropics/claude-for-legal..."). Vedi
  `LICENSE-ANTHROPIC` per il testo integrale Apache 2.0.

- **MIT** (parti originali nostre): `skills/adattamento-italiano/`,
  `skills/verifica-fonti/`, `skills/catalogo/adaptation_prompt.md`,
  configurazione bollettino (`bollettino.json` + threshold policy in
  `BOLLETTINO_FORMAT.md`), documentazione (`README.md`,
  `DISTRIBUZIONE.md`, `BUILD_NOTES.md`). Vedi `LICENSE` per il testo
  MIT e l'elenco esplicito dei file.

Vedi `NOTICE` per l'attribution Anthropic come richiesto da
Apache 2.0 §4(d).

## Contribuire

- **Issue per suggerire una skill** → apri issue con link al repo; la
  routine `bollettino-research` la valuta in modalità ad-hoc applicando
  la stessa threshold policy.
- **PR su `verifica-fonti`** (pattern di citazioni, registri normativi
  mancanti, plausibilità numeriche) → benvenute.
- **PR su `adaptation_prompt.md`** (mappature normative IT/EU mancanti
  o da correggere) → benvenute.
- **PR su documentazione** → benvenute.
- **PR che aggiungono voci a `bollettino.json` a mano** → NON accettate.
  Il bollettino passa dalla routine automatica. Per casi mirati, apri
  issue e la routine la elabora in modalità ad-hoc.

---

# MHC-L — meta-plugin for Italian adaptation of open-source legal-tech AI skills

*Plugin for Claude cowork. Automatically monitors the open-source
legal-tech AI ecosystem, proposes Italian adaptations under lawyer
confirmation, verifies Italian and EU legal citations against
authoritative registers.*

## What it is

MHC-L is a **fork-and-extend** of `legal-builder-hub`
([anthropics/claude-for-legal](https://github.com/anthropics/claude-for-legal),
Apache-2.0) with three originally-authored MIT layers added on top:
Italian adaptation hook, IT/EU legal-citation verification, and an
ecosystem-monitored bollettino populated by an automatic routine. One
plugin for the lawyer, three concerns covered.

## What it is NOT

- Not an Italianization of the consolidated `legal` marketplace plugin
  by Anthropic.
- Not a legal advice service.
- Not a substitute for authoritative case-law / statute consultation.
- Not automation of legal judgment — every install passes through
  explicit lawyer confirmation.

## Install (5 clicks)

See **[DISTRIBUZIONE.md](./DISTRIBUZIONE.md)** (Italian — that's the
target audience). The URL to paste is
`https://github.com/MicheleLoi/legal-tech-cowork`. Path:
`Customize → Plugin → "+" → Crea plugin → marketplace URL`. Yes,
"Crea plugin" is the counter-intuitive UX path for "add existing
marketplace" — known Claude Desktop Pro quirk as of May 2026.

## Licensing

Dual: Apache-2.0 for forked parts (legal-builder-hub from Anthropic),
MIT for originally-authored parts. See `LICENSE`, `LICENSE-ANTHROPIC`,
and `NOTICE`.
