
## FAQ: How can I report a bug?

Report a bug here:

<https://github.com/sustrik/libdill/issues>

## FAQ: What's structured concurrency?

Check this short introductory [article](structured-concurrency.html) about structured concurrency.

## FAQ: What's libdill's performance?

The best way to find out is to check for yourself. In `perf` subdirectory you'll find small performance bencharks.

Generally speaking, though, libdill's concurrency primitives only a bit slower than simple C flow control statements. Context switch have been seen to execute in 6 ns, coroutine creation in 26 ns. Passing a message via a channel in 40 ns.

## FAQ: How does libdill concurrency differ from Go concurrency?

1. No interaction between threads. Each thread is treated as a separate process.
2. Channels are always unbuffered.
3. `choose`, unlike `select` is deterministic. If multiple clauses can be executed, a clause closer to the beginning of the pollset wins.
4. `chdone` signals closing of the channel to both senders and receivers.
5. Coroutines can be canceled.

## FAQ: How does libdill differ from libmill?

`libmill` was a project to copy Go's concurrency model to C 1:1 without introducing any innovations or experiments. The project is finished now. It will be maintained but won't change in the future.

`libdill` is a follow-up project that diverges from the Go model and experiments with structured concurrency.

Technically, there are following differences:

1. It's C-idiomatic. Whereas libmill takes Go's concurrency API and implements them in almost identical manner in C, libdill tries to provide the same functionality via more C-like and POSIX-y API. For example, `choose` is a function rather than a language construct, Go-style panic is replaced by returning an error code and so on.
2. Coroutines can be canceled. This creates a foundation for "structured concurrency".
3. `chdone` causes blocked `recv` on the channel to return `EPIPE` error rather than a value.
4. `chdone` will signal senders to the channel as well as receivers. This allows for scenarios like multiple senders and single receiver communicating via single channel. The receiver can let the senders know that it's terminating via `chdone`.
5. libmill's `fdwait` was replaced by `fdin` and `fdout`. The idea is that if we want data to flow via the connection in both directions in parallel we should use two coroutines rather than single one.
6. There's no networking library in libdill itself. It is a separate library (dsock).

## FAQ: How can I access source code?

To clone the repository:

```
$ git clone https://github.com/sustrik/libdill.git
```

To build from the source (you'll need automake and libtool):

```
$ ./autogen.sh
$ ./configure
$ make
$ make check
$ sudo make install
```

For easy debugging use the following configure options:

```
$ ./configure --disable-shared --enable-debug --enable-valgrind
```

The above will turn the optimisation out, generate debug symbols and link all the tests with the static version of the library. The second option means that the executables in `tests` subdirectory will be actual debuggable binaries rather that wrapper shell scripts. The last option instructs valgrind about where are the coroutine stacks located and thus prevents spurios valgrind warnings.

## FAQ: Is there continuous integration for libdill?

<iframe width="100%" height="20" frameBorder="0"
src="https://api.travis-ci.org/sustrik/libdill.svg?branch=master"></iframe>

Travis: <https://travis-ci.org/sustrik/libdill>

## FAQ: How can I contribute?

To contribute to libdill create a GitHub pull request. You have to state that your patch is submitted under MIT/X11 license, so that it can be incorporated into the mainline codebase without licensing issues.

If you make a substantial contribution to a file, add your copyright to the file header. Irrespective of whether you do so, your name will be added to the AUTHORS file to indicate you own copyright to a part of the codebase.

## FAQ: What is the release process?

These instructions are intended for the project maintainers:

* Run `make distcheck` to check whether the packaging process still works.
* Bump ABI version as appropriate (see here: <http://250bpm.com/blog:41>).
* Commit it and push it back to the master on GitHub.
* Tag the new version and push the tag to GitHub (e.g. `git tag -a 0.3-beta; git push origin 0.3-beta`).
* Clone a clean repo from GitHub.
* Build the package (`./autogen.sh; ./configure; make distcheck`).
* Add the package to `gh-pages` branch.
* Adjust the download.md in `gh-pages` branch.
* Regenerate the documentaton using `./gendocs.sh`.
* Commit and push to `gh-pages`.
* Announce the release on twitter, etc.
