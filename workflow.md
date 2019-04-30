### List of seq SRR code and download seq

quello che rende è una lista contenente gli ID e i metadati da scaricare formattati in una tabella. A questo punto si va a cercare i link corrispondenti e li scarica in parallelo con la funzione seguente. Ricordare il db da inserire. (Questo è stao effettuato tramite uno [script SRR] che scarica le sequenze basasndosi sugli Accession codes. Ricorda che le sequenze che verranno scaricate dovranno essere gzippate e, tale funzione, verrà implementata nello script)

        esearch -db sra -query PRJNA301162 | efetch --format runinfo |cut -d "," -f 1 > SRR.numbers

        parallel --jobs 3 "fastq-dump --split-files --origfmt --gzip {}" ::: SRR.numbers


### Running Metaphlan2 recostructing taxonomic profile.

Tutto ciò è stato implementato in uno script che trova i file, genera un file sam, bowtie2 e il profilo. Si splitta la tabella nella sue colonne in maniera pogressiva mantenendo sempre il clade_name. Infine, si mergiano le tabelle basate sui singoli split. Lo scheletro funzionale dei comandi dello script è riportato di seguito e lo [script di Metaphlan2] è nominato:
        metaphlan_script.sh

1.  Profili tassonomici

Per effettuare un ciclo for: 

        for i in $(find . -name "*_trim_adapters.fastq.bz2"); do metaphlan2.py $i ${i%.fastq.bz2}_profile.txt --input_type fastq --nproc 30 --bowtie2out ${i%.fastq.bz2}_bowtie2out.txt --samout ${i%.fastq.bz2}.sam.bz2; done

Questo script mi genera due file: un file .txt, contenente le informazioni sui profili tassonomici, un file intermedio fastq.bz2.bowtie2out.txt e un file sam.bz2.


2. Splittare le colonne delle singole tabelle

3. Per fare il marging delle tabelle splittate:


        python /usr/local/metaphlan2/utils/merge_metaphlan_tables.py *_profile.txt > merged_abundance_table.txt


### Scelta dell'articolo

L'articolo sostitutivo presenta il seguente doi e codice progetto:

        doi:10.1038/nature20796

Su tale paper sono stati lanciati entrambi gli script con la finalità di avere una matrice di abbondanza tassonomica che possa contenere il minimo numero di zeri possibili tra tassonomia e campioni 


### Analilsi in R: preprocessing datas [cutoff_analysis.R]

Quello che è stato fatto è stato prendere le reads che mappano su ogni campione (merged_abundance_table_estimated_number_of_reads_from_the_clade.txt). Si è lanciato Phyloseq e, solo in un secondo momento, si sono normalizzati i dati per RPM, ovvero reads per million e non CPM. Successivamente è stata fata un'analisi che mirava a mettere in risalto la correlazione tra i vari paired, ottenendo un multiplot, in cui:

A) paired contro paired, tenendo presente che le reads sono stata normalizzare in RPM. Quello che si osserva è la distribuzione delle reads per vederne la loro correlazione (scatterplot). L'Obiettivo di questo grafico è valutare, anche graficamente, quale sia il cutoff migliore da usare. Le linee tratteggiate rappresentano il valore al quale sono state cutoffate le reads. Entrambi gli assi riportano il log2p ovvero (log2(x+1))

B) Per avere ulteriore conferm del cutoff utilizzato, abbiamo valutato l'indice di Spearman e abbiamo visto, tratteggiando la linea rossa come area più punteggiata, abbiamo notato che il cutoff utilizzato (linea nera tratteggiata) corrispondeva realmente con il punto di plateaux.

C) cololassando tutta la tassonomia a livello di specie siamo andati a valutara prevalenza relativa sulla media delle conte (abbondanza assoluta e non relativa). quello che è emerso è che è possibile tracciare una retta di regressione lineare tra di loro

D) barplot che mostra l'abbondanza assoluta tenendo conto di una differenziazione tassonomica. Le barre rosse rappresentano tutte le species che sono state mantenute nello studio a seguito del cutoff.


### Network Bayesiano [Bayesian.R]

Utilizzando un algortimo hc abbiamo proceduto con la realizzazione del grapho bayesiano. Questo è stato ripetuto per ogni gruppo nello studio: HCF, NC, HFD.

### Model training [prova.R -> model.training.R]

Ottenuto il modello bayesiano, bisogna trainarlo.

1. Cross-Validation: LOOCV
2. Training: rf

A questo punto

