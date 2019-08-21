<a name="doc-pure-tk"></a>

pure-tk
=======

<a name="module-tk"></a>

Version 0.5, March 06, 2017

Albert Graef &lt;<aggraef@gmail.com>&gt;

Pure's [Tcl/Tk](http://www.tcl.tk) interface.

Introduction
------------

This module provides a basic interface between Pure and Tcl/Tk. The operations
of this module allow you to execute arbitrary commands in the Tcl interpreter,
set and retrieve variable values in the interpreter, and invoke Pure callbacks
from Tcl/Tk.

A recent version of Tcl/Tk is required (8.0 or later should do). You can get
this from <http://www.tcl.tk>. Both releases in source form and binary
releases for Windows and various Unix systems are provided there.

Some information on how to use this module can be found below. But you'll find
that pure-tk is very easy to use, so you might just want to look at the
programs in the examples folder to pick it up at a glance. A very basic
example can be found in tk\_hello.pure; a slightly more advanced example of a
tiny but complete Tk application is in tk\_examp.pure.

pure-tk also offers special support for Peter G. Baum's
[Gnocl](http://www.gnocl.org/) extension which turns Tcl into a frontend for
[GTK+](http://www.gtk.org/) and [Gnome](http://www.gnome.org/). If you have
Gnocl installed then you can easily create GTK+/Gnome applications, either
from Tcl sources or from [Glade](http://glade.gnome.org/) UI files, using the
provided gnocl.pure module. See the included uiexample.pure and the
accompanying Glade UI file for a simple example. Also, some basic information
on using Gnocl with pure-tk can be found in the [Tips and
Tricks](#tips-and-tricks) section below.

One nice thing about Tcl/Tk is that it provides a bridge to a lot of other
useful libraries. A prominent example is [VTK](http://www.vtk.org/), a
powerful open-source 3D visualization toolkit which comes with full Tcl/Tk
bindings. The examples directory contains a simple example (earth.pure and
earth.tcl) which shows how you can employ these bindings to write cool
animated 3D applications using either Tk or Gnocl as the GUI toolkit.

Copying
-------

Copyright (c) 2010 by Albert Gräf, all rights reserved. pure-tk is distributed
under a BSD-style license, see the COPYING file for details.

Installation
------------

Get the latest source from
<https://bitbucket.org/purelang/pure-lang/downloads/pure-tk-0.5.tar.gz>.

As with the other addon modules for Pure, running `make && sudo make install`
should usually do the trick. This requires that you have Pure and Tcl/Tk
installed. `make` tries to guess your Pure installation directory and
platform-specific setup. If it gets this wrong, you can set some variables
manually. In particular, `make install prefix=/usr` sets the installation
prefix. Please see the Makefile for details.

------------------------------------------------------------------------------

> **Note:** When starting a new interpreter, the Tcl/Tk initialization code
> looks for some initialization files which it executes before anything else
> happens. Usually these files will be found without any further ado, but if
> that does not happen automatically, you must set the TCL\_LIBRARY and
> TK\_LIBRARY environment variables to point to the Tcl and Tk library
> directories on your system.

------------------------------------------------------------------------------

All programs in the examples subdirectory have been set up so that they can be
compiled to native executables, and a Makefile is provided in that directory
to handle this. So after installing pure-tk you just need to type `make` there
to compile the examples. (This step isn't necessary, though, you can also just
run the examples with the Pure interpreter as usual.)

Basic Usage
-----------

<a name="tk"></a>`tk cmd`
:   execute a Tcl command

<!-- -->
You can submit a command to the Tcl interpreter with `tk cmd` where `cmd` is a
string containing the command to be executed. If the Tcl command returns a
value (i.e., a nonempty string) then [`tk`](#tk) returns that string,
otherwise it returns `()`.

[`tk`](#tk) also starts a new instance of the Tcl interpreter if it is not
already running. To stop the Tcl interpreter, you can use the
[`tk_quit`](#tk_quit) function.

<a name="tk_quit"></a>`tk_quit`
:   stop the Tcl interpreter

<!-- -->
Note that, as far as pure-tk is concerned, there's only one Tcl interpreter
per process, but of course you can create secondary interpreter instances in
the Tcl interpreter using the appropriate Tcl commands.

Simple dialogs can be created directly using Tk's `tk_messageBox` and
`tk_dialog` functions. For instance:

    tk "tk_dialog .warning \"Warning\" \"Are you sure?\" warning 0 Yes No Cancel";

Other kinds of common dialogs are available; see the Tcl/Tk manual for
information.

For more elaborate applications you probably have to explicitly create some
widgets, add the appropriate callbacks and provide a main loop which takes
care of processing events in the Tcl/Tk GUI. We discuss this in the following.

Callbacks
---------

pure-tk installs a special Tcl command named `pure` in the interpreter which
can be used to implement callbacks in Pure. This command is invoked from Tcl
as follows:

    pure function args ...

It calls the Pure function named by the first argument, passing any remaining
(string) arguments to the callback. If the Pure callback returns a (nonempty)
string, that value becomes the return value of the `pure` command, otherwise
the result returned to the Tcl interpreter is empty.

Pure callbacks are installed on Tk widgets just like any other, just using the
`pure` command as the actual callback command. For instance, you can define a
callback which gets invoked when a button is pushed as follows:

    using tk, system;
    tk "button .b -text {Hello, world!} -command {pure hello}; pack .b";
    hello = puts "Hello, world!";

The Main Loop
-------------

<a name="tk_main"></a>`tk_main`
:   call the Tk main loop

<!-- -->
The easiest way to provide a main loop for your application is to just call
[`tk_main`](#tk_main) which keeps processing events in the Tcl interpreter
until the interpreter is exited. You can terminate the interpreter in a Pure
callback by calling [`tk_quit`](#tk_quit). Thus a minimalistic Tcl/Tk
application coded in Pure may look as follows:

    using tk;
    tk "button .b -text {Hello, world!} -command {pure tk_quit}; pack .b";
    tk_main;

The main loop terminates as soon as the Tcl interpreter is exited, which can
happen, e.g., in response to a callback which invokes the
[`tk_quit`](#tk_quit) function (as shown above) or Tcl code which destroys the
main window (`destroy .`). The user can also close the main window from the
window manager in order to exit the main loop.

Accessing Tcl Variables
-----------------------

<a name="tk_set"></a>`tk_set var val`, <a name="tk_unset"></a>`tk_unset var`, <a name="tk_get"></a>`tk_get var`
:   set and get Tcl variables

<!-- -->
pure-tk allows your script to set and retrieve variable values in the Tcl
interpreter with the [`tk_set`](#tk_set), [`tk_unset`](#tk_unset) and
[`tk_get`](#tk_get) functions. This is useful, e.g., to change the variables
associated with entry and button widgets, and to retrieve the current values
from the application. For instance:

    > tk_set "entry_val" "some string";
    "some string"
    > tk_get "entry_val";
    "some string"
    > tk_unset "entry_val";
    ()
    > tk_get "entry_val";
    tk_get "entry_val"

Note that [`tk_set`](#tk_set) returns the assigned value, so it is possible to
chain such calls if several variables have to be set to the same value:

    > tk_set "foo" $ tk_set "bar" "yes";
    "yes"
    > map tk_get ["foo","bar"];
    ["yes","yes"]

Conversions Between Pure and Tcl Values
---------------------------------------

As far as pure-tk is concerned, all Tcl values are strings (in fact, that's
just what they are at the Tcl language level, although the Tcl interpreter
uses more elaborate representations of objects such as lists internally).
There are no automatic conversions of any kind. Thus, the arguments passed to
a Pure callback and the result returned by [`tk`](#tk) are simply strings in
Pure land. The same holds for the [`tk_set`](#tk_set) and [`tk_get`](#tk_get)
functions.

However, there are a few helper functions which can be used to convert between
Tcl and Pure data. First, the following operations convert Pure lists to
corresponding Tcl lists and vice versa:

<a name="tk_join"></a>`tk_join xs`, <a name="tk_split"></a>`tk_split s`
:   convert between Pure and Tcl lists

<!-- -->
``` {.sourceCode .pure}
> tk_join ["0","1.0","Hello, world!"];
"0 1.0 {Hello, world!}"
> tk_split ans;
["0","1.0","Hello, world!"]
```

The [`tk_str`](#tk_str) and [`tk_val`](#tk_val) operations work in a similar
fashion, but they also do automatic conversions for numeric values (ints,
bigints and doubles):

<a name="tk_str"></a>`tk_str xs`, <a name="tk_val"></a>`tk_val s`
:   convert between Pure and Tcl values with numeric conversions

<!-- -->
``` {.sourceCode .pure}
> tk_str [0,1.0,"Hello, world!"];
"0 1.0 {Hello, world!}"
> tk_val ans;
[0,1.0,"Hello, world!"]
```

In addition, these operations also convert single atomic values:

    > tk_str 1.0;
    "1.0"
    > tk_val ans;
    1.0

Tips and Tricks
---------------

Here are a few other things that are worth keeping in mind when working with
pure-tk.

-   Errors in Tcl/Tk commands can be handled by giving an appropriate
    definition of the [`tk_error`](#tk_error) function, which is invoked with
    an error message as its single argument. For instance, the following
    implementation of [`tk_error`](#tk_error) throws an exception:

        tk_error msg = throw msg;

    If no definition for this function is provided, then errors cause a
    literal `tk_error msg` expression to be returned as the result of the
    [`tk`](#tk) function. You can then check for such results and take an
    appropriate action.

-   The Tcl interpreter, when started, displays a default main window, which
    is required by most Tk applications. If this is not desired (e.g., if only
    the basic Tcl commands are needed), you can hide this window using a
    `tk "wm withdraw ."` command. To redisplay the window when it is needed,
    use the `tk "wm deiconify ."` command. It is also common practice to use
    `wm withdraw` and `wm deiconify` while creating the widgets of an
    application, in order to reduce "flickering".
-   Instead of calling [`tk_main`](#tk_main), you can also code your own main
    loop in Pure as follows:

        main = do_something $$ main if tk_ready;
             = () otherwise;

    Note that the [`tk_ready`](#tk_ready) function checks whether the Tcl
    interpreter is still up and running, after processing any pending events
    in the interpreter. This setup allows you to do your own custom idle
    processing in Pure while the application is running. However, you have to
    be careful that your `do_something` routine runs neither too short nor too
    long (a few milliseconds should usually be ok). Otherwise your main loop
    may turn into a busy loop and/or the GUI may become very sluggish and
    unresponsive. Thus it's usually better to just call [`tk_main`](#tk_main)
    and do any necessary background processing using the Tcl interpreter's own
    facilities (e.g., by setting up a Pure callback with the Tcl `after`
    command).

-   The [`tk`](#tk) function can become rather tedious when coding larger Tk
    applications. Usually, you will prefer to put the commands making up your
    application into a separate Tcl script. One way to incorporate the Tcl
    script into your your Pure program is to use the Tcl `source` command,
    e.g.:

        tk "source myapp.tcl";

    However, this always requires the script to be available at runtime.
    Another method is to read the script into a string which is assigned to a
    Pure constant, and then invoke the [`tk`](#tk) command on this string
    value:

        using system;
        const ui = fget $ fopen "myapp.tcl" "r";
        tk ui;

    This still reads the script at runtime if the Pure program is executed
    using the Pure interpreter. However, you can now compile the Pure program
    to a native executable (see the Pure manual for details on this), in which
    case the text of the Tcl script is included verbatim in the executable.
    The compiled program can then be run without having the original Tcl
    script file available:

        $ pure -c myapp.pure -o myapp
        $ ./myapp

    This is also the method to use for running existing Tk applications, e.g.,
    if you create the interface using some interface builder like
    [vtcl](http://vtcl.sourceforge.net).

<!-- -->

-   The Tcl `package` command allows you to load additional extensions into
    the Tcl interpreter at runtime. For instance:

        tk "package require Gnocl";

    This loads Peter G. Baum's [Gnocl](http://www.gnocl.org/) extension which
    turns Tcl into a frontend for [GTK+](http://www.gtk.org/) and
    [Gnome](http://www.gnome.org/). In fact, pure-tk includes a special module
    to handle the nitty-gritty details of creating a GTK+/Gnome application
    from a [Glade](http://glade.gnome.org/) UI file and set up Pure callbacks
    as specified in the UI file. To use this, just import the gnocl.pure
    module into your Pure scripts:

        using gnocl;

    Note that the Glade interface requires that you have a fairly recent
    version of Gnocl installed (Gnocl 0.9.94g has been tested). The other
    facilities provided by the gnocl.pure module should also work with older
    Gnocl versions such as Gnocl 0.9.91. Please see the gnocl.pure module and
    the corresponding examples included in the sources for more information.

-   The Tcl `exit` procedure, just as in tclsh or wish, causes exit from the
    current process. Since the Tcl interpreter hosted by the pure-tk module
    runs as part of a Pure program and not as a separate child process, this
    might not be what you want. If you'd like `exit` to only exit the Tcl
    interpreter, without exiting the Pure program, you can redefine the `exit`
    procedure, e.g., as follows:

        tk "proc exit { {returnCode 0} } { pure tk_quit }";

    If you want to do something with the exit code provided by `exit`, you
    will have to provide an appropriate callback function, e.g.:

        tk "proc exit { {returnCode 0} } { pure quit_cb $returnCode }";

    A suitable implementation of `quit_cb` might look as follows:

        quit_cb 0 = puts "Application exited normally." $$ tk_quit;
        quit_cb n = printf "Application exited with exit code %d.\n" n $$
                    tk_quit otherwise;

-   If you need dialogs beyond the standard kinds of message boxes and common
    dialogs, you will have to do these yourself using a secondary toplevel.
    The dialog toplevel is just like the main window but will only be shown
    when the application needs it. You can construct both non-modal and modal
    dialogs this way, the latter can be implemented using Tk's `grab` command.

