# Prokaryotic transcriptome analysis
Welcome!!! This is a brief introduction to RNA-seq analysis of prokaryotes. The key thing to take away from this is that prokaryotes do not perform alternative splicing like eukaryotes. We will analyze the activation of competence in Streptococcus sinensis to learn how to analyze prokaryotic RNA-seq data. Competence is a physiological phenomena that occurs in certain bacteria where they can uptake and integrate exogenous DNA from the environment into a bacteria genome. This phenomena is how certain bacteria can acquire antibiotic resistance genes from different species of bacteria. The competence phenomena is regulated by quorum sensing, which is a chemical communication method bacteria use to communicate with each other.
- The genome data can be found here: https://www.ncbi.nlm.nih.gov/assembly/GCF_019090945.2/
- The SRA numbers for RNA-seq data are: SRR11954369, SRR11954371, SRR11954372, SRR11954374
- You can fastq-dump or fasterq-dump to download the RNA-seq data.

# Brief overview of prokaryotic RNA-seq
We will be doing a reference-based mapping of transcripts for analysis, so having a complete or close to complete genome is needed. We are going to be using a mostly complete genome of Streptococcus sinensis that was obtained by a hybrid assembly of two different sequencing technologies.

<a href="https://ibb.co/JjLkz3b"><img src="https://i.ibb.co/nM2kDPV/Screenshot-2023-03-19-155926.png" alt="Screenshot-2023-03-19-155926" border="0"></a>
