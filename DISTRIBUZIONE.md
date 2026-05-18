# Come installare MHC-L

*Per l'avvocato. Nessun comando terminale, nessun account GitHub, nessuna
conoscenza tecnica richiesta.*

---

## Video walkthrough (asset primario)

**[Inserire qui link al video walkthrough screen-recording founder-recorded]**

Il video dura ~3 minuti e mostra l'intero percorso di installazione su
Claude Desktop reale, click per click. È l'asset primario di
distribuzione: in 5 minuti netti hai il plugin attivo.

Se per qualche motivo non puoi vedere il video, segue la versione
testuale di fallback (accessibilità).

---

## Versione testuale di fallback (5 click)

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
   "aggiungere marketplace esistente". (Questo è un noto gotcha
   dell'interfaccia Claude Desktop Pro standard a maggio 2026.)
4. Nel dialog **"Aggiungi marketplace"** incolla:

   ```
   https://github.com/MicheleLoi/legal-tech-cowork
   ```

   Premi **"Add"**.
5. Nella lista che compare, accanto a **"mhc-l"**, clicca
   **"Add plugin"**.

Fatto. Vedrai un messaggio di conferma. Il plugin è attivo.

### Primo utilizzo

In una nuova conversazione Cowork scrivi:

> *"Mostrami il catalogo"*

Al primo utilizzo apparirà un breve disclaimer (una volta sola) che
chiede conferma di aver letto. Poi vedrai il catalogo.

Vedrai una nota che dice che il catalogo è la copia installata con
il plugin: è normale, è la modalità di default. Per l'opzione
"sempre aggiornato in tempo reale" vedi la pagina istruzioni:
[`https://micheleloi.pro/mhc-l/istruzioni/`](https://micheleloi.pro/mhc-l/istruzioni/).

Se il catalogo è vuoto (succede al primo rilascio), il plugin te lo
dirà:

> *"Il catalogo è ancora in costruzione. La routine automatica
> `bollettino-research` monitora mensilmente l'ecosistema legal-tech
> open source e pubblica le skill che superano la threshold policy.
> Quando saranno disponibili, le vedrai qui."*

### Aggiornamenti del bollettino

Il bollettino vive come file pubblico sul repo GitHub del plugin. La
routine automatica `bollettino-research` lo aggiorna **il 17 di ogni
mese alle 22:40 ora italiana** (CEST estate / CET inverno),
aggiungendo le nuove skill legal-tech open source che hanno superato
la threshold policy.

**Modalità di default — copia installata con il plugin.** Il catalogo
che vedi è la versione del bollettino fissata al momento in cui hai
installato (o aggiornato) il plugin in Claude Desktop. Le nuove voci
pubblicate dalla routine mensile **non sono visibili** finché non
aggiorni il plugin. Per quel caso le sezioni di gestione plugin di
Claude Desktop sono la via canonica (Customize → Plugin).

Se vuoi che il bollettino si aggiorni automaticamente ad ogni
apertura del catalogo senza dipendere dagli aggiornamenti plugin,
vedi la pagina istruzioni:
[`https://micheleloi.pro/mhc-l/istruzioni/`](https://micheleloi.pro/mhc-l/istruzioni/).

---

## Se qualcosa non funziona

### Non vedo "Crea plugin"

Questa opzione richiede piano Claude Pro o superiore. Verifica nelle
impostazioni del tuo account.

### Il plugin non risponde quando chiedo il catalogo

Vai su **Customize → Plugin** e verifica che accanto a MHC-L ci sia
il tag "Active". Se attivo ma non risponde, apri una issue su
`https://github.com/MicheleLoi/legal-tech-cowork/issues` con una
breve descrizione.

### Errori di rete quando provo a installare una skill

Il plugin scarica il codice della skill da GitHub al momento
dell'installazione. Verifica la connessione internet. Se la rete è OK
ma l'errore persiste, segnalalo via issue.

---

## Privacy

Il plugin opera **dentro la sandbox Claude cowork**, isolata per
conversazione. Le cartelle che colleghi a una conversazione sono
visibili al plugin; nient'altro del tuo computer lo è.

Il plugin **non invia dati** a server di terze parti oltre a:

- GitHub (per scaricare il bollettino e il codice delle skill — entrambi
  pubblici);
- Anthropic (per le normali chiamate a Claude — stesso traffico di una
  qualsiasi conversazione Cowork).

Nessuna telemetria, nessun tracking, nessun analytics.

---

## Domande frequenti

**Posso disinstallare il plugin?**
Sì, da **Customize → Plugin → Remove** accanto a MHC-L.

**Le skill installate restano se disinstallo MHC-L?**
No. Le skill installate vivono dentro la cartella di configurazione del
plugin e vengono rimosse con esso.

**Posso suggerire una skill da aggiungere al bollettino?**
Sì, apri una issue sul repo con il link. La routine `bollettino-research`
la valuta in modalità ad-hoc applicando la stessa threshold policy delle
voci automatiche.

**Il plugin funziona offline?**
No per il primo download del bollettino. Sì per skill già installate
(operano dentro la sandbox Cowork senza ulteriori chiamate al bollettino).

---

## Modalità live (opzionale)

*Per uso avanzato. Non necessaria per usare il plugin nella sua
versione default. Versione web della guida con screenshot:
[`https://micheleloi.pro/mhc-l/istruzioni/`](https://micheleloi.pro/mhc-l/istruzioni/).*

Se vuoi che il catalogo sia sempre aggiornato all'ultima versione
del bollettino senza dover aspettare gli aggiornamenti del plugin,
puoi consentire al plugin di leggere il file direttamente dal repo
GitHub. Operazione una volta sola, va fatta nelle impostazioni di
Claude Desktop.

**Procedura:**

1. Apri Claude Desktop, vai in **Impostazioni** (icona ingranaggio
   in basso a sinistra, oppure scorciatoia da tastiera del tuo
   sistema operativo).
2. Cerca la sezione **Connettori → Egress allowlist** (può chiamarsi
   anche *"Domini consentiti"* o *"Outbound network allowlist"* a
   seconda della versione Claude Desktop).
3. Aggiungi il dominio:
   ```
   raw.githubusercontent.com
   ```
4. Salva le impostazioni.

Da quel momento, ogni volta che chiedi al plugin di mostrarti il
catalogo, scarica il bollettino aggiornato direttamente dal repo
GitHub — senza più dipendere dagli aggiornamenti plugin per vedere
le nuove voci.

**Perché non è attivo di default:** Claude Desktop tiene i plugin
isolati a livello di rete come scelta di sicurezza standard. Il
dominio `raw.githubusercontent.com` serve solo a leggere file
pubblici di GitHub, non comporta rischi materiali; ma l'opt-in
manuale resta a tua discrezione. Senza, il plugin funziona
comunque — semplicemente vedi la copia installata.

---

*DISTRIBUZIONE.md — v2.1.0 — 2026-05-18 (REV2.5 onboarding revision)*
