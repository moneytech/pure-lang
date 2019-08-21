<a name="doc-install"></a>

Installing Pure (and LLVM)
==========================

Version 0.66, March 06, 2017

| Albert Graef &lt;<aggraef@gmail.com>&gt;
| Eddie Rucker &lt;<erucker@bmc.edu>&gt;

These instructions explain how to compile and install LLVM (which is the
compiler backend required by Pure) and the Pure interpreter itself. The
instructions are somewhat biased towards Linux and other Unix-like systems;
the [System Notes](#system-notes) section at the end of this file details the
tweaks necessary to make Pure compile and run on various other platforms. More
information about installing LLVM and the required LLVM source packages can be
found at <http://llvm.org>.

Pure is known to work on Linux, FreeBSD, NetBSD, Mac OS X and MS Windows, and
should compile (with the usual amount of tweaking) on all recent
UNIX/POSIX-based platforms. Pure should compile out of the box with reasonably
recent versions of either gcc or clang. You'll also need a Bourne-compatible
shell and GNU make, which are also readily available on most platforms. For
Windows compilation we support the Mingw version of gcc along with the MSYS
environment.

A binary package in msi format is provided for Windows users in the download
area at <http://purelang.bitbucket.org>. Information about ports and packages
for other (UNIX-like) systems is provided at the same location.

Quick Summary
-------------

Here is the executive summary for the impatient. This assumes that you're
using LLVM 3.4 and Pure 0.66, please substitute your actual version numbers in
the commands given below as needed.

------------------------------------------------------------------------------

> **Note:** If you're reading this documentation online, then the Pure version
> described here most likely is still under development, in which case you can
> either grab the latest available release, or install from the development
> sources instead (see [Installing From Development
> Sources](#installing-from-development-sources) below).

------------------------------------------------------------------------------

Prerequisites: gcc, GNU make, flex/bison (development sources only), libltdl,
libgmp and libmpfr (including header files for development), wget (for
downloading and installing the online documentation), GNU emacs (if you want
to use Emacs Pure mode). These should all be available as binary packages on
most systems.

You'll also need LLVM and, of course, Pure. The LLVM 3.4 and Pure 0.66
tarballs are available here:

| <https://bitbucket.org/purelang/pure-lang/downloads/pure-0.66.tar.gz>
| <http://llvm.org/releases/3.4/llvm-3.4.src.tar.gz>
| <http://llvm.org/releases/3.4/clang-3.4.src.tar.gz>
| <http://llvm.org/releases/3.4/dragonegg-3.4.src.tar.gz>

------------------------------------------------------------------------------

> **Note:** The present Pure version requires an LLVM version that still
> supports the "old" (pre-MCJIT) just-in-time compiler back-end (JIT). This
> means that the latest *supported* LLVM release is **3.5.2**, which was
> released in April 2015. Earlier versions in the 3.x series should still work
> fine as well, if you can make them compile on your system. The following
> instructions assume that you use the very stable LLVM 3.4 release, otherwise
> you'll have to adjust the commands in the instructions accordingly.

------------------------------------------------------------------------------

Installing LLVM and clang (the latter is optional but recommended):

    $ tar xfvz llvm-3.4.src.tar.gz
    $ tar xfvz clang-3.4.src.tar.gz && mv clang-3.4 llvm-3.4/tools/clang
    $ cd llvm-3.4
    $ ./configure --enable-shared --enable-optimized --enable-targets=host-only
    $ make && sudo make install

Be patient, this takes a while...

You may want to leave out `--enable-shared` to install LLVM as static
libraries only, and `--enable-targets=host-only` if you want to enable cross
compilation for all supported targets in LLVM. (With some older LLVM versions
you may also have to add `--disable-assertions --disable-expensive-checks` to
disable stuff that makes LLVM very slow and/or breaks it on some systems.)
Also, if you're still running gcc 4.5 or 4.6, you may want to install the LLVM
"DragonEgg" plugin for gcc, which provides LLVM bitcode compilation for
languages other than C/C++. Please check the [dragonegg](#dragonegg) section
below for details.

Installing Pure:

    $ tar xfvz pure-0.66.tar.gz
    $ cd pure-0.66
    $ ./configure --enable-release
    $ make && sudo make install

Depending on your system you may have to run a utility to announce the shared
LLVM and Pure libraries to the dynamic loader. E.g., on Linux:

    $ sudo /sbin/ldconfig

It is also recommended that you run the following to make sure that the Pure
interpreter works correctly on your platform (see step 5 below for details):

    $ make check

The following is optional, but if you want to read the online documentation in
the interpreter or in [Emacs Pure mode](#emacs-pure-mode), you'll have to
download and install the documentation files:

    $ sudo make install-docs

This needs the `wget` program. You can also download and install the pure-docs
tarball manually in the usual way, e.g.:

    $ tar xfvz pure-docs-0.66.tar.gz
    $ cd pure-docs-0.66
    $ sudo make install

That's it, Pure should be ready to go now:

    $ pure

Uninstalling:

    $ cd pure-0.66
    $ sudo make uninstall
    $ cd ../llvm-3.4
    $ sudo make uninstall

Please see below for much more detailed installation instructions.

Basic Installation
------------------

The basic installation process is as follows. Note that steps 1-3 are only
required once. Steps 2-3 can be avoided if binary LLVM packages are available
for your system (but see the caveats about broken LLVM packages on some
systems below). Additional instructions for compiling Pure from the latest
repository sources can be found in the [Installing From Development
Sources](#installing-from-development-sources) section below. Moreover, you
can refer to the [Other Build And Installation
Options](#other-build-and-installation-options) section below for details
about various options available when building and installing Pure.

**Step 1.** Make sure you have all the necessary dependencies installed
(`-dev` denotes corresponding development packages):

-   GNU make, GNU C/C++ and the corresponding libraries;
-   the GNU multiprecision library (`libgmp`, `-dev`) or some compatible
    replacement (see comments below);
-   the GNU multiprecision floating point library (`libmpfr`, `-dev`);
-   the PCRE library (`libpcre`, `-dev`), if you want Perl-compatible regex
    support (see below);
-   GNU readline (`libreadline`, `-dev`) or some compatible replacement (only
    needed if you want command line editing support in the interpreter; see
    comments below).
-   GNU Emacs (if you want to use the Emacs Pure mode).
-   GNU TeXmacs (if you want to use the TeXmacs Pure plugin).

In addition, the following will be required to compile the development version
(see the [Installing From Development
Sources](#installing-from-development-sources) section below):

-   autoconf and automake;
-   flex and bison;
-   git (needed to fetch the development sources).

The following may be required to build some LLVM versions:

-   GNU ltdl library (`libltdl`, `-dev`).

All dependencies are available as free software. Here are some links if you
need or want to install the dependencies from source:

-   Autoconf: <http://www.gnu.org/software/autoconf>
-   Automake: <http://www.gnu.org/software/automake>
-   GNU C/C++: <http://gcc.gnu.org>
-   GNU make: <http://www.gnu.org/software/make>
-   Flex: <http://flex.sourceforge.net>
-   Bison: <http://www.gnu.org/software/bison>
-   GNU Emacs: <http://www.gnu.org/software/emacs>
-   GNU TeXmacs: <http://savannah.gnu.org/projects/texmacs>
-   GNU ltdl (part of the libtool software):
    <http://www.gnu.org/software/libtool>

Git can be obtained from <https://git-scm.com/>. A number of graphical
frontends can be found there as well. Windows users may also want to check
TortoiseGit, see <https://tortoisegit.org/>.

The GNU multiprecision library or some compatible replacement is required for
Pure's bigint support. Instead of GMP it's also possible to use MPIR. You can
find these here:

-   GMP: <http://www.gnu.org/software/gmp>
-   MPIR: <http://www.mpir.org>

If you have both GMP and MPIR installed, you can specify `--with-mpir` when
configuring Pure to indicate that Pure should be linked against MPIR. Note
that using this option might cause issues with some Pure modules which
explicitly link against GMP. If you run into any such problems then you should
build MPIR with the `--enable-gmpcompat` configure option so that it becomes a
drop-in replacement for GMP (in this case the `--with-mpir` option isn't
needed when configuring Pure).

In addition, Pure 0.48 and later also require the GNU multiprecision floating
point library:

-   MPFR: <http://www.mpfr.org>

(Pure doesn't really have built-in support for MPFR numbers, this is provided
through a separate pure-mpfr addon module instead, please check the Pure
website for details. However, there is now some support for printing both GMP
and MPFR numbers in the `printf` and `scanf` functions of the
[system](#module-system) module, for which the MPFR library is needed.)

To make interactive command line editing work in the interpreter, you'll also
need GNU readline or some compatible replacement such as BSD editline/libedit:

-   GNU readline: <http://cnswww.cns.cwru.edu/php/chet/readline/rltop.html>
-   BSD editline/libedit: <http://www.thrysoee.dk/editline>

We recommend GNU readline because it's easier to use and has full UTF-8
support, but in some situations BSD editline/libedit may be preferable for
license reasons or because it's what the operating system provides. Pure's
configuration script automatically detects the presence of both packages and
also lets you disable readline and/or editline support using the
`--without-readline` and `--without-editline` options.

Pure normally uses the system's POSIX regex functions for its regular
expression support ([regex](#module-regex) module). Instead, you can also
build it with support for Perl-style regular expressions. This is done by
linking in the POSIX compatibility layer of the PCRE library:

-   PCRE: <http://www.pcre.org/>

Please note that the PCRE support is still experimental and currently disabled
by default. Configuring with `--with-pcre` enables it. Also note that at
present this is only available as a compile-time switch; to change the regex
library used by the Pure runtime you need to recompile the interpreter.

**Step 2.** Get and unpack the LLVM sources.

You can find these at <http://llvm.org/releases/download.html>.

You only need the LLVM tarball which contains the LLVM library as well as most
of the LLVM toolchain. LLVM 3.5.2 is the latest *supported* release at the
time of this writing. LLVM versions 2.5 thru 3.5 have all been tested and are
known to work with Pure. We really recommend using LLVM 3.0 or later, however,
because LLVM has improved considerably in recent releases. (Support for older
versions may be dropped in the future.)

At this point we also recommend getting an LLVM-capable C/C++ compiler. This
is completely optional, but you'll need it to take advantage of the new
bitcode loader in Pure 0.44 and later. The easiest way to go is to take the
clang tarball which corresponds to your LLVM version, unpack its contents into
the LLVM tools directory and rename the `clang-x.y` directory to just `clang`.
The clang compiler will then be built and installed along with LLVM. On older
systems, you can also use llvm-gcc or dragonegg instead, please see
[Installing an LLVM-capable C/C++
Compiler](#installing-an-llvm-capable-cc-compiler) below for further details.

------------------------------------------------------------------------------

> **Note:** Some (older) Linux and BSD distributions provide LLVM packages and
> ports which are compiled with wrong configure options and are thus broken.
> If the Pure interpreter segfaults on startup or fails its test suite
> (`make check`) then you should check whether there's a newer LLVM package
> available for your system, or compile LLVM yourself.

------------------------------------------------------------------------------

**Step 3.** Configure, build and install LLVM as follows (assuming LLVM 3.4):

    $ cd llvm-3.4
    $ ./configure --enable-shared --enable-optimized --enable-targets=host-only
    $ make
    $ sudo make install

LLVM 2.7 and earlier may also require the flags `--disable-assertions`
`--disable-expensive-checks` to disable some features which make LLVM slow
and/or buggy on some systems. With LLVM 2.8 and later these options aren't
needed any more.

Note that the `--enable-shared` option builds and installs LLVM as a shared
library, which is often preferable if you're running different LLVM-based
tools and compilers on your system. This requires LLVM 2.7 or later and may be
broken on some systems. You can always leave out that option, in which case
LLVM will be linked statically into the Pure runtime library. (You can also
force Pure to be linked statically against LLVM even if you have the shared
LLVM library installed, by configuring Pure with the `--with-static-llvm`
flag. This may be useful if you plan to deploy the Pure runtime library on
systems which don't have LLVM installed.)

Also note that the configure flags are for an optimized (non-debug) build and
disable all compilation targets but the one for your system. You might wish to
play with the configure options, but note that some options (especially
`--enable-expensive-checks`) can make LLVM very slow and may even break the
Pure interpreter on some systems.

**Step 4.** Get and unpack the Pure sources.

These can be downloaded from <http://purelang.bitbucket.org>. The latest
source tarballs can always be found in the "Downloads" section.

**Step 5.** Configure, build and install Pure as follows:

    $ cd pure-0.66
    $ ./configure --enable-release
    $ make
    $ sudo make install

The `--enable-release` option configures Pure for a release build. This is
recommended for maximum performance. If you leave away this option then you'll
get a default build which includes debugging information and extra runtime
checks useful for the Pure maintainers, but also runs considerably slower.

To find out about other build options, you can invoke configure as
`./configure --help`.

The `sudo make install` command installs the pure program, the runtime.h
header file, the runtime library libpure.so, a Pure pkg-config file (pure.pc)
and the library scripts in the appropriate subdirectories of /usr/local; the
installation prefix can be changed with the `--prefix` configure option, see
[Other Build And Installation Options](#other-build-and-installation-options)
for details. (The runtime.h header file is not needed for normal operation,
but can be used to write C/C++ extensions modules or embed Pure in your C/C++
applications.)

In addition, if the presence of GNU Emacs was detected at configure time, then
by default pure-mode.el and pure-mode.elc will be installed in the Emacs
site-lisp directory. make tries to guess the proper location of the site-lisp
directory, but if it guesses wrong or if you want to install in some custom
location then you can also set the `elispdir` make variable accordingly. If
you prefer, you can also disable the automatic installation of the elisp files
by running configure with `./configure --without-elisp`. (In that case, it's
still possible to install the elisp files manually with
`make install-el install-elc`.)

Similarly, if you have [GNU
TeXmacs](http://savannah.gnu.org/projects/texmacs/) on your system and
configure can locate it, the corresponding Pure plugin will be installed into
the TeXmacs plugins directory. make tries to guess the proper location of the
TeXmacs plugins directory, but if it guesses wrong or if you want to install
in some custom location then you can also set the `tmdir` make variable
accordingly. If you prefer, you can also disable the automatic installation of
the TeXmacs plugin by running configure with `./configure --without-texmacs`.
(In that case, it's still possible to install the plugin manually with
`make install-tm`.)

On some systems you have to tell the dynamic linker to update its cache so
that it finds the Pure runtime library. E.g., on Linux this is done as
follows:

    $ sudo /sbin/ldconfig

After the build is complete, you can (and should) also run a few tests to
check that Pure is working correctly on your computer:

    $ make check

(This can be done before actually installing Pure, but make sure that you
first run `ldconfig` or similar if you installed LLVM as a shared library,
otherwise `make check` may fail simply because the LLVM library isn't found.)

If all is well, all tests should pass. If not, the test directory will contain
some `*.diff` files containing further information about the failed tests. In
that case please zip up the entire test directory and mail it to the author,
post it on the Pure mailing list, or enter a bug report at
<https://bitbucket.org/purelang/pure-lang/issues>. Also please include precise
information about your platform (operating system and cpu architecture) and
the Pure and LLVM versions and/or source revision numbers you're running.

Note that `make check` executes the run-tests script which is generated at
configure time. If necessary, you can also run individual tests by running
run-tests directly (e.g., `./run-tests test/test020.pure test/test047.pure`)
or rerun only the tests that failed on the previous invocation
(`./run-tests -f` or, equivalently, `make recheck`).

Also note that MSYS 1.0.11 or later (or at least the diffutils package from
that version) is required to make `make check` work on Windows. Also, under MS
Windows this step is expected to fail on some math tests in test020.pure; this
is nothing to worry about, it just indicates that some math routines in
Microsoft's C library aren't fully POSIX-compatible. The same applies to some
BSD systems.

If Pure appears to be broken on your system (`make check` reports a lot of
failures), it's often because of a miscompiled LLVM. Please review the
instructions under step 3, and check the [System Notes](#system-notes) section
to see whether your platform is known to have issues and which workarounds may
be needed. If all that doesn't help then you might be running into LLVM bugs
and limitations on not-so-well supported platforms; in that case please also
report the results of `make check` as described above, so that we can try to
figure out what is going on and whether there's a fix or workaround for the
problem.

------------------------------------------------------------------------------

> **Note:** If you have one of the LLVM C/C++ compilers installed (see
> [Installing an LLVM-capable C/C++
> Compiler](#installing-an-llvm-capable-cc-compiler)), you can use those to
> compile Pure by passing the appropriate compiler names on the configure
> line:
>
>     $ ./configure --enable-release CC=clang CXX=clang++
>
> Or, when using llvm-gcc:
>
>     $ ./configure --enable-release CC=llvm-gcc CXX=llvm-g++
>
> llvm-gcc 4.2 and clang 2.8 or later should build Pure cleanly and pass all
> checks.

------------------------------------------------------------------------------

**Step 6.** Download and install the online documentation as follows:

    $ sudo make install-docs

This isn't necessary to run the interpreter, but highly recommended, as it
gives you a complete set of manuals in html format which covers the Pure
language and interpreter, the standard library, and all addon modules
available from the Pure website. You can read these manuals with the `help`
command in the interpreter. You also need to have a html browser installed to
make this work. By default, the interpreter assumes w3m (a text-based
browser), you can change this by setting the `BROWSER` or the `PURE_HELP`
variable accordingly.

The `install-docs` target requires a working Internet connection and the wget
command. Instead, you can also download the pure-docs-0.66.tar.gz tarball
manually and then install the documentation from the downloaded tarball in the
usual way:

    $ tar xfvz pure-docs-0.66.tar.gz
    $ cd pure-docs-0.66
    $ sudo make install

As a bonus, downloading the package manually also gives you the documentation
in pdf format (900+ pages), so that you can print it if you like. In addition,
as of version 0.56 the tarball also contains the documentation in TeXmacs
format so that you can read it inside TeXmacs (see [TeXmacs
Plugin](#texmacs-plugin) below). After unpacking the tarball and installing
the html documentation, you can install the TeXmacs-formatted documentation as
follows:

    $ sudo make install-tm

**Step 7.** The Pure interpreter should be ready to go now.

Run Pure interactively as:

    $ pure

     __ \  |   |  __| _ \    Pure 0.66 (x86_64-unknown-linux-gnu)
     |   | |   | |    __/    Copyright (c) 2008-2017 by Albert Graef
     .__/ \__,_|_|  \___|    (Type 'help' for help, 'help copying'
    _|                       for license information.)

    Loaded prelude from /usr/lib/pure/prelude.pure.

Check that it works:

    > 6*7;
    42

Read the online documentation:

    > help

Exit the interpreter (you can also just type the end-of-file character at the
beginning of a line, i.e., `Ctrl-D` on Unix):

    > quit

You can also run the interpreter from GNU Emacs and GNU TeXmacs (see below),
and for Windows there is a nice GUI application named "PurePad" which makes it
easy to edit and run your Pure scripts.

Emacs Pure Mode
---------------

This step is optional, but if you're friends with Emacs then you should
definitely give Pure mode a try. This is an Emacs programming mode which turns
Emacs into an advanced IDE to edit and run Pure programs. If Emacs was
detected by configure then after running `make` and `sudo make install` the
required elisp files should already be installed in the Emacs site-lisp
directory (unless you specifically disabled this with the `--without-elisp`
configure option).

Note: make tries to guess the Emacs installation prefix. If it gets this
wrong, you can also set the make variable `elispdir` to point to your
site-lisp directory. (In fact, you can specify any directory on Emacs'
loadpath for `elispdir`.)

Before you can use Pure mode, you still have to add some stuff to your .emacs
file to load the mode at startup. A minimal setup looks like this:

``` {.sourceCode .scheme}
(require 'pure-mode)
(setq auto-mode-alist
      (cons '("\\.pure\\(rc\\)?$" . pure-mode) auto-mode-alist))
(add-hook 'pure-mode-hook 'turn-on-font-lock)
(add-hook 'pure-eval-mode-hook 'turn-on-font-lock)
```

This loads Pure mode, associates the .pure and .purerc filename extensions
with it, and enables syntax highlighting.

Other useful options are described at the beginning of the pure-mode.el file.
In particular, we recommend installing emacs-w3m and enabling it as follows in
your .emacs file, so that you can read the online documentation in Emacs:

``` {.sourceCode .scheme}
(require 'w3m-load)
```

Also, you can enable code folding by adding this to your .emacs:

``` {.sourceCode .scheme}
(require 'hideshow)
(add-hook 'pure-mode-hook 'hs-minor-mode)
```

These lines should come before the loading of Pure mode in your .emacs, so
that Pure mode can adjust accordingly.

Once Emacs has been configured to load Pure mode, you can just run it with a
Pure file to check that it works, e.g.:

    $ emacs examples/hello.pure

The online help about Pure mode can be read with `C-h m`. The Pure
documentation can be accessed in Pure mode with `C-c h`.

TeXmacs Plugin
--------------

Pure 0.56 has full support for running Pure as a session in [GNU
TeXmacs](http://savannah.gnu.org/projects/texmacs/). This is triggered by the
[`--texmacs`](#cmdoption-pure--texmacs) option of the interpreter. If TeXmacs
was detected by configure then after running `make` and `sudo make install`
the required plugin files should already be installed in the system-wide
TeXmacs plugins directory and you should be able to find Pure in TeXmacs'
Insert / Session and Document / Scripts menus. TeXmacs help is also included;
after installation this should be available with the Help / Plug-ins / Pure
menu option. Or you can just go and read the TeXmacs documents (.tm files) in
texmacs/plugins/pure/doc included in the distribution.

Note: make tries to guess the TeXmacs installation prefix. If it gets this
wrong, you can also set the make variable `tmdir` to point to your TeXmacs
plugins directory. By default, the plugin files will be installed in the
plugins/pure subdirectory of your system-wide TeXmacs directory, usually
/usr/local/share/TeXmacs or similar. Removing the plugins/pure directory is
sufficient to uninstall the plugin; you can also do this with
`sudo make uninstall-tm`.

It's also possible to disable the automatic installation by invoking configure
with the `--without-texmacs` option. To do a manual installation, it should be
sufficient to drop the texmacs/plugins/pure directory in the distribution into
your personal TeXmacs plugins folder, usually \~/.TeXmacs/plugins. You can
also do this with `make install-tm tmdir=~/.TeXmacs` (in this case, uninstall
with `make uninstall-tm tmdir=~/.TeXmacs`). This has the advantage that it
doesn't require root access and you can easily edit the plugin files under
\~/.TeXmacs/plugins/pure/progs afterwards to tailor them to your needs.

The distributed plugin has support for reading the Pure online help in TeXmacs
format. See Step 6 under [Basic Installation](#basic-installation) above for
instructions on how you can obtain the necessary TeXmacs files and install
them in the Pure library directory along with the html documentation. (The
TeXmacs-formatted documentation needs a little style file named puredoc.ts
which is included in the distribution and will be installed when doing
`make install` or `make install-tm`. You can also just drop the file into your
\~/.TeXmacs/packages folder to make TeXmacs find it.)

If the distributed plugin doesn't work for you, as a fallback option you can
try the following minimal setup instead. This lacks all the bells and whistles
of the distributed plugin, but should be sufficient to run a basic Pure
session in TeXmacs, and should hopefully work with any TeXmacs version which
has plugin support at all.

``` {.sourceCode .scheme}
(plugin-configure pure
  (:require (url-exists-in-path? "pure"))
  (:launch "pure -i --texmacs")
  (:session "Pure"))
```

This Scheme file should go into \~/.TeXmacs/plugins/pure/progs/init-pure.scm.
Note that *both* the `-i` and `--texmacs` options are required in the launch
command to make this work (you might also want to add the `-q` option to
suppress the signon message of the interpreter).

Installing an LLVM-capable C/C++ Compiler
-----------------------------------------

As already mentioned above, we suggest that you also install a C/C++ compiler
with an LLVM backend. clang, llvm-gcc as well as the dragonegg gcc plugin are
all fully supported by Pure. (Nowadays, the LLVM project's main focus is on
clang, however; llvm-gcc and dragonegg are not supported on recent systems any
more.) Pure can be used without this, but then you'll miss out on the LLVM
bitcode loader and C/C++ inlining facilities in Pure 0.44 and later. (Note
that you can always install clang, llvm-gcc and/or dragonegg at a later time
to enable these features.)

### clang

With LLVM 2.8 and later, we recommend installing clang, the new LLVM-based
C/C++ compiler (<http://clang.llvm.org/>). It's much easier to build, runs
faster and has better diagnostics than gcc. Also, as of Pure 0.55 it is the
default for compiling inline C/C++ code, so it's the easiest way to go if you
want to use that feature.

If you haven't built clang along with LLVM yet, you can now just drop the
contents of the clang tarball into the `llvm/tools` directory, renaming the
resulting `clang-x.y` directory to just `clang`. Then build and install clang
as follows:

    $ cd llvm-3.4/tools/clang
    $ make
    $ sudo make install

### llvm-gcc

------------------------------------------------------------------------------

> **Note:** LEGACY ALERT: This section applies to LLVM versions up to 2.9.
> With LLVM 3.0 or later, llvm-gcc is not supported any more and you should
> use [clang](#clang) instead.

------------------------------------------------------------------------------

If available, llvm-gcc can be installed either as an alternative or in
addition to clang. The main advantage of llvm-gcc over clang is that it has
additional language frontends (Ada and Fortran).

Installing llvm-gcc from source actually isn't all that difficult, if a bit
time-consuming. Assuming that you have unpacked both the LLVM and the llvm-gcc
sources in the same directory, you can build and install llvm-gcc as follows:

    $ cd llvm-gcc-4.2-2.9.source
    $ mkdir obj
    $ cd obj
    $ ../configure --program-prefix=llvm- --enable-llvm=$PWD/../../llvm-2.9 --enable-languages=c,c++
    $ make
    $ sudo make install

(You might wish to add `fortran` to `--enable-languages` if you also want to
build the Fortran compiler, and, likewise, `ada` for the Ada compiler.)

Be patient, this takes a while.

Having installed llvm-gcc, you can add something like the following lines to
your shell startup files, so that Pure uses it for inlined C/C++/Fortran code:

    export PURE_CC=llvm-gcc
    export PURE_CXX=llvm-g++
    export PURE_FC=llvm-gfortran

### dragonegg

------------------------------------------------------------------------------

> **Note:** LEGACY ALERT: This section applies to LLVM versions up to 3.3 and
> gcc 4.5 or 4.6. If you're running a newer LLVM and/or gcc version, you
> should use [clang](#clang) instead.

------------------------------------------------------------------------------

Instead of llvm-gcc it's also possible to use "DragonEgg"
(<http://dragonegg.llvm.org/>). This is provided in the form of a gcc plugin
which, if you have one of the supported gcc versions, readily plugs into your
existing system compiler.

To install DragonEgg, make sure that you have the mpc and gcc plugin
development files installed (packages `mpc-dev` and `gcc-plugin-dev` on
Ubuntu). Then, after unpacking the dragonegg source tarball or downloading the
svn sources, install dragonegg as follows:

    $ make
    $ sudo cp dragonegg.so `gcc -print-file-name=plugin`

Finally, add something like the following lines to your shell startup files,
so that Pure uses gcc+dragonegg for all inlined C/C++/Fortran code:

    export PURE_CC="gcc -fplugin=dragonegg"
    export PURE_CXX="g++ -fplugin=dragonegg"
    export PURE_FC="gfortran -fplugin=dragonegg"

(Please also check the README file included in the dragonegg package for
further installation and usage instructions. Also, examples/bitcode/Makefile
in the Pure distribution demonstrates how to use gcc+dragonegg as an external
compiler to generate LLVM bitcode from the command line.)

Installing From Development Sources
-----------------------------------

The latest development version of Pure is available in its Git source code
repository. You can browse the repository at:

<https://bitbucket.org/purelang/pure-lang>

(You'll notice that the repository also contains various addon modules. See
the pure subdirectory for the latest sources of the Pure interpreter itself.)

Note that if you're going with the development sources, you'll also need
fairly recent versions of autoconf, flex and bison (autoconf 2.63, flex 2.5.31
and bison 2.3 or later should be ok).

To compile from the development sources, replace steps 4 and 5 above with:

**Step 4'.** Fetch the latest sources from the repository:

    $ git clone https://bitbucket.org/purelang/pure-lang

This clones the repository and puts it into the pure-lang subdirectory in the
current directory. This step needs to be done only once; once you've cloned
the repository, you can update it to the latest revision at any time by
running `git pull`.

**Step 5'.** Configure, build and install Pure.

This is pretty much the same as with the distribution tarball, except that you
need to run `autoreconf` once to generate the configure script which isn't
included in the source repository.

    $ cd pure-lang/pure
    $ autoreconf
    $ ./configure --enable-release
    $ make
    $ sudo make install

(Don't forget to also run `make check` to make sure that the interpreter is in
good working condition.)

**Step 6'.** In addition, you can also build and install a recent snapshot of
the documentation from the repository.

This can be a bit tricky to set up, please check the FAQ wiki page on the Pure
website for details.

Alternatively, a ready-made recent snapshot of the documentation in html and
pdf formats is also available in its own repository, which can be cloned as
follows:

    $ git clone https://bitbucket.org/puredocs/puredocs.bitbucket.org.git

Other Build and Installation Options
------------------------------------

The Pure configure script takes a few options which enable you to change the
installation path and control a number of other build options. Moreover, there
are some environment variables which also affect compilation and installation.

Use `./configure --help` to print a summary of the provided options.

### Installation Path

By default, the pure program, the runtime.h header file, the runtime library,
the pure.pc file and the library scripts are installed in /usr/local/bin,
/usr/local/include/pure, /usr/local/lib, /usr/local/lib/pkg-config and
/usr/local/lib/pure, respectively. This can be changed by specifying the
desired installation prefix with the `--prefix` option, e.g.:

    $ ./configure --enable-release --prefix=/usr

In addition, the `DESTDIR` variable enables package maintainers to install
Pure into a special "staging" directory, so that installed files can be
packaged more easily. If set at installation time, `DESTDIR` will be used as
an additional prefix to all installation paths. For instance, the following
command will put all installed files into the tmp-root subdirectory of the
current directory:

    $ make install DESTDIR=tmp-root

Note that if you install Pure into a non-standard location, you may have to
set `LD_LIBRARY_PATH` or a similar variable so that the dynamic linker finds
the Pure runtime library, libpure.so. Also, when compiling and linking addon
modules you might have to set `C_INCLUDE_PATH` and `LIBRARY_PATH` (or similar)
so that the header and library of the runtime library is found. (This will
become unnecessary once all addon modules have been converted to use
pkg-config, see below, but this isn't the case right now.) On some systems
this is even necessary with the default prefix, because /usr/local is not
always in the default search paths.

As of Pure 0.47, Pure also installs a pkg-config file which may be queried by
module Makefiles to determine how to build a module and link it against the
Pure runtime library; see [Pkg-config Support](#pkg-config-support) below.
This file will usually be installed into `$(prefix)/lib/pkg-config`. Again, if
you use a non-standard installation prefix, you will have to tell pkg-config
about the location of the file by adjusting the `PKG_CONFIG_PATH` environment
variable accordingly, see pkg-config(1) for details.

### Tool Prefix and LLVM Version

On some systems the LLVM toolchain may be located in special directories not
on the `PATH`, so that different LLVM installations can coexist on the same
system. This is often the case, e.g., if LLVM was installed from a binary
package.

To deal with this situation, the configure script distributed with Pure 0.55
and later allows you to specify the directory with the LLVM toolchain using
the `--with-tool-prefix` configure option. E.g.:

    $ ./configure --with-tool-prefix=/usr/lib/llvm-3.4/bin

This is also the directory where configure will first look for the
`llvm-config` script, so that the proper LLVM version is selected for
compilation.

You can also specify the desired LLVM version with the `--with-llvm-version`
option. This causes configure to look for the `llvm-config-x.y` script on the
`PATH`, where `x.y` is the specified version number. If this option isn't
specified, the default is to look for the `llvm-config` script on the `PATH`
(this should always work if you installed LLVM from source).

If none of these yield a usable `llvm-config` script, configure will try to
locate an `llvm-config-x.y` script by iterating through some recent LLVM
releases, preferring the latest version if found. If this fails, too,
configure gives up and prints an error message indicating that it couldn't
locate a suitable LLVM installation. This is a fatal error, so if you see this
then you'll either have to install LLVM yourself, or try to locate a suitable
LLVM installation and tell configure about it, using the options explained
above.

### Versioned Installations

Pure fully supports parallel installations of different versions of the
interpreter. To enable this you have to specify `--enable-versioned` when
running configure:

    $ ./configure --enable-release --enable-versioned

When this option is enabled, bin/pure, include/pure, lib/pure,
lib/pkg-config/pure.pc and man/man1/pure.1 are actually symbolic links to the
current version (bin/pure-x.y, include/pure-x.y etc., where x.y is the version
number). If you install a new version of the interpreter, the old version
remains available as pure-x.y.

Note that versioned and unversioned installations don't mix very well, it's
either one or the other. If you already have an unversioned install of Pure,
you must first remove it before switching to the versioned scheme.

It *is* possible, however, to have versioned and unversioned installations
under different installation prefixes. For instance, having an unversioned
install under /usr and several versioned installations under /usr/local is ok.

### Separate Build Directory

It is possible to build Pure in a separate directory, in order to keep your
source tree tidy and clean, or to build multiple versions of the interpreter
with different compilation flags from the same source tree.

To these ends, just cd to the build directory and run configure and make
there, e.g. (this assumes that you start from the source directory):

    $ mkdir BUILD
    $ cd BUILD
    $ ../configure --enable-release
    $ make

### Compiler and Linker Options

There are a number of environment variables you can set on the configure
command line if you need special compiler or linker options:

-   `CPPFLAGS`: preprocessor options (`-I`, `-D`, etc.)
-   `CXXFLAGS`: C++ compilation options (`-g`, `-O`, etc.)
-   `CFLAGS`: C compilation options (`-g`, `-O`, etc.)
-   `LDFLAGS`: linker flags (`-s`, `-L`, etc.)
-   `LIBS`: additional objects and libraries (`-lfoo`, `bar.o`, etc.)

(The `CFLAGS` variable is only used to build the pure\_main.o module which is
linked into batch-compiled executables, see "Batch Compilation" in the manual
for details.)

For instance, the following configure command changes the default compilation
options to `-g` and adds /opt/include and /opt/lib to the include and library
search paths, respectively:

    $ ./configure CPPFLAGS=-I/opt/include CXXFLAGS=-g LDFLAGS=-L/opt/lib

More details on the build and installation process and other available targets
and options can be found in the Makefile.

### Predefined Build Types

For convenience, configure provides some options to set up `CPPFLAGS` and
`CXXFLAGS` for various build types. Please note that most of these options
assume gcc right now, so if you use another compiler you'll probably have to
set up compilation flags manually by using the variables described in the
previous section instead.

The default build includes debugging information and additional runtime checks
which provide diagnostics useful for maintainers if anything is wrong with the
interpreter. It is also noticeably slower than the "release" build. If you
want to enjoy maximum performance, you should configure Pure for a release
build as follows:

    $ ./configure --enable-release

This disables all runtime checks and debugging information in the interpreter,
and uses a higher optimization level (`-O3`), making the interpreter go
substantially faster on most systems.

To get smaller executables with either the default or the release build, add
`LDFLAGS=-s` to the configure command (gcc only, other compilers may provide a
similar flag or a separate command to strip compiled executables and
libraries).

You can also do a "debug" build as follows:

    $ ./configure --enable-debug

This is like the default build, but disables all optimizations, so compilation
is faster but the compiled interpreter is *much* slower than even the default
build. Hence this build is only recommended for debugging purposes.

You can combine all build types with the `--enable-warnings` option to enable
compiler warnings (`-Wall`):

    $ ./configure --enable-release --enable-warnings

This option is useful to check the interpreter sources for questionable
constructs which might actually be bugs. However, for some older gcc versions
it spits out lots of bogus warnings, so it is not enabled by default.

In addition, there is an option to build a "monolithic" interpreter which is
linked statically instead of producing a separate runtime library:

    $ ./configure --enable-release --disable-shared

We strongly discourage from using this option, since it drastically increases
the size of the executable and thereby the memory footprint of the interpreter
if several interpreter processes are running simultaneously. It also makes it
impossible to use batch compilation and addon modules which require the
runtime library. We only provide this as a workaround for older LLVM versions
which cannot be linked into shared libraries on some systems.

In general, the build options can be combined freely with the variables
described in the previous section, except that `--enable-release` and
`--enable-debug` will always override the debugging (`-g`) and optimization
(`-O`) options in `CFLAGS` and `CXXFLAGS`. Other options will be preserved.
For instance, the following configures for a release build, but sets the
warning flags manually and enables C++0x support in gcc:

    $ ./configure --enable-release CFLAGS="-Wall" CXXFLAGS="-std=c++0x -Wall"

If you mix build types and manual compilation flags in this way then it's
always a good idea to check the resulting compilation options printed out at
the end of the `configure` run. If `configure` seems to get things wrong then
you'll have to set up all required flags manually instead.

### Running Pure From The Source Directory

After your build is done, you should also run `make check` to verify that your
Pure interpreter works correctly. This can be done without installing the
software. In fact, there's no need to install the interpreter at all if you
just want to take it for a test drive, you can simply run it from the source
directory, if you set up the following environment variables (this assumes
that you built Pure in the source directory; when using a separate build
directory, you'll have to change the paths accordingly):

`LD_LIBRARY_PATH=.`

:   This is required on Linux systems so that libpure.so is found. Other
    systems may require an analogous setting, or none at all.

`PURELIB=./lib`

:   This is required on all systems so that the interpreter finds the prelude
    and other library scripts.

After that you should be able to run the Pure interpreter from the source
directory, by typing `./pure`.

### Other Targets

The Makefile supports the usual `clean` and `distclean` targets, and
`realclean` will remove all files created by the maintainer, including test
logs and C++ source files generated from Flex and Bison grammars. (Only use
the latter if you know what you are doing, since it will remove files which
require special tools to be regenerated.)

Maintainers can roll distribution tarballs with `make dist` and
`make distcheck` (the latter is like `make dist`, but also does a test build
and installation to verify that your tarball contains all needed bits and
pieces).

Last but not least, if you modify configure.ac for some reason then you can
regenerate the configure script and config.h.in with `make config`. This needs
autoconf, of course. (The distribution was prepared using autoconf 2.69.)

### Pkg-config Support

Pure 0.47 and later install a pkg-config file (pure.pc) which lets addon
modules query the installed Pure for the information needed to build and
install a module. Besides the usual information provided by pkg-config, such
as `--cflags` and `--libs` (which are set up so that the Pure runtime header
and library will be found), pure.pc also defines a few additional variables
which can be queried with pkg-config's `--variable` option:

-   `DLL`: shared library extension for the host platform
-   `PIC`: position-independent code flag if required on the host platform
-   `shared`: flag used to create shared libraries on the host platform
-   `extraflags`: same as `$shared $PIC`

Together with the `libdir` variable, this provides you with the information
needed to build and install most Pure modules without much ado. As of Pure
0.55, pure.pc also defines the `tool_prefix` variable which gives the LLVM
toolchain prefix specified at configure time, cf. [Tool Prefix and LLVM
Version](#tool-prefix-and-llvm-version).

If you want to use this information, you need to have pkg-config installed,
see <http://pkg-config.freedesktop.org>. This program should be readily
available on most Unix-like platforms, and a Windows version is available as
well. An example illustrating the use of pkg-config can be found in the
examples/hellomod directory in the sources.

System Notes
------------

Pure is known to work on recent Linux, Mac OS X and BSD versions under x86,
x86-64 (AMD/Intel x86, 32 and 64 bit) and ppc (PowerPC), as well as on MS
Windows (AMD/Intel x86, 32 bit). There are a few known quirks for specific
platforms and/or LLVM versions which are discussed below, along with
corresponding workarounds. As a fairly demanding LLVM application, Pure is
also known to shake the bugs out of LLVM, so if you run into any LLVM-related
issues, Pure isn't necessarily the culprit! However, most of the bugs have
been ironed out over the years, so Pure should work fine with any recent LLVM
version (3.4 or later).

### All Platforms

Compiling the default and release versions using gcc with all warnings turned
on (`-Wall`) might give you the warning "dereferencing type-punned pointer
will break strict-aliasing rules" at some point in util.cc with some older gcc
versions. This is harmless and can be ignored.

If your Pure program runs out of stack space, the interpreter may segfault.
While the Pure interpreter does advisory stack checks to avoid that kind of
mishap and generate an orderly exception instead, it has no way of knowing the
actual stack size available to programs on your system. So if you're getting
segfaults due to stack overflows then you'll have to set an appropriate stack
size limit manually with the [`PURE_STACK`](#envvar-PURE_STACK) environment
variable; see the Pure manual for details.

### LLVM 2.5

The LLVM 2.5 JIT is broken on x86-32 if it is built with `--enable-pic`, so
make sure you do *not* use this option when compiling LLVM &lt;=2.5 on 32 bit
systems. On the other hand, building the Pure runtime library (libpure) on
x86-64 systems *requires* that you configure LLVM 2.5 with `--enable-pic` so
that the static LLVM libraries can be linked into the runtime library. With
LLVM 2.6 and later, this option isn't needed anymore.

This LLVM version also has issues on PowerPC. Use LLVM 2.6 or later instead
and check the notes on [PowerPC](#powerpc) below.

Please also note that LLVM 2.5 is the oldest LLVM version that we still
support right now. If you're still running this version (or any of the 2.x
versions) then you should really upgrade. Newer LLVM versions offer
substantial improvements in both compilation time and code quality. Support
for LLVM versions older than 3.0 is likely to be dropped in future Pure
releases.

### LLVM 3.3

Reportedly the Pure batch compiler is broken when using LLVM 3.3 on Mac OS X.
If you need the batch compiler then use either LLVM 3.2 or 3.4+ on OS X
instead, these are known to work on that platform.

### LLVM 3.4+

On Linux, the `llc` program from LLVM 3.4 and 3.5 is known to create native
assembler (.s) code which doesn't mangle assembler symbols as it used to be in
earlier releases. When compiling Pure modules with the batch compiler, this
results in native assembler code which can't be compiled using the system
assemblers. As a remedy, the batch compiler in Pure 0.61 and later now creates
native object (.o) files directly using llc, without going through the native
assembler stage.

### LLVM 3.6+

Pure doesn't work with LLVM versions 3.6 and beyond yet because these don't
include the "old" LLVM JIT any more. So until Pure gets ported to the new
MCJIT, you will have to stick with LLVM 3.5. Fortunately, this version is
still readily available on most platforms.

### PowerPC

(This hasn't been checked in a *long* time, so this information may well be
outdated.) You'll need Pure &gt;= 0.35 and LLVM &gt;= 2.6. Also make sure that
you always configure LLVM with `--disable-expensive-checks` and Pure with
`--disable-fastcc`. With these settings Pure should work fine on ppc (tested
on ppc32 running Fedora Core 11 and 12), but note that tail call optimization
doesn't work on this platform because of LLVM limitations.

### Linux

Linux is the primary development platform for this software, and the sources
should build out of the box on all recent Linux distributions. Packages for
various Linux distributions are also available, please check the Pure website
for details.

### Mac OS X

Pure should build fine on recent OS X versions, and a port by Ryan Schmidt
exists in the MacPorts collection, see <http://www.macports.org/>. If you
install straight from the source, make sure that you use a recent LLVM version
(LLVM 2.7 or later should work fine on all flavors of Intel Macs).

Even if you compile Pure from source, we recommend installing LLVM and the
other dependencies from MacPorts. In that case you'll have to add a few
options to the configure command so that the required header files, libraries
and LLVM tools are found:

    ./configure --enable-release --prefix=/opt/local \
    CPPFLAGS=-I/opt/local/include LDFLAGS=-L/opt/local/lib \
    --with-tool-prefix=/opt/local/libexec/llvm-3.4/bin

This assumes that your MacPorts installation lives under /opt/local and that
you're using LLVM 3.4; otherwise you'll have to adjust the configure options
accordingly.

On really old Macs from the bygone PPC era you'll have to be prepared to deal
with all kinds of issues with compilers, LLVM toolchain etc. If you're still
using one of these, your best bet is to find an older port on MacPorts which
works for your OS X version.

### BSD

FreeBSD now offers a fairly extensive selection of Pure packages in their
distribution.

Compilation from source should also work fine on recent NetBSD and FreeBSD
versions if you use Pure 0.33 or later. Also make sure that you install a
recent port of LLVM which has the `--enable-optimized` flag enabled.

Building Pure requires GNU make, thus you will have to use gmake instead of
make. In addition to gmake, you'll need recent versions of the following
packages: perl5, flex, bison, gmp, mpfr and readline (or editline). Depending
on your system, you might also have to set up some compiler and linker paths.
E.g., the following reportedly does the trick on NetBSD:

    export C_INCLUDE_PATH=/usr/local/include:/usr/pkg/include
    export LIBRARY_PATH=/usr/local/lib:/usr/pkg/lib
    export LD_LIBRARY_PATH=/usr/pkg/lib:/usr/local/lib

### MS Windows

Thanks to Jiri Spitz' perseverance, tireless testing and bug reports, the
sources compile and run fine on Windows, using the Mingw port of the GNU C++
compiler and the MSYS environment from <http://www.mingw.org/>. Just do the
usual `./configure && make && make install`. You'll need LLVM, of course
(which builds with Mingw just fine), and a few additional libraries for which
headers and precompiled binaries are available from the Pure website
(<http://purelang.bitbucket.org>).

However, the easiest way is to just go with the Pure MSI package available on
the Pure website. This includes all required libraries and some shortcuts to
run the Pure interpreter and read online documentation in html help format, as
well as "PurePad", an alternative GUI frontend for editing and running Pure
scripts on Windows. The only limitation is that the binaries in this package
still are for x86 (32 bit) only right now, but these do work fine on all
recent 64 bit flavors of Windows (tested on Windows 7, 8 and 10).