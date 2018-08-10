uORF-Tools Documentation
========================

**UNDER CONSTRUCTION**
 
Introduction
============

uORF-Tools are a workflow and a collection of tools for the analysis of **Upstream Open Reading Frames** (short uORFs). The workflow is based on the workflow management system snakemake and handles installation of all dependencies via `bioconda <https://bioconda.github.io/>`_ :cite:`GRU:KOE:2018`, as well as all processings steps. The source code of uORF-Tools is open source and available under the License. Installation and basic usage is described below.

Program flowchart
=================

The following flowchart describes the processing steps of the workflow and how they are connected. there is a variant of the workflow accepting a preprocessed uORF-annotation file, to skip the time consuming ribotish step for reruns of the workflow.

.. image:: uORFTools.png
    :width: 1400px
    :align: center
    :height: 980px

More text

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
