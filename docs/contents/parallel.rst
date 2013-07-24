Parallel execution
------------------

The pipeline runs in parallel in two different ways:

-  multiple cores -- Analyses will run in parallel using multiple cores
   on a single machine. This requires only the ``mulitprocessing``
   Python library, included by default with most Python installations.

-  parallel messaging -- This allows scaling beyond the cores
   available on a single machine, and requires multiple machines
   with a shared filesystem like standard cluster environments.
   Machine to machine communication occurs via messaging, using the
   `IPython parallel`_ framework.

Multiple cores
~~~~~~~~~~~~~~
Running using multiple cores only requires setting the ``-n``
command line flag::

    bcbio_nextgen.py bcbio_system.yaml bcbio_sample.yaml -t local -n 12

IPython parallel
~~~~~~~~~~~~~~~~

`IPython parallel`_ provides a distributed framework for performing
parallel computation in standard cluster environments. The
bcbio-nextgen setup script installs both IPython and `pyzmq`_, which
provides Python bindings for the `ZeroMQ`_ messaging library. The only
additional requirement is that the work directory where you run the
analysis is accessible to all processing nodes. This is typically
accomplished with a distributed file system like
`NFS`_, `Gluster`_ or `Lustre`_.

Run an analysis using ipython for parallel execution::

    bcbio_nextgen.py bcbio_system.yaml bcbio_sample.yaml -t ipython -n 12 -s lsf -q queue

The ``-s`` flag specifies a type of scheduler to use ``(lsf, sge, torque)``.

The ``-q`` flag specifies the queue to submit jobs to.

The ``-n`` flag defines the total number of cores to use on the
cluster during processing. The framework will select the appropriate
number of cores and type of cluster (single core versus multi-core) to
use based on the pipeline stage (see the :ref:`internals-parallel`
section in the internals documentation for more details). For
multiple core steps, the number of cores to use for programs like
``bwa``, ``novoalign`` and ``gatk`` comes from the
:ref:`config-resources` section of the configuration.
Ensure the ``cores`` specification matches the physical cores
available on machines in your cluster, and the pipeline will divide
the total cores specified by ``-n`` into the appropriate number of
multicore jobs to run.

The pipeline default parameters assume a system with minimal time to
obtain processing cores and consistent file system accessibility. These
defaults allow the system to fail fast in the case of cluster issues
which need diagnosis. For running on shared systems with high resource
usage and potential failures due to intermittent cluster issues, there
are turning parameters that increase resiliency. The ``--timeout``
flag specifies the numbers of minutes to wait for a cluster to start
up before timing out. This defaults to 15 minutes. The ``--retries``
flag specify the number of times to retry a job on failure. In systems
with transient distributed file system hiccups like lock errors or disk
availability, this will provide recoverability at the cost of
resubmitting jobs that may have failed for reproducible reasons.

Finally, the ``-r resources`` flag specifies resource options to pass along
to the underlying queue scheduler. This currently supports SGE's
``-l`` parameter and SLURM native flags. This allows specification
or resources to the scheduler (see the `qsub man page`_). You may specify multiple
resources separated with a ``;``, so a ``-r mem=4g;ct=01:40:00``
translates to ``-l mem=4g -l ct=01:40:00`` when passed to ``qsub`` or 
``-r "account=a2010002;timelimit=04:00:00"`` when using SLURM, for instance.

.. _qsub man page: http://gridscheduler.sourceforge.net/htmlman/htmlman1/qsub.html
.. _IPython parallel: http://ipython.org/ipython-doc/dev/index.html
.. _pyzmq: https://github.com/zeromq/pyzmq
.. _ZeroMQ: http://www.zeromq.org/
.. _Gluster: http://www.gluster.org/
.. _Lustre: http://wiki.lustre.org/index.php/Main_Page
.. _NFS: https://en.wikipedia.org/wiki/Network_File_System_%28protocol%29
