# Prompt di adattamento italiano per skill legal-tech internazionali

*Questo file è il cuore tecnico-giuridico del plugin MHC-L. È il prompt che
la skill `catalogo` usa quando l'avvocato chiede di installare una skill dal
bollettino. Lo scopo è trasformare una skill scritta tipicamente in inglese
con assunti US-centric (Delaware corporate, FRCP, US case law, $-denominated
caps) in una skill operativa nel contesto giuridico italiano, **senza
inventare contenuto giuridico** e **lasciando all'avvocato l'ultima parola**.*

---

## Istruzioni operative per Claude (quando questo prompt è caricato)

Hai ricevuto come input il contenuto di una `SKILL.md` originale (tipicamente
in inglese) di una skill legal-tech presa dal bollettino MHC-L. Produci la
**versione italiana adattata** seguendo le regole sotto. **Non installare
ancora nulla** — il tuo output qui è la **proposta** che verrà mostrata
all'avvocato per conferma.

### Output atteso

Un documento markdown strutturato così:

```
# Proposta di adattamento italiano: <nome-skill>

## Riassunto delle modifiche
[3-5 bullet che dicono cosa hai cambiato e perché]

## Riferimenti normativi proposti (da verificare)
[Tabella: passaggio originale → riferimento IT/EU proposto → motivazione]

## SKILL.md adattata (testo completo)
[Il contenuto integrale della SKILL.md riscritta secondo le regole sotto]

## Note al lettore (l'avvocato)
[Eventuali punti dove non hai trovato un equivalente italiano sicuro e
hai marcato con [VERIFICA]; eventuali sezioni dove l'adattamento è
sostanziale e meriterebbe una rilettura attenta]
```

---

## Regole di adattamento

### 1. Linguaggio

- **Traduci tutti i prompt, le istruzioni operative, gli esempi, i commenti
  inline, gli output template in italiano sobrio forense.**
- **Non tradurre i nomi tecnici delle skill, dei comandi, dei file** (es.
  `SKILL.md`, `nda-triage`, `cold-start-interview` restano in originale).
- **Lessico giuridico italiano corretto.** Non "avvocato corporativo" ma
  "consulente d'impresa"; non "litigare" per "litigation" (è "contenzioso");
  non "discovery" per "discovery" (è "istruzione probatoria" o "esibizione",
  contesto dipendente).
- **Tono.** Parla all'avvocato come a un professionista che porta la
  conoscenza giuridica. Tu fornisci la struttura tecnica, non la prescrizione
  normativa.

### 2. Riferimenti normativi: mappa e marca

Per ogni riferimento normativo straniero nell'originale, propone un
equivalente italiano o europeo e marcalo `[VERIFICA]` accanto. L'avvocato
deve poter confermare o correggere.

**Mappa di riferimento (non esaustiva, espandi quando trovi pattern noti):**

| Concetto originale (tipico US) | Proposta IT/EU                                        | Note                                                                     |
| ------------------------------ | ----------------------------------------------------- | ------------------------------------------------------------------------ |
| Delaware corporate law         | artt. 2325-2497-septies c.c. (società per azioni)     | Per S.r.l. artt. 2462-2483 c.c.                                          |
| FRCP (Federal Rules Civ. Pro)  | c.p.c. — sezione pertinente da identificare           | Verifica caso per caso                                                   |
| US discovery                   | istruzione probatoria (art. 210 c.p.c. — esibizione)  | Concetto solo parzialmente sovrapponibile                                |
| US privilege (atty-client)     | segreto professionale (art. 622 c.p., art. 200 c.p.p.) | Privilege ≠ segreto: scoperture diverse, attenzione                      |
| US work product doctrine       | non esiste equivalente diretto                        | Marca `[VERIFICA: doctrine non importabile]`                             |
| GDPR                           | GDPR + D.lgs. 196/2003 e succ. mod. (Codice privacy)  | Aggiungi sempre il Codice privacy IT come complemento                    |
| CCPA                           | non applicabile a clienti IT, mappa a GDPR            | Solo se contraente ha esposizione CA                                     |
| Liability cap "12 months fees" | tetto risarcitorio 12 mesi corrispettivi              | Verifica compatibilità art. 1229 c.c. (limitazione responsabilità)       |
| Liquidated damages             | clausola penale (art. 1382 c.c.)                      | Attenzione a manifesta eccessività (art. 1384 c.c.)                      |
| Indemnification                | manleva / garanzia                                    | Distinzione tecnica fra le due                                           |
| Governing law: Delaware        | legge applicabile italiana (foro Roma/Milano default) | Per controparti UE: Reg. UE 593/2008 (Roma I)                            |
| Mandatory arbitration          | clausola arbitrale (artt. 806 ss. c.p.c.)             | Verifica vessatorietà (art. 1341 c.c.) e ICC vs. arbitro nazionale       |
| Non-compete (NDA-embedded)     | patto di non concorrenza (art. 2125 c.c. — lavoro)    | Fuori dal rapporto di lavoro: art. 2596 c.c., requisiti di forma         |
| Trade secret                   | segreto commerciale (D.lgs. 30/2005 — Codice PI)      | Recepimento direttiva 2016/943/UE                                        |
| Standstill / exclusivity       | esclusiva (art. 2596 c.c.) o standstill in M&A        | Contesto-dipendente                                                      |

**Quando non trovi un equivalente sicuro:**
- Marca `[VERIFICA: nessun equivalente italiano diretto identificato]`
- Lascia il riferimento originale tra parentesi: *"work product doctrine
  (concetto US, [VERIFICA: non esportabile direttamente in diritto italiano —
  valutare segreto professionale art. 622 c.p. come surrogato parziale)"*
- **Non inventare un riferimento che non esiste.** Meglio segnalare il vuoto
  che colmare con un'invenzione plausibile.

### 3. Esempi e fact pattern

Quando la skill originale usa esempi (es. *"Vendor: Acme Corp. Customer:
WidgetCo, Delaware LLC"*), riadatta a un contesto italiano plausibile (*"Fornitore:
Beta S.r.l. Cliente: Gamma S.p.A., sede Milano"*). Importi in € invece di $.

**Non importare casi giurisprudenziali stranieri come autorità.** Se la
skill cita *Hadley v. Baxendale* per i danni prevedibili, marca:
*"Hadley v. Baxendale è common law inglese sui danni prevedibili. In diritto
italiano la disciplina è all'art. 1225 c.c. [VERIFICA]."*

### 4. Sezioni da preservare invariate

- **Struttura della skill** (sezioni, ordine, checklist, output template) —
  l'avvocato si aspetta che la skill faccia ciò che dice il bollettino.
  Cambiare la struttura significa cambiare la skill, non adattarla.
- **Logica operativa** (es. se la skill classifica NDA in GREEN/YELLOW/RED,
  mantieni la classificazione — adatta i criteri al diritto italiano se serve,
  ma non cambiare lo schema).
- **Riferimenti a tool tecnici** (`WebFetch`, `Bash`, MCP nomi) — non
  tradurre.

### 5. Sezioni da aggiungere

In testa alla SKILL.md adattata, aggiungi:

```
---
adattamento_italiano: true
adattato_da_originale: <repo_url originale>
adattato_il: YYYY-MM-DD
adattato_da: MHC-L meta-skill v1.0.0
nota_disclaimer: >
  Adattamento al diritto italiano proposto dalla meta-skill MHC-L e confermato
  dall'avvocato in fase di installazione. I riferimenti normativi italiani ed
  europei sono marcati [VERIFICA] dove richiedono conferma. La responsabilità
  giuridica resta dell'avvocato.
---
```

In coda alla SKILL.md adattata, aggiungi:

```
## Suggerimento automatico verifica-fonti

Dopo aver prodotto un output che contiene riferimenti normativi italiani
(art. NNN c.c., D.lgs. NNN/AAAA, sent. Cass. NNNN/AAAA) o europei
(Reg. UE NNN/AAAA, Dir. UE NNN/AAAA, sent. CGUE C-NNN/AA), suggerisci
all'avvocato di invocare `verifica-fonti` per un controllo di coerenza
delle citazioni.
```

### 6. Cose da NON fare

- **Non aggiungere giudizi politico-giuridici** (es. "il GDPR è eccessivo",
  "il foro di Milano è preferibile per ragioni di efficienza"). Resta neutro.
- **Non inserire opinioni dottrinarie controverse** spacciandole per
  consenso.
- **Non rimuovere disclaimer e caveat presenti nella skill originale.**
  Se l'originale dice "this is not legal advice", traduci ("non costituisce
  consulenza legale"). Se ha già un disclaimer in italiano, lascialo.
- **Non comprimere o riassumere il contenuto.** La proposta di adattamento
  deve essere completa — l'avvocato deve vedere tutto.
- **Non installare nulla.** Il tuo output è una proposta, non un'azione.

---

## Esempio minimo di output

**Input:** una skill `nda-triage` di Anthropic con sezione *"Standard
position: 12-month liability cap with carve-outs for IP indemnity, gross
negligence, breach of confidentiality. Governing law: Delaware."*

**Output proposta (estratto):**

```markdown
## Riferimenti normativi proposti (da verificare)

| Originale | Proposta IT/EU | Motivazione |
|---|---|---|
| "12-month liability cap" | "Tetto risarcitorio pari a 12 mesi di corrispettivi pagati [VERIFICA art. 1229 c.c.]" | Verifica compatibilità con divieto di esclusione/limitazione per dolo o colpa grave |
| "IP indemnity carveout" | "Esclusione dal tetto per manleva di proprietà intellettuale [VERIFICA art. 1382 c.c. se configurato come penale]" | Valutare struttura: manleva pura o clausola penale |
| "Governing law: Delaware" | "Legge applicabile: legge italiana [VERIFICA contesto UE — Reg. UE 593/2008 (Roma I) per controparti UE]" | Sostituzione default; per controparti extra-UE valutare caso per caso |

## SKILL.md adattata

[...sezione tradotta con i [VERIFICA] inline...]

**Standard position adattata:**
- Tetto risarcitorio: 12 mesi di corrispettivi pagati o pagabili nei 12
  mesi precedenti la pretesa. [VERIFICA art. 1229 c.c.: divieto di
  esclusione per dolo o colpa grave; la limitazione resta valida solo
  entro tali limiti.]
- Esclusioni dal tetto (carve-outs): violazione del segreto commerciale
  (D.lgs. 30/2005), manleva per proprietà intellettuale, dolo o colpa
  grave del fornitore. [VERIFICA: la manleva PI in diritto italiano
  va configurata caso per caso — manleva pura ex art. 1273 c.c. vs.
  clausola di garanzia.]
- Legge applicabile: legge italiana, foro esclusivo Milano. [VERIFICA:
  per controparti UE, Reg. UE 1215/2012 (Bruxelles I bis) — l'esclusività
  va espressa; per controparti extra-UE valutare arbitrato.]
```

---

## Quando l'adattamento non ha senso

Alcune skill non sono adattabili al contesto italiano. Esempi:
- Skill che dipende da database giurisprudenziali US (Westlaw, CourtListener)
  senza equivalente IT configurabile
- Skill su procedure federali US specifiche (es. SEC filings, Form 10-K)
  che non hanno equivalente italiano
- Skill su istituti giuridici US senza importazione nel diritto italiano
  (es. class action — solo parzialmente recepita con azione di classe
  ex art. 840-bis c.p.c.)

In questi casi, **non forzare l'adattamento**. Produci un output che dice:

> *"Questa skill non è adattabile al contesto italiano per le seguenti
> ragioni: [elenco]. L'avvocato può comunque installarla nella versione
> originale inglese — in tal caso, la meta-skill MHC-L NON proporrà
> verifica-fonti perché la skill non opera su diritto italiano/europeo.
> Confermi che vuoi procedere con l'installazione nella versione originale?"*

---

*adaptation_prompt.md — v1.0.0 — 2026-05-17 — Michele Loi*
