Newbrew automates installing packages on a new homebrew system.

Homebrew is a package manager for Mac OSX.  It occasionally needs to
be nuked from orbit, such as after a major XCode upgrade.  It's
inconvenient to rebuild everything after that.

Newbrew isn't completely automatic.  It must be told which packages to
install, and what custom flags to use (if any).

It reads this configuration from standard input, in the form of
"package:" followed by zero or more "+package" and/or "-package"
flags.  Those represent "--with-package" and "--without-package",
respectively.

For example, this will "brew install libtiff --with-xz":

    libtiff: +xz

There is nascent support for running custom commands after a package
is installed.  For example:

    wireshark: +gtk+ `brew cask install wireshark-chmodbpf`

Newbrew writes a Makefile to standard output containing the steps
needed to install everything as specified.  Progress is written to
standard error.  For example:

    % echo 'libtiff: +xz' | ./newbrew
    Discovering dependencies for package 'libtiff'...
      strange option --c++11 at ./newbrew line 55, <> line 1.
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

    /usr/local/Cellar/xz:
            brew install xz

Newbrew's Makefile represents the entire dependency tree so that
customized packages are always installed before anything that needs
them.

Build-only dependenices that aren't explicitly mentioned in the
newbrew configuration are included as .PHONY Makefile targets.  This
completes the dependency tree but lets homebrew decide whether they
really need to be installed.

Newbrew is provided under the two-clause BSD license, which is
included in the source for convenience.
