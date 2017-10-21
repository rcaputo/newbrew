Newbrew is a script to automate installing packages on a clean
homebrew system.

Homebrew is a package manager for Mac OSX.  It occasionally needs to
be nuked from orbit, such as after a major XCode upgrade.  It's
inconvenient to rebuild everything after that.

Newbrew isn't completely automatic.  It must be told which packages to
install and which nonstandard flags to use (if any).

It reads its configuration from standard input, in the form of
`package:` followed by zero or more `+package` and `-package` flags.
Those represent `--with-package` and `--without-package` options,
respectively.

For example, this will `brew install libtiff --with-xz`:

    libtiff: +xz

Some packages advise additional commands to be run after installation.
They can be included as zero or more `!{command}` options, as in:

    wireshark: +gtk+ !{brew cask install wireshark-chmodbpf}

Additional `!{}` commands will be executed in the order they are
configured.  Beware: These commands will be filtered through at least
two shell-like processes.  Try to avoid characters that shells find
interesting, or be prepared to quote them.

Newbrew writes an executable dependency graph to standard output.
Progress is written to standard error.  Each node on the graph
includes the expanded `brew install` command and any additional `!{}`
commands needed to install the package.  For example:

    % echo 'libtiff: +xz !{@echo "So that happened."}' | ./newbrew
    Discovering dependencies for package 'libtiff'...
      Ignoring non-package option: --c++11
    Discovering dependencies for package 'jpeg'...
    Discovering dependencies for package 'xz'...
    Discovering build dependencies for package 'jpeg'...
    Discovering build dependencies for package 'xz'...
    Discovering build dependencies for package 'libtiff'...
    Writing Makefile to standard output.
    all: \
            /usr/local/Cellar/jpeg \
            /usr/local/Cellar/libtiff \
            /usr/local/Cellar/xz

    /usr/local/Cellar/jpeg:
            brew install jpeg

    /usr/local/Cellar/libtiff: \
                    /usr/local/Cellar/jpeg \
                    /usr/local/Cellar/xz
            brew install libtiff \
                    --with-xz
            @echo "So that happened."

    /usr/local/Cellar/xz:
            brew install xz

It's recommended to direct newbrew's output to a file, as in:

    echo 'openssl:' | ./newbrew > Makefile

From there, the graph may be tested using `make -n` (or `make -n -f
graph_file` if the file's name isn't Makefile).  The `-n` flag tells
make to display the commands that would be run but not execute them.

    % echo 'openssl:' | ./newbrew | make -n -f -
    Discovering dependencies for package 'openssl'...
    Discovering build dependencies for package 'openssl'...
    Discovering build dependencies for package 'makedepend'...
    Discovering build dependencies for package 'pkg-config'...
    Writing Makefile to standard output.
    echo 'Traversing build-only rule for pkg-config.'
    echo 'Traversing build-only rule for makedepend.'
    brew install openssl

Use `make` or `make -f graph_file` to really install things.

Newbrew's output represents the entire dependency graph.  This way
customized packages are always installed before anything that might
need them.

Build-only dependencies that are not mentioned in the configuration
are included as .PHONY targets with no-op build steps.  This completes
the dependency graph but lets homebrew decide whether they must be
installed.

    % echo 'openssl:' | ./newbrew
    Discovering dependencies for package 'openssl'...
    Discovering build dependencies for package 'openssl'...
    Discovering build dependencies for package 'makedepend'...
    Discovering build dependencies for package 'pkg-config'...
    Writing Makefile to standard output.
    all: \
            /usr/local/Cellar/makedepend \
            /usr/local/Cellar/openssl \
            /usr/local/Cellar/pkg-config

    .PHONY: /usr/local/Cellar/makedepend
    /usr/local/Cellar/makedepend: \
                    /usr/local/Cellar/pkg-config
            @echo 'Traversing build-only rule for makedepend.'

    /usr/local/Cellar/openssl: \
                    /usr/local/Cellar/makedepend
            brew install openssl

    .PHONY: /usr/local/Cellar/pkg-config
    /usr/local/Cellar/pkg-config:
            @echo 'Traversing build-only rule for pkg-config.'

Newbrew is provided under the two-clause BSD license, which is
included in the source for convenience.
