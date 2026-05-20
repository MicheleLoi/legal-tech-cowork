# Come installare e testare BeccarIA

*Per l'avvocato. Nessun comando terminale, nessun account GitHub, nessuna
conoscenza tecnica richiesta.*

> **BeccarIA** è uno dei tre prodotti del brand ombrello **RegIA**,
> insieme a **MHC** (governance framework) e **Recode IT**
> (pseudonimizzazione web). I tre sono moduli plug-and-play: ciascuno
> standalone, oppure combinabili a stack. Questo documento descrive
> l'installazione di BeccarIA. Per gli altri due moduli vedi
> [micheleloi.pro/regia](https://micheleloi.pro/regia/) (quando la
> guida modulare sarà pubblicata).

---

## Cosa installi

Un plugin per Claude cowork. **Default**: una skill, `verifica-fonti`,
che controlla che le citazioni normative e giurisprudenziali italiane
ed europee in un testo siano formalmente corrette, plausibili e
coerenti con il contesto. Fonti coperte: Cassazione, Corte
Costituzionale, Consiglio di Stato, TAR, Normattiva, EUR-Lex, Garante
Privacy, AGCM, ANAC, Banca d'Italia.

**Modalità avanzata opt-in.** Il plugin include cinque skill aggiuntive
che restano inerti finché non le attivi esplicitamente:

- `catalogo` — gateway al bollettino curato di skill terze italiane
- `skill-installer` — installa skill terze dopo tuo trigger esplicito
- `adattamento-italiano` — adatta al volo skill anglophone a IT/EU
- `ecosystem-scout` — panoramica intelligente dell'ecosistema legal-AI
  open source (Mike, fork nazionali, ecc.)
- `pattern-extractor` — applica pattern dall'ecosistema con attribution
  AGPL obbligatoria

Vedi la sezione **Come testare la modalità avanzata** più sotto. Se non
ti servono, ignorale: il default copre l'80% dei casi.

**Doctrine duale fetch.** La modalità avanzata funziona via trigger
semantici, **niente configurazione di rete da impostare in Claude
Desktop**. Le skill si dividono in due regimi:

- **Pointer** (`catalogo`): non fa fetch in background. Ti indica l'URL
  del bollettino skill e il fraseggio per chiedermi di aprirlo io, e il
  contenuto entra in chat.
- **Autonomous post-trigger esplicito** (`skill-installer`,
  `ecosystem-scout`, `pattern-extractor`): dopo che tu hai scelto
  esplicitamente di installare una skill / chiedere informazioni
  sull'ecosistema / applicare un pattern, la skill fa fetch puntuale del
  payload necessario. Il tuo trigger esplicito è l'autorizzazione. Per
  `ecosystem-scout` e `pattern-extractor` c'è anche un **fallback
  pointer-pure** documentato: se il validator di Claude Desktop blocca
  il fetch autonomous, la skill ricalibra a suggerirti l'URL come fa
  `catalogo`.

Out-of-box in entrambi i casi.

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
5. Nella lista che compare, accanto a **"beccaria"**, clicca **"Add
   plugin"**.

Fatto. Vedrai un messaggio di conferma. Dovrebbero comparire **6 skill**
installate: `verifica-fonti`, `catalogo`, `skill-installer`,
`adattamento-italiano`, `ecosystem-scout`, `pattern-extractor`. Solo la
prima è attiva di default; le altre cinque restano inerti finché non le
invochi esplicitamente (vedi sotto).

> **Nota durante lo sviluppo pre-1.0.** Il push delle nuove versioni su
> GitHub avviene solo dopo verifica del fondatore. Finché nella UI di
> Claude Desktop non vedi `beccaria` versione **4.0.0**, il marketplace
> remoto potrebbe puntare a una release precedente (es. `iuris-it`
> 3.3.1): in quel caso usa il percorso "zip locale" qui sotto, oppure
> attendi il push del founder.

> **Se vedi `iuris-it` invece di `beccaria`.** Il rename è arrivato con
> v4.0.0 (2026-05-19). Le versioni precedenti del plugin si chiamavano
> `iuris-it` (3.x) e prima ancora `mhc-l` (≤2.x). Sono lo stesso
> plugin nella sua evoluzione storica; aggiorna alla 4.0.0 per ottenere
> identità + due skill nuove + supporto AGPL ecosystem.

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
   C:\Users\<nome>\...\beccaria-4.0.0-plugin.zip
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
il plugin risponda davvero. Cinque prove, dalla più semplice (default)
alle più articolate (modalità avanzata, sei skill).

### 1) Test della modalità default — `verifica-fonti`

Esempio concreto:

1. Apri una nuova conversazione **Cowork** (con il plugin `beccaria`
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

**Come riconosci che la skill è davvero attiva.** La risposta di Claude
include un **marker esplicito di modalità** a ogni turn (per esempio
una riga iniziale che indica quale skill sta operando). Se non vedi
alcun marker, la skill non si è attivata: riprova con un trigger più
diretto come *"usa la skill verifica-fonti su questo testo"*.

### 2) Test modalità avanzata — `catalogo` (pointer)

Per chi vuole esplorare l'ecosistema italiano-curato di skill terze.

1. In una conversazione Cowork qualsiasi, scrivi:

   > *"mostrami il catalogo delle skill italiane"*

   Trigger equivalenti: *"apri il bollettino"*, *"che skill posso
   installare?"*, *"mostrami le skill legal-tech disponibili"*.
2. **Atteso**: la skill `catalogo` si attiva e ti punta all'URL del
   bollettino pubblico, suggerendoti il fraseggio per chiedermi di
   aprirlo. La skill **non apre l'URL da sola** — è una tua richiesta
   esplicita che autorizza il fetch.
3. Scrivi (come suggerito):

   > *"apri questo URL e mostrami le skill disponibili"*

   Claude apre l'URL tramite `WebFetch` standard e ti presenta le novità
   del mese, eventuali alert su licenze o publisher.

### 3) Test modalità avanzata — `skill-installer` + `adattamento-italiano`

1. Dal catalogo (test precedente), scegli una skill anglophone — per
   esempio una skill US/UK di NDA review.
2. Scrivi:

   > *"installa la skill [nome]"*

3. **Atteso**: `skill-installer` recupera autonomous il `SKILL.md` della
   skill candidata (il tuo trigger esplicito di installazione autorizza
   il fetch puntuale), esegue silenziosamente cinque check di sicurezza
   (allowlist, tier, license, trust, freshness) e ti mostra una sola
   riga per-tier con un prompt di conferma `sì/no`.
4. Su `sì`, lo `skill-installer` scrive i file della skill in
   `~/.claude/plugins/config/beccaria/installed_skills/[nome]/` e
   appende un'entry all'`install-log.yaml`. **Niente passi UI manuali**:
   la skill è installata e disponibile.
5. Se la skill non dichiara una giurisdizione italiana o europea, ti
   propone l'`adattamento-italiano` come secondo passo.

### 4) Test modalità avanzata — `ecosystem-scout` (nuova in 4.0.0)

Per chi vuole una panoramica dell'ecosistema legal-AI open source
globale (non solo skill terze installabili, ma anche progetti come Mike
e i suoi fork nazionali).

1. In una conversazione Cowork qualsiasi, scrivi:

   > *"che tool open source esistono per estrarre clausole?"*

   Trigger equivalenti: *"esiste un fork italiano di Mike?"*,
   *"panoramica strumenti legal-AI"*, *"che alternative ho per il diritto
   svizzero?"*.
2. **Atteso**: `ecosystem-scout` si attiva, fa `WebFetch` del bollettino
   ecosystem (dal VPS BeccarIA), e risponde con un elenco strutturato:
   per ogni strumento → nome, owner, descrizione, **licenza esplicita**,
   giurisdizione inferita, capabilities, stato (attivo/dormiente). Se
   uno strumento è AGPL, aggiunge nota sulle implicazioni per studio
   legale.
3. Se il fetch fallisce (validator block o network), la skill ricalibra
   automaticamente a pattern pointer-pure: ti dice apertamente di non
   essere riuscita a contattare il bollettino, e ti suggerisce il
   fraseggio per chiederle di aprire l'URL come azione utente.

### 5) Test modalità avanzata — `pattern-extractor` (nuova in 4.0.0)

Per applicare nella conversazione corrente un pattern derivato
dall'ecosistema AGPL (Mike, fork) con attribution obbligatoria.

1. Dopo aver visto in `ecosystem-scout` che uno strumento implementa
   il task che ti serve, scrivi:

   > *"applica l'approccio di [strumento] al mio contratto"*

   Trigger equivalenti: *"usa il pattern di [strumento] su questo
   testo"*, *"fai redline review con il metodo di [strumento]"*.
2. **Atteso**: `pattern-extractor` si attiva, fa `WebFetch` del
   bollettino pattern, recupera il pattern matching, e **espone il
   prefisso di attribution obbligatorio** all'inizio della risposta:

   > Sto applicando un approccio derivato da **[source_repo]**
   > ([owner], **AGPL-3.0**). [Nota sul perché questo pattern è
   > appropriato per il task].

   Poi applica il pattern al task.
3. Se nel bollettino non c'è un pattern matching, la skill **NON inventa
   attribuzioni**: dichiara apertamente l'assenza, propone un pattern
   adiacente (con conferma) oppure procede con approccio generale di
   Claude segnalandolo esplicitamente.

Come per le altre modalità, la presenza dei **marker di modalità** nelle
risposte di Claude conferma che la skill sta effettivamente operando.

---

## Cosa NON aspettarsi

Il plugin è onesto sui propri limiti:

- **Latenza maggiore in modalità avanzata.** L'adattamento + il fetch
  dei bollettini richiede chiamate aggiuntive: qualche secondo in più
  rispetto al default. Normale.
- **Niente polling in background.** Nessuna skill fa fetch ricorrente
  autonomo. Il bollettino viene aperto o quando lo chiedi esplicitamente
  (`catalogo` pointer doctrine) o come reazione a un tuo trigger
  esplicito (`ecosystem-scout`, `pattern-extractor`, `skill-installer`
  autonomous post-trigger).
- **Possibili falsi positivi sui trigger impliciti.** Se chiedi cose
  vagamente assimilabili a "mostrami qualcosa di skill" o "che tool
  esiste per X", `catalogo` o `ecosystem-scout` potrebbero attivarsi
  anche se non lo volevi. Niente viene installato o scaricato senza un
  tuo `sì` esplicito, quindi è recuperabile.
- **`verifica-fonti` non garantisce la correttezza sostanziale.** Una
  citazione formalmente corretta può comunque essere inapplicabile al
  caso concreto. La verifica sostanziale finale resta del professionista.
- **`pattern-extractor` non inventa attribuzioni.** Se l'ecosistema non
  ha un pattern per il tuo task, te lo dice e procede con approccio
  generale senza falsa attribuzione AGPL.

---

## Se qualcosa non funziona

### Non vedo "Crea plugin"

Questa opzione richiede piano Claude Pro o superiore. Verifica nelle
impostazioni del tuo account.

### Il plugin non risponde quando chiedo di verificare le fonti

Vai su **Customize → Plugin** e verifica che accanto a `beccaria` ci sia
il tag "Active". Se attivo ma non risponde, prova a essere esplicito
nella richiesta: *"usa la skill verifica-fonti su questo testo"*. Se
persiste, apri una issue su
`https://github.com/MicheleLoi/legal-tech-cowork/issues`.

### Vedo `beccaria` ma le skill non sono 6

Probabile che il marketplace remoto sia su una versione precedente
(`iuris-it` 3.3.1 con 4 skill, o 2.x con la sola `verifica-fonti`).
In attesa che la 4.0.0 sia pushata, installa via zip locale come
descritto sopra, oppure attendi il push del founder.

### Vedo ancora `iuris-it` invece di `beccaria`

Cache stale di Claude Desktop. Prova a fare logout + login, oppure
ri-aggiungere il marketplace. Il push origin/main della 4.0.0 segue il
gate di smoke test del founder.

### `ecosystem-scout` o `pattern-extractor` falliscono il fetch

Il validator di Claude Desktop ha bloccato il fetch autonomous su
dominio custom (problema noto della UI di configurazione allowlist).
Le due skill includono un **fallback pointer-pure** documentato: ti
suggeriscono il fraseggio per chiedere a Claude di aprire l'URL come
azione utente. Segui quel fraseggio e il bollettino entra in contesto
come fetch user-initiated.

---

## Privacy

Il plugin opera **dentro la sandbox Claude cowork**, isolata per
conversazione. Le cartelle che colleghi a una conversazione sono
visibili al plugin; nient'altro del tuo computer lo è.

In modalità default, il plugin **non invia dati** a server di terze
parti oltre al normale traffico con Anthropic. Nessuna telemetria,
nessun tracking, nessun analytics.

In modalità avanzata, le uniche chiamate di rete esterne sono letture
di bollettini JSON pubblici dal VPS BeccarIA (o GitHub pubblico per
fallback pointer-pure): il bollettino skill lo apri tu chiedendolo
esplicitamente a Claude; il bollettino ecosystem / patterns lo recupera
la skill autonomous post-tuo-trigger. In tutti i casi è una tua azione
esplicita ad autorizzare la chiamata di rete, mai un polling nascosto
in background.

I bollettini servono solo metadati pubblici (GitHub API + descrizioni
README) e pattern verbatim da repo pubblici AGPL. Nessun dato del tuo
caso o del tuo testo passa attraverso i bollettini.

---

## Domande frequenti

**Che differenza c'è da Claude senza plugin?**
Senza plugin, Claude risponde citando normativa italiana ma non ha un
modulo strutturato di auto-controllo: se cita una sentenza con
numerazione anacronistica o un articolo del codice civile incoerente
col concetto descritto, non lo segnala. Con `verifica-fonti` attiva,
quando glielo chiedi, produce un rapporto formale citazione-per-citazione
con flag di formato, plausibilità e coerenza. Le skill ecosystem
(`ecosystem-scout`, `pattern-extractor`) aprono accesso strutturato ai
pattern del legal-AI open source italiano + globale.

**Posso disinstallare il plugin?**
Sì, da **Customize → Plugin → Remove** accanto a `beccaria`.

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

**Perché ora è AGPL e prima era MIT?**
Solo le **due skill nuove** di v4.0.0 (`ecosystem-scout`,
`pattern-extractor`) sono sotto AGPL-3.0, per coerenza con l'ecosistema
legal-AI open source AGPL che consumano (Mike e fork). Tutti gli altri
componenti BeccarIA originali restano MIT (`verifica-fonti`,
`adattamento-italiano`, documentazione, bollettino schema). I file
forkati da `anthropics/claude-for-legal` restano Apache-2.0. Vedi
`LICENSE`, `LICENSE-AGPL`, `LICENSE-ANTHROPIC` per i dettagli.

**Cosa è RegIA?**
RegIA è il brand ombrello / azienda che produce BeccarIA + due altri
moduli: **MHC** (governance framework per AI-assisted knowledge work,
audience cross-domain) e **Recode IT** (pseudonimizzazione web,
audience consumer/prosumer). I tre moduli sono plug-and-play. La
combinazione completa MHC + BeccarIA + Recode IT è la "versione
modulare" di quello che storicamente è stato chiamato MHC-L (ora
disaggregato in moduli separati). Vedi
[micheleloi.pro/regia](https://micheleloi.pro/regia/) per il quadro
strategico.

---

*DISTRIBUZIONE.md — BeccarIA v4.0.0 — 2026-05-19. Sei skill totali
(`verifica-fonti` default; `catalogo`, `skill-installer`,
`adattamento-italiano`, `ecosystem-scout`, `pattern-extractor` modalità
avanzata opt-in). Doctrine duale fetch (pointer per `catalogo`,
autonomous post-trigger esplicito per gli altri) con fallback pointer
documentato per le skill ecosystem. Multi-license (AGPL-3.0 per nuove
skill ecosystem + MIT per BeccarIA originale + Apache-2.0 per parti
Anthropic forkate). Posizionamento: modulo plug-and-play sotto RegIA.*
