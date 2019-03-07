.. _preprocessing-workflow:

######################
Preprocessing workflow
######################

The retrieval of input files and running the workflow locally and on a server cluster via a queuing system is demonstrated using an example with data available from SRA via NCBI.
The dataset is available under the GEO accession number *GSE103719*. The retrieval of the data is described in this tutorial.

.. note:: Ensure that you have **miniconda3** installed and a conda environment set-up. Please refer to the :ref:`overview <overview:Tools>` for details on the installation.

Setup
=====

First of all, we start by creating the project directory and changing to it.

.. code-block:: bash

    $ mkdir preprocessing_project
    $ cd preprocessing_project

.. code-block:: bash

    $ wget https://github.com/Biochemistry1-FFM/uORF-Tools/archive/3.0.0.tar.gz
    $ tar -xzf 3.0.0.tar.gz; mv uORF-Tools-3.0.0 uORF-Tools; rm 3.0.0.tar.gz;

Retrieve and prepare input files
================================

Before starting the workflow, we have to acquire and prepare several input files. These files are the annotation file, the genome file, the fastq files, the configuration file and the sample sheet.

Annotation and genome files
***************************

.. code-block:: bash

    $ wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_28/gencode.v28.annotation.gtf.gz && wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_28/GRCh38.p12.genome.fa.gz

Then, we are going to unpack both files:

.. code-block:: bash

    $ gunzip gencode.v28.annotation.gtf.gz && gunzip GRCh38.p12.genome.fa.gz

Finally, we will rename these files to *annotation.gtf* and *genome.fa*.

.. code-block:: bash

    $ mv gencode.v28.annotation.gtf annotation.gtf && mv GRCh38.p12.genome.fa genome.fa

Another webpage that provides these files is `Ensembl Genomes <http://www.ensembl.org/Homo_sapiens/Info/Index>`_ :cite:`Ensembl:2018`. This usually requires searching their file system in order to find the wanted files. For this tutorial, we recommend to stick to GenCode instead.

Fastq files
***********
In this example, we will use both *RNA-seq* and *RIBO-seq* data. In order to fasten up the tutorial, we download only 2 of the 4 replicates available for each Condition.

.. note:: Please note that you should always use all available replicates, when analyzing your data.

European Nucleotide Archive (ENA)
---------------------------------

For many datasets, the easiest way to retrieve the fastq files is using the `European Nucleotide Archive <https://www.ebi.ac.uk/ena>`_ :cite:`SIL:KEA:2017european` as it provides direct download links when searching for a dataset.
Use the interface on ENA or type the follwing commands:

.. code-block:: bash

    $ wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR602/005/SRR6026765/SRR6026765.fastq.gz;
    $ wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR602/006/SRR6026766/SRR6026766.fastq.gz;
    $ wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR602/009/SRR6026769/SRR6026769.fastq.gz;
    $ wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR602/000/SRR6026770/SRR6026770.fastq.gz;
    $ wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR602/003/SRR6026773/SRR6026773.fastq.gz;
    $ wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR602/004/SRR6026774/SRR6026774.fastq.gz;
    $ wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR602/007/SRR6026777/SRR6026777.fastq.gz;
    $ wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR602/008/SRR6026778/SRR6026778.fastq.gz;


Then, we create a fastq folder and move all the *.fastq.gz* files into this folder.

.. code-block:: bash

    $ mkdir fastq; mv *.fastq.gz fastq/;


Configuration file and sample sheet
***********************************

Finally, we will prepare the configuration file (*config.yaml*) and the sample sheet (*samples.tsv*). We start by copying templates for both files from the *uORF-Tools/templates/* into the *uORF-Tools/* folder.

.. code-block:: bash

    $ cp uORF-Tools/templates/fastq-samples.tsv uORF-Tools/

The template looks as follows:

+--------+-----------+-----------+---------------------------+
| method | condition | replicate | inputFile                 |
+========+===========+===========+===========================+
| RNA    |  A        | 1         | fastq/SRR6026769.fastq.gz |
+--------+-----------+-----------+---------------------------+
| RNA    |  A        | 2         | fastq/SRR6026770.fastq.gz |
+--------+-----------+-----------+---------------------------+
| RNA    |  B        | 1         | fastq/SRR6026765.fastq.gz |
+--------+-----------+-----------+---------------------------+
| RNA    |  B        | 2         | fastq/SRR6026766.fastq.gz |
+--------+-----------+-----------+---------------------------+
| RIBO   |  A        | 1         | fastq/SRR6026777.fastq.gz |
+--------+-----------+-----------+---------------------------+
| RIBO   |  A        | 2         | fastq/SRR6026778.fastq.gz |
+--------+-----------+-----------+---------------------------+
| RIBO   |  B        | 1         | fastq/SRR6026773.fastq.gz |
+--------+-----------+-----------+---------------------------+
| RIBO   |  B        | 2         | fastq/SRR6026774.fastq.gz |
+--------+-----------+-----------+---------------------------+

.. warning:: **Please ensure that you do not replace any tabulator symbols with spaces while changing this file.**
.. note:: For simplicity, we provided a ready-to-use sample file *fastq-samples.tsv*.


Next, we are going to set up the *config.yaml*.

.. code-block:: bash

    $ cp uORF-Tools/templates/config.yaml uORF-Tools
    $ vi uORF-Tools/config.yaml


This file contains the following variables:

    • **taxonomy** Specify the taxonomic group of the used organism in order to ensure the correct removal of reads mapping to ribosomal genes (Eukarya, Bacteria, Archea). (Option for the preprocessing workflow)
    •	**adapter** Specify the adapter sequence to be used. If not set, *Trim galore* will try to determine it automatically. (Option for the preprocessing workflow)
    •	**samples** The location of the samples sheet created in the previous step.
    •	**genomeindexpath** If the STAR genome index was already precomputed, you can specify the path to the files here, in order to avoid recomputation. (Option for the preprocessing workflow)
    •	**uorfannotationpath** If a uORF-annotation file was already pre-computed, you can specify the path to the file here. Please make sure, that the file has the same format as the uORF_annotation_hg38.csv file provided in the git repo (i.e. same number of columns, same column names)
    • **alternativestartcodons** Specify a comma separated list of alternative start codons.

Change the config file as follows:

.. code-block:: bash

    #Taxonomy of the samples to be processed, possible are Eukarya, Bacteria, Archea
    taxonomy: "Eukarya"
    #Adapter sequence used
    adapter: ""
    samples: "uORF-Tools/fastq-samples.tsv"
    genomeindexpath: ""
    uorfannotationpath: ""
    alternativestartcodons: ""


Running the workflow
====================

Now that we have all the required files, we can start running the workflow, either locally or in a cluster environment.

Information about processing parameters
***************************************

In this pipeline we use trim_galore for adapter and quality trimming (Parameters: --phred33 -q 20 --length 15 --trim-n --suppress_warn --clip_R1 1), sortmerna for rRNA removal (rRNA fasta files are obtained from `https://github.com/biocore/sortmerna/tree/master/rRNA_databases <https://github.com/biocore/sortmerna/tree/master/rRNA_databases>`_, according to the specified Taxon (i.e. Eukarya, Bacteria, Archea)) and STAR for read alignment (Parameters: --outSAMtype BAM SortedByCoordinate --outSAMattributes All --outFilterMultimapNmax 1 --alignEndsType Extend5pOfRead1).

Run the workflow locally
************************

Use the following steps when you plan to execute the workflow on a single server or workstation. Please be aware that some steps
of the workflow require a lot of memory, specifically for eukaryotic species.

.. code-block:: bash

    $ snakemake --use-conda -s uORF-Tools/Preprocessing_Snakefile --configfile uORF-Tools/config.yaml --directory ${PWD} -j 20 --latency-wait 60

Run Snakemake in a cluster environment
**************************************

Use the following steps if you are executing the workflow via a queuing system. Edit the configuration file *cluster.yaml*
according to your queuing system setup and cluster hardware. The following system call shows the usage with Grid Engine:

.. code-block:: bash

    $ snakemake --use-conda -s uORF-Tools/Preprocessing_Snakefile --configfile uORF-Tools/config.yaml --directory ${PWD} -j 20 --cluster-config uORF-Tools/cluster.yaml

Example: Run Snakemake in a cluster environment
***********************************************

.. warning:: **Be advised that this is a specific example, the required options may change depending on your system.**

We ran the tutorial workflow in a cluster environment, specifically a TORQUE cluster environment.
Therefore, we created a bash script *torque.sh* in our project folder.

.. code-block:: bash

    $ vi torque.sh

We proceeded by writing the queueing script:

.. code-block:: bash

    #!/bin/bash
    #PBS -N <ProjectName>
    #PBS -S /bin/bash
    #PBS -q "long"
    #PBS -d <PATH/ProjectFolder>
    #PBS -l nodes=1:ppn=1
    #PBS -o <PATH/ProjectFolder>
    #PBS -j oe
    cd <PATH/ProjectFolder>
    source activate uORF-Tools
    snakemake --latency-wait 600 --use-conda -s uORF-Tools/Preprocessing_Snakefile --configfile uORF-Tools/config.yaml --directory ${PWD} -j 20 --cluster-config uORF-Tools/templates/torque-cluster.yaml --cluster "qsub -N {cluster.jobname} -S /bin/bash -q {cluster.qname} -d <PATH/ProjectFolder> -l {cluster.resources} -o {cluster.logoutputdir} -j oe"

We then simply submitted this job to the cluster:

.. code-block:: bash

    $ qsub torque.sh

Using any of the presented methods, this will run the workflow on our dataset and create the desired output files.

Report
******

Once the workflow has finished, we can request an automatically generated *report.html* file using the following command:

.. code-block:: bash

    $ snakemake --latency-wait 600 --use-conda -s uORF-Tools/Preprocessing_Snakefile --configfile uORF-Tools/config.yaml --report report.html

==========

.. bibliography:: references.bib
