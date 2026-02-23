# Identificazione automatica del Discorso Indiretto Libero nel corpus narrativo italiano (1850–1929): un esperimento comparativo tra annotatori umani e Large Language Models

---

## Abstract

Il presente capitolo descrive un esperimento di narratologia computazionale volto a valutare la capacità di diversi sistemi di annotazione automatica nell'identificare il Discorso Indiretto Libero (DIL) in un corpus di narrativa italiana del periodo 1850–1929. Lo studio confronta le prestazioni di quattro sistemi — un baseline rule-based fondato sul pattern matching, due configurazioni del modello Claude Sonnet 4.5 (denominate rispettivamente v1 e v2) e il modello GPT-5.2 con *reasoning effort* ridotto — su un campione stratificato di 500 trigrammi, annotato manualmente come gold standard. I risultati mostrano che GPT-5.2 raggiunge un'accuracy del 77,0% e un Cohen's κ di +0,540, classificato come accordo moderato-buono, distanziandosi considerevolmente dai due sistemi basati su Claude Sonnet (accuracy compresa tra 56,4% e 58,8%, κ tra +0,128 e +0,176). L'analisi inter-annotatore tra le due versioni di Claude Sonnet rivela un accordo notevolmente basso (45,6%), con κ = −0,054, suggerendo instabilità sistematica nelle prestazioni del medesimo modello in sessioni distinte. Il capitolo documenta inoltre l'infrastruttura software sviluppata per la conduzione dell'esperimento, discute le fonti di errore per ciascun sistema e formula alcune ipotesi interpretative sui differenziali di performance osservati.

---

## 1. Introduzione

Il Discorso Indiretto Libero (DIL, in inglese *Free Indirect Discourse*, FID) è una tecnica narrativa che consiste nella rappresentazione del pensiero o del discorso di un personaggio attraverso la voce del narratore, senza l'impiego di verbi dichiarativi espliciti (*disse*, *pensò*, *si chiese*) né di congiunzioni subordinanti. In quanto fenomeno di confine tra la prospettiva deittica del narratore e quella del personaggio, il DIL occupa da decenni una posizione centrale negli studi di narratologia (Bally 1912; Banfield 1982; Fludernik 1993; Caracciolo 2014) e rappresenta uno dei costrutti più sfidanti per i sistemi di elaborazione automatica del linguaggio naturale (Brunner 2013; Semino & Short 2004).

L'annotazione manuale del DIL è notoriamente difficile anche per gli esseri umani: studi di inter-rater agreement mostrano concordanze tipicamente comprese tra il 60% e l'80% a seconda della granularità definitoria adottata (Muzny et al. 2017; Krestel et al. 2008). Queste caratteristiche rendono il DIL un banco di prova particolarmente idoneo per valutare le capacità di comprensione pragmatica e stilistica dei Large Language Models (LLMs) al di là delle tradizionali metriche di valutazione su compiti di NLP più convenzionali.

L'esperimento descritto in questo capitolo si inserisce in un programma di ricerca più ampio sulla narratologia computazionale italiana e persegue tre obiettivi principali: (1) stabilire se i LLMs attuali siano in grado di identificare il DIL con precisione accettabile per applicazioni di analisi letteraria su larga scala; (2) confrontare le prestazioni di diversi modelli e configurazioni sperimentali in condizioni controllate; (3) valutare la variabilità intra-modello (*inter-session consistency*) come indicatore di affidabilità metodologica per l'annotazione automatica.

---

## 2. Il corpus

Il corpus di riferimento è composto da 36 opere di narrativa italiana in prosa pubblicate nel periodo 1850–1929, per un totale di 87.841 frasi e 29.293 trigrammi sequenziali (unità di tre frasi consecutive non sovrapposte). Il corpus è stato assembleato a partire da testi liberamente disponibili attraverso fonti digitali (Wikisource, Progetto Gutenberg, Liber Liber), pre-processati con strumenti di tokenizzazione e segmentazione a livello di frase, e normalizzati secondo uno schema di annotazione morfologica e strutturale uniforme.

Le opere incluse coprono un arco temporale che va dall'Unità d'Italia alle soglie del Modernismo, rappresentando autori canonici (Giovanni Verga, Luigi Capuana, Federico De Roberto, Antonio Fogazzaro) accanto a figure meno studiate della narrativa italiana di fine Ottocento e primo Novecento (Enrico Annibale Butti, Grazia Deledda, Ada Negri, Milly Dandolo, tra altri). La diversità stilistica del corpus garantisce condizioni sperimentali ecologicamente valide per testare la robustezza dei sistemi di annotazione.

Il corpus master è conservato nel file `corpus_labelled.csv`, strutturato secondo le colonne `author`, `work`, `year`, `doc_id`, `sentence_id`, `text` e `DIL` (annotazione binaria: *yes*/*no*). Il file dei trigrammi sequenziali è `corpus_labelled-trigrams.csv`, nel quale ogni record aggrega tre frasi consecutive dello stesso documento.

---

## 3. Il campione sperimentale e il gold standard

Per rendere la valutazione computazionalmente fattibile e statisticamente controllata, è stato estratto un campione stratificato di 500 trigrammi dal corpus completo. La stratificazione è stata eseguita in modo da ottenere esattamente 250 istanze DIL = *yes* e 250 istanze DIL = *no* (bilanciamento perfetto 50/50), evitando così potenziali distorsioni nelle metriche di classificazione binaria associate a dataset sbilanciati.

Il campione è stato annotato manualmente da un ricercatore esperto in stilistica e narratologia italiana, che ha operato secondo criteri definitòri esplicitati in un manuale di annotazione interno. La definizione operativa adottata (cfr. §4) corrisponde alla nozione classica di DIL presente nella letteratura critica: assenza di verbi dichiarativi introduttori, mantenimento della terza persona, presenza di marcatori della soggettività del personaggio. L'annotazione umana costituisce il gold standard rispetto al quale vengono valutate tutte le prestazioni dei sistemi automatici.

Il campione di 500 trigrammi è disponibile nel file `corpus_labelled-trigrams_500_sample.csv`; la versione con le annotazioni dei diversi sistemi automatici è progressivamente arricchita nei file `corpus_labelled-trigrams_500_LLM_annotated.csv` e `corpus_labelled-trigrams_500_DUAL_annotated.csv`.

---

## 4. Definizione operativa del Discorso Indiretto Libero

Ai fini dell'annotazione automatica, è stata adottata la seguente definizione operativa del DIL:

> Il Discorso Indiretto Libero è una tecnica narrativa che rappresenta il pensiero o il discorso di un personaggio **senza verbi dichiarativi espliciti** (come *pensò*, *disse*, *si chiese*), mantenendo la terza persona grammaticale e il punto di vista soggettivo del personaggio.

I marcatori diagnostici considerati comprende:

- **Assenza di verbi dichiarativi introduttori**: la principale caratteristica discriminante
- **Interrogative retoriche**: segnalano la prospettiva dubitante del personaggio
- **Esclamazioni ed interiezioni emotive**: *Oh!*, *Ah!*, *Dio mio!*
- **Shift deittico temporale**: uso di tempi verbali coerenti con il punto di vista del personaggio piuttosto che con quello del narratore
- **Sintassi frammentata**: frasi nominali, ellissi, strutture anacolute
- **Modalizzatori epistemici**: *forse*, *certamente*, *senza dubbio*
- **Lessico soggettivo**: vocabolario valutativo e affettivo congruente con il personaggio

Sono stati identificati i seguenti **casi difficili** che possono indurre classificazioni errate nei sistemi automatici:

1. **DIL parziale** (*blended narration*): la sintassi è quella del narratore ma il punto di vista è del personaggio
2. **Narrazione con tono valutativo**: il narratore usa un lessico connotato senza cedere la prospettiva
3. **DIL dopo verbo dichiarativo**: la frase successiva a un verbo introduttore può essere autonomamente DIL

---

## 5. Sistemi di annotazione automatica

### 5.1 Baseline rule-based: pattern matching

Come baseline di riferimento è stato sviluppato un sistema di pattern matching basato su regole esplicite NLP. Il sistema analizza ciascun trigramma ricercando un insieme di indicatori linguistici predefiniti (interrogative retoriche, esclamazioni, interiezioni, puntini di sospensione, marcatori modali) e assegna un punteggio composito sulla base della presenza cumulata di tali pattern. La soglia di classificazione è stata calibrata su un sottoinsieme di sviluppo separato. Il sistema è implementato in Python e fa uso della libreria `re` per il matching di espressioni regolari e di `spaCy` per l'analisi morfologica.

Il baseline costituisce un punto di riferimento metodologicamente rilevante poiché permette di distinguere le prestazioni dei LLMs da quelle ottenibili attraverso euristiche linguistiche esplicite, non dipendenti da capacità di comprensione contestuale del significato.

### 5.2 Claude Sonnet 4.5 — versione 1 (configurazione conservativa)

Il primo sistema LLM è stato Claude Sonnet 4.5 (Anthropic, 2025), accessibile tramite l'API di Anthropic in modalità subagent all'interno di Claude Code. Il sistema riceve in input ciascun trigramma e produce un giudizio binario (*yes*/*no*) sulla presenza di DIL, applicando il proprio ragionamento linguistico come LLM senza ricorso a regole programmatiche esplicite.

La configurazione v1 è caratterizzata da una strategia **conservativa**: il sistema classifica come DIL soltanto i casi in cui i marcatori diagnostici sono multipli, chiari e concordanti, mostrando una tendenza sistematica al falso negativo in presenza di segnali ambigui. Questa configurazione emerge dall'interazione tra la distribuzione a priori del modello sul task e le istruzioni del prompt, che enfatizzano la necessità di ragionamento linguistico autentico.

Il prompt somministrato comprende: la definizione operativa del DIL, quattro esempi annotati (due positivi, due negativi), e la descrizione dei casi difficili. Il sistema è stato eseguito in sessione separata e i risultati sono conservati nella colonna `DIL_Sonnet` del file `corpus_labelled-trigrams_500_LLM_annotated.csv`.

### 5.3 Claude Sonnet 4.5 — versione 2 (configurazione liberale/multi-criterio)

La seconda configurazione del medesimo modello (v2) adotta una strategia **liberale**, basata su un approccio di scoring multi-criterio. Il prompt arricchisce significativamente la definizione del DIL includendo una lista più ampia di marcatori sintattici, semantici e prosodici, con la conseguenza che il sistema tende ad attivare la classificazione *yes* in presenza anche di segnali deboli o isolati.

Questa configurazione è stata eseguita in una sessione separata e indipendente, con l'obiettivo esplicito di misurare la **variabilità inter-sessione** dello stesso modello. I risultati rivelano un bias sistematico verso la sovra-classificazione: il 70,4% dei trigrammi viene classificato come DIL (contro il 50% del gold standard). I risultati sono conservati nella colonna `DIL_Sonnet_v2` del file `corpus_labelled-trigrams_500_Sonnet_v2_annotated.csv`.

### 5.4 GPT-5.2 con reasoning effort LOW

Il quarto sistema è GPT-5.2 (OpenAI, 2025), un modello con capacità di ragionamento avanzato (*extended thinking*). Il modello è stato interrogato tramite la nuova Responses API di OpenAI (`client.responses.create()`) con il parametro `reasoning={"effort": "low"}`, che attiva una catena di ragionamento interno controllata e riduce i costi computazionali rispetto alla configurazione a pieno sforzo.

Il prompt somministrato a GPT-5.2 è formalmente equivalente a quello usato per Claude Sonnet v1, al fine di rendere la comparazione il più possibile controllata. Il sistema elabora i trigrammi in batch da 5 unità e include un meccanismo di salvataggio progressivo ogni 50 righe per prevenire la perdita di dati in caso di interruzione. I risultati sono conservati nella colonna `DIL_gpt_5_2` del file `corpus_labelled-trigrams_500_DUAL_annotated.csv`.

---

## 6. Infrastruttura software

L'infrastruttura sperimentale è stata interamente sviluppata in Python 3 con l'ausilio delle librerie `pandas` (manipolazione dati), `openai` (≥1.0.0, API GPT), `anthropic` (API Claude), `scikit-learn` (metriche di valutazione) e `tqdm` (barre di avanzamento).

Il componente principale per l'annotazione GPT è lo script `annotate_dil_gpt_500_v2.py`, che implementa le seguenti funzionalità:

- **Gestione dell'input**: lettura del file CSV con i 500 trigrammi, validazione della struttura e delle colonne richieste
- **Interfaccia con l'API OpenAI**: utilizzo della Responses API (`client.responses.create()`) con parametri configurabili (`--model`, `--reasoning-effort`, `--batch-size`)
- **Prompt engineering**: costruzione dinamica del prompt con definizione del DIL, esempi annotati e istruzione di risposta strutturata
- **Salvataggio progressivo**: scrittura su CSV ogni 50 annotazioni per garantire la persistenza dei dati
- **Valutazione automatica** (`--eval`): calcolo di accuracy, precision, recall, F1-score, specificity e Cohen's κ rispetto al gold standard, con esportazione delle metriche in formato JSON
- **Confronto inter-annotatore** (`--compare`): calcolo del tasso di accordo e del Cohen's κ rispetto all'annotazione Claude Sonnet preesistente

Lo script è eseguito tramite lo script bash `test_gpt52_low.sh`, che gestisce la verifica delle variabili d'ambiente (chiave API), l'installazione delle dipendenze, la verifica dei file di input e la raccolta dei metadati di esecuzione (durata totale, file generati).

---

## 7. Metriche di valutazione

Per ciascun sistema di annotazione automatica sono state calcolate le seguenti metriche standard di classificazione binaria, rispetto al gold standard umano:

- **Accuracy** (proporzione di classificazioni corrette sul totale)
- **Precision** (proporzione di veri positivi sul totale dei positivi predetti)
- **Recall** / **Sensitivity** (proporzione di veri positivi sul totale dei positivi reali)
- **Specificity** (proporzione di veri negativi sul totale dei negativi reali)
- **F1-Score** (media armonica di precision e recall)
- **Cohen's κ** (accordo inter-annotatore corretto per il caso)

Il Cohen's κ è interpretato secondo la scala convenzionale di Landis e Koch (1977): κ < 0 = accordo peggiore del caso; 0,00–0,20 = scarso; 0,21–0,40 = discreto; 0,41–0,60 = moderato; 0,61–0,80 = buono; > 0,80 = ottimo.

Poiché il campione è perfettamente bilanciato (250 yes / 250 no), la proporzione di accordo atteso per caso risulta Pe = 0,50 per qualsiasi distribuzione di previsione. Conseguentemente vale la relazione: κ = 2 × accuracy − 1, che semplifica l'interpretazione dei risultati.

---

## 8. Risultati

### 8.1 Performance comparata

La tabella seguente riassume le metriche di classificazione per ciascun sistema rispetto al gold standard umano.

| Sistema               | Accuracy | Precision | Recall | Specificity | F1-Score | Cohen's κ |
|-----------------------|----------|-----------|--------|-------------|----------|-----------|
| Pattern matching      | 69,0%    | —         | —      | —           | —        | +0,380    |
| Claude Sonnet v1      | 56,4%    | 57,0%     | 52,4%  | 60,4%       | 54,6%    | +0,128    |
| Claude Sonnet v2      | 58,8%    | 56,2%     | 79,2%  | 38,4%       | 65,8%    | +0,176    |
| **GPT-5.2 (low)**     | **77,0%**| **78,7%** | **74,0%** | **80,0%** | **76,3%** | **+0,540** |

*Nota*: I valori di precision, recall e specificity del pattern matching non sono riportati in quanto non disponibili per quella configurazione sperimentale.

I dati indicano una differenza sostanziale tra i sistemi testati. GPT-5.2 con *reasoning effort* LOW supera Claude Sonnet v1 di 20,6 punti percentuali in accuracy e di 21,6 punti in F1-Score; la distanza da Claude Sonnet v2 è rispettivamente di 18,2 e 10,5 punti. Il pattern matching baseline raggiunge una accuracy del 69%, superiore a entrambe le configurazioni Claude ma inferiore a GPT-5.2 di 8 punti percentuali, il che suggerisce che parte delle prestazioni dei sistemi rule-based sia già incorporata nelle capacità di Claude Sonnet.

È degno di nota il comportamento differenziale delle due configurazioni Claude sul piano precision/recall. Claude Sonnet v1 mantiene una relazione bilanciata tra precision (57,0%) e recall (52,4%), con un profilo classificatorio relativamente simmetrico. Claude Sonnet v2 presenta invece un forte sbilanciamento: recall elevato (79,2%) a scapito di precision (56,2%) e specificity (38,4%), indicando una strategia classificatoria orientata alla minimizzazione dei falsi negativi con conseguente proliferazione di falsi positivi.

### 8.2 Matrici di confusione

Le matrici di confusione per i tre sistemi LLM sono riportate di seguito. In ciascuna matrice, le righe rappresentano le classi reali (gold standard), le colonne le classi predette. Le celle riportano il conteggio assoluto e la percentuale rispetto alla classe reale.

#### Claude Sonnet v1

```
                   Predetto NO        Predetto YES
  Reale NO (250)   151 (60,4%)         99 (39,6%)
  Reale YES (250)  119 (47,6%)        131 (52,4%)
```

- **True Negatives (TN)**: 151 — il 60,4% dei trigrammi non-DIL è correttamente classificato
- **True Positives (TP)**: 131 — solo il 52,4% dei trigrammi DIL è correttamente identificato
- **False Positives (FP)**: 99 — il 39,6% dei non-DIL è erroneamente classificato come DIL
- **False Negatives (FN)**: 119 — il 47,6% dei DIL reali viene mancato

#### Claude Sonnet v2

```
                   Predetto NO        Predetto YES
  Reale NO (250)    96 (38,4%)        154 (61,6%)
  Reale YES (250)   52 (20,8%)        198 (79,2%)
```

- **True Negatives (TN)**: 96 — soltanto il 38,4% dei non-DIL è correttamente classificato
- **True Positives (TP)**: 198 — il 79,2% dei DIL reali è correttamente identificato
- **False Positives (FP)**: 154 — il 61,6% dei non-DIL è erroneamente classificato come DIL
- **False Negatives (FN)**: 52 — solo il 20,8% dei DIL reali viene mancato

Il confronto tra v1 e v2 rivela una inversione sistematica nel tipo di errore dominante: v1 produce prevalentemente falsi negativi (FN=119), v2 prevalentemente falsi positivi (FP=154). Questo pattern è interpretabile come il risultato di strategie prompt-indotte di regolazione della soglia classificatoria piuttosto che come differenza nelle capacità di comprensione del fenomeno.

#### GPT-5.2 (reasoning effort LOW)

```
                   Predetto NO        Predetto YES
  Reale NO (250)   200 (80,0%)         50 (20,0%)
  Reale YES (250)   65 (26,0%)        185 (74,0%)
```

- **True Negatives (TN)**: 200 — l'80,0% dei non-DIL è correttamente classificato
- **True Positives (TP)**: 185 — il 74,0% dei DIL reali è correttamente identificato
- **False Positives (FP)**: 50 — solo il 20,0% dei non-DIL è erroneamente classificato
- **False Negatives (FN)**: 65 — il 26,0% dei DIL reali viene mancato

GPT-5.2 mostra una distribuzione degli errori notevolmente più equilibrata rispetto a entrambe le configurazioni Claude. La specificità (80,0%) e la recall (74,0%) sono entrambe elevate, segnalando che il modello riesce a discriminare efficacemente tra DIL e narrazione oggettiva in entrambe le direzioni.

### 8.3 Distribuzione delle classificazioni

La tabella seguente riporta la distribuzione delle etichette assegnate da ciascun sistema, in confronto con il gold standard.

| Sistema           | Etichette YES    | Etichette NO     | Bias rispetto al gold |
|-------------------|------------------|------------------|-----------------------|
| Gold standard     | 250 (50,0%)      | 250 (50,0%)      | —                     |
| Claude Sonnet v1  | 230 (46,0%)      | 270 (54,0%)      | −4,0% (sotto-classificazione) |
| Claude Sonnet v2  | 352 (70,4%)      | 148 (29,6%)      | +20,4% (sovra-classificazione) |
| GPT-5.2 (low)     | 235 (47,0%)      | 265 (53,0%)      | −3,0% (lieve sotto-classificazione) |

GPT-5.2 produce una distribuzione quasi identica al gold standard (47% vs 50%), con un bias residuo minimo di −3 punti percentuali. Claude Sonnet v2 mostra il bias più pronunciato, con una sovra-classificazione sistematica di oltre 20 punti percentuali.

### 8.4 Accordo inter-annotatore

La tabella seguente riporta il tasso di accordo e il Cohen's κ tra i diversi pari di annotatori, incluso il confronto tra le due sessioni Claude.

| Coppia di annotatori              | Accordo (%) | Cohen's κ | Interpretazione       |
|----------------------------------|-------------|-----------|------------------------|
| Claude Sonnet v1 vs Gold         | 56,4%       | +0,128    | Scarso                 |
| Claude Sonnet v2 vs Gold         | 58,8%       | +0,176    | Scarso                 |
| GPT-5.2 vs Gold                  | 77,0%       | +0,540    | Moderato-buono         |
| Claude Sonnet v1 vs v2           | 45,6%       | −0,054    | Peggiore del caso      |

Il dato più critico riguarda la concordanza tra le due sessioni Claude Sonnet: con un accordo del 45,6% e κ = −0,054, le due versioni producono annotazioni praticamente anti-correlate, una condizione peggiore di quella attesa per un classificatore casuale. Questo risultato ha implicazioni metodologiche rilevanti: indica che i sistemi LLM possono essere altamente sensibili a variazioni nella formulazione del prompt, nella configurazione della sessione e nella temperatura di campionamento, producendo risultati non riproducibili tra sessioni distinte.

Per contestualizzare il risultato di GPT-5.2 (κ = +0,540), si noti che valori di κ tra +0,40 e +0,60 sono tipicamente considerati accettabili in ambito di annotazione linguistica, anche per compiti complessi come l'identificazione di fenomeni pragmatici e stilistici.

### 8.5 Analisi dei disaccordi tra le sessioni Claude

Tra le 272 istanze di disaccordo tra Claude Sonnet v1 e v2 (54,4% del campione totale), la tipologia di discordanza è asimmetrica: 197 casi (72,4%) corrispondono a un'inversione v1=*no* → v2=*yes*, mentre soltanto 75 casi (27,6%) mostrano l'inversione opposta v1=*yes* → v2=*no*. Questa asimmetria è coerente con il bias di sovra-classificazione documentato per v2.

Rispetto al gold standard, i 272 casi di disaccordo si ripartiscono come segue: Claude Sonnet v2 fornisce la risposta corretta in 142 casi (52,2%), mentre Claude Sonnet v1 è corretto in 130 casi (47,8%). La differenza non è statisticamente rilevante e suggerisce che le due versioni non esprimano strategie classificatorie qualitativamente distinte, bensì varianti di soglia di un medesimo processo sottostante.

### 8.6 Analisi degli errori: marcatori linguistici associati alle classificazioni errate

L'analisi qualitativa dei casi di errore nel sistema Claude Sonnet v2 permette di identificare i marcatori linguistici più frequentemente associati a falsi positivi e falsi negativi.

**Falsi Positivi (154 casi)**: i marcatori più frequenti nei FP sono la sintassi frammentata (80,5% dei FP) e lo shift temporale (74,7%). Questi dati suggeriscono che il sistema interpreta strutture sintattiche spezzate o costruzioni narrative con variazione temporale come indicatori necessari di DIL, anche quando sono impiegati per altri scopi stilistici (es. anacoluto narratoriale, effetto retorico della voce narrante onnisciente).

**Falsi Negativi (52 casi)**: nei FN prevale ancora lo shift temporale (51,9%) e la sintassi frammentata (21,2%), con 4 casi in cui sono presenti verbi dichiarativi che hanno probabilmente inibito il riconoscimento del DIL nella frase successiva. Questo profilo suggerisce che casi di *blended narration* — in cui il DIL è incorporato in un contesto sintattico privo di marcatori forti — risultino sistematicamente sottoriconosciuti.

---

## 9. Performance per autore

L'analisi della performance di Claude Sonnet v2 disaggregata per autore (per i 10 autori con almeno 15 trigrammi nel campione) rivela una variabilità considerevole, con accuracy comprese tra il 39,1% (Capuana) e l'82,4% (Rovetta).

| Autore                      | Trigrammi | Accuracy |
|-----------------------------|-----------|----------|
| Rovetta Gerolamo            | 17        | 82,4%    |
| Negri Ada                   | 48        | 81,2%    |
| Deledda Grazia              | 47        | 76,6%    |
| Dandolo Milly               | 30        | 70,0%    |
| Butti Enrico Annibale       | 60        | 73,3%    |
| Campanile Achille           | 36        | 55,6%    |
| De Roberto Federico         | 19        | 47,4%    |
| Capuana Luigi               | 23        | 39,1%    |

La bassa performance su Capuana (39,1%) e De Roberto (47,4%) è coerente con il fatto che entrambi gli autori impiegano il DIL in forme particolarmente sfumate e integrate nel tessuto narrativo (il cosiddetto "stile indiretto libero verista"), senza ricorrere ai marcatori sintattici più prototipici. Questo risultato suggerisce che i LLMs tenderanno a generalizzare meglio su forme canoniche di DIL che su implementazioni stilisticamente idiosincratiche.

---

## 10. Discussione

I risultati dell'esperimento permettono di formulare le seguenti osservazioni:

**10.1 Differenziale di performance tra GPT-5.2 e Claude Sonnet.** Il distacco di circa 20 punti percentuali in accuracy tra GPT-5.2 e le configurazioni Claude Sonnet è sostanziale e difficilmente spiegabile con sole differenze di prompt engineering. Una possibile spiegazione è che il meccanismo di *extended thinking* di GPT-5.2, anche nella configurazione a sforzo ridotto, consenta al modello di effettuare inferenze multi-step sul testo narrativo che risultano inaccessibili a Claude Sonnet nelle condizioni testate. Una spiegazione alternativa è che le due famiglie di modelli differiscano nelle rispettive distribuzioni a priori sui concetti stilistici della tradizione narrativa italiana, il che richiederebbe uno studio sistematico con probing o interpretability analysis per essere verificato.

**10.2 Instabilità inter-sessione di Claude Sonnet.** L'accordo del 45,6% tra le due sessioni Claude (κ = −0,054) costituisce il risultato più critico dell'intero esperimento dal punto di vista metodologico. Esso implica che l'annotazione con Claude Sonnet nelle condizioni testate non soddisfa il requisito minimo di riproducibilità richiesto per qualsiasi applicazione scientifica. Questo problema potrebbe essere in parte mitigato da tecniche di *prompt freezing*, *temperature=0*, o dall'adozione di approcci ensemble che aggregano più sessioni indipendenti dello stesso modello.

**10.3 Pattern matching come baseline competitivo.** Il baseline rule-based raggiunge il 69% di accuracy, superando entrambe le configurazioni Claude Sonnet. Questo risultato, apparentemente paradossale, è plausibilmente spiegabile con la natura del task: il DIL in testi narrativi dell'Ottocento italiano è spesso segnalato da marcatori superficiali relativamente stabili (esclamazioni, interrogative retoriche, puntini di sospensione), che un sistema rule-based è in grado di cogliere con affidabilità. I LLMs, tuttavia, sono in grado di riconoscere il DIL in assenza di tali marcatori (DIL "implicito"), un punto di forza che emerge soprattutto nella configurazione ad alto recall di Claude Sonnet v2, ma che in assenza di adeguata calibrazione produce tassi elevati di falsi positivi.

**10.4 Implicazioni per la narratologia computazionale.** I risultati indicano che GPT-5.2 con *reasoning effort* LOW raggiunge prestazioni compatibili con l'uso in applicazioni di annotazione assistita su larga scala (accuracy 77%, κ +0,540), a condizione di mantenere un supervisore umano per la revisione dei casi incerti. Claude Sonnet, nelle configurazioni testate, non raggiunge invece soglie di affidabilità metodologicamente accettabili per questo tipo di applicazione.

---

## 11. Conclusioni

Il presente capitolo ha documentato un esperimento sistematico di valutazione della performance di diversi sistemi di annotazione automatica nell'identificazione del Discorso Indiretto Libero in un corpus di narrativa italiana del periodo 1850–1929. I risultati principali sono:

1. GPT-5.2 con *reasoning effort* LOW raggiunge un'accuracy del 77,0% e un Cohen's κ di +0,540, il valore più elevato tra i sistemi testati e classificabile come accordo moderato-buono.
2. Claude Sonnet 4.5 nelle due configurazioni testate (v1 conservativa e v2 liberale) produce accuracy comprese tra 56,4% e 58,8% e κ tra +0,128 e +0,176 (accordo scarso), con profili di errore sistematicamente opposti.
3. Il pattern matching rule-based, pur raggiungendo il 69% di accuracy, non consente il riconoscimento del DIL in assenza di marcatori superficiali espliciti.
4. L'accordo inter-sessione tra le due versioni Claude (κ = −0,054) evidenzia una problematica di non-riproducibilità che rende il sistema inadatto per applicazioni di annotazione sistematica senza un'esplicita procedura di stabilizzazione.

Ricerche future dovranno investigare: (a) se le prestazioni di Claude Sonnet possano essere migliorate attraverso tecniche di few-shot learning con esempi tratti dallo stesso corpus; (b) se configurazioni di GPT-5.2 a maggiore *reasoning effort* producano miglioramenti statisticamente significativi rispetto alla configurazione LOW; (c) se approcci ensemble che combinino annotazioni da sessioni multiple dello stesso LLM possano ridurre la variabilità inter-sessione osservata.

---

## Riferimenti bibliografici

Bally, C. (1912). Le style indirect libre en français moderne. *Germanisch-Romanische Monatschrift*, 4, 549–556.

Banfield, A. (1982). *Unspeakable sentences: Narration and representation in the language of fiction*. Routledge.

Brunner, A. (2013). Automatic recognition of speech, thought, and writing representation in German narrative texts. *Literary and Linguistic Computing*, 28(4), 563–575.

Caracciolo, M. (2014). *The experientiality of narrative: An enactivist approach*. De Gruyter.

Fludernik, M. (1993). *The fictions of language and the languages of fiction: The linguistic representation of speech and consciousness*. Routledge.

Krestel, R., Bergler, S., & Witte, R. (2008). Minding the source: Automatic tagging of reported speech in newspaper articles. In *Proceedings of LREC 2008*, 2823–2828.

Landis, J. R., & Koch, G. G. (1977). The measurement of observer agreement for categorical data. *Biometrics*, 33(1), 159–174.

Muzny, G., Fitz, M., Chang, A., & Jurafsky, D. (2017). A two-stage sieve approach for quote attribution. In *Proceedings of EACL 2017*, 460–470.

Semino, E., & Short, M. (2004). *Corpus stylistics: Speech, writing and thought presentation in a corpus of English writing*. Routledge.

---

*Nota metodologica*: Tutti i file di dati, gli script di annotazione e i risultati sperimentali sono disponibili nella cartella del progetto (`corpus_labelled.csv`, `corpus_labelled-trigrams.csv`, `corpus_labelled-trigrams_500_DUAL_annotated.csv`, `annotate_dil_gpt_500_v2.py`, `test_gpt52_low.sh`). Le metriche di valutazione per GPT-5.2 sono esportate in formato JSON nei file `corpus_labelled-trigrams_500_DUAL_annotated_metrics_gpt_5_2.json` e `corpus_labelled-trigrams_500_DUAL_annotated_comparison_gpt_5_2.json`.
