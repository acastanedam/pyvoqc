# Measuring VOQC Performance

## Benchmarks

The benchmarks we use are in the [VOQC-benchmarks repository](https://github.com/inQWIRE/VOQC-benchmarks), added here as a submodule. For convenience, the file 'results.zip' contains the results of running VOQC, PyZX, Qiskit, and tket on the programs in VOQC-benchmarks/Arithmetic_and_Toffoli_partial.

## Running VOQC

In the top (`../..`) directory, run `make voqc`. This will compile the OCaml code we have extracted from the verified Coq code. If you have modified the Coq code, then be sure to run `make optimizer` first. To run the optimizer, run `dune exec --root .. -- ./voqc.exe -i <prog> -o <out>`, which will optimize program prog and write the optimized result to out. It will print the initial and final gate counts. `--root ..` is needed because the voqc executable is built in the parent directory. We also support a command line option `-n <N>` that specifies a number of iterations to consider in the resource count and an option '-l' that indicates whether to use a "light" version of the optimizer that excludes rotation merging.

To generate the data in Tables 2-3 of the VOQC paper (which runs a variety of optimizers on a subset of the full benchmark suite) use the `run_voqc_artifact.sh` script. It requires PyZX, Qiskit, and tket to be appropriately installed (see below) as well as NumPy. On our test machine (a Macbook Pro running macOS Catalina with OCaml 4.10.0) this script takes about 15 minutes.

To run VOQC on *all* the benchmarks run `./run_voqc.sh`. Some of the benchmarks are quite large, so this will take a while (~24 hours on our test machine). This script creates files Arithmetic_and_Toffoli_results.csv, PF_results.csv, and QFT_and_Adders.csv to summarize results.

## Other Optimizers

In our paper, we compare VOQC's performance to existing unverified compilers. We used the scripts in this directory to generate our data. If you are familiar with these compilers and see a problem with how we are running them, please let us know! (Contact <kesha@cs.umd.edu>.)

All of these scripts print gate count data to the console and write to a CSV file. **All of these compilers require Python 3**.

### PyZX

To install PyZX using pip, run `pip install pyzx`. To run Qiskit on the benchmarks, use the command `python run_pyzx.py d f` where `d` is the input directory and `f` is the output filename.

#### Translation Validation

PyZX provides a function for checking equivalence of quantum programs, which can be used to validate compiler outputs (see Sec 3.2 of [arxiv:1904.04735](https://arxiv.org/pdf/1904.04735.pdf)). We can use this as a sanity check on VOQC's outputs. Use the `validate_outputs.sh` script to run VOQC on all the programs in VOQC-benchmarks/Arithmetic_and_Toffoli_partial and validate the results using PyZX. 

At the time of writing, PyZX's translation validator will not succeed on programs compiled by VOQC from the VOQC-benchmarks/PF directory (see `validate_outputs_fail.sh`). This is not an indication of a bug in VOQC, but rather evidence that translation validation is an incomplete solution -- PyZX's translation validator cannot always show that two circuits are equivalent. It may also be the result of different roundoff error in PyZX versus VOQC. VOQC begins by converting float parameters to rational numbers, performs optimization over rationals, and then converts  rationals back to floats with 53 bits of precision.

### Qiskit

To install Qiskit using pip, run `pip install qiskit`. To run Qiskit on the benchmarks, use the command `python run_qiskit.py d f` where `d` is the input directory and `f` is the output filename.

### tket

To install tket using pip, run `pip install pytket`. To run Qiskit on the benchmarks, use the command `python run_tket.py d f` where `d` is the input directory and `f` is the output filename.

