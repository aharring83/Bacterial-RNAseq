# Prokaryotic transcriptome analysis
Welcome!!! This is a brief introduction to RNA-seq analysis of prokaryotes. The key thing to take away from this is that prokaryotes do not perform alternative splicing like eukaryotes. We will analyze the activation of competence in Streptococcus sinensis to learn how to analyze prokaryotic RNA-seq data. Competence is a physiological phenomena that occurs in certain bacteria where they can uptake and integrate exogenous DNA from the environment into their genome. This phenomena is one of a few mechanisms bacteria use to acquire antibiotic resistance genes from different species of bacteria. The competence phenomena is regulated by quorum sensing, which is a chemical communication method bacteria use to communicate with each other.
- We will using data from this publication: Brennan, A. A., Harrington, A., Guo, M., Renshaw, C. P., Tillett, R. L., Miura, P., & Tal-Gan, Y. (2022). Investigating the Streptococcus sinensis competence regulon through a combination of transcriptome analysis and phenotypic evaluation. Microbiology, 168(10), 001256.
- The genome data can be found here: https://www.ncbi.nlm.nih.gov/assembly/GCF_019090945.2/
- The SRA numbers for RNA-seq data are: SRR11954369, SRR11954371, SRR11954372, SRR11954374
- You can use fastq-dump or fasterq-dump to download the RNA-seq data.

# Brief overview of prokaryotic RNA-Seq
- We will be doing a reference-based mapping of transcripts for analysis, so having a complete or close to complete genome is needed. The genome for this tutorial was created using a hybrid assembly of two different sequencing technologies.

<a href="https://ibb.co/JjLkz3b"><img src="https://i.ibb.co/nM2kDPV/Screenshot-2023-03-19-155926.png" alt="Screenshot-2023-03-19-155926" border="0"></a>

- We will be mapping our RNA-Seq data (transcripts) to the reference genome to determine which gene/s are activated or repressed for a particular condition/experiment. 

<a href="https://ibb.co/XfBMtL3"><img src="https://i.ibb.co/bjhDXm2/Screenshot-2023-03-19-161406.png" alt="Screenshot-2023-03-19-161406" border="0"></a>

- We will be comparing transcript data of 5 minute exposure to exogenous competence stimulating peptide (CSP) versus 5 minute exposure to DMSO. 
- Experimental condition = 5 minute CSP exposure
- Control condition = 5 minute exposure to DMSO. 
- This study used RNA-Seq data to demonstrate that exogenous competence stimulating peptide (CSP) exposure will activate the genes involved in competence activation, thus providing supporting evidence that the competence stimulating peptide regulates the competence phenotype.

# Packages/Software requirements
- FastQC
- fastp
- bowtie2
- R/Rstudio

# Downloading data
Commands for downloading the RNA-seq data.
```
fasterq-dump -S SRR11954369
```
```
fasterq-dump -S SRR11954371
```
```
fasterq-dump -S SRR11954372
```
```
fasterq-dump -S SRR11954374
```
Command to download reference genome
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/019/090/945/GCF_019090945.2_ASM1909094v2/GCF_019090945.2_ASM1909094v2_genomic.fna.gz
```

# Quality control and trimming of raw data
We need to zip the files first. Make a folder, "data" and move the fastq files to the data folder. Inside the data folder, zip the files. You can also create a bash script to streamline the process.
```
gzip *.fastq
```
```
fastp -i <SRR###_1> -I <SRR###_2> -o <SRR###_trim_1.fastq.gz> -O <SRR###_trim_2.fastq.gz>
```
Some people prefer to run fastqc on both the raw and trim reads. It is up to the analyst to decide how they want to proceed. I usually run fastqc on the trim reads. Make a folder, "QC". We will put the QC files in that folder.
'''
fastqc <trim_fastq.gz> -o QC
'''
Go into the QC folder.
```
multiqc .
```
Now we can look at the multiqc_report.html to see whether is clean enough for downstream analysis.

# Mapping trimmed reads to reference genome
We need to create an index of our reference genome first. Go to the folder where the reference genome is located.
```
bowtie2-build GCA_019090945.2_ASM1909094v2_genomic.fna.gz ref
```
We create an index called ref, which we will use for mapping the reads.
