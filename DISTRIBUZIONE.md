# Come installare MHC-L

*Per l'avvocato. Nessun comando terminale, nessun account GitHub, nessuna
conoscenza tecnica richiesta.*

---

## Cosa installi

Un plugin per Claude cowork. **Default**: una skill, `verifica-fonti`,
che controlla che le citazioni normative e giurisprudenziali italiane
ed europee in un testo siano formalmente corrette, plausibili e
coerenti con il contesto. Fonti coperte: Cassazione, Corte
Costituzionale, Consiglio di Stato, TAR, Normattiva, EUR-Lex, Garante
Privacy, AGCM, ANAC, Banca d'Italia.

**Modalità avanzata opt-in.** Il plugin include anche un ecosistema
opzionale di skill terze italiano-curate (bollettino + catalogo +
installer + adattamento), che resta inerte finché non lo attivi
esplicitamente. Vedi la sezione **Modalità avanzata** in fondo a
questo documento. Se non ti serve, ignorala: il default copre l'80%
dei casi.

Versione web della guida con screenshot:
[`https://micheleloi.pro/mhc-l/istruzioni/`](https://micheleloi.pro/mhc-l/istruzioni/).

---

## Versione testuale (5 click)

### Prerequisiti

- Claude Desktop installato (scaricabile da
  [claude.ai/download](https://claude.ai/download))
- Piano Claude Pro o superiore (il piano gratuito non include plugin
  di terze parti su cowork)

### I 5 click

1. **Apri Claude Desktop**, vai sulla tab **Cowork** nella sidebar
   sinistra.
2. Sempre nella sidebar di Cowork, clicca **Customize → Plugin → "+"**
   in alto a destra.
3. Nel menu che si apre clicca **"Crea plugin"** — sì, "Crea plugin",
   anche se non stai creando nulla: è il path UX counter-intuitivo per
   "aggiungere marketplace esistente". (Noto gotcha dell'interfaccia
   Claude Desktop Pro standard a maggio 2026.)
4. Nel dialog **"Aggiungi marketplace"** incolla:

   ```
   https://github.com/MicheleLoi/legal-tech-cowork
   ```

   Premi **"Add"**.
5. Nella lista che compare, accanto a **"mhc-l"**, clicca
   **"Add plugin"**.

Fatto. Vedrai un messaggio di conferma. Il plugin è attivo.

### Installazione da percorso locale (alternativa)

Se hai una copia del repo già scaricata sul tuo computer (es. via
git clone, o cartella sincronizzata via SwitchDrive / Dropbox /
OneDrive), puoi installare il plugin puntando al percorso locale invece
che all'URL GitHub. Al passo 4 sopra, incolla il percorso assoluto
della cartella `legal-tech-cowork` (es. `/Users/<nome>/legal-tech-cowork`
o `C:\Users\<nome>\legal-tech-cowork`) anziché l'URL. Il resto del
flusso è identico.

### Primo utilizzo

In una conversazione Cowork, dopo che Claude ha prodotto una risposta
con citazioni normative o giurisprudenziali, scrivi:

> *"Controlla le citazioni di questa risposta."*

oppure

> *"Passa l'output a verifica-fonti."*

Riceverai un rapporto strutturato che elenca ogni citazione trovata,
segnala formato anomalo, numerazione implausibile, possibili
incoerenze contestuali o invenzioni, e indirizza ai registri
authoritative (Normattiva, EUR-Lex, Italgiure, ecc.) per la verifica
sostanziale.

---

## Se qualcosa non funziona

### Non vedo "Crea plugin"

Questa opzione richiede piano Claude Pro o superiore. Verifica nelle
impostazioni del tuo account.

### Il plugin non risponde quando chiedo di verificare le fonti

Vai su **Customize → Plugin** e verifica che accanto a MHC-L ci sia
il tag "Active". Se attivo ma non risponde, prova a essere esplicito
nella richiesta: *"usa la skill verifica-fonti su questo testo"*. Se
persiste, apri una issue su
`https://github.com/MicheleLoi/legal-tech-cowork/issues`.

---

## Privacy

Il plugin opera **dentro la sandbox Claude cowork**, isolata per
conversazione. Le cartelle che colleghi a una conversazione sono
visibili al plugin; nient'altro del tuo computer lo è.

Il plugin **non invia dati** a server di terze parti oltre al normale
traffico con Anthropic (le tue conversazioni cowork passano da lì
come sempre). Nessuna telemetria, nessun tracking, nessun analytics,
nessuna chiamata di rete fuori dal contesto cowork.

---

## Domande frequenti

**Che differenza c'è da Claude senza plugin?**
Senza plugin, Claude risponde citando normativa italiana ma non ha un
modulo strutturato di auto-controllo: se cita una sentenza con
numerazione anacronistica o un articolo del codice civile incoerente
col concetto descritto, non lo segnala. Con `verifica-fonti` attiva,
quando glielo chiedi (*"controlla le citazioni"*), produce un rapporto
formale citazione-per-citazione con flag di formato, plausibilità e
coerenza. Lavorano in due passi: prima Claude risponde, poi tu — se
intendi usare le citazioni in un atto — gli chiedi il rapporto.

**Posso disinstallare il plugin?**
Sì, da **Customize → Plugin → Remove** accanto a MHC-L.

**Il plugin funziona offline?**
Sì. La skill `verifica-fonti` opera sulla knowledge interna di Claude
+ il testo che le passi. Nessuna chiamata di rete necessaria.

**Posso usarla su testi non legali?**
Sì, ma sarebbe rumore: la skill cerca pattern di citazione normativa
italiana ed europea. Su una bozza di email generica non ha nulla da
segnalare. Usala dopo risposte di Claude che contengono riferimenti
normativi o giurisprudenziali.

**Garantisce che le citazioni siano corrette?**
No. Controlla formato, plausibilità numerica, coerenza contestuale e
indirizza ai registri authoritative per la verifica sostanziale. La
responsabilità professionale finale resta dell'avvocato.

---

## Modalità avanzata (opt-in)

Il plugin ha **due modalità**. Quella descritta sopra — usa
`verifica-fonti` su un testo per controllare le citazioni — copre il
caso d'uso dell'80% degli avvocati. Per chi vuole anche un ecosistema
italiano-curato di skill terze legal-tech, esiste la **modalità
avanzata**, che si attiva solo se la chiedi esplicitamente.

### Come si attiva

In una conversazione Cowork con il plugin installato, scrivi a Claude
qualcosa come:

> *"Apri il bollettino delle skill italiane."*

oppure

> *"Cosa skill ci sono nel catalogo italiano?"*

oppure

> *"Mostrami le skill legal-tech disponibili."*

Questo invoca il `bollettino` e Claude apre la pipeline guidata. Dal
catalogo scegli una skill, lo `skill-installer` la installa applicando
i check di sicurezza (allowlist, license, tier, heuristic) in modo
silenzioso, e successivamente — se serve e lo chiedi tu come **secondo
passo cosciente** — l'`adattamento-italiano` adatta il prompt al
linguaggio giuridico italiano.

### Cosa aspettarsi

- **Prima invocazione lenta.** Il bollettino viene scaricato da GitHub
  raw, e l'eventuale adattamento italiano richiede chiamate LLM
  aggiuntive. Aspettati qualche secondo in più rispetto alla modalità
  default.
- **Rate-limit GitHub.** Se installi molte skill in poco tempo, la
  fetch raw può temporaneamente sospendersi (limite anonimo ~60
  richieste/ora). Aspetta, riprova.
- **Niente persiste tra sessioni Cowork diverse** oltre allo stato
  che Claude Desktop conserva per la conversazione corrente.

### Non è necessario per la maggior parte degli avvocati

Se ti interessa solo verificare le citazioni di un atto, la modalità
default (`verifica-fonti` da sola) basta. La modalità avanzata è uno
strato addizionale per il power-user — non c'è gating che ti spinge
ad attivarla. Se non la invochi, non si attiva.

### Disinstallarla?

Non c'è da disinstallare nulla: la modalità avanzata è una pipeline
inerte finché non la chiami. Se cambi idea, semplicemente non
invocarla più.

---

*DISTRIBUZIONE.md — v3.1.0 — 2026-05-18 (allineato a mhc-l 3.1.0:
default `verifica-fonti` + modalità avanzata opt-in via bollettino).*
