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

Se il catalogo è vuoto (succede al primo rilascio), il plugin te lo
dirà:

> *"Il catalogo è ancora in costruzione. La routine automatica
> `bollettino-research` monitora mensilmente l'ecosistema legal-tech
> open source e pubblica le skill che superano la threshold policy.
> Quando saranno disponibili, le vedrai qui."*

### Aggiornamenti

Il bollettino si aggiorna **automaticamente il primo di ogni mese**
(ore 10:00 italiane in estate, 9:00 in inverno). Le nuove voci
appaiono al prossimo apertura di Cowork.

Per forzare un controllo aggiornamenti **subito** vai su
**Customize → Plugin** e clicca **"Update"** accanto a MHC-L.

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

*DISTRIBUZIONE.md — v2.0.0 — 2026-05-17*
