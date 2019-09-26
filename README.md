# homework2
A Rust Find utility

## Setup
No code will be given for this homework. Please create your own Rust Project using cargo. Name your project `rust_find`:

```bash
cargo new rust_find
```

## Assignment
We will be creating a command line utility similar to the Linux command `find`. That is, given a directory `root_dir` and a regular
expression pattern: `pattern`, your program will recusively go through `root_dir` and all it's sub-directories finding file
names which match `pattern`.

Example:
```bash
> cargo run -- -p ".*\.rs" -d .src/
../temp/homework1/src/lib3.rs
../temp/homework1/src/lib2.rs
../temp/homework1/src/lib.rs
../homework1/src/lib3.rs
../homework1/src/lib2.rs
../homework1/src/lib.rs
../rust_find/src/main.rs
```

Notice the separation with `--` between our programs arguments. This is needed to disambiguate the arguments to cargo
from our arguments. Alternatively, you could avoid this by calling the executable directly via the `./target` directory.
(See next example down below).

This command will return all files which end with `.rs` in the directory ./src and all sub-directories. We will use Rust's
regular expression engine, so the syntax for regular expressions may be different from what you're used to in other
languages or shells.

For example `".*\.rs"`, first `.*` matches any string of any length, then we escape the period
before the file extension: `\.` and finally we match `rs` (Note technically this could match files ending with say "rs.*"
but that's okay.)

So for simplicity, as long you're using the Rust Regex matcher you're doing it right.

Additionally, we may take multiple patterns, or multiple search directories via files. We may also specify an output file
to write the results to. With multiple patterns/dirs, we search all dirs against all files.
(See input section below for more details)

This assignment is vague on purpose. Feel free to implement things as ou wish as long as you handle all errors. The answer to most questions you may have about the specification will be: Do something reasonable.

Please observe how in this example, we can also match at the directory structure, that is, "any file ending with `.rs` under a `src/` directory.

```bash
>>> ./target/debug/rust_find  "src/.*\.rs" ../
../temp/homework1/src/lib3.rs
../temp/homework1/src/lib2.rs
../temp/homework1/src/lib.rs
../homework1/src/lib3.rs
../homework1/src/lib2.rs
../homework1/src/lib.rs
../rust_find/src/main.rs
```

### Input
We will generate our command line input interface using the [StructOpt Crate](https://docs.rs/structopt/0.3.2/structopt/).
The arguments should match this interface:
```bash
rust-find 0.1.0
A command line utility for searching for files with regexes

USAGE:
    rust_find [FLAGS] [OPTIONS] --dir <dir> --pattern <pattern>

FLAGS:
    -h, --help       Prints help information
    -r, --robust     On encountering an error, continue running the program, printing out a wa
rning.
    -V, --version    Prints version information

OPTIONS:
    -d, --dir <dir>                          Directory to search in.
        --dirs-input <dirs-input>            Take directories from file instead of command lin
e.
    -o, --output <output>                    Write results to output file instead of stdout.
    -p, --pattern <pattern>                  Pattern to use.
        --patterns-input <patterns-input>    Take patterns from file instead of command line.
```

Hints:
StructOpt allows you to specify arguments as optional by making their type
an `Option<T>`.

`pattern` and `dir` should be required unless the user specifies either
`pattern-input` or `dirs-input` respectively. This can be accomplished,
by adding the extra option to the StructOpt compiler directive for the field,
like so `#[structopt(..., required_unless = "dirs-input")]`.
This may seem magical but it's not too fancy: StructOpt is built off `Clap`
a crate for command line parsing. You `required_unless` is found
[here](https://docs.rs/clap/2.33.0/clap/struct.Arg.html#method.required_unless).
StructOpt does some macro magic to generate the correct Clap code, and pass in the
correct arguments.

### Output
We print file paths, one per line, which match the `pattern` to stdout. Unless the `output`
option is specified, then the results are written to the specified file.

File path order does not matter, chose whichever order is easiest for you to implement.

There is many places where errors could occur, please exit the program and report the error gracefully,
i.e. do not use panic or expect. **Unless:** see "Robust" section below.

All errors must be written to stderr.

## Robust
Usually, once we encounter an error, we should report and error and end the program. When the `robust`
flag is specified by the user, we attempt to do better. The flag will do the following things:
1) If a pattern in the patterns-file is malformed, we print a warning about the malformed pattern,
    and continue processing the rests of the patterns.
2) If some dir in the dirs-file results in an error (e.g. is not a dir, does not exists, etc), we
    we continue processing the rest of the dirs.

Notice (for simplicity) robust only applies when processing lists of either dirs or patterns. This way
you can have your functions that process single patterns or directories be "all or nothing". That is, you're
not expected to handle your functions partially finding some matches and finding some errors and having to
returns the partial matches, but also the errors (for any given dir). That is, our functions fail or succeed
at the "root directory" level. (This way you can use '?' all the way up your functions).

Aside: if you wanted to properly handle partial results/errors, we would probably need to use iterators on
Result which we have not gone over.

## Design and implementation
Your code is expected to be separated into logical functions, each doing one task. Please use common sense software
engineering practices. Please see the `Grading Policy` section of the class website under `homework assignments`.

### Regexes
Please use the [Regex Crate](https://docs.rs/regex/1.0.5/regex/) for regular expressions.

### Paths
The `Path` type represents OS paths/files/directories, this type is unsized, meaning it is not adequate to pass around
functions on it's own. Instead we will use `PathBuf` for our functions.

You may see `&Path` or consider the type signature signature for `read_dir`:
```rust
pub fn read_dir<P: AsRef<Path>>(path: P) -> Result<ReadDir>
```
This functions can be understood as "takes a `P` as long as `P` can be turned into a Path reference (`P: AsRef<Path>`).
Even though we haven't discussed this feature, pretty much anything that looks like a path will work: PathBuf, String, &str, etc. As long as you pass it by reference.

Similarly: PathBuf implements a `Deref<Target=Path>` meaning `PathBuf` can be thought of a pointer/reference be converted to Path, so any method on `Path` will work on `PathBuf`. Do not worry, we will cover this feature later. You can see
the methods "inherited" from Path [here](https://doc.rust-lang.org/std/path/struct.PathBuf.html). So if you have a PathBuf:
```rust
let pathbuf: PathBuf = ...;
pathbuf.is_dir(); // Just works
```
### Functions to use.
In general, any function on the standard library is fair game to use. You may only use external crates specified in the
homework. Do NOT use any other external crates. Feel free to use functions from external crates that are specified in
the assignment however.

Of interest to you, may be the following functions:
```rust
std::fs::read_dir()
Regex method is_match()
Path/Pathbuf method to_string_lossy()
Vector method extend() and push()
```

### Dealing With Errors:
Many of the filesystem functions we will be using return Result<_>, this can quickly get out of hand. Please use the Rust `?` syntax for propagating errors. I have the following structure for my program which is recommended but not mandatory.

```rust
// Result is aliased to std::io::Result for all these functions.

/// Iterate through file paths, returning only those that match our pattern.
/// Notice that this fuction cannot error.
fn get_matches(paths: Vec<PathBuf>, pattern: Regex) -> Vec<PathBuf>


/// Recurse through the directory structure returning all files in all directories.
fn get_directories(dir: ReadDir) -> Result<Vec<PathBuf>>


/// Given the command line arguments, return a regex and the path to search through.
/// On error, prints message to stderr and returns None.
/// Errors: Not enough or too many arguments.
///         Bad regular expression pattern.
fn parse_args(args: Vec<String>) -> Option<(Regex, PathBuf)>
```

### Testing
Testing programs that interact with the Operating Systems is far more difficult than testing pure code. Therefore, I recommend
separating your pure code (code that does not interact with the OS) with code that does. That is, instead of having a single
function which iterates over the directory substructure, and does the regular expression matching. I recommend two functions,
one function which returns the directory structure as a list (`f1`), and another function that matches `pattern` on element of that
list (`f2`). This way, you can create simple unit tests for `f2`.

Testing is a tool, you should stop testing when you feel confident with your implementation and tests.

## Submisions
Pleas submit this assignment though github class room. This assigment will be due on Wednesday October 9th.
