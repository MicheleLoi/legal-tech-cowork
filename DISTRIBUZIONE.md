# Come installare e testare iuris-it

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
opzionale di skill terze italiano-curate (catalogo + installer +
adattamento), che resta inerte finché non lo attivi esplicitamente.
Vedi la sezione **Come testare la modalità avanzata** più sotto. Se
non ti serve, ignorala: il default copre l'80% dei casi.

Versione web della guida con screenshot:
[`https://micheleloi.pro/iuris-it/istruzioni/`](https://micheleloi.pro/iuris-it/istruzioni/).

---

## Prerequisiti

- Claude Desktop installato (scaricabile da
  [claude.ai/download](https://claude.ai/download))
- Piano Claude Pro o superiore (il piano gratuito non include plugin
  di terze parti su cowork)

---

## Installazione — percorso standard (GitHub marketplace)

Questo è il percorso che useranno tutti gli avvocati. Cinque click in
Claude Desktop.

1. **Apri Claude Desktop**, vai sulla tab **Cowork** nella sidebar
   sinistra.
2. Sempre nella sidebar di Cowork, clicca **Customize → Plugin → "+"**
   in alto a destra.
3. Nel menu che si apre clicca **"Crea plugin"** — sì, "Crea plugin",
   anche se non stai creando nulla: è il path UX counter-intuitivo per
   "aggiungere un marketplace esistente". (Noto gotcha dell'interfaccia
   Claude Desktop Pro standard a maggio 2026.)
4. Nel dialog **"Aggiungi marketplace"** incolla:

   ```
   https://github.com/MicheleLoi/legal-tech-cowork
   ```

   Premi **"Add"**.
5. Nella lista che compare, accanto a **"iuris-it"**, clicca
   **"Add plugin"**.

Fatto. Vedrai un messaggio di conferma. Dovrebbero comparire **4 skill**
installate: `verifica-fonti`, `catalogo`, `skill-installer`,
`adattamento-italiano`. Solo la prima è attiva di default; le altre tre
restano inerti finché non le invochi (vedi sotto).

> **Nota durante lo sviluppo pre-1.0.** Il push delle nuove versioni su
> GitHub avviene solo dopo verifica del fondatore. Finché nella UI di
> Claude Desktop non vedi `iuris-it` versione **3.2.0**, il marketplace
> remoto potrebbe puntare a una release precedente: in quel caso usa
> il percorso "zip locale" qui sotto.

---

## Installazione — percorso alternativo (zip locale)

Per chi ha già una copia del plugin sul proprio computer (fondatore,
tester, copia ricevuta via SwitchDrive o e-mail).

Path UI **diverso** dal precedente: si usa il pulsante **"Aggiungi
plugin"** (non "Aggiungi marketplace").

1. Claude Desktop → tab **Cowork** → **Customize → Plugin → "+"** in
   alto a destra.
2. Clicca **"Aggiungi plugin"**.
3. Seleziona il file zip del plugin sul tuo computer, ad esempio:

   ```
   C:\Users\<nome>\...\iuris-it-3.2.0-plugin.zip
   ```

4. Claude Desktop estrae e installa il plugin. Comparirà nella lista
   plugin con tag "Active".

In alternativa allo zip, al passo 4 della procedura standard puoi
incollare il **percorso assoluto della cartella locale** del repo
(es. `C:\Users\<nome>\legal-tech-cowork`) al posto dell'URL GitHub: il
resto del flusso è identico.

---

## Come testare il plugin

Una volta installato, conviene fare un giro di test per verificare che
il plugin risponda davvero. Tre prove, dalla più semplice (default)
alla più articolata (advanced).

### 1) Test della modalità default — `verifica-fonti`

Esempio concreto:

1. Apri una nuova conversazione **Cowork** (con il plugin `iuris-it`
   attivo).
2. Chiedi a Claude:

   > *"Stendi un parere sulla nuova disciplina del whistleblowing nel
   > d.lgs. 24/2023, citando giurisprudenza recente."*

3. Claude produce un parere con citazioni (norme, sentenze di
   Cassazione, provvedimenti del Garante, ecc.).
4. Scrivi:

   > *"verifica le fonti di questo parere"*

   Varianti equivalenti: *"controlla le citazioni"*, *"queste citazioni
   reggono?"*, *"passa l'output a verifica-fonti"*.
5. **Atteso**: la skill `verifica-fonti` si attiva e restituisce un
   rapporto strutturato citazione-per-citazione — quali sono
   formalmente corrette, quali sono sospette (formato anomalo,
   numerazione implausibile, possibile refuso), quali non si risolvono
   a una norma reale.

**Variante più diretta**: in qualsiasi conversazione incolla un tuo
testo con citazioni e chiedi *"verifica queste fonti"*.

**Come riconosci che la skill è davvero attiva.** La risposta di
Claude include un **marker esplicito di modalità** a ogni turn (per
esempio una riga iniziale che indica quale skill sta operando). Se
non vedi alcun marker, la skill non si è attivata: riprova con un
trigger più diretto come *"usa la skill verifica-fonti su questo
testo"*.

### 2) Test della modalità avanzata — `catalogo`

Per chi vuole esplorare l'ecosistema italiano-curato di skill terze.

1. In una conversazione Cowork qualsiasi, scrivi:

   > *"mostrami il catalogo delle skill italiane"*

   Trigger equivalenti: *"apri il bollettino"*, *"che skill posso
   installare?"*, *"mostrami le skill legal-tech disponibili"*.
2. **Atteso**: la skill `catalogo` si attiva, presenta il bollettino
   curato (la lista già inclusa nel plugin) e — **prima** di scaricare
   l'elenco aggiornato da GitHub — ti chiede un consenso esplicito.
   Questo perché il fetch GitHub è una chiamata di rete: la skill non
   esce dalla sandbox cowork senza tuo via libera.
3. Se autorizzi, scarica l'elenco fresh e ti mostra le skill disponibili
   con eventuali alert/novità. Se non autorizzi, usa l'elenco bundled
   nel plugin.

### 3) Test della modalità avanzata — `skill-installer` + `adattamento-italiano`

1. Dal catalogo (test precedente), scegli una skill anglophone — per
   esempio una skill US/UK di NDA review.
2. Scrivi:

   > *"installa la skill [nome]"*

3. **Atteso**: `skill-installer` esegue silenziosamente cinque check di
   sicurezza (allowlist sorgenti, verifica licenza, integrità
   strutturale, scan euristico, freshness) e ti mostra solo una riga
   per-tier (tier 1 ok / tier 2 ok / tier 2 WARN / REFUSE). Chiede
   approvazione esplicita prima di scrivere qualsiasi file.
4. Se l'installazione va a buon fine e la skill non dichiara una
   giurisdizione italiana o europea (campo `jurisdiction` `[?]`,
   `other`, `none`, oppure mancante), ti viene chiesto direttamente
   sì/no se vuoi un adattamento italiano.
5. Su **"sì"**: `adattamento-italiano` legge la skill installata,
   traduce il prompt, mappa i riferimenti normativi originali agli
   equivalenti italiani/EU plausibili, marca con `[VERIFICA]` ogni
   riferimento che richiede controllo manuale, esegue un pre-flight
   `verifica-fonti` e ti mostra un diff per approvazione esplicita
   prima di sovrascrivere.

Come per la modalità default, la presenza dei **marker di modalità**
nelle risposte di Claude conferma che le skill stanno effettivamente
operando.

---

## Cosa NON aspettarsi

La modalità avanzata è onesta sui propri limiti:

- **Latenza maggiore.** Adattamento e fetch GitHub richiedono chiamate
  aggiuntive: qualche secondo in più rispetto al default. Normale.
- **Rate-limit GitHub.** L'API raw di GitHub limita le fetch anonime a
  ~60 richieste/ora. Se installi molte skill in poco tempo, può
  sospendersi temporaneamente: aspetta, riprova.
- **Possibili falsi positivi sui trigger impliciti.** Se chiedi cose
  vagamente assimilabili a "mostrami qualcosa di skill", il `catalogo`
  potrebbe attivarsi anche se non lo volevi. Non scrive nulla senza
  conferma esplicita, quindi è recuperabile: basta dirgli di
  fermarsi.
- **`verifica-fonti` non garantisce la correttezza sostanziale.** Una
  citazione formalmente corretta può comunque essere inapplicabile al
  caso concreto. La verifica sostanziale finale resta del
  professionista.

---

## Se qualcosa non funziona

### Non vedo "Crea plugin"

Questa opzione richiede piano Claude Pro o superiore. Verifica nelle
impostazioni del tuo account.

### Il plugin non risponde quando chiedo di verificare le fonti

Vai su **Customize → Plugin** e verifica che accanto a `iuris-it` ci sia
il tag "Active". Se attivo ma non risponde, prova a essere esplicito
nella richiesta: *"usa la skill verifica-fonti su questo testo"*. Se
persiste, apri una issue su
`https://github.com/MicheleLoi/legal-tech-cowork/issues`.

### Vedo `iuris-it` ma le skill non sono 4

Probabile che il marketplace remoto sia su una versione precedente
(2.x con la sola `verifica-fonti`). In attesa che la 3.2.0 sia pushata,
installa via zip locale come descritto sopra.

---

## Privacy

Il plugin opera **dentro la sandbox Claude cowork**, isolata per
conversazione. Le cartelle che colleghi a una conversazione sono
visibili al plugin; nient'altro del tuo computer lo è.

In modalità default, il plugin **non invia dati** a server di terze
parti oltre al normale traffico con Anthropic. Nessuna telemetria,
nessun tracking, nessun analytics.

In modalità avanzata, l'unica chiamata di rete esterna è la fetch del
bollettino da GitHub raw — e avviene solo dopo tuo consenso esplicito,
mai automaticamente.

---

## Domande frequenti

**Che differenza c'è da Claude senza plugin?**
Senza plugin, Claude risponde citando normativa italiana ma non ha un
modulo strutturato di auto-controllo: se cita una sentenza con
numerazione anacronistica o un articolo del codice civile incoerente
col concetto descritto, non lo segnala. Con `verifica-fonti` attiva,
quando glielo chiedi (*"controlla le citazioni"*), produce un rapporto
formale citazione-per-citazione con flag di formato, plausibilità e
coerenza.

**Posso disinstallare il plugin?**
Sì, da **Customize → Plugin → Remove** accanto a `iuris-it`.

**Il plugin funziona offline?**
La modalità default sì: `verifica-fonti` opera sulla knowledge interna
di Claude più il testo che le passi, nessuna chiamata di rete. La
modalità avanzata richiede rete (solo se la invochi).

**Posso usare `verifica-fonti` su testi non legali?**
Sì, ma sarebbe rumore: la skill cerca pattern di citazione normativa
italiana ed europea. Usala dopo risposte di Claude che contengono
riferimenti normativi o giurisprudenziali.

**Garantisce che le citazioni siano corrette?**
No. Controlla formato, plausibilità numerica, coerenza contestuale e
indirizza ai registri authoritative per la verifica sostanziale. La
responsabilità professionale finale resta dell'avvocato.

---

*DISTRIBUZIONE.md — v3.2.0 — 2026-05-19 (rename plugin mhc-l → iuris-it;
onboarding riallineato post install-verification: default `verifica-fonti`
+ modalità avanzata opt-in via catalogo, con sezione "Come testare"
step-by-step).*
