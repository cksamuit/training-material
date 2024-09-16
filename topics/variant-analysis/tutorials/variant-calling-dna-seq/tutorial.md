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
Visitiamo la homepage di Galaxy ed effettuaimo il **login o la registrazione** per accedere al nostro account, cliccando sulla voce "**Login or register**", situato in alto nel pannello centrale.  

![Galaxy](https://training.galaxyproject.org/training-material/topics/introduction/images/galaxy-login.png)

## Creazione di una history su Galaxy
Concentriamoci sul **pannello Cronologia** (situato alla destra dellosservatore). Questo consente la visualizzazione di tutti i dataset ottenuti mediante l’uso degli strumenti e di 
tutte le operazioni eseguite sui dati. Per ogni dataset, oltre allo stato (in attesa, in esecuzione, riuscito, fallito), la cronologia tiene traccia di una serie di informazioni come
nome, eventuali tag, formato, dimensione, ora di creazione, metadati specifici del tipo di dati, ID strumento, versione, input, parametri, standard output e standard error.

E’ possibile creare **molteplici history**, ciasuna delle quali dovrebbe corrispondere ad **un’analisi diversa**. E' sempre consigliabile assegnare un nome significativo alle history
create.

{% snippet faqs/galaxy/histories_create_new.md box_type="hands_on" %}

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
> 2. Inspect the web page output of FastQC {% icon tool %} for the **multiple datasets**. 
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

## Allineamento rispetto ad un genoma di riferimento

Eseguiamo l'allineamento o la mappatura delle reads per determinare la loro origine nel genoma. Esistono numerosi strumenti tra cui scegliere e, sebbene non esista un gold standard, ve ne sono alcuni più adatti a particolari analisi NGS. Noi utilizzeremo il Burrows Wheeler Aligner (BWA), utile per la mappatura di sequenze a bassa divergenza rispetto a un grande genoma di riferimento.

In particolare, utilizzeremo il tool **BWA-MEM2**, ricercandolo nella barra di ricerca del pannello Strumenti di Galaxy. Si tratta di una versione più performante rispetto al classico **BWA-MEM**. 

A questo punto, dovremmo già aver caricato il genoma di riferimento su Galaxy, che ricordiamo essere **ecoli_rel606.fasta.gz**. 

Il processo di allineamento consiste in due fasi:

1. indicizzazione del genoma di riferimento (da fare solo la prima volta)
2. allineamento delle letture al genoma di riferimento

Utilizzando il tool **BWA-MEM2**, non avremo bisogno di indicizzare ne di convertire in BAM tramite l'utilizzo di strumenti come SAMTOOLS. **BWA-MEM2** infatti, indicizza già di suo e offre l'output in formato BAM, senza obbligare alla conversione SAM-BAM.

> <hands-on-title> Mapping </hands-on-title>
> 1. **BWA-MEM2** {% icon tool %} ("*BWA-MEM2 - map medium e long reads (>100 bp) againts reference genome*") with the following parameters:
>    - *"Will you select a reference genome from your history or use a built-in index?"*: `Use a genome from history and build index`
>      - *"Use the following dataset as the reference sequence*"*: `ecoli_rel606.fasta.gz (as fasta)` (il genoma di riferimento caricato precedentemente)
>    - In "*Single or Paired-end reads*":
>      - *"Select between paired and single end data"*: `Paired Collection`
>      - *"Select a paired collection*"*: `Trim.fastq.out_paired` (l'output ottenuto dall'esecuzione di Trimmomatic)
>    - In "*tipo di analisi*":
>      - *"Select analysis mode"*: `1. Simple Illumina mode`
>      - *"BAM sorting mode*"*: `Sort by chromosomal coordinates`
>    - Click the "**Run Tool**" button 
> {: .hands_on}

Al termine del processo, otterremo un output da rinominare, per rendere più facile la futura comprensione dell’intera pipeline. Cliccando sull’icona della matita situata in corrispondenza delll'output "**BWA-MEM2 on collection 24 (mapped reads in BAM format)**" e cambiamo il nome in "**Alignement.output_bam**".

### **Formato SAM/BAM**

Il file SAM è un file di testo tabulato che contiene informazioni su ogni singola read e sul suo allineamento al genoma. Non abbiamo il tempo di entrare nel dettaglio delle caratteristiche del formato SAM, ma l'articolo di [Heng Li et al.](http://bioinformatics.oxfordjournals.org/content/25/16/2078.full) fornisce molti altri dettagli sulle specifiche.

La versione binaria compressa di SAM è chiamata file BAM. Utilizziamo questa versione per ridurre le dimensioni e per consentire l'indicizzazione, che permette un accesso casuale efficiente ai dati contenuti nel file.

Il file inizia con un header opzionale. L'intestazione è utilizzata per descrivere la fonte dei dati, la sequenza di riferimento, il metodo di allineamento, ecc. Dopo l'intestazione si trova la sezione dedicata all'allineamento. Ogni riga che segue corrisponde alle informazioni di allineamento per una singola read. Ogni riga di allineamento presenta 11 campi obbligatori per le informazioni di mappatura essenziali e un numero variabile di altri campi per le informazioni specifiche dell'allineatore.

### **Eliminazione delle sequenze duplicate**

Dopo aver indicizzato il file, possiamo **identificare le sequenze duplicate**. Le sequenze duplicate sono quelle che provengono dalla stessa molecola dopo l'estrazione e il taglio del DNA genomico. Ne esistono due tipi: **duplicati ottici** e **duplicati della reazione a catena della polimerasi (PCR)**. I duplicati ottici sono un errore introdotto dal sequenziatore. I duplicati PCR sono introdotti dai protocolli di preparazione delle librerie che utilizzano la PCR. 

I duplicati possono causare errori nella statistica usata per chiamare le varianti. Per questo motivo è necessario escludere le sequenze duplicate dalla chiamata di varianti. Queste possono essere identificate più facilmente dai dati paired-end come quelle sequenze per le quali entrambe le reads hanno siti di inizio identici. Ciò può eliminare alcune sequenze che in realtà derivano da frammenti unici nella libreria originale, ma se la frammentazione è effettivamente casuale, i frammenti identici dovrebbero essere rari. Una volta identificate, le sequenze duplicate possono essere contrassegnate e ignorate durante la chiamata di varianti (o altri tipi di analisi) a valle.

Per questa operazione utilizziamo il tool **MarkDuplicates**, che ci permette di identificare ed eliminare i duplicati.

> <hands-on-title> Rimozione duplicati </hands-on-title>
> 1. **MarkDuplicates** {% icon tool %} ("*MarkDuplicates examine aligned records in BAM dataset to locate duplicate molecules*") with the following parameters:
>    - *"Select SAM/BAM dataset or dataset collection*"*: `Alignment.output_bam`(l’output ottenuto dall’esecuzione di BWA-MEM2)
>    - *"REMOVE_DUPLICATES"*: `yes`
>    - *"Assume the input file is already sorted*"*: `yes`
>    - *"The scoring strategy for choosing the non-duplicate among candidates*"*: `SUM_OF_BASE_QUALITIES`
>    - *"Select validation stringency*"*: `Lenient`
>    - Click the "**Run Tool**" button 
> {: .hands_on}

Al termine del processo, otterremo l'output nei formati **tabulare** e **BAM**.

Rinominiamoli, cosi da rendere più facile la futura comprensione dell’intera pipeline, cliccando sull’icona della matita situata in corrispondenza di ciascun output.Cambiamo il nome dell’output "**MarkDuplicates on collection 38: tabular**" in "**No.duplicate_tabular**" e quello dell’output "**MarkDuplicates on collection 38: BAM**" in "**No.duplicate_bam**".

Prima di procedere alla fase successiva, vediamo le statistiche dell'allineamento di un campione per capire se ci sono stati problemi. Il tool **samtools flagstat** ci aiuta in questo compito. 

> <hands-on-title> samtools </hands-on-title>
> 1. **Samtools** {% icon tool %} ("*Samtools flagstat tabulate descriptive stats for BAM datset*") with the following parameters:
>    - *"BAM File to report statistics of*"*: `Markduplicates_bam` (l'output in formato BAM ottenuto dall'esecuzione di MarkDuplicates)
>    - *"Output format*"*: `txt`
>    - Click the "**Run Tool**" button 
> {: .hands_on}

Al termine del processo, otterremo un output in formato testuale che possiamo rinominare in  "**Statistics.aligment_txt**".

Proviamo a visualizzare il risultato ottenito sul campione SRR2584866:

```
5301929 + 0 in total (QC-passed reads + QC-failed reads)
5285672 + 0 primary
0 + 0 secondary
16257 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
5235971 + 0 mapped (98.76% : N/A)
5219714 + 0 primary mapped (98.75% : N/A)
5285672 + 0 paired in sequencing
2663237 + 0 read1
2622435 + 0 read2
5162084 + 0 properly paired (97.66% : N/A)
5205220 + 0 with itself and mate mapped
14494 + 0 singletons (0.27% : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```

## Variant calling

La chiamata delle varianti è il processo che stabilisce se sono presenti differenze nucleotidiche rispetto a un riferimento in una determinata posizione in un genoma individuale o in un trascrittoma, spesso indicata come una variante a singolo nucleotide (SNV). La chiamata è solitamente accompagnata da una stima della frequenza della variante e da una misura di confidenza. Come per le altre fasi di questo flusso di lavoro, sono disponibili diversi strumenti per la chiamata di varianti.

In questo tutorial utilizzeremo **bcftools**, il più semplice, ma anche potente, strumento per la chiamata delle varianti. **Bcftools** è utilizzato per trovare tutte le possibili mutazioni in un campione. Ma le varianti in un campione possono essere germinali, se avvengono nelle cellule da cui ha origine una nuova vita (cellule germinali), o somatiche, se avvengono in qualunque altra cellula del corpo (cellule somatiche). Cosa possiamo fare allora per identificare solo le mutazioni somatiche? Ad esempio, abbiamo un malato di tumore e vogliamo capire quali mutazioni possono aver innescato la patologia. Queste mutazioni potrebbero essere somatiche? Si potrebbe ad esempio, sfruttare un tool più potente chiamato GATK per vedere come si fa la chiamata delle varianti somatiche su un campione umano.

In entrambi i modelli viene applicato un modello statistico per valutare la presenza di una variante. Ma perchè applicare un modello statistico e non andare semplicemente a vedere le differenze rispetto al genoma di riferimento?

In breve, ciò avviene perché i dati di sequenziamento che osserviamo sono stati generati attraverso diversi processi di laboratorio e di calcolo soggetti a errori.

Possiamo suddividere questo processo in diverse fasi:

* **Campionamento del genoma**: Un individuo diploide ha due copie di ogni cromosoma. Se questo individuo è eterozigote per un determinato sito, la probabilità di osservare solo l'allele di riferimento è 0,5^n, dove n è il numero di sequenze (non duplicate). Supponendo quindi di avere una copertura 5x di un particolare sito, avremmo una probabilità del 3,125% di non osservare l'allele alternativo.
* **Preparazione della libreria**: Durante la preparazione della libreria, le polimerasi possono incorporare erroneamente le basi. Queste basi saranno lette dal sequenziatore e probabilmente gli verrà assegnato un punteggio di qualità elevato. Se si usa la PCR durante la preparazione della libreria, si possono introdurre cambiamenti casuali nell'equilibrio allelico nei siti eterozigoti, oppure si possono amplificare i bias casuali esistenti nel pool di DNA estratto, aggravando il problema della fase precedente.
* **Sequenziamento**: Quando carichiamo la nostra libreria di frammenti di DNA sul sequenziatore per analizzarla, la macchina può leggere le basi in modo errato.
* **Mappatura di riferimento**: Dopo aver sequenziato la nostra libreria, dobbiamo mappare le sequenze al genoma di riferimento. La maggior parte dei genomi di riferimento sono a) incompleti e b) hanno molte copie di sequenze simili o identiche. Questi problemi possono far sì che un frammento di DNA mappato sul genoma di riferimento possa non derivare effettivamente da quella regione. Le differenze tra questo frammento e quello di riferimento non rappresentano quindi il tipo di polimorfismo che stiamo cercando.
* **Allineamento delle sequenze**: Anche se abbiamo assegnato un frammento di DNA alla posizione corretta nel genoma, dobbiamo allinearlo correttamente. In presenza di indels, questo può diventare molto difficile e possono esserci diversi allineamenti ugualmente probabili.

Tutto questo per dire che quando osserviamo un insieme di sequenze allineate al genoma di riferimento e sembrano esserci delle basi che differiscono, abbiamo bisogno di un modello statistico in grado di tenere conto di queste complesse fonti di errore per aiutarci a decidere se la variazione è reale e assegnare i genotipi agli individui.

### **Bcftools**

La pipeline per la chiamata delle varianti con **`bcftools`** è molto semplice e si compone di due fasi. Innanzitutto genera un file di _pileup_ utilizzando **`bcftools mpileup`**. Il file _pileup_ riassume la coverage per base in ogni sito del genoma. Ogni riga rappresenta una posizione genomica e ogni posizione con copertura di sequenziamento è presente nel file. La seconda fase, utilizzando `bcftools call`, applica il modello statistico per valutare le prove di variazione rappresentate nel riepilogo e genera un output nel formato di chiamata di variante (VCF).

Poiché i file pileup per il sequenziamento dell'intero genoma contengono un riassunto per ogni sito del genoma per ogni campione, possono essere molto grandi. Inoltre, in genere non abbiamo bisogno di guardarli direttamente o di fare qualcosa con essi al di fuori della chiamata di varianti. Pertanto, se si utilizza questo approccio per i propri dati, di solito è sufficiente inviare l'output di `bcftools mpileup` direttamente a `bcftools call`. Applicheremo la pipeline a tutti e tre i campioni ovviamente, sfruttando la collection che abbiamo creato inizialmente.

> <hands-on-title> Mapping </hands-on-title>
> 1. **bcftools mpileup** {% icon tool %} ("*bcftools mpileup Generate VCF or BCF containing genotype likelihoods for one or multiple alignment (BAM or CRAM) files*") with the following parameters:
>     - In "*Input dataset*":
>      - *"Alignment Inputs"*: `Single BAM/CRAM`
>      - *"Input BAM/CRAM*"*: `No.duplicates_bam` (l'output senza duplicati, ottenuto dallo strumento MarkDuplicates)
>    - In "*Reference Genome*":
>      - *"Choose the source for the reference genome"*: `History`
>      - *"Genome Reference*"*: `ecoli_rel606.fasta.gz (as fasta)`
>    - In "*Indel calling*":
>      - *"Perform INDEL calling"*: `1. Simple Illumina mode`
>      - *"Genome Reference*"*: `ecoli_rel606.fasta.gz (as fasta)`
>    - In "*Input Filtering Options*":
>      - *"Max reads for BAM"*: `250`
>      - *"Quality Options"*: `Set base and mapping quality options`
>      - *"Coefficient for downgrading mapping quality for reads containing excessive mismatches*"*: `0`
>      - *"Minimum mapping quality for an alignment to be used*"*: `20` (stiamo indicando allo strumento di escludere quelle reads che hanno una qualità di mapping minore di 20)
>      - *"Minimum base quality for a base to be considered"*: `30` (stiamo indicando allo strumento di escludere tutte quelle basi con una qualità minore di 30 nella read)
>    - In "*Output Options*":
>      - *"Output_type*"*: `Compressed BCF` (indichiamo a bcftools di generare un file di output in formato BCF, variante binaria e compressa del formato VCF, di cui parleremo a breve)
>    - Click the "**Run Tool**" button 
> {: .hands_on}

Al termine del processo, otterremo un output che, per comodità rinominiamo in "**Mpileup.out_bcf**".

Per la chiamata delle varianti usiamo `bcftools call` che applica un modello statistico per valutare se la variazione nella sequenza che stiamo osservando è una vera variazione biologica o un errore prodotto a seguito delle fasi di preparazione della libreria, sequenziamento, mapping o allineamento. 

> <hands-on-title> Mapping </hands-on-title>
> 1. **bcftools call** {% icon tool %} ("*bcftools call SNP/indel variant calling from VCF/BCF*") with the following parameters:
>    - *"VCF/BCF Data*"*: `Mpileup.out_bcf` (l'output ottenuto dall'esecuzione di bcftools mpileup)
>    - In "*Consensus/variant calling Options*":
>      - *"Calling method"*: `Multiallelic and rare-variant caller`
>    - In "*File format options*":
>      - *"Select predefined ploidy"*: `Treat all samples ad haploid`
>    - In "*Input/output Options*":
>      - *"Output variant sites only"*: `yes`
>      - *"Output_type"*: `Uncompressed VCF`
>    - Click the "**Run Tool**" button 
> {: .hands_on}

 Rinomineremo l'output ottenuto, per comodità, in "**Final.outraw_vcf**.

 L'output di **bcftools** dovrà essere filtrato con **`CVFfilter`** per ottenere l'elenco finale delle varianti:

> <hands-on-title> Mapping </hands-on-title>
> 1. **VCFfilter** {% icon tool %} ("*VCFfilter: filter VCF data in a variety of attributes*") with the following parameters:
>    - *"VCF dataset to filter*"*: `Final.outraw_vcf`
>    - In "*More filters*":
>      - *"Select the filter type*"*: `Info filter(-f)`
>      - *"Specify filterting value"*: `DP>10`
>    - Click the "**Run Tool**" button 
> {: .hands_on}

Rinominiamo l'output ottenuto, per comodità, in "**Final_vcf**.

### **Esploriamo il file VFC**

Un file VCF sintetizza il risultato della chiamata delle varianti. Il file è diviso in due sezioni. Una sezione dei metadati, posta all'inizio del file, e una sezione delle varianti. Possiamo distinguere la sezione dei metadati perchè ogni riga inizia con `##`.

Vediamo, quindi, cosa contengono i metadati direttamente del campione **SRR2584866**:

```
<div><span class="hljs-meta">$</span><span class="bash"> less -S SRR2584866_final.vcf</span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#fileformat=VCFv4.2</span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#FILTER=<ID=PASS,Description="All filters passed"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#bcftoolsVersion=1.9+htslib-1.9</span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#bcftoolsCommand=mpileup --threads 25 -O b -q 20 -Q 30 -f ecoli/genome/ecoli_rel606.fasta ecoli/bam/SRR2584866.no_dups.bam</span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#reference=file://ecoli/genome/ecoli_rel606.fasta</span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#contig=<ID=NC_012967.1,length=4629812></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#ALT=<ID=*,Description="Represents allele(s) other than observed."></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=INDEL,Number=0,Type=Flag,Description="Indicates that the variant is an INDEL."></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=IDV,Number=1,Type=Integer,Description="Maximum number of reads supporting an indel"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=IMF,Number=1,Type=Float,Description="Maximum fraction of reads supporting an indel"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=DP,Number=1,Type=Integer,Description="Raw read depth"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=VDB,Number=1,Type=Float,Description="Variant Distance Bias for filtering splice-site artefacts in RNA-seq data (bigger is better)",Version="3"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=RPB,Number=1,Type=Float,Description="Mann-Whitney U test of Read Position Bias (bigger is better)"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=MQB,Number=1,Type=Float,Description="Mann-Whitney U test of Mapping Quality Bias (bigger is better)"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=BQB,Number=1,Type=Float,Description="Mann-Whitney U test of Base Quality Bias (bigger is better)"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=MQSB,Number=1,Type=Float,Description="Mann-Whitney U test of Mapping Quality vs Strand Bias (bigger is better)"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=SGB,Number=1,Type=Float,Description="Segregation based metric."></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=MQ0F,Number=1,Type=Float,Description="Fraction of MQ0 reads (smaller is better)"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#FORMAT=<ID=PL,Number=G,Type=Integer,Description="List of Phred-scaled genotype likelihoods"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#FORMAT=<ID=GT,Number=1,Type=String,Description="Genotype"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=ICB,Number=1,Type=Float,Description="Inbreeding Coefficient Binomial test (bigger is better)"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=HOB,Number=1,Type=Float,Description="Bias in the number of HOMs number (smaller is better)"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=AC,Number=A,Type=Integer,Description="Allele count in genotypes for each ALT allele, in the same order as listed"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=AN,Number=1,Type=Integer,Description="Total number of alleles in called genotypes"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=DP4,Number=4,Type=Integer,Description="Number of high-quality ref-forward , ref-reverse, alt-forward and alt-reverse bases"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#INFO=<ID=MQ,Number=1,Type=Integer,Description="Average mapping quality"></span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#bcftools_callVersion=1.9+htslib-1.9</span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment">#bcftools_callCommand=call --threads 25 --ploidy 1 -m -v -o SRR2584866_raw.vcf; Date=Fri May 26 06:44:35 2023</span></span>
</div>
```

Andiamo a vedere ora il consenuto della seconda parte. Questo è formattato come una tabella i cui campi sono separati dal carattere tabulazione.

Di seguito vediamo un esempio partendo dal campione appena analizzato. Il contenuto è riportato sotto forma di tabella.

| CHROM | POS | ID | REF | ALT | QUAL | FILTER | INFO | FORMAT | ecoli/bam/SRR2584866.no_dups.bam |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| NC_012967.1 | 1521 | . | C | T | 225.417 | . | AC=1;AN=1;DP=120;DP4=0,0,30,48;FS=0;MQ=60;MQ0F=0;MQSBZ=0;SGB=-0.693147;VDB=0.980893	 | GT:PL | 1:255,0 |
| NC_012967.1 | 1612 | . | A | G | 225.417 | . | AC=1;AN=1;DP=113;DP4=0,0,43,48;FS=0;MQ=60;MQ0F=0;MQSBZ=0;SGB=-0.693147;VDB=0.0699653 | GT:PL | 1:255,0 |
| NC_012967.1 | 9092 | . | A | G | 228.414 | . | AC=1;AN=1;BQBZ=1.07874;DP=172;DP4=1,0,71,73;FS=0;MQ=60;MQ0F=0;MQBZ=0;MQSBZ=0;RPBZ=1.5538;SCBZ=0.25708;SGB=-0.693147;VDB=0.994801 | GT:PL | 1:255,0 |
| NC_012967.1 | 9972 | . | T | G | 225.417 | . | AC=1;AN=1;DP=151;DP4=0,0,46,90;FS=0;MQ=60;MQ0F=0;MQSBZ=0;SGB=-0.693147;VDB=0.946333 | GT:PL | 1:255,0 |
| NC_012967.1 | 10563 | . | G | A | 225.417 | . | 	AC=1;AN=1;DP=148;DP4=0,0,55,66;FS=0;MQ=60;MQ0F=0;MQSBZ=0;SGB=-0.693147;VDB=0.833823 | 1:255,0 |
| NC_012967.1 | 16518 | . | TCCCCCCC | TCCCCCCCC | 151.416 | . | AC=1;AN=1;DP=18;DP4=0,0,16,0;FS=0;IDV=16;IMF=0.888889;MQ=40;MQ0F=0;MQBZ=0;RPBZ=-2.25597;SCBZ=0;SGB=-0.689466;VDB=0.125884;INDEL | GT:PL | 1:181,0 |
| NC_012967.1 | 22257 | . | C | T | 225.417 | . | AC=1;AN=1;DP=73;DP4=0,0,35,29;FS=0;MQ=60;MQ0F=0;MQSBZ=0;SGB=-0.693147;VDB=0.785042 | GT:PL | 1:255,0 |
| NC_012967.1 | 38971 | . | A | G | 225.417 | . | AC=1;AN=1;DP=185;DP4=0,0,69,87;FS=0;MQ=60;MQ0F=0;MQSBZ=0;SGB=-0.693147;VDB=0.195101 | GT:PL | 1:255,0 |
| NC_012967.1 | 42306 | . | A | G | 225.417 | . | AC=1;AN=1;DP=167;DP4=0,0,50,87;FS=0;MQ=59;MQ0F=0;MQSBZ=-0.758098;SGB=-0.693147;VDB=0.999992 | GT:PL | 1:255,0 |
| NC_012967.1 | 45277 | . | A | G | 225.417 | . | AC=1;AN=1;DP=170;DP4=0,0,75,62;FS=0;MQ=60;MQ0F=0;MQSBZ=0;SGB=-0.693147;VDB=0.0753226 | GT:PL | 1:255,0 |

**Nota:** Il campo GT in questo caso assumerà sempre il valore 1 perchè il genoma è aploide. Nel caso di genomi poliploidi, come il genoma umano, potremmo avere valori diversi. Ad esempio nel genoma umano, i valori possibili saranno:

* **0/0**: nessuno dei due alleli presenta mutazione
* **0/1**: solo uno dei due alleli presenta mutazione (l'altro allele ha la base REF)
* **1/1**: entrambi gli alleli sono mutati

Tipicamente in un file VCF filtrato non dovremmo mai trovare il valore **0/0**, perchè indica assenza di mutazione.

Una descrizione più dettagliata del formato VCF può essere reperita a questo indirizzo https://samtools.github.io/hts-specs/VCFv4.2.pdf.

Abbiamo terminato la chiamata delle varianti e possiamo ora esaminare i risultati tramite **IGV**.

### **Valutazione dell'allineamento (visualizzazione) - opzionale**

Spesso è istruttivo guardare i dati in un browser del genoma. La visualizzazione consente di farsi un'idea dei dati e di individuare anomalie e problemi. Inoltre, l'esplorazione dei dati in questo modo può fornire idee per ulteriori analisi. Gli strumenti di visualizzazione sono quindi utili per l'analisi esplorativa. Per poter visualizzare i file di allineamento, si può utilizzare l'**Integrative Genomics Viewer (IGV)** del Broad Institute. 

**IGV** è un browser autonomo, che ha il vantaggio di essere installato localmente e di fornire un accesso rapido. I browser del genoma basati sul Web, come Ensembl o il browser UCSC, sono più lenti, ma forniscono più funzionalità. Non solo consentono una visualizzazione più raffinata e flessibile, ma forniscono anche un facile accesso a una vasta gamma di annotazioni e fonti di dati esterne. Ciò rende semplice mettere in relazione i dati con informazioni su regioni ripetute, geni noti, caratteristiche epigenetiche o aree di conservazione tra specie, per citarne solo alcuni.

Per poter utilizzare IGV, dovremo scaricare sul nostro computer locale, i file che utilizzeremo come input. Possiamo farlo facilmente tramite l'interfaccia Galaxy, sulla quale abbiamo sempre lavorato. Lavoreremo sul pannello Cronologia in basso a destra, dove dovremo semplicemente selezionare i file desiderati selezionando i loro nomi. 

Quindi, facciamo clic sull'opzione "**Download**", che possiamo trovare su detto pannello nella parte alta della scheda che si aprirà. La nostra selezione verrà scaricata localmente. 

Una guida completa può essere reperita a questo indirizzo https://software.broadinstitute.org/software/igv/UserGuide.

![igv](https://software.broadinstitute.org/software/igv/sites/cancerinformatics.org.igv/files/images/igv_desktop_callouts.jpg)

### **Estrazione del flusso di lavoro**

I **workflow** o **flussi di lavoro** aiutano gli utenti a riprodurre facilmente una specifica analisi che magari è già stata eseguita precedentemente su altri set di dati o a eseguire la medesima analisi ma con parametri degli strumenti differenti.  

Si tratta semplicemente dell’estrazione di una specifica cronologia, caratterizzata dell’esecuzione ordinata di una serie di strumenti, che viene salvata e utilizzata all’occorenza. Una **pipeline**, insomma!

Per trasformare una determinata cronologia di strumenti eseguiti in un workflow, nelle opzioni della storia sulla quale si sta lavorando, bisogna cliccare sulla voce “**Extract Workflow**”. 

Cosi facendo il flusso di lavoro sarà visionabile ed editabile all’interno della sezione “**Worflow**” situata nel menu in alto. Cliccandoci, si viene indirizzati ad una pagina nella quale vengono riportati, in ordine di esecuzione, tutti gli strumenti utilizzati nel corso del flusso di lavoro. Questi vengono selezionati dalla history che abbiamo creato.

Bisogna selezionare quelli che vogliamo includere nel nostro workflow e deselezionare quelli che non ci interessano. Nel nostro caso li selezioniamo tutti quanti, ma cancelliamo i testi riportanti all'interno della sezione di destra relativa ai primi due strumenti di upload della collezione di dati sulla quale eseguire l'analisi e al genoma di riferimento. 

Per la precisione, eliminiamo "**E.coli collection**" e "**ecoli_rel606.fasta.gz**" dalle due caselle riportanti le diciture "**Treat as input dataset E.coli collection**" e "**Treat as input dataset ecoli_rel606.fasta.gz**".

 Successivamente definiamo un nome da assegnare al nostro flusso di lavoro, come ad esempio, "**Pipeline chiamata di varianti**", e clicchiamo su ""**Create Worflow**".
 
A questo punto facciamo un controllo del workflow appena creato. Visitiamo la pagina di tutti i workflow creati cliccando sulla tasto “**Worflow**” situato nel menu  centrale in alto e, facciamo clic con il tasto destro del mouse sul sul nome del nostro flusso di lavoro appena creato (nel nostro caso **Pipeline chiamata di varianti**").

Si aprirà un piccolo menù a tendina riportante una serie di voci, tra le quali dobbiamo cliccare su "**edit**". Verremo indirizzati su una pagina contenenente una tavola riportante lo schema ordinato di tutti gli strumenti del nostro workflow (rappresentati sottoforma di vari blocchi, collegati tramite archi)

Ciascuno strumento, già adeguatamente settato, può essere spostato a piacimento sulla tavola. In una pipeline generalmente, l’output di uno strumento rappresenta l’input di quello successivo.

Per richiamarne uno e magari visionarne o modificarne i parametri, basta cliccarci sopra. Apparirà un pannello sulla destra riportante tutti i dettagli dell'analisi svolta dallo strumento. Possiamo anche inserire la sua funzione in corrispondenza del campo "**Step Annotation**", in maniera tale da ricordare la sua funzione anche in futuro. 

Per avviare il flusso di lavoro con altri dati, basta cliccare su run worflow e caricare i dati di riferimento. A questo punto non rimane che aspettare il termine dell'intera pipeline, che verrà eseguita in maniera totalmente automatica. 
 
