uORF-Tools Documentation
========================

**UNDER CONSTRUCTION**
 
Introduction
============

uORF-Tools are a workflow and a collection of tools for the analysis of **Upstream Open Reading Frames** (short uORFs). The workflow is based on the workflow management system snakemake and handles installation of all dependencies via `bioconda <https://bioconda.github.io/>`_ :cite:`GRU:KOE:2018`, as well as all processings steps. The source code of uORF-Tools is open source and available under the License. Installation and basic usage is described below.

Program flowchart
=================

The following flowchart describes the processing steps of the workflow and how they are connected. there is a variant of the workflow accepting a preprocessed uORF-annotation file, to skip the time consuming ribotish step for reruns of the workflow.

.. image:: images/uORFTools.png
    :scale: 50%
    :align: center

The output is written to a directory structure that corresponds to the workflow steps, you can decide at the bedginning of the workflow if you want to keep the intermediary files (default) or only the final result.

**MISSING TABLE**

Installation
============

We recommend to install `uORF-Tools` with all dependencies via conda. Once you have `conda <https://conda.io/docs/user-guide/install/index.html>`_ installed simply type:

.. code-block:: bash

    conda create -c bioconda -c conda-forge -n uORF-Tools snakemake
    source activate uORF-Tools


Usage
=====

.. toctree::
   :maxdepth: 2
   :caption: Contents:



Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

References
==========

.. bibliography:: references.bib
