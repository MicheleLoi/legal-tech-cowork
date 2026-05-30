---
name: prova-schema
description: >
  Raccoglie l'esito onesto quando l'avvocato ha messo alla prova uno
  schema del bollettino di BeccarIA su un fascicolo reale. Tre stati di
  esito ("mi è servito" / "non mi è servito" / "mi avrebbe portato fuori
  strada"), una nota di contesto, un consenso leggero, e un'email
  pre-compilata da inviare con un click (mailto a mhcl@micheleloi.pro).
  Nessun dato esce dal computer senza il click esplicito dell'avvocato.
  Si attiva su trigger naturali come "ho provato uno schema", "voglio
  dire com'è andato uno schema", "metti alla prova", "/prova-schema".
  Opt-in: non si attiva da sola.
---

<!-- SPDX-License-Identifier: AGPL-3.0-only (skill code) -->
<!-- Bulletin content licensed proprietary © MicheleLoi 2026; see
     https://github.com/MicheleLoi/regia-bollettino-updater/blob/main/NOTICE.md -->

# prova-schema — registra com'è andata una prova di uno schema

## A cosa serve

Quando l'avvocato ha **messo alla prova** uno schema del bollettino
(un'impalcatura di lavoro applicata via `schemi-di-ragionamento`) su un
fascicolo reale, questa skill raccoglie l'esito onesto della prova e lo
trasmette a chi cura il bollettino, tramite un'email pre-compilata che
l'avvocato invia con un click.

Il patto è semplice e va detto chiaro fin dall'inizio:

> **Comunque sia andata, questo schema resta tuo.** Il perk si guadagna
> mettendo alla prova lo schema sul tuo lavoro e raccontando onestamente
> com'è andata — non dipende dall'esito. Anche un "mi avrebbe portato
> fuori strada" è un contributo prezioso: aiuta a migliorare lo schema
> per tutti.

Non dire mai "se ti è servito, tienilo". Il perk **non** dipende
dall'esito positivo: scatta sull'atto della prova onesta.

## Quando attivarsi

**Opt-in. Non auto-attivarti.** Questa skill parte solo quando
l'avvocato lo chiede esplicitamente, tipicamente *dopo* aver usato uno
schema del bollettino (via `schemi-di-ragionamento`) sul proprio lavoro.

**Trigger naturali:**

- *"Ho provato uno schema."*
- *"Voglio dire com'è andato uno schema."*
- *"Quello schema che ho usato — ti racconto com'è andata."*
- *"Metti alla prova"* / *"ho messo alla prova lo schema X."*
- `/prova-schema`

**Trigger via concatenazione:** se in un turno precedente
`schemi-di-ragionamento` ha applicato uno schema e l'avvocato, più avanti,
dice qualcosa come "alla fine mi è servito" / "non mi ha aiutato" /
"mi avrebbe sviato", offri di registrare l'esito con questa skill —
ma chiedi conferma, non procedere d'ufficio.

## Quando NON attivarti

- L'avvocato vuole **applicare** uno schema nuovo al suo caso → quello è
  `schemi-di-ragionamento`.
- L'avvocato chiede *quali* schemi esistono → quello è `ecosystem-scout`.
- L'avvocato chiede di verificare citazioni normative → quello è
  `verifica-fonti`.
- L'avvocato non ha ancora messo alla prova nessuno schema → non c'è
  niente da registrare; non sollecitare.

## Cosa fai, passo per passo

Quattro step in conversazione, poi l'assemblaggio dell'email. Tieni il
tono forense sobrio: stai raccogliendo l'esito di una prova professionale,
non un sondaggio di gradimento.

### Step 1 — Quale schema è stato messo alla prova

Identifica lo schema. Se l'avvocato lo nomina, usa quel nome. Altrimenti,
deducilo dal contesto (se `schemi-di-ragionamento` ha applicato uno schema in
questa stessa conversazione, è quasi certamente quello) e **conferma**:

> *"Stai mettendo alla prova lo schema `<pattern_id>` — confermi? Se è un
> altro, dimmi quale."*

Il riferimento utile è il `pattern_id` (o, in mancanza, il `task_name`)
dello schema così come compare nel bollettino. Se l'avvocato non lo
ricorda, va bene una descrizione in parole sue: registreremo quella.

### Step 2 — Com'è andata (tre stati esatti)

Chiedi l'esito della prova. **Tre stati, e solo questi tre**, con esattamente
queste etichette:

1. **"mi è servito"** — lo schema ha aiutato il lavoro.
2. **"non mi è servito"** — lo schema non ha aggiunto valore al caso.
3. **"mi avrebbe portato fuori strada"** — seguire lo schema avrebbe
   condotto a un approccio sbagliato per questo fascicolo.

Presentali come scelta chiara:

> *"Com'è andata la prova? Scegli l'esito più onesto:*
> *(a) mi è servito*
> *(b) non mi è servito*
> *(c) mi avrebbe portato fuori strada"*

Gli ultimi due esiti ("non mi è servito", "mi avrebbe portato fuori
strada") sono una **segnalazione**: un contributo che aiuta a migliorare
lo schema. Non chiamarli "scarto" o "schema scartato" — non lo sono.
L'avvocato che segnala un limite sta facendo un favore alla community.

Ribadisci, se serve, che l'esito non cambia il perk:

> *"Qualunque sia l'esito, hai già messo alla prova lo schema: il perk è
> tuo."*

### Step 3 — Nota di contesto (richiesta)

Chiedi una nota di contesto. **Non è opzionale**: è la parte che rende la
segnalazione utile.

> *"Raccontami in breve: come l'hai usato e perché questo esito? (massimo
> ~800 caratteri) Questa nota è la parte che serve di più — è ciò che
> permette di capire dove lo schema funziona e dove va migliorato."*

Spiega all'avvocato il perché: la nota di contesto è ciò che chi cura il
bollettino legge per decidere come affinare lo schema. Un esito senza
contesto è quasi muto; un esito con una buona nota è un miglioramento
concreto.

Se l'avvocato consegna una nota molto più lunga di ~800 caratteri,
proponi gentilmente di sintetizzarla insieme (l'email pre-compilata
resta leggibile), senza perdere il punto.

### Step 4 — Consenso leggero per la citazione

Cattura ora il segnale di consenso (servirà a una fase successiva, in cui
gli schemi che hanno retto verranno marcati "collaudato"):

> *"Se in futuro questo schema diventerà 'collaudato' — cioè avrà retto
> a più prove — posso citare il tuo nome come uno di chi l'ha messo alla
> prova? (sì / no)"*

**Default prudente:** se l'avvocato non risponde, o risponde in modo
ambiguo, registra `no`. Il consenso alla citazione si dà attivamente; in
dubbio, non si cita.

## Assemblaggio dell'email (markdown con frontmatter)

Componi un messaggio in markdown e **mostralo verbatim all'avvocato per
revisione prima di qualsiasi invio**. Struttura:

```markdown
---
beccaria_prova_version: 1
pattern_id: <pattern_id o task_name dello schema messo alla prova>
esito: <mi è servito | non mi è servito | mi avrebbe portato fuori strada>
submitted_date: <YYYY-MM-DD di oggi>
consenso_nome: <sì | no>
---

## Nota di contesto

<la nota di contesto dello Step 3, verbatim>
```

Note di compilazione:

- `esito` riporta **esattamente** una delle tre etichette canoniche dello
  Step 2 — niente sinonimi, niente "validato/non validato".
- `submitted_date` è la data odierna in formato `YYYY-MM-DD`.
- `consenso_nome` è `sì` o `no` (default `no` per lo Step 4).
- Il body contiene la nota di contesto verbatim. Se l'avvocato ha
  fornito anche il proprio nome per la citazione (Step 4 = sì), puoi
  aggiungerlo in coda al body come riga `Citabile come: <nome>` — solo
  se `consenso_nome` è `sì`.

## Link mailto pre-compilato

Costruisci un link `mailto` con subject e body già pronti:

```
mailto:mhcl@micheleloi.pro?subject=[BeccarIA prova] <pattern_id> — <esito>&body=<markdown url-encoded>
```

- `<pattern_id>` e `<esito>` nel subject in chiaro (l'esito è una delle
  tre etichette canoniche).
- `<markdown url-encoded>` è l'intero blocco markdown sopra, codificato
  secondo le regole di URL-encoding (spazi, accapo, accenti, `#`, `—`,
  ecc. tutti codificati correttamente — RFC 3986).

Mostra all'avvocato:

1. Il **markdown verbatim** (ultima possibilità di revisione e correzione).
2. Il **link mailto cliccabile** come URL completo.
3. I disclaimer qui sotto.

## Privacy e UX (vincoli rigidi)

Mostra all'avvocato, accanto al link:

> *Quando clicchi il link, si apre il tuo client email con il messaggio
> già scritto. Premi Invia per condividere l'esito della prova. Se
> preferisci non condividere, semplicemente non cliccare: **niente esce
> dal tuo computer senza il tuo click esplicito.***

E l'avviso sui dati dei clienti:

> *Nota: non includere nella nota dati di clienti reali — nomi, partite
> IVA, riferimenti di fascicolo, informazioni riservate. Descrivi il tipo
> di caso e perché lo schema ha (o non ha) funzionato, non il caso
> specifico.*

**Zero storage — vincolo architetturale assoluto:**

- Questa skill **non** salva nulla: nessun file locale, nessun file di
  stato, nessuna scrittura su filesystem.
- Questa skill **non** chiama alcun backend HTTP, non fa `WebFetch`, non
  invia dati a nessun server.
- L'esito della prova esiste **solo** nella conversazione corrente e
  sparisce quando la conversazione finisce. L'unico modo in cui lascia
  il computer è il click manuale dell'avvocato sul link `mailto`, che
  apre il *suo* client email locale.
- Nessuna automazione nascosta. Nessun invio in background. Il gate è,
  sempre e solo, il click dell'avvocato.

## Chiusura

Dopo aver mostrato il link, chiudi ricordando il patto con il lessico
giusto:

> *Grazie per aver messo alla prova lo schema. **Comunque sia andata,
> questo schema resta tuo.** Se hai messo alla prova altri schemi, possiamo
> registrarli uno alla volta — dimmi pure.*

Non condizionare il perk all'esito. Non dire "se ti è servito tienilo".
Il perk è verdict-neutral: si guadagna con la prova onesta, qualunque ne
sia stato l'esito.

## Lessico (vincolante)

Due famiglie di parole, da non confondere mai:

- **L'atto dell'avvocato** = "mettere alla prova" / "provare" uno schema.
  Mai "validare" lo schema per indicare l'atto.
- **Lo stato dello schema** = "in collaudo" (ha avuto poche prove) /
  "collaudato" (ha retto a più prove). Mai "validato" per lo stato.
- **Gli esiti** sono tre, e solo tre: "mi è servito" / "non mi è servito"
  / "mi avrebbe portato fuori strada".
- Gli ultimi due esiti sono una **segnalazione**, mai uno "scarto".
- **Il perk** è verdict-neutral: "comunque sia andata, è tuo per sempre".

## Tono

Forense sobrio. Stai raccogliendo l'esito di una prova professionale, non
gestendo un sondaggio di soddisfazione. Niente entusiasmo da prodotto,
niente "grazie per il tuo prezioso feedback!". L'avvocato che mette alla
prova uno schema e ne segnala i limiti merita la stessa serietà con cui
lavora.

---

*BeccarIA — prova-schema — AGPL-3.0-only — zero storage, mailto-only —
gemella di `schemi-di-ragionamento` (stesso meccanismo mailto, contenuto
diverso: registra la prova di uno schema esistente, non ne propone uno
nuovo) — il perk è verdict-neutral*
