# Usare BeccarIA con Codex (percorso sperimentale)

Questa pagina serve a chi arriva qui da GitHub e vuole usare la skill
`verifica-fonti` con **Codex**, anche senza familiarità con GitHub,
terminale o sviluppo software.

> **Stato: sperimentale.** Questo percorso è pensato per provare
> `verifica-fonti` in OpenAI Codex. Il canale principale e stabile di
> BeccarIA resta l'installazione Claude Cowork descritta nel README e in
> `DISTRIBUZIONE.md`. Il percorso Codex lavora su una copia locale
> separata e non modifica l'installazione Claude.

L'idea è semplice: apri Codex, copi il prompt qui sotto, e Codex installa
la skill dal repository GitHub, la adatta al proprio ambiente e ti dice
come provarla.

## Che cos'è Codex

Codex è l'agente di OpenAI pensato per lavorare su file e cartelle. Non è
una semplice chat nel browser: può leggere i file della cartella di
lavoro, creare o modificare file, eseguire comandi e spiegarti cosa ha
fatto.

Per un avvocato, la metafora più semplice è questa: **Codex lavora dentro
un fascicolo tecnico**, cioè una cartella. Vede quello che metti in quella
cartella e può operarci sopra. Se la cartella è vuota, può comunque usarla
come spazio di lavoro per installare una skill. Se contiene documenti
riservati, Codex può leggerli: quindi inserisci solo materiale che vuoi
davvero usare in quella sessione.

Secondo la documentazione OpenAI consultata il 24 maggio 2026, Codex è
incluso nei piani ChatGPT Plus, Pro, Business ed Enterprise/Edu. OpenAI
indica anche disponibilità temporanee e limiti diversi a seconda del
piano. Verifica sempre la pagina ufficiale OpenAI prima di dare per
scontato un piano specifico.

Fonte ufficiale:
<https://help.openai.com/en/articles/11369540-using-codex-with-your-chatgpt-plan>

## Prima di iniziare

Ti serve:

1. un account ChatGPT con accesso a Codex;
2. una cartella di lavoro aperta in Codex, anche vuota;
3. una nuova chat Codex in quella cartella;
4. connessione internet, perché Codex deve leggere questo repository
   GitHub.

## Quale progetto scegliere in Codex

Quando Codex ti chiede dove lavorare, il termine **progetto** significa
semplicemente "cartella di lavoro". Non deve essere un progetto
informatico vero e proprio.

Per provare BeccarIA senza rischiare di mischiarla con altri file e
senza scegliere manualmente una cartella, scegli:

```text
Aggiungi nuovo progetto -> Parti da zero
```

Codex creerà una **nuova cartella di lavoro vuota gestita da Codex**:
non è una normale chat web, perché Codex potrà creare file e installare
la skill in quella cartella. Non stai però scegliendo tu una cartella
precisa del tuo computer: la posizione è gestita dall'app.

Se vuoi sapere esattamente dov'è la cartella sul disco, crea prima una
cartella vuota, per esempio `BeccarIA-Codex-test`, e scegli:

```text
Aggiungi nuovo progetto -> Usa una cartella esistente
```

Questa seconda opzione è preferibile se hai già una cartella preparata
per questo test, o se vuoi usare la skill su file contenuti in quella
cartella.

Per questa installazione, **non è consigliato** scegliere "Lavora senza
progetto": la skill va installata e provata meglio dentro una cartella
di lavoro riconoscibile.

Non devi installare tutto il plugin Claude. Per Codex basta partire dalla
skill `verifica-fonti`, che si trova nel percorso:

```text
beccaria/skills/verifica-fonti
```

## Non danneggia l'installazione Claude Cowork

Questo percorso è separato dall'installazione Claude Cowork.

La guida non chiede di modificare il plugin Claude già installato. Codex
deve installare e, se serve, adattare **solo una copia locale per Codex**
della skill `verifica-fonti`.

In particolare, Codex non deve:

1. modificare la cartella `.claude-plugin`;
2. modificare i file sorgente del plugin nel repository GitHub;
3. scrivere in `~/.claude/...`;
4. disinstallare o aggiornare il plugin Claude Cowork;
5. installare l'intero plugin BeccarIA dentro Codex.

Se usi sia Claude Cowork sia Codex, puoi quindi tenere due copie separate:
una per Claude Cowork e una per Codex. Le modifiche di adattamento Codex
restano nella copia Codex.

## Prompt da copiare in Codex

Copia tutto il testo qui sotto in una nuova chat Codex.

```text
Voglio installare e rendere usabile in Codex la skill `verifica-fonti` di BeccarIA.

Repository GitHub: MicheleLoi/legal-tech-cowork
Percorso della skill: beccaria/skills/verifica-fonti

Obiettivo pratico: voglio poter chiedere a Codex di controllare citazioni normative e giurisprudenziali italiane o UE in un testo, usando la skill `verifica-fonti`.

Per favore:

1. Usa la skill di sistema `skill-installer`, se disponibile.
2. Scarica solo `beccaria/skills/verifica-fonti` dal repository GitHub indicato.
3. Installa la skill tra le skill locali di Codex.
4. Non installare l'intero plugin Claude e non installare la skill `skill-installer` contenuta nel repository BeccarIA.
5. Non modificare in alcun modo l'installazione Claude Cowork: non scrivere in `~/.claude/...`, non modificare `.claude-plugin`, non modificare i file sorgente del repository GitHub, non disinstallare e non aggiornare il plugin Claude.
6. Dopo l'installazione, leggi il file `SKILL.md` installato nella copia Codex e verifica se contiene istruzioni specifiche per Claude Desktop, Claude Cowork, `WebFetch`, `mcp__Claude_in_Chrome__...` o percorsi `~/.claude/...`.
7. Se trovi elementi specifici di Claude, adatta solo la copia locale per Codex, senza cambiare il repository GitHub originale e senza toccare l'installazione Claude. In particolare:
   - conserva il contenuto giuridico e i limiti della skill;
   - sostituisci i riferimenti operativi a Claude con riferimenti a Codex;
   - sostituisci `WebFetch` con gli strumenti web disponibili in Codex o con richiesta esplicita di permesso quando serve rete;
   - ignora o rimuovi la dipendenza da `mcp__Claude_in_Chrome__...`, se quel tool non esiste in questa sessione;
   - usa la cartella skill di Codex, oppure un file locale `.beccaria-state.json` nella cartella di lavoro;
   - mantieni chiaro che la verifica delle fonti non sostituisce la valutazione professionale dell'avvocato.
8. Verifica che il frontmatter YAML della skill resti valido: deve avere almeno `name` e `description`.
9. Dimmi se devo riavviare Codex per far caricare la skill.
10. Dammi un mini-test pronto da copiare per verificare che `verifica-fonti` funzioni.

Non limitarti a spiegare cosa fare: fai l'installazione e le modifiche locali necessarie, poi riassumi cosa hai cambiato.
```

## Test dopo l'installazione

Dopo il riavvio di Codex, apri una nuova chat nella stessa cartella e
prova:

```text
Usa `verifica-fonti` su questo testo:
"L'art. 1382 c.c. disciplina la responsabilità extracontrattuale, mentre l'art. 2043 c.c. regola la clausola penale."
```

Risultato atteso: Codex dovrebbe segnalare che le due attribuzioni sono
invertite. In sintesi, l'art. 1382 c.c. riguarda la clausola penale,
mentre l'art. 2043 c.c. riguarda la responsabilità extracontrattuale.

## Cosa aspettarsi

Codex potrebbe dirti che serve riavviare l'app o aprire una nuova chat
per vedere la skill appena installata. È normale: molte installazioni di
Codex leggono l'elenco delle skill all'avvio della sessione.

Durante il primo test, Codex può mostrare un file modificato chiamato:

```text
.beccaria-state.json
```

È normale. È un piccolo promemoria locale della skill, per esempio per
ricordare che l'avviso introduttivo è già stato mostrato. Non tocca
Claude Cowork, non modifica il repository GitHub e non contiene il testo
del tuo caso. Se lo cancelli, la conseguenza pratica è solo che BeccarIA
potrebbe ripetere l'avviso iniziale.

La skill non trasforma Codex in un avvocato e non garantisce che una tesi
sia corretta. Serve a controllare se le citazioni sembrano formalmente
plausibili, coerenti con il testo e, quando possibile, verificabili su
fonti autorevoli.

## Se qualcosa non funziona

Se Codex non trova la skill di sistema `skill-installer`, chiedigli di
cercare nelle skill disponibili un modo per installare una skill da
GitHub.

Se la rete è bloccata, autorizza Codex a leggere il repository GitHub
quando te lo chiede.

Se la skill viene installata ma non si attiva, riavvia Codex e ripeti il
test.
