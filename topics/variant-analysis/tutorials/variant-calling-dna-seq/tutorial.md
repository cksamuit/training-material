---
layout: tutorial_hands_on

title: Variant Calling DNA SEQ Workflow
zenodo_link: https://zenodo.org/records/10519826
questions:
- What are the steps to perform a variant analysis?
- How do you perform this type of analysis on Galaxy?
- Which Galaxy Toolshed tools can we use?
objectives:
- Describe the types of data formats encountered during variant calling.
- Learn the basics of Galaxy
- Learn how to run the tools
- Learn how to create a workflow
time_estimation: ''
key_points:
- Galaxy provides an easy-to-use graphical user interface for often complex commandline tools
- Workflows enable you to repeat your analysis on different data
contributors:
- contributor1
- contributor2

---


# Introduction

<!-- This is a comment. -->

Per lo svolgimento di questo tuorial ci si basera su un set di dati di sequenziamento a lungo termine di una popolazione di Escherichia coli.
Eseguiremo l'identificazione delle varianti per vedere come la popolazione è cambiata nel tempo. Vogliamo capire come questa popolazione è 
cambiata rispetto alla popolazione originale, il ceppo REL606 di E. coli. Pertanto, allineeremo ciascuno dei nostri campioni al genoma di riferimento REL606 di E. coli 
e vedremo quali differenze esistono nelle nostre letture rispetto al genoma.

> <agenda-title></agenda-title>
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# Background

....

## Che cos'è l'Escherichia coli?
Escherichia coli è un batterio a forma di bastoncino che può sopravvivere in un’ampia varietà di condizioni, tra cui temperature variabili, 
disponibilità di nutrienti e livelli di ossigeno. La maggior parte dei ceppi è innocua, ma alcuni sono associati ad avvelenamenti alimentari.

## Perché l'E. coli è importante?
L’E. coli è uno degli organismi modello più studiati in ambito scientifico. Essendo un organismo unicellulare, l’E. coli si riproduce rapidamente, 
raddoppiando in genere la sua popolazione ogni 20 minuti, il che significa che può essere facilmente manipolato negli esperimenti. Inoltre, 
la maggior parte dei ceppi di E. coli presenti in natura è innocua. Soprattutto, la genetica dell’E. coli è abbastanza ben compresa e può essere 
manipolata per studiare l’adattamento e l’evoluzione.

## I dati
I dati che utilizzeremo fanno parte di un esperimento di evoluzione a lungo termine condotto da [Richard Lenski](https://en.wikipedia.org/wiki/E._coli_long-term_evolution_experiment).
L'esperimento è stato progettato per valutare l'adattamento in E. coli. Una popolazione è stata propagata per oltre 40.000 generazioni in un terreno di coltura 
minimo limitato al glucosio (nella maggior parte delle condizioni il glucosio è la migliore fonte di carbonio per l'E. coli, in quanto fornisce una crescita più 
rapida rispetto ad altri zuccheri). Questo terreno è stato integrato con citrato, che E. coli non può metabolizzare nelle condizioni aerobiche dell'esperimento. 
Il sequenziamento delle popolazioni a intervalli regolari ha rivelato che la variante spontanea che utilizza il citrato (Cit+) è comparsa tra 31.000 e 31.500 generazioni, 
causando un aumento delle dimensioni e della diversità della popolazione. Inoltre, **questo esperimento ha mostrato l'ipermutabilità in alcune regioni**. 
L'ipermutabilità è importante e può contribuire ad accelerare l'adattamento a nuovi ambienti, ma può anche essere selezionata contro le popolazioni ben adattate.

## I metadati
Lavoreremo con tre eventi campione del ceppo Ara-3 di questo esperimento, uno da 5.000 generazioni, uno da 15.000 generazioni e uno da 50.000 generazioni. 
La popolazione è cambiata in modo sostanziale nel corso dell'esperimento e ne esploreremo l'evoluzione attraverso lo studio della chiamata delle varianti. Il file di 
metadati associato a questa lezione può essere visualizzato su [Github](https://github.com/datacarpentry/wrangling-genomics/blob/gh-pages/files/Ecoli_metadata_composite.csv). 
Se volete conoscere i dettagli di come è stato creato il file, potete consultare alcune note e fonti [qui](https://github.com/datacarpentry/wrangling-genomics/blob/gh-pages/files/Ecoli_metadata_composite_README.md).

Questi metadati descrivono le informazioni sui cloni Ara-3 e le colonne che li rappresentano:

| Colonna | Descrizione |
| --- | --- |
| strain | strain name |
| generation | generation when sample frozen |
| clade | based on parsimony-based tree |
| reference | study the samples were originally sequenced for |
| population | ancestral population group |
| mutator | hypermutability mutant status |
| facility | facility samples were sequenced at |
| run | Sequence read archive sample ID |
| read_type | library type of reads |
| read_length | length of reads in sample |
| sequencing_depth | depth of sequencing |
| cit | citrate-using mutant status |



# Introduzione a Galaxy
**Galaxy** è una piattaforma web che permette l’esecuzione di flussi di lavoro riproducibili e versatili a ricercatori, normali utenti e appassionati, che non necessariamente 
possiedono una certa familiarità con la ricerca ad elevata intensità di dati.

La piattaforma, open source con licenza accademica gratuita, è stata sviluppata presso Penn State, Johns Hopkins, Oregon Health & Science University e Cleveland Clinic, 
da un team composto da bioinformatici e ingegneri del sofware, grazie al contributo di diversi enti.

Il funzionamento di Galaxy è basato su tre concetti fondamentali: **accessibilità**, **riproducibilità** e **trasparenza**. In pratica lo scopo della piattaforma è garantire a 
qualunque utente inesperiente con la programmazione, il caricamento dei dati, l’esecuzione dei vari strumenti di analisi e dei diversi flussi di lavoro, e la visualizzazione dei 
dati ottenuti. In Galaxy, le analisi computazioni possono essere ripetute facilmente, oltre che condivise o pubblicate.

La homepage di Galaxy è suddivisa in tre diversi pannelli: **Strumenti** (a sinistra in blu), **Visualizzazione** (al centro in rosso) e **Cronologia** (a destra in verde). 
Nella parte alta, al centro (in giallo), c’è invece un menù riportante le voci: **Homepage**, **Workflow**, **Visualizza**, **Condivisione dati**, **Aiuto** e **Utente**.

![Galaxy](https://training.galaxyproject.org/training-material/topics/introduction/images/galaxy_interface.png)

L’aspetto dell’homepage di Galaxy potrebbe, tuttavia, subire leggere modifiche e seconda del server che si utilizza. Il principale server di Galaxy è *usegalaxy.org*, mentre 
quello europeo è *usegalaxy.eu*.

## Login in Galaxy
Visitiamo il sito https://usegalaxy.eu/ ed effettuaimo il **login o la registrazione** per accedere al nostro account, cliccando sulla voce "**Login or register**", situato in alto
nel pannello centrale.  

![Galaxy](https://training.galaxyproject.org/training-material/topics/introduction/images/galaxy-login.png)

## Creazione di una history su Galaxy
Concentriamoci sul **pannello Cronologia** (situato alla destra dellosservatore). Questo consente la visualizzazione di tutti i dataset ottenuti mediante l’uso degli strumenti e di 
tutte le operazioni eseguite sui dati. Per ogni dataset, oltre allo stato (in attesa, in esecuzione, riuscito, fallito), la cronologia tiene traccia di una serie di informazioni come
nome, eventuali tag, formato, dimensione, ora di creazione, metadati specifici del tipo di dati, ID strumento, versione, input, parametri, standard output e standard error.

E’ possibile creare **molteplici history**, ciasuna delle quali dovrebbe corrispondere ad **un’analisi diversa**. E' sempre consigliabile assegnare un nome significativo alle history
create.

Nel nostro caso, clicchiamo sul simbolo "**+**" situato in alto a sinistra, nel pannello Cronologia, accanto al termine History. Abbiamo appena creato la nostra prima History, alla 
quale è stato assegnato il nome di default "**Unnamed history**".

![history_Galaxy](https://training.galaxyproject.org/training-material/shared/images/rename_history.png) 

Decidiamo di attribuire il nome "**DNA_SEQ**" alla nostra nuova history. Lo facciamo semplicemente cliccando sul tasto "**edit**", identificabile mediante l'icona di una matita e 
situato nella parte alta del pannello cronologia, accanto al nome di default "**Unnamed history**". Per salvare il nuovo nome bisogna fare clic sul tasto "**Save**".

![history_Galaxy](https://training.galaxyproject.org/training-material/shared/images/rename_history_old.png)



# Bioinformatic pipelines
Quando si lavora con dati di sequenziamento high-throughput, le raw read ottenute dal sequenziatore devono passare attraverso una serie di strumenti diversi per generare 
l'output finale desiderato. L'esecuzione di questo insieme di strumenti in un ordine specifico viene comunemente workflow o pipeline.

Di seguito viene fornito un esempio del flusso di lavoro che utilizzeremo per la nostra analisi di chiamata di varianti, con una breve descrizione di ciascuna fase.
![pipeline](https://www.maverixbio.com/wp-content/uploads/2014/11/DNASEQ_WORKFLOW_IMAGE.jpg)

## Iniziare con i dati
Spesso il primo passo in un flusso di lavoro bioinformatico è quello di scaricare i dati su un computer dove poter lavorare. Se avete esternalizzato il sequenziamento dei vostri dati,
il centro di sequenziamento di solito vi fornirà un link da utilizzare per scaricare i vostri dati. Oggi lavoreremo con i dati di sequenziamento disponibili pubblicamente.

I dati sono paired-end, quindi scaricheremo due file per ogni campione. Per ottenere i dati utilizzeremo l'[European Nucleotide Archive](https://www.ebi.ac.uk/ena). 
L'ENA "fornisce un registro completo delle informazioni sul sequenziamento dei nucleotidi a livello mondiale, che comprende dati di sequenziamento grezzi, informazioni 
sull'assemblaggio delle sequenze e annotazioni funzionali". L'ENA fornisce anche dati di sequenziamento in formato fastq.


## Upload dei dati in Galaxy

In Galaxy i dati possono essere caricati tramite un semplice copia/incolla del testo, effettuando l’upload dei file conservati in locale sul pc o tramite un URL internet. 
E’ anche possibile caricare i dati da da DB online, tremite FTP o importare dati condivisi.

Gli strumenti accettano det di dati di input nel formato appropriato e, generalmente quando vengono caricati dei dati, Galaxy prova a rilevarne automaticamente il formato, 
che comunque può essere corretto o modificato in un secondo momento, tramite le voci “**Modifica attiributi e tipi di dati**” e “**Modifica attributi e converti**”.
I set di dati prodotti in output da uno strumento possiedono il formato assegnato dallo stumento stesso.

> <hands-on-title> Data Upload: campioni </hands-on-title>
>
> 1. Create a new history for this tutorial
> 2. Click on the upload button in the upper left of the interface o Per farlo clicchiamo sul tasto "**Upload Data**", nella parte alta del **pannello Strumenti** 
     (situato alla sinistra dell'osservatore).
> 3. Import the files from [Zenodo]({{ page.zenodo_link }}):
     In questo tutorial importeremo i dati dei campioni, sottoforma di collezione, utilizzando la **funzionalità copia/incolla** dei seguenti link:
>    
>    ```
>    https://zenodo.org/api/records/10519826/files/SRR2584866_1.fastq.gz/content
>    https://zenodo.org/api/records/10519826/files/SRR2584863_2.fastq.gz/content
>    https://zenodo.org/api/records/10519826/files/SRR2589044_1.fastq.gz/content
>    https://zenodo.org/api/records/10519826/files/SRR2584863_1.fastq.gz/content
>    https://zenodo.org/api/records/10519826/files/SRR2584866_2.fastq.gz/content
>    https://zenodo.org/api/records/10519826/files/SRR2589044_2.fastq/content
>    ```
> 5. A questo punto nella finestra che appare, clicchiamo su "**Collection**", selezioniamo la voce "**Paste/Fetch data**" e incolliamo i link riportati di sopra. 
	 Non dimentichiamoci di selezionare la voce "**List of Pairs**" dal menù a tendina "**Collection Type**".	 
> 6. Non ci rimane che cliccare su "**Start**". Una volta caricati i dati, la finestra assumerà una colorazione verde e sarà il momento di cliccare sul tasto "**Build**".
     Apparirà una nuova finestra denominata "**Create a collection of paired datasets**", attraverso la quale è possibile settare alcune caratteristiche della collezione di dati 
	   che abbiamo caricato. Nel nostro caso ci limitiamo ad inserire soltanto il nome che vogliamo attribuire alla nostra collezione, ovvero "**E.coli collection**" e clicchiamo 
	   "**Create collection**" per terminare questa fase. 
> 7. Add to each database a tag corresponding to ...
>
>    {% snippet faqs/galaxy/datasets_add_tag.md %}
>
{: .hands_on}

La nostra analisi necessità anche del **genoma di riferimento**. Quindi dobbiamo caricarlo su Galaxy eseguendo un processo simile a quello appena descritto.

> <hands-on-title> Data Upload: genoma di riferimento </hands-on-title>
>
> 2. Click on the upload button in the upper left of the interface o Per farlo clicchiamo sul tasto "**Upload Data**", nella parte alta del **pannello Strumenti** 
     (situato alla sinistra dell'osservatore).
> 3. Import the files from [Zenodo]({{ page.zenodo_link }}):
     In questo tutorial importeremo i dati del genoma di riferimento, utilizzando la **funzionalità copia/incolla** dei seguenti link:
>    
>    ```
>    https://zenodo.org/api/records/10519826/files/SRR2584866_1.fastq.gz/content
>    https://zenodo.org/api/records/10519826/files/SRR2584863_2.fastq.gz/content
>    https://zenodo.org/api/records/10519826/files/SRR2589044_1.fastq.gz/content
>    https://zenodo.org/api/records/10519826/files/SRR2584863_1.fastq.gz/content
>    https://zenodo.org/api/records/10519826/files/SRR2584866_2.fastq.gz/content
>    https://zenodo.org/api/records/10519826/files/SRR2589044_2.fastq/content
>    ```
> 5. Nella finestra che appare, lasciamo "**Regular**", selezioniamo la voce "**Paste/Fetch data**" e incolliamo il link riportato di sopra. 
     Possiamo anche attrbuire un nome diverso al file mediante l'apposita voce "**name**", ma in questa fase lasciamo quello assegnatogli di default da Galxy, ovvero 
     "**ecoli_rel606.fasta.gz**".	 
> 7. Non ci rimane che cliccare su "**Start**". Una volta caricati i dati, la finestra assumerà una colorazione verde e sarà il momento di cliccare sul tasto "**Close**". 
> 8. Add to each database a tag corresponding to ...
>
>    {% snippet faqs/galaxy/datasets_add_tag.md %}
>
{: .hands_on}

## Un primo sguardo ai dati

Visualizziamo alcune righe di uno dei file fastq appena caricati, cliccando sulla collection appena caricata su Galaxy, evidenziata in verde, e disponibile all'interno dell pannello cronologia. 

Ricordiamoci che la nostra collection è composta da **dati paired-end provenienti da tre campioni**, due file per ogni campione, per un totale di sei file. Quindi bisogna selezionare il campione desiderato, ad esempio "**SRR2584863.fastq**", e poi il tasto "**Display**" (l'incona dell'occhio), presente in corrispondenza di uno dei due campioni disponibili,  "**forward**" o "**reverse**".

![file](https://training.galaxyproject.org/training-material/topics/introduction/images/eye-icon.png)

Un file fastq è un file di testo suddiviso in blocchi di 4 righe. Ogni blocco corrisponde ad una read sequenziata. La prima riga è una intestazione ed inizia sempre con il simbolo `@` seguito da un identificativo univoco per la read e da altre informazioni specifiche dell'apparato di sequenziamento.
La seconda riga contiene la sequenza nucleotidica. La terza riga è un separatore che contiene sempre il testo `+`. La quarta riga mostra la qualità di ciascun nucleotide della read.

```
<div>@SRR2584863.1 HWI-ST957:244:H73TDADXX:1:1101:4712:2181/1
TTCACATCCTGACCATTCAGTTGAGCAAAATAGTTCTTCAGTGCCTGTTTAACCGAGTCACGCAGGGGTTTTTGGGTTACCTGATCCTGAGAGTTAACGGTAGAAACGGTCAGTACGTCAGAATTTACGCGTTGTTCGAACATAGTTCTG
+
CCCFFFFFGHHHHJIJJJJIJJJIIJJJJIIIJJGFIIIJEDDFEGGJIFHHJIJJDECCGGEGIIJFHFFFACD:BBBDDACCCCAA@@CA@C>C3>@5(8&>C:9?8+89<4(:83825C(:A#########################
</div>
```

La qualità è interpretata come la probabilità di una chiamata di base errata (ad esempio, 1 su 10) o, equivalentemente, l'accuratezza della chiamata di base (ad esempio, 90%). Per poter associare ogni singolo nucleotide al suo punteggio di qualità, il punteggio numerico viene convertito in un codice in cui ogni singolo carattere rappresenta il punteggio numerico di qualità per un singolo nucleotide.

Il valore numerico assegnato a ciascuno di questi caratteri dipende dalla piattaforma di sequenziamento che ha generato le read. La macchina di sequenziamento utilizzata per generare i nostri dati utilizza la codifica del punteggio di qualità Sanger PHRED standard, utilizzando Illumina dalla versione 1.8 in poi. A ogni carattere viene assegnato un punteggio di qualità compreso tra 0 e 41.

```
<div>Quality encoding: !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJ
                   |         |         |         |         |
Quality score:    01........11........21........31........41
</div>
```

Ogni punteggio di qualità rappresenta la probabilità che la chiamata nucleotidica corrispondente sia errata. Il punteggio di qualità è logaritmico, quindi un punteggio di qualità pari a 10 riflette un'accuratezza di chiamata di base del 90%, mentre un punteggio di qualità pari a 20 riflette un'accuratezza di chiamata di base del 99%. Questi valori di probabilità sono i risultati dell'algoritmo di chiamata di base e dipendono dalla quantità di segnale catturato per l'incorporazione della base.

## Valutazione della qualità con FastQC

Nella vita reale, non si valuterà la qualità delle letture ispezionando visivamente i file FASTQ. Piuttosto, si utilizzerà un programma software per valutare la qualità delle letture e filtrare quelle di scarsa qualità. Utilizzeremo un tool chiamato **`FastQC`** per visualizzare la qualità delle letture. 

**FastQC** ha una serie di funzioni che permettono di comprendre rapidamente la presenza di eventuali problemi dei dati, in modo da poterli prendere in considerazione prima di procedere con le analisi. Invece di esaminare i punteggi di qualità per ogni singola lettura, FastQC esamina la qualità collettivamente per tutte le letture di un campione.

![fastqc](https://cyverse-fastqc-quickstart.readthedocs-hosted.com/en/latest/_images/perbase_good.png)

> <hands-on-title> Quality control </hands-on-title>
> 1. **FastQC** {% icon tool %} ("*FastQC Read Quality reports*") with the following parameters:
>    - *"Raw read data from your current history:"*: `E.coli collection`
     (nel nostro caso, trattandosi di una collazione di dati, va selezionata l’icona rappresentante una piccola cartellina, che sta per "Dataset collection").
> {: .hands_on}

Modifichiamo subito i nomi dei due output ottenuti, cosi da rendere più facile la futura comprensione dell'intera pipeline, cliccando sull'icone della matita situata in corrispondenza di ciascun output. Cambiamo il nome dell'output "**FastQC on collection 7: Webpage**" in "**FastQC_report (webpage)**" e quello dell'output "**FastQC on collection 7: RawData**" in "**FastQC_report (rawdata)**".

Il tool con i parametri indicati produrrà due output, per ogni file fastq, un file contenente i dati grezzi ottenuti e un file html che contiene un report grafico sulla qualità del campione. Abbiamo anche modificato subito i nomi dei due output ottenuti, cosi da rendere più facile la futura comprensione dell'intera pipeline, cliccando sull'icone della matita situata in corrispondenza di ciascun output.

Precedentemente abbiamo esaminato un esempio di grafico FastQC sulla "qualità della sequenza per base", ma ci sono altri nove grafici di cui non abbiamo parlato. Di seguito una breve panoramica delle interpretazioni di ciascuno di questi grafici. Per ulteriori informazioni, consultare la documentazione di FastQC.

* **Qualità della sequenza per tile**: le macchine che eseguono il sequenziamento sono divise in tile. Questo grafico mostra gli schemi della qualità delle basi lungo queste celle. Punteggi costantemente bassi si trovano spesso intorno ai bordi, ma si possono verificare punti caldi anche al centro se è stata introdotta una bolla d'aria in qualche momento della run.
* **Punteggi di qualità per sequenza**: un grafico di densità della qualità per tutte le read in tutte le posizioni. Questo grafico mostra quali punteggi di qualità sono più comuni.
* **Contenuto della sequenza per base**: traccia la proporzione di ogni posizione di base su tutte le letture. In genere, ci aspettiamo di vedere ogni base circa il 25% delle volte in ogni posizione, ma questo spesso non avviene all'inizio o alla fine della lettura a causa della qualità o del contenuto dell'adattatore.
* **Contenuto di GC per sequenza**: un grafico di densità del contenuto medio di GC in ciascuna lettura.
* **Contenuto di N per base**: la percentuale di volte in cui "N" si presenta in una posizione in tutte le letture. Se c'è un aumento in una particolare posizione, ciò potrebbe indicare che qualcosa è andato storto durante il sequenziamento.
* **Distribuzione della lunghezza di sequenza**: la distribuzione delle lunghezze di sequenza di tutte le letture nel file. Se i dati sono grezzi, spesso c'è un picco netto, ma se le letture sono state tagliate, può esserci una distribuzione di lunghezze inferiori.
* **Livelli di duplicazione delle sequenze**: Una distribuzione di sequenze duplicate. Nel sequenziamento, ci aspettiamo che la maggior parte delle letture si verifichi una sola volta. Se alcune sequenze si ripetono più di una volta, ciò potrebbe indicare un bias di arricchimento (ad esempio, dalla PCR). Se i campioni sono ad alta copertura (o RNA-seq o ampliconi), questo potrebbe non essere vero.
* **Sequenze sovrarappresentate**: Un elenco di sequenze che si presentano più frequentemente di quanto ci si aspetterebbe per caso.
* **Contenuto di adattatori**: un grafico che indica dove si trovano le sequenze di adattatori nelle read.
* **Contenuto di K-mer**: un grafico che mostra tutte le sequenze che possono mostrare un bias posizionale all'interno delle letture.

Abbiamo appena controllato la qualità di ogni campione. In questo esempio abbiamo solo 3 campioni e 6 file fastq, quindi, questo processo non richiederebbe un grande sforzo. Tuttavia, con molti campioni ci viene in aiuto un altro tool chiamato **`multiqc`** che, a partire da report di qualità, sviluppa una sintesi in un singolo report. Non lo vedremo in questo tutorial, ma tale tool genererà un report in HTML che potremmo visualizzare e che presenta gli stessi grafici di fastqc in una forma aggregata.

## Pulizia delle read

Abbiamo dato un'occhiata di alto livello alla qualità di ciascuno dei nostri campioni utilizzando **FastQC**. Abbiamo visualizzato i grafici della qualità per base che mostrano la distribuzione della qualità della read in ogni base su tutte le letture di un campione e abbiamo estratto informazioni su quali campioni non superano i controlli di qualità. Alcuni dei nostri campioni non hanno superato diverse metriche di qualità utilizzate da FastQC. Ciò non significa, tuttavia, che i nostri campioni debbano essere scartati. È molto comune che alcune metriche di qualità falliscano e questo può essere o meno un problema per la vostra applicazione a valle. Per la nostra pipeline di chiamata di varianti, elimineremo alcune sequenze di bassa qualità per ridurre il tasso di falsi positivi dovuti a errori di sequenziamento.

Utilizzeremo un tool chiamato **Trimmomatic** per filtrare le letture di scarsa qualità e tagliare le basi di scarsa qualità dai nostri campioni.

> <hands-on-title> Trimming </hands-on-title>
> 1. **Trimmomatic** {% icon tool %} ("*Trimmomatic flexible read trimming tool for Illumina NGS data*") with the following parameters:
>    - *"Single-end or paired-end reads?"*: `Paired-end (as collection)`
>      - *"Select FASTQ dataset collection with R1/R2 pair"*: `E.coli collection` (il dataset che abbiamo caricato precedentemente)
>    - In "*Trimmomatic Operation*":
>      - *"Select Trimmomatic operation to perform"*: `Sliding window trimming (SLIDINGWINDOW)`
>      - *"Number of bases to average across*"*: `4`
>      - *"Average quality required*"*: `20`
>
>    - Click the "**Run Tool**" button 
> {: .hands_on}

Al termine del processo, se l'input consiste in una raccolta di set di dati con la coppia FASTQ R1/R2, gli output consisteranno in due raccolte di set di dati: una per gli output "**accoppiati o paired**" e uno per quelli "**non abbinati o unpaired**".

Modifichiamo subito i nomi dei due output ottenuti, cosi da rendere più facile la futura comprensione dell’intera pipeline, cliccando sull’icona della matita situata in corrispondenza di ciascun output.

Cambiamo il nome dell’output "**Trimmomatic on collection 7: paired**" in "**Trim.fastq.out_paired**" e quello dell’output "**Trimmomatic on collection 7: unpaired**" in "**Trim.fastq.out_unpaired**".

**Trimmomatic** è uno dei tool usati per la pulizia delle reads usato soprattutto quando i dati di sequenziamento provengono da macchine Illumina. Un'altro strumento molto usato, al posto di trimmomatic, per la pulizia delle reads è **`cutadapt`**. Cutadapt è uno strumento molto generale e può essere ampiamente personalizzato. Tuttavia, questa personalizzabilità può rendere complesso l'uso del tool, per questo motivo sono nati diversi wrapper per semplificarne l'utilizzo. Per il nostro caso si potrebbe anche utilizzare ul tool **`Trim Galore`**, un wrapper costruito attorno a Cutadapt e FastQC per applicare in modo coerente il processo di quality e adapter trimming, automatizzando molti passaggi intermedi.


## Pipeline per la chiamata delle varianti

Ora che abbiamo esaminato i nostri dati per assicurarci che siano di alta qualità e abbiamo rimosso le chiamate di base di bassa qualità, possiamo eseguire la chiamata di varianti per vedere come è cambiata la popolazione nel tempo. Ci interessa sapere come è cambiata questa popolazione rispetto alla popolazione originale, E. coli ceppo REL606. Pertanto, allineeremo ciascuno dei nostri campioni al **genoma di riferimento di E. coli REL606** e vedremo quali differenze esistono nelle nostre letture rispetto al genoma.
