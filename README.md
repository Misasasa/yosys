```
yosys -- Yosys Open SYnthesis Suite

Copyright (C) 2012 - 2016  Clifford Wolf <clifford@clifford.at>

Permission to use, copy, modify, and/or distribute this software for any
purpose with or without fee is hereby granted, provided that the above
copyright notice and this permission notice appear in all copies.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
```


yosys – Yosys Open SYnthesis Suite
===================================

This is a framework for RTL synthesis tools. It currently has
extensive Verilog-2005 support and provides a basic set of
synthesis algorithms for various application domains.

Yosys can be adapted to perform any synthesis job by combining
the existing passes (algorithms) using synthesis scripts and
adding additional passes as needed by extending the yosys C++
code base.

Yosys is free software licensed under the ISC license (a GPL
compatible license that is similar in terms to the MIT license
or the 2-clause BSD license).


Web Site
========

More information and documentation can be found on the Yosys web site:
http://www.clifford.at/yosys/


Getting Started
===============

You need a C++ compiler with C++11 support (up-to-date CLANG or GCC is
recommended) and some standard tools such as GNU Flex, GNU Bison, and GNU Make.
TCL, readline and libffi are optional (see ENABLE_* settings in Makefile).
Xdot (graphviz) is used by the ``show`` command in yosys to display schematics.
For example on Ubuntu Linux 16.04 LTS the following commands will install all
prerequisites for building yosys:

	$ sudo apt-get install build-essential clang bison flex \
		libreadline-dev gawk tcl-dev libffi-dev git mercurial \
		graphviz xdot pkg-config python3

There are also pre-compiled Yosys binary packages for Ubuntu and Win32 as well
as a source distribution for Visual Studio. Visit the Yosys download page for
more information: http://www.clifford.at/yosys/download.html

To configure the build system to use a specific compiler, use one of

	$ make config-clang
	$ make config-gcc

For other compilers and build configurations it might be
necessary to make some changes to the config section of the
Makefile.

	$ vi Makefile            # ..or..
	$ vi Makefile.conf

To build Yosys simply type 'make' in this directory.

	$ make
	$ make test
	$ sudo make install

Note that this also downloads, builds and installs ABC (using yosys-abc
as executable name).

Yosys can be used with the interactive command shell, with
synthesis scripts or with command line arguments. Let's perform
a simple synthesis job using the interactive command shell:

	$ ./yosys
	yosys>

the command ``help`` can be used to print a list of all available
commands and ``help <command>`` to print details on the specified command:

	yosys> help help

reading the design using the Verilog frontend:

	yosys> read_verilog tests/simple/fiedler-cooley.v

writing the design to the console in yosys's internal format:

	yosys> write_ilang

elaborate design hierarchy:

	yosys> hierarchy

convert processes (``always`` blocks) to netlist elements and perform
some simple optimizations:

	yosys> proc; opt

display design netlist using ``xdot``:

	yosys> show

the same thing using ``gv`` as postscript viewer:

	yosys> show -format ps -viewer gv

translating netlist to gate logic and perform some simple optimizations:

	yosys> techmap; opt

write design netlist to a new Verilog file:

	yosys> write_verilog synth.v

a similar synthesis can be performed using yosys command line options only:

	$ ./yosys -o synth.v -p hierarchy -p proc -p opt \
	                     -p techmap -p opt tests/simple/fiedler-cooley.v

or using a simple synthesis script:

	$ cat synth.ys
	read_verilog tests/simple/fiedler-cooley.v
	hierarchy; proc; opt; techmap; opt
	write_verilog synth.v

	$ ./yosys synth.ys

It is also possible to only have the synthesis commands but not the read/write
commands in the synthesis script:

	$ cat synth.ys
	hierarchy; proc; opt; techmap; opt

	$ ./yosys -o synth.v tests/simple/fiedler-cooley.v synth.ys

The following very basic synthesis script should work well with all designs:

	# check design hierarchy
	hierarchy

	# translate processes (always blocks)
	proc; opt

	# detect and optimize FSM encodings
	fsm; opt

	# implement memories (arrays)
	memory; opt

	# convert to gate logic
	techmap; opt

If ABC is enabled in the Yosys build configuration and a cell library is given
in the liberty file ``mycells.lib``, the following synthesis script will synthesize
for the given cell library:

	# the high-level stuff
	hierarchy; proc; fsm; opt; memory; opt

	# mapping to internal cell library
	techmap; opt

	# mapping flip-flops to mycells.lib
	dfflibmap -liberty mycells.lib

	# mapping logic to mycells.lib
	abc -liberty mycells.lib

	# cleanup
	clean

If you do not have a liberty file but want to test this synthesis script,
you can use the file ``examples/cmos/cmos_cells.lib`` from the yosys sources.

Liberty file downloads for and information about free and open ASIC standard
cell libraries can be found here:

- http://www.vlsitechnology.org/html/libraries.html
- http://www.vlsitechnology.org/synopsys/vsclib013.lib

The command ``synth`` provides a good default synthesis script (see ``help synth``).
If possible a synthesis script should borrow from ``synth``. For example:

	# the high-level stuff
	hierarchy
	synth -run coarse

	# mapping to internal cells
	techmap; opt -fast
	dfflibmap -liberty mycells.lib
	abc -liberty mycells.lib
	clean

Yosys is under construction. A more detailed documentation will follow.


Unsupported Verilog-2005 Features
=================================

The following Verilog-2005 features are not supported by
yosys and there are currently no plans to add support
for them:

- Non-synthesizable language features as defined in
	IEC 62142(E):2005 / IEEE Std. 1364.1(E):2002

- The ``tri``, ``triand``, ``trior``, ``wand`` and ``wor`` net types

- The ``config`` keyword and library map files

- The ``disable``, ``primitive`` and ``specify`` statements

- Latched logic (is synthesized as logic with feedback loops)


Verilog Attributes and non-standard features
============================================

- The ``full_case`` attribute on case statements is supported
  (also the non-standard ``// synopsys full_case`` directive)

- The ``parallel_case`` attribute on case statements is supported
  (also the non-standard ``// synopsys parallel_case`` directive)

- The ``// synopsys translate_off`` and ``// synopsys translate_on``
  directives are also supported (but the use of ``` `ifdef .. `endif ```
  is strongly recommended instead).

- The ``nomem2reg`` attribute on modules or arrays prohibits the
  automatic early conversion of arrays to separate registers. This
  is potentially dangerous. Usually the front-end has good reasons
  for converting an array to a list of registers. Prohibiting this
  step will likely result in incorrect synthesis results.

- The ``mem2reg`` attribute on modules or arrays forces the early
  conversion of arrays to separate registers.

- The ``nomeminit`` attribute on modules or arrays prohibits the
  creation of initialized memories. This effectively puts ``mem2reg``
  on all memories that are written to in an ``initial`` block and
  are not ROMs.

- The ``nolatches`` attribute on modules or always-blocks
  prohibits the generation of logic-loops for latches. Instead
  all not explicitly assigned values default to x-bits. This does
  not affect clocked storage elements such as flip-flops.

- The ``nosync`` attribute on registers prohibits the generation of a
  storage element. The register itself will always have all bits set
  to 'x' (undefined). The variable may only be used as blocking assigned
  temporary variable within an always block. This is mostly used internally
  by yosys to synthesize Verilog functions and access arrays.

- The ``onehot`` attribute on wires mark them as onehot state register. This
  is used for example for memory port sharing and set by the fsm_map pass.

- The ``blackbox`` attribute on modules is used to mark empty stub modules
  that have the same ports as the real thing but do not contain information
  on the internal configuration. This modules are only used by the synthesis
  passes to identify input and output ports of cells. The Verilog backend
  also does not output blackbox modules on default.

- The ``keep`` attribute on cells and wires is used to mark objects that should
  never be removed by the optimizer. This is used for example for cells that
  have hidden connections that are not part of the netlist, such as IO pads.
  Setting the ``keep`` attribute on a module has the same effect as setting it
  on all instances of the module.

- The ``keep_hierarchy`` attribute on cells and modules keeps the ``flatten``
  command from flattening the indicated cells and modules.

- The ``init`` attribute on wires is set by the frontend when a register is
  initialized "FPGA-style" with ``reg foo = val``. It can be used during synthesis
  to add the necessary reset logic.

- The ``top`` attribute on a module marks this module as the top of the
  design hierarchy. The ``hierarchy`` command sets this attribute when called
  with ``-top``. Other commands, such as ``flatten`` and various backends
  use this attribute to determine the top module.

- The ``src`` attribute is set on cells and wires created by to the string
  ``<hdl-file-name>:<line-number>`` by the HDL front-end and is then carried
  through the synthesis. When entities are combined, a new |-separated
  string is created that contains all the string from the original entities.

- In addition to the ``(* ... *)`` attribute syntax, yosys supports
  the non-standard ``{* ... *}`` attribute syntax to set default attributes
  for everything that comes after the ``{* ... *}`` statement. (Reset
  by adding an empty ``{* *}`` statement.)

- In module parameter and port declarations, and cell port and parameter
  lists, a trailing comma is ignored. This simplifies writing verilog code
  generators a bit in some cases.

- Modules can be declared with ``module mod_name(...);`` (with three dots
  instead of a list of module ports). With this syntax it is sufficient
  to simply declare a module port as 'input' or 'output' in the module
  body.

- When defining a macro with `define, all text between triple double quotes
  is interpreted as macro body, even if it contains unescaped newlines. The
  tipple double quotes are removed from the macro body. For example:

      `define MY_MACRO(a, b) """
         assign a = 23;
         assign b = 42;
      """

- The attribute ``via_celltype`` can be used to implement a Verilog task or
  function by instantiating the specified cell type. The value is the name
  of the cell type to use. For functions the name of the output port can
  be specified by appending it to the cell type separated by a whitespace.
  The body of the task or function is unused in this case and can be used
  to specify a behavioral model of the cell type for simulation. For example:

      module my_add3(A, B, C, Y);
        parameter WIDTH = 8;
        input [WIDTH-1:0] A, B, C;
        output [WIDTH-1:0] Y;
        ...
      endmodule

      module top;
        ...
        (* via_celltype = "my_add3 Y" *)
        (* via_celltype_defparam_WIDTH = 32 *)
        function [31:0] add3;
          input [31:0] A, B, C;
          begin
            add3 = A + B + C;
          end
        endfunction
        ...
      endmodule

- A limited subset of DPI-C functions is supported. The plugin mechanism
  (see ``help plugin``) can be used to load .so files with implementations
  of DPI-C routines. As a non-standard extension it is possible to specify
  a plugin alias using the ``<alias>:`` syntax. For example:

      module dpitest;
        import "DPI-C" function foo:round = real my_round (real);
        parameter real r = my_round(12.345);
      endmodule

      $ yosys -p 'plugin -a foo -i /lib/libm.so; read_verilog dpitest.v'

- Sized constants (the syntax ``<size>'s?[bodh]<value>``) support constant
  expressions as <size>. If the expression is not a simple identifier, it
  must be put in parentheses. Examples: ``WIDTH'd42``, ``(4+2)'b101010``

- The system tasks ``$finish`` and ``$display`` are supported in initial blocks
  in an unconditional context (only if/case statements on parameters
  and constant values). The intended use for this is synthesis-time DRC.


Non-standard or SystemVerilog features for formal verification
==============================================================

- Support for ``assert``, ``assume``, and ``restrict`` is enabled when
  ``read_verilog`` is called with ``-formal``.

- The system task ``$initstate`` evaluates to 1 in the initial state and
  to 0 otherwise.

- The system task ``$anyconst`` evaluates to any constant value.

- The system task ``$anyseq`` evaluates to any value, possibly a different
  value in each cycle.

- The SystemVerilog tasks ``$past``, ``$stable``, ``$rose`` and ``$fell`` are supported
  in any clocked block.

- The syntax ``@($global_clock)`` can be used to create FFs that have no
  explicit clock input ($ff cells).


Supported features from SystemVerilog
=====================================

When ``read_verilog`` is called with ``-sv``, it accepts some language features
from SystemVerilog:

- The ``assert`` statement from SystemVerilog is supported in its most basic
  form. In module context: ``assert property (<expression>);`` and within an
  always block: ``assert(<expression>);``. It is transformed to a $assert cell.

- The ``assume`` and ``restrict`` statements from SystemVerilog are also
  supported. The same limitations as with the ``assert`` statement apply.

- The keywords ``always_comb``, ``always_ff`` and ``always_latch``, ``logic`` and
  ``bit`` are supported.

- SystemVerilog packages are supported. Once a SystemVerilog file is read
  into a design with ``read_verilog``, all its packages are available to
  SystemVerilog files being read into the same design afterwards.


Building the documentation
==========================

Note that there is no need to build the manual if you just want to read it.
Simply download the PDF from http://www.clifford.at/yosys/documentation.html
instead.

On Ubuntu, texlive needs these packages to be able to build the manual:

	sudo apt-get install texlive-binaries
	sudo apt-get install texlive-science      # install algorithm2e.sty
	sudo apt-get install texlive-bibtex-extra # gets multibib.sty
	sudo apt-get install texlive-fonts-extra  # gets skull.sty and dsfont.sty
	sudo apt-get install texlive-publishers   # IEEEtran.cls

Also the non-free font luximono should be installed, there is unfortunately
no Ubuntu package for this so it should be installed separately using
`getnonfreefonts`:

	wget https://tug.org/fonts/getnonfreefonts/install-getnonfreefonts
	sudo texlua install-getnonfreefonts # will install to /usr/local by default, can be changed by editing BINDIR at MANDIR at the top of the script
	getnonfreefonts luximono # installs to /home/user/texmf

Then execute, from the root of the repository:

	make manual

Notes:

- To run `make manual` you need to have installed yosys with `make install`,
  otherwise it will fail on finding `kernel/yosys.h` while building
  `PRESENTATION_Prog`.
