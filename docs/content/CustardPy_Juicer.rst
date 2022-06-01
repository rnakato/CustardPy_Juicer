CustardPy_Juicer
=====================

**CustardPy_Juicer** is a docker image for Juicer analysis in `CustardPy <https://github.com/rnakato/Custardpy>`_.
This is a wrapper of `Juicer <https://github.com/aidenlab/juicer/wiki>`_ and internally executes `juicertools <https://github.com/aidenlab/juicer/wiki/Feature-Annotation>`_.
See the original website for the full description about each command.

Dependencies
---------------------------

**CustardPy_Juicer** internally implements the tools below:

- Juicer version 1.6
- Juicertools version 2.13.07
- JuiceBox version 2.13.07
- `BWA <http://bio-bwa.sourceforge.net/>`_ version 0.7.17

juicer_map.sh
----------------------------------------------------------------

``juicer_map.sh`` generates ``.hic`` file from fastq.
The input fastq files can be gzipped (.fastq.gz).
The results including the ``.hic`` file is outputted in ``$odir``.
The BWA index file is needed.

.. code-block:: bash

    juicer_map.sh [options] <fastqdir> <odir> <build> <gt> <bwaindex> <fastq_post>
      <fastqdir>: directory that contains input fastq files (e.g., "fastq/sample1")
      <odir>: output directory (e.g., "JuicerResults/sample1")
      <build>: genome build (e.g., hg38)
      <gt>: genome table
      <bwaindex>: index file of BWA
      <fastq_post (_|_R)>: if the filename of fastqs is *_[1|2].fastq, supply "_". if *_[R1|R2].fastq, choose "_R".

    Options:
        -e enzyme: enzyme (default: HindIII)
        -p ncore: number of CPUs (default: 32)
        -m tmpdir: tempdir
    Example:
      juicer_map.sh $(pwd)/fastq/Hap1-A/ $(pwd)/JuicerResults/Hap1-A hg38 genometable.hg38.txt bwaindex/hg38 _R

.. note::

    The input fastq files of each sample should be stored in the separated directory.
    For example, if there are three Hi-C samples (``sample1``, ``sample2``, and ``sample3``), the fastq files should be in ``fastq/sample1/``,  ``fastq/sample2/``, and ``fastq/sample3/``.

- Output

    - merged_nodups.txt ... mapped fragments
    - inter.hic ... .hic file (without quality filtering)
    - inter.txt ... stats for inter.hic
    - inter_30.hic ... .hic file (Q>=30) 
    - inter_30.txt ... stats for inter_30.hic

We recommend using ``inter_30.hic`` for the downstream analysis.

(Optional) juicer_pigz.sh
-----------------------------------------------------------------

The output of Juicer is quite large, we provide a script ``juicer_pigz.sh`` that compresses the intermediate files.
This command is optional.

.. code-block:: bash

     juicer_pigz.sh <odir>
       <odir> output directory of juicer_map.sh (e.g., "JuicerResults/sample1")

Note that some commands provided in Juicer use the intermediate files (e.g, mega.sh).
Because these commands do not accept the compressed format, use ``juicer_unpigz.sh`` that uncompresses the compressed files.

.. code-block:: bash

     juicer_unpigz.sh <odir>
       <odir> output directory of juicer_map.sh (e.g., "JuicerResults/sample1")

plot_distance_count.sh
----------------------------------------------------------------

``plot_distance_count.sh`` calcultes the fragment distance and generates a figure (.pdf).
The result is outputted in ``distance/`` directory.

.. code-block:: bash

     plot_distance_count.sh <label> <odir>
       <label>: title of the figure
       <odir> output directory of juicer_map.sh (e.g., "JuicerResults/sample1")

- Output

    - distance_vs_count.10kb.MAPQ30.pdf ... figure of distance plot
    - distance_vs_count.10kb.MAPQ30.txt ... values for the plot
    - distance_vs_count.10kb.MAPQ30.log.pdf ... figure of distance plot (log scale)
    - distance_vs_count.10kb.MAPQ30.log.txt ... values for the plot (log scale)

call_HiCCUPS.sh (GPU required)
----------------------------------------------------------------

``call_HiCCUPS.sh`` calls loops using Juicer HiCCUPS.
Supply ``--nv`` option to the singularity command to activate GPU as follows:

.. code-block:: bash

    singularity exec --nv custardpy_juicer.sif call_HiCCUPS.sh

.. code-block:: bash

    call_HiCCUPS.sh <norm> <odir> <hic> <build>
      <norm>: normalization (NONE|VC|VC_SQRT|KR|SCALE)
      <odir>: output directory
      <hic>: .hic file
      <build>: genome build

- Output

    - merged_loops.simple.bedpe ... loop file

call_MotifFinder.sh
----------------------------------------------------------------

If you have peak files of cohesin and CTCF, you can use MotifFinder by ``call_MotifFinder.sh``:

.. code-block:: bash

    call_MotifFinder.sh <build> <motifdir> <loop>
      <build>: genome build
      <motifdir>: the directory that contains the BED files
      <loop>: loop file (.bedpe) obtained by HiCCUPS

If the ``<build>`` is ``(hg19|hg38|mm9|mm10)``, this command automatically supplies `FIMO <http://meme-suite.org/doc/fimo.html>`_ motifs provided by Juicer.

- Output

    - merged_loops_with_motifs.bedpe

See `MotifFinder manual <https://github.com/aidenlab/juicer/wiki/MotifFinder>`_ for more information.

.. note::

    Because an error occurs in the latest version of juicertools, ``CustardPy`` uses juicertools version 1.9.9 for MotifFinder.
