# pyvoqc

This repository provides a Python wrapper around the VOQC quantum circuit compiler.

VOQC is a **verified optimizer for quantum circuits**, implemented and *formally verified* in the [Coq proof assistant](https://coq.inria.fr/). All VOQC optimizations are *guaranteed* to preserve the semantics of the original circuit, meaning that any optimized circuit produced by VOQC has the same behavior as the input circuit. VOQC was presented as a [distinguished paper at POPL 2021](https://arxiv.org/abs/1912.02250). Coq code and proofs are available at [github.com/inQWIRE/SQIR](https://github.com/inQWIRE/SQIR) and the extraced OCaml code (which the Python wraps) is available at [github.com/inQWIRE/mlvoqc](https://github.com/inQWIRE/mlvoqc)

To run VOQC, we (1) extract the verified Coq code to OCaml, (2) compile the extracted OCaml code to a library, (3) wrap the OCaml library in C, and (4) provide a Python interface for the C wrapper.

## Table of Contents

* [Setup](#setup)
* [Installation](#installation)

* [Acknowledgements](#acknowledgements)

## Setup

pyvoqc requires **Python 3** and a compatible version of pip. We recommend using [Anaconda](https://www.anaconda.com/products/individual) to make sure you get the right versions (see the instructions for setting up a Qiskit environment [here](https://qiskit.org/documentation/getting_started.html))

Although pyvoqc is a Python package, it requires OCaml to build the underlying library code. At some point in the future we will remove this dependency by pre-compiling binaries, but for now you will need to install [opam](https://opam.ocaml.org/doc/Install.html). Once you have opam installed, follow the instructions below to set up your environment.
```
# environment setup
opam init
eval $(opam env)

# install some version of the OCaml compiler in a switch named "voqc"
opam switch create voqc 4.10.0
eval $(opam env)
```

Now, to install VOQC, run:
```
# get the most recent package list
opam update

# install current supported version of voqc (0.2.1)
opam install voqc.0.2.1
```

*Notes*:
* Depending on your system, you may need to replace 4.10.0 in the instructions above with something like "ocaml-base-compiler.4.10.0". Opam error messages and warnings are typically informative, so if you run into trouble then make sure you read the console output.

## Installation

After installing voqc through opam (following the instructions under [Setup](#setup) above), run `./install.sh`. This will build the VOQC library using dune and then "install" our Python package with pip.

*Notes:*
* If you are building the voqc library on a Mac, you will likely see the warning `ld: warning: directory not found for option '-L/opt/local/lib'`. This is due to zarith (see [ocaml/opam-repository#3000](https://github.com/ocaml/opam-repository/issues/3000)) and seems to be fine to ignore.

To check that installation worked, open a Python shell and try `from pyvoqc.voqc import VOQC`.

## Tutorials

The tutorials require JuPyter and Qiskit (`pip install jupyter qiskit`). 

To run the tutorials locally, run `jupyter notebook` on the command line from the pyvoqc directory and open (for example) http://localhost:8888/notebooks/tutorial.ipynb. To view the tutorials online, go to https://nbviewer.jupyter.org/ and copy the link of the relevant tutorial when prompted. Note that you will not be able to run code examples online.

## Directory Contents

* `lib` contains code for building a C libary that wraps around the OCaml VOQC package.
* `pyvoqc/` contains the Python wrapper code.
* `tutorial_files/` contains files for the pyvoqc tutorial.

## API

VOQC currently supports the OpenQASM 2.0 file format, excluding measurement, and the following gates:
* i, x, y, z
* h
* s, sdg
* t, tdg
* rx(f), ry(f), rz(f)
* rzq(i,i)
* u1(f)
* u2(f,f)
* u3(f,f,f)
* cx
* cz
* swap
* ccx
* ccz

Above, "f" is a float expression (possibly including the constant pi) and "i" is an integer expression. rzq is a non-standard gate that we have defined specifically for VOQC. rzq(num,den) performs a rotation about the z-axis by ((num /den) * pi) for integers num and den. We have defined the gate this way to avoid floating point numbers, which significantly complicate verification. 

The following functions are exposed by our Python interface:
* `VOQC(fname)` VOQC class constructor, takes an OpenQASM filename as input
* `print_info` Print circuit information
* `write` Write circuit to a qasm file
* `count_gates`
* `count_clifford_rzq`
* `total_gate_count`
* `check_well_typed`
* `convert_to_rzq`
* `convert_to_ibm`
* `decompose_to_cnot`
* `replace_rzq`
* `optimize_1q_gates`
* `cx_cancellation`
* `optimize_ibm`
* `not_propagation`
* `hadamard_reduction`
* `cancel_single_qubit_gates`
* `cancel_two_qubit_gates`
* `merge_rotations`
* `optimize_nam`
* `check_layout`
* `check_graph`
* `check_constraints`
* `make_tenerife`
* `make_lnn`
* `make_lnn_ring`
* `make_grid`
* `trivial_layout`
* `layout_to_list`
* `list_to_layout`
* `simple_map`
There are examples of using some of these functions in our tutorial. Otherwise, you can use the [documentation for our OCaml library](https://inqwire.github.io/mlvoqc/voqc/Voqc/index.html) for reference.

## Acknowledgements

This project is supported by the U.S. Department of Energy, Office of Science, Office of Advanced Scientific Computing Research, Quantum Testbed Pathfinder Program under Award Number DE-SC0019040.
