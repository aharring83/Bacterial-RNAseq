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
- sra-tools
- FastQC
- MultiQC
- fastp
- bowtie2
- subread
- R/Rstudio

# Downloading data
Commands for downloading the RNA-seq data.
```
mk reads
cd reads
```
```
fasterq-dump -S SRR11954369 -o csp1
```
```
fasterq-dump -S SRR11954371 -o csp2
```
```
fasterq-dump -S SRR11954372 -o dmso1
```
```
fasterq-dump -S SRR11954374 -o dmso2
```
Command to download reference genome
```
cd ..
mkdir ref
cd ref
```
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/019/090/945/GCF_019090945.2_ASM1909094v2/GCF_019090945.2_ASM1909094v2_genomic.fna.gz
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/019/090/945/GCA_019090945.2_ASM1909094v2/GCA_019090945.2_ASM1909094v2_genomic.gtf.gz
```

# Quality control and trimming of raw data
We need to zip the files first. Make a folder, "data" and move the fastq files to the data folder. Inside the data folder, zip the files. You can also create a bash script to streamline the process.
```
cd ../reads
gzip *.fastq
```

```
We are going to make a script to automate the trimming step where we will placing the fastp trimmed files in a folder called 'trim'. Remember to make the saved script executable using chmod +x <script_name.sh>
```
```
#!/bin/bash

# Create the output directory if it doesn't exist
mkdir -p trim

# Loop through each read file that matches the pattern *_1.fastq.gz
for read in *_1.fastq.gz; do
  # Generate the corresponding second read filename by replacing _1.fastq.gz with _2.fastq.gz
  read2=${read/_1.fastq.gz/_2.fastq.gz}
  
  # Generate the output filenames for the trimmed reads
  output1="trim/${read/_1.fastq.gz/_trim_1.fastq.gz}"
  output2="trim/${read2/_2.fastq.gz/_trim_2.fastq.gz}"

  # Run fastp with the input and output files
  fastp -i "$read" -I "$read2" -o "$output1" -O "$output2"
done

```
Some people prefer to run fastqc on both the raw and trim reads. It is up to the analyst to decide how they want to proceed. I usually run fastqc on the trim reads. Here is a script that will create a folder called, "QC" and put the FastQC files in that folder.
'''
#!/bin/bash
mkdir QC
for i in /trim/*.fastq.gz
do
fastqc ${i} -o QC
done
'''
Go into the QC folder.
```
multiqc .
```
Now we can look at the multiqc_report.html to see whether is clean enough for downstream analysis.

# Mapping trimmed reads to reference genome
We need to create an index of our reference genome first. 
```
cd ref
gunzip *.gz
bowtie2-build GCF_019090945.2_ASM1909094v2_genomic.fna ref
mkdir index
mv *.bt2 index
cd ..
```
We created an index called ref that is stored in /ref/index. Now we will map the trim reads to the index and generate a sam file. Here is a script.
```
#!/bin/bash

# Set the path to the bowtie2 index (without the .1.bt2 or other extensions)
BOWTIE2_INDEX="/ref/index/ref"

# Create the output directory if it doesn't exist
mkdir -p alignments

# Loop through each read file that matches the pattern *_trim_1.fastq.gz in the reads/trim directory
for read1 in reads/trim/*_trim_1.fastq.gz; do
  # Print the name of the file being processed
  echo "Processing file: $read1"

  # Check if the first read file exists
  if [[ ! -f "$read1" ]]; then
    echo "Error: Input file $read1 not found!"
    continue
  fi

  # Generate the corresponding second read filename by replacing _trim_1.fastq.gz with _trim_2.fastq.gz
  read2=${read1/_trim_1.fastq.gz/_trim_2.fastq.gz}
  
  # Check if the second read file exists
  if [[ ! -f "$read2" ]]; then
    echo "Error: Paired input file $read2 not found!"
    continue
  fi

  # Generate the output SAM filename by extracting the base name and placing it in the alignments directory
  base_name=$(basename "$read1" _trim_1.fastq.gz)
  output="alignments/${base_name}.sam"

  # Print the bowtie2 command being executed for debugging
  echo "Running bowtie2: bowtie2 -x $BOWTIE2_INDEX -1 $read1 -2 $read2 -S $output --very-sensitive -p 8 --no-discordant"

  # Run bowtie2 with the input files and the designated output file
  bowtie2 -x "$BOWTIE2_INDEX" -1 "$read1" -2 "$read2" -S "$output" --very-sensitive -p 8 --no-discordant --end-to-end
done
```

