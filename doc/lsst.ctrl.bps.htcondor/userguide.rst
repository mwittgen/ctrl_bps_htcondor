.. _htc-plugin-preqs:

Prerequisites
-------------

#. `ctrl_bps`_, the package providing LSST Batch Processing Service.

#. `HTCondor`_ cluster. NCSA hosts a few of such clusters, see `this`__ page
   for details.

#. HTCondor's Python `bindings`__.

.. __: https://developer.lsst.io/services/batch.html
.. __: https://htcondor.readthedocs.io/en/latest/apis/python-bindings/index.html

.. _htc-plugin-installing:

Installing the plugin
---------------------

Install the HTCondor plugin similarly to any other LSST package:

.. code-block:: bash

   git clone https://github.com/lsst/ctrl_bps_htcondor
   cd ctrl_bps_htcondor
   setup -k -r .
   scons

.. warning::

   Under normal circumstances, we **strongly** recommend to use `ctrl_bps`
   commands for performing the operations described below first (run ``bps
   --help`` to see currently supported commads).  However, using HTCondor
   commands directly may be useful to bypass `ctrl_bps`_ completely during
   debugging/troubleshooting.

.. _htc-plugin-status:

Checking status
---------------

To check the status of the submitted run use `condor_q`_.


.. _htc-plugin-cancelling:

Canceling submitted jobs
------------------------

To remove the jobs from the job queue, use `condor_rm`_ which takes the
``runId`` printed by `condor_q`_. For example:

.. code-block:: bash

   condor_rm 176270.0

If jobs are hanging around in the queue with an ``X`` status in the report
displayed by `condor_q`_, you can add the following to force delete those jobs
from the queue ::

        --pass-thru "-forcex"

If you want to just clobber all of the runs that you have currently submitted,
you can just do the following:

.. code-block:: bash

   condor_rm <username>

.. _htc-plugin-restarting:

Restarting a failed run
-----------------------

Restart a failed run with

.. code-block::

   condor_submit_dag <submit_dir>/dag

.. note::

   When execution of a workflow is managed by HTCondor, the BPS is able to
   instruct it to automatically retry jobs which failed due to exceeding their
   memory allocation with increased memory requirements (see the documentation
   of ``memoryMultiplier`` option for more details).  However, these increased
   memory requirements are not preserved between restarts.  For example, if a
   job initially run with 2 GB of memory and failed because of exceeding the
   limit,  HTCondor will retry it with 4 GB of memory.  However, if the job and
   as a result the entire workflow fails again due to other reasons, the job
   will ask for 2 GB of memory during the first execution after the workflow is
   restarted.

.. _htc-plugin-settings:

Supported setttings
-------------------

HTCondor plugin supports all settings described in ``ctrl_bps`` `documentation`__ *except* **preemptible**.

.. __: https://pipelines.lsst.io/v/weekly/modules/lsst.ctrl.bps/quickstart.html#supported-settings

.. _htc-plugin-troubleshooting:

Troubleshooting
---------------

Where is stdout/stderr from pipeline tasks?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For now, stdout/stderr can be found in files in the submit run directory. The
names are of the format:

.. code-block:: bash

   <run submit dir>/jobs/<task label>/<quantum graph nodeNumber>_<task label>_<templateDataId>[.<htcondor job id>.[sub|out|err|log]

Why did my submission fail?
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Check the ``*.dag.dagman.out`` in run submit directory for errors, in
particular for ``ERROR: submit attempt failed``.

How to match jobs to glideins?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Specify compute site as follow:

.. code-block:: YAML

   site:
     acsws02:
       profile:
         condor:
           requirements: "(GLIDEIN_NAME == &quot;test_gname&quot;)"
           +GLIDEIN_NAME: "test_gname"

.. _ctrl_bps: https://github.com/lsst/ctrl_bps
.. _HTCondor: https://htcondor.readthedocs.io/en/latest/
.. _condor_q: https://htcondor.readthedocs.io/en/latest/man-pages/condor_q.html
.. _condor_rm: https://htcondor.readthedocs.io/en/latest/man-pages/condor_rm.html
