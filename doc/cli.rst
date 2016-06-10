Command line interface
======================

perf.timeit CLI
---------------

Microbenchmark::

    python3 -m perf.timeit
        [-p PROCESSES] [-n SAMPLES] [-l LOOPS] [-w WARMUPS]
        [--metadata] [--json [FILENAME]] [--raw]
        [-h/--help] [-v]
        [-s SETUP]
        stmt [stmt ...]

Iterations:

* ``PROCESSES``: number of processes used to run the benchmark (default: 25)
* ``SAMPLES``: number of samples per process (default: 3)
* ``WARMUPS``: the number of samples used to warmup to benchmark (default: 1)
* ``LOOPS``: number of loops per sample. By default, the timer is calibrated
  to get samples taking between 100 ms and 1 sec.

Options:

* ``-v`` enables verbose mode
* ``-vv`` enables very verbose mode
* ``--metadata`` displays metadata
* ``--raw`` runs a single process (must only be used internally)
* ``--json`` writes result as JSON into stdout, and write other messages
  into stderr
* ``--json-file=FILENAME`` writes result as JSON into *FILENAME*, and write
  other messages into stdout

Example::

    $ python3 -m perf.timeit 1+1
    .........................
    Average: 18.3 ns +- 0.3 ns

Use ``-v`` to enable the verbose mode::

    $ python3 -m perf.timeit -v 1+1
    .........................
    Metadata:
    - aslr: enabled
    - cpu_count: 4
    - cpu_model_name: Intel(R) Core(TM) i7-3520M CPU @ 2.90GHz
    - platform: Linux-4.4.8-300.fc23.x86_64-x86_64-with-fedora-23-Twenty_Three
    - python_executable: /usr/bin/python3
    - python_implementation: cpython
    - python_version: 3.4.3
    - timeit_loops: 10^7
    - timeit_setup: 'pass'
    - timeit_stmt: '1+1'

    Average: 18.3 ns +- 0.3 ns (25 runs x 3 samples; 1 warmup)

Try also ``-vv`` to enable the very verbose mode.

.. note::
   timeit ``-n`` (number) and ``-r`` (repeat) options become ``-l`` (loops) and
   ``-n`` (runs) in perf.timeit.


perf CLI
--------

Display a result file::

    python3 -m perf
        [-v/--verbose] [-M/--no-metadata]
        show filename.json [filename2.json ...]

Compare two result files::

    python3 -m perf
        [-v/--verbose] [-M/--no-metadata]
        compare ref.json changed.json

If a filename is ``-``, read the JSON content from stdin.

Example: first create a JSON file using timeit::

    $ python3 -m perf.timeit --json-file=run.json 1+1
    .........................
    Average: 18.6 ns +- 0.4 ns

Display the JSON file::

    $ python3 -m perf show run.json
    Metadata:
    - aslr: enabled
    - cpu_count: 4
    - cpu_model_name: Intel(R) Core(TM) i7-3520M CPU @ 2.90GHz
    - platform: Linux-4.4.8-300.fc23.x86_64-x86_64-with-fedora-23-Twenty_Three
    - python_executable: /usr/bin/python3
    - python_implementation: cpython
    - python_version: 3.4.3
    - timeit_loops: 10^7
    - timeit_setup: 'pass'
    - timeit_stmt: '1+1'

    Average: 17.4 ns +- 0.8 ns

Metadata is displayed by default, whereas timeit hides them by default. Use
``-M`` (``--no-metadata``) to hide metadata and ``-v`` (``--verbose``) to enable
the verbose mode::

    $ python3 -m perf -M -v show run.json
    Run 1/25: warmup (1): 19.4 ns; runs (3): 18.2 ns, 18.2 ns, 18.2 ns
    Run 2/25: warmup (1): 18.2 ns; runs (3): 18.2 ns, 18.2 ns, 18.2 ns
    Run 3/25: warmup (1): 18.2 ns; runs (3): 18.2 ns, 18.2 ns, 18.2 ns
    (...)
    Run 25/25: warmup (1): 18.2 ns; runs (3): 18.2 ns, 18.2 ns, 18.2 ns
    Average: 18.6 ns +- 0.4 ns (25 runs x 3 samples; 1 warmup)

Try also ``-vv`` to enable the very verbose mode.


perf.metadata CLI
-----------------

Display collected metadata::

    python3 -m perf.metadata

Example::

    $ python3 -m perf.metadata
    aslr: enabled
    cpu_count: 4
    cpu_model_name: Intel(R) Core(TM) i7-3520M CPU @ 2.90GHz
    date: 2016-06-09T21:39:57
    hostname: selma
    platform: Linux-4.4.8-300.fc23.x86_64-x86_64-with-fedora-23-Twenty_Three
    python_executable: /usr/bin/python3
    python_implementation: cpython
    python_version: 3.4.3


timeit versus perf.timeit
=========================

The timeit module of the Python standard library has multiple issues:

* It displays the minimum
* It only runs the benchmark 3 times using a single process (1 run, 3 samples)
* It disables the garbage collector

perf.timeit is more reliable and gives a result more representative of a real
use case:

* It displays the average and the standard deviation
* It runs the benchmark in multiple processes (default: 25 runs, 3 samples)
* By default, it uses a first sample in each process to "warmup" the benchmark
* It does not disable the garbage collector

If a benchmark is run using a single process, we get the performance for one
specific case, whereas many parameters are random:

* Since Python 3, the hash function is now randomized and so the number of
  hash collision in dictionaries is different in each process
* Linux uses address space layout randomization (ASLR) by default and so
  the performance of memory accesses is different in each process

The article `My journey to stable benchmark, part 3 (average)
<https://haypo.github.io/journey-to-stable-benchmark-average.html>`_ explains
in depth the multiple issues of being focused on the minimum.


.. _metadata:

Metadata
========

Benchmark:

* ``inner_loops``: number of inner iterations per sample, see the
  :attr:`~perf.text_runner.TextRunner.inner_loops` attribute of
  :class:`~perf.text_runner.TextRunner`
* ``loops``: number of (outter) iterations per sample

Python metadata:

* ``python_implementation``: Python implementation. Examples: ``cpython``,
  ``pypy``, etc.
* ``python_version``: Python version, ex: ``2.7.11``
* ``python_executable``: path to the Python binary program
* ``python_unicode``: Implementation of Unicode, ``UTF-16`` or ``UCS-4``,
  only set on Pyhon 2.7, Python 3.2 and older

System metadata:

* ``hostname``: Host name
* ``platform``: short string describing the platform
* ``cpu_count``: number of CPUs

Linux metadata:

* ``cpu_model_name``: CPU model name
* ``aslr``: Address Space Layout Randomization (ASLR), ``enabled`` or
  ``disabled``
* ``cpu_affinity``: if set, the process is pinned to the specified list of
  CPUs

Misc:

* ``date``: date when the benchmark started, formatted as ISO 8601

See the :func:`perf.metadata.collect_metadata` function.