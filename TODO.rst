TODO
====

* Integration with timeit: IN PROGRESS
* Integration with pybench
* Integration with perf.py
* Add duration to Result (metadata?)
* Error or warning if timing is too short:

    $ python3 -m perf.timeit -n 1 1+1
    Average: 5 runs x 3 samples x 1 loop: 420 ns +- 284 ns
    $ python3 -m perf.timeit -n 10 1+1
    Average: 5 runs x 3 samples x 10 loops: 66.1 ns +- 33.2 ns
    $ python3 -m perf.timeit -n 100 1+1
    Average: 5 runs x 3 samples x 100 loops: 21.4 ns +- 2.8 ns
    $ python3 -m perf.timeit 1+1
    Average: 5 runs x 3 samples x 10^7 loops: 17.4 ns +- 0.4 ns