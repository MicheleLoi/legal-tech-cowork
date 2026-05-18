# MHC-L — verifica fonti normative italiane ed europee

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

## Perché solo una skill (e non un meta-plugin di skill terze)

Versioni precedenti del plugin (2.x) tentavano un meta-pattern: catalogo
+ skill-installer + adattamento italiano al volo per skill US-centric.
Ratifica founder 2026-05-18 (strada B), riassunta:

1. **Install runtime troppo lento** — il flusso di installazione runtime
   delle skill terze, anche con security gate, era pesante per l'avvocato
   non-tech.
2. **Non sono avvocato** — la curation editoriale di skill legali terze
   richiede competenza giuridica che resta esterna al progetto.
3. **Claude parla già italiano nativo** — non serve "tradurre" una skill
   inglese: se l'avvocato scrive italiano a Claude, le skill Anthropic
   ufficiali (in inglese) producono output in italiano direttamente.

Il valore aggiunto residuo è dove Claude non può supplire da solo: la
verifica puntuale di citazioni normative italiane (formato, plausibilità,
registri authoritative). Quello è `verifica-fonti`. Il resto del lavoro
legale generale → marketplace ufficiale Anthropic.

Riferimento decisione: MHC-Work `_org/decision_log.md` voce
"Plugin Cowork mhc-l: ratifica strategica riduzione a verifica-fonti only"
(2026-05-18).

## Cosa NON fa

- **Non cura skill terze.** Niente bollettino, catalogo, skill-installer.
  Le skill legali generali stanno sul marketplace Anthropic.
- **Non traduce skill upstream in italiano.** Non serve: Claude risponde
  italiano nativo se gli parli italiano.
- **Non garantisce la correttezza sostanziale.** Una citazione formalmente
  corretta può comunque essere inapplicabile al caso concreto.
- **Non consulta i registri live.** Il controllo è basato su formato,
  plausibilità numerica, coerenza testuale e knowledge interna — i link
  ai registri authoritative indirizzano l'avvocato alla verifica, non la
  sostituiscono.

## Installazione

Vedi **[DISTRIBUZIONE.md](./DISTRIBUZIONE.md)** per il flusso
passo-passo (5 click in Claude Desktop, no terminale, no account
GitHub). Versione web della guida con screenshot:
[`https://micheleloi.pro/mhc-l/istruzioni/`](https://micheleloi.pro/mhc-l/istruzioni/).

## Licensing

MIT. Vedi `LICENSE`.

## Contribuire

- **PR su `verifica-fonti`** (pattern di citazione mancanti, registri
  normativi da aggiungere, plausibilità numeriche affinate) → benvenute.
- **PR su documentazione** → benvenute.
- **Issue** per segnalare false positive / false negative del controllo,
  o nuovi pattern di citazione → benvenute.

---

# MHC-L — Italian and EU legal-citation verification

*Claude cowork plugin. One skill: checks that Italian and EU legal
references cited in a text correspond to real documents and are
formally coherent. Reports format anomalies, implausible numbering,
contextual inconsistencies, and possible hallucinated citations,
directing the lawyer to authoritative registers (Normattiva, EUR-Lex,
Italgiure, Corte Costituzionale, Garante Privacy, AGCM, Banca d'Italia,
InfoCuria CGUE).*

The 2.x meta-plugin scope (skill curation, installer, on-the-fly Italian
adaptation) has been retired (2026-05-18): general legal skills are on
Anthropic's official marketplace, Claude already replies in Italian
natively when addressed in Italian. The residual value — Italian/EU
citation verification — is what remains here.

License: MIT.
