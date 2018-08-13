.. _example-workflow:
################
Example Workflow
################

Example workflow
================

The retrieval of input files and running the workflow locally and on a server cluster via a queuing system is demonstrated using an example with data available from SRA via NCBI.
The dataset is available under the GEO accession number GSE66929. The retrieval of the data is described in this tutorial.

Setup
=====
First of all, we start by creating the project directory and changing to it.

.. code-block:: bash

    mkdir tutorial; cd tutorial;|
	
We then download the *uORF-Tools* into the newly created project folder.

.. code-block:: bash

    git clone git@github.com:anibunny12/uORF-Tools.git

Retrieve and prepare input files
================================

Before starting the workflow, we have to acquire and prepare several input files. These files are the annotation file, the genome file, the fastq files, the configuration file and the sample sheet.

Annotation and genome files
***************************
First, we want to retrieve the annotation file and the genome file. In this case we can find both on the `Link GENCODE <https://www.gencodegenes.org/releases/current.html>´_ webpage for the human genome.

.. image:: images/GenCode_download.png
    :scale: 50%
    :align: center

On this page, we can directly retrieve both files by clicking on the according download links next to the file descriptions. Alternatively, you can directly download them using the following commands:

.. code-block:: bash

    wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_28/gencode.v28.annotation.gtf.gz
    wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_28/GRCh38.p12.genome.fa.gz

Then, we are going to decompress both files.

.. code-block:: bash

    gunzip gencode.v28.annotation.gtf.gz
    gunzip GRCh38.p12.genome.fa.gz
	
Finally, we will rename these files to *annotation.gtf* and *genome.fa*. 

.. code-block:: bash

    mv gencode.v28.annotation.gtf annotation.gtf
    mv GRCh38.p12.genome.fa genome.fa

Another webpage that provides these files is `Link Ensembl Genomes <http://www.ensembl.org/Homo_sapiens/Info/Index>`_. This usually requires searching their file system in order to find the wanted files. For this tutorial, we recommend to stick to GenCode instead.


References
==========

.. bibliography:: references.bib
