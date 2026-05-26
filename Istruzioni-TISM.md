
## CONTESTO E OBIETTIVO

Devo eseguire reverse engineering di un programma COBOL legacy per produrre
documentazione utile alla riscrittura su nuova piattaforma. L'output finale
sono due documenti distinti, prodotti in due fasi separate:

FASE A — Pseudocodifica AS-IS: rappresentazione strutturata del programma
COBOL in pseudo-codice, fedele al sorgente, organizzata per paragrafi/sezioni,
con riferimenti precisi alle righe del codice. Serve a capire COSA FA il
programma.

FASE B — Documento di Analisi Funzionale AS-IS: estrazione delle regole di
business, dei dati, delle dipendenze e degli aspetti operativi a partire
dalla pseudocodifica. Serve a capire PERCHÉ lo fa e COSA NON DEVE PERDERSI
in una riscrittura.

Le due fasi sono separate intenzionalmente: la pseudocodifica è uno strato
intermedio verificabile autonomamente, e il documento di Analisi Funzionale
costruisce sopra di essa senza dover continuamente rileggere il sorgente.

## MATERIALI

- Sorgente COBOL: [nome programma e percorso]
- Eventuali copybook collegati: [percorsi]
- Schema delle tabelle DB (DB2/SQL Server): [percorsi]
- Documentazione collaterale (specifiche storiche, manuali utente, JCL, ecc.):
  [percorsi]
- Eventuali sorgenti di programmi chiamati o chiamanti (subroutine RDSUT*,
  programmi CALL, ecc.): [percorsi]

## VINCOLI METODOLOGICI

Prima di partire, applica questi principi e dichiarali esplicitamente:

1. FEDELTÀ AL CODICE: la pseudocodifica deve riflettere il comportamento
   reale del codice, non quello che "dovrebbe" fare secondo intuizioni o
   pattern noti. Se trovi un controllo o un bypass apparentemente strano,
   non normalizzarlo: marcalo come anomalia da chiarire.

2. RIFERIMENTI DI RIGA: ogni paragrafo della pseudocodifica deve riferire
   il range di righe del sorgente che traduce. Questo serve a verificare e
   ad aggiornare in caso di modifiche al sorgente.

3. BLACK-BOX ESPLICITE: le CALL a sottoprogrammi non disponibili (es.
   gestione fiscale, telematico, utility di sistema) vanno trattate come
   black-box, documentando solo input/output dichiarati. NON inventare
   logica interna.

4. LIVELLO DI DETTAGLIO PROPORZIONATO: variabili di lavoro tecniche,
   contatori di servizio, indici di tabelle interne NON vanno nella
   pseudocodifica al primo livello. Vanno solo i flussi di controllo, le
   letture/scritture DB, le CALL, le decisioni di business. La pseudocodifica
   non è una traduzione 1:1 del COBOL, è una traduzione interpretativa
   strutturata.

5. SEPARAZIONE TRA OSSERVAZIONE E INTERPRETAZIONE: nella Fase A scrivi solo
   ciò che il codice fa. Le interpretazioni ("questo controllo serve a...",
   "probabilmente è un legacy di una versione precedente") vanno tenute per
   la Fase B o segnalate come NOTA / DOMANDA APERTA.

## PROCESSO IN 5 STEP

Ti chiedo di seguire questi step e di fermarti per validazione a ciascuno.

### STEP 0 — Riconoscimento e domande preliminari

Prima di leggere il codice in profondità:
- Apri il sorgente e fammi un riassunto strutturale: numero di righe,
  struttura DIVISION, elenco SECTION/paragrafi principali, elenco delle
  CALL esterne, elenco delle tabelle DB lette/scritte, elenco dei copybook
  inclusi.
- Identifica il PUNTO DI INGRESSO e i PUNTI DI USCITA del programma.
- Dichiara esplicitamente quali sono le aree che ti aspetti di trattare
  come black-box (sottoprogrammi non disponibili).
- Chiedimi se ci sono priorità: certe sezioni più rilevanti di altre,
  parti già documentate da non ripercorrere, contesti di business da
  conoscere prima di partire.

NON iniziare a scrivere pseudocodifica in questo step.

### STEP 1 — Pseudocodifica AS-IS, prima passata grezza

Produci una pseudocodifica completa del programma con questi criteri:
- Una sezione per ogni paragrafo principale del COBOL, con riferimento
  al range di righe
- Pseudo-codice in italiano (o nella lingua scelta), strutturato con
  IF/ELSE, CICLI, CALL, READ/WRITE, MOVE — ma con nomi di variabili
  parlanti dove possibile
- Per ogni READ/WRITE: indica la tabella, le condizioni di ricerca, i
  campi letti/scritti
- Per ogni CALL: indica programma chiamato, parametri di input/output,
  comportamento atteso (se la CALL è black-box, dichiararlo esplicitamente)
- Per ogni decisione di business identificata: marca con [BUSINESS_RULE]
- Per ogni anomalia, comportamento sospetto, codice morto, o pattern
  che non capisci: marca con [DA_CHIARIRE: domanda specifica]
- Mantieni una sezione "Variabili globali significative" con
  l'elenco dei campi di lavoro (working storage) usati per decisioni
  o flussi (escluso il rumore tecnico)

Output: file Markdown strutturato, con TOC e numerazione gerarchica dei
paragrafi.

### STEP 2 — Pseudocodifica AS-IS, raffinamento

A partire dalla prima passata:
- Verifica coerenza: i flussi di controllo collegati correttamente, le
  variabili usate dopo essere state valorizzate, le letture coerenti con
  gli schemi DB
- Consolida i [DA_CHIARIRE] in un elenco numerato (PA-01, PA-02, ...)
  con breve descrizione e riferimento di riga
- Aggiungi una sezione iniziale "Sintesi del programma" (max 1 pagina)
  che descrive lo scopo, gli input principali, gli output principali, le
  dipendenze esterne
- Aggiungi un diagramma di flusso testuale del paragrafo principale di
  orchestrazione (se esiste un MAIN o PARAGRAFO-PILOTA)

Fermati qui per validazione PRIMA di passare alla Fase B. La pseudocodifica
è un deliverable autonomo che deve essere approvato da chi conosce il sistema.

### STEP 3 — Estrazione delle entità per il documento di Analisi Funzionale

Una volta validata la pseudocodifica, comincia l'estrazione tematica:
- ENTITÀ DATI: tabelle DB lette/scritte, campi chiave, prefissi e
  convenzioni di naming, relazioni inter-tabella desumibili dal codice
- REGOLE DI BUSINESS: estrazione delle [BUSINESS_RULE] marcate in Fase A,
  classificate per tipo (eligibilità, calcolo, esclusione, soglia,
  controllo formale, ecc.). Codifica con prefisso (es. RE-XX eligibilità,
  RC-XX calcolo, RX-XX esclusione, RS-XX soglia).
- DIPENDENZE: lista delle CALL in entrata (chi chiama questo programma)
  e in uscita (chi viene chiamato), con scopo funzionale
- ASPETTI OPERATIVI: codici di ritorno, gestione errori, transazionalità
  (commit/rollback, COMMIT periodici), modalità di esecuzione (online vs
  batch), tracciamenti
- AMBIGUITÀ E DOMANDE APERTE: i PA-XX sopravvissuti dalla pseudocodifica,
  riformulati come domande funzionali e non più tecniche

Restituiscimi questa estrazione come gap analysis testuale, NON come
documento finale: voglio rivedere insieme la classificazione delle regole
prima che diventino capitolo del documento.

### STEP 4 — Documento di Analisi Funzionale AS-IS

A partire dall'estrazione validata, produci un documento Word (.docx)
strutturato in capitoli:

Cap. 1 — Introduzione (scopo del documento, ambito, glossario)

Cap. 2 — Quadro generale del programma (cosa fa, in quale flusso si
inserisce, principali interlocutori sistema)

Cap. 3 — Input e trigger di esecuzione (come viene innescato, parametri
di ingresso, condizioni di partenza)

Cap. 4 — Flusso AS-IS (descrizione narrativa del flusso principale,
suddivisa per fasi macroscopiche, ogni fase con riferimento ai paragrafi
di pseudocodifica)

Cap. 5 — Modello dati (tabelle, campi chiave, convenzioni)

Cap. 6 — Regole di business (RE/RC/RX/RS classificate)

Cap. 7 — Dipendenze (CALL in entrata e in uscita, programmi correlati,
sottoprogrammi black-box e relativi contratti di interfaccia)

Cap. 8 — Aspetti operativi (codici ritorno, errori, transazionalità,
output telematici, log)

Cap. 9 — Considerazioni TO-BE (sezione propositiva: aree di refactoring,
debiti tecnici evidenziati, opportunità di parametrizzazione, dipendenze
da rompere)
Appendici:

- App. A — Ambiguità e questioni aperte (PA-XX) per i referenti
- App. B — Domande funzionali per i business owner
- App. C — Mapping pseudocodifica → capitoli del documento

Convenzioni grafiche del documento (Sirio Design System se INPS,
altrimenti template aziendale):
- Font Titillium Web (o Calibri come fallback)
- Palette istituzionale (header colorati, tabelle con header scuro testo
  bianco)
- Indice con TOC automatico
- Numerazione capitoli e paragrafi gerarchica

## STILE DI COLLABORAZIONE

- Agisci come mentore rigoroso. Non concordare per default. Se trovi un
  pattern nel codice che non capisci, NON inventare: marcalo come
  [DA_CHIARIRE].
- Distingui sempre tra ciò che è OSSERVATO nel codice e ciò che è
  INTERPRETATO. La pseudocodifica deve essere osservativa; le
  interpretazioni vanno marcate.
- Quando il codice contiene comportamenti apparentemente sbagliati
  (bypass, valori magici, flag con nomi enigmatici), NON normalizzarli:
  documentali e chiedi.
- Procedi sempre per step, fermandoti a validazione. Una pseudocodifica
  approvata è il prerequisito di un documento di analisi affidabile. Non
  produrre il documento di analisi prima della validazione della
  pseudocodifica.
- Se alla fine di Step 1 ti accorgi che il programma è molto più grande
  di quanto valutato in Step 0, fermati e propondi una suddivisione
  (per es. "trattiamo prima il flusso principale, poi le ramificazioni
  secondarie in una sessione successiva").
- Se ti accorgi che certe sezioni richiederebbero un sorgente che non hai
  (es. copybook mancante, sottoprogramma chiave non fornito), dichiaralo
  esplicitamente in Step 0 e indica l'impatto sulla qualità della
  pseudocodifica.

## VINCOLI TECNICI

- Pseudocodifica: file Markdown (.md) ben strutturato
- Documento di Analisi Funzionale: file Word (.docx)
- Lingua: italiano (terminologia tecnica in inglese accettata: SQL, CALL,
  COMMIT, ecc.)
- Riferimenti di riga sempre presenti: formato [righe XXXX-YYYY]
- Codifica delle ambiguità: PA-XX progressivo, conservato dalla
  pseudocodifica al documento di analisi