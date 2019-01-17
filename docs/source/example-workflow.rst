.. _example-workflow:

################
Example workflow
################

The retrieval of input files and running the workflow locally and on a server cluster via a queuing system is demonstrated using an example with data available from our FTP-Server.

.. note:: Ensure that you have miniconda3 installed and a conda environment set-up. Please refer to the :ref:`overview <overview:Tools>` for details on the installation.
If you have not yet activated the conda environment:

.. code-block:: bash

    source activate snakemake

Setup
=====
First of all, we start by creating the project directory and changing to it.

.. code-block:: bash

    mkdir tutorial; cd tutorial;

We then download the lastest version of the **uORF-Tools** into the newly created project folder and unpack it.

.. code-block:: bash

    wget https://github.com/Biochemistry1-FFM/uORF-Tools/archive/2.0.0.tar.gz
    tar -xzf 2.0.0.tar.gz; mv uORF-Tools-2.0.0 uORF-Tools; rm 2.0.0.tar.gz;

Retrieve and prepare input files
================================

Before starting the workflow, we have to acquire and prepare several input files. These files are the annotation file, the genome file, the bam files, the configuration file and the sample sheet.

Annotation and genome files
***************************
First, we want to retrieve the annotation file and the genome file. In this case, we can find both on the `GENCODE <https://www.gencodegenes.org/human/>`_ :cite:`Gencode` webpage for the human genome.

.. image:: images/GenCode_download.png
    :scale: 50%
    :align: center

On this page, we can directly retrieve both files by clicking on the according download links next to the file descriptions. Alternatively, you can directly download them using the following commands:

.. code-block:: bash

    wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_28/gencode.v28.annotation.gtf.gz
    wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_28/GRCh38.p12.genome.fa.gz

Then, we are going to unpack both files.

.. code-block:: bash

    gunzip gencode.v28.annotation.gtf.gz
    gunzip GRCh38.p12.genome.fa.gz

Finally, we will rename these files to *annotation.gtf* and *genome.fa*.

.. code-block:: bash

    mv gencode.v28.annotation.gtf annotation.gtf
    mv GRCh38.p12.genome.fa genome.fa

Another webpage that provides these files is `Ensembl Genomes <http://www.ensembl.org/Homo_sapiens/Info/Index>`_ :cite:`Ensembl:2018`. This usually requires searching their file system in order to find the wanted files. For this tutorial, we recommend to stick to GenCode instead.

.bam files
**********

Next, we want to acquire the bam files. The bam files for the tutorial dataset can be downloaded from our FTP-Server:

.. note:: we provide both a .zip and a .tar.gz file. We recommend the .tar.gz file as most linux systems can decompress them via commandline by default.

.. code-block:: bash

    wget ftp://biftp.informatik.uni-freiburg.de/pub/uORF-Tools/bam.tar.gz; tar -zxvf bam.tar.gz;
    rm bam.tar.gz;

This will create a bam folder containing all the files necessary to run the workflow.
If you prefer using your own .bam files, we suggest creating a bam folder and copying the files into it.

.. code-block:: bash

    mkdir bam; mv *.bam bam/;


Configuration file and sample sheet
***********************************

Finally, we will prepare the configuration file (*config.yaml*) and the sample sheet (*samples.tsv*). We start by copying templates for both files from the *uORF-Tools/templates/* into the *uORF-Tools/* folder.

.. code-block:: bash

    cp uORF-Tools/templates/* uORF-Tools/

The standard template *samples.tsv* looks as follows:

+--------+-----------+-----------+--------------------+
| method | condition | replicate | inputFile          |
+========+===========+===========+====================+
| RIBO   |  A        | 1         | bam/FP-treat-1.bam |
+--------+-----------+-----------+--------------------+
| RIBO   |  A        | 2         | bam/FP-treat-2.bam |
+--------+-----------+-----------+--------------------+
| RIBO   |  B        | 1         | bam/FP-ctrl-1.bam  |
+--------+-----------+-----------+--------------------+
| RIBO   |  B        | 2         | bam/FP-ctrl-2.bam  |
+--------+-----------+-----------+--------------------+

Using any text editor (vim, nano, gedit, atom, ...), we will first edit the *samples.tsv*.
It has to be changed to:

+-----------+-----------+-----------+------------------+
|   method  | condition | replicate | inputFile        |
+===========+===========+===========+==================+
| RIBO      |  A        | 1         | bam/RIBO-A-1.bam |
+-----------+-----------+-----------+------------------+
| RIBO      |  A        | 2         | bam/RIBO-A-2.bam |
+-----------+-----------+-----------+------------------+
| RIBO      |  A        | 3         | bam/RIBO-A-3.bam |
+-----------+-----------+-----------+------------------+
| RIBO      |  A        | 4         | bam/RIBO-A-4.bam |
+-----------+-----------+-----------+------------------+
| RIBO      |  B        | 1         | bam/RIBO-B-1.bam |
+-----------+-----------+-----------+------------------+
| RIBO      |  B        | 2         | bam/RIBO-B-2.bam |
+-----------+-----------+-----------+------------------+
| RIBO      |  B        | 3         | bam/RIBO-B-3.bam |
+-----------+-----------+-----------+------------------+
| RIBO      |  B        | 4         | bam/RIBO-B-4.bam |
+-----------+-----------+-----------+------------------+

.. warning:: **Please ensure not to replace any tabulator symbols with spaces while changing this file.**
.. note:: For simplicity, we provided a ready-to-use sample file *bam-samples.tsv*.
Simply overwrite the *samples.tsv* using:

.. code-block:: bash

    mv uORF-Tools/bam-samples.tsv uORF-Tools/samples.tsv

Next, we are going to set up the *config.yaml*.

.. code-block:: bash

    vim uORF-Tools/config.yaml

This file contains the following variables:

• **taxonomy** Specify the taxonomic group of the used organism in order to ensure the correct removal of reads mapping to ribosomal genes (Eukarya, Bacteria, Archea).
•	**adapter** Specify the adapter sequence to be used. If not set, *Trim galore* will try to determine it automatically. (Option for the extended workflow)
•	**samples** The location of the samples sheet created in the previous step.
•	**genomeindexpath** If the STAR genome index was already precomputed, you can specify the path to the files here, in order to avoid recomputation.
•	**uorfannotationpath** If the uORF-file was already precomputed, you can specify the path to the files here, in order to avoid recomputation.
• **alternativestartcodons** Specify a list of alternative start codons.

.. code-block:: bash

    #Taxonomy of the samples to be processed, possible are Eukarya, Bacteria, Archea
    taxonomy: "Eukarya"
    #Adapter sequence used
    adapter: ""
    samples: "uORF-Tools/samples.tsv"
    genomeindexpath: ""
    uorfannotationpath: ""
    alternativestartcodons: "CTG,GTG,TTG"

For this tutorial, we can keep the default values for the *config.yaml*. The organism analyzed in this tutorial is *homo sapiens*, therefore we keep the taxonomy at *Eukarya*. The path to *samples.tsv* is set correctly.

Running the workflow
====================

Now that we have all the required files, we can start running the workflow, either locally or in a cluster environment.

Run the workflow locally
************************

Use the following steps when you plan to execute the workflow on a single server or workstation. Please be aware that some steps
of the workflow might require a lot of memory, specifically for eukaryotic species.

.. code-block:: bash

    snakemake --use-conda -s uORF-Tools/Snakefile --configfile uORF-Tools/config.yaml --directory ${PWD} -j 20 --latency-wait 60

Run Snakemake in a cluster environment
**************************************

Use the following steps if you are executing the workflow via a queuing system. Edit the configuration file *cluster.yaml*
according to your queuing system setup and cluster hardware. The following system call shows the usage with Grid Engine:

.. code-block:: bash

    snakemake --use-conda -s uORF-Tools/Snakefile --configfile uORF-Tools/config.yaml --directory ${PWD} -j 20 --cluster-config uORF-Tools/sge-cluster.yaml

Example: Run Snakemake in a cluster environment
***********************************************

.. warning:: **Be advised that this is a specific example, the required options may change depending on your system.**

We ran the tutorial workflow in a cluster environment, specifically a TORQUE cluster environment.
Therefore, we created a bash script *torque.sh* in our project folder.

.. code-block:: bash

    vim torque.sh

..note:: Please note that all arguments enclosed in <> have to be customized. This script will only work if your cluster uses the TORQUE queuing system.
We proceeded by writing the queuing script:

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
    snakemake --latency-wait 600 --use-conda -s uORF-Tools/Snakefile --configfile uORF-Tools/config.yaml --directory ${PWD} -j 20 --cluster-config uORF-Tools/torque-cluster.yaml --cluster "qsub -N {cluster.jobname} -S /bin/bash -q {cluster.qname} -d <PATH/ProjectFolder> -l {cluster.resources} -o {cluster.logoutputdir} -j oe"

We then simply submitted this job to the cluster:

.. code-block:: bash

    qsub torque.sh

Using any of the presented methods, this will run the workflow on our dataset and create the desired output files.

Report
******

Once the workflow has finished, we can request an automatically generated *report.html* file using the following command:

.. code-block:: bash

    snakemake --latency-wait 600 --use-conda -s uORF-Tools/Snakefile --configfile uORF-Tools/config.yaml --report report.html

The report for this tutorial can also be downloaded via:

.. code-block:: bash

    wget ftp://biftp.informatik.uni-freiburg.de/pub/uORF-Tools/report_tutorial.html

References
==========

.. bibliography:: references.bib
