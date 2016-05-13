# cpp-script-template

An impractical way to run scripts written in C++, by abusing `lli`.

## Requirments

  1. A Unix-like environment in which shebang commands are parsed.
  2. `clang`, including `lli`.

## Instructions

    $ chmod 700 cpp-script-template
    $ ./cpp-script-template "a b" c
    This is a test script.
    Parameter 0 is: -
    Parameter 1 is: a b
    Parameter 2 is: c

## Explanation

    ///bin/true; exec /usr/bin/env clang -x c++ -S -emit-llvm "$0" -o - | lli - "$@" ; exit "$?"

A shebang can only execute binaries, and not shell commands.
The first part `true` is a no-op to bypass that restriction.
It allows the shell command `exec` to be called
  so that `env` will not spawn a grand-child process.

The actual program ran is `clang` for compiling the code.
Since the script does not have the usual C++ extension,
  the `-x c++` flag is needed to tell `clang`
  that the input file contains C++ code
The `-S` flag tells `clang` to process up to assembly code generation
  and, because of the `-emit-llvm` flag, to output the LLVM IR.
After that, comes the input file path given by 0-th position parameter`$0`.
This parameter expands to the path of the script file.
And finally, the flag `-o -` tells `clang` to write the LLVM IR
  to `stdout`.

After `clang` is done,
  the produced LLVM IR
  is then piped to the LLVM interpreter `lli`.
As the name suggests,
  it runs the intermediate code.
The variable `"$@"` stores the parameter given to the script
  when it was called.
The flag `-` tells `lli` to forward those parameters to the running script.

After the interpreter exits, all is done.
However, the shell is not because`env` does not know where to stop.
So `exit` is called with exit code `$?`,
  which is the exit code returned by the previous command.

## Additional notes

It is possibly useful to add the following flags to `clang`:

    -pedantic
    -std=c++14
    -Wall

Note that using `-Wall` on the script template will give warnings
  because of the links in the comments.
In this case, `-Werror` may be used instead.

## References

See https://stackoverflow.com/a/30082862 on how the shebang works,
and http://stackoverflow.com/q/75538#comment119366_78840 for the links.
