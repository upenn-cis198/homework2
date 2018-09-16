# homework2
A Rust Find utility

## Setup
No code will be given for this homework. Please create your own Rust Project using cargo. Name your project `rust_find`:

```bash
cargo new rust_find
```

## Assignment
We will be creating a command line utility similar to the Linux command `find`. That is, given a directory `root_dir` and a regular
expression pattern: `pattern`, your program will recusively go through `root_dir` and all it's subdirectories finding file
names which match `pattern`.

Example:
```bash
# ./rust_find <pattern> <root_dir>
./rust_find *.rs ./src
```

This command will return all files which end with `.rs` in the directory ./src and all subdirectories.

### Input
TODO

### Output
TODO

### Design and implementation
Your code is expected to be separated into logical functions, each doing one task. Please use common sense software
engineering practices. Please see the `Grading Policy` section of the classe's website under `homework assigments`

### Testing
Testing programs that iteract with the Operating Systems is far more difficult than testing pure code. Therefore, I recommend
separting your pure code (code that does not interact with the OS) with code that does. That is, instead of having a single
function which iterates over the directory substructure, and does the regular expression matching. I recommend two functions,
one function which returns the directory structure as a list (`f1`), and another function that matches `pattern` on element of that
list (`f2`). This way, you can create simple unit tests for `f2`.

## Submisions
Pleas submit this assignment though github class room. This assigment will be due on Wednesday September 26.
