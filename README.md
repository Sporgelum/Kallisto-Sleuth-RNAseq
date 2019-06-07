# Kallisto-Sleuth-RNAseq
A full analysis of RNAseq using pseudo alligner kallisto and sleuth for analysis.
---
title: "RNAseq Kallisto(bash) + sleuth(R)"
author: 'Marius:  marius.botos@ana.unibe.ch'
date: "07/06/2019"
output:
  pdf_document: default
  html_notebook: default
  html_document:
    df_print: paged
---

<!---
\begin{center}
#For the use of the kallisto __pseudo allignment__ performed in __bash__ and further __DEG__(differentially expressed genes) analysis with sleuth performed in #__R__, follow the next steps for _Ubuntu_.
#\end{center}>
--->


<center>
For the use of the kallisto __pseudo allignment__ performed in __bash__ and further __DEG__(differentially expressed genes) analysis with sleuth performed in __R__, follow the next steps for _Ubuntu_.
</center>

____
  
  
* Will need to perform different steps to achieve the process:  
  + Indexing 
  + Quantification
  + DEG analysis
  
<center>
Index
</center>


____


For starting we will need to create the Index for the pseudoallignment. It is required the cDNA of the organism used.  
Make sure to download all in the same folder.


Download the cDNA file from ensembl and copy the scripts to your folder from github.  

```{bash}
cd ~/Desktop  
mkdir working_with_kallisto
cd working_with_kallisto
wget --no-verbose ftp://ftp.ensembl.org/pub/release-96/fasta/danio_rerio/cdna/Danio_rerio.GRCz11.cdna.all.fa.gz
git clone 
```



Continue indexing the cDNA using kallisto.  

```{bash}
cd ~/Desktop/working_with_kallisto
kallisto index --make-unique --index Danio_rerio_ensembl_cdna_fa_GRCz11.idx Danio_rerio.GRCz11.cdna.all.fa.gz
```



Index will be obtained with the name __Danio_rerio_ensembl_cdna_fa_GRCz11.idx__ now move to the next step.

<center>
Quantification
</center>

____



For this step first crete the files required for the scrip to run them with parallel cores.


1.Ordered list of samples and paths to the samples. Position in the folder where your samples are and run the following.

```{bash}
cd ~/Desktop/working_with_kallisto
echo "$(ls $PWD/Sample_*/*.gz | sort -V)" >> fastq_files_paths.txt
```


2.Add the number of sample at the begginig of the file to recognize the files.

```{bash}
cd ~/Desktop/working_with_kallisto
awk 'BEGIN{count=1; OFS="\t"}{print "sample"count++, $0}' fastq_files_paths.txt >> fastq_files_paths_samples.txt
```



* With this 2 new files will be obtained:
  + fastq_files_paths.txt _Paths to all the fasrtq.gz of all the samples_
  + fastq_files_paths_samples.txt  _Paths to all the fasrtq.gz of all the samples and the sample number as reference_


It will be used fastq_files_paths_samples.txt to determine the path and launch the command for running the data with multi-core.


Run the command maker for using kallisto in parallel.
```{bash}
cd ~/Desktop/working_with_kallisto
bash kallisto_test.sh fastq_files_paths_samples.txt job_list_for_parallel.txt
```

* It will perform the following:
  1.[bash kallisto_test.sh] → runs the script for creating the multiple jobs.
  2.[fastq_files_paths_samples.txt] → the file previously generated with the paths to the samples.
  3.[job_list_for_parallel.txt] → output of the script.
  + bash kallisto_test.sh → create a new file _job_list_for_parallel.txt_ with the first argument received the fastq_files_paths_samples.txt.  
  

For completing the first part run the kallisto quantification in parallel in your computer for half of the processors in your computer _(recommended)_  

Run the last script and obtain the result in different folders called as samples1..n
Notice `2>&1 | tee final report.txt` is an extra report of all the output redirected to final_report.txt  

```{bash}
cd ~/Desktop/working_with_kallisto
parallel --progress --jobs 4 --joblog kallisto_joblog.txt < job_list_for_parallel.txt 2>&1 | tee final_report.txt
```

<center>
DEG analysis
</center>
____

Run sleuth in R following the codes.

